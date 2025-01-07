---
layout: post
title: "2025-01-06 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## 문제 상황
포켓몬 데이터를 화면에 렌더링하면서 대량의 이미지 데이터를 네트워크 요청으로 받아오다 보니 데이터가 많은 경우 스크롤 시 화면이 끊기는 문제 발생

## 해결책: Prefetching 적용
### UICollectionView Prefetching 도입
RxSwift의 rx.prefetchItems를 활용하여 스크롤 중 데이터를 미리 로드하도록 설정.

~~~swift
let input = MainViewModel.Input(
    viewWillAppear: self.rx.viewWillAppear.asObservable(),
    loadNextPageTrigger: collectionView.rx.didScroll
        .throttle(.milliseconds(200), scheduler: MainScheduler.instance)
        .withUnretained(self)
        .filter { vc, _ in
            vc.isRead && vc.isNearBottom()
        }.map { _ in () },
    selectedItem: collectionView.rx.itemSelected.map { $0.row },
    prefetchTrigger: collectionView.rx.prefetchItems.asObservable()
)
~~~

### ViewModel에서 Prefetching 처리 로직
스크롤 동작에 따라 데이터를 Prefetch하는 로직을 MainViewModel에 추가   
prefetchTrigger를 통해 데이터를 미리 가져오도록 설계   

~~~swift
input.prefetchTrigger
    .withUnretained(self)
    .filter { vm, indexPaths in
        guard let maxIndex = indexPaths.map({ $0.row }).max() else { return false }
        return vm.canPrefetch(at: maxIndex)
    }.flatMapLatest { vm, _ in vm.fetchNextPage() }
    .subscribe()
    .disposed(by: disposeBag)

~~~


ddd