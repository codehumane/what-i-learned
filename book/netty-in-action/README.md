# Chapter 1. Netty-asynchronous and event-driven

## 1.1 Networking in Java

처음의 `java.net` API는 아래처럼 블럭킹 함수만 지원했음.

```java
// A new ServerSocket listenes for connection request on the specified port.
ServerSocket serverSocket = new ServerSocket(portNumber);

// accept() call blocks until a connection is established.
Socket clientSocket = serverSocket.accept();

// Stream objects are derived from those of the Socket.
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);

String request, response;
// Processing loop begins.
while ((request = in.readLine()) != null) {

    // If the client has sent "Done", the processing loop is exited.
    if ("Done".equals(request) {
        break;
    }

    // The request is passed to the server's processing method.
    response = processRequest(request);

    // The server's response is sent to the client.
    out.println(response);

// The processing loop continues.
}
```

- 위 코드는 한 번에 한 커넥션만 다룸.
- 동시에 여러 클라이언트를 다루기 위해서는, 새로운 클라이언트 `Socket` 마다 새로운 스레드를 할당해야 함.

![Multiple connections using blocking I/O](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/01fig01.jpg)

- 이 방식에서는 많은 스레드들이 휴면 상태일 수 있음.
- 단지 입출력 데이터가 나타나길 기대하면서 말이다.
- 이는 리소스의 낭비.
- 한편, 각 스레드는 스택 메모리 할당을 필요로 함.
- 이는 OS에 따라서 64KB에서 1MB까지 그 기본 크기가 다양.
- 그리고 JVM이 많은 스레드를 물리적으로 지원할 수 있다 하더라도 컨텍스트 스위칭 비용이 문제가 됨.
- 만약, 동시에 많은 클라이언트를 대응해야 하는 경우라면 이 문제들은 심각해짐.

### 1.1.1 Java NIO

- 네이티브 소켓 라이브러리는 오랫동안 논블럭킹 호출을 제공해 왔음.
- `setsockopt()`를 사용하면, 읽기/쓰기 호출이 데이터가 없을 땐 즉시 반환하게 할 수 있음.
- 시스템 이벤트 통지 API를 이용해서 논블럭킹 소켓 집합을 등록해 두면, 읽기나 쓰기 준비가 된 데이터가 있는지 알 수 있음.
- 자바는 2002년에 JDK 1.4 패키지 `java.nio`를 통해 논블럭킹 I/O를 지원하기 시작.

### 1.1.2 Selectors

![Non-blocking I/O using Selector](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/01fig02.jpg)

- 위 그림은 소켓 별로 하나씩 스레드가 할당될 때의 단점을 극복한 논블럭킹 설계.
- `java.nio.channels.Selector` 클래스는 자바 논블럭킹 I/O 구현의 핵심.
- 이는 이벤트 통지 API를 이용해서 논블럭킹 소켓들 중 I/O 준비가 된 것을 식별함.
- 읽기/쓰기 연산의 완료 상태를 언제든 확인할 수 있으므로 단일 스레드가 여러 커넥션을 동시에 다룰 수 있는 것.
- 이렇게 적은 스레드만으로 여러 커넥션을 관리할 수 있으므로, 메모리 관리나 컨텍스트 스위칭 면에서 부하를 줄일 수 있음.
- 다뤄야 하는 I/O가 없을 때 스레드들은 다른 작업에 할당 될 수 있음.

## 1.2 Introducing Netty

일반적인 이야기들

## 1.3 Netty's core components

아래 4가지 컴포넌트 다룸. 이들은 리소스, 로직, 통지 등 서로 다른 타입의 구성 요소.

- `Channel`s
- Callbacks
- `Future`s
- Events and handlers

### 1.3.1 Channels

> A channel represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing. As specified in the Channel interface, channels are either open or closed, and they are both asynchronously closeable and interruptible.

