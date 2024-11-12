---
layout: post
title: "Design Patterns #Section01 MVC(02) - basic MVC vs Cocoa MVC"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, design_patterns]
---

* toc
{:toc}

## 📋 Section List
[**👉 MVC(01) - Model, View, Controller**](https://tjdrb3807.github.io/study/design_patterns/2024-11-11-design_patterns_MVC/)

## 🖐️ Introduce
지난 글에서 MVC 디자인 패턴를 구성하는 객체들 Model, View, Controller 각각의 특징에 대해 알아봤다.   
이번 파트에서는 Basic MVC 패턴과 Cocoa MVC 동작 방식을 확인하고 차이점에 대해 알아본다.

## 🌱 Basic MVC
당연한 말이겠지만 MVC 패턴은 iOS 개발에만 국한된 것이 아니다.    
범용적으로 사용되는 MVC 패턴은 Composit, Strategy, Observer 패턴으로 구성된 복합 디자인 패턴이다.

### Composit pattern(View)
  * App의 View 객체는 실제로 조직화된 방식으로 함께 동작하는 중첩된 View의 복합체이다.
  * Interface 구성 요소들은 Window부터 Compound View나 Individual View까지 다양하다.

### Strategy Pattern(Controller)
  * Controller 객체들은 하나 이상의 View 객체들을 위한 로직을 구현해야 한다. 
  * View 객체는 시각적 기능을 구현하는 데 국한되며, Inteface 동작들이 구체적으로 어떻게 돌아가는지는 Controller에게 위임한다.

### Observer Pattern(Model)
  * Model 객체는 App에서 Model의 Data에 관심있는 객체(일반적으로 View 객체)에게 상태 변화를 지속적으로 알려준다.

<br>

![Memory-structure image](/assets/img/blog/architecture/section01_image01.png){: width="100%" height="100%" loading="lazy"}
Basic MVC Diagram
{:.figcaption}

위 그림은 Composit, Strategy, Observer 패턴이 함께 작동하는 Basic MVC 패턴을 도식화한 것인다.    
1. Composit Pattern을 적용한 View 객체에서 User Action을 통해 Event가 방출된다.       
2. Controller 객체는 해당 Event를 수신하여 설계된 비즈니스 로직을 기반으로 처리한다.    
    이때 해당 Controller는 Model에게 Data 상태 변경을 요청할 수도 있고 View 객체에 Interface 동작이나 모양을 변경하도록 요청할 수 있다.    
3. Model 객체는 상태가 변경될 때 Observer로 등록된 모든 객체에게 알린다. ***만약 Oberver가 View 객체라면 내역에 따라 Interface 변경을 할 수도 있다.***      


## 🌱 Cocoa MVC
사실 Basic MVC 구조에 기반한 App 설계는 가능하다.    
바인딩(Binding) 기술을 사용하면 Model의 Notification이 View에 다이렉트로 전달되므로 Model의 상태 변경 알림을 수신하는 Cocoa MVC App을 쉽게 만들 수 있다.     
하지만 위 방법으로 MVC 패턴을 설계하면 ***View와 Model 객체의 재사용성을 확보할 수 없는 문제***에 직면하게 된다.    
* View는 App에 정의된 내용(비즈니스 로직)을 OS가 보고 표현한 객체이다.    
    따라서 OS입장에서 View 객체는 Interface와 기능에 일관성이 있어야 하며, 높은 수준의 재사용성을 가질 것을 요구한다.     
* Model 객체는 정의에 의해 특정 영역에 연관된 Data를 캡슐화하고 해당 Data들에 대한 동작을 수행한다. 
      
그러므로 ***설계측면에서 재사용성을 높이기위해 Model과 View 객체를 서로 분리하는 것이 가장 좋다.***    

<br>

### Cocoa MVC

![Memory-structure image](/assets/img/blog/architecture/section01_image02.png){: width="100%" height="100%" loading="lazy"}
Cocoa MVC Diagram
{:.figcaption}

대부분의 Cocoa App에서 Model의 상태변화에 대한 Notificaion들은 Controller 객체를 통해 View로 전달된다.


