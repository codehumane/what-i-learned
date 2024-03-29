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

### 1.3.4 Events and handlers

- Netty는 I/O 연산의 상태 변화를 알리기 위해 구별된 이벤트를 사용.
    - Active or inactive connections
    - Data reads
    - User events
    - Error events
    - Opening or closing a connection to a remote peer
    - Writing or flushing data to a socket
- 그리고 사용자는 핸들러 구현체를 만들어 등록하면 이벤트를 전달 받을 수 있음.
- Netty는 미리 정의된 다양한 핸들러들을 제공하고 있음.

# Chapter 2. Your first Netty application

## 2.2 Netty client/server overview

![Echo client and server](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/02fig01.jpg)

- 위 그림의 동작을 할 Echo 클라이언트와 서버를 작성할 예정.
- 여러 클라이언트가 동시에 서버로 연결됨.
- 지원하는 클라이언트의 수 제한은 시스템 리소스 가용성에 달려 있음.
- 클라이언트와 서버의 동작은 단순.
- 클라이언트가 연결을 맺으면, 서버로 1개 이상의 메시지를 전송.
- 그러면 다시 각 클라이언트로 동일한 메시지가 다시 돌아옴.

## 2.3 Writing the Echo server

모든 Netty 서버는 다음을 필요로 함.

- 적어도 1개의 `ChannelHandler`: 클라이언로부터 받은 데이터를 처리하는 서버측 컴포넌트.
- 부트스트랩핑: 서버를 구성하는 초기 구동 코드. 서버를 특정 포트로 바인딩하고, 연결 요청을 기다리게 됨(listen).

### 2.3.1 ChannelHandlers and business logic

- Echo 서버는 들어오는 메시지에 응답해야 하므로,
- `ChannelInboundHandler` 인터페이스를 구현해야 함.
- 이 인터페이스는 인바운드 이벤트에 대해 동작하는 메서드들을 정의.
- 우리가 작성할 애플리케이션에서는 이 중 몇 개의 메서드만 필요하므로,
- `ChannelInboundHandlerAdapter`만을 서브클래싱하는 것으로 충분.
- 이 클래스는 `ChannelInboundHandler`의 기본 구현들을 제공하고 있음.
- 우리가 관심 있는 메서드는 `channelRead()`, `channelReadComplete()`, `exceptionCaught()`.

```java
@ChannelHandler.Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        final ByteBuf in = (ByteBuf) msg;
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in); // Writes the received message to the sender without flushing the outbound messages
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER) // Flushes pending messages to the sender without flushing the outbound messages
                .addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        // Closes the channel
        ctx.close();
    }
}
```

- `ChannelHandler`들은 서로 다른 종류의 이벤트에 의해 작동됨.
- 애플리케이션은 `ChannelHandler`를 구현하거나 상속해서, 이벤트 생애주기를 훅킹해서 커스텀 애플리케이션 로직을 실행시킬 수 있음.
- 아키텍처 적으로, `ChannelHandler`는 비즈니스 로직을 네트워킹 코드와 디커플링 할 수 있게 도와줌.

### 2.3.2 Bootstrapping the server

이제 다음을 포함하는 서버 부트스트랩핑을 할 수 있음.

- 포트를 바인딩해서 서버가 들어오는 연결 요청을 받아줄 준비를 한다.
- `Channel` 설정을 통해 `EchoServerHandler` 인스턴스에게 인바운드 메시지를 통지한다.

```java
public class EchoServer {

    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {

        if (args.length != 1) {
            System.err.println("Usage: " + EchoServer.class.getSimpleName() + " <port>");
        }

        int port = Integer.parseInt(args[0]);
        new EchoServer(port).start();
    }

    public void start() throws Exception {
        final EchoServerHandler serverHandler = new EchoServerHandler();

        // Creates the EventLoopGroup
        final NioEventLoopGroup group = new NioEventLoopGroup();

        try {
            // Creates the ServerBootstrap
            final ServerBootstrap b = new ServerBootstrap();

            b.group(group)
                    // Specifies the use of an NIO transport Channel
                    .channel(NioServerSocketChannel.class)
                    // Sets the socket address using the specified port
                    .localAddress(new InetSocketAddress(port))
                    // Adds an EchoServerHandler to the Channel's ChannelPipeline
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        // EchoServerHandler is @Sharable so we can always use the same one
                        @Override
                        protected void initChannel(SocketChannel sc) throws Exception {
                            sc.pipeline().addLast(serverHandler);
                        }
                    });

            // Binds the server asynchronously;
            // sync() waits for the bind to complete
            final ChannelFuture f = b.bind().sync();

            // Gets the CloseFuture of the Channel and blocks the current thread until it's complete
            f.channel().closeFuture().sync();
        } finally {
            // Shuts down the EventLoopGroup, releasing all resources
            group.shutdownGracefully().sync();
        }
    }
}
```

