
# 도메인 주도 설계 첫걸음

Learning Domain-Driven Design. 기록하면서 읽을 만한 책은 아님. 아주 일부만 기록.

## CH6. 복잡한 비즈니스 로직 다루기

- 앞서 트랜잭션 스크립트, 액티브 레코드 패턴을 다룸.
- 이들은 간단한 비즈니스 구현을 위한 도구들.
- 이번엔 복잡한 비즈니스를 다루는 도구 소개.
- 바로 도메인 모델.

### 6.1 구현

- 도메인 모델: 행동과 데이터 모두를 포함하는 도메인의 객체 모델.
- 애그리게이트, 밸류 오브젝트, 도메인 이벤트, 도메인 서비스가 그 구성 요소.
- 이들 모두 비즈니스 로직을 최우선으로 둠.

**복잡성**

- 비즈니스 로직은 이미 본질적으로 복잡.
- 따라서 여기에 우발적 복잡성이 더해지면 안 됨.
- 데이터베이스나 외부 시스템 호출 등의 인프라/기술 관심사는 피해야.

**유비쿼터스 언어**

- 기술 관심사를 배제하고 비즈니스 로직에 집중하면,
- BC(Bounded Context)의 UL(Ubiquitous Language) 용어를 따르기 쉬워짐.
- 즉, 코드가 도메인 전문가의 멘탈 모델을 따름.

### 6.2 구성요소

#### 6.2.1 밸류 오브젝트

**유비쿼터스 언어**

