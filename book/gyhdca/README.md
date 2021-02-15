# Get Your Hands Dirty on Clean Architecture

# What's Wrong with Layers?

- 레이어드<sup>layered</sup> 아키텍처의 좋은 점에 대해 언급.
- 좋은 레이어드 아키텍처에서, 우리의 선택지는 열려 있으며, 요구사항 변화를 빠르게 수용.

> keeping our options open and are able to quickly adapt to changing requirements and external factors. And if we believe Uncle Bob, this is excatly what architecture is all about.

- 하지만, 전통적인 레이어 아키텍처에는 열린 측면이 많으며, 이것이 나쁜 습관이 파고들게 만든다고 함. 이는 시간이 지날수록 변경을 어렵게 만듦.
- 여기서의 전통적 아키텍처는 `웹 -> 도메인 -> 영속` 구조를 가리킴.

## It Promotes Database-Driven Design

- 우리는 비즈니스를 다루는 규칙이나 정책 모델을 만들고자 함.
- 주로 상태가 아닌 행위를 모델링.
- 상태가 중요하긴 하나, 행위가 상태를 바꾸며, 따라서 비즈니스를 이끌어 나가는 주체는 행위.
- 하지만, 전통적 레이어드 아키텍처에서는 DB가 토대가 됨.
- 모든 것이 영속 레이어 위에 지어지는 것.
- ORM도 종종 DB 중심 아키텍처의 주역.
- ORM 엔티티들이 영속성 레이어의 일부가 되어,
- 비즈니스 규칙과 영속성 측면을 혼합시킴.
- 서비스들이 비즈니스 모델로 영속성 모델을 사용하게 되면,
- 도메인 로직 뿐만 아니라 로딩 방식(eager vs lazy), DB 트랜잭션, 캐시 플러시 등도 다루게 됨.
- 레이어드 아키텍처가 의도한 것과 반대로, 변경이 점점 어려워지는 것.

## It's Prone to Shortcuts

- 영속성 레이어에 도메인 엔티티가 섞이고,
- 유틸리티나 헬퍼 등의 추가로 영속성 레이어가 점점 더 비대해짐.
- 레이어드 아키텍처에서는 자기 자신이나 아래의 레이어만 접근만을 허용.
- 그러다 보니 자신보다 아래 레이어에 컴포넌트들을 추가하게 되는 경향.
- 이렇게 쉽게 아래 쪽 레이어를 비대하게 만드는 것을 저자는 "shortcut mode"라 칭함.

## It Grows Hard to Test

- 레이어가 스킵되기도.
- 컨트롤러가 도메인 레이어를 거치지 않고,
- 엔티티에 직접 접근하여 필드 한 두개를 조작하는 것.
- 이는 2가지 단점을 가짐.
- 먼저, 웹 레이어에 도메인 로직을 구현하게 됨.
- 처음엔 필드 하나만 조작했겠지만, 기능이 추가될 수록 웹 레이어에는 도메인 로직이 산재하게 됨.
- 다음으로, 웹 레이어 테스트에서, 도메인 레이어 뿐만 아니라 영속성 레이어도 목으로 대체해야 함.
- 이는 복잡성의 증대.
- 테스트를 위한 준비 과정에 많은 시간 소요.
- 종종 적은 테스트 코드로 이어짐.

## It Hides the Use Cases

- 로직이 산재하면 기능을 추가할 때 어디에 해야 할지 막막.
- 게다가 레이어드 아키텍처는 "넓이"에 대한 규칙이 없음.
- 시간이 지날수록 광범위한(여러 유스 케이스들을 다루는) 서비스가 만들어짐.
- 이런 서비스들은 영속 레이어에 많은 의존성을 가지고,
- 웹 레이어의 많은 컴포넌트들이 이 레이어에 의존.
- 이런 서비스는 테스트하기 어렵고,
- 기능을 추가할 적절한 서비스 산정이 어려움.
- 하나의 유스 케이스만을 담당하는, 꽤나 전문화된 좁은 도메인 서비스가 있다면 어떨까?
- 사용자 등록이라는 유스 케이스를 찾기 위해 `UserService` 대신 `RegisterUserService`를 열어보게 된다면?

## It Makes Parallel Work Difficult

