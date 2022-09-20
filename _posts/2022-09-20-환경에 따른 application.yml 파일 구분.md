---
layout: post
title: "[football] 환경에 따른 application.yml 파일 구분"
author: kimcno3
categories: f-lab
tags:  f-lab project2
published: true
---

## # 문제점
***
AWS가 제공하는 원격 서버에 파일을 배포하다보니 로컬환경과 배포환경에서 필요한 환경 변수가 동일하지 않는 경우가 발생했습니다. 특히 DB를 연동하는 과정에서 사용되는 변수의 차이가 있었습니다. 그래서 프로젝트가 동작하는 환경에 따라 이를 구분된 파일로 관리하고 지정해줄 필요를 느꼈습니다.

## # 해결방안
***

### Spring Profiles
Spring에서는 Profile을 지정해 Application.yml 파일을 환경별로 구분해 생성 및 관리할 수 있습니다. Profile을 로컬(local)과 배포환경(prod)로 구분해 각기 다른 YAML 파일로 환경변수를 구성해보길 했습니다.

> local, dev, prod로 총 세개의 profile을 구성하고 싶었지만 AWS의 프리티어를 활용하기에 dev 환경에 대한 설정은 포기하고 prod로만 구성했습니다.

#### Application-local.yml
```yaml
server:
  port: 9090
  host:
    api: localhost:9090
    chatting:
      public: localhost:9000
      private: localhost:9000
spring:
  profiles: local

  datasource:
    url: jdbc:mysql://localhost:3306/football_db?characterEncoding=UTF-8
    username: football
    password: 1234
    driver-class-name: com.mysql.cj.jdbc.Driver
```

#### Application-prod.yml
```yaml
server:
  port: 8080
  host:
    api: football-lb-app-1899246839.ap-northeast-2.elb.amazonaws.com
    chatting:
      public: 43.201.5.143:8000
      private: 10.0.11.55:8000

spring:
  profiles: prod

  datasource:
    url: jdbc:mysql://football-db-mysql.ckt0scabogid.ap-northeast-2.rds.amazonaws.com:3306/football_db_mysql
    username: admin
    password: 1q2w3e4r!
    driver-class-name: com.mysql.cj.jdbc.Driver
```

위 두 파일에 선언된 변수의 값들이 다른 것을 확인할 수 있습니다. 

`application-{환경 이름}.yml` 형식으로 파일을 구분해 생성하고, 어플리케이션을 실행할 때 다음 명령어와 같이 `jvm property`로 환경 이름을 전달하면 그에 맞는 변수 값이 설정됩니다.

- **어플리케이션 실행 시 명령어**

```text
# 로컬환경에서 실행 시
java -jar -Dspring.profiles.active=local {Jar 파일명}

# 배포환경에서 실행 시
java -jar -Dspring.profiles.active=prod {Jar 파일명}
```

## # 마치며
***
Spring Profile을 설정해 환경에 따라 변수를 각기 다른 YAML 파일로 구성함으로써 배포 환경에 따른 독립적인 운용과 관리가 가능해졌습니다. 또한 실제 서비스를 운용하는 과정에서 필요한 환경변수 설정 방법을 학습하고 경함할 수 있었습니다.

## # 참고자료
***
- https://www.lesstif.com/spring/spring-profile-deploy-18220309.html
- https://gaemi606.tistory.com/entry/Spring-Boot-profile%EC%84%A4%EC%A0%95