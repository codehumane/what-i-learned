# Reactive Programming with RxJava

## How RxJava Works

- 데이터나 이벤트 스트림을 나타내는 Observable 타입
- 밀어내기(reactive) 지향 but 끌어오기(interactive)도 가능
- 즉시가 아니라 지연 실행
- 비동기와 동기 모두 사용 가능
- 시간에 따라 0, 1, 다수 혹은 무한 개의 이벤트를 다룰 수 있음.

### Push versus Pull

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

### Async versus Sync

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

### Concurrency and Parallelism

병렬성(praralleism)

- 동시에 수행하는 작업들.
- 일반적으로 서로 다른 CPU나 기기에서 처리.
- 동시성의 특수한 형태.

동시성(concurrency)

- 여러 작업들을 합성하거나 번갈아 수행.
- 하나의 CPU가 여러 스레드들을 처리하는 것은 시 분할. 병렬 실행 아니고 동시 실행.

Observable 스트림의 직렬성.

- onNext, onCompleted, onError 이벤트는 동시에 방출되지 않음.
- 항상 직렬화되어 스레드에 안전해야 함.
- Observable의 규약임.
- 이에 따라, 아래의 예제는 잘못된 사용. 규약 위반.
- 두 개의 스레드가 동시에 onNext를 호출할 수 있음.

```java
Observable.creaet(s -> {
  new Thread(() -> {
    s.onNext("one");
    s.onNext("two");
  }).start();
  
  new Thread(() -> {
    s.onNext("three");
    s.onNext("four");
  }).start();
})
```

- 동시성과 병렬성의 이점을 취하고자 한다면 구성(composition) 사용을 권장.
- merge와 flatMap 등이 자주 사용되는 이유이기도.
- 아래는 그 예.

```java
Observable<String> a = Observable.create(s -> {
  new Thread(() -> {
    s.onNext("one");
    s.onNext("two");
    s.onCompleted();
  }).start();
});

Observable<String> a = Observable.create(s -> {
  new Thread(() -> {
    s.onNext("three");
    s.onNext("four");
    s.onCompleted();
  }).start();
});

Observable<String> c = Observable.merge(a, b);
```

왜 동시에 `onNext()`를 호출하면 안 될까?

- 일단, `onNext()`는 사람이 사용하기를 의도한 것. 동시성은 어려움.
- 게다가, `scan`이나 `reduce` 같은 몇몇 연산자는 동시 방출이 불가.
- 교환법칙이나 결합법칙이 성립하지 않는 이벤트들의 스트림을 그대로 축적하기 위한 연산.
- 마지막으로, 모든 옵저버와 연산자가 thread-safe여야 한다면 동기화 오버헤드로 인한 성능 저하가 발생.
- 실제로, `.parallel(Function f)` 연산자가 원래 존재했지만, 라이브러리 버전 1에서 제거했다고 함.

### Lazy versus Eager

- Observable 타입은 lazy.
- 구독되기 전까지 아무 것도 하지 않음.
- Future와 같은 eager 타입과는 구분됨.
- 따라서, 캐싱을 쓰지 않고도(Future와 비교됨),
- 레이스 컨디션으로 인한 데이터 유실 없이,
- Observable들의 구성이 가능함.

책에서는 아래 2가지로 다시 설명하고 있음.

1. Subscription, not construction starts work.
2. Overvables can be reused.

재사용의 의미는 아래 코드로 확인 가능.

```java
someData.subscribe(s -> println("Subscriber 1: " + s));
someData.subscribe(s -> println("Subscriber 2: " + s));
```

### Duality

- Rx Observable은 Iterable의 비동기 "쌍대(dual)".
- 다시 말해서, Observable은 Iterable의 모든 기능을 제공하지만,
- 데이터의 플로우는 정 반대. pull이 아니라 push.

| Pull ()  | Push ()       |
| -------- | ------------- |
| T next() | onNext()      |
|          | onError()     |
|          | onCompleted() |

- 이로 인해, Iterable로 할 수 있는 건 Observable로도 할 수 있음.

```java
getDataFromLocalMemorySynchronously()
  .skip(10)
  .limit(5)
  .map(s -> s + "_transformed")
  .forEach(System.out::println);

getDataFromNetworkAsynchronously()
  .skip(10)
  .take(5)
  .map(s -> s + "_tranformed")
  .subscribe(System.out::println);
```

### Cardinality

- Observable은 여러 개의 값들을 비동기로 푸시할 수 있음.
- `Future<List<Friend>>` 같은 식으로 할 수도 있겠으나,
- 반환해야 할 목록이나 데이터 소스가 크다면, 성능이나 반응 시간 측면에서 좋지 않음.
- 무엇보다도, 컬렉션 전체가 도착할 때까지 기다리지 않고, 항목을 받는 즉시 처리 가능.
- 구성 또한 가능. 모든 방출을 기다리는 `zip`이나, 각 값을 즉시 방출하는 `merge` 등을 이용.

```java
Observable<String> o1 = getDataAsObservable(1);
Observable<String> o1 = getDataAsObservable(2);
Observable.merge(o1, o2);
Observable.zip(o1, o2);
```

### Single

Single 형도 제공함. API를 사용할 때 보다 단순하게 생각할 수 있음.

```java
public static Single<String> getDataA() {
  return Single.<String> create(o -> {
    o.onSuccess("DataA");
  }).subscribeOn(Schedulers.io());
}

public static Single<String> getDataB() {
  return Single.just("DataB").subsribeOn(Schedulers.io());
}

getDataA().mergeWith(getDataB());
```

### Completable

반환형이 없을 때는 `Observable<Void>`나 `Single<Void>`를 사용할 수 있으나 다소 어색할 수 있음. 이 때, Completable 형을 사용하는 게 도움이 될 수 있음.

```java
static Completable writeToDatabase(Object data) {
  return Completable.create(s -> {
    doAsyncWrite(
      data,
      () -> s.onCompleted(),
      error -> s.onError(error)
    )
  });
}
```

### Zero to infinity

|      |                           |                     |                         |
| ---- | ------------------------- | ------------------- | ----------------------- |
|      | void doSomething()        | T getData()         | Iterable<T> getData()   |
|      | Completable doSomething() | Single<T> getData() | Observable<T> getData() |



