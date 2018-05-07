# 6. Object-Oriented Style

> Always design a thing by considering it in its next larger context—a chair in a room, a room in a house, a house in an environment, an environment in a city plan.

## Introduction

- 작성하기 쉬운 코드보다 유지보수 하기 쉬운 코드를 더 가치 있게 여김.
- 장기적인 관점<sup>long-term concern</sup>과 단기적인 것 사이의 균형을 중요시.

## Designing for Maintainability

코드가 늘어남에도 가독성과 유지보수성을 유지할 수 있는 유일한 방법은 기능을 객체로, 객체를 패키지로, 패키지를 프로그램으로, 프로그램을 시스템으로 구조화 하는 것. 이런 구조화를 위한 2가지 휴리스틱을 소개.

1. Separation of Concern
2. Higher levels of abstraction

### Separation of Concern

> we gather together code that will change for the same reason.

- 코드는 한 번에 한 곳에서 같은 이유로 바뀌어야 함.
- 이를 위해, 관심사가 같은 것들을 한 곳에 모아둠.
- 예컨대, 인터넷 프로토콜을 해석하는 코드와 비즈니스 코드는 서로 분리.

### Higher levels of abstraction

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

TBD