- 위 코드에서 `ChannelInitializer`가 핵심이라고 함.
- 새로운 연결이 수락되면, 새로운 자식 `Channel`이 만들어지고,
- `ChannelInitializer`는 `EchoServerHandler`의 인스턴스를 `Channel`의 `ChannelPipeline`으로 추가.
- 앞서 얘기한 것처럼, 핸들러는 인바운드 메시지에 대한 통지를 받게 됨.

## 2.4 Writing an Echo client

Echo 클라이언트는 아래의 일들을 함.

1. 서버에 연결
2. 1개 이상의 메시지 전송
3. 각 메시지에 대해 서버로부터 똑같은 메시지를 수신
4. 연결 해제

### 2.4.1 Implementing the client logic with ChannelHandlers

- 서버처럼 클라이언트도 `ChannelInboundHandler`를 이용해서 데이터를 처리함.
- 여기서는 `SimpleChannelInboundHandler`의 `channelActive()`, `channelRead()`, `exceptionCaught()`를 오버라이딩.

```java
@ChannelHandler.Sharable // Marks this class as one whose instances can be shared among channels
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // When notified that the channel is active, sends a message
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", StandardCharsets.UTF_8));
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        // Logs a dump of the received message
        System.out.println("Client received: " + in.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // On exception, logs the error and closes channel
        cause.printStackTrace();
        ctx.close();
    }

}
```

### 2.4.2 Bootstrapping the client

- 서버 부트스트랩핑과 매우 유사한데, 리스닝 포트를 바인딩하는 대신 호스트와 포트 파라미터를 이용해서 원격 주소에 연결함.

```java
public class EchoClient {

    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            // Creates Bootstrap
            final Bootstrap b = new Bootstrap();

            // Specifies EventLoopGroup to handle client events; NIO implementation is needed
            b.group(group)
                    // Channel type is the one for NIO transport
                    .channel(NioSocketChannel.class)
                    // Sets the server's InetSocketAddress
                    .remoteAddress(new InetSocketAddress(host, port))
                    // Adds and EchoClientHandler to the pipeline when a Channel is created
                    .handler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });

            // Connects to the remote peer; waits until the connect completes
            final ChannelFuture f = b.connect().sync();

            // Blocks until the Channel closes
            f.channel().closeFuture().sync();
        } finally {
            // Shuts down the thread pools and the release of all resources
            group.shutdownGracefully().sync();
        }
    }

    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("Usage: " + EchoClient.class.getSimpleName() + " <host> <port>");
            return;
        }

        final String host = args[0];
        int port = Integer.parseInt(args[1]);
        new EchoClient(host, port).start();
    }
    
}
```

# Chapter 3. Netty components and design

- 좀 더 추상화 된 수준에서 바라보면, Netty는 2가지 관심사를 다루고 있음.
- 이는 기술<sup>technical</sup>, 아키텍처<sup>architectural</sup>라고 부를 수도 있음.
- 첫 번째로, Java NIO에 기반한 비동기 그리고 이벤트 주도 구현으로, 높은 부하 속에서도 애플리케이션의 성능과 확장성을 보장.
- 두 번째로, 애플리케이션 로직을 네트워크 로직으로부터 디커플링하는 디자인 패턴을 수용함. 이는 개발을 쉽게 만드는 한편, 코드의 테스트성, 모듈화, 재사용성을 극대화 함.
- 이번 장에서는 개별 모듈을 좀 더 자세히 살펴보면서, 이런 관심사가 어떻게 녹아졌는지도 다룰 예정.

## 3.1 Channel, EventLoop, and ChannelFuture

Netty의 네트워킹 추상화를 표현하는 아래 3개 클래스를 상세히 다룸.

- **Channel** - 소켓
- **EventLoop** - 제어 흐름, 멀티스레딩, 동시성
- **ChannelFuture** - 비동기 통지

### 3.1.1 Interface Channel

- Java 기반의 네트워킹에서 가장 핵심 요소는 `Socket` 클래스.
- Netty의 `Channel` 인터페이스는 `Socket`을 직접 다룰 때의 복잡성을 크게 감소시켜 줌.
- 더불어, `Channel`은 미리 정의된 여러 클래스 계층들의 최상위 루트.
    - `EmbeddedChannel`
    - `LocalServerChannel`
    - `NioDatagramChannel`
    - `NioSctpChannel`
    - `NioSocketChannel`

### 3.1.2 Interface EventLoop

- `EventLoop`는 Netty의 핵심 추상화.
- 커넥션 생애주기 동안 일어나는 이벤트들을 핸들링 함.
- 아래 그림은 `Channel`, `EventLoop`, `Thread`, `EventLoopGroup` 간의 관계를 표현.

![Channels, EventLoops, and EventLoopGroup](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/03fig01.jpg)

