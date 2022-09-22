---
layout: post
title: "[soldout] JUnit5, Mockito를 활용한 효율적인 단위 테스트"
author: kimcno3
categories: f-lab
tags: f-lab project1
---

## # 문제점
***
애플리케이션 개발에서 작성한 코드에 대한 테스트는 필수적인 절차지만, 항상 실제 서버에 JSON 요청을 보내는 테스트를 통해서 비즈니스 로직의 문제점을 확인하기 위해서 테스트 시간도 오래걸릴 뿐만 아니라 발생된 문제를 야기하는 지점을 찾기도 매우 힘들 수 있습니다. 이러한 문제를 발생시키지 않기 위해서는 비즈니스 로직을 구현하는 과정에서 생성되는 메소드들의 코드 동작에 대한 테스트를 해보면서 기능 개발을 할 수 있도록 간단한 테스트 구현이 가능한 환경을 구성해야 한다고 생각했습니다.

## # 해결방안
***

### 단위 테스트(유닛 테스트)
단위 테스트란 쉽게 말해 하나의 단위에 대한 테스트를 의미하며, 일반적으로 클래스 단위로 테스트를 진행합니다.

단위 테스트를 해야 하는 이유 다음과 같습니다.

1. **문제점 발생 지점을 명확하게 알 수 있습니다.**
: 사용자의 데이터가 잘못 들어와 에러를 발생시키는 경우, 이것에 DB의 문제인지, 단순 코드의 문제인지 등 어느 지점에서 문제가 발생한 것인지 명확하게 알 수 있습니다. 또한 TDD 방식으로 프로젝트를 진행할 경우, 자동화된 단위 테스트를 전재로 하기 때문에 프로젝트가 커질수록 디버깅 시간을 단축될 수 있습니다.

2. **테스트 시간을 단축시킬 수 있습니다.**
: 유닛 테스트는 DB나 Json 같은 외부에서 발생하는 요청을 발생시키지 않고 온전히 하나의 클래스의 코드 동작에 대해서만 테스트를 합니다. 이를 통해 테스트 시간을 매우 단축시킬 수 있습니다.

3. **독립적인 테스트 케이스를 통해 완벽한 모듈화가 가능합니다.**
: : 유닛 테스트는 기본적으로 다른 테스트와의 의존성이 절대 없어야 하고 테스트 간의 영향을 줘선 안됩니다. 그래서 테스트 케이스를 모두 통과하는 코드를 작성하다 보면 좀 더 완벽한 모듈화로 이뤄진 코드를 작성할 수 있습니다.

4. **테스트 케이스가 쌓이면서 쉽고 안전한 리팩토링이 가능합니다.**
: 독립적인 테스트 케이스를 작성하고 추가하는 과정을 통해 보다 여러 예외상황에 대해 대처할 수 있는 코드를 쉽고 빠르게 구현할 수 있습니다.

위와 같은 장점을 가진 단위테스트를 구현한다면 테스트 시간은 단축시킬 수 있을 뿐더러 여러 테스트 케이스가 쌓이고, 이를 모두 통과하는 코드를 작성한다면 안전성이 높은 코드를 작성 할 수 있습니다.

### JUnit
JUnit은 Java에서 독립된 단위테스트를 지원해주는 프레임워크입니다. JUnit을 활용한다면 보다 쉽게 단위 테스트 코드를 작성하고 관리할 수 있습니다.

#### 어노테이션

**작성한 테스트 코드**
```java

class UserServiceImplTest {

  @BeforeEach
  void init() {

  }

  @Test
  @DisplayName("이메일로 회원 정보 조회 로직 테스트")
  void findByEmailTest() {
    // given

    // when

    // then

  }
  
  @AfterEach
  void exit() {
    
  }

}
```
JUnit을 활용하면 JUnit에서 제공해주는 여러 어노테이션을 통해 위 코드처럼 간단한 테스트 코드를 작성할 수 있습니다.
- `@Test` : 테스트 케이스로 사용되는 메소드
- `@DisplayName` : 테스트 케이스를 실행할 때 명시할 테스트 케이스 설명
- `@BeforeEach` : 각각의 테스트 케이스가 실행되기 전에 동작하는 메소드
- `@AfterEach` : 각각의 테스트 케이스가 실행된 이후에 동작하는 메소드

