---
layout: post
title: "Git #Section03 - Basic of Git"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section02.png
categories: [study, etc]
tags: [etc]
description: 본 글에서는 Git이 다른 VCS들과의 차이점에 초점을 맞춰 Git의 기초에 대해 알아도보록 한다.
---

* toc
{:toc}

## ✍🏻 데이터를 다루는 방식
Subversion이나 CVS, Bazaar 등의 VCS와 Git의 가장 큰 차이점은 데이터를 다루는 방법에 있다. 큰 틀에서 봤을 때 대부분의 VCS이 관리하는 정보는 파일들의 목록이다. Git을 제외한 대부분의 VCS은 파일의 변화를 Delta 방식인 시간순으로 관리하는 반면에 Git을 스트림과 같은 Snapshot 방식을 채택하여 데이터를 다룬다.

<br>

### 델타(Delta) 기반 버전관리 시스템
델타 방식은 Subversion이나 Perforce 등 에서 버전을 관리하는 방식이다.    

델타 방식은 각 파일이 생겨난 Version에 해당 파일 전체가 저장되고, 이후 Version에서 해당 파일의 수정이 가해질 때 그 변경점들이 저장되는 방식이다. 

![Full-width image](/assets/img/blog/etc/git_section03_image01.png){:.lead width="800" height="100" loading="lazy"}

각 파일에 대한 변화를 저장하는 시스템.
{:.figcaption}

위 그림은 시간의 경과에 따른 Delta 방식을 모식도 한것이며, Version05 시점에 FileC의 내용은 Version01의 
원본으로 부터 ∆1, ∆2변화 그리고 ∆3 변화가 누적된 것으로 계산된다.

<br>

### 스냅샷(Snapshot) 기반 버전관리 시스템
스냅샷 기반 버전관리 시스템은 Git이 채택한 방식이며, 시간순으로 관리하는 델타 방식과 달리 이전 Version에서 파일의 변경 사항이 없다면 현재 Version에서는 이전 Version의 파일에 대한 링크만 저장해둔다.   


이러한 매커니즘으로 새로운 Version이 생성될때 마다 해당 Version에서 각 파일의 최종 상태 그대로 저장되며, 내역들 전반도 용량을 크게 차지하지 않는 효율적인 방식으로 관리된다.

![Full-width image](/assets/img/blog/etc/git_section03_image02.png){:.lead width="800" height="100" loading="lazy"}

Version 순으로 프로젝트의 스냅샷을 저장.
{:.figcaption}

위 그림은 스냅샷 방식을 Version 경과에 따라 모식도한 것이며, Version 5의 경우 File A에는 변화가 없으므로 Version 4의 File A를 그대로 연결해서 가져오고, 변화가 있는 File B와 FileC 는 각각의 최종 파일 내용이 그대로 저장된다.

<br>

### 스냅샷 VS 델타
만약 프로젝트의 Commit 내역이 많은 Repository를 델타 방식으로 다루게 되면, Git에서 Branch를 바꾸는 작업을 할 때마다 각 파일들의 처음 생성 시점부터 변경 사항들을 더해서 현재 내용을 계산해내야 한다. 위 방법은 히스토리가 길어질수록 속도 이슈가 발생할 수 있다.   

반면에 스냅샷 방식을 채택하면 현 시점의 각 파일들이 pull 명령어를 통해 최송 상태로 저장되어있으니 빠르게 처리할 수 있는 장점이 있다.    

이러한 이유로 많은 사용자들이 스냅샷 기반 버전관리 시스템을 채택한 Git을 사용한다.

<br>
<br>

## ✍🏻 Git이 채택한 버전 관리

### 중앙집중식 버전 관리(CVCS)
Subversion이나 CVS는 중앙 집중식 버전 관리를 채택하고 있으며, 이는 원격 서버의 모든 관리 내역들이 저장된다.   
Subversion이나 CVS에 참여하는 인원들의 컴퓨터 즉 로컬에는 중앙에 저장된 현 Version으로 다운받은 파일들로만 작업을 할 수 있다. 이는 원격 저장소에 의존적이며, 만약 네트워크 이슈가 발생하면 로컬에서 할 수 있는 작업이 제한된다.

### 분산 버전 관리(DVCS)
Git의 경우 원격에 있는 내역을 다운로드 하는 것이 아니라 clone 명령어를 통해 받아온다. 이때 로컬에 파일들 뿐만 아니라 전체 Git 커밋이랑 브랜치들까지 받아지므로 네트워크 이슈와 상관 없이 로컬에서 자유롭게 작업할 수 있다.

모든 구성원이 Git의 상태까지 공유하기 때문에 각자의 작업을 하다 원할때 프로젝트를 push와 pull로 동기화하면서 협업을 진행할 수 있다.