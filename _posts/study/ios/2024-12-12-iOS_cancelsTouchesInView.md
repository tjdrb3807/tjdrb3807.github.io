---
layout: post
title: "iOS - cancelsTouchesInView"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
<div style="text-align: center;">
  <img src="/assets/img/blog/ios/cancelTouchesInView_image01.png" alt="Memory-structure image" width="200" height="350" loading="lazy" />
</div>

<br>

위 화면과 같은 상황에서 UIViewController의 View를 Tap 했을 때 키보드가 내려가는 상황을 연출하기 위해 다음과 같은 코드를 작성했다.

~~~swift
class BaseViewController: UIViewController {
    // ...

    override func viewDidLoad() {
        super.viewDidLoad()
        // ...
        
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(dismissKeyboard))
        view.addGestureRecognizer(tapGesture)
    }

    @objc private func dismissKeyboard() { view.endEditing(true) }
}
~~~

해당 프로젝트는 코드의 중복을 줄이고 공통 기능에 대한 재사용성을 높이고자 BaseViewController를 구현해 사용중이었으며, 추후 앱의 기능들이 확장되서 다양한 ViewController가 구현된다 가정할 때 키보드를 내리는 기능은 어떠한 ViewController에서도 필요하다 판단하여 BaseViewController에 구현하게 됐다.

<br>

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/cancelTouchesInView_image02.png" alt="Memory-structure image" width="200" height="350" loading="lazy" />
</div>

<br>

하지만 위 사진처럼 BaseViewController를 상속받은 ViewController에서 SubView로 존재하는 UITableView의 didSelectRowAt 메서드가 호출되지 않는 상황이 발생했다. 

처음에는 UITableView의 Layout이 잘못 설정된 것이 아닌가?, isUserInteractionEnabled를 false로 설정한 것이 아닌가? 하고 확인해봤지만 모두 정상이었다.

그렇다면 오류를 발생시킨 남은 선택지는 Tap 이벤트를 핸들링하는 `UITapGestureRecognizer`에 있다 판단했다.

혹시나 하는 마음에 아래 코드를 UITableViewCell에서 정의하고 앱을 실행해 Cell을 탭 해봤지만 역시나 콘솔로그에 아무 기록이 남지 않았다.

~~~swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesBegan(touches, with: event)
        
    print("touchesBegin in PhoneBookTableViewCell")
}
~~~

구글링에 "swift tap gesture 인식 안됨"이라 검색하니 나와 비슷한 사례의 글들을 볼 수 있었고 그중 중복적으로 보이는 키워드는 `cancelsTouchesInView = false`였다. 

이 한 줄의 코드작성으로 오류를 잡을 수 있었지만 매커니즘을 모르고 사용하는 코드는 의미없다 생각하기에 본 글에서는 cancelTouchedInView에 대해서 알아보고자 한다.

## Discussion
<div style="text-align: center;">
  <img src="/assets/img/blog/ios/cancelTouchesInView_image03.png" alt="Memory-structure image" width="700" height="350" loading="lazy" />
</div>

<br>

공식문서는 해당 프로퍼티를 다음과 같이 설명하고있다.

>cancelsTouchedInView 프로퍼티는 `true`가 기본값이며, Gesture recognizer가 제스처를 인식한 경우, 해당 제스처의 보류 중인 터치 이벤트는 뷰에 전달되지 않으며, 이미 전달된 터치 이벤트는 `touchesCancelled(_:with:)` 메서드를 통해 취소된다.     
>만약 Gesture recognizer가 제스처를 인식하지 못했거나, 해당 프로퍼티의 값이 `false`인 경우, View는 멀티터치 시퀀스의 모든 터치 이벤트를 계속해서 받을 수 있다.

### cancelsTouchesInView = true
* 제스처가 인식되면, 해당 터치와 관련된 이벤트(.touchesBegan 같은것들)를 더이상 터치가 발생한 View로 전달되지 않게한다.
* 대신 View의 `touchesCancelled(_:with:)` 메서드가 호출돼서 터치 이벤트를 취소한다.

### cancelsTouchesInView = false
* 제스처가 인식되더라도 터치 이벤트는 View로 계속 전달된다.
* View의 터치 이벤트가 정상적으로 호출된다.

### 정리
터치 이벤트가 발생하면, UIKit은 Gesture recognizer에 먼저 인벤트를 전달하게되고 Gesture recogniser가 터치 이벤트를 감지하고, 이를 제스처로 인식하면 해당 제스처를 처리하게된다.

Gesture recogniser가 이벤트를 처리하면, 해당 터치 이벤트는 기본적으로 View에 전달되지 않게되는데, 이때 `cancelsTouchesInView = false`로 설정하면 제스처가 인식 되더라도 터치 이벤트는 View로 계속 전달된다.

추가적으로 `cancelsTouchsInView = true` 상태면 View가 이벤트를 처리하지 못하게 되고 해당 이벤트는 super view controller나 상위 객체로 이벤트가 전달된다.

BaseViewController에 아래 코드를 작성후 cancelsTouchesInView = true 상태에서 cell을 탭하니 콘솔로그에 찍히는 것을 확인할 수 있다

~~~swift
@objc private func dismissKeyboard() {
    print("CALL: dismissKeyboard()")
    view.endEditing(true)
}
~~~









