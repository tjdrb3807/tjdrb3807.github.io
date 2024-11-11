---
layout: post
title: "Design Patterns #Section01 MVC(1)"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, design_patterns]
---

* toc
{:toc}

## 🖐️ Introduce
애플의 iOS 프로그래밍을 처음 접했을 때 MVC 디자인 패턴에 대해 가장 먼저 들었던 말은 Massive View Controller였다. 

그 당시 디자인패턴에 대한 견해가 넓지 않아 굳이 구식? 디자인 패턴을 공부할 필요가 있나 싶었던 필자는 MVC를 상호 운용할 수 있는 MVVM 패턴을 먼저 익혔고 프로젝트에 적용해왔다.(물론 MVVM도 써보기만 했지 깊게는 모름..)

하지만 Cocoa MVC는 엄연히 Apple이 iOS 프로그래밍에 채택한 디자인 패턴인다.(Apple도 Massive View Controller를 인정하긴 함..)   
따라서 본 글에서 iOS 프로그래밍의 가장 기본인 MVC 디자인 패턴에 대한 견문을 넓히고, 앞으로 필자의 개발 인생에서 MVC를 적용하지 않는 명백한 이유를 수립하고자 작성한다.

## MVC 객체의 3가지 타입과 연관관계
Cocoa MVC 패턴을 알아보기 앞서 MVC 디자인 패턴을 구성하는 객체의 타입에 대해 알아보도록 한다. MVC 디자인 패턴은 총 3개의 객체 타입으로 구성되어 있으며, 그 종류로는 Model, View, Controller 세 가지가 존재한다.

### Model
Model을 한 마디로 표현하자면 앱의 아이텐티티이다. 
데이터 또는 서버와 API통신 하기위한 네트워크 로직이 담기며, 해당 데이터를 활용해 앱에서 제공하고자 하는 서비스(비즈니스 로직)이 Model에 담긴다.

Model 객체의 특징으로는 뒤에서 설명할 View 객체(UI)와 연관관계를 성립하지 않는다는 것이다. 

### View
View는 Model객체에 위치한 데이터를 Interface에 뿌리는 코드가 담기며, Storyboard나 xib 파일들이 View에 해당된다.

위 Model 객체의 특징에서 언급한 것처럼 View 객체 역시 Model 객체, 즉 데이터를 저장하는 것에는 책임이 없어야 한다.

### Controller
Controller는 Model과 View 사이에서 데이터 흐름을 제어한다.
View를 통해 사용자 Action이 Controller로 전달되면, Controller는 해당 Action을 기반으로 Model이 가지고있는 데이터를 어떻게 처리할 것인지 명령한다. 해당 명령을 기반으로 Model에 위치한 데이터화 View에서 사용자에게 보여지는 인터페이스가 수정된다. 보통 @IBAction 함수들이 정의된다.