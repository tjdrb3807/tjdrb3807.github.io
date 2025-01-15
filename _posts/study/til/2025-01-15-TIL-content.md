---
layout: post
title: "2025-01-15 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## ChartDataEntry
* x, y 값을 기반으로 포인터(점)을 표시

~~~swift
oepn class ChartDataEntry: ChartDataEntryBase, NSCopying {
    @objc public init(x: Double, y: Double) {
        super.init(y: y)
        self.x = x
    }
}
~~~

y값은 모든 차트 데이터 항목이 값(value)로 사용할 수 있는 필수 요소이다. 따라서 부모 클래스인 ChartDataEntryBase에서 관리한다.

그래프를 그리려면 x도 필수 아닌가??

x는 데이터의 위치 또는 카테고리를 나타내며, 차트는 유형에 따라 x의 의미와 사용 방식이 달라질 수 있으므로 필수 요소가 아니다.

## ChartDataSet
* ChartDataEntry 그룹을 관리하며, 그룹별 시각적 스타일(색, 라인, 두께...)과 데이터를 처리한다.

~~~swift
open class ChartDataSet: ChartBaseDataSet {
	@objc public init(entries: [ChartDataEntry], label: String) {
		self.entries = entries
				
		super.init(label: label)
		self.calcMinMax() 
	}
}
~~~

차트에 데이터를 올바르게 랜더링하려면 표시할 x축 y축 범위를 알아야한다. 따라서 초기화 시점에 `self.calcMinMax()` 메서드를 호출하여 차트의 값을 렌더링한다.

## ChartData
* 차트에 표시될 데이터를 관리하는 "최상위" 데이터 컨테이너 클래스
  
~~~swift
open class ChartData: NSObject, ExpressibleByArrayLiteral {
	@objc public convenience init(dataSet: Element) {
	    self.init(dataSet: [dataSet])
	}
}
~~~

ChartData 클래스는 다수의 ChartDataSet을 지원한다. 따라서 dataSet을 파라미터로 받았지만 내부적으로 배열에 감싸 관리한다.

다수의 ChartDataSet을 지원한다는 의미는 하나의 Chart에 여러 종류의 선, 막대를 표시할 수 있다는 것이다.

## 두 개의 선 구현
~~~swift
import UIKit

import SnapKit
import DGCharts

class ViewController: UIViewController {
    private var chartView = LineChartView()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(chartView)
        
        chartView.snp.makeConstraints { $0.edges.equalTo(view.safeAreaLayoutGuide) }
    
        setupLineChartDataSets()
}
    
    private func setupLineChartDataSets() {
		    // ChartDataEntry
        let entries01 = (1...10).map { index in
                ChartDataEntry(x: Double(index), y: Double.random(in: 0.0...100.0))
            }
        let entries02 = (1...10).map { index in
                ChartDataEntry(x: Double(index), y: Double.random(in: 50.0...150.0))
            }
        
        // ChartDataSet
        let dataSet01 = LineChartDataSet(entries: entries01, label: "Data01")
        let dataSet02 = LineChartDataSet(entries: entries02, label: "Data02")
        
        dataSet01.colors = [.red]
        dataSet02.colors = [.blue]
        
        // ChartData
        let chartData = LineChartData(dataSets: [dataSet01, dataSet02])
       
        chartView.data = chartData
    }
}
~~~

![Memory-structure image](/assets/img/blog/til/til03.png){: width="800" height="350" loading="lazy"}