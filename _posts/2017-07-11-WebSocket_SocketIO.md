---
layout: post
title: WebSocket과 Socket.io
---

 WebSocket과 Socket.io를 공부하기 이전에 알아야할 개념들이 있다.
HTTP 통신은 client의 요청(request)에 server가 응답한다. 즉, server는 클라이언트가 요청하기 전까지는 client에게 메시지를 전달 할 수 있는 방법이 없다. 정리하면, HTTP 프로토콜은 server-client간의 단방향 커뮤니케이션만 가능하다.
하지만 생각해보자. 일반적인 웹 사이트에서 정적인 데이터들로만 구성되어 있다면 사용자가 브라우저를 새로고침하지 않는 이상 데이터는 갱신되지 않을 것이다. 만약 서버가 데이터 갱신에 맞춰 클라이언트들에게 request없이 response를 전달한다면 이러한 문제를 해결될 것 같다.
 그래서 HTTP 프로토콜을 이용해 위와 같이 양방향 커뮤니케이션이 가능하게끔 하는 기법들이 있다.

### polling
 클라이언트가 서버에 주기적으로 요청을 보내고 즉시 응답을 받는다. 서버는 새로운 데이터를 클라이언트에 응답한다.
주기적으로 변경사항이 있는지 확인하는 구조이므로 쓸모없는 client의 request가 발생하고 주기에 따라 갱신되는 정도가 다르므로 real-time 반영이라고 볼 수 없는 맹점이 있다.

### long polling
  서버가 클라이언트의 요청을 바로 응답하지 않고 필요한 시점에 응답하고 다시 커넥션을 맺는 기법이다. real-time으로 응답은 가능하지만 client가 수가 많아질수록 커넥션을 맺고 있는 server의 thread수도 많아 질 것이고 이는 성능에 중대한 영향을 미친다. 하나의 tomcat에 300개의 thread를 사용 가능하다면 하나의 tomcat으로 처리할 수 있는 사용자는 300명이다.

### streaming
  클라이언트의 요청에 대해서 서버는 완벽한 응답을 수행하지 않는다. - 응답완료라 명시하지 않는다. 즉 커넥션을 계속 열어두고 서버가 준비될때마다(일정량의 데이터가 쌓이면) response를 보내고 커넥션은 계속 유지한다.

위 방법들은 HTTP 프로토콜을 이용해 실시간으로 서버가 클라이언트에게 PUSH 하는 것과 같은 느낌을 주지만 많은 완벽하지 못하다.
이를 해결하고자 HTML5 표준의 일부로 WebSocket 프로토콜이 만들어지게 되었다.

```
webSocket이 기존의 일반 TCP Socket과 다른 점은 최초 접속이 일반 http request를 통해 
handshaking과정을 통해 이루어 진다는 점이다. http request를 그대로 사용하기 때문에 
기존의 80, 443 포트로 접속을 하므로 추가로 방화벽을 열지 않고도 양방향 통신이 가능하고, 
http 규격인 CORS적용이나 인증등의 과정을 기존과 동일하게 가저갈 수 있는것이 장점이다.

출처: http://adrenal.tistory.com/20 [시나몬 브레드]
```

자 그럼 본격적으로 WebSocket에 대해 알아보자.

### WebSocket
싱글 소켓을 활용해 양방향 커뮤니케이션이 가능하게 하는 프로토콜이다. 웹소켓은 확장가능한 실시간 어플리케이션을 만들 수 있는 기능을 제공한다. 웹 소켓은 서버 오버헤드를 줄이고 복잡도를 극적으로 감소시킨다.
웹소켓 커넥션을 맺기 위해서는 핸드쉐이킹 과정에서 upgrade HTTP 프로토콜을 사용해야한다.

```
Example 1: The WebSocket handshake (browser request and server response)
GET /text HTTP/1.1
Upgrade: WebSocket
Connection: Upgrade
Host: www.websocket.org
...
HTTP/1.1 101 WebSocket Protocol Handshake
Upgrade: WebSocket
Connection: Upgrade
...
```

한번 커넥션을 맺은 이후에는 서버-클라이언트가 양방향 통신을 수행 할 수 있다.

이[포스팅](http://java.sys-con.com/node/1315473)을 보면 long polling 방식에 기반해 동작하는 RabbitMQ와 WebSocket을 사용한 애플리케이션의 성능을 잘 비교해두었다. 

하지만 WebSocket 또한 치명적인 단점이 있으니.. 버전에 따라 지원하지 않는 브라우저가 있다는 것이다.
- 브라우저 버전별 webSocket 지원 여부 ([출처](http://caniuse.com/#search=webSocket))
![Image](https://media.oss.navercorp.com/user/1083/files/e06e9004-66b3-11e7-9ed4-c4b76a6c0719)


이를 해결하고자, Socket.io가 나타났다.

### Socket.io
[Socket.io](https://socket.io/)는 실시간 웹 어플리케이션을 위한 자바스크립트 라이브러리이다. 서버-클라이언트간 양방향 통신이 가능하며
서버, 클라이언트 두가지 파트가 존재한다.
서버사이드 라이브러리는 Node.js기반에서 동작하고 클라이언트사이드 라이브러리는 브라우저에서 사용 가능하다.
Socket.io는 클라이언트의 지원 여부에 따라 WebSocket, FlashSocket, AJAX Long Polling, AJAX Multi part Streaming, IFrame, JSONP Polling 를 하나의 API로 추상화 한 것이다. 해당 라이브러리를 사용하는 입장에서는 브라우저에 관계없이 동일한 인터페이스를 사용할 수 있게끔 해준다.
하지만 WebSocket과 달리 표준이 아니며 단지 Node.js의 모듈이다. (글 작성자 2011년 기준, 현재는 Node.js외의 다른 프레임웍에서 제공할수도 있다.)

