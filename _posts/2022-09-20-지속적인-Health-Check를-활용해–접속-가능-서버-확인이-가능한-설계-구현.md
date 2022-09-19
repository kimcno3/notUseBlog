---
layout: post
title: "[football] 지속적인 Health Check를 활용해 접속 가능 서버 확인이 가능한 설계 구현"
author: kimcno3
categories: f-lab
tags:  f-lab project2
published: true
---

## # 문제점
***
API 서버와 웹소켓 구분하고 각 서버를 여러 서버로 Scale Out 할 수 있는 구조로 설계함으로써 관리해야 될 서버의 수가 늘어나면서 각 웹소켓이 다운되는 것으로 인해 발생할 수 있는 장애를 제어해야 할 필요가 있다고 판단했습니다. 그 중 하나의 경우로, 여러 웹소켓 중 다운이 되어 동작할 수 없는 경우에 해당 웹소켓으로 사용자가 접속하지 못하도록 구성할 필요가 있었습니다.

## # 해결방안
***
### Health Check
Health Check 기능은 다수의 서버의 작동 상태에 대한 정보를 주기적으로 체크하면서 문제가 생기는 지 모니터링 하는 것을 의미합니다.

현재 프로젝트 구조에선 채팅 서버가 자신들의 Heart Beat를 API 서버에 주기적으로 전송하는 방식으로 Health Check 기능을 구현했습니다. 그리고 채팅서버에서 보내준 정보를 API 서버에선 Redis에 저장하도록 구성했습니다.

### @Scheduled
`@Scheduled` 일정한 시간 간격으로, 혹은 특정 일정에 코드가 실행되도록 해주는 어노테이션입니다. 그래서 `@Scheduled`를 활용해 채팅 서버에서 주기적으로 Heart Beat를 API 서버에 보내주도록 메소드를 선언해봤습니다.

### Health Check 구현코드(채팅 서버)

#### WebsocketController
```java
@RestController
@RequestMapping("/ws")
@RequiredArgsConstructor
public class WebSocketController {

  private final HeartBeatService heartBeatService;
  
  // 3초 간격으로 해당 메소드를 실행
  @Scheduled(fixedRate = 3000)
  public void heartBeat() {

    heartBeatService.sendHeartBeat();

  }
  
}
```
우선 `WebsocketController`에서 `@Scheduled`를 선언된 메소드를 생성하고 Heart Beat 기능을 수행하는 로직을 호출하도록 구성했습니다. 그리고 고정적 주기로 3초에 한번씩 해당 로직이 수행하도록 구성했습니다.

#### HeartBeatServiceImpl
```java

@Service
@RequiredArgsConstructor
public class HeartBeatServiceImpl implements HeartBeatService {

  private final RestTemplate restTemplate;

  private final SessionService sessionService;

  @Value("${server.host.chatting.public}")
  private String publicChattingServerAddress;

  @Value("${server.host.api}")
  private String apiAddress;

  public void sendHeartBeat() {

    HeartBeatRequest request = HeartBeatRequest.builder()
        .address(publicChattingServerAddress) // 현재 채팅 서버 주소
        .connectionCount(sessionService.getSessionCount()) // 연결된 websocket Session 수
        .heartBeatTime(LocalDateTime.now()) // 체크 시간
        .build();

    // API 서버에 Post 요청
    restTemplate.postForObject(
        "http://" + apiAddress + "/chat/health/check",
        request,
        ResponseDto.class
    );

  }

}
```
Heart Beat 로직의 구성 코드는 현재 채팅 서버의 주소, 연결된 Websocket Session 수, 체크 시간을 Request 객체에 담에 API 서버에 Post 요청을 보냅니다.

### Health Check 구현코드(API 서버)

