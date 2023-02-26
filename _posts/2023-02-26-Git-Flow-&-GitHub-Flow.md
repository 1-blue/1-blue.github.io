---
layout: post
title: Git Flow & GitHub Flow
author: admin
date: 2023-02-26 18:47:00 +900
lastmod: 2023-02-26 18:47:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, Git/GitHub]
tags: [Git, Git Flow, GitHub Flow]
---

> 해당 포스트는 `Git-Flow`전략에 대한 포스트입니다.<br />혹시 `Git`에 대한 명령어에 대해 알고 싶다면 [여기](/posts/Git-GitHub/){:target="_blank"}를 추천드립니다.
{: .prompt-info}

# ⏳ Git Flow
`Git Flow`란 `Git`을 이용하여 개발을 진행하는 전략 중에 하나입니다.<br />

개인 혹은 팀원들과 협업을 통해서 `Git`을 사용한다면 팀원들 모두가 이해만 하고 있다면 어떤 전략을 사용해도 상관없습니다.<br />
하지만 `Git Flow`에 대해 배우고 공부하는 이유는 많은 사람들이 사용하고 사용했던 유명한 전략이기 때문이라고 생각합니다.<br />
그만큼 안정적이고 자료도 많으며 처음 협업하면서 어떤 전략을 사용해야 할지 감을 잡기 어려워서 일단은 유명한 전략을 공부하고 이후에 필요한 부분을 바꿔가면서 적용하자는 의도로 해당 포스트를 작성하게 되었습니다.<br />

