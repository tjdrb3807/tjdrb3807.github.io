---
layout: post
title: "iOS - UIImagePickerController"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
오늘은 한 달간 인털활동을 하는 기업에서 진행하는 외주 프로젝트중 카메라와 라이브러리 접근 기능을 구현하기 앞서 iOS 환경에서는 이를 어떻게 구현할 수 있는지 관련 레퍼런스를 공부해보려합니다.    
iOS에서 카메라에 접근할 수 있는 방법은 UIImagePickerController와 AVFoundation 두 가지 방법이 있는데 해당 프로젝트는 간단히 카메라만 띄우는 요구사항이 필요하므로 UIImagePickerController를 채택했습니다.

~~~swift
@MainActor
class UIImagePickerController
~~~

## SoucreType
~~~swift
open var sourceType: UIImagePickerController.sourceType
~~~
UIImagePickerController의 역할과 모양은, 해당 컨트롤러를 화면에 표시하기 전에 설정한 `UIImagePickerController.sourceType` 프로퍼티에 따라 달라집니다.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/uiImagePickerController01.png" alt="Memory-structure image" loading="lazy" />
</div>   
<br>

UIImagePickerContoller.SourceType은 `.photoLibrary`, `.camera`, `.savedPhotosAlbum` 총 세 가지 케이스가 있는 열거형입니다. 하지만, 아래 오픈 소스에서 확인할 수 있듯 .photoLibrary와 .savedPhotosAlubm은 추후 릴리즈에서 제거될 예정이므로, `PHPicker`를 사용하라고 하네요.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/uiImagePickerController02.png" alt="Memory-structure image" loading="lazy" />
</div>
<br>

발주사에서 카메라와 라이브러리 접근 모두 의뢰했지만, 신규 프로젝트에서 deprecated될 기능을 적용할 수 없으니 UIImagePickerController에서는 SourceType.camera만 적용하고 라이브러리 접근은 PHPicker를 적용해야겠네요.(PHPicker에 관한 포스팅은 다음에 작성해보겠습니다.)

그렇다면 UIImagePickerController.SourceType.camera만 설정하면 디바이스의 카메라 기능을 사용할 수 있냐? 사실을 됩니다. 하지만 Apple 공식문서에 따르면 특정 SourceType은 해당 디바이스에서 물리적으로 존재하지 않거나 접근할 수 없는 경우 사용이 불가능할 수 있다고 합니다.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/uiImagePickerController03.png" alt="Memory-structure image" loading="lazy" />
</div>
<br>

따라서 UIImagePickerController.SourceType 열거형 상수를 사용해서 `isSourceTypeAvailable(_:)` 메서드를 호출하라고 하네요.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/uiImagePickerController04.png" alt="Memory-structure image" loading="lazy" />
</div>
<br>

파라미터에 해당 디바이스에서 사용하고 싶은 UIImagePickerController.SourceType을 전달했을 때 디바이스에서 해당 SourceType을 지원하면 true, 지원하지 않으면 false 값을 반환합니다.

Apple의 모든 디바이스가 카메라나 라이브러리 같은 SourceType을 지원하는 것이 아니기 때문에 해당 메서드를 반드시 호출해야한다고 합니다.

또, 사용자가 라이브러리에서 이미지를 선택하려고 할 때 라이브러리가 비어있거나, 카메라가 이미 사용중인 경우에 해당 메서드는 false를 반환한다고 합니다.

그 다음 SourceType.camera를 설정했을 때 사용하려는 `mediaType` 프로퍼티를 설정해야 합니다.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/uiImagePickerController05.png" alt="Memory-structure image" loading="lazy" />
</div>
<br>

이 프로퍼티에 설정된 mediaType에 따라, UIImagePickerController는 사진이나 동영상 전용 인터페이스를 표시하거나, 사용자가 인터페이스를 선택할 수 있는 컨트롤을 제공합니다.

이 속성을 설정하기 전에 availableMediaType(for:) 클래스 메서드를 호출해서 해당 디바이스에 어떤 mediaType이 사용 가능한지 확인하세요.

이 프로퍼티는 빈 배열로 설정하거나, 현재 sourceType에 대해 사용 가능한 미디어 타입이 포함되지 않은 배열로 설정하면 시스템에서 예외가 발생합니다.

미디어를 캡처할 때 이 속성의 값은 어떤 카메라 인터페이스를 표시할지를 결정합니다.

