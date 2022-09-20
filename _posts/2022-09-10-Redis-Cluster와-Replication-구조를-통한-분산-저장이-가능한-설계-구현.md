---
layout: post
title: "[football] Redis Cluster와 Replication 구조를 통한 분산 저장이 가능한 설계 구현"
author: kimcno3
categories: f-lab
tags: f-lab project2
---

## # 문제점
***
Redis를 구성할 때, 하나의 서버로만 구성하다 보니 다음과 같은 문제점이 발생할 수 있는 가능성이 높았습니다.

- 서버 과부화
- 서버 다운시 발생하는 치명적인 서비스 장애

그래서 이러한 문제점을 해결하기 위해 다중 서버를 통해 요청에 대한 분산 처리가 가능한 구조로 구현하면서 Redis 또한 이와 같은 구조로 리팩토링할 필요가 있었습니다.

Redis Cluster로 구성하는 것이였고 Redis Cluster를 통해 제가 직접 어떠한 로직을 구현하지 않고도 Redis 내부적으로 저장될 데이터에 대한 분산 처리를 담당해줌으로써 효율적인 다중 서버 구조를 구현할 수 있었습니다. 추가로 각 Redis Node들에 대해서 Replication 구조를 추가적으로 설계함으로써 Master 노드가 다운된 경우에도 Slave 서버가 승격함으로써 데이터 처리에 대한 장애가 발생하지 않도록 구성했습니다.

## # 해결 방안
***
### 1. Redis Cluster

Redis Cluster는 요청 처리에 대한 과부화를 방지하기 위해 여러 개의 노드 서버를 구성해 데이터를 분산 저장 및 처리 할 수 있도록 구성하는 것을 의미합니다.