![Git-Flow](https://user-images.githubusercontent.com/63289318/221396296-a8064f7e-7ad4-4a89-893a-6e915f306461.png){: width="800" height="500" }
_Git-Flow_

`Git Flow`에는 몇 가지 브랜치와 규칙이 있습니다.<br />
그것들에 대해서 하나씩 알아보겠습니다.<br />

## 0️⃣ master
기준이 되는 브랜치입니다. ( `main` 혹은 `master` )<br />
해당 브랜치는 항상 즉시 배포가 가능한 상태여야합니다.<br />
또한 `master`는 `PR`을 통해서만 작업을 해야합니다. ( 바로 `push`하면 안됩니다. )<br />

해당 브랜치를 기반으로 `develop` 브랜치를 생성합니다.<br />

## 1️⃣ develop
개발 진행에 사용하는 브랜치입니다.<br />
또한 `develop`도 `PR`을 통해서만 작업을 해야합니다. ( 바로 `push`하면 안됩니다. )<br />

해당 브랜치를 기반으로 `feature`와 `release` 브랜치를 생성합니다.<br />

## 2️⃣ feature
기능별 개발에 사용되는 브랜치입니다.<br />

`feature/기능명` 형태로 브랜치를 만들고 해당 기능에 대한 개발을 진행합니다.<br />
( `feature` 브랜치는 여러 개를 만들고 각자 기능을 작업합니다. )<br />

한 가지 기능을 만드는 동안 필요한 커밋을 직접 `push`합니다.<br />

기능 개발이 끝났다면 `PR`을 통해서 `develop`와 병합합니다. ( + `review` )<br />
만약 `PR`에 의한 병합에 `conflict`가 발생한다면 `feature`에서 `develop`과 먼저 병합한 뒤 충돌 대상자끼리 협의 후 해결하고 다시 `PR`을 합니다.<br />

## 3️⃣ release
배포전에 테스트를 진행하는 브랜치입니다.<br />

여러 기능을 만들어 배포할 준비가 된 `develop` 브랜치를 기반으로 생성합니다.<br />
배포전 테스트를 진행하고 수정할 부분을 바로 추가합니다.<br />
테스트가 끝나고 확실하게 배포할 준비가 되었다면 `PR`을 통해서 `master` 브랜치와 병합합니다.<br />

## 4️⃣ hotfix
배포 이후 즉, `master`와 `develop`을 병합하고 난 뒤에 버그를 발견하는 경우 사용하는 브랜치입니다.<br />

`master`를 기준으로 생성하고 버그 수정에 필요한 부분을 바로 `hotfix`에서 수정하고 버그가 해결되면 `master` 브랜치와 병합합니다.<br />

`master`가 병합되고 난 뒤에는 `PR`을 통해서 `develop`에 병합해야합니다.<br />
`master` 브랜치로 부터 시작되기 때문에 `develop`에 적용한 충돌은 `develop`에서 해결해야 합니다.<br />

## 5️⃣ 요약
+ `master`: 항상 배포가 가능한 원본 브랜치
+ `develop`: 다음 버전 개발을 진행하는 브랜치 ( 중간 다리 역할 )
+ `feature`: 기능 단위로 개발을 진행할 때 사용하는 브랜치
+ `release`: 배포 전에 테스트를 실행할 브랜치
+ `hotfix`: 배포 이후에 발견된 버그를 빠르게 해결하는데 사용하는 브랜치

1. `master`로 부터 `develop` 생성
2. `develop`으로 부터 `feature/기능명` 생성
3. `feature`에서 각 기능 개발 후 완료 시 `PR`과 `Review`를 통해 `develop`과 병합
4. 원하는 기능을 모두 구현하고 배포할 준비가 되었다면 `develop`을 기반으로 `release` 생성
5. `release`로 배포 및 테스트 진행하고 수정할 부분은 바로 커밋
6. `release`에서 테스트가 끝났다면 `PR`로 `master`에 병합
7. 만약 이후에 `master`에서 버그가 발견되었다면 `master`를 기준으로 `hotfix`를 생성하고 버그를 해결
8. 버그를 해결하고 `master`에 바로 병합하고 이후 `master`를 `develop`에 병합

## 6️⃣ 주의 사항
1. 모든 브랜치를 병합할 때는 항상 브랜치 병합 커밋 이력을 남겨야 추적을 할 수 있기 때문에 [`--no-ff`](/posts/Git-GitHub/#1%EF%B8%8F⃣9%EF%B8%8F⃣-git-merge-branch-name){:target="_blank"} 옵션을 사용해서 병합해야합니다.<br />
2. `PR`에 대한 규칙을 협의 후 정해둬야 합니다. ( 몇 명의 승인이 있어야 `PR`할지 등 )

# 🫗 GitHub Flow
`GitHub Flow`란 `Git`을 이용하여 개발을 진행하는 전략 중에 하나입니다.<br />
`Git Flow`보다는 비교적 단순한 구조입니다.<br />

항상 `master`를 기반으로 브랜치를 생성합니다.<br />

기능 추가, 버그 해결 등 모두 `master`를 기반으로 생성하되 어떤 역할인지 알 수 있도록 네이밍을 해야합니다.<br />
( 팀원끼리 협의를 통해 상황에 맞는 네이밍을 정하기만 하면 될 것 같습니다. )<br />

특정 단위를 작업을 하고 `master`와 병합해야 하는 경우에는 `PR`을 통해서 `Review` 이후에 병합합니다.<br />
( `Git Flow`처럼 중간 테스트 과정이 없기 때문에 확실하게 `Review`하고 병합해야합니다. )<br />

대부분 `GitHub Action`같은 `CI/CD`를 통해서 자동으로 배포되도록 합니다.<br />
만약 `CI/CD`를 처리하지 않았다면 수동으로 배포하면 됩니다.<br />

+ 요약
  1. 기능, 버그 등 모든 브랜치는 `master`를 기반으로 생성
  2. 개발이 완료되었다면 `PR`과 `Review`를 통해 해당 브랜치와 `master`를 병합
  3. `master`를 기준으로 배포

# 📮 레퍼런스
1. [Youtube - Git Flow](https://www.youtube.com/watch?v=JHWrQWsiwQ8){:target="_blank"}
2. [Youtube - 테코톡 - Git 브랜칭 전략](https://www.youtube.com/watch?v=wtsr5keXUyE){:target="_blank"}
3. [Youtube - 생활 코딩 - Git Flow Model](https://www.youtube.com/watch?v=EzcF6RX8RrQ){:target="_blank"}
4. [inpa - 깃 브랜치 전략 정리](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5){:target="_blank"}

1. [1-blue - Git 명령어 정리](/posts/Git-GitHub/){:target="_blank"}
1. [1-blue - Git 명령어 정리](/posts/Git-GitHub/){:target="_blank"}