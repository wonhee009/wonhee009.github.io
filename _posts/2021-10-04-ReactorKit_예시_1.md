---
toc: true
toc_sticky: true
title: "ReactorKit 예시 - 1"
exerpt: "ReactorKit example - 1"
categories: ReactorKit
tags: Swift rxSwift ReactorKit
date: 2021-10-04 20:22:00
---

## Intro

앞에서 알아본 `ReactorKit`으로 정말 간단한 예제를 만들어보려 한다.

[카운터 앱](https://github.com/ReactorKit/ReactorKit/tree/master/Examples/Counter)은 해당 링크를 참고했다.

---

## Counter App

![reactorKit_4](/assets/images/reactorKit_4.png)

간단하게 카운터 앱을 만들어보자.

<br>

### 명세서

아래는 카운터 앱의 명세서이다.

- 가운데에 숫자를 표시하는 `label`이 있다.
- 숫자를 표시하는 `label` 왼쪽에는 - `button`이, 오른쪽에는 + `button`이 있다.
- -`button`을 tap하면 숫자가 1 감소한다.
- +`button`을 tap하면 숫자가 1 증가한다.
- 숫자가 증감하는 동안 `label` 아래의 `indicator` 애니메이션이 실행된다.



### Step1

화면을 구성했다.

![reactorKit_5](/assets/images/reactorKit_5.png)



### Step2

`CounterViewReactor` 파일을 생성했다.

이름에서 보이듯 해당 앱의 `Reactor` 역할을 할 파일이므로 `Reactor` 프로토콜을 채택한다.

```swift
final class CounterViewReactor: Reactore { ... }
```

`Reactor`는 `Action`, `State`,  `Mutation`,  `initialState`를 선언해줘야한다.



#### Action

```swift
enum Action {
  case increase
  case decrease
}
```

`Action` 사용자 인터랙션을 나타낸다.

카운터 앱에서는 값을 '증가/감소' 시키는 인터랙션만 있으므로 `increase`, `decrease`를 선언한다.



#### State

```swift
struct State {
  var value: Int
  var isLoading: Bool
}
```

`State`는 현재 뷰의 상태를 표현한다.

카운터 앱에서는 숫자를 표시할 `value`와 indicatorView의 애니메이션 여부를 알려줄 `isLoading`이 필요하다.



#### Mutation

```swift
enum Mutation {
  case increaseValue
  case decreaseValue
  case setLoading(Bool)
}
```

`Mutation`은 'represent state changes'라고 한다.

즉, state의 변화를 표현한다.

+`button`을 tap하면 value `State`가 증가해야 하므로 `increaseValue`를 정의한다.

-`button`을 tap하면 value `State`가 감소해야 하므로 `decreaseValue`를 정의한다.

indicatorView 애니메이션 여부는 `Bool` 값을 통해 확인한다.



#### initialState, init()

```swift
let initialState: State

init() {
  self.initialState = State(
    value: 0,
    isLoading: false
  )
}
```

초기 `State`를 정의해준다.

`CounterViewReactor`는 class이기 때문에 `init()`할 때 모든 프로퍼티가 정의되어야 한다.

따라서 `init()`에서 `initialState`를 정의해준다.

처음에는 숫자를 표시하는 `label`은 0 을 표시해야 하고, indicatorView의 애니메이션은 정지된 상태여야 한다.



### Step3

`Reactor`는 전달받은 `Action`을 2가지 단계를 거쳐 `State`로 변환한다.

- `mutate()`: `Action` -> `Mutation`
- `reduce()`: `Mutation` -> `State`



#### mutate()

```swift
func mutate(action: Action) -> Observable<Mutation> {
  switch action {
    case .increase:
    return Observable.concat([
      Observable.just(Mutation.setLoading(true)),
      Observable.just(Mutation.increaseValue).delay(.milliseconds(500), scheduler: MainScheduler.instance),
      Observable.just(Mutation.setLoading(false))
    ])
    case .decrease:
    return Observable.concat([
      Observable.just(Mutation.setLoading(true)),
      Observable.just(Mutation.decreaseValue).delay(.milliseconds(500), scheduler: MainScheduler.instance),
      Observable.just(Mutation.setLoading(false))
    ])
  }
}
```

increase/decrease `Action`이 전달되면 `Mutation`으로 변환시킨다.

rxSwift는 나중에 다루도록 하고, 코드를 봐보자.

위에서 정의한 `enum Mutation`을 사용한다.

- `Mutation.setLoading(true)`로 indicatorView의 애니메이션을 시작한다.

- `Mutation.increaseValue`로 value를 증가시키고, `delay(.milliseconds(500))`으로 지연을 준다.

- `Mutation.setLoading(false)`로 indicatorView의 애니메이션을 멈춘다.

위의 과정을 담은 `Mutation`을 생성하고 반환한다.



#### reduce()

```swift
func reduce(state: State, mutation: Mutation) -> State {
  var state = state
  switch mutation {
    case .increaseValue:
	    state.value += 1
    case .decreaseValue:
    	state.value -= 1
    case let .setLoading(isLoading):
    	state.isLoading = isLoading
  }
  return state
}
```

`reduce()`는 이전의 `State`와 전달받은 `Mutation`으로 새로운 `State`를 생성한다.

`case .increaseValue`를 보자.

`state.value += 1`이 실행되는데 이는, `이전의 State.value += 1`이라고 볼 수 있다.

`return state`에서의 state는 새로운 `State`로 볼 수 있다.



### Step4

`Reactor` 구성을 완료했으니 `View`를 구성하자.

Reactor에서 View를 구성하기 위해서는 `View` 프로토콜을 채택해야 한다.

해당 예제는 Storyboard로 화면을 구성했으므로 `StoryboardView` 프로토콜을 채택한다.

```swift
final class ViewController: UIViewController, StoryboardView { ... }
```

<br>

rxSwift를 사용하므로 `var disposeBag = DisposeBag()`을 선언해준다.



#### reactor

`ViewController`는 `StoryboardView`를 채택함으로써 `reactor`라는 프로퍼티가 생겼다.

`View`의 밖에서 `reactor` 프로퍼티를 지정해준다.

```swift
viewController.reactor = CounterViewReactor()
```



#### bind()

```swift
func bind(reactor: CounterViewReactor) {
  // Action
  increaseButton.rx.tap
  	.map { Reactor.Action.increase }
  	.bind(to: reactor.action)
  	.disposed(by: disopseBag)
  
  decreaseButton.rx.tap
  	.map { Reactor.Action.decrease }
  	.bind(to: reactor.action)
  	.disposed(by: disposeBag)
  
  // State
  reactor.state.map { $0.value }
  	.distinctUntilChanged()
  	.map { "\($0)" }
  	.bind(to: valueLabel.rx.text)
  	.disposed(by: disposeBag)
  
  reactor.state.map { $0.isLoading }
  	.distinctUntilChanged()
  	.bind(to: activityIndicatorView.rx.isAnimating)
  	.disposed(by: disposeBag)
}
```

`View`는 `Action`을 전달하기도 하고 `State`를 받기도 한다.

따라서 `bind()`에 `Action`과 `State` 둘 다에 대해서 구현해야 한다.

---

## Outro

간단한 예제를 구현해 봤다.

역시 직접 구현해보는게 이해하는데 더 빠른거 같다.

<br>

구현하면서 느낀점은

- `View`, `Reactor`로 구분을 하니 역할 분담이 명확하게 잘 되는거 같다.

- 일정한 패턴이 있다보니 손에 익으면 작성이 좀 더 쉬워질거 같다.

- 한 화면에서 관리하는 `Action`, `State`가 많아지면 `bind()` 관리가 어려워지지 않을까?


다음에는 좀 더 복잡한 예제를 구현하면서 `ReactorKit`에 대해 느낀 점을 정리해봐야겠다.

