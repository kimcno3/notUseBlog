---
layout: post
title: "[football] @AuthenticationPrincipal을 활용한 로그인 회원 정보 조회"
author: kimcno3
categories: f-lab
tags: f-lab project2
---

## # 문제점
***
프로젝트 진행시 빈도수가 높게 필요한 기능이 컨트롤러 레이어에서 파라미터로 로그인한 user의 정보를 가져오는 기능입니다. 이를 구현하기 위해 이전 프로젝트에선 Custom Resolver 클래스를 생성해 직접 회원 정보를 가져오는 로직을 구현해야 하는 소요가 발생했었습니다. 

또한 회원정보를 조회하는 로직을 실행하는 경우 필연적으로 DB에 Select문을 보내야하며 이에 따른 성능적인 부분에서 과부화가 올 수도 있는 위험도 존재한다고 볼 수 있습니다.

하지만 Spring Security에선 `@AuthenticationPrincipal`을 사용해 위와 같은 문제를 간단하게 해결할 수 있습니다.

## # 해결 방안 
***
### `@AuthenticationPrincipal`
UserDetailsService 구현 클래스의 `loadUserByUserName()` 의 리턴 객체를 컨트롤러 레이어의 매개변수로 가져올 수 있도록 설정된 어노테이션입니다. 리턴 객체 타입은 UserDetails 인터페이스 타입을 리턴합니다.

#### 예제 코드
```java
  @GetMapping("/login/check")
  public ResponseDto logInCheck(@AuthenticationPrincipal UserDetails user) {

    log.info("user.userName = {}", user.getUsername()); // user.userName = 10

    return new ResponseDto(true, null, "로그인 체크", null);

  }
```

위 예제처럼 @AuthenticationPrincipal을 매개변수에 선언하면 어떠한 설정이나 추가 작업 없이 UserDetails 타입의 객체로 user 정보를 조회할 수 있습니다. 또한 [Jwt Token에 담길 사용자 정보에 대한 결정과 표준에 대한 이해](https://velog.io/@kimcno3/Jwt-Token%EC%97%90-%EB%8B%B4%EA%B8%B8-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EB%B3%B4%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B2%B0%EC%A0%95%EA%B3%BC-%ED%91%9C%EC%A4%80%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%ED%95%B4)를 보면 Jwt 토큰의 표준 양식으로 subject에 userId를 저장해두었기에 UserDetails 객체의 username()으로 userId를 조회할 수 있습니다.

이렇게  `@AuthenticationPrincipal`를 활용해 원하는 API에 userId를 통해 회원 정보를 조회해 활용해볼 수 있습니다.

## # 마치며
***
이렇게 Spring Security를 활용해본다면 로그인과 관련된 여러 기능을 제공해주기 때문에 따로 클래스를 커스텀으로 구현하지 않아도 되고, 인증, 인가에 대한 처리를 Spring Security가 전담하도록 책임의 구분도 명확히 할 수 있다는 장점이 있습니다.

## # 이후 작업 예정 사항
***
이후 UserDetails 객체를 직접 사용하는 것이 아니라 user 객체와 추가적인 정보를 담아놓을 수 있는 DTO 객체를 추가로 생성해 `@AuthenticationPrincipal`을 통해 호출해올 수 있도록 추가 리팩토링 작업을 진행할 예정입니다.

## # 참고 자료
***
https://velopert.com/2389