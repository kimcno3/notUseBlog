---
layout: post
title: "Collection Framework"
author: kimcno3
categories: java
tags: java
---

### Collection Framework란?
`Collection FrameWork`는 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미합니다.

`Collection Framework`의 구성요소는 자료구조에 따라 인터페이스로 구분하고, 이를 구체적인 클래스들로 구현되어 있습니다.

핵심 인터페이스는 다음과 같습니다.
- `List` : 순서가 있는 데이터 집합, 중복 허용
- `Set` : 순서가 없는 데이터 집합, 중복 혀용하지 않음
- `Map` : 키와 값으로 구분된 데이터 집합

이 중에서 `List`와 `Set`인터페이스는 같은 `Collection` 인터페이스를 상속받지만, `Map` 인터페이스는 자료 구조상 차이점으로 인해 별도로 정의되어 있습니다. 

### Collection Framework 상속 관계
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDhAEy%2FbtradRzasBQ%2FCjKa3OnW5k8tGYrkrqjVJ1%2Fimg.png) <br>
> 이미지 출처는 하단에 추가했습니다.

위 인터페이스들을 구현한 클래스는 다양하며 클래스마다 차이점이 존재합니다.

배열이 아닌 `Collection Framework`를 활용하는 이유는 배열의 크기를 정해두지 않고 활용하기 위해서(= 동적 메모리 할당)입니다.

### Iterable 인터페이스
Iterable 인터페이스는 `iterator()` 메소드가 정의되어 있는 인터페이스이며 Collection 인터페이스가 상속하고 있기 때문에 `iterator()`는 어떤 자료구조 타입의 객체에서도 사용가능한 메소드이다.

`iterator()`를 특정 자료구조에 사용하면 해당 자료구조를 통째로 `Iterator` 타입의 객체로 리턴하는데 `Iterator` 인터페이스에는 세가지 메소드가 구현되어 있다.

- `hasNext()` : 다음 위치에 값이 존재하는지 여부를 판단하여 boolean값을 리턴
- `next()` : 다음 위치의 값을 리턴
- `remove()` : 특정 값을 제거

`Iterator`는 위 세가지 메소드를 통해 어떠한 자료구조에서도 반복문을 통해 저장된 값들을 순회시키는데 편리함을 제공한다. 즉, `모든 자료구조에서 공통적으로 사용 가능`하고 `사용법이 간단하다`는 장점이 있다.(메소드명 자체가 용도를 이해하기 좋음.)

### 예제코드
```java
import java.util.HashSet;
import java.util.Iterator;

public class Memo {
    public static void main(String[] args) {
        // HashSet 객체 생성
        HashSet<Integer> set = new HashSet<Integer>();
        // 1 ~ 10 까지의 값 add
        for(int i=1; i<= 10; i++){
            set.add(i);
        }
        // 기본 for문 Loop
        for(Integer temp : set){
            System.out.println(temp);
        }
        // Iterator 활용 Loop
        Iterator<Integer> tempSet = set.iterator();
        while(tempSet.hasNext()){
            System.out.println(tempSet.next());
        }
    }
}
```

> [참고사이트 1](http://www.tcpschool.com/java/java_collectionFramework_concept)

> [참고사이트 2](https://www.crocus.co.kr/1553)

> [참고사이트 3(+ 이미지 출처)](https://tlatmsrud.tistory.com/61)