![](https://blog.kakaocdn.net/dn/7DUkE/btq4AuCX4X0/QKWCiSwAvJiHO7G4qBA9X0/img.png)

Redis Cluster로 구성하게 되면 위 그림처럼 각 노드별로 slot(1~16383)을 나눠 할당받아, 할당된 slot에 대해서만 처리하는 책임을 가지게 됩니다. 만약 하나의 데이터가 저장된 slot이 1이라면 A 노드에서 처리할 수 있도록 redis에서 관리해 준다는 장점도 있습니다.

Redis Cluster로 구성하게 되면 의도한대로 분산처리 구조를 가지고 Scale Out이 가능하게 됩니다.

하지만 Redis Cluster를 통해 노드의 수만 증가시킨 구조라면 해결되지 않는 문제점이 있습니다. 각 노드간의 데이터 공유는 되지 않아 하나의 노드가 다운될 경우, 다운된 노드에 저장된 데이터에 대해 분실할 수 있는 위험이 있습니다. 즉, 분산 처리에 대한 장점만 가지고 있는 형태라고 할 수 있습니다.

### 2. Replication

Replication 구조는 이전 프로젝트에서도 다루었듯이 서버 다운에 대한 위험요소를 즉각적으로 대처할 수 있도록 해줍니다.

> 이전 프로젝트에서 [Replication에 대한 개념을 정리한 글](https://github.com/kimcno3/resume/blob/main/portfolio/project2/troublesShooting/8_replication.md)

만약 Master 노드 서버가 다운될 경우 Slave 서버가 Master 서버에 대한 데이터를 이어 받고 Master 서버로 대체되어 데이터 처리에 대한 문제가 없도록 구성했습니다.

### 3. 작성 코드
#### 노드별 conf 파일
```
port 7001 // 노드별 다른 port로 지정
cluster-enabled yes
cluster-config-file node.conf
cluster-node-timeout 5000
appendonly yes
```

Redis Cluster를 구성하기 위해선 각 노드별 conf 파일을 구성해야 합니다. 저는 master1.conf, master2.conf ... slave3.conf 방식으로 총 6개의 conf 파일을 생성했고 port는 7001 ~ 7006까지 사용했습니다.

#### docker-compose.yml
```yaml
version : "3"
services:

  # 노드 생성

  redis-cluster:
    container_name: redis-master-1
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/master1.conf:/usr/local/etc/redis/redis.conf # 생성한 conf 파일의 위치와 파일명에 맞게 지정
    restart: always
    ports:
      - "7001:7001"
      - "7002:7002"
      - "7003:7003"
      - "7004:7004"
      - "7005:7005"
      - "7006:7006"

  redis-master-2:
    network_mode: "service:redis-cluster"
    container_name: redis-master-2
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/master2.conf:/usr/local/etc/redis/redis.conf
    restart: always

  redis-master-3:
    network_mode: "service:redis-cluster"
    container_name: redis-master-3
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/master3.conf:/usr/local/etc/redis/redis.conf
    restart: always

  redis-slave-1:
    network_mode: "service:redis-cluster"
    container_name: redis-slave-1
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/slave1.conf:/usr/local/etc/redis/redis.conf
    restart: always

  redis-slave-2:
    network_mode: "service:redis-cluster"
    container_name: redis-slave-2
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/slave2.conf:/usr/local/etc/redis/redis.conf
    restart: always

  redis-slave-3:
    network_mode: "service:redis-cluster"
    container_name: redis-slave-3
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis/slave3.conf:/usr/local/etc/redis/redis.conf
    restart: always

  # Master 노드를 Cluster 모드로 생성
  redis-cluster-entry:
    network_mode: "service:redis-cluster"
    platform: linux/x86_64
    image: redis:latest
    container_name: football-redis-cluster-entry
    command: redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 --cluster-yes
    depends_on:
      - redis-cluster
      - redis-master-2
      - redis-master-3

  # Master 노드별 Slave 노드 구성

  redis-cluster-replicas-1:
    network_mode: "service:redis-cluster"
    platform: linux/x86_64
    image: redis:latest
    container_name: football-redis-cluster-replicas-1
    command: redis-cli --cluster add-node 127.0.0.1:7004 127.0.0.1:7001 --cluster-slave
    depends_on:
      - redis-slave-1

  redis-cluster-replicas-2:
    network_mode: "service:redis-cluster"
    platform: linux/x86_64
    image: redis:latest
    container_name: football-redis-cluster-replicas-2
    command: redis-cli --cluster add-node 127.0.0.1:7005 127.0.0.1:7002 --cluster-slave
    depends_on:
      - redis-slave-2

  redis-cluster-replicas-3:
    network_mode: "service:redis-cluster"
    platform: linux/x86_64
    image: redis:latest
    container_name: football-redis-cluster-replicas-3
    command: redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7003 --cluster-slave
    depends_on:
      - redis-slave-3


```

Redis를 Docker에 구성했기 때문에 Docker-compose 파일도 변경해야 했습니다. 코드 동작 순서는 우선적으로 각 노드를 생성하고 Master 노드로 활용할 노드들을 Cluster로 생성한 다음, Master 노드별 slave 노드를 연결해주는 순으로 구성했습니다.

> 각 노드별로 지정된 conf 파일이 연결되도록 volume 변수값을 수정해야 합니다.


#### application.yml
```yaml
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:7001
        - 127.0.0.1:7002
        - 127.0.0.1:7003
        - 127.0.0.1:7004
        - 127.0.0.1:7005
        - 127.0.0.1:7006
```

총 6개의 노드가 생성되었고 이에 대한 주소값을 가지고 있도록 application.yml 파일도 수정되었고, 위와 같이 작성하게 되면 하나의 배열로 저장된 값을 매핑할 수 있게 됩니다.

#### RedisConfig 클래스
```java
@Getter
@Setter
@Configuration
@ConfigurationProperties(prefix = "spring.redis.cluster") // appilcation.yml 파일에서 추적할 위치를 지정
public class RedisConfig {

  private List<String> nodes; // 변수명까지 매핑해 원하는 배열을 가져온다.


  @Bean
  public RedisConnectionFactory redisConnectionFactory() {

    LettuceClientConfiguration clientConfiguration = LettuceClientConfiguration.builder()
        .readFrom(ReadFrom.REPLICA_PREFERRED) // Slave 노드에 우선으로 접근
        .build();

    RedisClusterConfiguration redisClusterConfig = new RedisClusterConfiguration(nodes);

    return new LettuceConnectionFactory(redisClusterConfig, clientConfiguration);

  }

}

```
Redis 구성 클래스도 application.yml에서 저장한 노드들의 주소값을 배열로 가져오기 위해 `@ConfigurationProperties`를 활용했고 `RedisConnectionFactory` 객체 생성시 RedisClusterConfig 객체를 매개변수로 담아 넘겨주도록 합니다. 

또한 Master-Slave 구조에서 우선적으로 접근할 대상 노드를 Slave 노드로 지정해, Slave 노드가 다운된 경우에 Master 노드로 접근하도록 구성했습니다. (`.readFrom(ReadFrom.REPLICA_PREFERRED)`)

## # 마치며
***
이처럼 Redis Cluster와 Replication 구조를 구성함으로써 트래픽 과부화와 서버 다운시 발생할 수 있는 데이터 분실에 대한 위험을 줄이기 위한 고민을 해결할 수 있었고, Redis에서 제공해주는 기능을 활요하기 위해 직접 코드를 수정해보고 고민해보는 시간을 가질 수 있었습니다.

## # 참고 자료
***
- https://velog.io/@18k7102dy/Redis-Redis-cluster-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0
- https://co-de.tistory.com/24
- https://daddyprogrammer.org/post/1601/redis-cluster/
- https://xzio.tistory.com/1599
- https://brunch.co.kr/@springboot/218