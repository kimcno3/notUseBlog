---
layout: post
title: "equals() 와 hashcode()"
author: kimcno3
categories: java
tags: java
---

## equals() 오버라이딩
***
`equals()`는 객체간의 동일함을 비교하기 위해 사용되는 Object 클래스의 메소드이다. 하지만 `equals()`를 선언된 그대로 사용한다면 객체간의 값이 아닌 **참조변수에 담긴 메모리 주소를 비교**하여 논리적 동일함을 구분해 낼 수 없다. 

그래서 해당 메소드를 사용할 경우 오버라이딩하여 새롭게 정의한 다음 활용하는게 일반적인데, 대부분의 IDE에서 제공하는 `equals()` 오버라이딩 코드는 다음과 같다.

```java
@Override
public boolean equals(Object obj){
    if(this == obj) return true;
    if(obj == null) return false;
    if(this.getClass() != obj.getClass()) return false;
    Student other = (Student) obj; // ??? 부모객체 생성자로 자식객체를 생성한다?? 에러가 왜 안나지???

    if(name == null){
        if(other.name != null) return false;
    } else if(!name.equals(other.name)) return false;

    if(address == null){
        if(other.address != null) return false;
    } else if(!address.equals(other.address)) return false;

    if(phone == null){
        if(other.phone != null) return false;
    } else if(!phone.equals(other.phone)) return false;

    if(email == null){
        if(other.email != null) return false;
    } else if(!email.equals(other.email)) return false;

    return true;
}
```

주의할 부분은 주석이 달린 코드다. 해당 코드를 보면 부모객체인 Object의 생성자를 자식객체로 형변환하여 자식객체를 생성한다. 이러면 에러가 발생할텐데(참조자료형 형변환시 주의점) 에러발생이 안한다.

차근차근 코드를 살펴보니 그 이유를 알았다.

먼저 `equals()` 에서 매개변수로 받는 객체는 어떤 클래스의 생성자로 만들어진 객체인지 모르기 때문에 가장 최상위 객체인 Object 타입으로 받아야만 한다. 이렇게 되면 어떠한 객체 타입이 매개변수로 오더라도 Object보단 하위 클래스이기 때문에 문제가 생길 일이 없다.

하지만 `equals()` 메소드가 사용되는 경우는 사실상 받아온 객체가 Object 객체의 하위객체, 즉 상속된 또 다른 클래스이거나 같은 클래스의 객체이다. 그리고 형변환 전에 `getClass()`를 활용해서 두 객체 타입이 다르다면 false를 리턴하고 메소드를 종료하게 설계되어 있다.

그렇기 때문에 형변환 코드가 실행되는 경우는 **this와 obj 객체가 같은 클래스일 경우**에만 실행된다. 기본 예외 발생 상황에 대한 대비가 되어 있게 기본 오버라이딩 코드는 설계되어 있었다.

<br>

## hashcode() 오버라이딩
***
`hashCode()` 메소드도 Java에서 Object 객체에 선언된 메소드 중 하나로 객체의 **주소값을 int 타입의 16진수로 리턴**하는 메소드이다.

`hashCode()` 메소드는 `equals()`와 함께 오버라이딩하여 사용하는 것이 일반적이다. 그 이유는 **해시코드 규약**을 보면 이해할 수 있다.

### **해시코드 규약**
- `equals()`의 리턴값이 true일 경우, 두 객체의 `hashCode()` 리턴값도 같아야 한다.
- `equals()`의 리턴값이 false일 경우, 두 객체의 `hashCode()` 리턴값이 무조건 달라야 하는 것은 아니지만, 두 리턴값이 다르다면 hashTable의 성능을 향상 시킬 수 있다.

위 해시코드 규약을 참고했을 때, `eqauls()`가 `true`라면 `hashCode()` 또한 같은 값으로 리턴되어야 한다.

