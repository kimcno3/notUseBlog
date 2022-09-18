---
layout: post
title: GitHub flow를 통한 브랜치 관리
author: kimcno3
categories: f-lab
tags: f-lab project1
---

## # 문제점
여러 개발자가 협업을 통해 하나의 서버를 구현하기 위해선 코드 버전관리에 대한 규약을 정할 필요해 이를 정해볼 필요가 있었습니다.

## # 해결 방안
### 1. Git Flow

![](https://user-images.githubusercontent.com/43775108/125800526-2ea36d8e-6262-4ba5-9ef0-af7845131d85.png)

git flow는 feature, develop, release, hotfix, master 5가지의 브랜치로 나눠 각 브랜치에 목적에 맞게끔 버전관리를 하는 전략입니다.

#### 1. feature
- feature 브랜치는 기능의 구현을 담당
- 브랜치명은 팀마다 컨벤션을 가지고 지을 수 있지만 feature/{구현기능명}과 같은 명칭을 준수하는 것이 일반적이다.
- feature 브랜치는 develop 브랜치에서 생성되며, develop 브랜치로 머지된다.
- 머지된 후에는 해당 브랜치가 삭제된다.

#### 2. develop
- develop 브랜치는 말 그대로 개발을 진행하는 브랜치로 중심적인 브랜치
- 하나의 feature 브랜치가 머지될 때마다 develop 브랜치에 해당 기능이 더해지며 살을 붙여나갑니다.
- develop 브랜치는 배포할 수준의 기능을 갖추면 release 브랜치로 머지

#### 3. release
- release 브랜치는 개발된 내용을 배포하기 위해 준비하는 브랜치이다.
- 브랜치명은 release-1과 같은 방식으로 첫번째 릴리즈, 두번째 릴리즈 등을 지정하는 것이 보편적이다.
- release 브랜치에서 충분한 테스트를 통해 버그를 검사하고 수정해 배포할 준비가 완전히 되었다고 판단되면 master로 머지해 배포된다.
- release 브랜치는 develop 브랜치에서 생성되며 버그 수정 내용을 develop 브랜치에도 반영하고, 최종적으로 master 브랜치에 머지된다.

#### 4. hotfix
- hotfix 브랜치는 배포된 소스에서 버그가 발생하면 생성되는 브랜치이다.
- 브랜치명은 hotfix-1로 지정된다. 
- release 브랜치를 거쳐 한차례 버그 검사를 했지만 예상치 못하게 배포 후에 발견된 버그들에 대해서 수정한다. hotfix 브랜치는 master 브랜치에서 생성되며, 수정이 완료되면 develop 브랜치, release 브랜치와 master 브랜치에 수정 사항을 반영한다.

#### 5. master
- master 브랜치는 최종적으로 배포되는 가장 중심의 브랜치이다.
- develop 브랜치에서는 개발이 진행되는 와중에도 이전 release 브랜치 내용이 master에 있어 배포되어 있다.

Git flow 브랜치 전략은 여러 브랜치들이 존재하고 각 브랜치마다 상황이 명확하게 분류되어 있지만, 오히려 이렇게 많은 브랜치가 흐름을 더욱 복잡하게 만들기도 하며 release와 master의 구분이 모호하기도 단점이 있습니다. 하지만 프로젝트의 규모가 커지면 커질수록 소스코드를 관리하기에 용이합니다.

### 2. Github Flow
github flow는 git flow의 브랜치 전략이 너무 복잡하고 적용하기 어렵다고 해서 생겨난 브랜치 전략입니다. 그레서 github flow는 master 브랜치 하나만을 가지고 진행되며 master 브랜치는 어떤 기능이 구현되든, 오류가 수정되든 모두 master에 머지되어 항상 update된 상태를 유지되어야 합니다.

![](https://subicura.com/git/assets/img/github-flow.2fafce92.png)

#### 진행 과정
1. master에서 새로운 브랜치를 만든다.
2. 만든 브랜치에 파일을 추가하고 커밋을 한다.
3. 수정한 내용이 담긴 브랜치를 원격 저장소에 Push한다.
4. GitHub에서 푸시 된 브랜치를 Pull Request한다.
5. GitHub에서 코드리뷰를 한다.
6. GitHub에서 Merge한다.
7. 로컬 저장소에서 원격 저장소에 머지된 내용을 Pull한다.

즉 Github Flow는 복잡한 여러 브랜치를 목적에 맞게 활용하는 것이 아니라 하나의 master 브랜치에서 모든 브랜치를 시작하고 merge하기 전에 PR을 통해 강력하게 코드 리뷰를 한 뒤, 모든 테스트를 통과했다면 merge시키는 방식으로 진행됩니다.

### 프로젝트에 적용할 방법은?
결론부터 말씀드리면 이번 프로젝트에는 Github Flow를 채택해 활용해보기로 결정했습니다.

Git Flow가 규격이 매우 뚜렷하게 브랜치의 목적을 나눠 관리한다면 이후 프로젝트의 규모가 커졌을 때 매우 유리한 점이 있겠지만, 처음 프로젝트를 진행하는 입장에선 복잡한 브랜치 전략보단 간단하고 직관적으로 브랜치를 관리할 수 있는 Github Flow가 더 적합하다고 생각했습니다.

또한 Github Flow는 master 브랜치에 대한 엄격한 리뷰 및 테스트 과정이 보장된다면 다른 브랜치에 대한 규정은 자유롭기 때문에 브랜치 명을 통해 해당 브랜치에서 작업된 내용들에 대해 자세하고 명확하게 표현할 수 있다는 점이 현재 프로젝트 진행 상황에선 적합한 전략이라고 판단했습니다.

## # 참고 자료
- https://subicura.com/git/guide/github-flow.html#github-flow-%E1%84%87%E1%85%A1%E1%86%BC%E1%84%89%E1%85%B5%E1%86%A8
- https://subicura.com/git/guide/github-flow.html#github-flow-%E1%84%87%E1%85%A1%E1%86%BC%E1%84%89%E1%85%B5%E1%86%A8