#### ChatController
```java
@RestController
@RequestMapping("/chat")
@RequiredArgsConstructor
public class ChatController {

  private final ChatService chatService;

  @PostMapping("/health/check")
  public ResponseDto healthCheck(@RequestBody HealthCheckRequest request) {

    chatService.healthCheck(
        request.getAddress(),
        request.getConnectionCount(),
        request.getHeartBeatTime()
    );

    return new ResponseDto(true, null, "헬스 체크 완료", null);

  }
}
```
채팅 서버에서 보낸 Heart Beat 정보를 처리해주는 API 입니다.

#### ChatServiceImpl
```java
@Service
@RequiredArgsConstructor
public class ChatServiceImpl implements ChatService {

  private final RedisService redisService;
  
  @Override
  @Transactional
  public void healthCheck(String address, int connectionCount, LocalDateTime lastHeartBeatTime) {

    redisService.setWebSocketServerInfo(address, connectionCount, lastHeartBeatTime);

  }
  
}
 
```
ChatService에서 Redis에 해당 Heart beat 정보를 저장하는 메소드를 호출합니다.

#### RedisServiceImpl
```java
@Service
@RequiredArgsConstructor
public class RedisServiceImpl implements RedisService {

  @Override
  public void setWebSocketServerInfo(String address, int connectionCount, LocalDateTime lastHeartBeatTime) {

    HashOperations<String, String, Object> hashOperations = redisTemplate.opsForHash();

    String key = WebSocketUtils.PREFIX_SERVER + address;

    hashOperations.put(key, WebSocketUtils.ADDRESS, address);

    hashOperations.put(key, WebSocketUtils.CONNECTION_COUNT, connectionCount);

    hashOperations.put(key, WebSocketUtils.LAST_HEARTBEAT_TIME, lastHeartBeatTime);
  }
}
```
Redis에 저장되는 자료구조는 HashMap을 선택했고, 채팅 서버의 주소를 조합한 key 안에 3가지 정보를 저장해두는 구조로 구현했습니다.

### Health Check 동작 확인

Health Check가 될 때마다 로그를 확인해보면 위 사진처럼 정상적으로 모니터링이 되는 것을 확인할 수 있습니다.

### 서버 다운 확인 로직 구현코드
추가적으로 채팅서버가 다운될 경우 해당 서버에 대한 정보를 삭제해둬야 이후 최적의 채팅 서버 주소를 리턴해주는 로직에서 다운된 서버는 후보군에서 제외될 수 있을 것이라 판단했습니다.

#### ChatServiceImpl
```java
public class ChatServiceImpl implements ChatService {

  @Scheduled(fixedRate = 3000)
  public void deleteWebSocketServer() {

    // 모든 Chatting 서버의 서버 정보에 대한 키들을 조회
    Cursor<String> keys = redisService.scanWebSocketServerKey();

    if (keys.hasNext()) {

      String key = keys.next();

      long lastHeartBeatTime = redisService.getWebSocketLastHeartBeatTime(key)
          .toEpochSecond(ZoneOffset.UTC);

      long currentTime = LocalDateTime.now()
          .toEpochSecond(ZoneOffset.UTC);

      // 최근 Health Check 시간이 10초 이상이라면 죽은 서버로 판단하고 데이터를 삭제합니다.
      if (currentTime - lastHeartBeatTime > 10) {
        
        // Redis에서 서버 정보에 대한 데이터를 삭제하는 메소드
        redisService.deleteWebSocketServerInfo(key);

      }

    }

  }
}
```

그래서 주기적으로 API 서버에선 채팅 서버의 연결 상태를 확인하고 만약 10초 이상 Health Check가 안되는 서버라면 죽은 서버라 판단하고 데이터를 삭제하도록 구현했습니다.

## # 마치며
***
Health Check 기능을 통해 다운된 서버에 대해 클라이언트의 요청이 전송되지 않도록 구성해 어플리케이션 동작시 발생할 수 있는 장애에 대한 위험을 줄일 수 있었습니다. 또한 `@Scheduled`를 활용해 주기적으로 수행되어야 하는 메소드를 구현해보는 경험을 할 수 있었습니다.

## # 참고자료
***
- https://www.baeldung.com/spring-scheduled-tasks
- https://data-make.tistory.com/699