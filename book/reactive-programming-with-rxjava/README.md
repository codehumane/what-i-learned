# Reactive Programming with RxJava

## How RxJava Works

- 데이터나 이벤트 스트림을 나타내는 Observable 타입
- 밀어내기(reactive) 지향 but 끌어오기(interactive)도 가능
- 즉시가 아니라 지연 실행
- 비동기와 동기 모두 사용 가능
- 시간에 따라 0, 1, 다수 혹은 무한 개의 이벤트를 다룰 수 있음.

### 밀어내기와 끌어오기

밀어내기(push)를 통한 이벤트 수신을 위해, Observable/Observer 쌍을 구독으로 연결함.

```java
interface Observable<T> {
    Subscription subscribe(Observer s)
}

interface Observer<T> {
    void onNext(T t)
    void onError(Throwable t)
    void onCompleted()
}
```

- onNext 메서드는 데이터 이벤트로, 0 ~ 무한 번 호출될 수 있음.
- onError와 onCompleted는 종료 이벤트로, 둘 중 하나만 단 한 번 호출됨.

아래는 대화형 끌어오기(interactive pull)를 지원하는 추가적인 타입 시그니처. 조금 더 발전된 Observer인, Subscriber와 함께 사용.

```java
interface Producer {
    void request(long n)
}

interface Subscriber<T> implements Observer<T>, Subscription {
    void onNext(T t)
    void onError(Throwable t)
    void onCompleted()
    ...
    void unsubscribe()
    void setProducer(Producer p)
}
```

- 첫 인상으로는 이름이 괜찮게 지어졌다고 생각 됨.

### 비동기와 동기

- 일반적으로 Observable은 비동기.
- 하지만 반드시 그럴 필요는 없고 기본값은 사실 동기 방식이라고 함.
- 명시적 요청이 없다면 RxJava는 동시성 처리를 하지 않음.
- 예컨대, 아래 코드는 동기 방식.

```java
Observable.create(s -> {
    s.onNext("Hello World!");
    s.onCompleted();
}).subscribe(hello -> System.out.println(hello));
```

- 물론, Observable을 동기화된 블로킹 I/O 방식으로 사용하는 것은 좋지 않은 예.
- 하지만, 동시성을 필요로 하지 않고, 비동기 스케쥴링을 하면 훨씬 더 느려지는 경우도 있음.

### Synchronous computation (such as operators)

- 동기를 유지하는 더 일반적인 이유는 아래의 방대한 API 사용 때문.
- `map`, `filter`, `take`, `flatMap`, `groupBy`, ...
- 스트림 조합과 변환 작업들.
- 이들은 이벤트를 전달받는 `onNext` 안에서 동기적으로 연산을 수행.
- 성능상의 이유로 그렇다고 함. 아래 코드에서 만약 `map`이 비동기로 수행된다면, 숫자별로 스레드가 할당되는데, 이는 비효율일 뿐만 아니라 스케쥴링, 컨텍스트 스위칭 등으로 인해 비결정적 지연이 발생.

```java
Observable<Integer> o = Observable.create(s -> {
  s.onNext(1);
  s.onNext(2);
  s.onNext(3);
  s.onCompleted();
});

o.map(i -> "Number " + i)
  .subscribe(s -> System.out.println(S));
```

- 대부분의 Observable 함수 파이프라인은 동기 방식. 반면, Observable 자체는 비동기일 수 있음. 아래가 그 예.

```java
Observable.create(s -> {
    ... async subscription and data emission ...
})
.doOnNext(i -> System.out.println(Thread.currentThread()))
.filter(i -> i % 2 == 0)
.map(i -> "Value " + i + " processed on " + Thread.currentThread())
.subscribe(s -> System.out.println("SOME VALUE =>" + s));

System.out.println("Will print BEFORE values are emitted");
```

- 위 예제에서 Observable은 비동기. 따라서, subscribe는 논블럭킹.
- 그리고 마지막 println은 이벤트 전파 전에 실행됨.
- filter와 map 함수는 이벤트를 방출하는 스레드에서 동기적으로 실행됨.
- 이것이 우리가 기대하는 동작: 비동기적 파이프라인과 효율적인 동기적 이벤트 연산.
- 실제 JUnit을 통해 확인해 보고자 아래와 같이 작성해 봄.

```java
Observable
  .<Integer>create(s -> {
    Executors.newCachedThreadPool().submit(() -> {
      s.onNext(1);
      Thread.sleep(500);
      s.onNext(2);
      Thread.sleep(500);
      s.onNext(3);
      Thread.sleep(500);
      s.onNext(4);
      Thread.sleep(500);
      s.onComplete();
      return null;
    });
  })
  .doOnNext(i -> System.out.println(Thread.currentThread()))
  .filter(i -> i % 2 == 0)
  .map(i -> "Value " + i + " processed on " + Thread.currentThread())
  .subscribe(s -> System.out.println("SOME VALUE =>" + s));

System.out.println("Will print BEFORE values are emitted" + Thread.currentThread());
for (int i = 0; i < 30; i++) {
    System.out.println("waiting.. " + i);
    Thread.sleep(100);
}
```