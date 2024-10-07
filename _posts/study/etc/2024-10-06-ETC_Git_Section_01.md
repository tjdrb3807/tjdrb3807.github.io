---
layout: post
title: "Git #Section01 - What is Git?"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section01.png
categories: [study, etc]
tags: [etc]
---
* toc
{:toc}

## 🤚 Introduce
프로젝트를 진행하다 보면 새로운 기능을 추가하거나 오류를 수정하는 등의 성능 개선 과정을 통해 다양한 Version이 만들어진다.    
이때 특정 Version에서 추가한 기능에 결함이 발생하여 이전 Version으로 프로젝트를 되돌려야 한다면? 
또는 초기 기획과 무관한 아이디어가 떠올라 앱에 적용해 보고 싶은데 메인 프로젝트 폴더에서 작업하지 않고 다른 공간에서 하고 싶다면?   

프로젝트를 압축해서 백업해두면 위에서 언급한 두 가지 이슈에 대처할 수 있지만, 프로젝트가 진행될수록 차지하는 용량이 비대해지는
불편함이 발생한다.    

__Git__ 은 프로젝트의 버전을 언제든 되돌릴 수 있고, 여러 차원(Branch)를 넘다는 것처럼 프로젝트의 내용들을 마치 다른 폴더인 것처럼 
여러 모드로 자유롭게 전환하고 변경 사항들을 쉽게 이동할 수 있다.

## ✍🏻 What is VCS?
VCS(Version Control System)은 파일 변화를 시간에 따라 기록하고 특정 시점의 Version을 다시 꺼내올 수 있는 시스템이다.

VCS를 사용하면 프로젝트 통째, 혹은 각 파일을 이전 Version으로 되돌릴 수 있고, 시간에 따라 수정 내용을 비교할 수 있다.    
또한, 협업 프로젝트에서 누가 이슈를 발생시켰는지 추척 및 언제 만들어진 이슈인지도 알 수 있다. 