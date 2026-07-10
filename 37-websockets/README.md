# 💬 WebSockets & Real-Time (Spring Boot) — Complete Guide

> **"HTTP is a one-way street: the client asks, the server answers. WebSockets provide a two-way, persistent connection, allowing the server to push real-time data to the client instantly."**

---

## 📑 Table of Contents

1. [HTTP vs WebSockets](#1-http-vs-websockets)
2. [What is STOMP?](#2-what-is-stomp)
3. [Setup & Configuration](#3-setup--configuration)
4. [Message Handling (`@MessageMapping`)](#4-message-handling-messagemapping)
5. [Broadcasting to Subscribers](#5-broadcasting-to-subscribers)
6. [Sending Messages to Specific Users](#6-sending-messages-to-specific-users)
7. [Client-Side Integration (JS/SockJS)](#7-client-side-integration-jssockjs)
8. [Securing WebSockets](#8-securing-websockets)
9. [Scaling WebSockets (External Broker)](#9-scaling-websockets-external-broker)
10. [Interview Questions & Answers (50+)](#10-interview-questions--answers-50)

---

## 1. HTTP vs WebSockets

**HTTP (Polling):**
*   Client: "Are there new messages?" → Server: "No."
*   Client: "Are there new messages?" → Server: "No."
*   Client: "Are there new messages?" → Server: "Yes, here is 1."
*(Inefficient, wastes bandwidth, high latency).*

**WebSockets:**
*   Client & Server perform a "handshake" and upgrade the HTTP connection to a TCP WebSocket connection.
*   The connection stays OPEN.
*   Server: "Hey client, here is a new message." (Pushed instantly!)
*(Highly efficient, low latency).*

**Use Cases:** Chat applications, live sports scores, real-time dashboards, multiplayer games.

---

## 2. What is STOMP?

WebSockets transmit raw text or binary data. There is no concept of "routes" or "headers" built-in. 

**STOMP** (Simple Text Oriented Messaging Protocol) sits on top of WebSockets. It acts like HTTP for WebSockets. It defines rules so you can have paths (like `/app/chat`), headers, and message bodies.

Spring Boot uses WebSockets + STOMP.

---

## 3. Setup & Configuration

**1. Dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

**2. Configuration**
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // The endpoint clients connect to (e.g., ws://localhost:8080/ws)
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS(); // Fallback mechanism for older browsers
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // Topics the server broadcasts to (Clients subscribe here)
        registry.enableSimpleBroker("/topic", "/queue");
        
        // Prefix for messages sent FROM client TO server
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```

---

## 4. Message Handling (`@MessageMapping`)

Just like `@RequestMapping` in MVC, we use `@MessageMapping` for WebSockets.

```java
@Controller
public class ChatController {

    // Client sends a message to "/app/chat.sendMessage"
    // The return value is automatically broadcast to "/topic/public"
    @MessageMapping("/chat.sendMessage")
    @SendTo("/topic/public")
    public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
        // You could save to DB here
        return chatMessage;
    }
}

// Simple DTO
public class ChatMessage {
    private String sender;
    private String content;
    private MessageType type; // CHAT, JOIN, LEAVE
    // getters/setters
}
```

---

## 5. Broadcasting to Subscribers

Sometimes you want to send a message to clients from a completely different part of your application (e.g., a REST controller, or a scheduled job), not just as a direct response to a WebSocket message.

Use `SimpMessagingTemplate`.

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final SimpMessagingTemplate messagingTemplate;

    // Call this method when an order is completed
    public void notifyUsers(String message) {
        // Broadcasts to all clients subscribed to "/topic/notifications"
        messagingTemplate.convertAndSend("/topic/notifications", message);
    }
}
```

---

## 6. Sending Messages to Specific Users

If you are building a private chat (1-to-1), you don't broadcast to `/topic/public`. You send it to a specific user.

**1. Configure User Destinations (Already done by default, `/user` prefix)**

**2. Controller Logic**
```java
@Controller
@RequiredArgsConstructor
public class PrivateChatController {

    private final SimpMessagingTemplate messagingTemplate;

    @MessageMapping("/chat.private")
    public void sendPrivateMessage(@Payload ChatMessage message) {
        // Target User, Destination, Payload
        messagingTemplate.convertAndSendToUser(
            message.getRecipient(), 
            "/queue/reply", 
            message.getContent()
        );
    }
}
```
*Note: The recipient client must subscribe to `/user/{username}/queue/reply`.*

---

## 7. Client-Side Integration (JS/SockJS)

How a frontend connects to your Spring Boot WebSocket.

```javascript
// 1. Connect
let socket = new SockJS('http://localhost:8080/ws');
let stompClient = Stomp.over(socket);

stompClient.connect({}, function (frame) {
    console.log('Connected: ' + frame);

    // 2. Subscribe (Listen for messages)
    stompClient.subscribe('/topic/public', function (message) {
        let chatMessage = JSON.parse(message.body);
        console.log(chatMessage.sender + ": " + chatMessage.content);
    });
});

// 3. Send a message
function sendMessage() {
    let msg = { sender: "Alice", content: "Hello World!" };
    stompClient.send("/app/chat.sendMessage", {}, JSON.stringify(msg));
}
```

---

## 8. Securing WebSockets

WebSockets bypass standard HTTP filters after the initial handshake.

1.  Secure the HTTP handshake using standard Spring Security (`http.authorizeHttpRequests()`).
2.  Pass the JWT token in the connection headers from the JS client.
3.  Implement a `ChannelInterceptor` in Spring to validate the token during the STOMP CONNECT phase.

```java
@Configuration
public class WebSocketSecurityConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    String token = accessor.getFirstNativeHeader("Authorization");
                    // Validate Token...
                    // Set Authentication in SecurityContext
                }
                return message;
            }
        });
    }
}
```

---

## 9. Scaling WebSockets (External Broker)

**The Problem:**
If you have 3 instances of your Spring Boot app behind a Load Balancer.
User A connects to App-1. User B connects to App-2.
If User A sends a message to `/topic/public`, only users on App-1 will receive it!

**The Solution:**
Replace the "Simple In-Memory Broker" with a Full-Featured External Broker (like RabbitMQ or ActiveMQ).

```java
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.setApplicationDestinationPrefixes("/app");
    
    // Instead of enableSimpleBroker:
    registry.enableStompBrokerRelay("/topic", "/queue")
            .setRelayHost("localhost")
            .setRelayPort(61613)
            .setClientLogin("guest")
            .setClientPasscode("guest");
}
```
*Now, App-1 forwards the message to RabbitMQ. RabbitMQ distributes it to App-2 and App-3, ensuring all users get the message.*

---

## 10. Interview Questions & Answers (50+)

### Beginner

**Q1: What are WebSockets?** A protocol providing full-duplex (two-way) communication channels over a single TCP connection.

**Q2: How does a WebSocket connection start?** With an HTTP Handshake requesting an "Upgrade" to the WebSocket protocol.

**Q3: What is Long Polling?** The client makes an HTTP request. The server holds the request open until data is available, sends it, and the client immediately reconnects. (A workaround before WebSockets existed).

**Q4: What is STOMP?** Simple Text Oriented Messaging Protocol. Adds routing and header semantics on top of raw WebSockets.

**Q5: What is SockJS?** A JavaScript library that provides a WebSocket-like object. If WebSockets fail (e.g., blocked by a corporate proxy), it falls back to Long Polling silently.

**Q6: What annotation handles incoming messages?** `@MessageMapping`.

**Q7: What annotation dictates where the return value goes?** `@SendTo`.

---

### Intermediate

**Q8: How do you send a message to a specific user?** Use `SimpMessagingTemplate.convertAndSendToUser()`.

**Q9: Why do WebSockets break when scaling to multiple instances?** Because the default SimpleBroker is in-memory. If User A is on Instance 1 and User B is on Instance 2, they cannot share messages.

**Q10: How do you fix the scaling issue?** Configure an external STOMP broker relay (like RabbitMQ or ActiveMQ) to synchronize messages across all instances.

**Q11: Can you use REST and WebSockets in the same Spring Boot app?** Yes, absolutely.

**Q12: How do you secure WebSockets with JWT?** Use a `ChannelInterceptor` to extract and validate the JWT from the `CONNECT` frame headers.

**Q13: What is the `destination` in STOMP?** The routing path (like a URL). e.g., `/topic/chat` or `/app/send`.

**Q14: Server-Sent Events (SSE) vs WebSockets?** SSE is one-way (Server to Client only). WebSockets are two-way. Use SSE for live stock tickers. Use WebSockets for chat.

---

### Rapid-Fire (Q16–Q50)

**Q15: Default port for WebSockets?** It uses standard HTTP/HTTPS ports (80/443).

**Q16: Protocol prefix for WebSockets?** `ws://` (unencrypted) or `wss://` (encrypted).

**Q17: Does WebSocket connection stay open?** Yes, it is persistent until explicitly closed by client or server.

**Q18: What is a `Payload`?** The body of the message. Annotated with `@Payload`.

**Q19: How do you handle path variables in WebSockets?** `@DestinationVariable`.

**Q20: What is `/topic` usually used for?** Pub-Sub broadcasting (1-to-Many).

**Q21: What is `/queue` usually used for?** Point-to-Point communication (1-to-1).

**Q22: What is `SimpMessagingTemplate`?** A Spring bean used to send messages to clients programmatically from anywhere in the app.

**Q23: How do you know when a user connects/disconnects?** Listen for `SessionConnectEvent` and `SessionDisconnectEvent` using `@EventListener`.

**Q24: What is the STOMP `CONNECT` frame?** The initial message sent by the client to establish a STOMP session.

**Q25: What is the STOMP `SUBSCRIBE` frame?** Client telling server it wants to listen to a specific destination.

**Q26: What is the STOMP `SEND` frame?** Client sending a message to a destination.

**Q27: Can WebSockets transmit binary data?** Yes, but STOMP is typically text-oriented (JSON).

**Q28: Why do load balancers drop WebSocket connections?** Many LBs have idle timeouts (e.g., 60 seconds). If no messages are sent, the LB kills the connection.

**Q29: How to prevent load balancers from dropping connections?** Implement "Heartbeats" (ping/pong messages) to keep the connection active. STOMP/SockJS handle this natively.

**Q30: What is a reverse proxy configuration for WebSockets?** Nginx requires specific headers (`Upgrade` and `Connection`) to be passed to support WebSocket proxying.

**Q31: Can I use Spring WebFlux for WebSockets?** Yes, WebFlux has native support for WebSockets, separate from the MVC STOMP implementation.

**Q32: Is STOMP mandatory for WebSockets in Spring?** No, you can implement raw `WebSocketHandler` for custom binary/text protocols, but STOMP makes routing much easier.

**Q33: What is CORS in WebSockets?** Cross-Origin Resource Sharing. You must allow specific origins using `setAllowedOrigins()` or `setAllowedOriginPatterns()`.

**Q34: Does Postman support WebSockets?** Yes, modern versions of Postman have a WebSocket testing client.

**Q35: How to get the logged-in user in a WebSocket controller?** The `Principal` object can be injected into the method arguments, provided auth was established during connection.

**Q36: What happens if the server crashes?** The WebSocket connection drops. The client must implement reconnection logic.

**Q37: Does SockJS reconnect automatically?** No, you have to write JS logic to attempt reconnection if the connection closes.

**Q38: Can I send headers in a STOMP message?** Yes, using `@Header` to extract them on the server side.

**Q39: How much overhead do WebSockets have?** Very little. Only 2-10 bytes of framing overhead per message, compared to hundreds of bytes for HTTP headers.

**Q40: Are WebSockets stateful?** Yes, the connection state is maintained in server memory.

**Q41: How do you handle message delivery guarantees?** WebSockets run on TCP (reliable delivery), but if the connection drops, messages are lost. Use `ACK` mechanisms for critical data.

**Q42: What is STOMP `ACK`?** Client explicitly acknowledging receipt of a message before the broker removes it from a queue.

**Q43: Which is better for IoT devices: WebSockets or MQTT?** MQTT is designed specifically for IoT (lightweight, handles poor networks). WebSockets are better for web browsers.

**Q44: Can you rate-limit WebSockets?** Yes, but it requires custom interceptors to track message frequency per session.

**Q45: What is `UserDestinationMessageHandler`?** Resolves `/user/{user}/queue/...` to the specific active session IDs for that user.

**Q46: Can a user have multiple WebSocket sessions?** Yes (e.g., logged in on phone and laptop). Spring automatically routes private messages to ALL active sessions for that user.

**Q47: How to broadcast to all users EXCEPT the sender?** `SimpMessagingTemplate` doesn't do this easily. You often have to handle it on the client side (ignore messages matching your own ID).

**Q48: Does Spring Cache work on WebSocket controllers?** No, `@Cacheable` is designed for request-response flows, not streaming sockets.

**Q49: How to test WebSockets in Spring?** It's complex. Requires creating a `WebSocketStompClient` in the test, connecting to the embedded test server, and using `CompletableFuture` to wait for callbacks.

**Q50: Why use Spring WebSockets instead of Node.js/Socket.io?** If your entire backend, domain logic, and security are already in Java/Spring, keeping WebSockets in the same app is much easier to maintain than splitting the stack.

---

## 📚 References

- [Spring WebSockets Documentation](https://docs.spring.io/spring-framework/reference/web/websocket.html)
- [STOMP Protocol Specification](https://stomp.github.io/stomp-specification-1.2.html)
- [SockJS](https://github.com/sockjs/sockjs-client)

---

> **Previous Topic:** [← 36 - GraalVM Native Image](../36-graalvm-native/README.md)  
> **Back to Root:** [Spring Boot Mastery 🚀](../../README.md)
