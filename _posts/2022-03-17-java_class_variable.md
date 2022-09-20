---
layout: post
title: "클래스 변수와 인스턴스 변수의 차이"
author: kimcno3
categories: java
tags: java
---

### **예제 코드**
```java
public class ReferenceStaticVariable{
    static String name; // 클래스변수 name
    public ReferenceStaticVariable(){}
    public ReferenceStaticVariable(String name){
        this.name = name; // 매개변수를 클래스변수에 할당해라! 라는 의미
    }
    public static void main(String[] args){
        ReferenceStaticVariable sample = new ReferenceStaticVariable();
        sample.checkName();
    }
    public void checkName(){
        ReferenceStaticVariable reference1 = new ReferenceStaticVariable("Kim");
        System.out.println(reference1.name); // Kim
        ReferenceStaticVariable reference2 = new ReferenceStaticVariable("Lee");
        System.out.println(reference1.name); // Lee
    }
}
```

위 예제를 보면 같은 `reference1`객체의 `name`값을 출력했는데 출력값은 다른 것을 확인할 수 있다.(객체의 변수를 출력한 것처럼 보인다.)

이는 매개변수값을 할당하는 변수 `name`이 클래스변수, 즉 **static**변수이기 때문에 위와 같은 결과값을 가지게 된건데, 자세히 설명하면 클래스 변수는 객체에 상관없이 하나의 메모리 주소만을 가지게 된다.

즉, JVM의 관점에서 보면 클래스 변수는 클래스 파일이 로딩되는 과정 내에서 `Method 영역`에 저장되고ㄴ 프로그램이 종료될 때까지 메모리에 할당되어 유지된다.

그렇기에 위 예제는 `reference1`만의 **인스턴스 변수**인 `name`에 매개변수값을 할당한 것이 아니라, 클래스 전체에 하나만 존재하는 **클래스변수**인 `name`에 매개변수 값을 할당한 것이다.(객체와는 무관하게 name이라는 변수는 해당 클래스에서 하나라는 뜻!)

그러므로 `name`은 `Kim`으로 한번, `Lee`로 한번씩 값을 할당받은 것이다.

> [참고 사이트](https://wikidocs.net/228)
