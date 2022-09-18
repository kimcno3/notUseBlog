---
layout: post
title: "변수 종류 별 쓰레드 동기화 필요성"
author: kimcno3
categories: java
tags: java
---

JVM 내에서 하나의 쓰레드가 생성되면 각각의 쓰레드는 고유의 Stack, Pc register, Native Method Area를 가진다. 대신 Heap 과 Method 영역은 하나의 공간을 공유하는 구조로 설계되어 있다.

이에 따라 다중 쓰레드에 접근 가능한 변수가 있고 그렇지 않는 변수가 존재해서 이를 제어해줄 필요성이 나뉘어진다.

### `지역변수`
기본자료형은 쓰레드 내 스택에 저장되어 절대 다른 쓰레드가 접근 불가, 참조 자료형 또한 힙 영역에 저장될 지라도 이에 접근하는 메소드가 다른 쓰레드에 존재할 수 없다.

### `매개변수`
지역변수처럼 하나의 매개변수를 가지는 메소드는 해당 쓰레드 내에서만 존재한다.

### `인스턴스 변수`
객체에 대한 정보가 heap 영역에 저장되고 이를 디수의 쓰레드에서도 접근이 가능하게 설계할 수 있다.

#### **예시 코드**
```java
public class MainThread {
    Integer age = new Integer(27);
    public static void main(String[] args){
        Thread thread1 = new Thread(obj);
        Thread thread2 = new Threade(obj);

    }
}
```
위 코드를 보면 두개의 다른 쓰레드가 하나의 멤버 변수를 매개변수로 받아 생성되는 것을 볼 수 있다. 이렇게 받아간 맴버변수를 각각의 쓰레드가 접근하여 값을 바꾼다면 원치않는 값이 나올 수 있다.

### `클래스 변수`
대부분 상수로 활용되는 값들이기 때문에 `enum` 클래스 혹은 `final` 변수로 저장하여 변경이 불가능하게 선언하는 것이 일반적이다. 그러므로 여러 쓰레드가 동시에 접근한다 하더라도 값이 변경될 가능성이 없어서 크게 상관이 없다.

> [참고 사이트(변수별 쓰레드 동기화 필요성)](https://parkcheolu.tistory.com/12)
> 
> [참고 사이트(main 쓰레드)](https://wisdom-and-record.tistory.com/48)
