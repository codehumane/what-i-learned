# Effective Java, 3rd Edition

## Item 42. Prefere Lambdas to Anonymous Classes

### Classic Function Object

- 기존에는 1개의 추상 메소드로 이뤄진 인터페이스가 함수 타입<sup>function types</sup>으로 사용됨.
- 이들의 인스턴스는 함수 객체라고 알려져 있으며, 함수나 동작<sup>action</sup>을 표현.
- 함수 객체를 만드는 주요 방법은 주로 익명 클래스.
- 일종의 전략 패턴이기도 함.
- `Comparator` 인터페이스가 대표적.
- 하지만 이 방식은 다소 장황해보임.

### Functional Interface

- Java 8 이후의 Function Type은 Functional Interface. 기존에도 있었지만, 이름과 함께 특별하게 취급됨.
- *lambda expression* 또는 *lambda*로 인스턴스를 생성.
- 기존과 비슷하긴 하지만 보일러 플레이트가 사라지고, 행위가 더 잘 드러난다는 점에서 상대적으로 더 간결함.

### Type Inference

- 입력과 출력 타입이 명시되지 않지만, 문맥을 통해 컴파일러가 타입을 추론함. 즉, 타입 추론<sup>type inference</sup>.
- 이 타입 선언을 통해 프로그램이 더 명쾌해진다면, 혹은 컴파일러가 타입을 추론할 수 없어서 오류를 만들어 낸다면, 그 때 타입을 선언. 비록 장황해지더라도.
- 타입 추론 관련해서 주의할 점으로 3가지 언급.
  - Item 26: 원시 타입을 사용하지 말라.
  - Item 29: 제너릭 타입을 선호하라.
  - Item 30: 제너릭 메소드를 선호하라.

### Constant-Specific Class Body

- Constant-specific Class Bodies를 lambda로 대체 가능.
- 하지만, 람다는 이름과 문서화가 부족. 만약 연산 자체만으로 표현력이 부족하다면, 혹은 라인 수가 많아진다면, 람다에 넣지 말라.
- 혹은 인스턴스 필드나 메서드에 접근이 필요하다면 람다 대신, Constant-specific class bodies를 사용.

### Anonymous class

- abstract class의 인스턴스를 만들고 싶은 경우, 람다로는 이것이 불가하고, 대신 익명 함수로는 가능함.
- 익명 함수에서는 다수의 추상 메서드를 구현할 수도 있음.
- 또한, 람다는 자기 자신을 가리킬 수 없음. `this` 키워드는 자신을 감싸고 있는 인스턴스를 가리킨다.

### Serialization

- 람다는 익명클래스와 프로퍼티를 공유할 수 있음.
- 이 프로퍼티에는 신뢰성 있게 serialize/deserialize 할 수 없는 것들이 존재할 수 있음.
- 따라서, 가능한 조심스럽게 사용해야 함.

## Item 43. Prefere Method References to Lambdas

> Where method references are shorter and clearer, ues them: where they aren't, stick with lambdas.

### 언제 써야 하는가

- 익명 클래스에 비해 람다가 가지는 주요 이점은 간결함.
- 메서드 레퍼런스<sup>method reference</sup>는 이보다 더 간결.
- 아래는 이 차이를 보여주는 multiset implementation 사례.

```java
map.merge(key, 1, (count, incr) -> count + incr);
map.merge(key, 1, Integer::sum);
```

- `count`, `incr`는 보일러 플레이트 코드일 수 있음.
- 특별히 이름이 더해주는 가치는 없으며, 공간도 차지.
- 메서드 레퍼런스는 또한 이름이 있으므로 자기 설명적.
- 람다가 길어진다면, 메서드 레퍼런스로 대체하여, 이름으로 의도를 드러낼 수도.

### 언제 쓰지 말아야 하는가

- 하지만, 람다의 파라미터 이름이 유용한 문서로써 역할을 하는 경우도 있음.
- 또, 아래처럼 메서드 레퍼런스가 오히려 덜 간결할 수 있음.

```java
service.execute(HelloMethodReference::do);
service.execute(() -> do());
```

- `Function.identity()`도 유사한 사례.

### 메서드 레퍼런스의 종류

| 종류              | 예시                    | 등가 람다                                          |
| ----------------- | ----------------------- | -------------------------------------------------- |
| Static            | Integer::parseInt       | str -> Integer.parseInt(str)                       |
| Bound             | Instnant.now()::isAfter | Instant then = Instant.now(); t -> then.isAfter(t) |
| Unbound           | String::toLowerCase     | str -> str.toLowerCase()                           |
| Class Constructor | TreeMap<K, V>::new      | () -> new TreeMap<K, V>                            |
| Array Constructor | int[]::new              | len -> new int[len]                                |

- Bound, Unbound의 구분이 재밌음.
- `Instant.now()::isAfter`와 `int[]::new가 가능한 것이 또한 재밌음.