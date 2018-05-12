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

> When composing objects into a new type, we want the new type to exhibit simpler behavior than all of its component parts considered together. The composite object's API must hide the existence of its component parts and the interactions between them, and expose a simpler abstraction to its peers.

컴포짓 객체의 API는 그 객체의 컴포넌트들의 단순 합보다 단순해야 함. 예컨대, 컴포넌트 끼리의 상호작용이나, 나아가서는 어떤 컴포넌트가 있는지 등. 물론, 하나의 컴포넌트만을 소유한 경우라도 적용될 수 있는 규칙. 아래는 이해를 돕는 코드.

```java
// moneyEditor가 amountField, currencyField라는 필드를 소유함이 드러남.
moneyEditor.getAmountField().setText(String.valueOf(money.amount()));
moneyEditor.getCurrencyField().setText(money.currenceyCode());

// 어떤 컴포넌트를 소유하는지를 숨김.
moneyEditor.setAmountField(money.amount());
moneyEditor.setCurrencyField(money.currencyCode());

// 컴포넌트들과의 상호작용을 숨김.
moneyEditor.setValue(money);
```

## Context Independence

잘못된 정보 은닉은 비용. 코드를 읽기 어렵게 만듦. 객체를 조합하는 것으로 행위를 만들어 내기도 어려움. 한편, 이와 대조적으로 캡슐화는 거의 대부분 좋게 작용한다는 의견은 흥미롭다.

이를 경계하기 위한 한 가지 규칙으로 컨텍스트 독립성<sup>context independence</sup>을 이야기.

> ... each object has no built-in knowledge about the system in which it executes. This allows us to take units of behavior (objects) and apply them in new situations. To be context-independent, whatever an object needs to know about the larger environment it’s running in must be passed in.

그리고 이것이 가져오는 효과를 다시 한 번 명시함.

> Context independence guide us towards coherent objects that can be applied in different contexts, and towards systems that we can change by reconfiguring how their objects are composed.

# Achieving Object-Oriented Design

## How Writing A Test First Helps The Design

먼저, 앞선 장에서 이야기 했던 원칙들을 요약.

1. 호출자는 객체가 어떻게 동작하는지가 아닌, 무엇을 하는지와 어디에 의존하는지를 알고 싶을 뿐. 객체 경계를 잘 설정해서 이웃들과 잘 동작할 수 있게 해야 함.
2. 더 큰 환경에서 잘 사용될 수 있도록 객체는 일관적이고 논리적인 단위를 표현해야 함.

TDD가 이를 돕는 이유는 3가지.

1. 테스트를 먼저 작성한다는 것은 어떻게 이전에 무엇을 할지 생각하는 것. 이는 올바른 수준의 추상화를 달성하는 데 도움이 됨.
2. 테스트 코드가 길어지고 초점을 흐린다면, 테스트 하려는 대상이 좀 더 작은 단위로 나뉘어야 하는 것은 아닌지를 알려주는 한 가지 신호. 이는 관심사의 분리를 도움.
3. 암묵적 의존성은 테스트 코드 작성을 힘들게 함. 테스트는 또 하나의 컨텍스트. 결국 Context Independence를 도와줌. 예컨대, 생성자와 같은 명시적인 수단으로 의존성을 주입하지 않고, `ThreadContextHolder`를 사용하는 등 암묵적인 의존성을 주입해야 한다면, 테스트 코드 작성을 고통스러울 것.

개인적으로도 TDD가 큰 도움이 되었던 이유는 HOW가 아닌 WHAT을 먼저 생각하게 한다는 것(Contract of Design). 그리고 불필요하거나 너무 과한 의존성을 알려주는 중요한 설계 신호가 된다는 것.

## Communication Over Classification

TBD

