---
layout: post
title: "Reflection API"
author: kimcno3
categories: java
tags: java
---

## Reflection API(리플렉션)이란?
Reflection은 자바에서 기본으로 제공해주는 API로 클래스, 메소드 인터페이스 등의 정보를 클래스를 생성하지 않고 확인할 수 있습니다.

Reflection에서 가져올 수 있는 클래스 종류는 대표적으로 아래 4개의 클래스로 분류할 수 있다.
- `Class` : 클래스 정보를 담은 클래스
- `Constructor` : 생성자 정보를 담은 클래스
- `Method` : 메소드 정보를 담은 클래스
- `Field` : 상수 정보를 담은 클래스

Class 클래스는 `java.lang.Class`로 선언되어 있기 때문에 따로 import할 필요 없이 모든 클래스에서 Class 클래스를 호출할 수 있다.

하지만 다른 세개의 클래스(`Constructor`, `Method`, `Field`)는 `java.lang.reflect` 패키지에 선언되어 있기 때문에, 해당 클래스 객체를 생성해야 할 때 따로 import해야만 해당 클래스를 호출하여 사용할 수 있다.

#### **Dog 클래스**
```java
public class Dog {
	// Fields
	String name = "coco";
	int age = 5;

	// Constructors
	public Dog() {

	}
	public Dog(String name, int age) {
		this.name = name;
		this.age = age;
	}
	// Methods
	public void move(String name) {
		System.out.println(name + "is moving");
	}
	public void bark() {
		System.out.println("mungmmung");
	}
}
```
구체적인 설명을 위해 Dog 클래스를 생성하고 Dog 클래스의 정보를 Reflection을 통해 가져와 보면서 알아보자

<br>

### ✔️ Class 클래스
#### **예제코드**
```java
Class classOfDog = Dog.class; // Dog 클래스 정보를 담은 Class 객체 생성
System.out.println(classOfDog.getName()); //생성한 Dog 타입 객체의 클래스 이름을 출력
```

#### **출력결과**
```java
패키지명.Dog
```
위 코드처럼 `클래스명.class`으로 코드를 작성하면 해당 클래스에 대한 정보들을 하나의 객체로 선언할 수 있다.

그 다음 생성한 객체(위 코드에서는 `classOfDog`)에 다양한 메소드를 사용하여 클래스, 메소드, 필드명을 필요에 따라 가져올 수 있다. (`getName()`, `getDeclaredMethods` 등)

<br>

### ✔️ Constructor, Method, Field 클래스
생성자, 메소드, 필드 정보는 위에서 생성한 클래스 타입 객체에서 Class 클래스에 선언된 메소드를 사용해서 정보를 가져올 수 있다.

#### **예제코드 - Constuctor 클래스**
```java
// 기본 생성자 정보
Constructor constructorOfDog = classOfDog.getDeclaredConstructor();
// 매개변수가 있는 생성자 정보
constructorOfDog2 = classOfDog.getDeclaredConstructor(String.class, int.class);
// 모든 생성자 정보
Constructor[] constructorsOfDog = classOfDog.getDeclaredConstructors();

System.out.println(constructorOfDog);
System.out.println(constructorOfDog2);
System.out.println(constructorsOfDog.length);
```
#### **출력 결과**
```
public 패키지명.Dog()
public 패키지명.Dog(java.lang.String,int)
2 // 선언된 생성자의 개수
```
`getDeclaredConstructor()` 메소드는 Dog클래스 정보 중 생성자 하나에 대한 정보를 보여준다.

기본 생성자는 매개변수가 없이 메소드를 선언하면 되지만, 매개변수가 지정되어 있는 생성자의 경우에는 매개변수의 클래스를 지정해줘야 한다.

만약 선언되어 있는 모든 생성자를 한번에 가져와야 할 때에는 `getDeclaredConstructors()` 메소드를 통해 Constructor 타입의 배열 형태로 생성자 정보를 가져올 수 있다.

<br>

#### **예제코드 - Method 클래스**
```java
// 메소드 정보(매소드명, 매개변수 타입)
Method methodOfDog = classOfDog.getDeclaredMethod("move", String.class);
// 모든 메소드 정보
Method[] methodsOfDog = classOfDog.getDeclaredMethods();

System.out.println(methodOfDog);
System.out.println(methodsOfDog.length);
```
#### **출력 결과**
```
public void 패키지명.Dog.move(java.lang.String)
2 // 선언된 메소드 개수
```
`getDeclaredMethod()` 클래스 객체에서 원하는 메소드 정보를 가져올 수 있는 기능을 수행하는데 매개변수로 `메소드명`과 `매개변수의 클래스`를 지정해야 한다.

만약 매개변수가 없는 메소드라면 매개변수 클래스를 null로 지정하면 된다.

모든 메소드 정보를 가져오고 싶을 때는 `getDeclaredMethods()` 메소드를 통해 배열 형태로 가져올 수 있다.

<br>

#### **예제코드 - Field 클래스**
```java
// 변수 정보(변수명)
Field fieldOfDog = classOfDog.getDeclaredField("name");
// 모든 변수 정보
Field[] fieldsOfDog = classOfDog.getDeclaredFields();

System.out.println(fieldOfDog);
System.out.println(fieldsOfDog.length);
```
#### **출력 결과**
```
java.lang.String 패키지명.Dog.name
2 // 선언된 변수 개수
```
`getDeclaredField()` 메소드도 객체의 클래스에서 원하는 변수 정보를 가져오는 기능을 하는데 매개변수로 `변수명`을 지정해주면 된다.

모든 변수를 가져오고 싶을 때는 생성자나 메소드 정보를 가져올 때와 마찬가지로 `getDeclaredFields()` 메소드를 통해 배열 형태로 가져오는 것이 가능하다.

> [참고 사이트](https://codechacha.com/ko/reflection/)
