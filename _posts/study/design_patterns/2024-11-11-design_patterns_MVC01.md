---
layout: post
title: "Design Patterns #Section01 MVC(01) - Model, View, Controller"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, design_patterns]
---

* toc
{:toc}

## 📋 Section List
[**👉 MVC(01) - Model, View, Controller**](https://tjdrb3807.github.io/study/design_patterns/2024-11-11-design_patterns_MVC/)

## 🖐️ Introduce
애플의 iOS 프로그래밍을 처음 접했을 때 Cocoa MVC 디자인 패턴에 대해 가장 먼저 들었던 말은 Massive View Controller였다. 

그 당시 디자인패턴에 대한 견해가 넓지 않아 굳이 구식? 디자인 패턴을 공부할 필요가 있나 싶었던 필자는 Cocoa MVC를 상호 운용할 수 있는 MVVM 패턴을 먼저 익혔고 다양한 프로젝트에 적용해왔다.(물론 MVVM도 써보기만 했지 깊게는 모름..)

하지만 Cocoa MVC는 엄연히 Apple이 iOS 프로그래밍에 채택한 디자인 패턴인다.(Apple도 Massive View Controller를 인정하긴 함..)   
따라서 해당 섹션을 통해 iOS 프로그래밍의 가장 기본인 Cocoa MVC 디자인 패턴에 대한 견문을 넓히고, 앞으로 필자의 개발 인생에서 Cocoa MVC를 적용하지 않는 명백한 이유를 수립하고자 작성한다.

## 🌱 MVC 객체의 3가지 타입과 연관관계
Cocoa MVC 패턴을 알아보기 앞서 MVC 디자인 패턴을 구성하는 객체 타입에 대해 알아보도록 한다.    
MVC 디자인 패턴은 총 3개의 객체 타입으로 구성되어 있으며, 그 종류로는 Model, View, Controller 세 가지가 존재한다.   
***세 가지 객체는 각각 추상적 경계로 다른 객체와 분리되어 있으며, 이러한 경계를 통해 다른 유형의 객체와 상호작용 한다.***

### Model
Model 객체를 한 마디로 표현하자면 "앱의 아이텐티티"이다.      

* Model 객체에는 Data 또는 DB에 위치한 인스턴스를 서버로 부터 Request하기 위한 네트워크 통신 로직이 담기며, ***MVC 기반으로 잘 설계된 App은 모든 Data를 Model 객체에 캡슐화 할 수 있다.***   
* Model 객체는 Data를 활용해 App에서 제공하고자 하는 비즈니스(비즈니스 로직)이 담며, ***특정 도메인에  관련된 Data를 표현하기 위해 재사용성이 확보되는 경향이 있다.***   
* Model은 Interface나 Presentation 계층에 속한 객체와는 명시적인 연관관계를 성립할 수 없다.

### View
View 객체는 Model 객체의 프로퍼티(Data)를 어떻게 Interface에 표현할지 정의하는 코드가 담긴다.    
그러므로 ***Model 객체에서 변경되는 Data 값을 View 객체에서 알아야 할 명분이 필요하다.***    
하지만 Model 객체가 특정 View 객체와 직접적인 연관관계를 성립하지 않으므로, Model 객체의 변경 사항을 View 객체에 알리기 위한 별도의 방법이 View 객체에 필요하다.  

* View 객체는 Data를 저장하는 것에 대한 책임이 없어야하며, 이는 ***Model 객체와 명시적인 연관관계를 성립할 수 없다***는 뜻이다.
* ***View 객체는 재사용성이 확보된다.***
  * Cocoa에서 UIKit 프레임 워크는 많은 View 객체들을 정의하고 이를 Interface Builder 라이브러리로 제공한다.
  * UIKit의 View 객체들을 재사용하므로써, 모든 Cocoa App에서 기능적으로 높은 수준의 일관성을 보장한다.


### Controller
Controller 객체를 한 단어로 표현하면 '중재자'이다.

Controller 객체는 View 객체가 Model 객체에 접근(?)할 수 있도록 보증하는 역할을 하며, View 객체가 Model 객체의 Data 변화를 인지하는 통로 역할을 한다.   

일반적인 Cocoa MVC 패턴은 View 객체에서 발생한 User Action이 해당 Event를 방출하고 이를 Controller 객체에 전달한다.    Controller 객체는 해당 Event를 기반으로 Model 객체가 가지고 있는 Data를 어떻게 처리할 것인지 명령한다.     
해당 명령을 통해 Model 객체에 위치한 데이터가 변경되며, 변경 된 Data를 User에게 전달할 필요가 있다면, Model 객체는 변경 내역을 Controller 객체에 전달한다.   
Model로 부터 전달된 Data 변경 내역을 Controller 객체는 다시 View 객체에 전달하므로서 User에게 보여지는 Interface가 수정되는 매커니즘을 갖는다.   

Controller 객체는 App의 전반적인 Set-up이나 객체의 Life Cycle을 관리할 수도 있다.

---