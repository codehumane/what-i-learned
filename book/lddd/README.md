
# 도메인 주도 설계 첫걸음

Learning Domain-Driven Design. 기록하면서 읽을 만한 책은 아님. 아주 일부만 기록.

## CH6. 복잡한 비즈니스 로직 다루기

- 앞서 트랜잭션 스크립트, 액티브 레코드 패턴을 다룸.
- 이들은 간단한 비즈니스 구현을 위한 도구들.
- 이번엔 복잡한 비즈니스를 다루는 도구 소개.
- 바로 도메인 모델.

### 6.1 구현

- 도메인 모델: 행동과 데이터 모두를 포함하는 도메인의 객체 모델.
- 애그리거트, 밸류 오브젝트, 도메인 이벤트, 도메인 서비스가 그 구성 요소.
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
