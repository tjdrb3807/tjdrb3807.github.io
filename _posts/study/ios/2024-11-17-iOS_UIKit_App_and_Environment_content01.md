---
layout: post
title: "iOS #UIKit - Life cycle"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, ios]
tags: [ios]
description: App이 Foreground 또는 Background 상태에 있을 때 시스템 Notification에 응답하고 기타 중요 시스템관련 이벤트를 처리한다.
---

* toc
{:toc}

## Overview 
현재 App의 상태에 따라 할 수 있는 작업과 할 수 없는 작업이 결정된다. 

* Foreground 상태는 사용자의 주의를 끌기 때문에 CPU를 포함한 시스템 리소스보다 우선순위가 높다.
* Background 상태의 앱은 Off-Screen 이므로 가능한 한 적은 작업만 수행하거나, 아무런 작업도 수행하지 않는 것이 좋다.    

App의 상태가 변경되면 그에 맞는 동작을 조정해야 한다.   
App의 상태가 변경되면 UIKit은 적절한 Delegate 객체의 호출 방법을 통해 사용자에게 알린다.

* iOS13 이상 버전에서는 UISceneDelegate 객체를 사용해 Life-cycle 이벤트에 응답한다.
* iOS12 이전 버전에서는 UIApplicationDelegate 객체를 사용해 Life-cycle 이벤트에 응답한다.

## Scnen
App에서 Scene을 지원하는 경우(iOS13 이상) UIKit은 각 Scene에 대한 별도의 Life-cycle 이벤트를 제공한다.    
Scene은 디바이스에서 실행되는 App의 UI 인스턴스 중 하나를 나타낸다.     
사용자는 각 App에 여러 Scene을 생성하고 이를 별도로 보여주거나 숨길 수 있다.    
각 Scene에서는 고유한 Life-cycle가 있으므로 각 Scene은 서로 다은 실행 상태에 있을 수 있다.    
예를 들어, 한 Scene은 Foreground에 있고 다른 Scene은 Background에 있거나 정지되어 있을 수 있다.    

> Scene 지원은 옵트인 기능을 기반한다.    
> 기본 지원을 활성화하려면 Info.plist 파일에 UIApplicationSceneMenifest 키를 추가한다.

## Scene 기반 Life-cycle 이벤트 응답
![Memory-structure image](/assets/img/blog/ios/uikit-content01-image01.png){: width="700" height="400" loading="lazy"}

* 사용자 또는 시스템이 App에 새로운 Scene을 요청하면 UIKit이 생성하여 Unattached 상태로 전환한다.
  * 사용자가 요청한 Scene은 화면에 표시되는 Foreground 상태로 빠르게 이동한다.
    * 홈 화면에서 앱을 처음 실행시켰을 때
  * 시스템이 요청한 Scene은 일반적으로 이벤트를 처리할 수 있도록 Background 상태로 이동한다.
    * 예를 들어, 시스템은 위치 이벤트를 처리하기 위해 Background에서 Scene을 실행할 수 있다.
* 사용자가 App의 UI를 무시하면 UIKit은 관련 Scene을 Background 상태로 이동하고 결국 Suspended 상태로 이동한다.
* UIKit은 언제든 Background 또는 Suspended 상태의 Scene의 연결을 끊고 리소스를 회수하여 해당 Scnene을 Unattached 상태로 되돌릴 수 있다.

* Scene 전환을 사용하여 다음 작업을 수행한다.
  * UIKit이 Scene을 App에 연결하면 Scene의 초기 UI를 구성하고 Scene에 필요한 데이터를 로드한다.
  * Foreground-active 상태로 전활할 때 UI를 구성하고 사용자와 상호 작용할 준비를 한다.
  * Foreground-actice 상태를 떠나려면 데이터를 저장하고 App의 동작을 저장한다. 
  * Background 상태로 들어가면 중요한 작업을 완료하고 가능한 한 많은 메모리를 확보한 후 App의 스냅샷을 준비한다.
  * Scene 연결 해제 시, Scene과 관련된 공유 리소스를 모두 정리한다.
  * Scene과 관련된 이벤트 외에도 UIApplicationDelegate 객체를 사용하여 App 실행에 응답해야 한다. 