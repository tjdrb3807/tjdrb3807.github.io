---
layout: post
title: "iOS - Pagination"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}


## Introduce
페이지네이션(Pagination)은 대량의 데이터를 한 번에 로드하지 않고, 작은 단위로 나누어 점진적으로 로드하는 기술이다.   
이는 사용자 경험을 개선하고, 성능을 최적화하며, 네트워크 리소스를 절약하는 데 주로 사용된다.

## 메커니즘
* 초기 데이터 로드
  * 앱이 시작되거나 화면이 열릴 때 첫 번째 페이지 데이터를 요청
  * UI에 데이터를 표시
* 다음 페이지 요청
  * 사용자의 행동(스크롤)을 기반으로 추가 데이터 요청
* 데이터 병합
  * 기존 데이터에 새 데이터를 병합하여 UI를 업데이트

## 다음 페이지 요청 트리거
### 페이징 트리거 처리를 위해 필요한 주요 속성
* contentOffset.y: 현재 스크롤 위치
* contentSize.height: 전체 콘텐츠 높이
* frame.size.height: ScrollView의 현재 화면 높이

### 페이징 트리거 로직
`contentOffset.y`가 `contentSize.height` - `frame.size.height`와 가까워지면 트리거 발동
~~~swift
let input = MainViewModel.Input(
    loadNextPageTrigger: collectionView.rx.didScroll
        .filter { [weak self] in
            guard let self = self else { return false }

            let contentOffset = self.collectionView.contentOffset.y
            let maximumOffset = self.collectionView.contentSize.height - self.collectionView.frame.size.height

            return maximumOffset - contentOffset <= 200
        }
)
~~~

### 다음 페이지 요청
~~~swift
final class MainViewModel: ViewModel {
    struct Input {
        let loadNextPageTrigger: Observable<Void>
    }

    struct Output {
        let thumbnailList: BehaviorSubject<[URL]>
    }

    private var currentPage = 0
    private let pageSize = 20

    let listSubject = BehaviorSubject<[URL]>(value: [])

    func transform(input: Input) -> Output {
        input.loadNextPageTrigger
            .withUnretained(self)
            .flatMap { vm, _ in
                vm.fetchNextPage()
            }.subscribe()
            .disposed(by: disposeBag)

        return Output(thumbnailList: listSubject)
    }

    private func fetchNextPage() -> Observable<Void> {
        let offset = currentPage * pageSize

        guard let url = URL(string: "https://pokeapi.co/api/v2/pokemon?limit=\(pageSize)&offset=\(offset)") else {
            isFetching.onNext(false)
            return Observable.empty()
        }

        return NexworkManager.shared.fetch(url: url)
            ...

            .do(onSuccess: { [weak self] newUrls in
                var currentList = try self.listSubject.value()
                currentList.append(contentOf: newUrls)

                self.listSubject.onNext(currentList)
                self.currentPage += 1
            })

            ...
    }
}
~~~