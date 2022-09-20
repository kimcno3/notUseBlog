---
layout: post
title: "[football] Github Actions를 활용한 CI/CD"
author: kimcno3
categories: f-lab
tags: f-lab project2
---

## # 문제점
***
로컬환경에서 수정이 계속되는 jar 파일을 직접 터미널을 통해 ec2 서버에 전송해주고 실행시키는 작업 환경이 매우 반복적이고 오랜 시간이 걸리는 불편함을 느꼈고 이를 자동화를 통해 해결해 했습니다.

## # 해결방안
***
### 1. Github Actions
> Github Actions에 대한 정보와 특징은 [CI 툴로 Github Actions를 선택했던 이유](http://localhost:4000/f-lab/2022/06/24/Github-Actions%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-CI-%EA%B5%AC%EC%84%B1.html)와 동일합니다.

이번 글에선 Github Actions가 구성하는데 알아야 했던 핵심 개념들에 대해 적어봤습니다.

### 2. Workflows
GitHub Actions에서 가장 상위 개념인 워크플로우(Workflow, 작업 흐름)는 쉽게 말해 자동화 해놓은 작업 과정이라고 볼 수 있습니다. 워크플로우는 코드 저장소 내에서 .github/workflows 폴더 아래에 위치한 YAML 파일로 설정하며, 하나의 코드 저장소에는 여러 개의 워크플로우, 즉 여러 개의 YAML 파일을 생성할 수 있습니다.

이 워크플로우 YAML 파일에는 크게 2가지를 정의해야 하는데, 첫번째는 `on` 속성을 통해서 해당 워크플로우가 언제 실행되는지를 정의합니다.

예를 들어, 코드 저장소의 main 브랜치에 push 이벤트가 발생할 때 마다 워크플로우를 실행하려면 다음과 같이 설정해줍니다.
```yaml
name: Java CI with Gradle

on:
  pull_request:
    branches: [ main ]
```

또는 사용자가 수동으로 워크플로우를 실행시키고 싶은 경우엔 `workflow_dispatch` 를 사용해 원하는 브랜치에서 원하는 시점에 워크플로우를 실행시킬 수 있습니다.

```yaml
name: Java CI with Gradle

on:
  workflow_dispatch
```

### 3. Jobs
GitHub Actions에서 작업(Job)이란 독립된 가상 머신(machine) 또는 컨테이너(container)에서 돌아가는 하나의 처리 단위를 의미합니다. 하나의 워크플로우는 여러 개의 작업으로 구성되며 적어도 하나의 작업은 있어야 합니다.그리고 모든 작업은 기본적으로 동시에 실행되며 필요 시 작업 간에 의존 관계를 설정하여 작업이 실행되는 순서를 제어할 수도 있습니다.

그리고 필수로 들어거야 하는 `runs-on` 속성을 통해 해당 리눅스나 윈도우즈와 같은 실행 환경을 지정해줘야 합니다.

### 4. Steps
작업 단계는 단순한 커맨드(command)나 스크립트(script)가 될 수도 있고 다음 섹션에서 자세히 설명할 액션(action)이라고 하는 좀 더 복잡한 명령일 수도 있습니다. 커맨드나 스크립트를 실행할 때는 run 속성을 사용하며, 액션을 사용할 때는 uses 속성을 사용합니다.

### 5. Actions
마지막으로 살펴볼 개념은 GitHub Actions의 꽃이라고 볼 수 있으며 서비스 이름에도 들어있는 바로 액션(Action)입니다. 액션은 GitHub Actions에서 빈번하게 필요한 반복 단계를 재사용하기 용이하도록 제공되는 일종의 작업 공유 메커니즘이며 하나의 코드 저장소 범위 내에서 여러 워크플로우 간에서 공유를 할 수 있을 뿐만 아니라, 공개 코드 저장소를 통해 액션을 공유하면 GitHub 상의 모든 코드 저장소에서 사용이 가능해집니다.

GitHub에서 제공하는 대표적인 공개 액션으로 바로 위 예제에서도 사용했던 체크 아웃 액션(`actions/checkout`)이며, 대부분의 CI/CD 작업은 코드 저장소로 부터 코드를 작업 실행 환경으로 내려받는 것으로 시작됩니다.

뿐만 GitHub Marketplace에서는 수많은 벤더(vendor)가 공개해놓은 다양한 액션을 쉽게 접할 수가 있어 액션을 중심으로 하나의 큰 커뮤니티가 형성이 되고 더 많은 사용자와 벤더가 GitHub Actions으로 몰려드는 선순환이 일어나고 있습니다.

### 6. 최종 코드
저는 CI 와 CD를 위한 동작 시점을 다르게 구성해 독립적인 파일로 관리하고자 두 워크플로우 파일을 생성했고 CI의 경우는 PR이 생성되고 진행되는 시점에, CD는 제가 원하는 시점에 동작하도록 트리거를 구성했습니다.

또한 CD 워크플로우의 경우 Actions를 적극 활용하면서 코드의 가독성과 편리성을 높일 수 있었습니다. 특히 ec2로 구성된 private 서버를 접속하는 경우 `proxy` 커맨드를 활용해 배스천 서버 통해 private 서버로 접근할 수 있는 구성을 간편하게 설정할 수 있었습니다.

#### CI Workflow
```yaml
name: Java CI with Gradle

on:
  pull_request:
    branches: [ main ]

permissions:
  contents: read

jobs:
  # 빌드 절차
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    # java 11 설치
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'

    # gradlew 에 대한 권한 설정
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew

    # Build 진행
    - name: Build with Gradle
      run: ./gradlew clean build
```

#### CD Workflow
```yaml
name: Java CD with Gradle

on:
  # 개발자가 원하는 지점에 배포될 수 있도록 구성
  workflow_dispatch:

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # java 11 설치
      - name: Set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'

      # gradlew 에 대한 권한 설정
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # Build 진행
      - name: Build with Gradle
        run: ./gradlew clean build

      # 디렉토리 생성
      - name: Make Directory
        run: mkdir -p football football/chatting football/api

      # Api Jar 파일 복사
      - name: Copy Api Jar
        run: cp ./football-api-server/build/libs/*.jar ./football/api

      # Chatting Jar 파일 복사
      - name: Copy Chatting Jar
        run: cp ./football-chatting-server/build/libs/*.jar ./football/chatting

      # 웹소켓 서버(= 배스천 서버)로 생성된 api jar 파일이 저장된 폴더를 전송
      - name: Deliver File To Bastion Server
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.SSH_WC_HOST }} # 인스턴스 IP 주소
          username: ${{ secrets.SSH_WC_USERNAME }} # 인스턴스에 지정된 user name(ex. ubuntu, ec2-user 등)
          key: ${{ secrets.SSH_KEY }} # SSH key의 private key
          port: ${{ secrets.SSH_PORT }} # port 번호(보통 22 포트 사용)
          source: "./"
          target: "source"
          rm: true

      # 배스천 서버에 접속해 api 서버로 jar 파일만 따로 전송
      - name: Deliver File To Private Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_WC_HOST }}
          username: ${{ secrets.SSH_WC_USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script:
            SOURCE_DIR=./source/football/api
            FILE_NAME=`find $SOURCE_DIR/*.jar -printf "%f\n"`
            
            scp -i ./.ssh/shkim-keypair.cer $SOURCE_DIR/$FILE_NAME ${{ secrets.SSH_API_USERNAME }}@${{ secrets.SSH_API_HOST }}:$SOURCE_DIR

      # Api 서버 접속 및 실행
      - name: Run Api Server
        uses: appleboy/ssh-action@master
        with:
          # 접속할 private 서버 관련 변수
          host: ${{ secrets.SSH_API_HOST }} 
          username: ${{ secrets.SSH_API_USERNAME }} 
          key: ${{ secrets.SSH_KEY }} 
          port: ${{ secrets.SSH_PORT }}

          # 배스천 서버 관련 변수
          proxy_host: ${{ secrets.SSH_WC_HOST }} 
          proxy_username: ${{ secrets.SSH_WC_USERNAME }} 
          proxy_key: ${{ secrets.SSH_KEY }} 
          proxy_port: ${{ secrets.SSH_PORT }}
          script: |
            SOURCE_DIR=./source/football/api
            FILE_NAME=`find $SOURCE_DIR/*.jar -printf "%f\n"`
            PID=`ps -ef | grep occupying | grep sudo | grep -v "bash -c" | awk '{print $2}'`

            if [ -z "$PID" ]; then
            echo "#### THERE IS NO PROCESS ####"
            else
            echo "#### KILL $PID ####"
            sudo kill $PID
            fi

            echo "#### RUN $SOURCE_DIR/$FILE_NAME ####"

            sudo java -jar -Dspring.profiles.active=prod $SOURCE_DIR/$FILE_NAME > /dev/null 2>&1 &

      # Chatting 서버 접속 및 실행
      - name: Run Chatting Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_WC_HOST }}
          username: ${{ secrets.SSH_WC_USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            SOURCE_DIR=./source/football/chatting
            FILE_NAME=`find $SOURCE_DIR/*.jar -printf "%f\n"`
            PID=`ps -ef | grep occupying | grep sudo | grep -v "bash -c" | awk '{print $2}'`

            if [ -z "$PID" ]; then
            echo "#### THERE IS NO PROCESS ####"
            else
            echo "#### KILL $PID ####"
            sudo kill $PID
            fi

            echo "#### RUN $SOURCE_DIR/$FILE_NAME ####"

            sudo java -jar -Dspring.profiles.active=prod $SOURCE_DIR/$FILE_NAME > /dev/null 2>&1 &
```
## # 마치며
***
Github Actions를 통해 수정된 jar 파일을 수동으로 aws 서버로 전송하지 않고 간편하게 배포까지 하는 로직을 구현하면서 엄청난 편리함을 얻을 수 있었고, CI/CD를 구성하는 과정을 직접 경험해보면서 하나의 제품이 어떻게 빌드되고 배포되는지 그 흐름을 이해할 수 있었습니다.

## # 참고자료
***
- https://www.daleseo.com/github-actions-basics/
- https://veluxer62.github.io/tutorials/tutorial-of-continuous-deployment-with-git-actions/
- https://cloud.ibm.com/docs/solution-tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server
- https://github.com/marketplace/actions/ssh-remote-commands#how-to-connect-remote-server-using-proxycommand