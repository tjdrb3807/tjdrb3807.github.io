---
layout: post
title: "FeelTalk - MVVM 설계 전략 (Input–Output)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

![Thumbanil](/assets/img/blog/ios/feelTalk__MVVM 설계 전략_thumbnail.png)

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에서 MVVM 패턴과 RxSwift를 적용하며
ViewController의 책임을 어떻게 재정의했고,
ViewModel을 어떤 기준으로 설계했는지를 정의한 기술 기록입니다.

이전 글에서는 UIKit 기반 MVC 구조에서 프로젝트 규모가 커지면서 발생했던
Massive ViewController 문제와,
이를 구조적으로 해결하기 위해 MVVM 패턴을 선택하게 된 배경을 다뤘습니다.

이번 글에서는 그 연장선상에서,
- MVVM에서 View와 ViewModel의 역할을 어떻게 분리했는지
- RxSwift를 통해 View-ViewModel을 어떤 방식으로 연결했는지
- Input-Output 패턴을 적용해 상태 흐름을 어떻게 설계했는지
- 그 결과 ViewController의 역할이 어떻게 변화했는지

를 중심으로 정리하고자 합니다.

## MVVM에서의 역할 분리
![MVVM 다이어그램](/assets/img/blog/ios/feelTalk_MVVM_diagram.png)

FeelTalk 프로젝트에서 MVVM은 ViewController의 책임을 명확히 제한하기 위한 기준으로 사용되었습니다.

- **View(ViewController)**
  - UI 구성 및 화면 랜더링(상태에 따른 UI 반영)
  - 사용자 입력을 이벤트 형태로 전달
  - ViewModel이 제공하는 상태를 바인딩하여 UI에 반영

- **ViewModel**
  - View로부터 전달받은 사용자 이벤트 처리
  - 화면에 필요한 상태 관리
  - 상태를 Observable 스트림 형태로 제공

이 구조에서 ViewController는 애플리케이션의 상태를 직접 보관하거나,
이벤트에 따라 어떤 동작을 수행할지 판단하는 역할을 맡지 않습니다.

대신 ViewController는 ViewModel이 생성한 상태를 구독하고, 그 상태를 화면에 표현하는 역할만 수행합니다.

즉, **"어떤 상태가 필요한가"는 ViewModel이 책임이고, "그 상태를 어떻게 보여줄 것인가"는 ViewController의 책임**으로 명확히 분리했습니다.


