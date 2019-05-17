
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

# Learn how to create Flux Instances

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
```

아래는 100ms 주기로 0부터 9까지를 방출하는 코드.

```java
Flux<String> emits10IncreasingValuesAtRegularPace = Flux
  .interval(Duration.ofMillis(100))
  .take(10);
```

# Learn how to create Mono instances

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Mono

## Description

- `Mono<T>` 역시 Reactive Streams `Publisher`.
- `Flux`의 특수화. 최대 1개의 `<T>` 엘리먼트 방출.
- vlued(complete with element), empty, failed(error) 중에 하나.
- 오직 완료 시그널에만 관심있는 경우에 사용. `Runnable`에 대응.

![mono](https://tech.io/servlet/fileservlet?id=26381681436051)

```java
Mono.just(1)
    .map(integer -> "foo" + integer)
    .or(Mono.delay(Duration.ofMillis(100)))
    .subscribe(System.out::println);
```

# StepVerifier and how to use it

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/StepVerifier

## Description

- `reactor-test` artifact (이로부터 많은 것들이 설명됨)
- 어떤 `Publisher`도 구독할 수 있음.
- 그리고 나서 사용자가 정의한, 시퀀스에 관한 expectation들을 assert.
- expectation을 충족하지 못하면 `AssertionError`를 생산.
- 정적 팩토리인 `crate`로 `StepVerifier` 인스턴스를 얻어낼 수 있음.

```java
StepVerifier.create(T<Publisher>).{expectations...}.verify();
```

- DSL로 데이터 파트에 대한 expectation 셋업.
- 단일 종료 expectation (completion, error, cancellation...)으로 끝남.

```java
StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofHours(3)))
            .expectSubscription()
            .expectNoEvent(Duration.ofHours(2))
            .thenAwait(Duration.ofHours(1))
            .expectNextCount(1)
            .expectComplete()
            .verify();
```

# Learn to transform our asynchronous data

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/transform

## Practice

- Mono와 Flux의 `map`에 대한 간단 소개.
- 그리고 나머지 부분은 `flatMap`에 대한 것.
- `flatMap`은 변환(transformation) `Function`을 인자로 받음.
- 그런데 이 `Function`은 `U`가 아니라, `Publisher<U>`를 반환.
- 그리고 이 퍼블리셔는 각 엘리먼트에 적용할 비동기 변환을 나타냄.
- 만약, `map`을 사용한다면, `Flux<Publisher<U>>`를 얻게 됨.
- 게다가 동기(synchronous) 호출.

> But `flatMap` on the other hand knows how to deal with these inner publishers: it will subscribe to them then merge all of them into a single global output, a much more useful `Flux<U>`. Note that if values from inner publishers arrive at different times, they can interleave in the resulting `Flux`.

이제 아래 그림이 이해됨. 'Async transformation'이라는 말도 와 닿음.

![async transformation](https://raw.githubusercontent.com/reactor/projectreactor.io/master/src/main/static/assets/img/marble/flatmap.png)

# Merge

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Merge

- Merging sequences
- 몇 개의 `Publisher`들로부터 값들을 듣고(listening),
- 단일 `Flux`로 값들을 방출(emitting)하는 것.
- `merge`와 `concat`의 차이에 대해서도 언급.
- 이 차이는 reactor document에 더 잘 나와 있다고 생각함.

> Merge data from Publisher sequences contained in an array / vararg into an interleaved merged sequence. Unlike concat, sources are subscribed to eagerly.

![merge](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/doc-files/marbles/mergeFixedSources.svg)

> Concatenate all sources provided in an Iterable, forwarding elements emitted by the sources downstream. Concatenation is achieved by sequentially subscribing to the first source then waiting for it to complete before subscribing to the next, and so on until the last sources completes. Any error interrupts the sequence immediately and is forwarded downstream.

![concat](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/doc-files/marbles/concatVarSources.svg)

# Request

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Request

## Description

![reactive stream sequences/interactions](https://tech.io/servlet/fileservlet?id=26381655039806)

- 처음 소개했던 그림이 다시 나옴.
- volume control을 다룰 차례이기 때문.
- Reactive Stream 용어로는 backpressure.
- 이를 통해 `Subscriber`가 `Publisher`에게,
- 얼마나 많은 데이터를 처리할 준비가 되었는지를 알려줌.
- `Publisher`가 생산하는 데이터 속도를 제한하는 피드백 메커니즘.
- 이러한 수요의 제어<sup>control of the demand</sup>는 `Subscription` 레벨에서 이뤄짐.
- `Subscription`은 각 `subscribe()` 호출에서 만들어지고,
- `cancel()`이나 `request(long)`를 통해 제어.
- 참고로, `request(Long.MAX_VALUE)`는 무한 수요<sup>unbounded demand</sup>를 의미.

## Practice

`StepVerifier`를 통해서도 수요 조정이 가능.

```java
// TODO Create a StepVerifier
// that initially requests all values
// and expect 4 values to be received
StepVerifier requestAllExpectFour(Flux<User> flux) {
  return StepVerifier
    .create(flux)
    .expectNextCount(4)
    .expectComplete();
}

