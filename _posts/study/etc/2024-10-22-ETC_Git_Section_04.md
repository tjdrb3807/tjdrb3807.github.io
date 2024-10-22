---
layout: post
title: "Git #Section04 - Git Settings"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section04.png
categories: [study, etc]
tags: [etc]
description: 본 글에서는 Local 환경에서 Git을 설정하고 관리하는 방법을 다룬다.
---

* toc
{:toc}

## ✍🏻 Git 최초 설정
본격적으로 Git이 관리하는 프로젝트를 생성하기에 앞서 몇가지 설정사항이 필요하다.

### 프로필 설정
Git을 사용하는 이유는 협업과정에서 팀원간 작업 내용을 공유하고 원활한 프로젝트를 진행하기 위함이다. 이를 위해 프로필을 설정해주어야 한다. 위 작업을 통해 누군가 작성한 commit 내역에서 어떤 팀원이 작업했는지, 또는 이슈가 발생했다면 누구에게 연락해야 하는지 확인하기 위해 작성하는 컨벤션이다.   
기본적인 프로필 설정에는 이름(name)과 이메일(email)이 있으며, `config` 명령어를 통해 설정할 수 있다. 

* 이름 설정
```Git
git config --global user.name "본인 이름"
```
* 이메일 설정
```Git
git config --global user.email "본인 이메일"
```
위 코드에서 `--global` 옵션은 사용자의 컴퓨터 전반적으로 Git의 설정이 세팅되도록 지원한다.
> 프로젝트마다 다르게 설정하는 법 추가 필요

두 개의 명령어를 통해 이름과 이메일을 설정했다면 현재 설정된 내역을 아래 명령어를 통해 확인할 수 있다.
```Git
git config --global user.name
```
```Git
git config --global user.email
```

### 기본 브랜치명 설정
구 버전의 Git을 사용하는 경우 기본으로 설정된 브랜치 명이 master로 확인될 것이다. 하지만 이는 흑인 노예들과 그들의 주인을 연상시키는 단어라고 판단되어 최근에는 다른 용어들로 대체하는 분위기다. Git의 경우도 기본 브랜치명을 `main` 또는 `trunk` 등으로 변경할 것을 권유하며, 자신의 메인 브랜치명이 mater라면 아래 명령어를 통해 `main`으로 설정하도록 한다. 

```Git
git config --global init.defaultBranch main
```

### 설정 확인
위에서 언급한 두 가지 설정을 완료했다면 현재 자신의 컴퓨터에 설정된 모든 리스트를 아래 명령어를 통해 확인할 수 있다. 
```Git
git config --list
```

<br>
<br>

## ✍🏻 프로젝트 생성 & Git 관리 시작
Git이 관리하는 프로젝트를 생성하기 위해 원하는 위치에 빈 폴더 하나를 생성하고 해당 폴터 위치에서 아래 명령어를 실행한다. 
```Git
git init
```

해당 명령어는 __.git__ 이라는 하위 디렉토리를 만들며, 이는 저장소에 필요한 뼈대(Skeleton)가 들어있다. 
> 맥에서 숨긴 파일 보는법: command + shift + .

하지만 __git init__ 명령어를 입력해도 현재 프로젝는 Git의 어떤한 관리도 하지 않는 상태이다.

![Full-width image](/assets/img/blog/etc/git_section04_image01.png){:.lead width="800" height="100" loading="lazy"}

git init 명령어 이후 git status 명령어 입력 상태
{:.figcaption}

### 첫 commit 과정

Git이 프로젝트 파일의 버전을 관리하도록 하려면 저장소에 파일을 추가하고 한 번의 commit 과정이 있어야한다.  


파일을 생성해서 저장을 하면 Git에 관리를 받은적 없는 파일의 내역들은 __Working directory__ 에 __Untrackted__ 상태로 존재한다.

![Full-width image](/assets/img/blog/etc/git_section04_image02.png){:.lead width="800" height="100" loading="lazy"}

T1.yaml에서 작업한 내역은 Git이 한 번도 관리한적 없는 Untrackted 상태
{:.figcaption}

해당 내역을 아래 명령어를 통해 __Staging area__ 로 이동시킨다. 
```git
git add (Staging area로 이동할 파일의이름과 확장자)
```
> . 옵션을 사용하면 현재 Working directory에 위치한 모든 내역을 선택할 수 있다.

![Full-width image](/assets/img/blog/etc/git_section04_image03.png){:.lead width="800" height="100" loading="lazy"}

git add 명령어를 통해 Staging area로 이동된 내역
{:.figcaption}

마지막으로 Staging area에 Tracked 상태의 내역을 아래 명령어를 통해 Repository로 보내면 첫 commit이 완료되며 Git이 프로젝트 폴더를 관리하기 시작한다.
```Git
git commit -m "commit message"
```
> 디테일한 커밋 메세지를 작성하고 싶으면 git commit 명령어를 통해 vim에서 작성

정상적으로 커밋이 완료됐다면 아래 명령어를 통해 commit 내역을 확인할 수 있다.
```Git
git log
```

![Full-width image](/assets/img/blog/etc/git_section04_image04.png){:.lead width="800" height="100" loading="lazy"}

git log 명령어 실행 결과
{:.figcaption}





<br>

## ✍🏻 Git에게 맡기지 않는 것들
Git을 통해 프로젝트를 관리하면서 특정 파일이나 폴터를 배제해야 하는 경우가 발생한다.    

대표적인 예시로 자동으로 생성 또는 다운로드되는 파일(빌드 결과물, 라이브러리), 또는 보안상 민감한 정보를 담은 파일, 서버의 주요 AccessToken이 담긴 파일 등을 Git 내역에서 관리하게 되면 GitHub를 통해 다른 사용자에게 노출될 우려가 있다. 

위와같은 문제가 발생하지 않도록 `.gitignore` 파일을 사용해 배제할 요소들을 지정할 수 있다.
### .gitignore 형식
```
# 모든 file.c
file.c

# 최상위 폴더의 file.c
/file.c

# 모든 .c 확장자 파일
*.c

# .c 확장자지만 무시하지 않을 파일
!not_ignore_this.c

# logs란 이름의 파일 또는 폴더와 그 내용들
logs

# logs란 이름의 폴더와 그 내용들
logs/

# logs 폴더 바로 안의 debug.log와 .c 파일들
logs/debug.log
logs/*.c

# logs 폴더 바로 안, 또는 그 안의 다른 폴더(들) 안의 debug.log
logs/**/debug.log
```
