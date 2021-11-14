---
toc: true
toc_sticky: true
title: "ReactorKit - 2"
exerpt: "ReactorKit - Advanced"
categories: ReactorKit
tags: Swift rxSwift ReactorKit
date: 2021-10-01 16:35:00
---

## Intro

[이전 포스트](https://wonhee009.github.io/reactorkit/ReactorKit/)에서 ReactorKit의 컨셉을 알아봤다.

이번 포스트에서는 ReactorKit의 Advanced에 대해서 알아보려 한다.

---

## Advanced

### Global States

Redux와 달리 ReactorKit은 전역 앱 상태를 정의하지 않는다.

즉, 전역 상태를 관리하기 위해 무엇이든 사용할 수 있다.

`BehaviorSubject`, `PublishSubject`, `Reactor`를 사용할 수 있다.

ReactorKit은 전역 상태에서 사용되도록 강제하지 않으므로 App의 특정 기능에서만 ReactorKit을 사용할 수 있다.

<br>

`Action` -> `Mutation` -> `State` 흐름에는 전역 상태가 없다.

전역 상태를 `Mutation`으로 변환하려면 `transform(mutation:)`을 사용해야 한다.



> 처음에 위의 글을 봤을때 잘 이해가 가지 않았다.
>
> 생각해보니 ReactorKit의 큰 흐름은 `Action` -> `Mutation` -> `State`인데 이는 전역적으로 사용하고 있는 변수 등등과는 연관성이 없다.
>
> 따라서 전역적으로 사용하고 있는 변수 등등을 어떤 식으로 처리해 ReactorKit의 흐름으로 편입시킬 수 있는지에 대한 얘기인거 같다.



전역적으로 사용되는 `currentUser`라는 BehaviorSubject가 있다고 가정해보자.

`currentUser`는 현재 인증된 사용자에 대해서 저장하고 있고, 인증된 사용자가 변경될때마다 `Mutation.setUser(User?)`를 방출해야 한다.

```swift
var currentUser: BehaviorSubject<User> // global state

func transform(mutation: Observable<Mutation>) -> Observable<Mutation> {
  return Observable.merge(mutation, currentUser.map(Mutation.setUser))
}
```

- `currentUser`는 BehaviorSubject로 전역적으로 사용되고 있다.
- `transform(mutation:)`을 통해 ReactorKit의 흐름으로 편입시킨다.



위의 코드를 통해 view가 reactor에 action을 보내고 currentUser가 변경될 때마다 Mutaion이 발생한다.



### View Communication

현재 다수의 view들의 communication에 callback 클로저나 delegate 패턴이 익숙하다.

ReactorKit은 `reactive extension`을 사용하도록 권장한다.

ControlEvent의 가장 일반적인 예는 UIButton.rx.tap이다.

custom view를 UIButton 또는 UILabel로 처리하는 경우를 예로 들 수 있다.

<br>

![reactorKit_3](/assets/images/reactorKit_3.png)

메시지를 표시하는 `ChatViewController`가 있다고 생각해보자.

`ChatViewController`는 `MessageInputView`를 가지고 있다.

User가 `MessageInputView`에서 보내기 버튼을 탭하면 텍스트가 `ChatViewController`로 전송되고 `ChatViewController`가 reactor의 action에 바인딩된다.

<br>

```Swift
extension Reactive where Base: MessageInputView {
  var sendButtonTap: ControlEvent<String> {
    let source = base.sendButton.rx.tap.withLatestFrom(...)
    return ControlEvent(events: source)
  }
}
```

`MessageInputView`의 reactive extension이다.



```swift
messageInputView.rx.sendButtonTap
  .map(Reactor.Action.send)
  .bind(to: reactor.action)
```

`ChatViewController`에서는 위와 같은 방식으로  extension을 사용할 수 있다.



### Testing

ReactorKit은 테스트를 위한 기능이 내장되어 있다.

해당 기능을 사용해서 view와 reactor 모두 쉽게 테스트할 수 있다.

<br>

아래와 같은 것들을 테스트할 수 있다.

- View
  - Action: 주어진 유저 interaction을 적절한 action으로 reactor로 전송했는가
  - State: 따르고 있는 state에서 view의 프로퍼티가 제대로 설정되어 있는가
- Reactor
  - State: state가 적절하게 action으로 변경되었는가



#### View testing

view는 `stub` reactor로 테스트할 수 있다.

reactor는 action 로그를 기록하고 state를 강제로 변경할 수 있는 `stub` 프로퍼티를 가지고 있다.

reactor의 `stub`이 사용가능하다면 `mutate()`와 `reduce()` 모두 실행되지 않는다.

`stub`에는 다음과 같은 속성이 있다.

```swift
var state: StateRelay<Reactor.State> { get }
var action: ActionSubject<Reactor.Action> { get }
var actions: [Reactor.Action] { get } // recorded actions
```



#### Reactor testing

reactor는 독립적으로 테스트할 수 있다.

```swift
func testIsBookmarked() {
  let reactor = MyReactor()
  reactor.action.onNext(.toggleBookmarked)
  XCTAssertEqual(reactor.currentState.isBookmarked, true)
  reactor.action.onNext(.toggleBookmarked)
  XCTAssertEqual(reactor.currentState.isBookmarked, false)
}
```

<br>

가끔 state는 하나의 action에 의해 두 번 이상 변경된다.

예를 들어, `.refresh` 작업은 처음에는 `state.isLoading`을 true로 설정하고 새로 고침 후에는 false로 설정한다.

이런 경우 currentState로 `state.isLoading`을 테스트하기 어렵기 때문에 `RxTest` 또는 `RxExpect`를 사용해야 할 수도 있다.



#### Scheduling

state 스트림을 줄이고 observing하는데 사용되는 스케줄러를 지정하려면 스케줄러 프로퍼티를 정의한다.

이 대기열은 직렬 대기열이다.

기본 스케줄러는 `CurrentThreadScheduler`이다.



---

## Outro

ReactorKit의 Advanced에 대해서 알아봤다.

뭔가 알듯 말듯하다....ㅋㅋㅋㅋ

예시를 보면서 좀 더 알아봐야할거 같다.