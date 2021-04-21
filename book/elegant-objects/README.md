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
