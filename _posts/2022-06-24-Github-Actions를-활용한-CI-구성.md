---
layout: post
title: Github Actions를 활용한 CI 구성
author: kimcno3
categories: f-lab
tags: f-lab project1
---

## # 문제점
Github Flow를 브랜치 전략으로 선택했기 때문에 CI에 대한 구성이 필요했고 현재 상황에서 가장 효율적인 자동화 전략이 무엇인지 고민하게 되었습니다.

## # 해결방안
### Github Actions
Github Actions은 Github 저장소를 기반으로 소프트웨어 개발 Workflow를 자동화 할 수 있는 도구입니다. 간단하게 말하자면 Github에서 직접 제공하는 CI/CD 도구라고 할 수 있습니다. 

Workflow는 Github 저장소에서 발생하는 build, test, package, release, deploy 등 다양한 이벤트를 기반으로 직접 원하는 Workflow를 만들 수 있습니다.Workflow는 Runners라고 불리는 Github에서 호스팅 하는 Linux, macOS, Windows 환경에서 실행된다. 그리고 이 Runners를 사용자가 직접 호스팅하는 환경에서 직접 구동시킬 수도 있습니다.


### Github Actions를 선택한 이유

CI/CD 자동화를 위한 WorkFlow를 구현하기 위해선 Jenkins와 같은 무료 플랫폼도 존재합니다. 대표적으로 Gitub Actions와 Jenkins의 차이는 다음과 같습니다.

#### Jenkins
- 서버 설치 필요
- 작업 또는 작업이 동기화되어 제품을 시장에 배포하는 데 더 많은 시간이 소요 
- 계정 및 트리거를 기반으로하며 github 이벤트를 준수하지 않는 빌드를 중심으로합니다.
- 환경 호환성을 위해 도커 이미지에서 실행해야 함
- 캐싱 메커니즘을 지원하기 위해 플러그인을 사용할 수 있습니다.
- 공유할 수 있는 능력이 없습니다.
- 전세계많은 사람들이 이용하여 문서가 다양

#### GitHub Actions
- 클라우드가 있으므로, **별도 설치 필요없음**
- 비동기 CI / CD 달성
- 모든 github 이벤트에 대한 작업을 제공하고 **다양한 언어와 프레임 워크를 지원**합니다.
- **모든 환경과 호환**
- 캐싱이 필요한 경우 자체 캐싱 메커니즘을 작성해야 합니다.
- github 마켓 플레이스를 통해 공유 가능
- 젠킨스에 비해 문서가 없음

위 특징들을 비교해볼 때 Github Actions가 Jenkins 보다 **유연하고 다양한 언어와 환경에 호환이 가능**하다는 점이 장점으로 다가왔습니다. 

그리고 **YAML 파일로 구성이 가능**해 프로젝트에서 다루는 언어로 관리할 수 있다는 측면이 큰 장점으로 다가왔고, 무엇보다 **별도의 설치 및 계정 생성에 대한 과정을 생략**하고 Github 내에서 CI에 대한 구성과 관리를 할 수 있다는 점에 작업의 효율을 높이는 데 매우 중요할 것이라 판단했습니다.

### gradle.yml 파일
```yml
name: Java CI with Gradle

on:
  # main 브랜치에 대한 PR이 올라오는 경우에 해당 workflow를 실행
  pull_request:
    branches: [ main ]

permissions:
  contents: read

jobs:
  build:

    # CI 과정은 ubuntu 서버에서 동작
    runs-on: ubuntu-latest

    steps:

    - uses: actions/checkout@v3

    # JDK 설치
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'

    # Build 과정을 수행
    - name: Build with Gradle
      uses: gradle/gradle-build-action@0d13054264b0bb894ded474f08ebb30921341cee
      with:
        arguments: build

```

github actions는 마켓 플레이스에서 사용자가 원하는 동작에 대한 코드를 제공해줘 손 쉽게 위와 같은 yml 파일을 구성할 수 있습니다. 위처럼 workflow 정의해두고 프로젝트에 파일을 포함시켜두면 Github 저장소 내에서 CI에 대한 관리를 할 수 있도록 구성할 수 있습니다.

## # 마치며
코드 작성 외적인 작업이지만 꽤 많은 시간을 소요하게 되는 테스트 및 빌드 과정에 대해 자동화 구성을 해보기 위해 Jenkins와 Github Actions의 차이를 비교해보면서 **효율적인 업무 환경을 구성해보는 이점**을 가져오면서 **현재 프로젝트 상황에 적합한 방법을 선정해보는 경험**을 해볼 수 있었습니다.

## # 참고자료
- https://choseongho93.tistory.com/295
- https://www.daleseo.com/github-actions-basics/
- https://velog.io/write?id=748f7b13-4081-4aa8-9442-cd3ffb20ada5