// TODO Create a StepVerifier
// that initially requests 1 value and expects User.SKYLER
// then requests another value and expects User.JESSE.
StepVerifier requestOneExpectSkylerThenRequestOneExpectJesse(Flux<User> flux) {
  return StepVerifier
    .create(flux)
    .expectNext(User.SKYLER)
    .expectNext(User.JESSE)
    .thenCancel(); // cancel하지 않으면 영원히 끝나지 않음.
}

// TODO Return a Flux
// with all users stored in the repository
// that prints automatically logs for all Reactive Streams signals
Flux<User> fluxWithLog() {
  return new ReactiveUserRepository()
    .findAll()
    .log();
}

// TODO Return a Flux with all users stored in the repository that prints "Starring:" on subscribe, "firstname lastname" for all values and "The end!" on complete
Flux<User> fluxWithDoOnPrintln() {
  return repository
    .findAll()
    .doOnSubscribe(s -> log.info("Starring:"))
    .doOnNext(u -> log.info(u.getFirstname() + " " + u.getLastname()))
    .doOnComplete(() -> log.info("The end!"));
}
```

# Error

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Error

## Description

- 에러를 전파<sup>propagate</sup>하고, 에러로부터 복구<sup>recover</sup>할 수 있는 연산자들이 내장.
- 예컨대, 폴백으로 다른 시퀀스를 사용하거나, 새로운 `Subscription`을 재시도하거나.

## Practice

- [Project Reactor 도큐먼트의 Static Fallback Value](https://projectreactor.io/docs/core/release/reference/#_static_fallback_value)를 보면, 
- 폴백 값 제공을 위한 `Mono#onErrorReturn`, `Flux#onErrorResume`,
- CheckedException을 처리하기 위한 `Exceptions#propagate` 설명이 잘 나와 있음.
- `onErrorReturn`와 `onErrorResume`은 각각 아래와 같이 설명.

> Catch and return a static default value <br/>
> Catch and execute an alternative path with a fallback method

```java
/**
 * TODO Return a Mono<User> containing User.SAUL
 * when an error occurs in the input Mono,
 * else do not change the input Mono.
 */
Mono<User> betterCallSaulForBogusMono(Mono<User> mono) {
  return mono.onErrorReturn(User.SAUL);
}

/**
 * TODO Return a Flux<User> containing User.SAUL and User.JESSE
 * when an error occurs in the input Flux,
 * else do not change the input Flux.
 */
Flux<User> betterCallSaulAndJesseForBogusFlux(Flux<User> flux) {
  return .onErrorResume(e -> Flux.just(User.SAUL, User.JESSE));
}

/**
 * TODO Implement a method
 * that capitalizes each user of the incoming flux
 * using the #capitalizeUser method
 * and emits an error containing a GetOutOfHereException error
 */
Flux<User> capitalizeMany(Flux<User> flux) {
  return flux
    .map(u -> {
      try {
        return capitalizeUser(u);
      } catch (GetOutOfHereException e) {
        throw Exceptions.propaget(e);
      }
    });
}
```

# Adapt

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Adapt

- RxJava2와 Reactor 3 타입들을 외부 라이브러리 없이도 서로 interact.
- `Flux`, `Flowable`은 모두 `Publisher`의 구현체이며,
- 모든 `Publisher`로부터 변환 가능한 팩토리 메서드를 제공.

```java
// TODO Adapt Flux to RxJava Flowable
Flowable<User> fromFluxToFlowable(Flux<User> flux) {
    return Flowable.fromPublisher(flux);
}

// TODO Adapt RxJava Flowable to Flux
Flux<User> fromFlowableToFlux(Flowable<User> flowable) {
  return Flux.from(flowable);
}
```

- `Observable`은 `Publisher` 구현체가 아니지만 약간의 트릭으로 `Flux` 어댑트가 가능.
- `Flowable`로 변환한 뒤 `Flux`로 변환하는 것.
- 참고로, `Observable`은 백프레셔를 지원하지 않아서,
- `Flowable` 변환 시 백프레셔 전략을 지정해 줘야 함.

```java
// TODO Adapt Flux to RxJava Observable
Observable<User> fromFluxToObservable(Flux<User> flux) {
  return Observable.fromPublisher(flux);
}

// TODO Adapt RxJava Observable to Flux
Flux<User> fromObservableToFlux(Observable<User> observable) {
  return Flux.from(observable.toFlowable(BackpressureStrategy.BUFFER));
}
```

- 그 외에 `Mono` <-> `Single` 변환,
- `Mono` <-> `CompletableFuture` 변환을 다루고 있음.

