---
layout: post
title: "2025-02-03 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## Extension에 대해 설명해 주세요.

Extension은 기존 클래스, 구조체, 열거형, 프로토콜에 새로운 기능을 추가할 수 있도록 하는 기능입니다. 원본 소스를 수정하지 않고도 기능을 확장할 수 있어 코드의 유지보수성과 재사용성을 높일 수 있습니다.

## Extension을 사용해보셨나요? 사용해본 경험이 있다면, 어떤 상황에서 어떻게 활용했는지 설명해 주세요.

Extension을 활용하여 기존 타입에 기능을 추가한 경험이 많습니다. 특히, RxSwift를 사용할 때 UIViewController의 라이프사이클 메서드를 Observable로 변환하는 Reactive Extension을 구현한 적이 있습니다. 이를 위해 메서드 스위즐링을 활용하여 viewDidAppear() 등의 이벤트를 Rx 스트림으로 변환했고, 이를 통해 ViewController에서 라이프사이클 이벤트를 더 직관적으로 구독하고 활용할 수 있도록 개선했습니다.


