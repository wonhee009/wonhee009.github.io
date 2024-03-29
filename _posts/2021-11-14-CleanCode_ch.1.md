---
toc: true
toc_sticky: true
title: "Clean Code - ch.1"
exerpt: "Clean Code - ch.1"
categories: CleanCode
tags: CleanCode
date: 2021-11-14 13:00:00
---

# Intro

Clean Code 책을 읽어보기로 해서 기록할 겸 한 장씩 남겨보려고 한다.

거창한건 아니고 그냥 읽으면서 나도 좀 더 명심할거... 아니면 느낀점? 정도 남기려고 한다.

---

제대로 명시한 요구사항은 코드만큼 **정형적**이고 테스트 케이스로 사용해도 좋다.

-> TDD 스터디 했을때 굉장히 많이 나왔던 말인거 같다.

-> TC를 만들어 가는 과정이 요구사항을 좀 더 명확하게 하고 명세의 애매한 부분들을 찾는다는 느낌을 받았었는데 여기서도 그에 비슷한 얘기가 나왔다.

<br>

나쁜 코드는 개발 속도를 떨어트린다.

-> 클린 코드는 비용을 절감할 뿐 아니라 전문가로서 살아남는 길이다.

<br>

좋은 코드를 사수하는 일은 프로그래머의 책임이다.

<br>

프로젝트의 기한을 놓치지 않는 방법 또한 코드를 깨끗하게 유지하는 습관에서 이뤄질 수 있다.

<br>

그렇다면 깨끗한 코드는?

- 우아하고 효율적인 코드
  - 논리가 간단해야 버그가 숨어들지 못한다.
  - 의존성을 최대한 줄여야 유지보수가 쉽다.
  - 오류는 전략에 의거해 철저하게 처리되어야 한다.
  - 한 가지만 잘 해야한다.
  - 즉, 세세한 사항까지 꼼꼼하게 처리하는 코드이다.
- 단순하고 직접적이며 잘 읽혀야 한다.
  - 설계자의 의도를 숨기지 않는다.
  - 명쾌한 추상화와 단순한 제어문으로 가득하다.
  - 즉, 코드는 추측이 아니라 사실에 기반한다.
- 작성자가 아니 사람도 읽기 쉽고 고치기 쉽다.
  - 의미 있는 이름을 붙인다.
  - 특정 목적을 달성하는 방법을 하나만 제공한다.
  - 의존성은 최소이고 의존성을 명확히 정의한다.
- 객체가 여러 기능을 수행한다면 여러 객체로 나눈다.
- 메서드가 여러 기능을 수행한다면 메서드 추출 리팩토링 기법을 적용해 기능을 명확히 기술하는 메서드 하나와 실제로 수행하는 메서드 여러 개로 나눈다.
- 중복을 줄이고 표현력을 높이고 초반부터 간단한 추상화를 고려한다.

**즉, 중복을 피하라, 한 기능만 수행해라, 작게 추상화 해라**

읽으면서 놀랄 일이 없어야 하고, 독해하느라 고생하지 않아야 한다.

---

# Outro

1장이니 간단하게 클린 코드란 무엇인지에 대해서 얘기한다.

사실 들어보면 다 맞는 말이고 늘 듣는 말이다.

하지만 이런거 지키기 힘들다.... 이유야 많겠지만.... (리소스 부족 등...) 근데 여기서는 그 이유조차도 클린 코드로 해결할 수 있다고 한다.

항상 코드의 역할을 잘 생각하고 그 역할을 명확하게 보이도록 작성하자.