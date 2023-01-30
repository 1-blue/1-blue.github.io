---
title: Git과 GitHub에 대한 정리
author: admin
date: 2022-12-28 10:15:00 +900
lastmod: 2023-01-29 11:08:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, Git/GitHub]
tags: [Git, GitHub]
---

> 해당 포스트는 `Git`과 `GitHub`에 대한 개념과 명령어들을 정리한 포스트입니다.<br />잘못된 내용이 포함될 수 있습니다.<br />
{: .prompt-info}

+ `[]`: 생략 가능
+ `<>`: 직접 입력

# 📌 용어 정리
## 0️⃣ Git
`Git`이란 버전 관리 시스템입니다. ( `Version Control System`, `vcs` )<br />
스냅샷 방식을 이용합니다.<br />

+ 스냅샷 방식: 모든 파일을 기록하는 것이 아닌 변경된 부분만 기록하는 방식

## 1️⃣ GitHub
`GitHub`은 `Git`의 호스팅 사이트입니다.<br />
다른 사이트로는 `GitLab`, `Bitbucket` 등이 있습니다.<br />

## 2️⃣ 저장 공간
+ `working`: 실제로 파일들이 존재하는 작업공간
+ `stage`: 임시로 저장되는 공간
+ `repository`: 버전의 파일들을 관리하고 저장하는 공간
	1. `local repository`: 개인 컴퓨터에 존재하는 `repository`
	2. `remote repository`: 원격 `repository` ( `GitHub`에 존재하는 `repository` )

`git push <remote-repository-alias> <branch-name>`를 통해서 `local repository`에 저장된 내용을 `remote repository`의 `branch`로 업로드합니다.<br />
`git pull <remote-repository-alias> <branch-name>`를 통해서 `remote repository`에 저장된 내용을 `local repository`의 `branch`로 가져옵니다.<br />

