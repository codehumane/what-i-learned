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

