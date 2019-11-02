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

