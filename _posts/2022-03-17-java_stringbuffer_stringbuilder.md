---
layout: post
title: "String, StringBuffer, StringBuilder 클래스"
author: kimcno3
categories: java
tags: java
---

## String
***
String은 불변한 성격(immutable)을 가지고 있습니다. 즉 한번 생성한 문자열은 수정이 불가능합니다.

이러한 특성때문에 이미 생성된 String 객체에 문자열을 수정할 경우, 같은 메모리에 저장된 값만 수정되는 것이 아니라 통째로 새로운 메모리 주소에 저장됩니다.

```java
String str = "안녕하세요."; // 문자열 생성
str = str + " 반갑습니다."; // 문자열 수정(?)
System.out.println(str); // output : 안녕하세요. 반갑습니다.
```

위 예제에서 출력 결과는 두 문자열이 합쳐진 결과로 나오지만, 문자열에 연산을 통해 새로운 값을 추가하면 새로운 객체로 다시 생성된다고 생각하면 됩니다. 그리고 이전의 문자열이 저장되어 있는 메모리는 GC(가비지 콜렉터)의 대상이 됩니다.

그래서 String 클래스는 문자열에 대한 수정 작업이 일어나지 않는 경우에 사용하는 것이 효율적입니다.

<br>

## StringBuffer, StringBuilder
***
StringBuffer와 StringBuilder 클래스는 String과 다르게 mutable한 성격을 가진 클래스입니다. 즉, **문자열의 수정이 가능한 클래스**입니다.

그래서 두 클래스는 문자열에 대한 수정이 빈번한 경우에 사용하는 것이 좋고 연산이 아닌 `append()`, `delete()`와 같은 메소드를 사용하여 수정할 수 있습니다.

```java
StringBuffer str = new StringBuffer("안녕하세요."); // 문자열 생성
str.append(" 반갑습니다."); // 문자열 수정
System.out.println(str); // output : 안녕하세요. 반갑습니다.
```

<br>

## StringBuffer와 StringBuilder 차이점
***
StringBuffer와 StringBuilder의 차이점은 **쓰레드 동기화 지원** 유무이며 StringBuffer가 쓰레드 동기화를 지원하는 클래스입니다.

> **용어 설명**
> - **쓰레드** : 프로세스의 실행 단위
> - **쓰레드 동기화** : 하나의 프로세스에 여러 쓰레드가 접근해야 하는 경우, 접근 순서에 대한 질서, 규칙을 정하는 것

> 즉 **쓰레드 동기화가 된다**는 것은 멀티 쓰레드 환경에서 수행이 가능하다는 뜻

그래서 동기화를 지원해주는 클래스인 StringBuffer가 하나의 변수에 여러 쓰레드가 동시에 접근할 경우가 있는 경우에 높은 안정성을 가집니다.

그렇지 않은 경우에는 StringBuilder가 쓰레드 동기화에 대한 고려하지 않아도 되기에 StringBuffer보다 성능이 뛰어납니다.

그리고 String 클래스 또한 문자열에 대한 수정이 불가능하기 때문에 멀티 쓰레드 환경에서 안정성을 가집니다.

> [참고 사이트 1](https://ifuwanna.tistory.com/221)

> [참고 사이트 2](https://popcorntree.tistory.com/65)
