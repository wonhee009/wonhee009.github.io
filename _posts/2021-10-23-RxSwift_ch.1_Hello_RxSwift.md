---
toc: true
toc_sticky: true
title: "RxSwift - ch.1 Hello RxSwift"
exerpt: "RxSwift - ch.1 Hello RxSwift"
categories: RxSwift
tags: Swift rxSwift
date: 2021-10-23 17:00:00

---

# Intro

[Raywenderlich](https://www.raywenderlich.com/sessions/new?return_path=%2F)로 RxSwift를 공부해보려 한다.

두근.... 두근.....

---

# Hello, RxSwift!

RxSwift에 대한 첫 설명은 다음과 같다.

> **RxSwift** *is a library for composing asynchrnous and event-based code by using observable sequences and functional style operators, allowing for parameterized execution via schedulers*.

RxSwift는 `observable`한 시퀀스와 `functional style` 연산자를 사용해 `asynchronous(비동기적)`이고 `event-based(이벤트 기반)` 코드를 작성하는 라이브러리이다.

`scheduler`를 통해 매개변수화된 실행을 허용한다.

<br>

위의 설명은 아래와 같다.

> **RxSwift**, in its essence, simplifies developing asynchronous programs by allowing your code to react to new data and process it in a sequential, isolated manner.

RxSwift는 코드가 **새로운 데이터에 반응**하고 **순차적이고 독립된** 방식으로 처리할 수 있도록 하여 **비동기식 프로그램 개발을 단순**화 했다.

<br>

기존에도 비동기 코드를 지원하기 위해 제공해주던 API들이 있다.

- NotificationCenter
- delegate Pattern
- GCD
- Closure
- Combine

보통 대부분의 클래스들은 비동기적으로 수행하고 모든 UI 요소들은 비동기적이다.

따라서 내가 작성한 코드가 어떤 순서로 동작하는지 가정하는건 무의미하다.



## 비동기 프로그래밍 용어집

### 1. State, and sepdifically, shared mutable stae

**State**는 정의하기 어려우므로 예를 들어보자.

- 노트북을 사고 처음 실행하면 정상적으로 실행된다.
- 며칠 혹은 몇 주 동안 사용하면 이상하게 작동하거나 갑자기 멈추기도 한다.
- 노트북을 처음 샀을때와 기간이 지난 지금 하드웨어와 소프트웨어는 그대로 유지되지만 변한건 **상태**뿐이다.
- 노트북을 재시동하거나 사용할 수록 메모리 상의 데이터, 디스크에 저장된 내용을 포함한 찌꺼기들이 노트북에 남게 된다.

--> 이런 것을 노트북의 **state**라고 할 수 있다.

--> 노트북을 사용하면서 데이터 교환 등이 이루어지고 남는 것들이 생기면서 상태가 변한다.



### 2. 명령형 프로그래밍

명령형 프로그래밍은 명령문을 사용하여 프로그램의 상태를 변경하는 프로그래밍 패러다임이다.

- 강아지와 놀 때 명령형 언어를 사용하는 것처럼 (ex. 누워! 죽은 척!) 명령형 코드를 사용하여 작업을 수행하는 시기와 방법을 앱에 정확하게 알려준다.
- 명령 코드는 컴퓨터가 이해하는 코드와 유사하다.
- CPU가 하는 모든 일은 간단한 명령의 시퀀스를 따르는 것이다.

문제는 인간이 복잡한 비동기식 앱을 위한 명령형 코드를 작성하는 것은 어렵다.

특히 공유 가능한 상태가 있을 때 더 그렇다.

아래의 코드를 통해 확인해보자.

```swift
 override func viewDidAppear(_ animated: Bool) {
 		super.viewDidAppear(animated)
 	
 		setupUI()
 		connectUIControls()
 		createDataSource()
 		listenForChanges()
 }
```

- 각각의 메서드들이 어떤 동작을 하는지 확인이 어렵다.

- viewController를 업데이트 하는지 아닌지도 확인이 어렵다.
- 각 메서드들이 순서대로 잘 실행될지도 보증할 수 없다.
- 누군가 메서드의 순서를 바꿨다면 앱이 다르게 동작할 수 있다.



### 3. Side effects

**Side effects**는 코드의 현재 scope에서 벗어나서 일어나는 모든 변화를 의미한다.

예를 들어, `2. 명령형 프로그래밍`의 코드 예시에서 `connectUIControls()`는 어떤 종류의 이벤트 핸들러를 일부 UI 구성 요소에 연결한다.

이로 인해 뷰의 상태가 변경되므로 **Side effects**가 발생한다.

앱은 `connectUIControls()`를 실행하기 전에 한 방향으로 동작하고 그 후에는 다르게 동작한다.

<br>

디스크에 저장된 데이터를 수정하거나 label 텍스트를 업데이트할 때마다 **Side effect**가 발생한다.

**Side effects**가 나쁜 것은 아니지만 통제 가능해야 한다.

코드 어디에서 Side effect가 일어나는지 알 수 있어야 한다.



### 4. 선언형 코드

명령형 프로그래밍은 State의 변경이 자유롭다.

함수형 프로그래밍은 Side effect를 일으키는 코드를 최소화 하는걸 목표로 한다.

**RxSwift**는 명령형 코드와 함수형 코드에서 좋은 측면을 결합했다.

--> 명령형 코드(State 변화가 자유로움) + 함수형 코드(Side effect 최소화 = 예측 가능한 결과)

<br>

선언형 코드를 통해 동작을 정의한다.

RxSwift는 관련 이벤트가 있을 때마다 해당 동작을 실행하고 작업할 **변경 불가능한** 데이터 입력을 제공한다.

이렇게 하면 변경할 수 없는 데이터로 작업하고 순차적으로 실행할 수 있다.



### 5. Reactive systems

반응형 시스템이란 의미는 추상적이고, 다음과 같은 특성의 대부분 또는 전부를 나타내는 iOS 앱을 다루게 된다.

- Responsive: 항상 UI를 최신 상태로 유지하며 가장 최근의 앱 상태를 표시한다.
- Resilient: 각각의 행동들은 독립적으로 정의되며 에러 복구를 위해 유연하게 제공된다.
- Elastic: 코드는 다양한 작업 부하를 처리하며 종종 lazy pull-driven 데이터 수집, 이벤트 제한 및 리소스 공유와 같은 기능을 구현한다.
- Message driven: 구성요소는 메시지 기반 통신을 사용하여 재사용 및 고유한 기능을 개선하고, 라이프 사이클과 클래스 구현을 분리한다.

---

# Foundation of RxSwift

Rx 코드의 세 가지 구성 요소는 `Observable`, `Operator`, `Scheduler`이다.



## Observable

`Observable<Element>`는 Rx 코드의 기초이다.

`Element` 형태의 데이터 snapshot을 **전달**할 수 있는 이벤트 시퀀스를 비동기적으로 생성하는 기능이다.

즉, 시간이 지남에 따라 다른 개체에서 내보내는 이벤트 혹은 값을 구독할 수 있다.

**Observable** 클래스를 사용하면 하나 이상의 `observer`가 실시간으로 모든 이벤트에 반응하고 앱의 UI를 업데이트하거나 새로운 데이터를 처리하고 활용할 수 있다.

<br>

`ObservableType`프로토콜(Observable이 준수하는)은 매우 간단하다.

**Observable**은 세 가지 유형의 이벤트만 내보낼 수 있다. (`observer`는 이를 받을 수 있다.)

- next event
  - 최신 데이터 값을 **전달**하는 이벤트이다.
  - `observer`가 value를 받는 방법이다.
  - Observable은 종료 이벤트가 방출될 때까지 이런 값을 무한하게 방출할 수 있다.
- completed event
  - 이벤트 시퀀스를 성공으로 종료한다.
  - Observable이 수명 주길르 성공적으로 완료했으며 추가 이벤트를 방출하지 않는다.
- error event
  - Observable은 오류와 함께 종료되고 추가 이벤트를 방출하지 않는다.

위의 Observable 이벤트는 `Observable` 혹은 `Observer`의 본질에 대해 어떤 가정도 하지 않는다.

따라서 delegate 프로토콜이나 클로저를 사용할 필요가 없다.



### Finite observable sequences

일부 Observable 시퀀스는 0개, 1개 이상의 값을 방출하고 나중에는 성공적으로 종료되거나 오류와 함께 종료된다.

iOS 앱에서 파일 다운로드 코드를 생각해보자.

- 먼저 다운로드를 시작하고 들어오는 데이터들을 observing하기 시작한다.
- 계속해서 파일 데이터를 받는다.
- 네트워크 연결이 끊어진다면 다운로드는 멈출 것이고 연결은 에러와 함께 일시정지 된다.
- 또는, 성공적으로 모든 파일 데이터를 다운로드 할 수 있다.

이런 흐름은 위의 Observable의 생명주기와 일치한다.

<br>

위의 흐름을 RxSwift 코드로 표현하면 아래와 같다.

```swift
API.download(file: "http://www...")
   .subscribe(
     onNext: { data in
      // Append data to temporary file
     },
     onError: { error in
       // Display error to user
     },
     onCompleted: {
       // Use downloaded file
     }
   )
```

- `API.download(file: )`은 네트워크를 통해 들어오는 데이터 값을 방출하는 `Observable<Data>` 인스턴스를 반환한다.
- `onNext` 클로저로 `next` 이벤트를 받을 수 있다.
  - 위의 코드에서는 받은 데이터를 temporary file에 저장한다.
- `onError` 클로저로 `error` 이벤트를 받을 수 있다.
- 최종적으로 `onCompleted` 클로저로 `completed` 이벤트를 받을 수 있다.



### Infinite observable sequences

파일 다운로드와 같이 자연스럽게든 강제로든 종료되어야 하는 활동과 달리 단순히 무한한 시퀀스가 있다.

UI 이벤트는 무한히 관찰 가능한 시퀀스이다.

<br>

예를 들어 앱의 기기 방향 변경에 반응하는 데 필요한 코드가 있다.

- `NotificationCenter`에서 `UIDeviceOrientationDidChange` 알림에 대한 observer를 추가한다.
- 방향 변경을 처리하기 위해 메서드 콜백을 제공해야 한다.
- UIDevice에서 현재 방향을 가져와 최신 값에 따라 반응해야 한다.

<br>

위에서 살펴본 기기 방향 전환에는 끝이 없다.

즉, 디바이스가 존재하는 한 방향 전환은 연속적으로 일어난다.

결국 이런 시퀀스는 무한하기 때문에 항상 최초값을 가지고 있어야 한다.

사용자가 기기를 회전하지 않는 경우가 발생할 수 있지만 그렇다고 해서 이벤트가 종료된 것은 아니다.

다만, 아직 이벤트가 발생하지 않은 것일 뿐이다.

<br>

위의 흐름을 RxSwift 코드로 표현하면 아래와 같다.

```swift
UIDevice.rx.orientation
  .subscribe(onNext: { current in
    switch current {
    case .landscape:
      // Re-arrange UI for landscape
    case .portrait:
      // Re-arrange UI for portrait
    }
  })
```

- `UIDevice.rx.orientation`은 `Observable<Orientation>`을 생성하는 가상의 컨트롤 프로퍼티이다.
- 이를 `subscribe`함으로써 현재 디바이스 방향에 따라서 UI를 업데이트 할 수 있다.
- 해당 이벤트에서는 `onError`, `onCompleted`가 발생할 수 없으므로 생략한다.



## Operators

`ObervableType`과 `Observable` 클래스의 구현에는 비동기 비동기 작업 및 이벤트 조작을 추상화하는 많은 메서드가 포함되어 있고, 이는 함께 구성하면 더 복잡한 논리를 구현할 수 있도록 설계되어 있다.

이러한 메서드들은 매우 독립적이고 구성 가능하기 때문에 연산자(Operator)라고 부른다.

**Operator**는 대부분 비동기식 입력을 받아 Side effect 없이 출력만 생성하기 때문에 퍼즐처럼 맞춰서 사용할 수 있다.

<br>

예를 들어 `(5 + 6) * 10 - 2`라는 식을 생각해보자.

- `*`, `()`, `+`, `-` 같은 연산자를 통해 데이터에 적용하고 결과를 가져와서 해결될 때까지 표현식을 계속 처리한다.

이와 비슷하게 표현식이 최종값으로 도출 될 때까지 `Observable`이 방출한 값에 Rx 연산자를 적용하여 side effect를 만들 수 있다.

<br>

위에서 살펴 본 방향 전환 코드에 Rx Operator를 적용시킨 코드이다.

```swift
UIDevice.rx.orientation
  .filter { $0 != .landscape }
  .map { _ in "Portrait is the best!" }
  .subscribe(onNext: { string in
    showAlert(text: string)
  })
```

`UIDevice.rx.orientation`이 `.landscape` 혹은 `.portrait`  값을 생성할 때마다 RxSwift는 필터를 적용하고 방출된 데이터를 매핑한다.

- `Observable<Orientaion>`에서 `.portrait, .landscape, .portrait`을 방출했다. --> `.portrait, .landscape, .portrait`
- `filter`에 의해서 `.landscape`가 아닌 값만 방출한다. -->  `.portrait, .portrait`
  - 만약 디바이스 방향이 `.landscape`인 경우는 `filter`에서 이벤트가 방출되지 않으므로 `subscribe`가 실행되지 않는다.
- `map` 연산자에 의해서 `Orientation` 타입은 `String`으로 변환된다.
- `subscribe`를 통해 결과로 `next` 이벤트를 구현한다. `map`에 의해 변환된 `String` 값을 전달하고 해당 텍스트로  aleret을 화면에 출력한다.

연산자는 항상 데이터를 입력으로 받아 결과를 출력하므로 쉽게 연결이 가능하다.



## Schedulers

스케줄러는 Rx에서 `dispatch queue`와 동일하지만 훨씬 강력하고 사용하기 쉽다.

특정 작업의 실행 컨텍스트를 정의할 수 있다.

RxSwift의 스케줄러는 99%의 상황에서 사용 가능하도록 사전에 정의되어 있으므로 스케줄러를 만들 필요가 거의 없다.

스케줄러는 초반에 보지 않아도 된다고 하니 나중에 다시 보기로 한다.



## App architecture

RxSwift는 기존의 앱 아키텍쳐에 영향을 주지 않는다.

주로 이벤트나 비동기 데이터 시퀀스 등을 주로 처리하기 때문이다.

MVC, MVP, MVVM 등 다양한 패턴에서도 자유롭다.



다만 RxSwift와 MVVM은 아주 잘 어울린다.

ViewModel을 사용하면 `Observable<T>` 속성을 노출할 수 있으며 ViewController의 UIKit에 직접 바인딩 가능하다.

이렇게 하면 데이터를 UI에 바인딩하고 표현하기가 매우 간단해진다.



## RxCocoa

RxCocoa는 UIKit 및 Cocoa 개발을 지원하는 RxSwift의 동반 라이브러리이다.

---

# Outro

본격적으로  RxSwift에 대해 공부하기 전에 기초?를 봤다.

흠... 이렇게 보면 쉬운거 같은데 막상 해보면 또 다르겠지...

