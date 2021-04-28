# Don't Create Objects That End With -ER

https://www.yegor256.com/2015/03/09/objects-end-with-er.html

- 클래스 이름은 객체가 노출하고 있는 기능에 기반 X.
- 무엇을 하는지(what he does) X
- 대신, 무엇인지(what it is)에 기반 O.

```java
List<Apple> sorted = new Sorter().sort(apples);
Apple biggest = sorted.get(0);
```

- 위 코드 대신 아래처럼 할 것을 권장.

```java
List<Apple> sorted = new Sorted(apples);
Apple biggest = sorted.get(0);
```

- 객체는 절차의 집합이 아니라고 주장.
- 하지만, 이름이 -er로 끝나면 이 역할이 강조됨.
- 대신, 객체는 캡슐화된 데이터의 대표자<sup>representative</sup>여야 한다고 함.
- 스스로 결정을 내리고 행동할 수 있는 자립적인 엔티티.
- 한편, [여기 글](https://www.pragmaticobjects.com/chapters/014_traits_of_high_quality_abstractions.html#trait-6-naming-your-interfaces-with-the-names-of-design-patterns-is-deep-mistake)에 보면, 네이밍의 어려움 때문에 저자는 -ER 규칙을 다음과 같이 바꿔서 말하고 있음.

> Never name your objects with the names of software design patterns

# Constructors Must Be Code-Free

https://www.yegor256.com/2015/05/07/ctors-must-be-code-free.html

```java
public final class EnglishName implements Name {
  private final String name;
  public EnglishName(final CharSequence text) {
    this.name = text.toString().split(" ", 2)[0];
  }
  @Override
  public String first() {
    return this.name;
  }
}
```

- 객체를 인스턴스화 하는 일과 객체가 일을 하게 하는 것은 서로 다른 일.
- 인스턴스화 중에 객체 호출자가 요청하지도 않은 일을 하는 것은 사이드 이펙트.
- 위 코드에서는 줘진 문자열 할당 외에 문자열 자르기를 함께 수행.
- 객체만 생성되고 `name`은 생명주기 동안 쓰이지 않을 수도 있으므로 성능 면에서도 비효율일 수도.
- 만약, 객체가 생성되면 `name`은 무조건 호출되며 여러 번일 수 있다고 하더라도, 아래와 같이 [composable decorator](https://www.yegor256.com/2015/02/26/composable-decorators.html)로 극복 가능.

```java
public final class CachedName implements Name {
  private final Name origin;
  public CachedName(final Name name) {
    this.origin = name;
  }
  @Override
  @Cacheable(forever = true)
  public String first() {
    return this.origin.first();
  }
}

final Name name = new CachedName(
  new EnglishName(
    new NameInPostgreSQL(/*...*/)
  )
);
```

- 하지만, 객체 생성 시 validation은 큰 이점을 가져다 줌.
- 저자의 [다른 글](https://www.yegor256.com/2018/05/29/object-validation.html)에서도 이와 관련된 내용을 찾을 수 있음.
- 그러나 validation 코드를 생성자에 넣는 것에 동의하면서도, connecting과 talking을 구분하며, connecting만을 생성자 validation에 허용.

# Encapsulate as Little as Possible

- 객체 안에 객체를 캡슐화 할 때, 최대 4개까지만 하라는 이야기.
- 우리의 사고방식이 4개 이상의 요소로 구성된 객체를 이해하기 어렵다는 것을 근거로 삼고 있음.

# Encapsulate something at the very least

https://www.yegor256.com/2014/12/15/how-much-your-objects-encapsulate.html

- "최소한 뭔가는 캡슐화하세요"보다는 위 링크가 좀 더 균형잡힌 글.
- 위 링크에서는 객체가 어떨 때 값을 캡슐화 해야 하는지,
- 그리고 어떨 때 값을 캡슐화 하지 말아야 할지 언급.
- 실 생활에서의 엔티티를 표현하고 있다면 전자를 선택.
- Universe를 나타낸다면 후자를 선택.
- Universe가 적절한 예시는 아래 참고.

```java
class HTTP {
  public String read(String url) {
    // read via HTTP and return
  }
  public boolean online() {
    // check whether we're online
  }
}
```

- Universe에 해당하는 이 `HTTP` 객체는,
- 어떤 웹 페이지든 읽어들일 수 있고,
- 어떤 웹 페이지든 접근이 가능한지 여부를 판별할 수 있음.
- 하지만 이런 Universe는 대부분의 경우 필요치 않다고 주장.

# Public Static Literals ... Are Not a Solution for Data Duplication

- `new String(array, "UTF-8")` 코드가 여러 군데 있다고 해보자.
- 매번 바이트 배열을 `String`으로 만들기 위해 "UTF-8"을 사용.
- Apache Commons가 그랬던 것처럼, `CharEncoding.UTF_8`을 정의해 두는 것이 편할 수도.

```java
package org.apache.commons.lang3;
public class CharEncoding {
  public static final String UTF_8 = "UTF-8";
  // some other methods and properties
}

// byte array -> String 변환
import org.apache.commons.lang3.CharEncoding;
String text = new String(array, CharEncoding.UTF_8);

// String -> byte array 변환
import org.apache.commons.lang3.CharEncoding;
byte[] array = text.getBytes(CharEncoding.UTF_8);
```

- 하지만 `public static` 프로퍼티는 결합도를 높이고 응집도를 낮춘다고 주장.
- [유틸리티 클래스](https://www.yegor256.com/2014/05/05/oop-alternative-to-utility-classes.html) 만큼 나쁘다고 이야기.
- `CharEncoding.UTF_8`이 바뀔 때 문자열과 바이트 배열의 변환부는 즉각 영향 받음.
- 물론, 잘 바뀌지 않는다면, 그리고 단순한 경우라면 문제가 되지 않을 것.
- 그러나 기능이 확장될 수 있다거나, 대체될 수 있다거나, 복잡한 경우 등에는 문제가 될 수도.
- 응집력은 위 예시로 설명은 어려움. UTF8 상수와 함께 UTF8로의 로직이 한 곳에 있어야 함을 의미.
- 그래서 아래와 같이 바꿀 수 있음.

```java
String text = new UTF8String(array); // 하지만 Java는 String을 final로 선언
String text = new UTF8String(array).toString(); // 따라서 이렇게 해야 함
```

- 상수에 대한 낮은 응집도와 높은 결합도가 문제되는 경우가 개인적으로 흔치는 않다고 생각.
- 하지만, 관련된 로직과 캡슐화 되지 않아서, 기능 확장과 변경에 어려움을 겪었던 일이 떠오르는 것도 사실.

# Objects Should Be Immutable

http://goo.gl/z1XGjO

- 불변 객체는 많은 이점을 가짐.
- 다만, 저자가 드는 근거들에 억지스러운 측면도 없지 않음.
- 식별자 가변성(Identity Mutability)
  - 식별자가 바뀌는 경우는 심각한 문제.
  - 하지만 값 객체일 경우는 심각도가 낮음.
- 실패 원자성(Failure Atomicity)
  - 불변이 아니라고 실패 원자성을 가지는 것은 아님.
  - 하지만 불변이 아닐 경우 추가적 노력이 들어가는 것은 분명.
- 시간적 결합(Temporal Coupling)
  - 2개 이상 필드가 할당되는 순서가 중요한 경우라면 필드를 함께 받는 메서드를 만들면 됨.
  - 하지만 마찬가지로 추가적 노력이 필요.
- 부수효과 제거(Side effect-free)
  - 불변에 부수효과가 없는 것은 큰 장점.
  - 유지보수와 문제 추적에 드는 비용이 월등히 적음.
- NULL 참조 없애기
  - 불변 이야기라기 보다는 여러 책임을 가지는 비대한 클래스의 문제에 가까움.
- 스레드 안정성
  - POEAA 책에서는 "둘 이상의 사용자가 동일한 데이터를 수정하려 하는 경우"가 동시성 문제의 조건이라 했음.
  - 문제 조건 자체가 사라지는 것.
  - 다만, 이 또한 주요 이점으로 보기는 어려워 보임.
- 더 작고 더 단순한 객체
  - 단순함.
  - 이것이 부수효과 제거와 함께 갖는 가장 큰 이점이라 생각.

# Don't mock; use fakes

https://www.yegor256.com/2014/04/18/jcabi-http-server-mocking.html

- mock을 쓰면 여러 곳에서 코드가 반복됨.
- 이 mock 대상이 바뀌면 이 여러 곳을 같이 바꿔줘야 함.
- 그리고 mock은 실제 코드를 모델링하는 정확도가 낮음. 대신 우리가 듣고 싶은 것을 말해줄 뿐.
- 이로 인해, 테스트는 성공하지만, 프로덕션 환경에서 문제가 발견.
- [tl;dw: Stop mocking, start testing](https://nedbatchelder.com/blog/201206/tldw_stop_mocking_start_testing.html) 글에 나오는 교훈 중 아래 2가지가 인상적.
  1. Share mocks among test modules.
  2. Maybe you don’t need a mock: if an object is cheap, then don’t mock it.
- 대신, fake 클래스를 사용할 것을 권장.
- 단위 테스트가 좀 더 간결해지고(그러나 그만큼 정보가 숨겨지기도),
- 테스트 대상 객체가, mocking된 행위가 아닌 다른 행위를 사용하도록 바뀌어도, 테스트 코드를 일일이 바꿔주지 않을 수도 있음.
- 하지만 fake 클래스를 만드려면 인터페이스가 있어야 한다는 제약이.
- 그리고 여전히 우리가 듣고 싶은 것을 말해주는 것에서 자유롭진 못함.

# Keep interfaces short; use smarts

https://www.yegor256.com/2016/04/26/why-inputstream-design-is-wrong.html

- 인터페이스 크다면 SRP가 깨진 것은 아닌지 의심.
- 정말 이 인터페이스를 사용하는 곳에서 이 모든 메서드들 필요로 할까?

```java
interface Exchange {
  float rate(String target); // 환율을 알 수 없을 경우 기본 환율을 사용하여 변환
  float rate(String source, String target);
}
```

- 위 예제에서 `rate(String target)`은 필요 없고 오히려 구현체들에게 너무 많은 것을 강요한다고 이야기.
- 그렇다고 또 다른 인터페이스를 만드는 것도 비효율.
- 그래서 아래와 같이 하라고 함.

```java
interface Exchange {
  float rate(String source, String target);

  final class Smart {
    private final Exchange origin;
    public float toUsd(String source) {
      return this.origin.rate(source, "USD");
    }
  }
}
```

- 구현체들에게 너무 많은 것을 강요하지도 않고,
- 구현체들의 중복 구현도 피할 수 있음.
- 조금 다른 예시는 아래와 같음.

```java
interface Exchange {
  float rate(String source, String target);

  final class Fast implements Exchange {
    private final Exchange origin;

    @Override
    public float rate(String source, String target) {v
      final float rate;

      if (source.equals(target)) {
        rate = 1.0f;
      } else {
        rate = this.origin.rate(source, target);
      }

      return rate;
    }

    public float toUsd(String source) {
      return this.origin.rate(source, "USD");
    }
  }
```

- [Why InputStream Design is Wrong](https://www.yegor256.com/2016/04/26/why-inputstream-design-is-wrong.html)에서는 `InputStream`을 예시로 들고 있음.

# Don't use static methods

https://www.yegor256.com/2014/05/05/oop-alternative-to-utility-classes.html

- 일단, Apache Commons의 [`FileUtils`](http://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/FileUtils.html)를 이용한 절차적 예시.

```java
void transform(File in, File out) {
  Collection<String> src = FileUtils.readLines(in, "UTF-8");
  Collection<String> dest = new ArrayList<>(src.size());
  for (String line : src) {
    dest.add(line.trim());
  }
  FileUtils.writeLines(out, dest, "UTF-8");
}
```

- 그리고 아래는 똑같은 기능을 객체지향 스타일로 구현한 것.

```java
void transform(File in, File out) {
  Collection<String> src = new Trimmed(
    new FileLines(new UnicodeFile(in))
  );
  Collection<String> dest = new FileLines(
    new UnicodeFile(out)
  );
  dest.addAll(src);
}
```

- 절차적 방식에서는 세부적인 것들까지 직접 관여하는 반면,
- 객체지향 방식에서는 세부적인 것들은 다른 객체들을 생성해서 위임하고 조합.
- 이는 좀 더 이해하기 쉽고 유지보수하기 쉬우며 단위테스트 하기에도 좋음.
- 실제로 `FileUtils`는 3,000라인이 넘어가며 80개가 넘는 메서드를 갖고 있다고.
- 그리고 객체지향 방식에서는 게으른 수행<sup>lazy execution</sup>이 가능.

## declarative vs. imperative style

- 명령형에서는 '프로그램의 상태를 변경하는 문장을 사용해서 계산 방식을 서술'
- 선언형에서는 '제어 흐름을 서술하지 않고 계산 로직을 표현'
- 표현이라 함은 아래 예시처럼 정의하는 것. CPU에게 할 일을 지시하지 않음.

```
// max의 행위를 정의
(defun max (a b)
  (if (> a b) a b)

// 그리고 이를 활용해서 x를 (max 5 9)에 바인딩
// 최댓값을 계산하라고 CPU에게 명령하지 않고 있음
(def x (max 5 9))
```

- 이번엔 Java 예시. 먼저 명령형 스타일.

```java
public static int between(int l, int r, int x) {
  return Math.min(Math.max(l, x), r);
}
```

- `between`을 호출하자마자 계산이 되고 9라는 결과를 반환 받음.
- 한편, 아래는 선언형 스타일.

```java
class Between implements Number {
  private final Number num;
  
  Between(Number left, Number right, Number x) {
    this.num = new Min(new Max(left, x), right);
  }

  @Override
  public int intValue() {
    return this.num.intValue();
  }
}

// 이를 사용할 땐 아래처럼
Number y = new Between(5, 9, 13);
```

- `y.intValue()`는 실제 결과가 필요한 시점까지 실행을 미룰 수 있음.

## declarative 방식이 주는 이점

먼저, 빠름.

- 명령형은 객체를 생성하지 않고 바로 연산을 수행하므로 어떤 경우엔 빠를 수도.
- 그러나 선언형은 성능 최적화를 제어할 수 있음.
- 제어권을 객체에게 넘겼기에, 조건에 따라 필요한 경우에만, 필요한 메서드만을 선택적으로 호출할 수 있음.
- 명령형은 9라는 값이 필요 없는 경우에도 계산을 수행.

다음으로, 다형성이 가능.

- 앞선 예제의 `between`을 다른 알고리즘으로 해결하고 싶을 수도.
- 이 경우 아래와 같이 생성자 추가 후 다른 클래스를 조합해서 사용할 수 있음.
- 하지만 명령형에서는 메서드를 새로 추가해야 하고 일일이 관련 부분을 바꿔줘야 함.

```java
class Between implement Number {
  ...
  Between(Number number) {
    this.num = number;
  }
}

// 사용
Integer x = new Between(new OtherAlgorithm(5, 9, 13));
```

세 번째로 표현력이 좋음.

- 어느 정도는 동의하지만, 개인적으로는 저자가 더 중요한 것을 놓친다고 생각.
- 기록도 생략.

마지막 네 번째는 응집도.

- 관련된 것들끼리 모아두므로,
- 변경이 일어날 때 관련 없는 것은 보지 않아도 되며,
- 문제가 될 가능성도 줄어듦.

# Never accept NULL arguments

https://www.yegor256.com/2014/05/13/why-null-is-bad.html

- 위 링크에 보면, 'Computer Thinking vs. Object Thinking' 내용이 있는데, 여기에 나오는 대화가 흥미로움.

> - Hello, is it a software department?
> - Yes.
> - Let me talk to your employee "Jeffrey" please.
> - Hold the line please...
> - Hello.
> - Are you NULL?

- 마지막 질문이 참 어색하지 않냐는.
- 대신, Jeffrey가 없다면 전화가 끊어지게 할 수도 있음(Exception).
- 그럼 다시 전화를 걸거나, 관리자에게 전화를 걸어 다른 대응을 할 수도.
- 혹은, Jeffrey는 아니지만 다른 사람을 연결해 줄 수 있음.
- 대신해서 도움을 주거나, Jeffrey가 아니면 할 수 없는 경우 요청을 거절하는(Null Object).
- "Ad-hoc Error Handling"도 이야기.
- null이 반환되면 곳곳에서 null 체크를 해야함. 코드가 점점 오염됨.
- "Ambiguous Semantic"은 null을 반환하는 `getByName()` 메서드가 `getByNameOrNullIfNotFound()`로 바뀌어야 함.
- 이런 네이밍이 많은 메서드에 적용된다면 그 모습은 끔찍.
- 그 외 "Slow Failing"과 "Mutabe and Incomplete Objects"는 다소 논란의 여지가 있어 보여 기록 생략.
- NULL의 대안은 2가지. Null Object를 사용하거나, 빠른 null 체크로 예외를 던지라고 이야기.

# Be loyal and immutable, or constant

https://www.yegor256.com/2014/12/22/immutable-objects-not-dumb.html

- 모든 것을 불변객체로 만들 수 있는 방법들에 대해 이야기.
- 값을 변경해야 할 때는 바뀌는 값과 함께 객체를 새로 만드는 방법을 여기서는 상수 방식이라 부름.
- 불변은 우리가 흔히 아는 방식. 아래 2개 예시는, 실제 세상에서는 상태가 변하는 것을, 객체는 불변으로 표현한 것.

```java
@Immutable
class Page {
  private final URI uri;
  Page(URI addr) {
    this.uri = addr;
  }
  public String load() {
    return new JdkRequest(this.uri)
      .fetch().body();
  }
  public void save(String content) {
    new JdkRequest(this.uri)
      .method("PUT")
      .body().set(content).back()
      .fetch();
  }
}

Class ImutableList<T> {
  private final List<T> items = LinkedList<T>();

  void add(T number) {
    this.items.add(number);
  }

  Iterable<T> iterate() {
    return Collections.unmodifiableList(this.items);
  }
}
```

# Never use getters and setters

https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html

- Tell, Don't Ask.
- getter, setter는 로직이 이곳 저곳에 산재하게 함.
- 이는 변경의 영향범위를 크게 만들고, 충돌과 빠뜨림과 더 많은 테스트와 버그 등을 야기.
- 자율적인 대상으로 간주하고 커뮤니케이션에 초점.
- 위 링크에서의 Object Thinking에는 개인적으로 반대.
- 살아있는 유기체는 생애 주기 동안 상태가 바뀔 수 있음. 불변 객체가 이를 잘 반영하는 것일지.
- 이 보다는 유지보수와 시스템 확장의 용이함 등의 실용적 이유로 접근하는 것이 좋아 보임.

# Never return NULL

https://www.yegor256.com/2014/05/13/why-null-is-bad.html

- "Never accept NULL arguments"에서와 마찬가지 이유로 NULL 사용 반대.
- 반환 값으로도 사용하지 말라는 것이 차이.
- 대안으로 제시하는 것은 3가지.
- 첫 번째: `boolean exists` 메서드를 하나 더 추가하라는 것.
- 두 번째: 컬렉션을 반환하라는 것.
- 세 번째: 널 객체 사용.
- 개인적으로는 첫 번째와 두 번째 방법이 별로라 생각.
- 여전히 클라이언트 코드가 null인지 아닌지 판별해야 함.
- 그런데, `Optional`을 반환하는 것에 비해 몇 가지 단점을 더 가질 뿐임.
- 예컨대, `exists`를 매번 호출해야 하는 것은 놓치기 쉽고, 성능 문제를 야기할 수 있으며, 변경이 일어날 때 2개를 같이 챙겨줘야 하는 부담감도 있음.
- 또한, 컬렉션은 집합을 의미하기에 `Optional`보다 더 혼란을 일으킬 가능성이 큼.
- 그래서 Optional을 사용하거나, 널 객체 패턴을 사용하거나, null이 필요 없도록 만들고 혹여나 발생하면 예외를 던지게 하는 방법들이 좋다고 생각.
- 그리고 어쩔 수 없는 상황에는 null 사용하되 `@Notnull`로 명시하는 것이 불필요한 클라이언트 코드의 방어 로직을 제거.

# Throw only checked exceptions

https://www.yegor256.com/2015/07/28/checked-vs-unchecked-exceptions.html

- 저자의 의견에 반대하는 부분이 많음. 개인적 생각 적고 마무리.
- Unchecked는 고통스러움. 따라서, 이 비용을 뛰어넘는 이유(예컨대 finally에서 리소스 해제를 꼭 해야 한다던가)가 있을 때만 사용해야.
- 하나의 예외 타입으로 절대 충분할 수 없음. AOP 등에서 상황에 따라 적절히 처리하기 위해서는 구체적인 에외 타입을 구분할 수 있어야 함.
- 일반적으로는 런타임 예외로 충분.
- https://stackoverflow.com/questions/6115896/understanding-checked-vs-unchecked-exceptions-in-java
