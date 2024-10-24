---
layout: post
title: "Git #Section07 - Merge and Rebase"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, etc]
tags: [etc]
description: 본 글에서는 Git이 각각의 Branch를 합치는 두 가지 방법에 대해 다룬다.
---

* toc
{:toc}

## ✍🏻 Merge
__Merge__ 는 병합이란 말 뜻처럼 두 개의 Branch를 이어 붙이는 기능이다. 두 개의 Branch를 Merge하는 과정에서 Commit 하나가 생성되는 특징이 있으며, 새로 생성된 Commit에는 원래 Branch랑 명합될 모든 변경사항들이 담겨있다.

merge를 사용하려면 브랜치가 합쳐서 주 대상이 될 브랜치에서 작업한다.   
```Swift
git merge (branch name)
```
merge는 하나의 새로운 commit을 생성하므로, 이를 reset으로 되돌리는 것이 가능하며, 병합된 branch는 삭제된다.

## ✍🏻 Rebase
__Rebase__ 는 Branch를 대상 Branch로 옮겨붙이는 기능이다. Rebase를 사용하면 대상 Branch에 지정한 Branch의 Commit 내역들을 추가한 결과를 갖게된다.

## ✍🏻 Merge VS Rebase
rebase를 한뒤 히스토리는 한 개의 브랜치로 정리되지만, merge는 브랜치의 흔적을 남긴다. 프로젝트에서 브랜치가 몇개 안된다면 문제 없지만, 많은 브랜치가 사용되는 프로젝트에서는 프로젝트의 진행 내역들을 보고 파악하기 복잡하다.   따라서 진행하는 프로젝트의 성격에 따라 브랜치의 사용 내역을 남겨둘 필요가 있다면 merge를 히스토리를 깔끔하게 만드는게 중요하다면 rebase를 채택한다.
> Tip. 팀원간에 공유된 Commit들에 대해서는 rebase를 사용하지 않는것을 권장한다.

rebase를 사용하려면 merge와는 반대로 대상 브랜치에서 다음 명령어를 실행한다.   
```Git
git rebase (branch name)
```

이때 대상 브랜치와 branch name 브랜치의 commit 위치가 다른것을 확인할 수 있다.   
branch name 브랜치로 switch 해보면 대상 브랜치에서 작업한 내용이 없는것을 확인할 수 있다.
현 상황은 대상 브랜치에서 작업한 내용은 최신화 됐는데, branch name 브랜치는 대상 브랜치가 rebase 한다고 해서 branch name의 끝 commit에 위치하지 않는다.
따라서 branch name 의 commit을 rebase한 대상 브랜치의 commit으로 옮겨주는 작업이 필요하다.
