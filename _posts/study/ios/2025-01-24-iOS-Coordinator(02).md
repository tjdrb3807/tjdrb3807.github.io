---
layout: post
title: "Book_Kitty - Coordinator(03)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce 
BookKitty 프로젝트에서는 MVVM-C 패턴을 도입하여 화면 전환 로직을 Coordinator에서 관리하도록 설계했습니다.
이전 블로그에서 소개한 TabBarCoordinator가 앱의 전체적인 화면 구조를 관리하는 역할을 했다면,
이번 글에서는 특정 기능에 집중한 AddQuestionCoordinator의 역할과 의존성 주입 방식을 설명하겠습니다.

## AddQuestionCoordinator 구현
```swift
final class AddQuestionCoordinator: Coordinator {
    weak var finishDelegate: CoordinatorFinishDelegate?
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    
    private let questionRepository: QuestionRepository
    private let bookRecommendationService: BookRecommendationService
    
    private var newQuestionViewController: NewQuestionViewController!
    private let disposeBag = DisposeBag()
    
    init(
        navigationController: UINavigationController,
        questionRepository: QuestionRepository,
        bookRecommendationService: BookRecommendationService
    ) {
        self.navigationController = navigationController
        self.questionRepository = questionRepository
        self.bookRecommendationService = bookRecommendationService
    }
    
    func start() {
        showNewQuestionScene()
    }
}
```

### Coordinator에서 의존성 주입
* 왜 Coordinator에서 의존성을 주입하는가?
  * 일반적으로 ViewController 내부에서 필요한 Repository나 Service 객체를 직접 생성하는 경우가 많습니다. 그러나 이런 방식은 **강한 결합(Strong Coupling)**을 초래하여 유지보수성과 테스트 용이성을 떨어뜨립니다.
* Coordinator를 통한 의존성 주입을 활용하면
  * ViewController는 UI 및 사용자 이벤트 처리에 집중할 수 있습니다.
  * 화면 전환을 담당하는 Coordinator가 필요한 의존성을 생성하여 주입함으로써 ViewController나 ViewModel은 구체적인 구현을 알 필요 없이 동작할 수 있습니다.
* 단위 테스트(Testability) 개선
  * ViewModel이 Repository나 Service로직을 직접 다루면 테스트가 어렵습니다.
  * 그러나 Coordinator에서 주입하면, Mock 객체를 사용하여 테스트가 가능해집니다.

```swift
final class MockQuestionHistoryRepository: QuestionHistoryRepository {

    let mockQuestionList = [
        QuestionAnswer(
            //...
        ),
    ]
    func fetchQuestion(by _: UUID) -> QuestionAnswer? {
        mockQuestionList[0]
    }

    func saveQuestionAnswer(data _: QuestionAnswer) -> UUID? {
        UUID()
    }

    func deleteQuestionAnswer(uuid _: UUID) -> Bool {
        true
    }

    func fetchQuestions(offset _: Int, limit _: Int) -> Single<[QuestionAnswer]> {
        Single.create { observer in
            observer(.success(self.mockQuestionList))
            return Disposables.create()
        }
    }

    func fetchQuestions(offset _: Int, limit _: Int) -> [QuestionAnswer] {
        mockQuestionList
    }

    func recodeAllQuestionCount() {
        print(mockQuestionList.count)
    }
}
```

### 첫 번째 화면: 질문 입력 
사용자가 질문을 입력하면, NewQuestionViewModel의 Relay가 화면 전환을 트리거합니다.
```swift
final class NewQuestionViewModel: ViewModelType {
    struct Input {
        let submitButtonTapped: Observable<String>

        let leftBarButtonTapTrigger: Observable<Void>
    }

    let disposeBag = DisposeBag()

    let navigateToQuestionResult = PublishRelay<String>()
    let navigateToRoot = PublishRelay<Void>()

    func transform(_ input: Input) -> Output {
        input.submitButtonTapped
            .bind(to: navigateToQuestionResult)
            .disposed(by: disposeBag)

        input.leftBarButtonTapTrigger
            .bind(to: navigateToRoot)
            .disposed(by: disposeBag)

        return Output()
    }
}
```

### AddQuestionCoordinator에서 화면 이동 처리 
```swift 
private func showNewQuestionScene() {
    let viewModel = NewQuestionViewModel(questionRepository: questionRepository)
    newQuestionViewController = NewQuestionViewController(viewModel: viewModel)

    viewModel.navigateToQuestionResult
        .withUnretained(self)
        .bind(onNext: { owner, question in
            owner.showQuestionResultScene(with: question)
        }).disposed(by: disposeBag)

    viewModel.navigateToRoot
        .withUnretained(self)
        .bind(onNext: { owner, _ in
            owner.finish()
        }).disposed(by: disposeBag)

    navigationController.pushViewController(newQuestionViewController, animated: true)
}
```

### 질문 결과 화면: 도서 추천
```swift
private func showQuestionResultScene(with question: String) {
    let recommendationService = BookRecommendationKit(
        naverClientId: Environment().naverClientID,
        naverClientSecret: Environment().naverClientSecret,
        openAIApiKey: Environment().openaiAPIKey
    )
    let repository = LocalBookRepository()
    let questionHistoryRepository = LocalQuestionHistoryRepository()
    let questionResultViewModel = QuestionResultViewModel(
        userQuestion: question,
        recommendationService: recommendationService,
        bookRepository: repository,
        questionHistoryRepository: questionHistoryRepository
    )

    let questionResultViewController =
        QuestionResultViewController(viewModel: questionResultViewModel)
        
    questionResultViewModel.navigateToBookDetail
        .withUnretained(self)
        .bind(onNext: { owner, book in
            owner.showBookDetailScene(with: book)
        }).disposed(by: disposeBag)

    questionResultViewModel.navigateToQuestionHistory
        .withUnretained(self)
        .bind(onNext: { owner, _ in
            owner.finish()
        }).disposed(by: disposeBag)
    navigationController.pushViewController(questionResultViewController, animated: true)
    
    navigationController.viewControllers = navigationController.viewControllers.filter {
        !($0 is NewQuestionViewController)
    }
}
```