이 외에도 여러 어노테이션을 통해 테스트 케이스 작성에 있어 필요한 기능들을 제공받을 수 있습니다.

#### Assertion
유닛 테스트를 작성할 때 중요한 건 검증에 대한 것입니다. JUnit은 Assertion 클래스를 활용해서 해당 테스트 케이스를 통해 얻고 싶은 결과에 대해 단언, 검증을 명확한 모습으로 작성할 수 있습니다.

**작성한 테스트 코드**
```java

import static org.assertj.core.api.Assertions.assertThat;

class UserServiceImplTest {

  @BeforeEach
  void init() {
  }

  @Test
  @DisplayName("이메일로 회원 정보 조회 로직 테스트")
  void findByEmailTest() {
    // given
    String email = "test@test.com";
    
    UserDto userDto = UserDto.builder()
        .email(email)
        .build();


    // when
    UserDto findUserDto = userService.findByEmail(email);

    // then
    assertThat(findUserDto.getEmail()).isEqualTo(userDto.getEmail());

  }

  @AfterEach
  void exit() {
    
  }

}
```
위 코드처럼 Asserstion 클래스의 assertThat(), isEqualTo() 등의 메소드를 통해 코드가 실행되면서 얻게되는 값과 기대값에 대한 검증을 진행할 수 있습니다. 위 코드 외에도 예외 처리에 대한 검증도 가능합니다.

#### Mockito

Mockito는 개발자가 동작을 직접 제어할 수 있는 가짜(Mock) 객체를 지원하는 테스트 프레임워크입니다. 

일반적으로 Spring으로 웹 애플리케이션을 개발하면, 여러 객체들 간의 의존성이 생기는데 이러한 의존성은 단위 테스트를 작성을 어렵게 합니다. 이를 해결하기 위해 의존성 주입 객체 대신 프록시 패턴을 통해 생성된 가짜 객체를 주입시켜주는 Mockito 라이브러리를 활용하면 가짜 객체에 원하는 결과를 Stub하여 단위 테스트를 진행할 수 있습니다.

**작성한 테스트 코드**
```java

import static org.assertj.core.api.Assertions.assertThat;

import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

class UserServiceImplTest {

  UserService userService;

  UserRepository userRepository;
  
  @BeforeEach
  void init() {
    userRepository = mock(MybatisUserRepository.class); // 목 객체 생성
    
    userService = new UserServiceImpl(userRepository); // 목 객체 주입
  }

  @Test
  @DisplayName("이메일로 회원 정보 조회 로직 테스트")
  void findByEmailTest() {
    // given
    UserDto userDto = UserDto.builder()
        .email(email)
        .build();


    // when
    when(userRepository.findByEmail(email)).thenReturn(userDto); // 기대 결과값을 개발자가 임의로 지정

    UserDto findUserDto = userService.findByEmail(email); // 로직 수행 후 얻는 결과값 저장

    // then
    verify(userRepository, times(1)).findByEmail(email); // 목 객체의 메소드 수행 횟수 검증

    assertThat(findUserDto.getEmail()).isEqualTo(userDto.getEmail()); // 임의로 지정된 결과값이 나오는지 검증

  }

  @AfterEach
  void exit() {
    
  }

}
```
위 코드처럼 when()을 통해 실제로 동작하지 않는 목 객체의 특정 메소드에 대한 결과값을 임의로 지정하고 로직을 수행했을 때, 의도한 대로 원하는 결과값을 도출하는 메소드가 수행하는지 검증할 수 있습니다.

Mockito를 복잡할 수 있는 의존성을 간소화시켜 테스트 실행 속도를 향상시킬 수 있습니다.

## # 마치며
***
이처럼 JUnit을 통해 단위 테스트 코드를 작성해가면서 단위 테스트 작성 방법과 중요성을 직접 느낄 수 있었습니다.


## # 참고자료
***
- http://www.yes24.com/Product/Goods/75189146
- https://beststar-1.tistory.com/27
- https://mangkyu.tistory.com/145
- https://tecoble.techcourse.co.kr/post/2020-10-16-is-ok-mockito/