---
layout: post
---
<br>

### Websocket
<br>

Websocket은 HTML5의 표준 사양으로 정의된 프로토콜로, 단일 TCP 연결을 통한 Full-Duplex(전이중) 통신이다. 전통적인 HTTP 통신은 클라이언트가 서버에 요청하는 단방향 구조였기에, 실시간 데이터를 처리하기에 어려움이 있었지만, Websocket은 양방향 구조이기에 매번 클라이언트와 서버를 연결할 필요가 없어졌다.

<br>

| **구분** | **HTTP** | **WebSocket** |
| --- | --- | --- |
| **통신 방식** | 단방향 (Request/Response) | 양방향 (Bi-directional) |
| **상태 유지** | Stateless (매번 새로 연결) | Stateful (연결 유지) |
| **헤더 크기** | 큼 (매번 쿠키, 헤더 포함) | 매우 작음 (최초 연결 후 오버헤드 최소화) |

<br>

![websocket.png](./img/websocket.png)

<br>

1. Client가 Handshake 요청을 보낸다.
2. 서버가 응답하면서, websocket protocol로 전환된다.
3. websocket frame 형식에 맞게, 서로 통신을 주고받는다.
4. 클라이언트가 연결을 종료한다.
5. 서버가 websocket protocol을 종료한다.

<br>
<br>

### STOMP

<br>

websockt은 양방향 통로만 제공할 뿐, 그 안에 담긴 데이터가 무엇인지(채팅인지, 시스템 알림인지) 서버는 알 방법이 없다. 그래서 websocket frame의 payload(데이터 본문) 안에 STOMP라는 규격을 집어넣어 사용한다. 

<br>

```java
SEND
destination:/topic
content-type:application/json
subscription:sub-0
message-id:a70f7353-3f37-5abe-0381-6cd3450a8442-2
content-length:49
```

<br>

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry messageBrokerRegistry) {
        messageBrokerRegistry.enableSimpleBroker("/topic");
        messageBrokerRegistry.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry stompEndpointRegistry) {
        stompEndpointRegistry.addEndpoint("/chat");
    }
}

```

<br>

messageBrokerRegistry.enableSimpleBroker("/topic");  해당 코드를 통해 클라이언트는 SimpleBroker를 거쳐서 구독자에게 메세지를 전달할 수 있다.

<br>

messageBrokerRegistry.setApplicationDestinationPrefixes("/app"); 해당 코드를 통해 클라이언트가 /app/… 으로 요청하면, SimpAnnotationMethod가 Controller의 @MessageMapping을 찾는다.

<br>

```java
@Controller
public class ChatController {

    @MessageMapping("/message")
    @SendTo("/topic")
    public Message sendMessage(Message message) {
        return message;
    }
```

<br>

Client가 /app/message를 통해서 메세지를 보내면, SimpAnnotationMethod가 @MessageMapping(”/message”)를 찾는다. 로직 처리 후, message를 반환하면, 
Spring은 이 반환 값을 가로채서 @SendTo(’’/topic’)에 적힌 목적지를 설정한다. 그리고 이 메시지를 BrokerChannel에 넣는다. 그리고 SimpleBroker가 BrokerChannel에 있는 메세지의 목적지를 확인하여, 해당 경로를 구독하고 있는 모든 구독자들에게 메세지를 전달한다. 

<br>

![stomp.png](./img/stomp.png)
