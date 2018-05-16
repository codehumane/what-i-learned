# Test-Driven Development with Objects

## A Web of Objects

성장 가능하고 좋은 시스템을 만드는 것의 핵심은, 모듈이 어떤 행위를 하고 어떤 속성을 가지는지 보다, 모듈들 간의 커뮤니케이션 설계가 더 중요.

> Object-oriented design focuses more on the communication between objects than on the objects themselves.
>
> — Alan Kay

커뮤니케이션이란, 객체들이 메시지를 주고 받는 것. 다른 객체로부터 메시지를 받고, 이에 대한 반응으로 다른 객체들에 메시지를 전달. 혹은 메시지를 보낸 객체에게 값을 반환하거나 예외를 던지거나.

그래서 객체 지향 시스템은 협력하는 객체들의 망<sup>web of collaborating objects</sup>.

개인적으로는 아래 문장이 더 균형 있는 설명이라고 생각.

> A system is built by creating objects and plugging them together so that they can send messages to one another.

![web of objects](web-of-objects.png)

이와 관련된 표현들로는 pluggable, plug-compatible, building block, web of objects, declarative, collaboration, message, composite, composed, Compose objects to describe system behavior, Communication over classification, scenario, Not how but what 등이 있음.

## Values and Objects

여기서 말하는 객체란 값 객체와는 구분되는 것.

클래스라는 동일한 구조물을 사용하기 때문에 흔히 헷갈리기 쉬움. 값 객체는 변하지 않는 수치나 양을 나타냄. 객체는 식별성<sup>identity</sup>을 가지고, 시간에 따라 상태가 변하며, computational processes를 모델링. 식별성이 의미하는 것은 내가 이름이 바뀌더라도 나는 이 세계에서 나인 것. 특정 객체의 상태나 값이 변하더라도 어플리케이션에서는 동일한 대상으로 간주되는 것이 식별성. Identity vs equality. mutable vs immutable.

## Follow the Messages

> We can benefit from this high-level, declarative approach only if our objects are designed to be easily pluggable.

TBD



# Object-Oriented Style

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

- 시스템을 객체들 간의 커뮤니케이션 망<sup>web</sup>으로 봄.
- 따라서, 잘 설계된 클래스 구조보다 객체들 간의 커뮤니케이션을 더 중요하게 생각.
- 테스트 코드에서 mock의 대상은 자신의 내부가 아닌 협력(이웃) 객체들.
- 이런 협력 객체들이 강조된 테스트에서는 이 객체들이 동료여야 하는지 아니면 자신의 내부에 포함되어야 하는지를 볼 수 있게 도와줌.
- 테스트 코드에서 냄새가 난다면, 너무 많은 세부사항을 노출했거나, 객체들 간의 책임을 재조정해야 하는 신호일 수 있음.

## Value Types

먼저, 값 객체가 왜 좋은지에 대한 설명.

같은 숫자 값이라고 하더라도 feet나 metres는 의미가 다름. 값 객체는 이런 표현력을 높이는 도구. self-explanatory. 또한, 단지 숫자형을 쓰는 것에 비해 변경에도 유리. 추적을 위해 메서드를 일일이 따라가지 않아도 됨. 게다가 상태와 관계를 가지는 객체에 비해 행위가 없으며 Immutable. 이해하고 수정하기에 더 용이함.

다음으로, 값 객체를 도입하는 기법 3가지를 소개.

**Breaking out**

- 객체 코드가 여러 관심사를 다루고 있다면, 특정 관심사와 관련된 행위들을 헬퍼 타입으로 추출.
- 예컨대, 문자열을 파싱하는 것과 파싱된 결과를 해석하는 것을 구분할 수 있음.

**Budding off**

- 개인적으로 좀 더 이해가 필요한 부분
- 새로운 도메인 개념 표기를 위한 placeholder type 추가.
- 처음엔 필드가 없을 수도 있지만 코드가 점차 자라나면서 필드와 메서드들로 채워질 대상.
- 추상화를 높이는 좋은 수단.

**Bundling up**

- 항상 함께 사용되는 값들이 있다면 새로운 타입 추가의 신호일 수도.
- "Composite simpler than the sum of its parts"를 만족.

## Where Do Objects Come From?

객체를 발견하는 방법은 값객체의 그것과 유사. 단위 테스트가 더 중요하다는 점을 제외하고 말이다.

**Breaking out**

