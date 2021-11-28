
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

이벤트 루프는 커넥션 생애주기 동안 일어나는 이벤트를 다루기 위해 작업들을 실행. 네티에서는 `io.netty.channel.EventLoop` 인터페이스에 대응하며, 기본 아이디어를 코드로 표현하면 아래와 같음.

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

네티의 `EventLoop`는 2가지 핵심 API인 동시성과 네트워킹을 사용.

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
