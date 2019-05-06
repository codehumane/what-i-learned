
# Introduction to Reactive Programming

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro

## Why

짧은 내용 안에 핵심을 잘 담아냈다고 생각함.

> declarative code (in a manner that is similar to functional programming) in order to build asynchronous sequences of events.

- 선언적 코드
- 비동기 프로세싱 파이파라인을 구축
- 이벤트 기반 모델
- 데이터가 이용 가능할 때, 컨슈머에게 푸시되는

이런 특징들이 중요한 이유는, 리소스들을 효율적으로 사용하고, 많은 클라이언트들을 받아주기 위한 가용량을 늘리기 위함. 저수준의 동시성 또는 병렬성 코드를 작성하지 않고도 말이다.

> It also facilitates composition, which in turn makes asynchronous code more readable and maintanable.

- 또한, 구성을 이용해 가독성과 유지보수성을 높임.

## Interactions

- source `Publisher` produces data
- does nothing until a `Subscriber`has registered(subscribed)

![reactive stream sequences/interactions](https://tech.io/servlet/fileservlet?id=26381655039806)

- 여기에 더해 *operators*라는 개념도 있음.
- 데이터의 각 단계에 어떤 처리를 적용할지를 나타냄.
- 오퍼레이터의 적용은 새로운 중간단계의 `Publisher`를 반환.
- 또한 체이닝 됨.
- 마지막 형태는 결국 `Subscriber`.
- 자바의 Stream이랑 많이 유사.

# Learn how to create Flux Instance

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Flux

## Description

- `Flux<T>`는 Reactive Streams `Publisher`.
- Flux 시퀀스를 만들고, 변환하고, 조율하는 많은 연산자들을 가지고 있음.
- `onNext` 이벤트를 통해 0 ~ n 개의 \<T\> 엘리먼트를 emit하고,
- `onComplete`그리고 `onError` 종료 이벤트를 통해 complete 또는 error 처리.
- 종료 이벤트가 없다면 무한 `Flux`.

![Flux 흐름](https://tech.io/servlet/fileservlet?id=26381665455829)

- Flux의 정적 팩토리 ― 원천을 직접 생성하거나, 몇 개의 콜백 타입을 이용해 만들어 냄.
- 연산자이자 인스턴스 메서드 ― 비동기 프로세싱 파이프라인 빌드.
- `Flux#subscribe()` 또는 멀티캐스팅 연산(`Flux#publish`, `Flux#publishNext`): 파이프라인 전용 인스턴스를 구체화하고, 그 안에서 데이터 흐름을 일으킴.

```java
Flux.fromIterable(getSomeLongList())
  .delayElements(Duration.ofMillis(100))
  .doOnNext(serviceA::someObserver)
  .map(d -> d * 2)
  .take(3)
  .onErrorResumeWith(errorHandler::fallback)
  .doAfterTerminate(serviceM::incrementTerminate)
  .subscribe(System.out::println);
```

## Practice

여러 팩토리를 이용한 `Flux` 생성.

```java
Flux<String> empty = Flux.empty();
Flux<String> fromReadilyAvailableData = Flux.just("1", "2");
Flux<String> fromIterable = Flux.fromIterable(Arrays.asList("1", "2"));
```

명령형 비동기 코드에서는 try-catch로 예외를 다뤘음. Reactive Streams에서는 `onError` 시그널을 정의해서 예외를 다룸. 그리고 이 이벤트는 종료 이벤트임을 다시 한 번 강조.

```java
Flux<string> fromError = Flux.error(new IllegalStateException("Hello"));
Flux<String> emits10IncreasingValuesAtRegularPace = Flux
	.interval(Duration.ofMillis(1000))
  .take(10);
```

