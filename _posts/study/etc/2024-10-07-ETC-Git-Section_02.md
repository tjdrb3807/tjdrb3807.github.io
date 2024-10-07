---
layout: post
title: "Git #Section02 - The three states of Git"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section02.png
categories: [study, etc]
tags: [etc]
---

* toc
{:toc}

## 🤚 Introduce
Git이 파일들의 변경사항을 버전별로 관리하기 위해서 파일들을 어떤 식으로 인식하고, 어떠한 단계로 관리하는지를 이해한다.

## ✍🏻 Git의 세 가지 상태
![Full-width image](/assets/img/blog/etc/git_section02_image01.png){:.lead width="800" height="100" loading="lazy"}

Git의 3가지 상태 
{:.figcaption}
### Working directory
* 새로운 파일 추가, 기존 파일의 변경 또는 삭제로 인한 수정내역이 위치하는 공간
* Untracked
  * .gitignore에 추가돼서 무시되는 경우
  * 새로 생성된 파일(add 된 적 없는 경우)
    * git status 명령어를 통해 Untracked files 확인 가능
* Tracked
  * Repository의 수정 내역
### Staging area
* Working directory에서 add 명령어를 통해 수정 내역이 담기는 공간
* commit해서 Repository에 들어가기 전 준비상태
### Repository
* 이미 어떠한 Version안에 들어있는 상태
* GitHub에서 생성한 저장소를 Repository라 칭하는데, 이는 저장소에는 이미 Commit된 내역들이 저장되기 때문이다.
* Commit들이 저장되는 공간