- `EventLoopGroup`은 1개 이상의 `EventLoop`를 가짐.
- `EventLoop`는 자신의 생애주기 동안 1개의 `Thread`에 바인딩 됨.
- `EventLoop`는 모든 I/O 이벤트를 자신에게 할당 된 스레드에서 처리함.
- `Channel`은 자신의 생애주기 동안 1개의 `EventLoop`에 등록됨.
- 1개의 `EventLoop`는 1개 이상의 `Channel`에 할당됨.
- 여기서 주목할 점은, `Channel`의 I/O가 같은 스레드에서 실행된다는 것.
- 이는 동기화의 필요를 가상적으로 제거해 줌.

### 3.1.3 Interface ChannelFuture

- Netty의 모든 I/O 연산은 비동기.
- 연산이 즉각 반환을 안 할 수 있기에, 나중에 따로 결과를 받을 수 있는 방법이 필요.
- 이를 위해 Netty는 `ChannelFuture`를 제공.
- 이 클래스의 `addListener()` 메서드는 `ChannelFutureListener`를 등록할 수 있게 해주며,
- 이 리스너를 통해 연산 완료를 통지받을 수 있음.

## 3.2 ChannelHandler and ChannelPipeline

데이터의 흐름을 관리하고, 애플리케이션 로직을 실행시키는 컴포넌트들을 살펴볼 차례.

### 3.2.1 Interface ChannelHandler

- 애플리케이션 개발자 관점에서는 `ChannelHandler`가 주요 컴포넌트.
- 애플리케이션 로직의 컨테이너 역할을 하며, 네트워크 이벤트 발생 시 이 로직이 실행됨.
- `ChannelInboundHandler`는 자주 구현되는 서브인터페이스.
- 이는 인바운드 이벤트를 수신하고 애플리케이션의 비즈니스 로직이 이 데이터를 핸들링 할 수 있게 해 줌.
- 또한, 연결된 클라이언트에 응답 데이터를 바로 전송할 수도 있음.

### 3.2.2 Interface ChannelPipeline

`ChannelPipeline` 간단 소개.

- `ChannelHandler` 체인의 컨테이너를 제공.
- 체인을 따라 인바운드/아웃바운드 이벤트를 전파시키는 API를 정의.

`ChannelHandler`가 `ChannelPipeline`에 설치되는 과정은 아래와 같음.

- `ChannelInitializer` 구현체가 `ServerBootstrap`에 등록됨.
- `ChannelInitializer.initChannel()`이 호출되면, `ChannelInitializer`가 커스텀 `ChannelHandler` 집합을 파이프라인에 설치.
- `ChannelInitializer`는 스스로를 `ChannelPipeline`에서 제거함.

`ChannelHandler` 좀 더 설명.

- `ChannelHandler`는 광범위한 용법을 지원하기 위해 설계됨.
- `ChannelPipeline`의 이벤트를 처리하는 코드라면 어느 것이든 담을 수 있는 범용적 컨테이너.
- 애플리케이션의 부트스트랩핑이나 초기화 시 `ChannelHandler`들이 등록됨.
- 이들은 이벤트를 받고, 자신들의 로직을 실행하고, 체인의 다음 핸들러에게 데이터를 넘겨줌.
- 이런 식으로 파이프라인에 이벤트들이 흘러다님.
- 실행되는 핸들러의 순서는 등록된 순서와 동일.

인바운드/아웃바운드 데이터 흐름 속에서 `ChannelHandler`와 `ChannelPipeline`의 관계.

![ChannelPipeline with inbound and outbound ChannelHandlers](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/03fig03_alt.jpg)

- 위 그림은 Netty 애플리케이션에서의 인바운드/아웃바운드 데이터 흐름을 구별해서 보여줌.
- 같은 파이프라인 안에 인바운드와 아웃바운드 핸들러 모두가 등록될 수 있음도 보여줌.
- 인바운드 이벤트가 읽히면, 파이프라인 head 부터 시작해서 첫 번째 `ChannelInboundHandler`에게 전달됨.
- 이 핸들러는 데이터를 수정할 수도 있고 아닐 수도 있음.
- 그리고 다음 핸들러에게도 차례로 전달됨.
- 마지막으로 데이터는 파이프라인의 tail에 다다르게 되며, 모든 처리는 종료됨.
- 아웃바운드 데이터 이동도 이와 유사하게 생각하면 됨.

`ChannelHandlerContext` 소개.

- `ChannelHandler`가 `ChannelPipeline`에 추가될 때, `ChannelHandlerContext`가 전달됨.
- 이는 `ChannelHandler`와 `ChannelPipeline` 간의 바인딩을 나타냄.
- 주로 아웃바운드 데이터를 쓰기 위해 활용됨.
- Netty에는 메시지를 보내는 방법이 2가지가 있는데,
- 그 중 1가지가 바로 `ChannelHandler`에 연결된 `ChannelHandlerContext`에게 데이터를 쓰는 것.
- 나머지 1가지는 `Channel`에 직접적으로 데이터를 쓰는 것.
- `ChannelHandlerContext`에 대한 쓰기는 파이프라인의 다음 핸들러에게 메시지가 전달되고,
- `Channel`에 대한 쓰기는 파이프라인의 tail로 메시지가 전달되게 함.

