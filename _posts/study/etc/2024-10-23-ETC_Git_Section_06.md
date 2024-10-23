---
layout: post
title: "Git #Section06 - Whit is Branch?"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section06.png
categories: [study, etc]
tags: [etc]
description: 본 글에서는 Git이 Branch를 동작시키는 개념과 Branch 생성/이동/삭제에 대해 다룬다.
---

* toc
{:toc}

## ✍🏻 브랜치란 무엇인가
프로젝트를 진행하다 보면 코드를 여러 개로 복사하는 경우가 빈번하게 발생한다. 코드를 통째로 복사하고 나서 원래 코드와는 상관없이 __독립적으로__ 개발을 진행할 수 있는데, 이렇게 독립적인 개발을 돕는 기능이 Branch 이다.

Git의 Branch는 매우 가벼우며, 순식간에 Brnach를 새로 만들고 Brnach 사이를 이동할 수 있다. 다른 VCS과는 달리 Git은 Branch를 만들어 작업하고 나중에 Merge 하는 방법을 권장한다. 또한 다량의 Brnach를 생성하고 이동해도 지장이 없다.

## ✍🏻 Git이 브랜치를 다루는 원리

Git은 데이터를 [Snapshot](https://tjdrb3807.github.io/study/etc/2024-10-22-ETC_Git_Section_03/) 으로 기록한다. Commit을 하면 Git은 현재 Stage Area에 있는 데이터의 스냅샷에 대한 포인터, Commit Message와 같은 메타데이터, 이전 Commit에 대한 포인터 등을 포함한 Commit Object를 저장한다. 이전 __Commit 포인터가 있어서 현재 Commit이 무엇을 기준으로 바뀌었는지 알 수 있다.__ 최초의 Commit을 제외한 나머지 Commit은 이전 Commit 포인터가 적어도 하나씩 있고 Branch를 합친 Merge Commit 같은 경우에는 이전 Commit 포인터가 여러 개 있다.

Git의 Branch는 Commit 사이를 이동할 수 있는 포인터 같은 것이며, 기본적으로 Git은 Main Branch를 만든다. 처음 Commit하면 이 Main Branch가 생성된 Commit을 가리킨다. 이후 Commit을 만들면 Main Branch는 자동으로 가장 마지막 Commit을 가리킨다.   

<br>

## ✍🏻 새 브랜치 생성 / 이동 / 삭제

### 브랜치 생성
```Git
git branch (Branch Name)
```

특정 Version에서 생성한 Branch들 각각에서 새로운 Commit 작업이 진행돼야 새로운 가지를 뻗는다. 다양한 Branch에서 작업을 수행하고 Branch들을 이동해보면 프로젝트 하나의 폴더에서 Branch를 어디로 Switch 하느냐에 따라 그 안의 내용들이 바뀌는 것을 확인할 수 있다.

### 브랜치 이동
```Git
git switfh (Brnach Name)
```

### 브랜치 생성과 동시에 이동
```Git
git switch -c (Branch Name)
```
'*' 표시가 있는 Branch 가 현재 위치한 Branch 이다.
{:.figcaption}

### 브랜치 삭제
```Git
git branch -d (Branch Name)
```
삭제할 Branch에 다른 Branch로 적용되지 않은 Commit 내역이 있다면 '-D' 옵션을 통해 강제 삭제 가능. 
{:.figcaption}

### 브랜치 이름 변경
```Git
git brnach -m (Current Branch Name) (New Branch Name)
```

### CLI 에서 여러 브랜치의 내역 확인
```Git
git log --all --decorate --oneline --graph
```