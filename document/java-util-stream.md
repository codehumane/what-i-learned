[Associativity](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#Associativity) 속성 때문에 [Package java.util.stream Description](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)을 읽다가, 그 밖에도 재밌는 내용이 많아 요약함.

# Package java.util.stream Description

스트림 패키지를 간단히 설명하면 다음과 같음.

- 스트림 엘리먼트에 대해 함수형 스타일 연산을 지원하는 클래스들.
- 아래 코드는 스트림에 대해 filter-map-reduce 연산을 수행하는 예.

```java
int sum = widgets.stream()
                 .filter(b -> b.getColor() == RED)
                 .mapToInt(b -> b.getWeight())
                 .sum();
```

스트림이 컬렉션과 다른 점.

- No storage. 엘리먼트 저장하는 대신, 파이프라인으로 실어 나름.
- Functional in nature. 원천<sup>source</sup>을 수정하지 않고 결과를 만들어 냄.
- Laziness-seeking.
  - 많은 스트림 연산들이 최적화를 위해 lazily로 구현 가능.
  - 예컨대, `findFirst`와 같은 것은 모든 엘리먼트를 검사하지 않아도 됨.
  - 스트림 연산은 2개로 나뉨. Intermediate operation(Stream-producing)과 Terminal operation(value- or side-effect-producing).
  - Intermediate operation들은 항상 lazy.
- Possibly unbounded. 컬렉션은 유한한 크기를 가지지만 스트림은 그럴 필요 X.
- Consumable. 스트림의 엘리먼트들은 스트림의 생애 동안 단 한 번만 방문 가능. Iterator와 유사.

## Stream operations and pipelines

- 스트림 연산은 intermediate과 terminal로 나뉨.
  - Intermediate 연산들은 항상 lazy. 연산을 수행하는 대신, 새로운 스트림을 만듦.
  - Terminal 연산들은 대부분 eager. lazy 연산들이 시작되고, 결과를 만들어 냄.
  - Terminal 연산이 수행되면 스트림은 소비 되었다고<sup>Consumed</sup> 여겨지고 더 이상 사용 불가.
  - 참고로, forEach도 Terminal 연산.
- Intermediate 연산은 stateless와 stateful 연산으로 다시 나뉨.
  - stateless는 `filter`나 `map` 처럼 연산이 각 엘리먼트에 대해 독립적.
  - stateful은 결과를 만들기 전 전체 입력을 처리해야 할 수도. 스트림 정렬이 그 예.
- 일부 연산들은 short-circuiting.
  - 무한 스트림으로부터 유한 스트림을 얻는 intermediate 연산이나,
  - 무한 입력을 유한 시간 내에 처리할 수 있는 terminal 연산을 가리킴.
  - short-circuiting은 무한 스트림 연산을 위한 필요조건<sup>necessary</sup>.
  - 충분조건<sup>sufficient</sup>은 아님.

### Non-interference

- 스트림은 다양한 데이터 소스에 대해 병렬 aggregate 연산을 지원.
- 심지어 `ArrayList`와 같은 non-thread-safe 컬렉션에 대해서도 지원.
- 다만, 스트림 파이프라인이 수행되는 동안 데이터가 간섭<sup>interference</sup>되지 않을 때에만 가능.
- 여기서의 간섭이란, 스트림 파이프라인이 수행되는 동안 데이터 소스가 수정될 수 있음을 가리킴.
- 아래는 간섭의 한 예시.

```java
List<String> l = new ArrayList(Arrays.asList("one", "two"));
Stream<String> sl = l.stream();
l.add("three");
String s = sl.collect(joining(" " ));
```

### Stateless behaviors

- 스트림 연산에 대한 행위적 파라미터<sup>behavioral parameter</sup>가 stateful이면,
- 파이프라인의 결과는 비결정적<sup>nondeterministic</sup>이거나 정확하지 않을 수 있음.
- 아래는 stateful의 예시. 병렬로 연산이 수행되면 결과가 매번 달라질 수도.
- 마찬가지 이유로, 행위적 파라미터에서 가변 상태<sup>mutable state</sup>에 접근을 시도하는 것도 안정성이나 성능 측면에서 나쁜 선택이 됨.
- 상태 접근을 synchronize 하지 않으면 데이터 레이스가 발생하고 코드가 깨질 수 있음.
- 반대로, 상태 접근을 synchronize 하면 병렬화로 인해 얻을 수 있는 이점이 적어짐.

```java
Set<Integer> seen = Collections
	.synchronizedSet(new HashSet<>());

stream
	.parallel()
	.map(e -> {
		if (seen.add(e)) return 0;
		else return e;
	})...
```

### Side-effects

- 너무 당연한 이야기지만, 스트림 연산에서 행위적 파라미터에 부수효과가 있는 것은 권장되지 않음.
- statelessness 요건을 의도치 않게 위배할 수도 있고, 스레드 안전성 위험도 가질 수 있음.
- 많은 경우에 부수 효과 없는 함수들로 교체 가능함. 아래는 그 예.

```java
ArrayList<String> results = new ArrayList<>();
stream.filter(s -> pattern.matcher(s).matches())
      .forEach(s -> results.add(s)); // Unnecessary use of side-effects!

List<String> results =
    stream.filter(s -> pattern.matcher(s).matches())
          .collect(Collectors.toList()); // No side-effects!
```

### Ordering

- 스트림에서는 encounter order가 있을 수도 있고 없을 수도 있음.
- 있고 없고는 데이터 원천과 중간 연산에 달려 있음.
- 스트림 원천인 `List`나 배열인 경우에는 순서가 있고, `HashSet`과 같은 경우에는 순서 없음.
- `sorted()`와 같은 중간 연산들은 정렬되지 않은 스트림에도 순서를 부과함.
- `BaseStream.unordered()` 같은 경우에는 정렬된 스트림을 무작위 순서로 렌더링 할 수도.
- 정렬된 스트림에 대한 대부분의 연산들은 순서 제약을 가짐. 예시) [1, 2, 3]을 가지는 `List`의 경우 `map(x -> x*2)`는 [2, 4, 6]의 결과를 보장해야 함.
- 병렬 스트림에 대해서 순서 제약을 풀어주는 것이 효과적인 실행을 이끌어내기도 함.