- [javadoc의 channel description](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/package-summary.html#package.description)으로 설명하고 있음.
- 그리고 책에서는 `Channel`을 인바운드/아웃바운드 데이터의 운송 수단이라고 생각하자고 함.
- 열리거나 닫힐 수 있고, 연결되거나 연결이 끊어진 상태일 수 있음.

### 1.3.2 Callbacks

- 쉽게 말하면 메서드.
- 다른 메서드에 참조로 제공됨.
- Netty는 이벤트 핸들링에 콜백들을 사용.

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 새로운 커넥션이 맺어지면 `channelActive`가 호출된다.
        System.out.println("Client " + ctx.channel().remoteAddres() + " connected");
    }
}
```

### 1.3.3 Futures

- 연산이 완료되면 애플리케이션에 이를 통지할 수 있는 방법을 제공함.
- 비동기 연산 결과에 대한 placeholder 처럼 동작하는 객체.
- 즉, 미래의 언젠가 완료가 될 것이고 결과에 접근하게 해 줄 것을 의미.
- Netty는 자바의 `java.util.concurrent.Future` 대신 직접 구현체를 만들어 제공.
- `ChannelFuture`는 1개 이상의 `ChannelFutureListener` 인스턴스를 등록할 수 있는 메서드를 추가로 제공.
- 리스너의 콜백 메서드인 `operationComplete()`은 연산이 완료될 때 호출됨.
- 이 때 리스너는 연산이 성공했는지 실패했는지 판별할 수 있음.
- 후자라면 `Throwable`을 얻을 수 있음.
- 요컨대, `ChannelFutureListener` 통지 메커니즘을 활용하면, 연산 완료 여부를 수동으로 확인하지 않아도 됨.
- Netty의 각 아웃바운드 I/O 연산은 `ChannelFuture`를 반환함.
- 즉, 어떤 것도 블럭 되지 않음.

```java
Channel channel = ...;
// 비동기로 원격에 연결하며 블럭되지 않는다.
ChannelFuture future = channel.connect(
    new InetSocketAddress("192.xxx", 25)
);
```

- 아래는 `ChannelFutureListener`를 이용한 예시 코드.

```java
Channel channel = ...;
// 비동기로 원격에 연결하며 블럭되지 않는다
ChannelFuture future = channel.connect(
    new InetSocketAddress("192.xxx", 25)
);

