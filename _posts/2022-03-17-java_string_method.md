---
layout: post
title: "String 주요 메소드 정리"
author: kimcno3
categories: java
tags: java
---

> **자세한 설명 및 매개변수 종류는 [API](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#matches(java.lang.String)) 참고**

### ✔️ 비교 & 검색
|Return Type|Method Name|Description|
|--|--|--|
|int|length()|문자열 길이를 출력|
|boolean|isEmpty()|문자열이 비어있는지 확인. 비어있다면 true|
|boolean|equals()|대상이 되는 문자열과 매개변수로 받아온 문자열이 동일한지 확인|
|boolean|equalsIgnoreCase()|equals()와 같은 기능이지만 대소문자를 구분하지 않고 비교|
|int|compareTo()|두 문자열이 같은지 확인하고 상대적인 위치값을 리턴|
|int|compareToIgnoreCase()|compareTo()와 같은 기능이지만 대소문자를 구분하지 않고 비교|
|boolean|contentEquals()|매개변수로 받는 CharSequence와 StringBuffer 객체가 String 객체와 같은지 비교|
|boolean|startsWith()|String 객체가 매개변수로 시작하는지 여부를 확인|
|boolean|endsWith()|String 객체가 매개변수로 끝나는지 여부를 확인|
|boolean|contains()|String 객체에 매개변수가 포함되어 있는지 여부를 확인|
|boolean|matches()|contains()와 같은 기능을 하지만 매개변수가 **정규표현식**인 점이 contains()와 차이점|
|boolean|regionMatches()|문자열 중 특정 영역이 매개변수로 넘어온 값과 동일한지 확인|

### ✔️ 위치 탐색
|Return Type|Method Name|Description|
|--|--|--|
|int|indexOf()|매개변수로 받는 값(String, char 등)과 동일한 문자열 중 가장 앞에 있는 위치값을 리턴|
|int|lastIndexOf()|매개변수로 받는 값(String, char 등)과 동일한 문자열 중 가장 마지막에 있는 위치값을 리턴|

> 두 메소드는 탐색 시작 위치를 매개변수를 통해 설정할 수 있고, 시작 위치값은 문자열의 가장 왼쪽부터 계산한다.

### ✔️ 추출
|Return Type|Method Name|Description|
|--|--|--|
|char|charAt()|특정 위치의 문자열에 해당하는 char을 리턴|
|void|getChars()|문자열에 해당하는 char값들을 매개변수로 지정한 char배열에 저장|
|int|codePointAt()|특정 위치의 문자열에 해당하는 유니코드값을 리턴|
|int|codePointBefore()|특정 위치 앞에 있는 문자열의 유니코드값을 리턴|
|int|codePointCount()|지정 범위 안의 유니코드 개수를 리턴|
|int|offsetByCodePoints()|지정된 index부터 offset(상대주소)이 설정된 인덱스를 리턴|
|static String|copyValueOf()|매개변수로 받아온 char배열을 문자열로 변환|
|char[]|toCharArray()|String 객체를 char 배열로 변환|
|String|subString()|매개변수값으로 지정된 위치의 문자열을 잘라내서 Sting 객체로 리턴|
|CharSequence|subSequence()|매개변수값으로 지정된 위치의 문자열을 잘라내서 CharSequence 객체로 리턴|
|String[]|split()|매개변수값(정규표현신)을 기준으로 문자열을 나눠, String 배열로 리턴|

### ✔️ 변경
|Return Type|Method Name|Description|
|--|--|--|
|String|trim()|문자열 맨 앞과 맨 뒤에 있는 공백을 제거|
|String|replace()|특정 문자열을 다른 문자열로 교체|
|String|replaceAll()|정규표현식에 해당되는 모든 문자열 원하는 문자열로 교체|
|String|replaceFirst()|정규표현식에 해당되는 문자열 중 가장 첫 번째 문자열만 원하는 문자열로 교체|
|static String|format()|특정 조건에 해당되는 문자열(`%s`, `%d`, `%f`, `%%`)을 매개변수값으로 변경|
|static String|valueOf()|매개변수값을 String 객체로 변환|