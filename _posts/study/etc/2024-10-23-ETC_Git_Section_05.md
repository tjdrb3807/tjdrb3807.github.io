---
layout: post
title: "Git #Section05 - Resset and Revert"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section05.png
categories: [study, etc]
tags: [etc]
description: 본 글에서는 Git에 작성된 히스토리에서 특정 시점(Version)으로 되돌리는 방법에 대해 알아본다.
---

* toc
{:toc}

## ✍🏻 특정 Version으로 되돌릴 수 있는 두 가지 방법
Git은 시간순으로 저장된 프로젝트의 Commit 히스토리를 특정 시점으로 되돌리는 기능을 지원한다.   
`git reset`과 `git revert` 명령어를 통해 해당 작업이 이루어지며, 통상적으로 두 기능을 특정 커밋으로 되돌리는 기능을 지원하지만 명백한 차이가 존재한다.    

### git reset
`git reset` 명령어는 특정 Commit 시점으로 Local 저장소를 되돌리는 기능을 지원한다. 해당 명령어가 수행되면 특정 시점 이후에 진행된 히슽토리 내역은 전부 사라짐과 동시에 특정 시점으로 되돌아온다.
```Git
git reset --hard (Commit Hash ID)
```
CLI 작업공간에서 git log 명령어를 통해 원하는 시점의 Commit Hash ID를 얻을 수 있다.
{:.figcaption}

### git revert
`git revert` 명령어는 `git reset`과 달리 특정 Commit 시점 이후 진행된 히스토리내역을 삭제하지 않고 특정 Commit시점 이후의 히스토리 변화를 거꾸로 수행한다.    
과거로 되돌린 작업을 히스토리 내역에 남기고 싶을 때 `git revert` 방식을 채택한다.    
또한, 협업 과정에서 원격 Repository에 공유된 Commit 내역을 reset 하면 원격 Repository에 등록된 내역을 기반으로 작업하던 다른 팀원들과 출돌을 일으킬 수 있으므로 __한 번 공유된 코드는 revert 명령어를 이용해서 되돌려야 한다.__

```Git
git revert (Commit Hash ID)..(Last Commit Hash ID)
```
다양한 Commit을 되돌리고 싶으면 '..' 으로 연결해서 사용 가능
{:.figcaption}