---
layout: post
title: "super 예약어는 부모 객체를 가르킨다"
author: kimcno3
categories: java
tags: java
---

`super` 예약어는 부모 객체를 가르킨다고 하는데 JVM의 메모리 영역에서 상속 클래스가 할당되는 과정을 보면서 알아가 보자.

![](https://media.vlpt.us/images/cham/post/5ff85e45-6b48-491c-bc07-8fa4e0838bac/image.png)<br>
출처 : [https://velog.io/@cham/백기선님자바스터디-6주차-상속](https://velog.io/@cham/%EB%B0%B1%EA%B8%B0%EC%84%A0%EB%8B%98%EC%9E%90%EB%B0%94%EC%8A%A4%ED%84%B0%EB%94%94-6%EC%A3%BC%EC%B0%A8-%EC%83%81%EC%86%8D)

위 그림을 보면 `자식클래스 객체(Iphone)`가 생성되면서 이에 관련된 데이터는 `Heap 영역`에 저장되고 `참조변수(iphone)`는 `Stack 영역`에 저장된다.

그런데 `자식클래스 객체`의 생성자에는 `super()` 라는 코드가 자동으로 추가된다고 했다. 이 `super()`은 부모 클래스의 생성자를 의미하며 자식 클래스의 객체가 생성되는 과정에서 `부모 클래스의 객체(CellPhone)`도 `Heap 영역`에 생성된다고 보면 된다.

<br>

![](https://t1.daumcdn.net/cfile/tistory/156CAE444F5EF9D101)<br>
출처 : https://stellan.tistory.com/entry/Java-%EC%83%81%EC%86%8D


두 객체의 연결은 `super`라는 이름으로 선언되는 참조변수에 의해 연결된다. `super`는 자식 객체에 선언되며 이 변수에 담긴 값은 부모 객체의 메모리 주소이다.

그러므로 자식 클래스 내에서 `super.변수명` 또는 `super.메소드명`으로 코드를 작성하면, 부모 객체에 선언되어 있는 변수명과 메소드명을 호출할 수 있다.

<br>

### **예제 코드**
```java
public class PrintInfo{
    public static void main(String[] args){
        Child child = new Child(); // 자식 클래스 생성

        child.printParentAge(); // 부모클래스의 변수를 출력
        child.printChildAge(); // 자식클래스의 변수를 출력
    }
}
// 부모 클래스
class Parent{
    String name = "Kim";
    int age = 56;
    public Parent(){
    }
}
// 자식 클래스
class Child extends Parent{
    String name = "Lee";
    int age = 27;
    public Child(){
        // super(); 부모 객체 생성
    }
    // 자식클래스에서 부모클래스 변수 지정(super == 부모 객체)
    public void printParentAge(){
        System.out.println(super.name + "의 나이는" + super.age + "입니다.");
    }
    // 자식클래스에서 자식클래스 변수 지정(this)
    public void printChildAge(){
        System.out.println(this.name + "의 나이는" + this.age + "입니다.");
    }
}
```
### **실행결과**
```
Kim의 나이는56입니다.
Lee의 나이는27입니다.
```

위 예시에서 보면 자식 클래스에서 `super.변수명`의 형식으로 코드를 작성하면 **부모클래스의 인스턴스변수를 지정**할 수 있다.

반대로 `this.변수명`은 **해당 클래스의 인스턴스 변수를 지정**하게 된다.

> [참고사이트](https://crazykim2.tistory.com/551)

> [참고사이트](https://velog.io/@cham/%EB%B0%B1%EA%B8%B0%EC%84%A0%EB%8B%98%EC%9E%90%EB%B0%94%EC%8A%A4%ED%84%B0%EB%94%94-6%EC%A3%BC%EC%B0%A8-%EC%83%81%EC%86%8D)

> [참고사이트](https://stellan.tistory.com/entry/Java-%EC%83%81%EC%86%8D)