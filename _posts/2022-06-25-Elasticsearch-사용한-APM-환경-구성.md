---
layout: post
title: "[soldout] Elasticsearch 사용한 APM 환경 구성"
author: kimcno3
categories: f-lab
tags: f-lab project1
---

## # 문제점
***
어플리케이션이 처리하는 비즈니스 로직들은 Spring 내에서는 여러 객체들의 의사소통을 통해 동작하고 그 외에 MySql, Redis처럼 다른 플랫폼에 요청을 보내기도 합니다. 이렇게 하나의 추상적인 로직은 여러 단계로 나뉘어 처리가 되기때문에 처리과정에서 발생할 수 있는 에러의 발생 지점이나 로직의 처리 속도의 최적화를 위해서는 각 단계의 처리 과정에 대한 정보를 제가 확인할 수 있어야 제대로된 관리를 할 수 있을 것이라 판단해 이를 위한 플랫폼이 무엇이 있을지 고민하게 되었습니다.

## # 해결방안
***
### APM(Application Performance Monitoring or Management)
APM이란 명칭 그대로 어플리케이션의 성능을 모니터링 혹은 관리해주는 시스템을 의미합니다. 리얼타임에서 서비스 어플리케이션의 성능을 보여주는데 정말 많은 지표들을 제공합니다.

APM을 활용한다면 제가 마주한 문제점인 요청을 통해 응답이 생성되고 사용자에게 제공되는 과정속에서 발생하는 에러가 정확히 어느 지점에서 발생하는지, 또는 성능 최적화를 위해 리팩토링해볼 지점이 어느 위치인지를 시각적으로 확인하고 정보를 활용할 수 있습니다.

저는 APM을 제공해주는 여러 툴 중에서도 `Elastic APM`을 활용해봤습니다.

### Elastic APM 구성 요소
- `APM Agent` : 어플리케이션에 붙어 성능을 자동으로 측정하고 에러를 추적하는 도구
- `APM server` : Agent가 측정한 데이터를 받아 Elasticsearch에 맞게 변환해서 보내주는 역할
- `Elasticsearch` : 분산형 검색, 분석 엔진으로 Agent가 보내준 데이터를 저장
- `Kibana` : Elasticsearch에 저장된 데이터를 기반으로 UI를 제공하면서 데이터를 가시화

### 적용 방법
적용 방법 및 과정은 다음 블로그를 참고했습니다.
> 참고 블로그로 이동을 원하시면 [링크](https://cheese10yun.github.io/elk-apm-1/#null)를 클릭하시기 바랍니다.

또한 Elastic APM 구성을 위해 별도의 `Docker-compose` 파일을 작성해 다른 플랫폼과 독립적으로 관리할 수 있도록 했습니다.

### Kibana를 통해 시각화된 데이터의 모습
**시각화되어서 나오는 데이터**
<p align="center"><img width="1416" alt="스크린샷 2022-09-12 오후 7 21 47" src="https://user-images.githubusercontent.com/77563468/189632929-0c783a74-05f4-4087-95ac-0b006a41c976.png"></p>

**요청이 처리되는 과정을 클래스, DB 별로 구분되어 여러 자료를 제공**
<p align="center"><img width="1102" alt="스크린샷 2022-09-12 오후 7 33 19" src="https://user-images.githubusercontent.com/77563468/189632872-2c77f25d-b7ec-46d2-b6dd-c117a31ecbfa.png"></p>

APM을 통해 위 로직에서 N+1 쿼리가 발생하는 지점을 확인할 수 있었고 이를 해결하기 위한 고민을 할 수 있었습니다.

## # 마치며
***
Elastic APM을 통해 시각화된 자료를 가지고 빠르게 문제를 해결할 수 있는 환경을 구성할 수 있었고 성능 최적화를 위해 매우 중요한 정보를 얻을 수 있게 되었습니다.

## # 참고자료
***
- https://beaniejoy.tistory.com/54
- https://cheese10yun.github.io/elk-apm-1/#null