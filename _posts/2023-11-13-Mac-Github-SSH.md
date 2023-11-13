---
title:  "여러 개의 Github 계정 등록하기(Mac)"
excerpt: "Mac, Macbook 에서 여러 개의 Github 계정 등록하기"

categories:
  - blog
tags:
  - github
  - ssh
last_modified_at: 2023-11-13T16:00:00-05:00

toc: true
toc_sticky: true
---

하나의 환경에서 회사 깃헙 계정에 더하여 개인 깃헙 계정을 사용해야 하는 경우처럼 여러 개의 깃헙 계정을 사용해야 하는 경우가 많습니다.

이러한 경우처럼 GitHub을 사용하면서 여러 개의 계정을 관리해야 하는 경우, 각 계정에 대한 별도의 SSH 키를 설정하는 방법을 설명합니다. 또한, 레포지토리마다 다른 계정으로 설정하는 방법을 알아보겠습니다.
## SSH 키 생성
먼저 맥의 터미널을 열고, SSH 기본 폴더로 이동합니다. 폴더가 없다면 생성합니다.
```bash
$ mkdir ~/.ssh
$ cd ~/.ssh
```

다음 명령어로 새로운 SSH 키를 생성합니다. `ssh-keygen -t [암호화 방식] -b [암호키 비트수] ... -C [코멘트]`
```bash
$ ssh-keygen -t ed25519 -C "company@example.com"
```
코멘트 파라미터는 보통의 경우 각자의 이메일 계정으로 명령어를 실행합니다. ed25519 는 암호화 방식 중에 하나 이며, **dsa** | **ecdsa** | **ecdsa-sk** | **ed25519** | **ed25519-sk** | **rsa** 의 옵션 중 선택이 가능합니다. 자세한 내용은 `man ssh-keygen` 명령어로 확인 가능합니다.

위의 명령어를 실행하면, 생성할 파일의 이름을 입력하라는 메시지가 나오며, 이때 등록하려는 계정에 맞게 본인이 알아볼수 있는 이름으로 설정합니다. 빈칸으로 입력하면 괄호 안의 기본값으로 설정됩니다.
```
Enter file in which to save the key (/Users/cocoslime/.ssh/id_ed25519):id_ed25519_company
```
파일 이름을 입력하고 Enter를 치면 비밀번호를 입력하라는 메시지가 나옵니다. 보통의 경우는  따로 비밀번호를 설정하지 않기에, 그냥 엔터를 치고 넘어갔습니다.
```
Enter passphrase (empty for no passphrase): // Enter
Enter same passphrase again: // Enter again
```
여러 줄의 메시지가 나오고 나면, ls 명령어로 공개키와 개인키 쌍이 생성되었는지 확인합니다.
```bash
$ ls
id_ed25519_company // private key
id_ed25519_company.pub // public key
```
여러 개의 계정에 대한 SSH 키를 같은 과정을 통해 생성합니다.
```bash
// id_ed25519_dongmen94 라는 이름으로 개인용 키를 만듭니다.
$ ssh-keygen -t ed25519 -C "dongmen94@gmail.com"
```
## SSH 키 등록
생성한 SSH 키를 현재 장치에 등록합니다. `ssh-add [ssh 키 경로]`
위의 챕터에서 생성한 두 개의 SSH private 키를 다음과 같은 명렁어를 통해 등록합니다.
```bash
$ ssh-add ~/.ssh/id_ed25519_company
$ ssh-add ~/.ssh/id_ed25519_dongmen94
```
## SSH config 파일 생성 및 설정
`~/.ssh/config` 파일을 열어 각 계정에 대한 호스트 및 키 파일을 설정합니다. `nano` 혹은 `vi` 를 사용합니다.
```bash
$ nano ~/.ssh/config
```

다음과 같이 설정합니다.
```
# 깃헙계정 - 회사
# ===============
Host github.com-company // 호스트 별명
  HostName github.com 
  User git 
  IdentityFile ~/.ssh/id_ed25519_company // SSH 개인키 위치
  
# 깃헙계정 - 개인
# ===============
Host github.com-dongmen 
  HostName github.com 
  User git 
  IdentityFile ~/.ssh/id_ed25519_dongmen94
```
별명은 자주 쓸 이름이므로 기억에 남고, 잘 구분되는 이름으로 설정합니다. 개인키 위치는 위에서 만든 경로로 설정합니다.

## Github 계정에 SSH key 추가
이제 위에서 생성하고 로컬에 설정한 SSH key 를 각 키에 맞는 Github 계정에 등록합니다.
먼저, SSH 공개키를 클립보드에 복사해놓습니다. 다음 명령어 실행으로 가능합니다.
```bash
$ pcopy < ~/.ssh/id_ed25519_dongmen94.pub
```

깃헙에 로그인 후 계정 설정 > SSH and GPG keys 에서 New SSH key 에 진입합니다.
아래와 같이 New SSH Key 에 복사한 공개키 값을 입력합니다. Title 에는 해당 키를 등록한 장치를 구분할 수 있도록, 이름을 정해서 입력합니다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2023-11-13/20231112221706.png" alt="qry" width="500" >
</p>

## SSH 키 테스트
설정이 완료되었으면 다음 명령어로 각 계정에 대한 연결을 테스트합니다.
```bash
$ ssh -T git@github.com-company
$ ssh -T git@github.com-dongmen
```
모든 단계를 마치면 각 GitHub 계정을 구분하여 사용할 수 있습니다. 이제 여러 개의 계정으로 프로젝트를 클론하거나 푸시할 때, 해당 계정의 호스트 설정을 이용하여 사용하면 됩니다.

이제 여러분은 간편하게 여러 개의 GitHub 계정을 관리할 수 있습니다. 추가로 필요한 경우 위의 단계를 반복하여 계정을 추가할 수 있습니다.
## SSH 키 사용
설정한 SSH 키는 다양한 용도로 사용할 수 있습니다. 기존에 호스트를 입력하는 곳에 호스트 별명을 대신 입력하면 됩니다.
예를 들어, work 회사의 Github 계정으로 회사 Github 의 private repository 를 clone 해야 하는 경우 다음과 같이 입력 가능합니다.
```bash
$ git clone git@github.com-company:work/private-repo.git
```