- 우리는 새로운 기능을 추가할 때 잠시 설계를 미루고, 일단 코드를 작성해 보기도.
- 어느 정도 코드를 작성하다 보면 코드가 비대해지고 이해하기 어렵다는 것을 발견하게 됨.
- 이때 응집력 있는 단위의 기능들을 새로운 타입으로 추출.

> We can start pulling out cohesive units of functionality into smaller collaborating objects, which we can then unit-test independently. Splitting out a new object also forces us to look at the dependencies of the code we're pullingo ut.

**Budding off**

- 코드가 안정적인 편이고 어느 정도 체계가 있을 땐 새로운 타입을 먼저 도입하기도.
- 이 때 새로운 타입은 단지 인터페이스.
- 이를 사용하는 객체의 테스트에서 새 인터페이스를 목킹.
- 그리고 이 협력 객체와 어떤 관계를 가져야 하는지를 작성해 나감.
- 클라이언트의 필요에 의해 새로운 타입이 무엇을 해야 하는지 결정해 가는 것.

> We think of this as “on-demand” design: we “pull” interfaces and  their implementations into existence from the needs of the client,  rather than “pushing” out the features that we think a class should  provide.

**Bundling up**

- "Composite simpler than the sum of its parts"를 적용하는 것.
- 함께 동작하는 객체 클러스터가 있을 때, 이들 협력을 하나의 객체로 패키징 할 수 있음.
- 어떤 객체들이 어떤 협력을 하는지를 추상화 할 수 있음.
- 이는 좀 더 추상화 된 수준으로 프로그래밍 할 수 있게 하는 것.

## Identify Relationships With Interfaces

저자가 인터페이스를 어떻게 사용하는지를 소개.

- 인터페이스를 많이 사용하는 편.
  - 이는 객체가 수행하는 역할을 네이밍하고, 어떤 메시지를 수신하지를 표현하는 수단.
  - 객체들 간의 관계를 강조할 수 있음.
- 한편, 인터페이스는 가능한 좁게(책에서는 narrow라고 표현) 유지함.
  - 호출자의 입장에서는 메소드가 적을수록 역할이 명확하게 드러남.
  - 어댑터나 데코레이터를 적용하기 쉬운 것은 덤.
  - "Budding Off"에서 소개했던 "pulling interface"가 좁은 인터페이스 유지에 도움이 됨.
  - 실제 필요에 기반한 인터페이스가 추출되므로.

## Refactor Interfaces Too

- 일단 인터페이스들이 만들어지면 이들 간의 유사점과 차이점에 주목하기 시작.
- 유사한 인터페이스들이 하나의 동일한 책임을 갖고 있는 건 아닌지 판단하고 병합.
- 컴포넌트들이 좀 더 "plug-compatible" 해지고, 추상화된 수준에서의 작업을 가능케 함. 설계가 "malleable"하게 된다고 표현.
- 차이점이 드러나는 인터페이스들 간에는 이름 등을 통해 구분을 명확하게 함.
- 구현을 하면서도 인터페이스가 리팩토링 됨. 가독성이 떨어지는 등 구현 코드에서 보이는 신호들에 따라서.

## Compose Objects To Describe System Behavior

저수준의 객체들을 빌딩 블럭처럼 사용해서 더 많은 일들을 하는 객체를 만듦. 이것이 바로 객체 망. 한 가지 좋은 사례로 jMock을 언급. 예컨대 아래와 같은 코드.

```java
context.checking(new Expectations() {{
    oneOf(example).doSomething(with(any(String.class)));
}});
```

아래는 이들의 관계를 도식화 한 그림.

![jMock - web of objects](jMock-web-of-objects.png)

이런 접근법의 이점은 상대적으로 적은 코드로 유연한 어플리케이션을 만들 수 있다는 것. 특히 관련 있는 여러가지 시나리오를 지원해야 하는 경우에 더 적합.

> For each scenario, we provide a different assembly of components to build, in effect, a subsystem to plug into the rest application. Such design are also easy to extend—just write a new plug-compatible component and add it in.

## Building Up To Higher-Level Programming

- 의도를 드러내는 코드를 두 가지 레이어로 조직화 하라고 함.
- 구현 계층<sup>implementation layer</sup>: describes how the code does it.
- 선언 계층<sup>declarative layer</sup>: what the code will do. 구현 계층 객체들의 조합.
- 이 두 계층은 목적이 다르므로 코딩 스타일도 다름.
- 구현 계층에서는 전통적인 객체 지향 스타일 가이드 라인을 고수.
- 선언 계층에서는 좀 더 유연함 — 정적 메소드를 사용하기도 하고, "train wreck" 식의 메소드 체이닝 호출도 허용.

