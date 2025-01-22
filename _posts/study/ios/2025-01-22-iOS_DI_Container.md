---
layout: post
title: "iOS - DI Container(01)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}


## Introduce
![Memory-structure image](/assets/img/blog/ios/dicontainer_image01.png){: width="800" height="350" loading="lazy"}

프로젝트 설계 과정에서 의존성 주입에 대한 안건이 나왔다. 
사진처럼 설계했을 때 의존성 객체가 필요한 시점에 프로젝트 여러곳에서 중복으로 객체 생성이 이루어져 자원 낭비가 일어나고, 해당 인스턴스를 초기화한다고 할때 DI로 인한 연속적인 주입이 필요하게된다. 이에따라 코드의 복잡성 증가, 가독성이 떨어지는 상황을 우려했다.



## DI Container
의존성을 관리하고 주입하기 위한 일종의 팩토리와 같다. DIContainer는 프로젝트에서 사용되는 다양한 ViewModel이나 Service 등의 인스턴스를 생성하고 팩토리에 담아 관리한다.

### DI Container의 주요 역할
의존성 등록   
프로젝트에서 사용될 의존성을 등록하는 역할을 한다. 일반적으로 인터페이스(Protocol)와 해당 인터페이스를 구현한 인스턴스(Class) 또는 구조체를 등록한다.   

의존성 해결    
등록된 의존성을 필요로 하는 객체에 주입하는 역할을 한다. DI Container를 통해 의존성을 해결하면 해당 객체는 필요한 의존성을 직접 생성하지 않고 DI Container로부터 주입받게 된다.