- 더 많은 사람이 함께 일할 때 더 빠를 수 있으려면,
- 병렬 작업이 가능한 아키텍처이어야 함.
- 넓은 서비스는 이를 어렵게 만듦.

# Inverting Dependencies

- 이제 대안을 다룰 차례.
- 먼저, SRP와 DIP 이야기로 시작.

## The Single Responsibility Principle

> A component should have only one reason to change.

- "책임"는 "한 번에 한 가지만 하라"가 아니라 "변화의 이유"로 해석되어야 함.
- SRP를 "Single Reason to Change Principle"로 바꾸고 싶다는 언급도.
- 하지만 한 가지 이유로 인한 변경은 여러 의존성들로 퍼져나가기 쉬움.
- 점점 더 많은 변경의 이유들을 가지게 됨.

## A Tale about Side Effects

- 과거 오래된 코드 베이스에서 일할 때,
- 사용자가 다소 이상하고 비용이 큰 요구사항을 가져왔는데,
- 좀 더 좋은 방식으로 설득해도 계속 완고하길래 좀 더 얘기를 나눠보니,
- 이전 개발팀이 변경을 반영하면 항상 부수 효과가 일어났기 때문이라고.

## The Dependency Inversion Principle

- 레이어드 아키텍처에서 레이어 간 의존성은 항상 아래쪽으로 향함.
- 상위 레이어일수록 더 많은 변경의 이유를 가짐.
- 도메인 레이어 아래 영속성 레이어가 존재한다면,
- 영속 레이어의 변경은 도메인 레이어의 변경을 수반.
- 하지만, 도메인 코드가 어플리케이션에서 가장 중요한 부분.
- 다른 이유로 도메인도 변경되게 하고 싶지 않음.
- 이 때 DIP가 도움이 됨.

> We can turn around (invert) the direction of any dependency within our codebase.

![Figure 2.2: By introducing an interface in the domain layer, we can invert the dependency so that the persistence layer depends on the domain layer](https://lh3.googleusercontent.com/-PtbCR5JCMpk/X20qOQRMciI/AAAAAAAAWEA/E-y6vzTc07su4Z4jj6XBWP6OJCbjQM0BwCLcBGAsYHQ/image.png)

- 위 그림은 도메인이 영속 레이어에 의존하던 문제를 DIP를 통해 극복한 결과.
- 영속 레이어에서 순수 엔티티를 분리해내고, 리포지토리에 인터페이스를 도입한 것.
- 참고로, 양쪽 의존성에 대한 제어권이 있어야 의존성 역전이 가능.
- 만약 써드 파티 라이브러리를 쓴다면 불가할 수도.

## Clean Architecture

> In clean architecture, in his opinion, the business rules are testable by design and independent of frameworks, databases, UI technologies, and other external applications or interfaces.

![Clean Architecture](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

- 내용 정리는 [엉클 밥의 클린 아키텍처 글](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 링크로 대체.
- 좀 특이한 건, 앞에서도 언급된 것처럼, 도메인 레이어가 영속 레이어에 의존하지 않도록, 엔티티 클래스가 도메인 레이어와 영속 레이어에 각각 별도로 존재한다는 것.
- 영속 레이어 엔티티에는 ORM 프레임워크를 위한 기본 생성자, 컬럼명 등의 메타 데이터, 로딩 타입 등이 명시됨.
- 도메인 엔티티는 이러한 영속 관련 규칙들로부터 자유로움.
- 물론, 이로 인한 비용이 뒤따름. 하지만 디커플링으로 인한 이점이 더 크다고 주장.
- fine-grained 또는 narrow 유스 케이스(서비스들) 언급도 잊지 않음.

## Hexagonal Architecture

- https://alistair.cockburn.us/hexagonal-architecture/
- 흔히 알려진 헥사고날 아키텍처 그리고 포트 앤 어댑터 이야기.

## How Does This Help Me Build Maintainable Software?

- 클린 아키텍처, 헥사고날 아키텍처, 포트 앤 어댑터 아키텍처 모두 의존성을 역전시킴.
- 이를 통해 도메인 코드가 바깥으로의 어떤 의존성도 갖지 않게 함.
- 다시 말해, 도메인 로직을 영속이나 UI 문제로부터 분리하고,
- 불필요한 변경의 이유를 제거하는 것.
- 이는 결국 더 나은 유지보수성으로 이어짐.
