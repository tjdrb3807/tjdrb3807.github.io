---
layout: post
title: "URL Session(01)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
URLSession Loding System은 HTTPs와 같은 표준 프로토콜이나 사용자가 커스텀한 프로토콜을 사용해 URL로 식별된 리소스에 접근할 수 있는 기능을 제공한다.

로딩 **작업은 비동기적으로 수행**되므로, 앱은 응답성을 유지하며 데이터가 도착하거나 오류가 발생할 때 이를 처리할 수 있다.

![Memory-structure image](/assets/img/blog/ios/urlsession_image01.png){: width="600" height="350" loading="lazy"}

![Memory-structure image](/assets/img/blog/ios/urlsession_image02.png){: width="600" height="350" loading="lazy"}

<br>

URLSession 인스턴스를 사용하여 하나 이상의 URLSessionTask 인스턴스를 생성할 수 있으며, 생성된 URLSessionTask 인스턴스는 데이터를 Request, Response, 파일을 다운로드할 수 있다.

URLSession을 구성하기 위해서는 URLSessionConfiguration 객체를 사용한다.

이를 통해 캐시와 쿠키를 사용하는 방법이나, 셀룰러 네트워크에서 연결을 허용할지 여부와 같은 동작을 제어할 수 있다.

## URLSession 종류
URLSession은 기본 요청을 처리하기 위한 싱글톤(shared) 세션을 제공한다.

해당 세션은 URLSessionConfiguration 객체를 사용하지 않는다.

사용자가 직접 생성한 세션만큼 커스텀할 수는 없지만, 요구사항이 매우 간단할 경우 사용하기 적절하다.

![Memory-structure image](/assets/img/blog/ios/urlsession_image03.png){: width="600" height="350" loading="lazy"}

<br>

다른 종류의 세션을 사용하려면 다음 URLSessionConfiguration 객체를 사용해 URLSession을 생성해야 한다.

### .default
.default 세션은 싱글톤 세션과 유사하게 동작하지만, 사용자가 설정을 구성할 수 있다.

또한 데이터를 받을 수 있도록 델리게이트를 할당할 수도 있다.

### .ephemeral
.ephemeral 세션은 싱글톤 세션과 유사하지만, 캐시, 쿠키와 같은 중요 데이터를 디스크에 기록하지 않는다.
### .background
.background 세션은 앱이 실행 중이 아닐 때에도 데이터를 업로드하거나 다운로드할 수 있도록 지원한다.
## 비동기성와 URLSession
URLSession API는 비동기적이며, 메서드 호출 방식에 따라 앱에 데이터를 반환하는 다양한 방법을 지원한다.
### async/await
async/await 키워드를 사용한 비동기 작업은 가독성이 completion handler 방식과 비교했을때 높으며, try-catch 구문을 사용해 비교적 간단한 에러 처리를 할 수 있다. 

하지만 코드를 실행하는 스레드가 지정되기 때문에 효율적인 컨텍스트 스위칭 및 QoS를 통한 성능 최적화를 기대하기는 어렵다.

~~~swift
var urlComponents = URLComponents(string: "url")
urlComponents?.queryItems = self.urlQueryItems
        
guard let url = urlComponents?.url else { return }
        
Task {
    let data = await fetchData(url: url)
    // ...          
}

private func fetchData(url: URL) async -> CurrentWeatherResponseDTO? {
    let urlSession = URLSession(configuration: .default)
    
    do  {
        let (data, response) = try await urlSession.data(from: url)
        if let response = response as? HTTPURLResponse,
           (200..<300).contains(response.statusCode) {
            guard let decodedData = try? JSONDecoder().decode(CurrentWeatherResponseDTO.self, from: data) else { return nil }
            return decodedData
        } else {
            return nil
        }
    } catch {
        return nil
    }
}
~~~

### completion handler
completion handler 방식은 직접 사용자가 해당 작업을 수행할 스레드를 지정할 수 있으며, QoS를 잘 적용하면 성능 최적화를 기대할 수 있다. 

하지만 async/await 키워드 방식에 비해 가독성이 떨어지며, 복잡한 비동기 작업 구현시 콜백 재옥 현상이 발생할 수 있다.

~~~swift
var urlComponents = URLComponents(string: "url")
urlComponents?.queryItems = self.urlQueryItems

guard let url = urlComponents?.url else { return }

fetchData(url: url) { [weak self] (data: CurrentWeatherResponseDTO?) in
    guard let self, let data else { return }

    DispatchQueue.main.async {
        //...
    }
}


private func fetchData<T: Decodable>(url: URL, completion: @escaping (T?) -> Void) {
    let urlSession = URLSession(configuration: .default)
    urlSession.dataTask(with: URLRequest(url: url)) { data, response, error in
        guard let data = data, error == nil { return completion(nil) }
    }

    if let response = response as? HTTPURLResponse,
       (200..<300).contains(response.statusCode) {
        guard let decodedData = try? JSONDecoder().decode(T.self, from: data) else { return complition(nil) }
        completion(decodedData)
    } else {
        return complition(nil)
    }.resume()
}
~~~



