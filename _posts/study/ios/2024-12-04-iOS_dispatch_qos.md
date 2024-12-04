---
layout: post
title: "iOS - Dispatch QoS(01)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
Dispatch QoS(Quality of Service)는 GCD에서 작업의 중요도와 우선순위를 결정하기 위해 사용되는 기능이다.

QoS를 설정하면 시스템이 작업의 중요도에 따라 CPU와 리소스를 효율적으로 배분하여 성능 최적화를 기대할 수 있다.

## Dispatch QoS Class
* .userInteractive
  * 즉각적인 반응이 필요한 작업
  * 지연이 없어야함
  * UI업데이트, 애니메이션, 터치 이벤트 처리
  * Main Queue는 내부적으로 Serial Queue로 구현되어 있으며, .userInteractive를 사용한다. 
* .userInitiated
  * 사용자가 요청한 작업
  * 빠른 작업 완료가 필요하지만, .userInteractive 보다 덜 긴급한 작업
  * 파일 열기, 데이터 로드, 네트워크 요청
* utility
  * 완료 시간이 길더라도 에너지 효율성이 중요한 작업
  * 데이터 백업, 파일 다운로드, 로그 처리
* .background
  * 사용자에게 즉시 보여지지 않는 작업
  * 시스템 리소스 최소화
  * 데이터 동기화, 백그라운드 다운로드
* .default
  * 명시적으로 QoS가 설정되지 않은 작업에 사용
  * 시스템 기본 우선순위
  * Global Queue의 qos 기본값은 .default 이다.
    * ![Memory-structure image](/assets/img/blog/ios/dispatch_qos_image02.png){: width="600" height="350" loading="lazy"}
* .unspecified
  * QoS가 명시되지 않은 작업
  * 시스템이 우선순위를 자체적으로 결정
  * 특별한 경우 외에는 사용하지 않음
  * async의 qos 기본값은 .unspecified 이다.
    * ![Memory-structure image](/assets/img/blog/ios/dispatch_qos_image03.png){: width="600" height="350" loading="lazy"}

## QoS의 동작 방식
* 작업 스케줄링
  * GCD는 작업을 Queue에 추가하면, 스레드 풀(Thread Pool)에서 실행할 작업을 선택한다.
  * QoS가 높은 작업은 더 빨리 실행되며, 이때 시스템 리소스를 더 많이 할당받는다.
  * QoS가 낮은 작업은 리소스가 유휴 상태일 때 실행된다.
* 스레드 우선순위
  * QoS는 스레드 우선순위와 밀접하게 연관되어 있다.
    * 높은 QoS: 메인 스레드 또는 즉시 사용 가능한 고우선순위 스레드에서 실행
    * 낮은 QoS: 유휴 상태 스레드에서 실행되거나, 높은 QoS 작업이 먼저 실행된다.