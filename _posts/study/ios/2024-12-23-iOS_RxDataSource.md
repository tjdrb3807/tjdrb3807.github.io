---
layout: post
title: "iOS - RxDataSource"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## RxDataSource의 주요 특징

* 반응형 데이터 업데이트
  * 데이터가 변경되면 UITableView 또는 UICollectionView가 자동으로 업데이트 퇸다.
* Diffing Algorithm 사용
  * 내부적으로 데이터 변경을 비교(Diffing)하여 변경된 셀만 업데이트한다
  * 성능 최적화
* 다양한 데이터 섹션 모델 지원
  * 단일 섹션뿐만 아니라, 다중 섹션 모델을 쉽게 정의하고 처리할 수 있다.
* 애니메이션 지원
  * 데이터 변경 시 삽입, 삭제, 이동 등에 애니메이션을 쉽게 적용할 수 있다.
* Swift 타입 안전성
  * Swift의 제네릭 타입 시스템을 활용하여 안전한 데이터 관리.

## 데이터 모델 정의
RxDataSource는 기본적으로 SecionModel 구조체를 사용하여 섹션 기반의 데이터를 정의한다.
~~~swift
struct MySection {
    var header: String
    var items: [String]
}

extension MySection: SectionModelType {
    typealias Item = String

    init(original: MySection, items: [Item]) {
        self = origin
        self.items = items
    }
}
~~~

## RxDataSource 객체 생성
RxTableViewSectionReloadDataSource를 사용해 데이터 소스를 정의한다.

~~~swift
let dataSource = RxTableViewSectionedReloadDataSource<MySection>(
    configureCell: { dataSource, tableView, indexPath, item in
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        cell.textLabel?.text = item
        return cell
    }
)
~~~

## 데이터 바인딩
RxSwift의 Obsesrvable을 사용해 데이터와 UI를 바인딩한다.

~~~swift
let sections = Observable.just([
    MySection(header: "First Section", items: ["Item 1", "Item 2", "Item 3"]),
    MySection(header: "Second Section", items: ["Item 4", "Item 5"])
])

sections
    .bind(to: tableView.rx.items(dataSource: dataSource))
    .disposed(by: disposeBag)
~~~