## Reduction operations

*reduction*(*fold*라고 불리기도 함) 연산은 일련의 입력 엘리먼트들을 받고, 이들을 결합하여 하나의 요약된 결과로 만듦. 요약된 결과란, 엘리먼트들의 합이나 최대값을 구하거나, 리스트 형태로 모아주거나 하는 것을 가리킴. 일반화된 reduction 연산인 [reduce()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-java.util.function.BinaryOperator-), [collect()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#collect-java.util.stream.Collector-)을 이용하거나, 좀 더 특화된 [sum()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#sum--), [max()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#max--), [count()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/IntStream.html#count--) 등을 사용할 수도 있음.

```java
int sum = 0;
for (int x: numbers)
    sum += x;

int sum = numbers.stream().reduce(0, (x,y) -> x+y);
int sum = numbers.stream().reduce(0, Integer::sum);
```

위 구현은 같은 결과를 만들어내는 서로 다른 구현. 하나는 mutative accumulation, 다른 하나는 reduce operation을 이용한 것. reduce operation이 상대적으로 가지는 이점은 2가지. 개별 엘리먼트에 대해서가 아니라 스트림 전체에 대한 연산이라는 점에서 좀 더 추상적이고, 또 `parallelStream` 추가만으로 병렬화가 가능함(단, associative이면서 stateless인 함수일 때만 가능).

### General form of *reduce*

reduce 연산의 가장 일반화된 형태는 3개의 파라미터를 받는 것.

```java
<U> U reduce(U identity,
             BiFunction<U, ? super T, U> accumulator,
             BinaryOperator<U> combiner);
```

각 인자에 대한 설명은 아래와 같음.

- **identity**
  - reduction을 위한 최초 시드 값이자, 아무 입력 값이 없는 경우의 기본값.
  - identity의 값은 combiner 함수의 identity여야 함. 즉, 모든 `u`에 대해 `combiner.apply(identity, u) == u` 가 보장돼야 함.
- **accumulator**
  - partial 결과와 다음 엘리먼트를 받고, 새로운 partial 결과를 만들어 냄.
- **combiner**
  - 2개의 partial 결과를 결합하여 새로운 partial 결과를 만듦. (병렬 reduction에 필수)
  - associative 이어야 하며, `accumulator` 함수와 호환되어야 함. 즉, 모든 `u` 와 `t`에 대해, `combiner.apply(u, accumulator.apply(identity, t))`가 `accumulator.apply(u, t)`에 `equals()`가 성립되어야 함.

이런 3개 인자를 받는 함수는 2개 인자를 받는 함수에 비해 일반화 된 형태이며, mapping 단계를 accumulation 단계로 통합시킨다. 아래는 이 둘에 대한 비교 사례.

```java
int sum = widgets.stream()
                 .filter(b -> b.getColor() == RED)
                 .mapToInt(b -> b.getWeight())
                 .sum();

int sumOfWeights = widgets.stream()
                          .reduce(0,
                                  (sum, b) -> sum + b.getWeight())
                                  Integer::sum);
```

map-reduce 형태가 더 가독성이 좋으며, 따라서 일반적으로 더 선호되어야 함. 위에서 이야기한 것처럼, mapping과 reducing을 한 곳에서 해야 하는 경우에는 3개 인자 형태를 사용.

### Mutable reduction

*mutable reduction operation*은 입력 엘리먼트들을 가변 결과 컨테이너<sup>mutable result container</sup>로 모음. `Collection`, `StringBuilder`와 같은 것이 그 예.

```jav
String concatenated = strings.reduce("", String::concat);
```

만약, 문자열들의 스트림을 하나의 긴 문자열로 만들고 싶다면 위와 같이 할 수 있음. 이는 심지어 병렬 작업에서도 잘 동작함. 하지만, 성능 면에서는 좋지 못함. 문자열의 갯수를 n이라고 할 때, 실행 시간이 O(n^2)라고 한다. `StringBuilder`로 결과를 모아주는 것이 성능 면에서 유리.

mutable reduction operation은 `collect()`라고 불림. 원하는 결과를 `Collection`과 같은 컨테이너로 모아주기 때문. 이 연산은 3개의 함수를 필요로 함.

- supplier: 결과 컨테이너 인스턴스 생성.
- accumulator: 입력 엘리먼트를 결과 컨테이너로 통합.
- combiner: 결과 컨테이너들을 하나로 병합.

`reduce()` 와 마찬가지로, collect를 이렇게 추상적인 방식으로 표현하게 되면 병렬화에 용이함. 물론, accumulator와 combiner가 associativity 등을 만족시켜야 하겠지만 말이다. 아래는 엘리먼트들을 문자열로 변환하여 `ArrayList`로 모으는 과정을 다양한 형태로 구현한 것.

```java
// for-each 형식을 활용한 문자열 모으기
ArrayList<String> strings = new ArrayList<>();
for (T element : stream)
    strings.add(element.toString());

// 쉽게 병렬화 할 수 있는 형태
ArrayList<String> strings = stream
    .collect(() -> new ArrayList<>(),
             (c, e) -> c.add(e.toString()),
             (c1, c2) -> c1.addAll(c2));

// mapping 연산을 accumulator로부터 분리
List<String> strings = stream
    .map(Object::toString)
    .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);

// Collector를 이용한 축약 버전
List<String> strings = stream
    .map(Object::toString)
    .collect(Collectors.toList());
```

표준 Collector를 이용하면 결합성<sup>composability</sup>라는 이점을 추가로 얻을 수 있음. [`Collectors`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)가 컬렉터를 위해 사전 정의된 여러 팩토리를 가지고 있기 때문. 아래는 그 예시.

```java
Collector<Employee, ?, Integer> summingSalaries
    = Collectors.summingInt(Employee::getSalary);

Map<Department, Integer> salariesByDept = employees
    .stream()
    .collect(Collectors
        .groupingBy(Employee::getDepartment, summingSalaries));
```

참고로, 위 코드에서 ? 기호가 사용된 것은 우리가 이 타입에 대해 관심 없음을 드러냄.

### Reduction, concurrency, and ordering

`Map`을 만들어내는 `collect()`와 같이 일부 복잡한 reduction 연산에서는 병렬화로 얻는 성능 상의 이점이 줄어들 수 있음. 일부 `Map` 구현체들에서는 combining 단계(2개의 `Map`을 키를 이용해 병합시키는)가 비용이 많이 들기 때문. 하지만, `ConcurrentHashMap`와 같이, 동시에 수정 가능한 결과 컨테이너라면, 하나의 공유된 컨테이너를 사용하게 되므로, 병합 절차가 필요 없게 됨.

이처럼 concurrent reduction을 지원하는 `Collector`들은 [`Collector.Characteristics.CONCURRENT`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html) 값을 통해 알 수 있음.

> Indicates that this collector is *concurrent*, meaning that the result container can support the accumulator function being called concurrently with the same result container from multiple threads.

또한, 순서가 무시될 수 있으므로, 순서가 중요하지 않은 스트림을 처리할 때만 concurrent reduction이 가능. (스트림이 unordered이거나, [`Collector.Characteristics.UNORDERED`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.Characteristics.html#UNORDERED)를 가지는 경우에 해당)

```java
Map<Buyer, List<Transaction>> salesByBuyer
    = txns.parallelStream()
        .unordered()
        .collect(groupingByConcurrent(Transaction::getBuyer));
```

### Associativity

`op`가 다음과 같다면 *associative*라고 할 수 있다.

```
(a op b) op c == a op (b op c)
```

associative operation의 예로는 덧셈, 최소값 또는 최대값 구하기, 문자열 연결 등이 있음. 뺄셈은 이런 결합 법칙이 성립하지 않음.

## 정리하며

어느 정도 안다고 생각했지만, 막상 전체를 정리하고 보니, 새로 알게 된 내용들이 있음. concurrent reduction를 위해 알고 있어야 하는 것, general form of reduce가 왜 필요한지 등. 그리고 스트림과 컬렉션의 차이 처럼, 암묵지가 형식지가 되는 경험도 있었고. 모름에는 끝이 없구나.