# Other Operations

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/OthersOperations

## Description

- 그 동안 다룬 범주에는 포함되지 않지만 유용한 연산자들을 소개.
- 이 외에도 [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)나 [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) javadoc, 그리고 [레퍼런스 가이드](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)를 잘 살펴볼 것을 권장.
- 서로 다른 지연을 가진 여러 `Flux`를 하나로 합치는 `zip`.
- 여러 `Mono` 중 가장 먼저 응답이 도착하는 것을 사용하는 `first`.
- `Flux`에도 마찬가지로 `first` 존재.
- `Flux`의 타입과 값에 관심 없고 단지 완료되기만을 바라는 경우 사용하는 `ignoreElements`와 `then`.
- `onNext`에서 기본적으로는 null 값을 허용치 않음. 아래의 `Mono#onNext`, `MonoJust`의 코드는 참고.

```java
// Mono#doOnNext
public final Mono<T> doOnNext(Consumer<? super T> onNext) {
  Objects.requireNonNull(onNext, "onNext");
  return doOnSignal(this, (Consumer)null, onNext, (Consumer)null, (Runnable)null, (LongConsumer)null, (Runnable)null);
}

// MonoJust
MonoJust(T value) {
    this.value = Objects.requireNonNull(value, "value");
}
```

- `Mono`에 넘겨주려는 값이 null일 수 있는 경우를 위한 `Mono#justOrEmpty`.
- `Mono`가 empty인 경우를 방지하기 위한 `Mono#defaultIfEmpty`.

```java
// TODO Create a Flux of user from Flux of username, firstname and lastname.
Flux<User> userFluxFromStringFlux(Flux<String> usernameFlux, Flux<String> firstnameFlux, Flux<String> lastnameFlux) {
  
  return Flux
      .zip(usernameFlux, firstnameFlux, lastnameFlux)
      .map(tuple3 -> new User(tuple3.getT1(), tuple3.getT2(), tuple3.getT3()));
}

// TODO Return the mono which returns its value faster
Mono<User> useFastestMono(Mono<User> mono1, Mono<User> mono2) {
  return Mono.first(mono1, mono2);
}

// TODO Return the flux which returns the first value faster
Flux<User> useFastestFlux(Flux<User> flux1, Flux<User> flux2) {
  return Flux.first(flux1, flux2);
}

// TODO Convert the input Flux<User> to a Mono<Void> that represents the complete signal of the flux
Mono<Void> fluxCompletion(Flux<User> flux) {
  return flux.ignoreElements().then();
}

// TODO Return a valid Mono of user for null input and non null input user (hint: Reactive Streams do not accept null values)
Mono<User> nullAwareUserToMono(User user) {
  return Mono.justOrEmpty(user);
}

// TODO Return the same mono passed as input parameter, expect that it will emit User.SKYLER when empty
Mono<User> emptyToSkyler(Mono<User> mono) {
  return mono.defaultIfEmpty(User.SKYLER);
}
```

참고로, `userFastestMono`를 검증하는 코드는 아래와 같음.

```java
@Test
public void fastestMono() {
  ReactiveRepository<User> repository = new ReactiveUserRepository(MARIE);
  ReactiveRepository<User> repositoryWithDelay = new ReactiveUserRepository(250, MIKE);
  Mono<User> mono = workshop.useFastestMono(repository.findFirst(), repositoryWithDelay.findFirst());
  StepVerifier.create(mono)
      .expectNext(MARIE)
      .verifyComplete();

  repository = new ReactiveUserRepository(250, MARIE);
  repositoryWithDelay = new ReactiveUserRepository(MIKE);
  mono = workshop.useFastestMono(repository.findFirst(), repositoryWithDelay.findFirst());
  StepVerifier.create(mono)
      .expectNext(MIKE)
      .verifyComplete();
}
```

# Reactive to Blocking

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/ReactiveToBlocking

- 일부 코드만 reactive로 바꿀 때가 있음.
- 이 경우 리액티브 시퀀스를 좀 더 명령형(imperative)으로 재사용 해야 할수도.
- `Mono#block()`을 사용하면 `Mono` 값이 이용 가능해질 때까지 대기하게 됨.
- 만약, `onError` 이벤트가 발생하면 `Exception`을 던지게 됨.
- MUST 대문자로 (굳이, 당연한 얘기 같음에도 불구하고) 아래의 경고도 하고 있음.

> You MUST avoid this at all cost in the middle of other reactive code, as this has the potential to lock your whole reactive pipeline.

- 아래는 각각 `Mono`와 `Flux`를 blocking 하는 코드들.

```java
// TODO Return the user contained in that Mono
User monoToValue(Mono<User> mono) {
  return mono.block();
}

// TODO Return the users contained in that Flux
Iterable<User> fluxToValues(Flux<User> flux) {
  return flux.toIterable();
}
```

