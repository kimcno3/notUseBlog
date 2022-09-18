---
layout: post
title: "BigDecimal 클래스를 사용하는 이유"
author: kimcno3
categories: java
tags: java
---

JAVA에서 기본자료형으로 활용되는 float나 double을 사용하지 않고 BigDecimal 클래스를 사용하는 이유는 BigDecimal 클래스를 활용하면 실수를 계산하는데 더 정확한 계산이 가능하다 한다.

이를 자세히 알아보기 위해 먼저 기본 자료형으로 실수 계산시 발생할 수 있는 문제점을 보자.

```java
double num1 = 3.1254;
double num2 = 2.2376;
System.out.println(num1 + num2); // 5.3629999999999995
```
위 예제에서 두 double 타입의 값을 더했을 때 정확히 기대값이 아닌 기대값의 매우 가까운 근사치가 나오는 것을 볼 수 있다.(2진법 사용)

매우 작은 차이이기에 가볍게 넘길 수 있다고 생각할 수 있지만 돈계산과 같은 오차가 허용되지 않는 상황일 땐 매우 치명적인 오류를 범할 수 있다.

다음은 BigDecimal 타입으로 같은 값을 계산했을 경우다.(10진법 사용)
```java
BigDecimal num1 = BigDecimal.valueOf(3.1254);
BigDecimal num2 = BigDecimal.valueOf(2.2376);
System.out.println(num1.add(num2)); // 5.3630
```

`valueOf()` 메소드는 인자로 받은 값을 원하는 클래스로 변경하는 역할을 수행한다.

즉, double타입의 변수를 BigDecimal 타입으로 형변환을 한 이후 두 값을 더했을 때 우리가 기대했던 값이 나온 것을 알 수 있다.

이처럼 자바에서 기본자료형으로 실수를 연산하기보단 BigDecimal 타입을 활용하는 것이 필수적이다.
