---
layout: post
title: "UITableViewDataSource"
sitemap: false
categories: [study, ios]
tags: [ios]
---
* toc
{:toc}

## 정의
UITableView는 데이터를 리스트 형식으로 보여주는 기능만 지원하며, 보여주는 데이터를 스스로 관리하지 않는다. 
이를 관리하기 위해서는 UITableView에 UITableViewDataSource 프로토콜을 구현한 객체를 제공해야 한다. 

## 역할
UITableViewDataSource는 다음과 같은 책임을 갖는다. 
1. Section의 수과 Row의 수를 보고한다. 
2. UITableView 각 Row에 대해 Cell을 제공한다.
3. 각 Section의 Header와 Footer에 대한 title을 제공한다.
4. UITableView의 Index를 구성한다.
5. 데이터가 변경해야 하는 상황에서 업데이트에 응답한다.

## 필수구현 메서드
UITableViewDataSource에는 필수로 구현해야 하는 메서드가 두 가지 존재한다.
```Swift
// Return the number of rows for the table.     
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
   return 0
}


// Provide a cell object for each row.
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
   // Fetch a cell of the appropriate type.
   let cell = tableView.dequeueReusableCell(withIdentifier: "cellTypeIdentifier", for: indexPath)
   
   // Configure the cell’s contents.
   cell.textLabel!.text = "Cell text"
       
   return cell
}
```

### IndexPath
UITableView의 행을 식별하는 인덱스 경로 
[secion, row]로 이루어져 있는 구조체

### dequeueReuseableCell
numberOfRowsInSection 메서드에 100 개의 count가 반환되어도 재사용 큐를 사용한다면 초기 화면에 보여질 Cell 개수만큼 Cell을 생성한다. (+a 여유분) 그 후 아래로 또는 위로 스크롤 될 경우 UITableView의 새로운 데이터가 화면에 보여질 때 초기 생성된 Cell들이 화면 밖으로 사라진다. 이때 화면 밖으로 벗어난 Cell들을 재사용한다.

ReuseableQueue에 enqueue되고(Cell 인스턴스 유지) 새로 보여져야 할 데이터는 Cell의 인스턴스가 필요한데 이는 재사용 큐에 저장되어있던 Cell이 dequque해서 재사용한다.

ReuseableQueue를 사용하지 않느다면 tableView(_:cellForRowAt")메서드에서 지정한 count 수 만큼 cell 인스턴스를 전무 만들어 두어야한다. 이럴경우 화면에 보여지지 않는 cell도 메모리에 남아있게되고 할당 해제되기 전까지 시스템 자원을 사용하게 돼서 메모리 누수가 발생한다.

### tableView(_:cellForRowAt:)
특정 Cell이 재사용 큐에 있다면 꺼내오고 dequeueReuseableCell(withIdentifier:for:), 재사용 큐에 dequeue된 cell이 없다면 cell 인스턴스를 생성해서 반환한다. 
이때 indexPath를 통해 어느 영역의 item인지에 대한 cell을 재사용 큐로부터 반환한다.