- 복합적인 값에 의해 식별되는 객체.
- 예컨대 색상. 빨강, 파랑 등은 RGB 수치의 조합으로 동등성 비교.
- 명시적인 식별 필드가 불필요. 오히려 문제를 일으킴.
- 여기서 잠시 [primitive obsession](https://wiki.c2.com/?PrimitiveObsession) code smell 언급.
- 바로 아래와 같은 코드.

```C#
class Person
{
    private int _id;
    private string _email;
    private string _countryCode;
    ...
}
```

- 이를 밸류 오브젝트 활용한 방식으로 변경할 수 있음.
- 이렇게 하면 명료성 향상.
- 예컨대, `countryCode`에서 `country`로 변수명이 짧아졌지만 오히려 타입과 함께 보니 직관적.
- 더불어 `Person`은 전달 받은 값에 대한 유효성 검사 책임을 덜 수 있음.
- 대신 값 객체 내부로 이런 책임이 응집되고 테스트는 더 용이해짐.
- 도메인 개념도 좀 더 잘 드러낼 수 있음.

**구현**

- `Color`를 예시로 들고 있음.
- 우선, 색상 표현에 필요한 값들을 encapsulate.
- 그리고 값 객체의 유효성 보장을 위해 생성자에서 validation 가능.
- 그러나 RGB는 0-255라서 이미 byte 타입 자체로 이를 보장.
- 값객체는 불변. 값이 바뀌면 객체는 새로 생성되어야 함. `MixWith`가 이를 보여줌.
- 왜 이렇게 만들어야 하는지 에릭 에반스 책 다시 찾아봤더니,
- 하나의 값객체가 여러 엔티티에 공유될 수도 있는데,
- 한 쪽에서의 변경이 다른 쪽에 의도치 않게 영향을 주기 때문이라 함.
- 그래서 값객체는 불변이며 변경을 원하면 객체 자체를 교환해야 함.
- 그런데 일반적으로 불변인 것이 읽기에도 디버깅에도 유리.
- 개인적으로는 객체는 기본적으로 불변이 좋은 선택이라 생각.
- 불변이 아니어야 하는 특별한 이유가 있을 때 가변. 예컨대 엔티티.

```C#
public Class Color
{
    public readonly byte Red;
    public readonly byte Green;
    public readonly byte Blue;

    public Color(byte r, byte g, byte b)
    {
        this.Red = r;
        this.Green = g;
        this.Blue = b;
    }

    public Color MixWith(Color other)
    {
        return new Color(
            r:(byte) Math.Min(this.Red + other.Red, 255),
            g:(byte) Math.Min(this.Gree + other.Gree, 255),
            b:(byte) Math.Min(this.Blue + other.Blue, 255)
        );
    }

    ...
}
```

- 마지막으로, 아래는 값객체 동등성 구현.

```C#
public Class Color
{
    ...
    public override bool Equals(object obj)
    {
        var other = obj as Color;
        return other != null &&
        this.Red == other.Red &&
        this.Green == other.Green &&
        this.Blue == other.Blue;
    }

    public static bool operator == (Color lhs, Color rhs)
    {
        if (Object.ReferenceEquals(lhs, null)) {
            return Object.ReferenceEquals(rhs, null);
        }
        return lhs.Equals(rhs);
    }

    public static bool operator != (Color lhs, Color rhs)
    {
        // 이걸 따로 구현해야 하는구나 신기하다
        return !(lhs == rhs);
    }

    public override int GetHashCode()
    {
        // ToString이 이미 Color 클래스에 오버라이딩 되어 있다는 전제인가?
        return ToString().GetHashCode();
    }
}
```

**밸류 오브젝트를 사용하는 경우**

- 가능한 모든 경우에 사용 권장.
- 표현력을 높이고, 산재할 수 있는 로직을 모아주며, 코드를 안전하게 함.
- 불변이므로 부작용과 동시성 문제가 없음.
- 일반적으로 엔티티의 속성에 밸류 오브젝트 사용.

#### 6.2.2 엔티티

- 명시적 식별 필드가 필요.
- `Person`이라는 엔티티가 있고 `Name` 필드를 가질 때,
- 이름이 바뀐다고 다른 사람이 되지는 않음.
- 한편, 이름이 같다고 같은 사람은 아님.
- 결국 별개의 식별자가 필요.
- 이는 GUID, 숫자, 문자열, 사회 보장 번호 등이 될 수 있음.
- 그리고 이 식별자는 엔티티의 생애 주기 동안 불변.
- 물론 엔티티는 변함.
- 참고로, 이 책은 엔티티를 도메인 모델의 구성 요소로 지정하지 않음.
- 애그리게이트 컨텍스트 안에서만 엔티티를 구현하기 때문이라 함.

#### 6.2.3 애그리게이트

- 애그리게이트는 엔티티.
- 명시적 식별 필드가 필요하고, 생애주기 동안 상태가 변한다는 점에서 그러함.
- 하지만 단순한 엔티티 그 이상임.
- 데이터의 일관성 보호 측면 때문.

**일관성 강화**

- 애그리게이트는 일관성을 강화하는 경계.
- 모든 들어오는 변경 요청을 검사하고,
- 비즈니스 규칙을 항상 준수하는 상태를 유지.
- 외부에서는 상태를 읽을 수만 있으며,
- 공개된 메서드를 통해서만 상태 변경이 가능.
- 이렇게 '어떤 것을 지시하는 명령'을 커맨드라 부르기도.
- 아래의 동시성 검사에 대한 이야기도.

```C#
public ExecutionResult Escalate(TicketId id, EscalationReason reason)
{
    try
    {
        var ticket = _ticketRepository.Load(id);
        var cmd = new Escalate(reason);
        ticket.Execute(cmd);
        _ticketRepository.Save(ticket);
        return ExecutionResult.Success();
    }
    catch (ConcurrencyException ex)
    {
        return ExecutionResult.Error(ex);
    }
}
```

- 애그리게이트에서는 동시성에 의한 lost update 방지를 위해,
- 아래와 같은 식의 버저닝을 구현하기도.

```C#
class Ticket
{
    TicketId _id;
    int _version;
    ...
}
```

```sql
UPDATE tickets
SET ticket_status = @new_status,
agg_version = agg_versino + 1
WHERE ticket_id=@id and agg_versino=@expected_version;
```

**트랜잭션 경계**

- 애그리게이트는 트랜잭션 경계의 역할을 함.
- 위에서 설명한 일관성을 위해 어떻게 보면 당연.
- 일부만 반영된다면 경계 내의 일관성이 깨지는 것.
- 그리고, 트랜잭션 하나 당 한 개의 애그리게이트만 두라고 함.
- 애그리게이트가 나오게 된 비즈니스 배경을 생각하면 이 규칙도 자연스러움.
- 트랜잭션이 커질수록 동시성 문제가 커지기도 함.
- 한 번에 불러들이는 데이터가 크면 느리고 메모리 소비도 큼.
- 가능한 작게 유지하는 것이 유리.
- 리포지토리도 애그리게이트에 대응된다는 얘기는 빠져서 아쉽.
- 애그리게이트에 소속된 엔티티도 전용 리포지토리가 있다면 일관성 문제.

**엔티티 계층**

- 여기서는 엔티티를 독립 패턴이 아님.
- 엔티티는 애그리게이트의 일부로서만 사용.
- 여러 객체의 변경을 원자적인 단일 트랜잭션으로 다루기 위함.
- 아래는 "에이전트가 상부 보고된 티켓의 응답 제한시간의 절반이 지나기 전에 티켓을 열람하지 않으면 자동으로 다른 에이전트가 할당된다."의 애그리게이트 구현 예시.

```C#
public class Ticket
{
    ...
    List<Message> _messages;
    ...

    public void Execute(EvaluateAutomaticActions cmd)
    {
        if (this.IsEscalated && this.RemainingTimePercentage < 0.5 &&
            GetUnreadMessagesCount(for: AssignedAgent) >0)
        {
            _agent = AssignNewAgent();
        }
    }

    public int GetUnreadMessagesCount(UserId id)
    {
        return _messages.Where(x => x.To == id && !x.WasRead).Count();
    }
}
```

**다른 애그리게이트 참조하기**

- 애그리게이트가 너무 커지면 성능과 확장 문제.
- 강력한 일관성이 필요한 것들만 경계 안에 포함시켜야.
- DDDI 책에서는 그래서 애그리게이트는 결과적으로 1개의 엔티티를 주로 갖게 된다고 했음.
- 참고로, 일관성을 가져야 한다고 하더라도, 그것이 즉시인지 아니면 결과적 일관성이어도 되는지도 중요.
- 이렇게 잘게 나누다 보면 애그리게이트가 다른 애그리게이트를 참조해야 함.
- 이 때도 역시 성능과 확장 문제를 제한해야 하므로,
- 다른 애그리게이트의 ID만을 참조하게 됨.

**애그리게이트 루트**

- 커맨드를 통해 애그리게이트의 상태를 변경하는데,
- 이 퍼블릭 인터페이스는 애그리게이트 루트로 단일화 되어야 일관성 유지 가능.
- 이 외에도 애그리게이트가 외부와 커뮤니케이션 하는 또 하나의 방법은 도메인 이벤트.

**도메인 이벤트**

- 비즈니스 도메인에서 일어나는 중요한 이벤트를 설명하는 메시지.
- 예컨대, `티켓이 할당됨`, `티켓이 상부에 보고됨`, `메시지가 수신됨` 등.
- 이미 발생한 것이므로 과거형으로 명명.
- 도메인 이벤트는 애그리게이트의 퍼블릭 인터페이스 일부.
- 자신에게 일어난 주요 일들을 알리고, 이 이벤트는 자기 자신이 받을 수도.

**유비쿼터스 언어**

- 애그리게이트 역시 UL 사용해야 함.
- 다른 개발자 및 도메인 전문가와 소통해야 하는 주요 대상.

#### 6.2.4 도메인 서비스

- 애그리게이트나 밸류 오브젝트에도 속하지 않거나,
- 복수의 애그리게이트에 관련된 비즈니스 로직을 다뤄야 할 수도.
- 이 경우 도메인 서비스를 활용.
- 이는 상태가 없는 객체.
- 보통 계산이나 분석을 수행하고자, 다양한 시스템 구성 요소를 호출하며 조율.

```C#
public class ResponseTimeFrameCalculationService
{
    ...
    public ResponseTimeframe CalculateAgentResponseDeadline(UserId agentId,
        Priority priority, bool escalated, DateTime startTime)
    {
        var policy = _departmentRepository.GetDepartmentPolicy(agentId);
        var maxProcTime = policy.GetMaxResponseTimeFor(priority);

        if (escalated) {
            maxProcTime = maxProcTime * policy.EscalationFactor;
        }

        var shifts = _departmentRepository.GetUpcomingShifts(agentId,
            startTime, startTime.Add(policy.MaxAgentResponseTime));

        return CalculateTargetTime(maxProcTime, shifts);
    }
    ...
}
```

#### 6.2.5 복잡성 관리

- 이는 제어와 동작 예측의 어려움을 평가하는데 시스템의 자유도 활용.
- 시스템의 상태를 설명하는 데 필요한 데이터 요소의 개수가 자유도.
- A, B 클래스를 소개하며 비교.
- A에는 단순한 getter/setter를 가진 변수가 5개이고,
- B에서는 5개 변수 중 3개가 나머지 2개로 추론할 수 있는 구현을 가짐.
- 그래서 A와 B의 자유도는 각각 5와 2.
- B 클래스가 코드 길이는 더 길지만 A가 사실은 더 어려움.
- 바로 이런 자유도 제한 역할을 애그리게이트와 밸류 오브젝트가 담당.


## CH10. 휴리스틱 설계

휴리스틱 정의.

- 모든 상황에 맞게 보장되고 수학적 검증된 규칙 X
- 당면한 목적에 충분할 만큼의 경험 기반 규칙 O

BC.

- 크기<sup>size</sup>로 경계 구분하는 것이, 가장 도움 안 됨.
- BC가 무효화되는 변경은 도메인이 잘 알려져 있지 않거나 비즈니스 요구사항이 빈번하게 바뀔 때.
- 변동성과 불확실성이 핵심 하위 도메인 특성이므로 이런 어려움에 유의.
- 보통 경계를 넓게 시작하는 것이 유리.
- 논리적 경계 리팩터링이 물리적인 것에 비해 비용이 적기 때문.

비즈니스 로직 구현 패턴.

- 도메인 모델과 그 변형인 이벤트 소싱 도메인 모델은,
- 복잡한 비즈니스 로직을 가진 핵심 하위 도메인에 적합.

테스트 전략.

- 트랜잭션 스크립트: 역전된 피라미드형 테스트
- 액티브 레코드: 다이아몬드형 테스트
- 도메인, 이벤트 소싱: 피라미드형 테스트

## CH11. 진화하는 설계 의사결정

- 하위 도메인의 진화에 주의 필요.
- 일반 -> 핵심 (기회)
- 핵심 -> 일반 (상품화)
- 일반 -> 지원 (통합 비용 절감)
- 지원 -> 일반 (상품화)
- 핵심 -> 지원 (단순화)
- 지원 -> 핵심 (기회)
- 일반이 핵심이 된 사례는 AWS.
- 지원이 핵심이 되는 전조 현상은 도메인이 점점 복잡해 지는 것.
- 복잡성에 비해 수익이 없다면 핵심이 지원이 되기도.
- 모든 유형의 도메인에 정교한 디자인 패턴 적용은 위험.
- 액티브 레코드, 도메인 모델 등을 적절하게 선택.
- 도메인 모델이 이벤트 소싱 도메인 모델로 변환 시,
- 과거 이력 복구가 어려울 수 있고,
- 이 때 아래와 같은 이벤트 타입을 고려하라고 안내.

```
{
    "lead-id": 12,
    "event-id": 10,
    "event-type": "migrated-from-legacy",
    "first-name": "Shauna",
    ...
}
```

## CH13. 실무에서의 도메인 주도 설계

- 그린필드 프로젝트에만 적용할 수 있다는 생각은 오해.
- 우리 엔지니어들은 대부분 커다란 진흙 덩어리 코드베이스에서 살아감.
- 브라운필드 프로젝트에 오히려 DDD 혜택이 큼.
- 그리고 모든 DDD 적용은 어려움. 필요한 도구만.
- DDD 도입 초반에 핵심 하위 도메인 식별이 필요한데,
- 이 때 아래 내용이 인상 깊음.

> 핵심 하위 도메인을 식별하는 또 하나의 강력하지만 유감스러운 휴리스틱은 최악으로 설계된 소프트웨어 컴포넌트, 즉 커다란 진흙 덩어리를 찾아내는 것이다. 이 커다란 진흙 덩어리는 모든 엔지니어가 싫어하지만 회사 측에서도 비즈니스 위험이 수반되기 때문에 처음부터 소스코드를 다시 작성하는 것을 꺼린다. 여기서 핵심은 레거시 시스템을 사용 시스템으로 대체할 수 없다는 것이며(일반 하위 도메인이 됨), 이를 수정하면 비즈니스 위험이 수반된다.

- 시스템 현대화에 big rewrite는 거의 성공하지 못함.
- 좀 더 안전한 접근 방식은 크게 생각은 하되 작게 시작하는 것.
- 물리적 경계 나누기에 앞서, 시스템의 하위 도메인 나누는 BC를 논리적으로 구분.
- 여기서의 논리적 경계는 네임스페이스, 모듈, 패키지 구분을 가리킴.
- 조직 차원에서 DDD를 하면 좋겠지만, 그렇지 못한 곳에서 엔지니어링 도구로 DDD 활용하는 방법 이야기도 언급.

## CH14. 마이크로서비스

- MSA와 DDD의 관계 탐구.
- 더불어, MSA에 DDD를 어떻게 활용하는지 소개.

### 서비스란 무엇인가?

- OASIS에 따르면, "미리 정의된 인터페이스를 사용해 하나 이상의 역량에 접근하기 위한 메커니즘".
- 여기서 인터페이스란, 서비스 데이터의 모든 입출력 메커니즘.
- 요청/응답의 동기식과 이벤트 발행의 비동기식 모두 포함.
- 이 퍼블릭 인터페이스는 서비스가 하는 일을 잘 설명함.

```
Campaign publishing service
- Publish(CampaignId): PublishingResult
- Pause(CampaignId): Confirmation
- Reschedule(CampaignId, Schedule): Confirmation
- GetStatistic(CampaignId): PublishingStatistics
- Deactivate(CampaignId, Reason): PublishingStatus
```
