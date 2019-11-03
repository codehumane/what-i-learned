# Domain-Driven Design Distilled

> 원서로 다시 읽고 다시 기록.

# Strategic Design with Bounded Contexts and the Ubiquitous Language

## Challenge and Unify

확장을 거듭하며 통제를 벗어난, 앞서 소개된 모델을 바꿔보는 시도. 가장 먼저, Ubiquitous Language를 사용. 기존 용어들을 스크럼에서 실제 사용되는 용어들로 바꿈.

- `Tenant` -> `Team`
- `User`, `Permission` -> `ProductOwner`, `TeamMember`

그리고 나서, 스크럼 프로젝트 관리와 관련성이 적은 것들을 컨텍스트에서 분리해 냄. `SupportPlans`, `Payments`, `ResourceManager`, `TimeConsumingResource` 들이 그 대상. 반대로, 누락되어 있던 개념인 `Volunteer`를 컨텍스트에 추가. 한편, `Discussions`을 핵심 모델에 추가하긴 했지만, Collaboration 컨텍스트에서  구현될 예정이며, 이 컨텍스트와의 연동을 통해 지원될 예정.

## Developing a Ubiquitous Language

Ubiquitous Language를 만들어내는 재료로 시나리오를 사용. 여기서의 시나리오란, 도메인 모델이 어떻게 동작해야 하는지에 관한 것. 예를 들면 아래와 같음. (참고로, 저자는 개발자들이 너무 명사에 주목하는 현상을 비판하고 있음)

> Allow each backlog item to be committed to a spring. The backlog item may be committed only if it is already scheduled for release. If it is already committed to a different spring, it must be uncommitted first. When the commit completes, notify interested parties.

시작점으로 괜찮으나 who를 추가하는 것이 좋음. 모델에 대한 이해가 깊어지고, 시나리오는 아래와 같이 좀 더 구체적이 될 것. 덕분에, 정족수 팀 멤버의 승인이 필요함이 발견됨. 전체 문장은 아래와 같음.

> The product owner commits a backlog item to a sprint. The backlog item may be committed only if it is already scheduled for release, and if a quorum of team members have approved commitment. If it is already committed to a different sprint, it must be uncommitted first. When the commitment completes, notify the sprint from which it was uncommitted and the sprint to which it is now committed.

### Putting Scenarios to Work

위에서 소개된 시나리오를 실행 가능한 명세로 바꾸는 것에 대해 이야기. 일종의 인수테스트. Specification by Example이나 Behavior-Driven Development 같은 것들이 한 예. 물론, 단위 테스트로 이를 만족시킬 수도 있음. 상대적으로 가독성이 떨어지므로 개발자의 도움이 더 필요할 뿐.

몇 줄만 써 보면 아래와 같음.

```
Scenario:
  The product owner commits a backlog item to a sprint
    Given a backlog item that is scheduled for release
    And the product owner of the backlog item
    And ...
    When ...
    Then ...
```

### What about the Long Haul?

유지보수 모드로 돌입한다고 해서 UL(너무 길어서 이제부터는 Ubiquitous Language를 줄여서 기록)의 지원이 필요 없거나 하는 것은 아님. 오히려, UL은 지속적으로 발전(꽤나 오랫동안). 따라서, 지속적인 노력이 필요. 그렇지 않다면 차별화가 필요한 핵심 도메인이 정말 맞는지를 반문.

## Architecture

BC(마찬가지로 Bounded Context가 너무 길어서 이제부터는 줄여서 기록) 안에는 무엇이 있나? Ports and Adapters 아키텍처 다이어그램에서 보듯, 도메인 모델 이상의 것들로 이뤄져 있음.

![ports and adpater diagram](https://kalele.io/wp-content/uploads/2018/10/ports_and_adapters.png)

- Input Adapter: REST 엔드포인트, 메시지 리스너
- Application Services: 유스 케이스 오케스트레이션, 트랜잭션 관리
- Domain Model (참고로, 이곳은 Technology-Free 구역!)
- Output Adapters: 영속성 관리나 메시지 센더

![ports-and-adapters-layer](./ports-and-adapters-layer.png)

# Strategic Design with Subdomains

- DDD 프로젝트를 하면 여러 개의 BC가 있기 마련.
- BC 중 하나는 핵심 도메인이 될 것이고,
- 나머지 BC들에는 여러 서브 도메인이 있을 것.
- 책에서는 BC와 서브 도메인이 1:1로 대응되는 것을 이상적으로 간주.

## What is a Subdomain?

먼저, IDDD 책에 나오는 아래 문장을 먼저 보는 게 더 도움이 될 것.

> the subdomains live in the problem space and the bounded contexts in the solution space

- 간단히 말하면, 서브 도메인은 사업 전체 도메인의 부분.
- 대부분의 비즈니스 도메인은 크기가 크고 복잡하기 때문에,
- 서브 도메인 하나씩 나누어 살피는 것이 이해하는 데 도움이 됨.

> Subdomains can be used to logically break up your whole business domain so that you can understand your problem space on a large, complext project.

- 한편, 서브 도메인은 전문 영역이기도 함.
- 이는 서브 도메인에 한 명 이상의 도메인 전문가가 있음을 암시함.

참고로, BC와 서브 도메인의 차이를 이해하는 데에는 아래의 글이 도움이 되었음.

http://gorodinski.com/blog/2013/04/29/sub-domains-and-bounded-contexts-in-domain-driven-design-ddd/

## Types of Subdomains

**Core Domain**

- 사업의 차별화와 경쟁력을 가져오는 도메인.
- 모든 분야에서 잘 할 수 없기 때문에, 이런 경쟁력을 가져오는 핵심 도메인이 있기 마련.
- 전략적 투자와 잘 정제된 도메인 모델이 필요한 곳이며,
- 명확한 BC 경계와 함께 UL을 잘 만들어가야 하는 부분.

**Supporting Subdomain**

- 핵심 도메인은 아니지만, 성공적인 핵심 도메인을 지원하기 위해 필요한 영역.
- 기성 솔루션이 존재하지 않으므로, 직접 만들거나,
- 핵심 도메인에 집중하기 위해, 아웃소싱을 할 수도.

**Generic Subdomain**

- 집중하고 싶은 서브도메인이 아니면서,
- 기성 솔루션이 존재하는 영역.

## Dealing with Complexity

- 소프트웨어를 구매했거나, 레거시 시스템의 경우, 경계가 그어져 있지 않은(unbounded) 경우들이 존재.
- 하지만, 내부에 논리적으로 구분된 도메인 모델들은 존재할 것.
- 이들을 서브 도메인으로 생각하면 도움이 됨.
- 예컨대, 규모 있는 시스템의 복잡성을 쪼개서 다룰 수 있게 함(서브 도메인이므로 특히 문제 영역을 말이다).
- 어느 도메인에 집중해야 하는지도 알려줌.
- 도메인 간의 관계와 의존성을 이해하는 데도 도움.
- 만약, 어떤 이유로 BC와 서브 도메인을 분리할 수 없다면,
- 모듈(DDD에서의 모듈은 Java나 Scala에서의 패키지를 가리킴)을 통해 구분하자.
