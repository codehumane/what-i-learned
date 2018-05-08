# 6. Object-Oriented Style

> Always design a thing by considering it in its next larger context—a chair in a room, a room in a house, a house in an environment, an environment in a city plan.

## Introduction

- 작성하기 쉬운 코드보다 유지보수 하기 쉬운 코드를 더 가치 있게 여김.
- 장기적인 관점<sup>long-term concern</sup>과 단기적인 것 사이의 균형을 중요시.

## Designing for Maintainability

코드가 늘어남에도 가독성과 유지보수성을 유지할 수 있는 유일한 방법은 기능을 객체로, 객체를 패키지로, 패키지를 프로그램으로, 프로그램을 시스템으로 구조화 하는 것. 이런 구조화를 위한 2가지 휴리스틱을 소개.

1. Separation of Concern
2. Higher levels of abstraction

먼저, **Separation of Concern**

> we gather together code that will change for the same reason.

- 코드는 한 번에 한 곳에서 같은 이유로 바뀌어야 함.
- 이를 위해, 관심사가 같은 것들을 한 곳에 모아둠.
- 예컨대, 인터넷 프로토콜을 해석하는 코드와 비즈니스 코드는 서로 분리.

다음으로, **Higher levels of abstraction**

> The only way for humans to deal with complexity is to avoid it, by working at higher levels of abstraction.

정말 오래 전에 읽었었는데 아직도 기억하는 문구. 정말로 와닿았었기 때문. 하지만, 추상화에 대해 균형을 잡게 도와준 글도 함께 떠오름. 다름 아닌, [The Law of Leaky Abstraction](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/). 이 글의 결론은 이렇다.

> So the abstractions save us time working, but they don't save us time learning.

다시 돌아와서 책의 내용을 정리하면 다음과 같음.

- 변수를 조작하고 흐름을 제어하는 대신, 유용한 기능들을 가진 컴포넌트를 조합.
- 메뉴에서 음식을 주문할 뿐, 접시를 고르고 레시피를 일일이 지정하지 않는 것과 같음.
- 더 많은 일을 좀 더 쉽게 할 수 있게 도와줌.

### Ports and adapters

관심사의 분리와 더 높은 수준의 추상화가 반복 적용되면, Cockburn의 "ports and adapters" 아키텍처와 같은 방향으로 구조가 만들어질 수 있다고 이야기. 즉, 비즈니스 도메인 코드가 데이터베이스나 사용자 인터페이스와 같은 기술적 인프라스트럭처로부터 분리됨. 

![ports and adapters](http://www.natpryce.com/articles/images/ports-and-adapters-architecture.png)

그림 출처는 [여기](http://www.natpryce.com/articles/000772.html).

## Internals vs. Peers

객체 내부에 둘 것인가 외부에 둘 것인가를 결정해야 함.

- Internals: 객체는 내부로의 접근을 캡슐화하고, 세부사항을 외부로부터 숨김.
- Peers: 그리고 객체 간에는 메시지를 주고 받음으로써 통신함.

이 결정은 시스템 품질 측면에서 중요. 객체 내부를 너무 많이 노출하게 되면, 객체의 행위가 여러 클라이언트로 분산됨. 결과적으로 하나의 변경이 여러 코드에 걸쳐 일어나게 됨. 열차 전복<sup>train wreck</sup>으로도 알려져 있음. [The Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter) 함께 참고.

## No And's, Or's, Or But's

> We should be able to describe what an object does without using any conjunctions ("and", "or").

위 문장은 [단일 책임 원칙](https://en.wikipedia.org/wiki/Single_responsibility_principle)<sup>Single Responsibility Principle</sup>을 준수하기 위한 한 가지 휴리스틱. 시스템에 행위를 추가할 때, 기존 객체를 확장해야 하는지 아니면 새로 만들어야 하는지에 대한 기준이 됨. 또한, 여러 객체들을 패키지로 추상화 할 때도 마찬가지. 그 패키지를 명확하고 단순하게 설명할 수 있는가?

## Object Peer Stereotypes

단일 책임을 가진 객체가 있고, 명료한 API를 통해 동료들과 메시지를 주고 받을 수 있다. 그런데 서로 무엇을 말해야 하는가?

객체의 동료는 대략 3가지 관계로 범주화 할 수 있음.

**Dependencies**

- 자신의 책임을 수행하기 위해 필요한 동료 객체들.
- 또한, 그들이 없으면 생성이 불가함.

**Notifications**

- 나의 활동 상황을 최신 상태로 알 수 있는 동료 객체들.
- 단지 "fire and forgot". 동료 객체가 수신하고 있는지 조차 모름.

**Adjustments**

- 나의 행동을 시스템의 더 넓은 요구에 맞게 조정하는 동료 객체들.
- 좀 더 구체적으로는, 객체를 대신해 결정을 내려주는 정책 객체나, 컴포짓인 경우 그 대상 컴포넌트들을 가리킴.
- Swing의 `JTable`은 `TableCellRenderer`에게 cell의 값을 표기해 달라고 요청. 렌더러를 변경하면 RGB를 표기하던 테이블이 HSB 값을 표기하게 될 수도.

이들 스테레오타입은 강제된 규칙이 아니라 단지 설계를 생각하게 해주는 휴리스틱. 예를 들어, 감사 로그가 법적 의무인 어플리케이션에서는 이 로그 객체가 depndency일 수 있음. 반면, 감사 로그가 사용자의 선택이라면, notification을 생각해 볼 수도. 혹은 notification을 단방향으로 바라볼 수도 있다. 반환 값도 없고, 호출자를 다시 호출할 수도 없으며, 예외도 던질 수 없기 때문. 한편, dependencies와 adjustment는 notification에서는 가능하다. 따라서, 직접적인 관계.

## Composite Simpler Than The Sum Of Its Parts

TBD















