# ITBookShelf
Swift, Combine, IT Book Store, pagination, image cache  
Standalone Application without thrird party library  
Local Swift Package Manager  

사용된 API는 이곳을 참고 [IT Bookstore API](https://api.itbook.store)

## 목적

서드파티 라이브러리 없이 Standalone 작동하는.  
Combine 기반의 MVVM 어플리케이션.  
  
Search, SearchResult View 컨트롤러 하이라키의 이해.  
Smooth한 pagination scrollview (tableview, collectionview)  

### 과제
로컬 데이터 저장을 더욱 generic하고 의존성을 배제 하도록 작성.

## CoreModular

**Feature**와 별도로 통신, 유틸리티 기능들을 모아둔  
Local SPM(Swift Package Manager)  
Local SPM을 이용하여 빈번 하게 사용 되는 기반/공용 기능을 담당함    
해당 기능의 수정이 없을 시 빌드 시간을 줄이는데 큰 역할을 함  

### ViewModelStream, ViewModelType  

https://github.com/kickstarter/ios-oss

MVVM Style을 기반으로 더적은 타이핑과 
구현과 인터페이스의 완전한 분리 
원본의 단방향 이벤트, 데이터 전달과 은닉성
유지를 목표료 개발 됨.

**sample code**
```
Class SomeClass {
  class ViewModel {
      // MARK: - Lazy properties
      lazy var inputs: Inputs = { Inputs(base: self) }()
      lazy var outputs: Outputs = { Outputs(base: self) }()
  }
}
...

extension SomeController.ViewModel: ViewModelStream {
    typealias ViewModel = BookShlefViewController.ViewModel
    /// Implement only functions, if possible
    struct Inputs: ViewModelStreamInternals {
        private unowned var base: ViewModel
        init(base: ViewModel) { self.base = base }
    }
    /// Implement only functions, if possible
    struct Outputs: ViewModelStreamInternals {
        private unowned var base: ViewModel
        init(base: ViewModel) { self.base = base }
    }
}

fileprivate typealias Inputs = SomeController.ViewModel.Inputs
fileprivate typealias Outputs = SomeController.ViewModel.Outputs

extension Inputs {
    /// event binder, these property types must not be relay and subject
    var someValueObservable: SomeObservable { base.someValue }
    func fetchSomething() { base.fetchSomething() }
    func configureSomeDepandancy(_ depandancy: Depandancy) { base.depandancy = depandancy }
}

extension Outputs {
    var item: SomeObservable { base.item }
    var error: SomeObservable<Error, Never> { base.error }
    var someValue: String { base.someValue }
}

```

### 1. Combine을 중점으로 한 통신 모듈
**sample code**
```
Remote<Some Codable>.somefunction().asObservable()
.compactMap({ $0 })
.sink(receiveCompletion: { [unowned self] in
    switch $0 {
        case .failure(let error):
            // handle someerror
        case .finished: print($0)
            // handle finished event
    }
}, receiveValue: { [unowned self] in
    handle receiveValue
}).cancel()
```


## Feature

**Base**

공용으로 사용되는 Base ViewController와  
TableView Cell을 구성 하기 위한 BaseCell Protocol
(MVVM의 View 기반 피쳐)

**Model**

MVVM의 **Model**에 해당하는 피쳐로  
CoreModular의 통신모듈(**Remote**)을 확장하여  
각 통신 인스턴스를 생성 함
Model은 Codable을 프로토콜을 따름.

```
protocol BookShelf: Codable {
    var total: String { get }
    var error: String { get }
    var books: [Model.Book] { get }
}

protocol BookInfo: Codable & Equatable {
    var title: String { get }
    var subtitle: String { get }
    var isbn13: String { get }
    var price: String { get }
    var image: String { get }
    var url: String { get }
}

    struct Book: BookInfo, Equatable {
        var title: String
        var subtitle: String
        var isbn13: String
        var price: String
        var image: String 
        var url: String
    }
```

#### View 공통

MVVM 규칙에 따라 ViewModel을 갖추고  
View의 Event를 감시하여 Model의 변화를 요청하고  
요청 결과에 따른 Model의 갱신과 그에 따른  
View 의 갱신 을 수행함.

**BookSehlf**

1. 사용자가 검색했던 책이 없는 경우 신규 책 리스트를 불러와서 표시함.
2. 사용자가 검색했던 책이 저장 되어있는 경우 리스트를 불러와서 표시함.
3. 이미지는 각 아이템 마 URL 문자열로 제공 되며 이미지는 비동기처리됨.  
(CoreModular에 속해있는 ImageCache 관련 기능 사용)
4. 책 검색 기능 제공


**Search Result**

**Book Detail**

**PDF**

**ETC Extensions**