### 3.2.3 A closer look at ChannelHandlers

- `ChannelHandler`은 여러 타입이 있으며, 슈퍼 클래스가 누구냐에 따라 그 기능이 결정됨.
- Netty는 어댑터 클래스의 형태로 기본 핸들러 구현체들을 제공.
- 이들은 체인의 다음 핸들러에게 이벤트를 전달해 주는 일을 함.
- 어플리케이션의 프로세싱 로직 개발을 쉽게 만들어 주는 것이 목적.
- 아래의 4가지는 커스텀 핸들러를 만들 때 주로 사용하는 어댑터들.
    - `ChannelHandlerAdapter`
    - `ChannelInboundHandlerAdapter`
    - `ChannelOutboundHandlerAdapter`
    - `ChannelDuplexHandlerAdapter`

### 3.2.4 Encoders and decoders

- Netty에서 메시지를 주고 받을 때 데이터 변환이 일어남.
- 인바운드 메시지는 byte에서 다른 포맷으로 디코드 되고, 아웃바운드 메시지는 반대로 인코드 됨.
- 다양한 요구에 맞게 여러 추상 클래스들이 기본으로 제공됨.
- 일반적으로 베이스 클래스들은 `ByteToMessageDecoder`나 `MessageToByteEncode`의 이름과 비슷함.
- 조금 특이한 것으로는 구글 프로토콜 버퍼를 지원하기 위한 `ProtobufEncode`, `ProtobufDecoder` 정도.
- 인바운드 데이터를 위해서 `channelRead` 메서드와 이벤트가 오버라이딩 됨.
- 이 메서드는 인바운드 채널로부터 각 메시지가 읽힐 때마다 호출됨.
- 그리고 나서 제공된 디코더의 `decode()` 메서드를 실행시키고, 디코딩 된 바이트를 다음 `ChannelInboundHandler`에게 전달.
- 아웃바운드 메시지는 반대 방향으로 처리됨.

### 3.2.5 Abstract class SimpleChannelInboundHandler

- 일반적으로는 애플리케이션이 핸들러를 사용해서 디코딩 된 메시지를 수신하고 이 데이터에 대해 비즈니스 로직을 적용함.
- 이런 `ChannelHandler`를 만들기 위해서는 단지 `SimpleChannelInboundHandler<T>` 베이스 클래스를 상속하면 됨.
- 제일 중요한 메서드는 `channelRead0(ChannelHandlerContext,T)`임.
- 구현은 온전히 구현자에게 달려 있긴 하지만 현재 I/O 스레드가 블럭킹 되면 안되는 제약이 있음.

## 3.3 Bootstrapping

- Netty의 부트스트랩 클래스는 어플리케이션의 네트워크 레이어 설정을 위한 컨테이너를 제공.
- 주어진 포트에 프로세스를 바인딩하거나, 지정된 호스트/포트의 다른 프로세스로 연결하는 것들을 포함.
- 전자를 서버 부트스트랩핑, 후자를 클라이언트 부트스트랩핑이라 부름.
- 이 2개의 클래스를 아래처럼 비교하고 있음.

| Category | Bootstrap | ServerBootstrap |
| -------- | --------- | --------------- |
| Networking function | Connects to a remote host and port | Binds to a local port |
| Number of EventLoopGroup | 1 | 2 |

- 더 중요한 차이점은 두 번째 것.
- 클라이언트 부트스트래핑에는 1개의 EventLoopGroup가,
- ServerBootstrap에는 2개가 필요.
- 서버는 2개의 채널 집합이 필요하기 때문.
- 첫 번째 세트는 1개의 `ServerChannel`을 가짐.
- 이는 로컬 포트에 바인딩 된, 자기 자신의 리스닝 소켓을 나타냄.
- 두 번째 세트는 incoming 클라이언트 커넥션을 다루기 위해 생성되는 모든 채널을 포함.
- 1개의 연결이 1개의 채널을 가리킴.
- 그림으로 나타내면 아래와 같음.

![Server with two EventLoopGroups](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/03fig04_alt.jpg)

- `ServerChannel`에 연결된 `EventLoopGroup`은,
- 들어오는 연결 요청에 대해 `Channel`을 생성하도록 `EventLoop`를 할당.
- 연결이 수락되면, 두 번째 `EventLoopGroup`이 이 `Channel`에 대해 `EventLoop`를 할당.

# Chapter 4. Transports

## 4.2 Transport API