// ChannelFutureListener를 등록해서 연산 완료됐을 때 통지받고자 함
future.addListener(new ChannelFutureListener() {

    @Override
    public void operationComplete(ChannelFuture future) {
        // 연산의 상태를 확인
        if (future.isSuccess()) {
            // 성공이라면 ByteBuf를 만들어 데이터를 담는다
            ByteBuf buffer = Unpooled.copiedBuffer("Hello", Charset.defaultCharset());
            // 원격으로 데이터를 비동기로 보내고 ChannelFuture를 얻음
            ChannelFuture wf = future.channel().writeAndFlush(buffer);
            // ...
        } else {
            // 에러가 있다면 원인을 명시하기 위해 Throwable에 접근한다
            Throwable cause = future.cause();
            cause.printStackTrace();
        }
    }

});
```

# Chapter 7. EventLoop and threading model

## 7.1 Threading model overview

스레딩 모델에 대한 간단한 언급.

- 스레딩 모델은 코드가 어떻게 실행될지를 명시.
- 동시 실행의 부수효과를 방지하려면, 적용된 모델의 영향을 이해해야 함.
- 이를 무시하고 단지 잘 되기를 바라는 것은 도박과 같음.
- 멀티 코어나 CPU가 흔하므로 대부분의 현대 애플리케이션은 복잡한 멀티스레딩 기법을 사용.

Java의 스레딩 모델.

- 초창기 Java에서는 단지 필요할 때 새 스레드를 만들고 실행시키는 것에 지나지 않음.
- 이는 큰 부하가 있을 땐 잘 동작하지 않는 원초적 접근법.
- Java 5에서는 `Executor` API의 스레드 풀(스레드 캐싱과 재사용)로 성능이 좋아짐.
- 스레드 풀링 방식 간단 설명.
    - 풀에서 가용한 스레드가 선택되고, 제출된 작업(`Runnable` 구현체)의 실행을 위해 할당.
    - 작업이 완료되면, 스레드는 다시 반환되어 재사용 가능한 상태가 됨.
- 불필요한 생성/삭제 비용을 줄이긴 했지만, 컨텍스트 스위칭 비용까지 줄이지는 못함.
- 이 컨텍스트 스위칭 비용은 스레드가 늘어나고 큰 부하 상황에서 문제가 됨.
- 이에 더해, 전체적인 복잡성이나 애플리케이션의 동시성 요구사항으로 스레드 관련 문제들이 종종 일어났음.

## 7.2 Interface EventLoop

이벤트 루프는 커넥션 생애주기 동안 일어나는 이벤트를 다루기 위해 작업들을 실행. Netty에서는 `io.netty.channel.EventLoop` 인터페이스에 대응하며, 기본 아이디어를 코드로 표현하면 아래와 같음.

```java
while (!terminated) {
    // blocks until there are events that are ready to run
    List<Runnable> readyEvents = blockUntilEventsReady();

    // loops over and runs all the events
    for (Runnable ev: readyEvents) {
        ev.run();
    }
}
```

Netty의 `EventLoop`는 2가지 핵심 API인 동시성과 네트워킹을 사용.

- `io.netty.util.concurrent`: JDK의 `java.util.concurrent` 패키지를 기반으로 해서, 스레드 익스큐터들을 제공.
- `io.netty.channel`: 첫 번째 클래스들을 확장해서 Channel 이벤트들과 상호작용.

이 `EventLoop` 클래스 계층을 그림으로 나타내면 아래와 같음.

![EventLoop class hierarchy](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/07fig02_alt.jpg)

`EventLoop` 동작 좀 더 설명.

- 이 모델에서 `EventLoop`는 정확히 1개의 스레드를 이용.
- 작업(`Runnable` 또는 `Callable`)들은 그것이 즉각적인 실행이든 스케쥴링된 실행이든 `EventLoop` 구현체에게 직접 제출됨.
- 설정과 가용한 코어에 따라, 리소스 사용의 최적화를 목적으로 여러 `EventLoop`들이 생성될 수 있음.
- 그리고 하나의 `EventLoop`는 여러 `Channel`들을 서비스함.

이벤트/작업 실행 순서 이야기.

- 이벤트와 작업들은 FIFO 순서로 실행됨.
- 이는 바이트 컨텐츠들을 정확한 순서로 처리되게 하여,
- 데이터 충돌의 가능성을 줄여줌.

### 7.2.1 I/O and event handling in Netty 4

- I/O 연산에 의해 촉발된 이벤트는 `ChannelPipeline`을 따라 흘러감.
- 이 파이프라인은 1개 이상의 `ChannelHandler`를 가지는데,
- 필요에 따라 `ChannelHandler`는 이벤트들을 전파하는 메서드 호출을 가로채 적절히 처리.
- 이벤트의 성질은 그것이 어떻게 다뤄져야 하는지를 결정(네트워크를 통한 데이터 전달이나 수신 등 다양).
- 하지만 이벤트 핸들링 로직은 범용적이어야 하고 모든 가능한 유스 케이스들을 다룰 수 있도록 유연해야 함.
- 따라서 Netty 4에서 모든 I/O 연산과 이벤트는 `EventLoop`에 할당된 `Thread`에 의해 처리됨.

### 7.2.2 I/O operations in Netty 3

- 이전 버전에서의 Netty는 오로지 인바운드 이벤트만을 I/O 스레드(버전4에서의 `EventLoop`에 대응)에서 실행.
- 그리고 모든 아웃바운드 이벤트는 호출 스레드에 의해 처리되었음.
- 처음엔 이 방식도 괜찮아 보였으나 금새 문제가 됨.
- 여러 스레드가 하나의 아웃바운드 이벤트에 동시 접근할 수 있었던 것.
- 또 다른 부정적 부수효과는 아웃바운드 이벤트의 결과로 인해 인바운트 이벤트가 촉발될 때 일어남.
- `Channel.write()`이 예외를 일으키면 `exceptionCaught` 이벤트를 만들고 촉발시켜야 함.
- Netty 3 모델에서는 이 이벤트가 인바운드 이벤트이므로, 호출 스레드에서 코드 실행을 끝내고, I/O 스레드로 이벤트를 보내야 함.
- 이는 결국 추가적 컨텍스트 스위치로 이어짐.
- Netty 4에서는 `EventLoop`에서 일어나는 모든 것을 같은 스레드에서 처리하는 것으로 이 문제를 극복.
- 이는 단순한 실행 아키텍처를 가져다 주고 `ChannelHandler`들의 동기화 필요성을 제거해 줌.

## 7.3 Task scheduling

- 때로는 작업을 나중에 또는 주기적으로 실행하도록 스케쥴링 할 수도 있음.
- 흔한 사례 중 하나는 커넥션이 살아있는지 확인을 위해 주기적으로 heartbeat를 보내는 것.
- 이어지는 절에서 Java API와 netty `EventLoop`를 통해 어떻게 작업을 스케쥴링 하는지 알아봄.
- 그리고 Netty 구현체의 내부를 알아보며 이것의 단점과 한계도 논의.

### 7.3.1 JDK scheduling API

Java 스케쥴링 방식의 변화.

- Java 5 이전에는 작업 스케쥴링이 `java.util.Timer` 위에 이뤄짐.
- 이는 백그라운드 `Thread`를 사용하며 표준 스레드의 한계를 그대로 가짐.
- 이후 JDK는 `java.util.concurrent`를 제공함.
- 여기에는 `ScheduledExecutorService` 인터페이스를 정의하고 있음.

아래는 `java.util.concurrent.Executors`의 팩터리 메서드들. 앞의 2개는 스레드 갯수를 지정하여 스레드 풀을 사용하고, 뒤의 것들은 단일 스레드를 사용.

- `newScheduledThreadPool(int corePoolSize`
- `newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)`
- `newSingleThreadScheduledExecutor()`
- `newSingleThreadScheduledExecutor(ThreadFactory threadFactory)`

다음은 `ScheduledExecutorService`를 이용해서 60초 지연 뒤에 작업을 수행하는 코드.

```java
// 10개 스레드 풀로 ScheduledExecutorService 생성
ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);