하지만 `equals()`만 오버라이딩한 경우에는 두 객체가 논리적으로 동일한 객체임을 의미하더라도, `hashCode()`는 아직 두 객체의 물리적인 메모리 주소를 비교해 다른 결과값을 리턴하게 된다. 그렇기 때문에 해시코드 규약을 지키기 위해선 `hashCode()`의 오버라이딩도 필요하다.

보편적으로 사용되는 hashCode()의 오버라이딩 코드는 다음과 같다.

#### **hashcode() 오버라이딩 코드**
```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + ((variableValue1 == null) ? 0 : variableValue1.hashCode());
    result = prime * result + ((variableValue2 == null) ? 0 : variableValue2.hashCode());
    result = prime * result + ((variableValue3 == null) ? 0 : variableValue3.hashCode());
    return result;
}
```
**코드 구동 순서 설명**
1. hashcode로 전환할 변수의 값을 `variableValue1 ~ 3`으로 정한다.
2. variableValue1의 값이 null 이라면 result에 0을 더한다.
3. 그렇지 않다면 variableValue1의 hashCode()값을 result에 더한다.
4. 모든 변수가 2~3번 과정을 반복한다.
5. 최종 result 값을 리턴한다.

위 코드가 호출되는 상황은 **이미 `eqauls()`를 통해 비교할 두 객체의 값이 동일하다는 것을 확인한 이후**이다. 그래서 두 객체에 선언된 변수나 메소드의 hashCode값들의 총합을 result로써 리턴하여 두 합계를 같게 만들어 주는 것이다. 그래야 두 객체의 `hashcode()` 결과값이 동일하게 리턴된다.

이렇게 `equals()` 와 `hashCode()` 를 함께 오버라이딩을 해야만 두 객체의 **논리적 동일성**을 나타낼 수 있게 된다.

<br>

그런데 자세히 보면 결과값에 31을 곱하는 것을 볼 수 있다.

**31을 사용하는 이유**는 Prime 변수에 31을 할당하고 result에 이 값을 곱하여 합계를 구한다. 그냥 null값이 아니면 해당 hashcode값만 더해서 result를 구해도 무방할 것처럼 보이는데 굳이 31이라는 값을 곱하는 이유는 무엇일까?

이는 result값을 구할 때 생겨날 에러를 방지하기 위해 상수로 곱해주는 것인데 자바의 hashCode()를 오버라이딩할 때는 주로 31을 사용할 뿐이다. 31이 에러를 방지할 수 있는 이유는 31이 **홀수**이자 **소수**이기 때문이다.

<br>

만약 31과 같은 홀수가 아닌 짝수를 곱한다면 비트를 왼쪽으로 Shift하는 것과 같기 때문에 에러를 발생할 수 있다.

가장 오른쪽 2진수를 보면 2를 곱했을경우 1의 위치가 왼쪽으로 한칸씩 이동한 것을 볼 수 있다. 이처럼 짝수를 곱할 경우 비트가 이동함과 동시에 오버플로우가 발생하면 값이 사라질 수 있기 때문에 짝수를 선택할 수 없다.

<br>

그렇다면 홀수를 사용하는 것은 이해했는데 왜 굳이 소수인 것일까??

소수를 사용하는 이점은 따로 알려진 것이 없지만 통상적으로 그렇게 사용된다고 한다. 약간 미신과 같은 이유라고 생각하면 된다.

소수를 사용하지 않아도 hashCode값 생성에는 큰 문제가 생기지 않지만, 짝수를 피해야하는 것은 명백한 이유가 존재한다는 것을 기억하면 좋을 듯 하다.

> [참고사이트](https://johngrib.github.io/wiki/Object-hashCode/#%EC%9D%B4%EC%83%81%EC%A0%81%EC%9D%B8-%ED%95%B4%EC%8B%9C-%ED%95%A8%EC%88%98%EC%97%90-%EA%B0%80%EA%B9%8C%EC%9A%B4-%ED%95%A8%EC%88%98-%EB%A7%8C%EB%93%A4%EA%B8%B0)

> [참고사이트](https://devlog-wjdrbs96.tistory.com/243)