+ `remote-repository-alias`은 기본적으로 `origin`을 사용합니다. ( [remote 명령어](/posts/Git-GitHub/#8%EF%B8%8F⃣-git-remote) )
+ `branch-name`은 새로 생성하지 않았다면 `main` 혹은 `master`를 사용합니다.

## 3️⃣ 파일 상태
+ `untracked`: 한번도 `stage`나 `repository`에 등록되지 않은 파일 ( `git`에서 변경을 감지하지 않음 )
+ `tracked`: 이미 `stage`나 `repository`에 등록된 파일 ( `git`에서 변경을 감지함 )
+ `unmodified`: `tracked`이면서 수정되지 않은 파일
+ `modified`: `tracked`이면서 수정된 파일

## 4️⃣ HEAD
커밋을 가리키는 포인터를 의미합니다.<br />
아무런 조작을 하지 않은 경우에는 최신 커밋을 가리킵니다.<br />

+ `HEAD^`: 현재 위치 기준으로 바로 이전 커밋 선택
+ `HEAD~n`: 현재 위치 기준으로 `n`번째 이전 커밋 선택<br />

+ `git reset --soft HEAD^`: 제일 최근 커밋 취소 ( `stage` 유지 )

## 5️⃣ commit
커밋을 파일의 변화를 깃 저장소에 영구적으로 기록하는 것을 의미합니다.<br />

+ `HEAD` + `stage` => 새로운 커밋


# ✏️ 명령어 정리
## 0️⃣ git config
각 `repository`마다 작성자와 이메일을 등록하는 방법입니다.<br />
레포지토리마다 등록하기 귀찮기 때문에 `--global`로 등록해두면 앞으로 `push`/`pull`하는 모든 레포지토리에 기본 유저와 이메일이 등록됩니다.<br />
+ `git config`
	1. `git config --global user.name <name>`
	2. `git config --global user.email <email>`

> 만약 `user.name`과 `user.email`을 등록하지 않으면 `GitHub`에 커밋 이력이 제대로 등록되지 않을 수 있습니다.<br />
{: .prompt-tip }

## 1️⃣ git init `[<location>]`
`git`에서 버전을 관리해주는 폴더로 만드는 명령어입니다.<br />
`git init [경로]`를 입력해주면 `.git`파일이 생기며 해당 파일에 현재 폴더의 버전에 대한 데이터가 담깁니다.<br />

## 2️⃣ git status
현재 파일들의 상태를 보여줍니다.<br />
`untracked`, `tracked`, `modified`, `unmodified` 등의 상태를 보여줍니다.<br />

## 3️⃣ git add `<location>`
지정한 파일들을 `stage`공간(임시공간)으로 옮기는 명령어입니다.<br />

## 4️⃣ git rm `--cached` `<file-name>`
`state`에 있는 파일을 `working`으로 이동하는 명령어입니다.<br />
즉, `git add`를 취소하는 명령어입니다.<br />

+ 테스트 결과 `.`으로 모든 파일은 안되고 명시한 파일만 가능합니다.<br />

## 5️⃣ git commit
`stage`에 있는 파일들을 `local repository`로 옮기는 명령어입니다.<br />
여기서 반드시 `stage`에 파일이 있어야 하며, 메시지가 있어야 합니다.<br />

만약 `-m`을 이용하지 않는다면 `git bash` 기준으로 `vim`으로 들어가지며, 자세한 커밋 메시지를 작성해야합니다.<br />

+ `-a`: `add`를 자동으로 해주고 커밋을 하는 옵션 값
+ `-m`: 간단하게 메시지를 작성하는 옵션 값
+ `--allow-empty-message`: 메시지를 비워두고 커밋을 허용하는 옵션 값
+ `-v`: 변경사항을 메시지에 추가해준다.

## 6️⃣ git log
커밋 목록을 보여주는 명령어이다.

+ `--oneline`: 한줄로 간단하게 로그를 보는 옵션 값
+ `--graph`: 그래프를 보여주는 옵션 값
+ `-<숫자>`: 숫자만큼의 기록만 보여줌

## 7️⃣ git diff
+ `git diff`: `working`과 `stage` 비교
+ `git diff head`: `HEAD`와 `state` 비교

## 8️⃣ git remote
+ `git remote add <원격 저장소 별칭> <원격 저장소 URL>`: 원격 저장소 연결
+ `git remote`: 원격 저장소 별칭 출력
+ `git remote -v`: 원격 저장소 목록 출력
+ `git remote rename <변경전> <변경후>`: 원격 저장소 별칭 변경
+ `git remote show <원격 저장소 별칭>`: 등록된 원격 저장소의 상세내용 출력
+ `git remote rm <원격 저장소 별칭>`: 등록된 원격 저장소 제거

## 9️⃣ git push
로컬 저장소에 커밋한 내용을 원격 저장소에 동기화하는 명령어

+ `git push <원격 저장소 별칭> <브랜치>[:<새로운 브랜치 이름>]`  
+ `-u`: 업스트림 설정
+ `git push origin --delete <원격 브랜치 이름>`: 원격 브랜치 삭제
+ `git push --set-upstream origin master`: 로컬과 원격 저장소의 master브랜치를 연결시켜주는 명령어

## 1️⃣0️⃣ git clone `<url>` `<location>`
원격 저장소의 파일들과 `.git`파일을 정해준 위치에 복사해주는 명령어입니다.<br />
기존 설정값까지 모두 복제됩니다.<br />
( 원격 저장소 별칭, `user.name`, `user.email` 등등 )

+ `.git`: `git`의 설정 값, 히스토리 등을 저장하고 있는 폴더

## 1️⃣1️⃣ git pull
> `pull` === `fetch` + `merge`입니다.
{: .prompt-tip }

원격 저장소 등록 후 사용이 가능하고, 사용 시 원격 저장소의 갱신된 내용을 내려받습니다.<br />
기본적으로 자동 병합이 일어나고 로컬 저장소의 변경점과 원격 저장소의 변경점에 충돌(`conflict`)이 있으면 병합을 중지하고 그게 맞게 처리하라는 문구를 띄워줍니다.<br />

+ 자동 병합: 로컬 저장소 + `pull`받은 내용을 저장한 임시 영역을 병합

## 1️⃣2️⃣ git fetch
원격 저장소 등록 후 사용이 가능하고, 사용 시 원격 저장소의 갱신된 내용을 내려받습니다.<br />
( `FETCH_HEAD`라는 `branch`에 내려받음 )<br />

```bash
# 아래 명령어 둘 다 fetch한 브랜치로 이동됩니다.
git checkout FETCH_HEAD
git checkout origin/master

# 로그 확인
git log --oneline -3

# 브랜치 검색
git branch
```

`git pull`과 다르게 직접 병합을 해줘야한다.<br />
( `git merge origin/master` 명령어 사용 )<br />

## 1️⃣3️⃣ git branch
현재 브랜치 목록과 사용 브랜치를 보여주는 명령어입니다.<br />

+ `git branch <branch-name> [<commit-id>]`
	`commit-id` 생략 시 `HEAD` 포인터를 기반으로 새로운 브랜치를 생성

+ `-v`: 브랜치 세부사항을 보여주는 옵션 값
+ `-r`: 원격 저장소의 브랜치 목록을 보여주는 옵션 값
+ `-vv`: 트래킹 브랜치 목록을 보여주는 옵션 값
+ `-d`: 로컬 브랜치 삭제
+ `-D`: 로컬 브랜치 강제 삭제
+ `--merged`: 병합된 브랜치는 `*`로 표시해주는 옵션 값

+ `git branch -u origin/<branch-name>`: 기존 브랜치에 업스트림 직접 연결

## 1️⃣4️⃣ git rev-parse `<branch-name>`
브랜치 생성의 기준점이 된 `커밋 아이디`를 출력하는 명령어

## 1️⃣5️⃣ git checkout
+ `git checkout <branch-name>`: 해당 브랜치로 이동 ( `working` 변경 )
+ `git checkout -- <file-name>`: 해당 파일로 이동 ( 해당 파일만 최신 커밋상태로 되돌려줌 )
+ `git checkout <commit-id>`: 커밋 아이디에 해당하는 커밋으로 이동

+ `git checkout HEAD~<숫자>`: 현재 위치(`HEAD`) 기준 숫자만큼 뒤의 커밋으로 이동
+ `git checkout -`: 제일 최신 커밋으로 이동

+ `git checkout --track origin/<branch-name>`: 원격 저장소의 브랜치를 로컬 저장소의 브랜치로 업스트림하는 명령어 ( 원격 브랜치를 로컬 브랜치로 가져오는 명령어 )
+ `git checkout -b <new-branch-name> origin/<branch-name>`: 원격 저장소의 브랜치를 로컬 저장소의 브랜치로 복사하는 명령어

+ `-b`: 브랜치를 생성과 동시에 해당 브랜치로 이동

## 1️⃣6️⃣ git stash
현재 변경 내용을 스택형태의 임시 저장소에 저장해두는 명령어입니다.<br />
`git add`시 저장되는 `stage`와는 다른 영역입니다.<br />

+ `git stash save "<message>"`: 임시 저장소에 메시지로 기록하는 명령어
+ `git stash list`: 임시 저장소 목록을 보여주는 명령어
+ `git stash show [-p]`: 임시 저장소와 현재의 차이를 보여주는 명령어 ( `-p`는 더 상세하게 보여줌 )
+ `git stash pop`: 임시 저장소의 가장 최근값을 가져오고 성공하면 스택에서 제거하는 명령어
+ `git stash branch <branch-name>`: 브랜치를 새로 생성하고 내용을 채워넣음 ( 성공시 `stash` 제거 )
+ `git stash apply <stash-name>`: 해당 `stash`를 복사하고 스택에서 제거하지 않음

+ `--index`: `stash`에서 불러올경우 `working`과 `stage`를 구분하지 않고 모두 `working`에 넣어서 돌려줌<br />
	해당 옵션을 부여하면 `working`과 `stage`를 구분해서 불러온다.

## 1️⃣7️⃣ git reset `<commit-id> | <HEAD^> | <HEAD~n>`
커밋을 되돌리는 명령어입니다.<br />

사용시 주의해야 할 점은 커밋의 이력 자체를 흔적도 없이 제거하기 때문에 타인과 같이 작업하는 공간에서 사용하면 다른 사람들에게 피해를 줄 수 있습니다.<br />
따라서 웬만하면 원격 저장소에 올리지 않은 커밋을 제거할 때나, 혼자 사용하는 원격 저장소에서만 사용하는 것이 좋습니다.<br />

+ 옵션
	1. `--soft`: 커밋되었던 부분들을 `stage`에 넣음
	2. `--mixed`: 커밋되었던 부분들을 `stage`에 넣지 않음 ( 기본 값 )
	3. `--hard`: 커밋되었던 부분들을 모두 삭제함

## 1️⃣8️⃣ git revert
TODO: 직접 사용해본 후 작성할 예정

## 1️⃣9️⃣ git merge `<branch-name>`
분기점으로 시작한 브랜치에 변화가 없다면 특별한 작업 없이 `Fast Forward`(`--ff`)로 병합됩니다.<br />

만약 `Conflict`가 발생했다면 충돌난 파일에서 `>>> HEAD`, `<<< 브랜치명`으로 변경된 부분을 수정하고 `add`, `commit`을 하면 정상적으로 병합됩니다.<br />
( `git log --oneline --graph -10`과 같은 명령어를 입력하면 병합된 이력을 확인할 수 있습니다. )<br />

+ `-e`: `3-way` 병합 시 병합 메시지를 직접 작성할 수 있게 해주는 옵션 값 ( `--edit` )
+ `--abort`: `conflict` 이후에 실행하면 병합을 취소시켜주는 옵션 값

1. `--ff`: 기본 옵션으로 병합 커밋을 생성하지 않고 커밋을 이어 붙임
2. `--no--ff`: 커밋을 이어 붙이고 병합 커밋을 생성
3. `--squash`: 브랜치에서 했던 커밋에 대한 기록을 남기지 않고 모든 커밋이 합쳐져서 통합할 브랜치의 `stage`에 들어감<br />( 즉, `add` 상태가 되기 때문에 새로운 이름으로 커밋을 해줘야 합니다. )

> 브랜치를 나눈 분기점이후에 기존 브랜치에 변화(`commit`)가 생겼다면 `--ff`를 사용해도 병합 커밋이 생깁니다.
{: .prompt-info}

## 2️⃣0️⃣ git rebase
TODO: 직접 사용한 후 작성할 예정

# 💡 개인적으로 자주 사용하는 개념들
## 0️⃣ add 취소
`git rm --cached <file-name>`

## 1️⃣ commit 취소
커밋을 취소하는 방법은 두 가지가 있습니다.<br />

1. [`git reset`](/posts/Git-GitHub/#1%EF%B8%8F⃣7%EF%B8%8F⃣-git-reset-commit-id--head--headn): 이전에 했던 커밋을 흔적도 남기지 않고 지워버리는 방법입니다.<br />
2. [`git revert`](/posts/Git-GitHub/#1%EF%B8%8F⃣8%EF%B8%8F⃣-git-revert): 이전에 했던 커밋을 지우는 커밋을 추가하면서 커밋을 제거하는 방법입니다.<br />

### 3. 브랜치 제거
`git branch -d <브랜치명>`를 사용하면 `master`와 병합되지 않아서 제거할 수 없다는 문구가 나오면서 제거하지 못합니다.<br />
병합할 목적이 없다면 원격 레포지토리에서도 제거했으니 강제로 로컬 레포지토리에서 제거해도 문제되지 않습니다.<br />

1. `git push origin --delete <브랜치명>`: 원격 저장소 브랜치 제거
2. `git branch -D <브랜치명>`: 로컬 저장소 브랜치 강제 제거
3. `git branch -d <브랜치명>`: 로컬 저장소 브랜치 제거 ( 커밋이력이 있는데 병합하지 않았다면 제거안됨 )

# 👀 팁
## 0️⃣ 커밋 아이디
`SHA1`이라는 해시 알고리즘을 사용하기 때문에 중복되지 않고 앞 7자리정도만 사용해도 겹치지 않기 때문에 굳이 전부를 복사해서 사용할 필요 없습니다.

## 1️⃣ 커밋과 푸시 순서
깃의 커밋은 이전 커밋을 기반으로 새로운 커밋을 만드는 것이기 때문에 반드시 최신 커밋을 기반으로 커밋을 해야 합니다.
공동 개발일 경우에 `push`를 할 때도 동시에 작업하고 각자의 작업 내용을 커밋 하게 되면 이후에 `push`하는 개발자는 최신 커밋을 기반으로 `push`하는 것이 아니기 때문에 오류가 나게 됩니다.

따라서 `push`전에는 `pull` or `fetch` + `merge`를 이용해서 최신 커밋을 적용해야 합니다.
가장 이상적인 방법은 `branch`를 나눠서 작업하고 `pull request`로 병합하는 방법을 사용하는 것입니다.<br />

## 2️⃣ 브랜치
브랜치를 만드는 것은 실제 코드를 복사하는 것이 아닌 시작 위치에 가상 폴더를 만드는 것입니다.<br />

+ 브랜치를 만드는 기준은 브랜치 or 커밋을 기준으로 새로운 브랜치를 만듭니다.

+ 브랜치를 생성한다고 즉시 새로운 브랜치가 생성되는 것은 아닙니다.<br />
	최초에는 포인터만 생성되고 이후에 생성한 브랜치에서 커밋을 하면 새로운 브랜치가 생성되는 것입니다.
    
+ 브랜치를 삭제할 때 조건은 현재 브랜치가 아니고, 수정사항이 없어야 합니다.<br />
	이미 커밋을 한 브랜치라면 `merge`를 해줘야 `-d`로 삭제가 가능합니다.

## 3️⃣ checkout
`checkout`시에는 `HEAD` 포인터가 같이 이동됩니다.<br />
`checkout`전에는 `working`을 정리하고 이동해야 한다.<br />

브랜치 이동 자체가 `working`을 바꾸는 행위이기 때문에 기존에 바꿨던 상태가 유지될 수 없습니다. 따라서 `commit`, `stash` 같은 행위로 기존 상태를 저장해두고 이동해야 합니다.

## 4️⃣ HEAD
`HEAD`는 기본적으로 현재 선택된 브랜치의 최신 커밋을 가리키는 포인터입니다.<br />
커밋, 브랜치 생성 등의 커밋을 이용해서 새로운 결과물을 생성하는데 사용합니다.<br />
( 즉시 접근이 가능하므로 성능적으로 유리 )<br />

+ `HEAD^`: 현재 `HEAD`를 기준으로 최신 커밋을 가리킴
+ `HEAD~n`: 현재 `HEAD`를 기준으로 `n`번째 이전의 커밋을 가리킴

## 5️⃣ 업스트림 트래킹
로컬 브랜치와 원격 저장소의 브랜치와의 매칭을 **업스트림 트래킹**이라고 합니다.<br />

**트래킹 브랜치**는 원격 저장소 브랜치와 로컬 브랜치를 연결해 주는 중간 다리 역할을 합니다.<br />

원격 저장소를 클론 한다고 해서 모든 브랜치까지 클론 되지 않습니다.<br />
효율성의 이유로 원하는 브랜치는 직접 업스트림을 해줘서 로컬과 원격 브랜치를 연결시켜줘야 합니다.<br />

## 6️⃣ 병합
브랜치단위로 병합을 실행합니다.<br />

병합은 `Fast-Forward` 병합과 `3-way` 병합이 존재합니다.<br />
병합은 실행하는 브랜치에 결과물이 생깁니다.<br />

### 1. Fast-Forward 병합
브랜치가 일직선으로 하나의 줄기로 이어진 상태에서 병합할 경우 적용하는 방식

### 2. 3-way 병합
브랜치1, 브랜치2, 두 개의 브랜치의 공통된 시작커밋 3가지가 합쳐지는 병합에 사용하는 방식<br />
`conflict`이 없을 경우에는 자동으로 `merge` 커밋 로그가 남습니다.( `--ff` )<br />
`conflict`이 있을 경우에는 직접 충돌난 부분을 수정한 후 커밋해줘야 합니다.( `--no-ff` )<br />

만약 `conflict` 발생한 경우에는 `VSCode`로 들어가서 확인해 보면 쉽게 충돌 부분을 수정할 수 있도록 도와줍니다.<br />

# 😶 추가할 것
TODO:
1. ssh<br />
1. upstream<br />

# 📮 레퍼런스
1. « Git 교과서 » ( 이호진 지음, 길벗, 2020 )