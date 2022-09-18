---
layout: post
title: "참조자료형 형변환"
author: kimcno3
categories: java
tags: java
---

### ✔️ 참조자료형 형변환 특징
- 자식 객체를 부모 객체로 형변환은 가능
- 부모 객체를 자식 객체로 형변환하는 것은 불가능

**예제 코드**

아래 예제에 example() 메소드에는 세가지 경우로 생성한 객체가 존재한다.

```java
class Sample{
	public static void main(String[] args){
		Sample sample = new Sample();
        sample.example();
    }
    // 예제 실행 메소드
    public void example(){
        // 1. 부모타입 객체 - 부모 생성자
        Parent parent1 = new Parent();
        parent1.printByParent(); // 결과 : This str is from Parent

        // 2. 부모타입 객체 - 자식 생성자
        Parent parent2 = new Child();
        parent2.printByParent(); // 결과 : This str is from Parent with Overriding method

        // 3. 자식타입 객체 - 부모생성자(처럼 보이는 자식 생성자)
        Child child1 = (Child)parent2;
        child1.printByChild(); // 결과 : This str is from Child with Only child method
	}
}
// 부모 클래스
class Parent{
    // 변수
    String str = "Parent";

    // 메소드
    public void printByParent(){
        System.out.println("This str is from " + str);
    }
}
// 자식 클래스
class Child extends Parent{
    // 자식 클래스에 새로 선언한 메소드
    String str2 = "Child";

    // 오버라이딩 메소드
    public void printByParent(){
        System.out.println("This str is from " + str + " with Overrinding method");
    }
    // 자식 클래스에 새로 선언한 메소드
    public void printByChild(){
        System.out.println("This str is from " + str2 + " with Only child method");
    }
}
```
### ✔️ 부모 클래스 생성자로 부모타입의 객체를 생성한 경우
이런 경우는 자식 클래스에 어떠한 영향을 받지 않는다.

<br>

### ✔️ 자식 클래스 생성자로 부모타입 객체를 생선한 경우
이런 경우에 자식 클래스에서 부모 클래스로부터 상속받은 변수나 메소드만 호출할 수 있다.

즉, 자식 클래스에만 선언된 `str2` 변수나 `printByChild()` 메소드는 호출이 불가능하다.

```java
Parent parent2 = new Child();
parent2.printByChild(); // 에러 발생(결국 부모타입이라)
```

또한 오버라이딩되어 자식 클래스에 재선언된 메소드나 변수는 오버라이딩된 상태로 호출된다.
> 이런 현상을 책에서는 폴리몰피즘(다형성)이라고 한다.

<br>

### ✔️ 자식타입 객체 - 부모생성자(처럼 보이는 자식 생성자)
자식 클래스 타입의 객체를 생성하는데 **그 생성자를 자식생성자로 만든 부모타입의 객체로 만든 경우**이다.

말이 많이 어렵지만 사실상 자식클래스 객체이지만 부모클래스 타입으로 보이는 객체로 이해하면 되겠다.

이 경우에 결국 실생성자의 출신이 어디냐를 본다면 **자식클래스**이지만 겉모습은 부모클래스를 타입으로 가지고 있기때문에 형변환을 통해 객체를 생성해줘야 한다.

```java
Child child2 = parent2; // 에러발생(겉보기는 부모타입)
```

하지만 형변환을 통해 객체를 생성했다면 자식클래스의 변수와 메소드를 호출한다.

### **정리하자면**
- 결국 생성자에 따라 호출되는 변수나 메소드가 결정된다.
- 그렇기에 오버라이딩된 변수나 메소드는 그 상태로 호출된다.
- 자식 생성자로 만든 부모 객체는 겉으로는 부모의 형상을 띄기 때문에 이 객체를 이용해 자식 객체를 만들려면 형변환이 필요하다.
- **겉** : 부모클래스 타입의 객체 / **안** : 오버라이딩되었거나 부모객체로부터 상속된 변수와 메소드만 호출 가능

<br>

## :pushpin: `instanceof` 예약어 추가 설명
참조자료형의 형변환을 하기 전에 형변환 가능 여부를 미리 파악하기 위해 사용되는 예약어로 boolean 값을 리턴한다.

- 자식객체 `instanceof` 부모클래스 : 이 객체가 부모클래스를 대신할 수 있는가? -> `return true`
- 부모객체 `instanceof` 자식클래스 : 이 객체가 자식클래스를 대신할 수 있는가? -> `return false`
