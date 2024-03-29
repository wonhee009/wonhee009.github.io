---
toc: true
toc_sticky: true
title: "RxSwift - ch.3 Subject"
exerpt: "RxSwift - ch.3 Subject"
categories: RxSwift
tags: Swift rxSwift
date: 2021-10-25 23:00:00

---

# Intro

여기서는 Subject에 대해서 알아보자.

이전에 알아본 Observable과 어떻게 다를까

<br>

**Observable**은 읽기 전용이다.



---

# What is an Observables?

Observable은 Rx의 핵심이다.

Rx를 보다보면  `Observable`, `Observable sequence`, `sequence`가 등장하는데 이는 모두 같은 것이다.

Observable의 가장 중요한 점은 **비동기적(asynchronous)**이라는 것이다.

Observable은 일정 기간 동안 **이벤트를 생성하고 이를 방출(emit)**한다.

이벤트는 숫자나 사용자가 정의한 타입 인스턴스와 같은 값을 포하고, tap과 같은 제스처도 인식할 수 있다.

이를 가장 잘 보여주는 건 marble diagram이다.

[marble diagram을 볼 수 있는 사이트](https://rxmarbles.com)



## Lifecycle of an observable

![rxSwift_1](/assets/images/rxSwift_1.png)

위의 marble diagram을 보면 **observable**이 세개의 이벤트를 방출하고 있다.

이 이벤트들은 **next 이벤트**에 의해서 방출된다.



![rxSwift_2](/assets/images/rxSwift_2.png)

위의 marble diagram처럼 오른쪽의  `|`수직 막대는 이벤트 종료를 의미한다.

**observable**은 3개의 tap 이벤트를 발생시킨 후 종료되었다.

이를 **완료(completed) 이벤트**라고 한다.

**observable**은 완료 이벤트가 발생하면  더 이상 아무 것도 방출할 수 없다.



![rxSwift_3](/assets/images/rxSwift_3.png)

빨간색 `X` 표시는 에러를 의미한다.

**Observable**이 종료된 것은 위와 동일하지만 **error 이벤트**에 의해서 종료되었다.



- Observable은 어떤 element를 가지는 **next 이벤트**를 방출한다.
- 종료 이벤트(**error 이벤트** 혹은 **completed 이벤트**)가 발생할 때까지 위의 작업이 계속 될 수 있다.
- Observable이 종료되면 더이상 이벤트를 생성할 수 없다.



### Event

이벤트는 **Enum**으로 표현된다.

```swift
/// Represents a sequence event.
///
/// Sequence grammar: 
/// **next\* (error | completed)**
@frozen public enum Event<Element> {
    /// Next element is produced.
    case next(Element)

    /// Sequence terminated with an error.
    case error(Swift.Error)

    /// Sequence completed successfully.
    case completed
}
```

- **next 이벤트**에는 `Element` 인스턴스가 포함되어 있다.
- **error 이벤트**에는 Swift.Error 인스턴스가 포함되어 있다.
- **error 이벤트와 completed 이벤트는** 데이터가 포함되어 있지 않고 단순히 중지하는 이벤트이다.

---

# Creating observables(1)

## just

(이번 챕터의 예제는 플레이그라운드를 사용했다.)

```swift
example(of: "just, of, from") {
     // 1
     let one = 1
     let two = 2
     let three = 3
     
     //2
     let observable:Observable<Int> = Observable<Int>.just(one)
 }
```

1. 예제에서 사용할 Int 상수를 정의했다.
2. `one` 상수와 `just` 메서드로 `Int` 타입의 **Observable sequence**를 만들었다.

`just`는 Observable의 타입 메서드이다.

단 하나의 Element를 포함하는 Observable sequence를 생성한다.

Rx에서는 메서드를 `operator`라고 한다. (즉, `just`도 operator이다.)



![rxSwift_4](/assets/images/rxSwift_4.png)

[출처: rx홈페이지-just](http://reactivex.io/documentation/ko/operators/just.html)

rx문서로 `just` 정의에 대해 좀 더 알아보자.

> `just`는 Element를 해당 Element 타입을 방출하는 **Observable로 변환**하는 operator이다.

이렇게만 보면 이해가 잘 안 간다.



```swift
public static func just(_ element: Element) -> Observable<Element> {
        Just(element: element)
    }
```

`just` operator는 위처럼 구성되어 있다.

`static`으로 정의되어 있으므로 **타입 메서드**임을 알 수 있다.

`just`의 파라미터 타입과 return 타입을 보면 위의 정의를 이해할 수 있다.

```swift
Observable<Int>.just(one)
```

위의 예시에 적용해 본다면 `just` operator는 Int 값을 Int 타입의 **이벤트를 방출하는 Observable로 변환** 해준다.



`just`에 nil을 전달하면 nil을 Element로 방출하는 Observable이 반환된다.

이는 빈 Observable을 반환하는게 아니다.

(마치 nil과 ""의 차이 같네...)

빈 Observable을 반환하고 싶다면 `Empty` operator를 사용할 수 있다.



## of

```swift
example(of: "just, of, from") {
  // 1
  let one = 1
  let two = 2
  let three = 3
     
  //2
  let observable = Observable.of(one, two, three)
  let observable2 = Observable.of([one, two, three])
}
```

- observable의 타입은 `Observable<Int>`
- observable2의 타입은 `Observable<[Int]>`

`of` operator는 주어진 값들의 타입추론을 통해 **Observable sequence**를 생성한다.

어떤 배열을 Observable array로 만들고 싶다면 `of` operator에 배열을 넣어주면 된다.



```swift
print("😀 of operator: Observable<Int>")
Observable.of(1,2,3).subscribe(onNext: { array in
    print(array)
}).disposed(by: disposeBag)

print("😀 of operator: Observable<[Int]>")
Observable.of([1,2,3]).subscribe(onNext: { array in
    print(array)
}).disposed(by: disposeBag)

😀 of operator: Observable<Int>
1
2
3
😀 of operator: Observable<[Int]>
[1, 2, 3]
```

위의 코드에서 보이듯 Int 타입의 Element를 `of` operator에 넣어주면 `1 2 3`으로 Int타입의 시퀀스가 방출된다.

Int 타입의 배열을 `of` operator에 넣어주면 배열 자체의 시퀀스가 방출된다.



### just? of?

```swift
Observable.just([1,2,3]).subscribe(onNext: { array in
    print(array)
}).disposed(by: disposeBag)

[1, 2, 3]
```

`just` operator에 Int 배열을 넣어줘도 동일한 결과를 얻을 수 있다.

`just`,`of` 모두 반환 타입도 `Observable<[Int]>`로 동일하다.



`just` 구현부에 가보면 파라미터에 아래와 같이 작성되어 있다.

> Single element in the resulting observable sequence.

파라미터는 observable sequence로 만들고 싶은 **single element**이다.



아직 이 부분은 명확하지 않은거 같다.

`just`	는 단일 값을 방출할때 사용하는건 이해하겠는데 단일 값을 어디까지로 봐야할까?

[1, 2, 3] <- 이런 배열도 하나의 배열로써 단일 값을 볼 수 있지 않을까? 라는 생각이 든다.



## from

```swift
example(of: "just, of, from") {
  // 1
  let one = 1
  let two = 2
  let three = 3
     
  //2
  let observable = Observable.from([one, two, three])
 }
```

- observable의 타입은 `Observable<Int>`

`from` operator는 배열 요소들을 하나씩 방출한다.

`from`은 배열만 입력할 수 있다.



![rxSwift_5](/assets/images/rxSwift_5.png)

[출처: rx홈페이지-from](http://reactivex.io/documentation/operators/from.html)



```swift
public static func from(_ array: [Element], scheduler: ImmediateSchedulerType = CurrentThreadScheduler.instance) -> Observable<Element> {
  ObservableSequence(elements: array, scheduler: scheduler)
}
```

`from` operator는 위와 같이 구성되어 있다.

파라미터 타입을 보면 배열만을 입력 받는걸 알 수 있다.

---

# Subscribing to observables

iOS 개발자라면 `NotificationCenter`에 익숙할 것이다.(observer에게 알림을 브로드캐스팅)

```swift
 let observer = NotificationCenter.default.addObserver(
   forName: .UIKeyboardDidChangeFrame,
   object: nil,
   queue: nil
 ) { notification in
 	// Handle receiving notification
 }
```

위의 코드는 클로저로 `UIKeyboardDidChangeFrame` notification observer 코드이다.



RxSwift의 observable을 subscribing하는 방식은 위와 비슷하다.

- `addObserver()` 대신 `subscribe()`를 사용한다.
- `NotificationCenter`는 `.default` 싱글톤 인스턴스에서만 가능했지만, Rx Observable은 다르다.
- 핵심은 Observable은 **subscriber가 있어야만 이벤트를 방출하거나 작업을 수행**한다.

**Observable은 sequence의 정의일 뿐**이며 Observable이 subscriber되어야만 이후의 스텝이 진행된다.



Observable의 구현은 Swift 반복문에서 `.next()`를 구현하는 것과 비슷하다.

```swift
 let sequence = 0..<3
 var iterator = sequence.makeIterater()
 while let n = iterator.next() {
 	print(n)
 }
 
 /* Prints:
  0
  1
  2
  */
```

Observable subscriber는 이보다 쉽다.

Observable이 **방출하는 이벤트 타입에 대해 handler를 추가**할 수 있다.

--> `.subscribe(onNext:_, onError:_, onCompleted:_)`로 이벤트 타입별로 핸들러 추가 가능하다.

Observable은 `next`, `error`, `completed` 이벤트를 방출한다.

- `next`: 핸들러로 방출된 Element를 보낸다.
- `error`: error 인스턴스가 포함된다.



## subscribe()

```swift
Observable.of(1, 2, 3)
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)

next(1)
next(2)
next(3)
completed
```

Observable은 각각의 Element에 대해서 `next` 이벤트를 방출한다.

종료 직전 `completed` 이벤트를 방출한다.



이벤트로 감싸진 Element가 아니라 직접 Element에도 접근할 수 있다.

```swift
Observable.of(1, 2, 3)
    .subscribe { event in
        if let element = event.element {
            print(element)
        }
    }
    .disposed(by: disposeBag)

1
2
3
```



Event 구현부를 보면 아래와 같이 `element` 프로퍼티를 확인할 수 있다.

```swift
public var element: Element? {
  if case .next(let value) = self {
    return value
  }
  return nil
}
```

`next` 이벤트에만 `element`가 존재하고 옵셔널이다.



## subscribe(onNext:)

위에서 살펴본 `subscribe()`에 대해서 더 알아보자.

RxSwift에는 Observable에서 방출하는 각 이벤트 유형에 subscribe operator가 존재한다.



위에서 살펴본 코드는 아래와 같이 작성할 수 있다.

```swift
Observable.of(1, 2, 3)
    .subscribe(onNext: {
        element in print(element)
    })
```

`next` 이벤트만 처리하고 나머지(`error`, `completed` 이벤트)에 대해서는 무시한다.

`onNext` 클로저는 `next` 이벤트의 element를 받으므로 위의 코드처럼 element를 따로 뽑지 않아도 된다.



### subscribe()? Subscribe(onNext:)

위에서 알아본 바와 두 메서드 모두  `next` 이벤트의 element를 사용할 수 있다.

그렇다면 둘은 어떤 차이가 있을까?



```swift
public func subscribe(_ on: @escaping (RxSwift.Event<Self.Element>) -> Void) -> RxSwift.Disposable

public func subscribe(onNext: ((Self.Element) -> Void)? = nil, onError: ((Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil) -> RxSwift.Disposable
```

`subscribe()`의 클로저 입력 타입은 `RxSwift.Event<Self.Element>`로 element가 이벤트로 감싸져 있다.

-> `subscribe()` 예제에서 `next(1)` 이런식으로 출력되는 이유이다.

`subscribe(onNext:)`에서 `next` 이벤트 부분을 보면 `Self.Element`로 element 자체를 받고 있다.

---

# Creating observables(2)

## empty()

element를 가지지 않는(갯수가 0인) Observable은 어떻게 만들 수 있을까.

`empty` operator를 통해 만들 수 있고 element를 방출하지 않고 `completed` 이벤트만 방출한다.



```swift
Observable<Void>.empty()
    .subscribe{ element in print(element)}
    .disposed(by: disposeBag)

completed
```

위의 코드에서 보듯 `empty` operator는 `completed` 이벤트만 방출한다.



empty observable은 어디에 쓰일까?

- 의도적으로 0개의 값을 가지 observable을 반환하고 싶을 때
- 즉시 종료할 수 있는 observable을 반환하고 싶을 때

사용할 수 있다고 한다.



### just(), empty()

그렇다면 `just`와 `empty` 구현부를 비교하면서 어떤 식으로 element를 반환하는지 알아보자.



```swift
public static func empty() -> Observable<Element> {
  return EmptyProducer<Element>()
}

final private class EmptyProducer<Element>: Producer<Element> {
  override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    observer.on(.completed)
    return Disposables.create()
  }
}
```

먼저  `empty`의 구현부이다.

`empty` operator는 `EmptyProducer` 클래스 인스턴스를 반환한다.

`EmptyProducer`의 `subscribe`를 보면 element 반환 없이 `completed` 이벤트를 방출하고 종료되는걸 확인할 수 있다.



```swift
public static func just(_ element: Element) -> Observable<Element> {
  return Just(element: element)
}

final private class Just<Element>: Producer<Element> {
  private let _element: Element
    
  init(element: Element) {
    self._element = element
  }
    
  override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    observer.on(.next(self._element))
    observer.on(.completed)
    return Disposables.create()
  }
}
```

`just`의 구현부이다.

`just` operator는 `Just` 클래스 인스턴스를 반환한다.

`Just` 인스턴스를 생성할때 `just` operator로 입력된 값을 주입한다.

`Just` 클래스의 `subscribe`를 보면 이벤트를 2번 방출하고 있다.

- `.next(self._element)` element 값을 주입한 `next` 이벤트를 방출한다.
- `.completed` 이벤트를 방출한다.

(`just` operator는 단 하나의 element만 방출하고 종료되므로 `next` 이벤트는 한 번만 호출된다.)



### empty와 타입

`empty` 예시 코드에 아무 element도 방출하지 않고 종료될텐데 `Observable<Void>`로 왜 타입을 지정해주는지 궁금했다.

그래서 아래와 같이 실험을 해봤다.

```swift
Observable<Void>.empty()
    .subscribe{ element in print(element) }
    .disposed(by: disposeBag)

Observable.empty()
    .subscribe{ element in print(element) }
    .disposed(by: disposeBag)
```

`completed`가 두 번 출력될 줄 알았는데 타입을 지정해준 Observable에서만 출력되었다.

책을 쭉 읽는데 `empty` operator를 사용할 때에는  Observable의 타입을 지정해줘야 한다고 한다.

앞에서 살펴본 `just`, `of`, `from`과 다르게 가지고 있는  element가 없어 타입 추론을 하지 못한다고 한다.



```swift
public static func empty() -> Observable<Element> {
  return EmptyProducer<Element>()
}
```

`empty` 구현부를 보면서 좀 이해할 수 있었다.

`empty`의 반환 타입을 보면  `Observable<Element>`로 타입이 필요하다.



흠.... 근데 어차피 아무런 element를 방출하지 않고 즉시  completed되는 Observable인데 Void 말고 다른 타입을 쓸 일이 있을까?



## never()

```swift
Observable<Void>.never()
    .subscribe{ element in print(element) }
    .disposed(by: disposeBag)
```

`never` operator는 `completed` 이벤트도 방출되지 않는다.

즉, 무한의 Observable을 반환한다.



```swift
public static func never() -> Observable<Element> {
  return NeverProducer()
}

final private class NeverProducer<Element>: Producer<Element> {
  override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    return Disposables.create()
  }
}
```

`never`의 구현부를 확인해보자.

`NeverProducer` 클래스 인스턴스를 반환한다.

`NeverProducer`의 `subscribe`를 확인해보면 어떤 element도 이벤트도 방출하지 않는걸 확인할 수 있다.



## range()

```swift
Observable.range(start: 1, count: 10)
    .subscribe{ element in print(element) }
    .disposed(by: disposeBag)

next(1)
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
```

`range` operator는 이름 그대로이다.

`start`로 시작 지점을 설정하고 `count`로 방출할 갯수를 지정한다.

---

# Disposing and terminating

Observable은 subscription되기 전까지 아무런 동작도 하지 않는다.

즉, Observable은 subscribe되어야만 동작을 한다.

이 동작은 `error`나 `completed`로 종료되기 전까지 반복된다.

그렇다면 subscribe를 취소하여 Observable의 동작을 종료할 수 있지 않을까?



## dispose()

```swift
// subscribe 구현부
public func subscribe(_ on: @escaping (RxSwift.Event<Self.Element>) -> Void) -> RxSwift.Disposable

// dispose() 관련 예제
example(of: "dispose") {
  // 1
  let observable = Observable.of("A", "B", "C")
     
  // 2
  let subscription = observable.subscribe({ (event) in
     // 3
     print(event)
  })
  subscription.dispose()
 }
```

Dispose() 관련 예제를 살펴보자.

- String Observable을 생성한다.

- Observable을 subscribe한다. -> `subscribe()`는 `Disposable`을 반환한다.
- 방출된 이벤트들을 출력한다.

`dispose()`를 통해 subscribe를 취소할 수 있다.

subscribe를 취소하거나  dispose한 뒤에는 이벤트 방출이 정지된다.

(`dispose()`하면 completed가 방출된 후 종료된다.)



## DisposeBag()

각 subscription을 개별적으로 관리하는건 효율적이지 못해서 `DisposeBag`을 이용할 수 있다.

`DisposeBag`은 (보통  `disposed(by:)`으로 추가된 ) disposables를 가지고 있다.

Dispose bag이 할당 해제되려고 할 때 각각 `dispose()`를 호출한다.

```swift
 example(of: "DisposeBag") {
     
   // 1
   let disposeBag = DisposeBag()
     
   // 2
   Observable.of("A", "B", "C")
   	.subscribe{ // 3
      print($0)
    }
   .disposed(by: disposeBag) // 4
 }
```

DisposeBag() 관련 예제를 살펴보자.

`dispose()` 예제와 크게 다르지 않다.

`DisposeBag()` 인스턴스를 만들어주고 `subscribe`로 반환된 `Disposable`을 `disposed(by:)`로 `disposeBag`에 추가한다.



`dispose bag`에서 subscription 추가를 잊거나 수동으로 `dispose()` 호출을 잊어버린다면 메모리 누수가 일어난다.



## Disposable과 DisposeBag

위에서 살펴본 `Just` 구현부와 `DisposeBag` 구현부를 확인해보자.

(아직 완벽하게 구현부를 본건 아녀서 겉핥기식으로만 보자....)

```swift
// 예제
Observable.just([1, 2, 3])
    .subscribe { element in print(element) }
    .disposed(by: disposeBag)

// Just 구현부
final private class Just<Element>: Producer<Element> {
  override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    observer.on(.next(self._element))
    observer.on(.completed)
    return Disposables.create()
  }
}

// disposed(by:)
extension Disposable {
  public func disposed(by bag: DisposeBag) {
    bag.insert(self)
  }
}

// DisposeBag
public final class DisposeBag: DisposeBase {
  private var _disposables = [Disposable]()
  
  public func insert(_ disposable: Disposable) {
    self._insert(disposable)?.dispose()
  }
    
  private func _insert(_ disposable: Disposable) -> Disposable? {
    self._lock.lock(); defer { self._lock.unlock() }
    if self._isDisposed {
      return disposable
   }

    self._disposables.append(disposable)
    return nil
  }

  /// This is internal on purpose, take a look at `CompositeDisposable` instead.
  private func dispose() {
    let oldDisposables = self._dispose()

    for disposable in oldDisposables {
      disposable.dispose()
    }
  }
}
```

`Just`의 `subscribe`를 보면 ` Disposable`을 반환하고 있다.

-> 예제에서 `.subscribe`에서 ` Disposable`을 얻을 수 있다.   

   

`Disposable`에 `disposed(by:)`가 구현되어 있다.

-> `.subscribe`에서 반환된 `Disposable`에 `disposed(by:)`를 호출한다.

`disposed(by:)`를 통해 입력 받은 `DisposeBag`에 `Disposable` 자체를 넣어준다.



`DisposeBag`에는  `Disposable` 배열 프로퍼티가 존재한다.

-> `disposed(by:)`는 해당 배열에 Disposable을 넣는다.

배열에 들어간 `Disposable`을 메모리 해제한다.

---

# Creating observables(3)

## create()

```swift
Observable.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)

    return Disposables.create()
}
.subscribe{ element in print(element) }
.disposed(by: disposeBag)

1
2
3
```

`create`는 탈출 클로저로 `AnyObserver -> Disposable` 형태이다.

`AnyObserver`는 제네릭 타입으로 Observable sequence에 값을 추가할 수 있다.

추가한 값은 subscriber에 방출된다.



`.onNext`는 `on(.next(_:))`와 같은 의미이다.

`onCompleted()`는 `on(.completed)`와 같은 의미이다.



```swift
Observable.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onCompleted()
    observer.onNext(3)

    return Disposables.create()
}
.subscribe(
    onNext: { print($0) },
    onError: { print($0) },
    onCompleted: { print("Completed") },
    onDisposed: { print("Disposed") }
).disposed(by: disposeBag)

1
2
Completed
Disposed
```

위의 코드를 확인해보자.

`onNext`로 1, 2, 3을 방출했지만 2까지만 나온다.

`completed` 이벤트가 발생하여 dispose되었기 때문에 3은 방출되지 않는다.



```swift
enum TestError: Error {
    case error
}

Observable.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onError(TestError.error)
    observer.onCompleted()
    observer.onNext(3)

    return Disposables.create()
}
.subscribe(
    onNext: { print($0) },
    onError: { print($0) },
    onCompleted: { print("Completed") },
    onDisposed: { print("Disposed") }
).disposed(by: disposeBag)

1
2
error
Disposed
```

위의 예시와 동일하게 2까지만 출력된다.

`error` 이벤트가 발생한 후  dispose되었기 때문에 뒤의 다른 이벤트들은 방출되지 않는다.

---

# Traits

`Trait`은 일반적인 Observable보다 좁은 범위의 Observable로 선택적으로 사용할 수 있다.

`Trait`을 사용해 가독성을 높일 수 있다.



## Single

`Single`은 `success(value)` 또는 ` error(error)` 이벤트를 방출한다.

`success(value)`는 `next` 이벤트와 `completed` 이벤트의 조합니다.

성공 혹은 실패로 확인 가능한 프로세스에 사용한다.



```swift
public typealias Single<Element> = PrimitiveSequence<SingleTrait, Element>

public enum SingleEvent<Element> {
    case success(Element)
    case error(Swift.Error)
}

extension PrimitiveSequenceType where Trait == SingleTrait {
  public static func create(subscribe: @escaping (@escaping SingleObserver) -> Disposable) -> Single<Element> {
        let source = Observable<Element>.create { observer in
            return subscribe { event in
                switch event {
                case .success(let element):
                    observer.on(.next(element))
                    observer.on(.completed)
                case .error(let error):
                    observer.on(.error(error))
                }
            }
        }
        
        return PrimitiveSequence(raw: source)
    }
}
```

`Single`의 구현부를 봐보자.

`SingleEvent`로 `success(value)`, `error(error)`가 있음을 알 수 있다.

`primitiveSequenceType`을 확인해보면 `success`는 `next` 이벤트로 element를 방출한 뒤  `completed` 이벤트를 방출한다.

(생긴게 좀 Result랑 비슷하게 생겼다....)



## Completable

`Completable`은 `completed` 혹은 ` error(error)` 이벤트만 방출한다.

즉, 어떠한 값도 방출하지 않는다.

파일 쓰기 같은 작업이 성공적으로 완료 혹은 실패인지에만 관심 있을 경우 사용한다.



```swift
public typealias Completable = PrimitiveSequence<CompletableTrait, Swift.Never>

public enum CompletableEvent {
    case error(Swift.Error)
    case completed
}
```

`Completable`의 구현부를 봐보자.

`CompletableEvent`로 `error(error)`, `completed`가 있음을 알 수 있다.



## Maybe

`Maybe`는 `Single`과 `Completable`을 섞은거와 같다.

`success(value)`, `completed`, `error(error)`를 방출할 수 있다.

성공, 실패 여부와 더해서 출력된 값도 필요할 수도 있을때 사용한다.



### 사용법

```swift
example(of: "Single") {
  // 1
  let disposeBag = DisposeBag()
     
  // 2
  enum FileReadError: Error {
    case fileNotFound, unreadable
  }
     
  // 3
  func loadText(from name: String) -> Single<String> {
    // 4
    return Single.create{ single in
       // 4 - 1
       let disposable = Disposables.create()
             
       // 4 - 2
       guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
         single(.error(FileReadError.fileNotFound))
         return disposable
       }
             
       // 4 - 3
       guard let data = FileManager.default.contents(atPath: path) else {
         single(.error(FileReadError.unreadable))
         return disposable
       }
             
       // 4 - 4
       single(.success(contents))
       return disposable
   }
  }
}
```

1: dispose bag 생성

2: Error 정의

3: 디스크 파일로부터 텍스트 불러와서 `Single`을 반환하는 메서드

4: `Single`을 생성하고 반환

​	4-1: `create`의 `subscribe` 클로저는 `disposable`을 반환하므로 `disposable` 생성

​	4-2: 파일명 경로를 받아오고 파일이 없다면 `single`에 `error`를 추가하고 `disposable` 반환

​	4-3: 파일로부터 데이터를 받아오고 파일을 읽을 수  없다면 `single`에 `error`를 추가하고 `disposable` 반환

​	4-4: 원하는 동작을 수행했으니 `single`에 `success(value)`를 추가하고 `disposable` 반환



```swift
loadText(from: "Copyright")
   .subscribe{
     switch $0 {
       case .success(let string):
           print(string)
       case .error(let error):
           print(error)
     }
   }
.disposed(by: disposeBag)
```

이런 식으로 사용할 수 있다.

볼수록 Result 같다...

---

# Outro

Observable에 대해서 알아봤다.

내용이 상당히 많다.

이후에 더 볼 내용들이 있지만 우선은 여기서 마무리하고 이후에 다시 정리해야지...

