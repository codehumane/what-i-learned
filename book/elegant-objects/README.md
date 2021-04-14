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