전송 API에서 핵심은 `Channel` 인터페이스이며, 모든 I/O 연산에 사용됨. [javadoc에서 Netty의 Channel 페이지](https://javadoc.io/static/io.netty/netty/4.0.0.Alpha3/io/netty/channel/Channel.html)에 보면 아래 그림이 나옴.

![](https://javadoc.io/static/io.netty/netty/4.0.0.Alpha3/io/netty/channel/Channel.png)

- `Channel`은 `ChannelPipeline`과 `ChannelConfig`를 가짐.
- `ChannelConfig`는 `Channel`에 관한 모든 설정을 가지며, hot changes를 지원.
- `Channel`은 고유하므로, `java.lang.Comparable`의 서브인터페이스로 선언하는 것은, 순서를 보장하기 위한 의도.
- `Channel`이 반환하는 해시코드가 겹치면 `AbstractChannel`에서의 `compareTo()`가 에러를 던짐.
- `ChannelPipeline`은 여러 `ChannelHandler` 인스턴스들을 가짐.
- 이들은 상태 변경을 처리하고 데이터를 프로세싱하는 애플리케이션 로직들을 담고 있음.

`ChannelHandler`의 일반적 쓰임은 아래의 것들.

- 데이터 형태 변환
- 예외 알림
- `Channel`의 활성화/비활성화 상태 알림
- `Channel`이 `EventLoop`에 등록되거나 제거될 때 알림
- 사용자 정의 이벤트 알림

`Channel`의 메서드로는 다음의 것들이 제공됨.

| Method name | Description |
| ----------- | ----------- |
| eventLoop | 채널에 할당된 이벤트루프 반환 |
| pipeline | 채널에 할당된 채널 파이프라인 반환 |
| isActive | 채널의 활성화 상태 여부. 여기서 활성화는 전송이 구체적으로 뭔지에 따라 다름. 소켓 전송의 경우, 원격으로의 연결이 성공하면 활성화로 간주함 |
| localAddress | 로컬 소켓 주소 반환 |
| remoteAddres | 원격 소켓 주소 반환 |
| write | 원격으로 데이터 쓰기. 이 데이터는 채널 파이프라인으로 전달되고 플러시 되기 전까지 큐에 적재됨 |
| flush | 쓰여진 데이터를 전송 |
| writeAndFlush | write, flush를 함께 호출하는 편의성 메서드 |

Channel에 쓰기 작업 예시.

```java
Channel channel = ...
// ByteBuf를 생성하고 쓰기를 위한 데이터를 담아둠.
ByteBuf buf = Unpooled.copiedBuffer("your data", CharsetUtil.UTF_8);
// 데이터를 쓰고 플러시.
ChannelFuture cf = channel.writeAndFlush(buf);
// ChannelFutureListener를 추가해서 쓰기 완료 시 알림 받음.
cf.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture channelFuture) {
        if (future.isSuccess()) {
            // 에러 없이 연산 성공했음을 로깅
            println("Write successful");
        } else {
            // 에러 로깅
            println("Write error");
            future.cause().printStacktrace();
        }
    }
});
```

`Channel` 구현체는 스레드 세이프. 아래 예시 코드처럼, 여러 스레드에서 쓰기 가능.

```java
final Channel channel = ...
final ByteBuf buf = Unpooled
    .copiedBuffer("your data", UTF_8)
    .retain();

Runnable writer = new Runnable() {
    @Override
    public void run() {
        channel.write(buf.duplicate());
    }
};
Executor executor = Executors.newCachedThreadPool();
executor.execute(writer);
executor.execute(writer);
```

## 4.3 Included transports

- 기본으로 제공되는 전송들로 NIO, Epoll, OIO, Local, Embedded가 있음.
- 다른 것은 관심 없고 NIO와 Epoll만 정리.

### 4.3.1 NIO - non-blokcing I/O

- NIO는 모든 I/O 연산에 대해 완전한 비동기 구현체를 제공.
- 셀렉터-기반 API를 이용.
- 셀렉터의 기본 개념은, 레지스트리처럼 동작하는 것.
- 채널의 상태가 변경되면 통지 받겠다고 이 레지스트리에 요청.
- 변경의 종류로는 아래와 같은 것들이 있음.
  - OP_ACCEPT: 새로운 채널이 수락됐고 준비 됨.
  - OP_CONNECT: 채널 연결이 완료됨.
  - OP_READ: 채널에 읽을 데이터가 준비됨.
  - OP_WRITE: 채널에 데이터를 쓸 수 있음.
- 애플리케이션이 이 상태 변화에 반응하면,
- 셀렉터는 리셋되고 다시 처리를 반복.
- 한 스레드에서 계속 변경을 체크하고 적절히 대응함.

![Selecting and processing state changes](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/04fig02_alt.jpg)

### 4.3.2 Epoll - native non-blocking transport for Linux

- 자바의 NIO는 플랫폼 독립성을 이뤘으나,
- 이는 그 자체로 한계이기도 함(모든 시스템에서 동일한 기능을 제공해야 하다 보니 타협을).
- 고성능 네트워킹 플랫폼으로써의 리눅스 중요성이 커짐에 따라,
- 확장성 높은 I/O 이벤트-통지를 지원하는 epoll 등이 개발됨.
- 이 API는 2002년부터 사용 가능했고,
- 기존의 POSIX select와 poll 시스템 호출보다 더 나은 성능을 보임.
- 지금은 리눅스에서 논블럭킹 네트워킹의 표준으로 자리 잡음.
- 개념적 동작 방식은 NIO 그림과 마찬가지임.

# Chapter 5. ByteBuf

- 네트워크의 기본 단위는 항상 바이트.
- Java NIO는 바이트 컨테이너로 `ByteBuffer`를 제공함.
- 하지만 다소 사용하기 어려움.
- Netty는 대안으로 `ByteBuf`를 제공.

## 5.1 The ByteBuf API

Netty의 데이터 핸들링을 위한 API는 아래 2가지를 통해 이뤄짐.

- `abstract class ByteBuf`
- `ByteBufHolder`

`ByteBuf`를 사용할 때의 이점은 다음과 같음.

- 사용자 정의 버퍼 타입으로 확장 가능.
- 내장된 컴포짓 버퍼 타입을 통해 투명한 zero-copy 가능(zero-copy 설명은 [여기](https://soft.plusblog.co.kr/7) 참고).
- 용량이 필요한 만큼 늘어남.
- `ByteBuffer`의 `flip()` 메서드를 호출하지 않고도 읽기/쓰기 모드 스위칭이 가능.
- 읽기와 쓰기가 서로 구별된 인덱스들을 사용.
- 메서드 체이닝 지원.
- 레퍼런스 카운팅 지원.
- 풀링 지원.

## 5.2 Class ByteBuf-Netty's data container

- 모든 네트워크 통신은 순차적 바이트들의 이동을 포함.
- Netty의 `ByteBuf`는 이런 데이터를 효율적이고 쉽게 다루게 도와줌.

### 5.2.1 How it works

- `ByteBuf`는 2개의 구별된 인덱스들을 관리.
- 각각은 읽기와 쓰기용.
- `ByteBuf`에 대한 읽기를 할 때, 읽어들인 바이트 만큼 `readerIndex` 값이 증가.
- 마찬가지로 쓰기를 할 때 `writerIndex` 값이 증가.
- 아래 그림은 비어 있는 상태의 `ByteBuf`를 나타냄.

![A 16-byte ByteBuf with its indices set to 0](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/05fig01.jpg)

- `writerIndex`와 `readerIndex`가 같다면 더 이상 읽을 데이터가 없다는 것.
- 이 때 더 읽기를 시도한다면 `IndexOutOfBoundsException` 발생.
- `ByteBuf`의 최대 용량도 지정할 수 있음. 기본 값은 `Integer.MAX_VALUE`.

### 5.2.2 ByteBuf usage patterns

- `ByteBuf`의 몇 가지 용법들을 소개.
- 읽기/쓰기 인덱스를 가진 바이트의 배열이라는 점을 상기하며 보는 것이 도움이 될 것.

**Heap buffers**

- 가장 많이 쓰이는 `ByteBuf` 패턴.
- JVM 힙 영역에 데이터를 저장하는 것.
- backing array라고도 불림.
- 빠른 할당/해제 가능.
- 레거시 데이터를 다룰 때 적합.

```java
ByteBuf heapBuf = ...;

// Cheks whether ByteBuf has a backing array...
if (heapBuf.hasArray()) {
    // ...if so, gets a reference to the array
    byte[] array = heapBuf.array();
    // Calculates the offset of the first byte
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
    // Gets the number of readable bytes
    int length = heapBuf.readableBytes();
    // Calls your method using array, offset, and length as parameters
    handleArray(array, offset, length);
}
```

**Direct buffers**

- JDK 1.4부터 NIO는 JVM 구현체가 메모리를 네이티브 호출로 할당할 수 있게 함.
- 버퍼 컨텐츠를 중간 단계의 버퍼로 복제하는 것을 피하게 해줌.
- [ByteBuffer javadoc](https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html)에는 아래와 같이 명시되어 있음.

> The contents of direct buffers will reside outside of the normal garbage-collected heap

- 따라서 네트워크 데이터 전송에서 이상적인 방법.
- 주요 단점은 힙 기반 버퍼에 비해 할당과 해제 비용이 더 크다는 것.
- 또한, 레거시 코드에서는 데이터 복제를 피할 수 없기도 함.

**Composite buffers**

- 여러 `ByteBuf`들에 대한 애그리거트 뷰를 제공.
- Netty에서 `CompositeByteBuf`가 여기에 해당.
- 이를 통해 `ByteBuf` 인스턴스를 필요할 때 추가/삭제할 수 있음.
- 설명을 위해, 헤더와 바디, 이렇게 두 부분으로 구성된 메세지가 있다고 해보자.
- 이들은 각각 애플리케이션의 서로 다른 모듈에서 생산되며, 메시지가 전송될 때 합쳐진다.
- 만약, 여러 메시지 전송에 같은 바디가 재사용된다면, `CompositeByteBuf`를 이용해 중복된 버퍼 할당을 줄일 수 있음.
- 같은 상황을 JDK의 `ByteBuffer`를 이용하면 아래와 같음.

```java
ByteBuffer[] message = new ByteBuffer[]{header, body};
ByteBuffer message2 = ByteBuffer.allocate(header.remaining() + body.remaining());
message2.put(header);
message2.put(Body);
message2.flip();
```

- `CompositeByteBuf`를 이용하면 아래와 같음.

```java
CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
ByteBuf headerBuf = ...;
ByteBuf bodyBuf = ...;
// Appends ByteBuf instances to the CompositeByteBuf
messageBuf.addComponents(headerBuf, bodyBuf);
....
// Removes ByteBuf at index 0 (first component) = remove the header
messageBuf.removeComponent(0);
// Loops over all the ByteBuf instances;
for (ByteBuf buf : messageBuf) {
    System.out.println(buf.toString());
}
```

## 5.3 Byte-level operations

### 5.3.1 Random access indexing

- `ByteBuf`의 인덱싱도 0부터 시작.
- 아래는 인덱스를 통해 바이트를 얻어내는 기본 접근.

```java
ByteBuf buffer = ...;
for (int i = 0; i < buffer.capacity(); i++) {
    byte b = buffer.getByte(i);
    System.out.println((char) b);
}
```

### 5.3.2 Sequential access indexing

- 읽기와 쓰기용 인덱스를 가지는 `ByteBuf`와 다르게, JDK의 `ByteBuffer`는 인덱스를 1개만 가짐.
- 이것이 읽기와 쓰기 모드 간의 전환 시 `filp()`을 호출해야 하는 이유.
- 아래 그림은 2개 인덱스로 `ByteBuf`를 3영역으로 나눈 것.

![ByteBuf internal segmentation](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/05fig03_alt.jpg)

### 5.3.3 Discardable bytes

- 위 그림에서, 2개의 인덱스로 나뉜 3개 영역 중, discardable 바이트라고 표기된 영역이 있었음.
- 언제든 버려질 수 있으며 `discardReadBytes()` 호출을 통해 공간이 비워짐.
- 최초의 사이즈는 `readerIndex`에 저장된 0과 같고, `read` 연산이 실행될 때마다 증가.
- 아래 그림은 `discardReadBytes()`가 호출된 결과. 위 그림과 비교해서 보면 됨.

![ByteBuf after discarding read bytes](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617291470/files/05fig04_alt.jpg)

- `discardReadBytes()`를 자주 호출하는 것이 좋아보일 수도 있으나,
- 이는 메모리 복제가 일어나기에, 실제로 필요한 시점에만 호출하기를 권장.

### 5.3.4 Redable bytes

- readable 바이트 영역은 실제 데이터를 저장.
- `readerIndex`의 기본 값은 0.
- read나 skip으로 시작하는 연산에 의해, 현재 `readerIndex`의 데이터가 읽어들여지거나 생략됨.
- 동시에 인덱스 값은 증가.
- 버퍼에 더 이상 readable 바이트가 없는데 읽기가 시도되면 `IOOBE` 발생.
- 아래 코드는 모든 바이트를 읽어들이는 예시.

```java
ByteBuf buffer = ...;
while (buffer.isRedable()) {
    System.out.println(buffer.readByte()));
}
```

### 5.3.5 Writable bytes

- writable 바이트 구역은 정의되지 않은 컨텐츠의 메모리 영역.
- 새로 할당된 버퍼의 `writableIndex` 기본 값은 0.
- `write`로 시작하는 모든 메서드는 현재 인덱스부터 데이터를 쓰고 인덱스를 증가시킴.
- 아래는 저장 공간이 꽉 찰 때까지 랜돔 정수 값으로 버퍼를 채우는 예시.

```java
ByteBuf buffer = ...;
while (buffer.writableBytes() >= 4) {
    buffer.writeInt(random.nextInt());
}
```

### 5.3.6 Index management

- JDK의 `InputStream`은 `mark(int readlimit)`과 `reset()` 메서드를 정의하고 있음.
- 이는 스트림에서 특정 값의 현재 포지션 표기하고, 그 포지션을 초기화하는 역할을 함.
- 유사하게, `ByteBuf`의 `readerIndex`와 `writeIndex`를 설정하고 재조정할 수 있음.
- `readerIndex(int)`와 `writerIndex(int)`를 통해 특정 포지션을 설정할 수도 있음.
- 유효치 않은 포지션으로의 설정은 `IOOBE`을 일으킴.
- `clear()`는 쓰기/읽기 포지션을 모두 0으로 설정함.
- 이는 메모리의 컨텐츠를 초기화하는 것은 아님에 유의.

### 5.3.7 Search operations

- `ByteBuf`에는 값의 인덱스를 알아낼 수 있는 몇 가지 방법이 있음.
- 가장 쉬운 방법은 `indexOf()` 메서드.
- 좀 더 복잡한 검색은 `ByteBufProcessor` 인자를 받는 메서드들의 실행으로 가능.
- 이 인터페이스는 `boolean process(byte value)`이라는 1개의 메서드를 가짐.
- 이는 입력 값이 찾으려는 값인지 여부를 반환.
- `ByteBufProcess`는 몇 가지 편의 상수를 제공해서 아래와 같이 자주 검색되는 값을 찾는데 도움을 줌.
- 최신 버전에서 `ByteBufProcessor`는 deprecated 되었고 `ByteProcessor` 사용을 권장하긴 함.

```java
ByteBuf buffer = ...;
int index = buffer.forEachByte(ByteBufProcessor.FIND_CR);
```

### 5.3.8 Derived buffers

- 파생된 버퍼들은 `ByteBuf`의 뷰(컨텐츠를 특수한 방식으로 표현)를 제공.
- 이런 뷰들은 아래 메서드들을 통해 생성됨.
    - duplicate()
    - slice()
    - slice(int, int)
    - Unpooled.unmodifiableBuffer(...)
    - order(ByteOrder)
    - readSlice(int)
- 각각은 새로운 `ByteBuf` 인스턴스를 반환.
- 내부 저장공간은 JDK의 `ByteBuffer` 같은 곳에서 공유됨.
- 이를 통해 파생된 버퍼들의 생성 비용을 줄이긴 하지만,
- 내용을 수정하면 모든 인스턴스의 것들에도 영향을 줌.

### 5.3.9 Read/write operations

read/write 연산에는 2가지 부류가 있음.

- get()과 set() 연산은 주어진 인덱스부터 시작하며 인덱스를 바꾸지는 않음.
- read()와 write() 연산은 주어진 인덱스부터 시작하고 접근한 바이트 수만큼 인덱스 값이 조정됨.

아래는 get()과 set()을 이용한 예시.

```java
Charset utf8 = Charset.forName("UTF-8");
// Creates a new ByteBuf to hold the bytes for the given String
ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);
// Prints the first char, 'N'
System.out.println((char) buf.getByte(0));
// Stores the current readerIndex and writerIndex
int readerIndex = buf.readerIndex();
int writerIndex = buf.writerIndex();
// Updates the byte at index 0 with the char 'B'
buf.setByte(0, (byte)'B');
// Prints the first char, now 'B'
System.out.println((char) buf.getByte(0));
// Succeeds because these operations don't modify the indices
assert readerIndex == buf.readerIndex();
assert writerIndex == buf.writerIndex();
```

## 5.4 Interface ByteBufHolder

- 실제 데이터 페이로드 외에 추가적인 프로퍼티 값들을 저장하고 싶을 때가 있음.
- HTTP 응답에 실제 응답 컨텐츠 외에도 상태 코드나 쿠키 등의 정보가 담겨 있는 것처럼.
- Netty는 `ByteBufHolder`를 통해 이 유스 케이스를 지원함.
- 더불어, `ByteBuf`의 풀링도 지원.

## 5.5 ByteBuf allocation

### 5.5.1 On-demand: interface ByteBufAllocator

- 메모리 할당과 해제의 오버헤드를 줄이고자,
- Netty는 `ByteBufAllocator` 인터페이스를 통해 풀링을 구현.
- `ByteBufAllocator`의 참조는 `Channel` 또는 `ChannelHandlerContext`를 통해 얻을 수 있음.
- Netty는 `ByteBufAllocator` 구현체로 2가지를 제공.
- 바로 `PooledByteBufAllocator`와 `UnpooledByteBufAllocator`.
- 전자는 성능 향상과 메모리 파편화를 줄이고자 `ByteBuf`를 풀링.
- 후자는 풀링을 하지 않고, 요청이 들어올 때마다 새로운 인스턴스를 만들어 반환.
- Netty는 기본으로 `PooledByteBufAllocator`를 사용.

## 5.6 Reference counting

- 레퍼런스 카운팅은 메모리 사용과 성능 최적화를 위한 기법.
- 다른 객체에 의해 더 이상 참조되지 않을 때 오브젝트가 가지고 있는 리소스를 해제함.
- `ByteBuf`와 `ByteBufHolder`는 모두 `ReferenceCounted` 인터페이스를 구현.
- `ReferenceCounted` 구현체 인스턴스는 보통 활성화 된 레퍼런스 카운팅을 1로 시작.
- 이 값이 0보다 클 경우는 리소스가 해제 되지 않음을 보장. 그리고 0이 되면 해제.
- 리소스 해제라는 말은 구현체마다 다르긴 하지만, 적어도 객체가 해제가 되면 더 이상 사용할 수 없음을 의미.
- 이미 해제된 객체를 사용하려 하면 `IllegalReferenceCountException` 발생.
- 레퍼런스 카운팅은 풀링 구현체에 있어 필수. 메모리 할당 오버헤드를 줄이기 때문.

```java
Channel channel = ...;
// Gets ByteBufAllocator from a channel
ByteBufAllocator allocator = channel.alloc();
...
// Allocates a BteBuf from the ByteBufAllocator
ByteBuf buffer = allocator.directBuffer();
// Checks for the expected reference count of 1
assert buffer.refCnt() == 1;
// Decrements the active references to the object. At 0, the object is released and the method returns true.
boolean released = buffer.release();
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