// Runnable을 생성하고 이를 나중에 실행하도록 스케쥴링
ScheduledFuture<?> future = executor.schedule(
    new Runnable() {
        @Override
        public void run() {
            // 스택에 찍힐 메시지
            System.out.println("60 seconds later");
        }
    },
    // 60초 뒤로 작업을 스케쥴링
    60,
    TimeUnit.SECONDS
);
...
// 작업이 완료되면 ScheduledExecutorService를 종료하여 리소스 해제
executor.shutdown();
```

`ScheduledExecutorService` API가 직관적이긴 하나, 큰 부하에서는 성능 비용을 감수해야 함. 다음 절에서는 Netty가 어떻게 같은 작업을 더 효율적으로 하는지 보여줌.

### 7.3.2 Scheduling tasks using EventLoop

`ScheduledExecutorService`는 한계가 있음.

- 풀 관리의 일환으로 추가 스레드들이 생성될 수 있음.
- 많은 작업들이 공격적으로 스케쥴링 된 경우 이것이 병목이 될 수 있음.
- Netty는 채널의 `EventLoop`를 이용해서 이 문제를 극복함.

```java
Channel ch = ...
ScheduledFuture<?> future = ch.eventLoop().schedule(
    new Runnable() {
        @Override
        public void run() {
            System.out.prinlnt("60 seconds later");
        }
    },
    60,
    TimeUnit.SECONDS
);
```

`EventLoop`을 통한 스케쥴링 설명.

- 채널에 할당된 `EventLoop`가 60초 경과 후 `Runnable` 인스턴스를 실행.
- Netty `EventLoop`는 `ScheduledExecutorService`를 확장했으므로,
- `schedule()`과 `scheduleAtFixedRate()`과 같은 JDK의 모든 메서드 사용 가능.
- 실행을 취소하거나 확인하고 싶으면 반환된 `ScheduledFuture`를 사용.

```java
// 작업을 스케쥴링하고 ScheduledFuture를 획득
ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(...);
boolean mayInterruptIfRunning = false;
// 작업을 취소해서 다시 실행되는 것을 방지
future.cancel(mayInterruptIfRunning);
```

코드만 봐서는 어떻게 성능적 이점을 얻는지 잘 알 수 없음. 이는 뒤이어 설명.

## 7.4 Implementation details

- Netty 스레딩 모델의 주요 요소와 스케쥴링 구현체를 상세히 설명.
- 현재 개발 진행 중인 영역과 더불어 한계점도 언급할 예정.

### 7.4.1 Thread management

이벤트 루프의 실행 로직 설명.

- Netty 스레딩 모델의 뛰어난 성능은 현재 실행중인 스레드를 식별하는 데 있음.
- 호출 스레드가 현재 `Channel`과 채널의 `EventLoop`에 할당된 것인지를 판별하는 것.
- 만약 호출 스레드가 `EventLoop`의 것이라면 코드 블럭은 실행됨.
- 그렇지 않으면 `EventLoop`는 작업을 나중에 실행하도록 스케쥴링 후 내부 큐에 적재.
- 큐에 적재된 이벤트는 다음에 다시 `EventLoop`가 처리.
- 이 방식이 `ChannelHandler`와의 동기화 없이 스레드가 `Channel`과 직접 상호작용할 수 있는지를 설명.
- 참고로 이벤트 루프는 각자 자신만의 작업 큐를 가지며 서로의 것과는 독립적.

오래 걸리는 작업을 실행 큐에 절대 넣지말라는 이야기.

- 블럭킹 호출 또는 오래 걸리는 작업을 반드시 수행해야 한다면,
- 전담 `EventExecutor`를 사용할 것을 권장.

### 7.4.2 EventLoop/thread allocation

- 채널을 위한 I/O와 이벤트를 서비스하는 `EventLoop`들은 `EventLoopGroup`에 속함.
- `EventLoop`들이 생성되고 할당되는 방식은 전송 구현체에 따라 달라짐.

**Asynchronous transports**

- 비동기 구현체들은 단지 몇 개의 `EventLoop`들을 사용.
- 그리고 현재 모델에서는 이들이 `Channel`들 간에 공유됨.
- 이는 많은 `Channel`들을 가능한 적은 수의 `Thread`로 지원할 수 있게 함.
- `Channel` 별로 `Thread`를 할당하지 않음.

![EventLoop allocation for non-blocking transports](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/07fig04_alt.jpg)

- `EventLoopGroup`은 새로 생성된 `Channel`을 `EventLoop`로 할당하는 책임을 짐.
- 현재 구현에서는 라운드 로빈으로 분배를 조정하고 있음.
- 같은 `EventLoop`가 여러 `Channel`에 할당됨.
- `Channel`이 일단 한 번 `EventLoop`에 할당되면, `Channel`은 자신의 생애 동안 같은 `EventLoop`를 사용.
- 이는 `ChannelHandler` 구현체들의 스레드 안정성이나 동기화 문제로부터 자유롭게 함.

**Blocking transports**

![EventLoop allocation of blocking transports](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/07fig05_alt.jpg)

- OIO(old blocking I/O) 같은 다른 전송의 설계는 조금 다름.
- 하나의 `EventLoop`(그리고 그것의 `Thread`)가 각 `Channel`에 할당됨.
- 각 채널의 I/O 이벤트들은 단지 하나의 스레드(`Channel`의 `EventLoop`)에 의해 처리됨을 보장.
