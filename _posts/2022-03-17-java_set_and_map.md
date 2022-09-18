---
layout: post
title: "Set & Map"
author: kimcno3
categories: java
tags: java
---

## HashSet
HashSet은 Set 구조를 구현한 클래스 중 하나로 중복과 순서가 없는 자료구조이다.

근데 HashSet의 내부 코드를 보면 **내부는 HashMap**으로 구현되어 있다.

### HashSet 구현 코드
```java
// PRESENT 상수
private static final Object PRESENT = new Object();

// 생성자
public HashSet() {
    map = new HashMap<>();
}

// add()     
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

// remove() 
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

```
자세히 말하면 HashMap의 키가 HashSet에선 요소에 해당하며 모든 HashMap의 value들은 PRESENT라는 이름의 객체로 채워져 있다. 

결론적으로 HashSet은 HashMap으로 구현된 클래스이며 Map의 성질을 활용하여 구현한 것이라고 볼 수 있다.

<br>

## HashMap
그렇다면 HashMap은 어떻게 구현되어 있을까?
> 상세한 내부 코드 설명은 블로그 링크로 대체합니다.(https://bepoz-study-diary.tistory.com/328?category=833599)

위 블로그 글의 내용을 간략하게 말하면

- 만약 키가 객체라면, 객체를 hashCode()를 통해 int값으로 바꿔서 활용하는데 이 hashcode값은 다른 객체들도 같은 값으로 치환될 수 있다.
- 그렇다면 같은 hashCode를 가지게 된 다른 키들은 하나의 버킷을 할당받게 된다.(해시 충돌)
- 그래서 HashMap의 키에 대한 버킷은 LinkedList로 되어 있다.
- 이미 값이 존재하는 키에 또 다른 값이 저장되어야 하는 경우, LinkedList로 오는 순서에 맞게 값을 저장한다.
- 이러한 구조에 맞게 add(), remove()등 요소를 처리하는 메소드들이 구현되어 있다.

이러한 구조로 인해 HashMap에서 Object 클래스의 메소드인 equals()와 hashCode()가 필수적으로 사용되며 이 둘의 오버라이딩은 필수적이라고 할 수 있다.

### equals(), hashCode() 오버라이딩이 필수적인 이유
> `equals()`와 `hasCode()` 오버라이딩 관련 설명은 다음 [링크](https://github.com/kimcno3/TIL/blob/main/programming_language/java/java_equals_and_hashcode.md)를 참고하세요.

첫번째로 key를 객체로 이용할 경우, 논리적 동일함이 보장되지 않는다면 같아야할 key가 다르다고 인식되어 수정이 아닌 원치 않는 값의 추가가 될 수 있다.

그래서 두 메소드를 오버라이딩하여 논리적 동일함을 보장해야 의도에 맞는 key를 설정할 수 있다. 여기서 말한 논리적 동일함을 보장하는 방법이 자바에선 `equals()`와 `hashCode()`를 오버라이딩하는 것을 의미한다.

<br>

두번째 이유는 두 메소드의 오버라이딩은 해시충돌을 해결할 때에도 두 메소드의 재정의는 필수적이다.

해시 충돌이란 두개의 다른 객체를 키로 가져올 때, 두 객체가 다른 값을 가지고 있지만 계산된 해시값이 같아 같은 버캇을 가르키는 경우를 의미한다.

이런 경우에는 하나의 버킷이 두개의 값이 저장되어야 하는데 이를 해결하는 방법론은 큰 범주에서 두가지다.

1. `Opening Addrress` : 해당 버킷에 이미 데이터가 존재한다면, 메모리 내 다른 버킷에 데이터를 저장해두는 방법이다.

2. `Separate Chaining` : 추가적인 메모리를 할당해 해당 버킷과 `LinkedList` 타입으로 연결하여 저장한다.

자바에선 2번 Chaining 방법으로 해싱 충돌을 해결한다.

이렇게 같은 해시에 연결된 데이터를 다시 불러올 경우(`get()`), 키를 구별하기 위해 내부적으로 `equals()` 메소드가 사용된다. 이때 `equals()`가 오버라이딩 되어있다면 정확히 동일한 키를 구별하고 그에 대응하는 값을 가져올 수 있을 것이다.

<br>

> [참고사이트 : HashMap() 내부 코드](https://bepoz-study-diary.tistory.com/328?category=833599)

> [참고사이트_Custom Key](http://www.gisdeveloper.co.kr/?p=5332)

> [참고사이트_equals()와 hashCode()](https://nesoy.github.io/articles/2018-06/Java-equals-hashcode)

> [참고사이트_해시 충돌](https://m.blog.naver.com/weplayicecream/221467971945)