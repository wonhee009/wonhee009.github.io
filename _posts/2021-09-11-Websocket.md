---
toc: true
toc_sticky: true
title: "WebSockets with Swift - 1"
exerpt: "Raywenderlich 'An Introduction to WebSockets' 따라하기"
categories: WebSocket
tags: WebSocket Swift Vapor
date: 2021-09-11 17:00:00
---

## Intro

[Raywenderlich](https://www.raywenderlich.com/13209594-an-introduction-to-websockets#toc-anchor-013) 로 웹소켓을 공부해보려 한다....ㅎ

웹소켓에 좋은 기억은 없지만...ㅋㅋㅋ 이번에는 잘 해봐야지...  

  

웹소켓은 **서버와 클라이언트 간의 양방향 통신**을 허용하는 네트워크 프로토콜이다.

Request-Response 패턴을 사용하는 HTTP와 달리 **웹소켓은 시점과 방향에 상관없이 메시지를 보낼 수 있다.**

따라서 웹소켓은 채팅과 같이 서버와 클라이언트 간에 지속적으로 대화해야하는 서비스에 자주 사용된다.

---

### WebSocket Connect

웹소켓 연결은 handshake에서 시작한다.

클라이언트는 일반 HTTP request에 `Upgrade: WebSocket`, `Connection: Upgrade` 라는 특별한 header와 인증과 같은 다른 필수 요청 데이터와 함께 보낸다.<br>



서버는 `HTTP 101 스위칭 프로토콜 상태 코드`를 클라이언트에 다시 보낸다.

`HTTP 101 스위치 프로토콜 상태 코드`와 함께 `Upgreade: WebSocket`, `Connection: Upgrade` header를 같이 보낸다.<br>



이런 과정을 거쳐 handshake가 완료되고 웹소켓 연결이 완료된다.<br>



#### WebSocket Message

웹소켓 프로토콜에서 데이터는 **프레임 시퀀스**를 사용하여 전송된다.

웹소켓 프레임은 HTTP 헤더와 비교할 수 있는 몇 가지 비트와 실제 메시지로 구성되어 있다.

메시지가 너무 크면 여러 프레임을 사용해 보낼 수 있다.<br>



- FIN: **단일 비트**로 메시지의 마지막 프레임인지 뒤에 더 많은 프레임이 있는지 나타낸다.
- opcode: **4비트**로 메시지의 유형과 페이로드 데이터를 처리하는 방법을 나타낸다.
- MASK: **단일 비트**로 메시지가 마스킹되었는지 여부를 나타낸다. 클라이언트-서버 메시지는 항상 마스킹되어야 한다.
- Payload length: **7/23/71비트**로 HTTP의 Content-Length 헤더와 마찬가지로 페이로드 데이터가 비트 단위로 얼마나 큰지 설명한다.
- Masking key: **4개의 선택적 비트**로 메시지를 마스킹하는데 사용한 키가 포함된다. MASK 비트가 0이면 생략 가능하다.
- Payload data: 나머지 메시지에는 실제 페이로드가 포함된다. 

---

### FIN

웹소켓에 대해서 간단하게 알아봤다.

Raywenderlich에 코드가 다 올라와 있지만 이거에 대해서 설명하는건 의미가 없다고 생각한다.

직접 코드를 짜고 내가 이해한 내용을 계속 적어볼까 한다.