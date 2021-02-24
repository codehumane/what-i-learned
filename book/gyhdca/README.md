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

# Organizing Code

## Organizing by Layer

아래 그림 방식.

```
buckpal
|-- domain
|   |-- Account
|   |-- Activity
|   |-- AccountRepository
|   |-- AccountService
|-- persistence
|   |-- AccountRepositoryImpl
ㄴ-- web
    ㄴ-- AccountController
```

- 도메인에서 `AccountRepository` 인터페이스를 바라보게 함으로써 제어 역전.
- 하지만, 3가지 이유로 이 패기지 구조는 최적이 아님.
- 먼저, 기능 또는 피처 간 패키지 경계가 없음.
    - 점점 더 많은 클래스가 같은 패키지 안에 생기며 서로 엮임.
    - 관련 없는 기능 간 부수 효과를 일으킬 수도.
- 다음으로, 애플리케이션이 어떤 유스 케이스를 제공하는지 드러나지 않음.
    - `AccountService`, `AccountController` 클래스 이름만 보고는 어떤 유스케이스 제공하는지 알기 어려움.
    - 특정 기능을 찾을 때, 일일이 서비스 안을 열어서 메서드들을 확인해 봐야 함.
    - 이건 레이어 기반 패키지 구조로 인한 문제는 아님. 저자의 의도는 뭘지.
- 마지막으로, 패키지 구조만으로는 목표 아키텍처를 알기 어려움.
    - 물론 패키지 구조로 헥사고날 아키텍처 스타일을 따르는 유추할 수 있음.
    - 하지만 어떤 기능이 웹 어댑터에서 호출되는지,
    - 어떤 기능이 영속 어댑터를 사용하는지는 알 수 없음.
    - 들어오고 나가는 포트는 코드 안에 숨겨짐.

## Organizing by Feature

```
buckpal
ㄴ-- account
    |-- Account
    |-- AccountController
    |-- AccountRepository
    |-- AccountRepositoryImpl
    ㄴ-- SendMoneyService
```

- 계좌와 관련된 코드를 모두 `account`라는 상위 수준의 패키지에 몰아 둠.
- 새로운 기능 그룹이 추가될 때도, `account`와 같은 상위 수준의 패키지가 추가됨.
- 또한 레이어 패키지는 제거함.
- 피처 간 불필요한 의존성은 package-private 접근 제한자로 보호.
- 한편, `AccountService`를 `SendMoneyService`로 리네임.
- 책임을 좁혀둔(narrow) 것.
- 이제 클래스 이름만으로 역할을 유추할 수 있음.
- 이렇게 코드에서 의도가 드러나게 하는 것을 로버트 마틴은 "screaming architecture"라 부름.
- 하지만 이 접근법을 even less visibile이라 표현. 레이어 패키지 방식에 비해서.
- 어댑터를 식별할 수 있는 어떤 패키지 명도 없고,
- 들어오고 나가는 포트도 여전히 드러나지 않음.
- 게다가 도메인 코드가 `AccountRepositoryImpl`을 의존할 수 있게 되어 제어 역전 위반 가능성 잠재.

## An Architecturally Expressive Package Structure

- 헥사고날 아키텍처에서는 엔티티, 유스 케이스, in/out 포트, in/out 어댑터가 주요 요소.
- 이에 맞는 패키지 구조는 아래와 같음.

```
buckpal
ㄴ-- account
    |-- adapter
    |    |-- in
    |    |    ㄴ-- web
    |    |         ㄴ-- AccountController
    |    |-- out
    |    |    ㄴ-- persistence
    |    |         |-- AccountPersistenceAdapter
    |    |         ㄴ-- SpringDataAccountRepository
    |-- domain
    |    |-- Account
    |    ㄴ-- Activity
    ㄴ-- application
         ㄴ-- SendMoneyService
         ㄴ-- port
             |-- in
             |    ㄴ-- SendMoneyUseCase
             ㄴ-- out
                  |-- LoadAccountPort
                  ㄴ-- UpdateAccountStatePort
```

- 위에서 언급한 주요 요소들이 각각 하나의 패키지에 대응.
- 그리고 최상위는 `account` 패키지로 계좌의 유스 케이스들을 구현하는 모듈임을 드러냄.
- `domain`패키지는 도메인 모델을 포함.
- `application` 패키지는 도메인을 둘러싼 서비스 레이어.
- `SendMoneyService`는 들어오는 포트 인터페이스인 `SendMoneyUseCase`의 구현체.
- 그리고 나가는 포트 인터페이스인 `LoadAccountPort`와 `UpdateAccountStatePort`를 사용하며, 이 둘은 영속 어댑터에서 구현.
- `adapter` 패키지는 들어오는 어댑터(애플리케이션 레이어의 들어오는 포트 호출)와 나가는 어댑터(애플리케이션 레이어의 나가는 포트를 구현)를 포함.
- 아키텍처가 코드에 그대로 녹아 있기에 커뮤니케이션과 실제 개발 시 용이함.
- 혼란스럽기보다는 오히려 도움이 됨을 주장.
- 한편, 너무 많은 패키지 경계로 인해 불필요하게 public 접근자가 설정 될 수도.
- 하지만 어댑터 패키지 같은 경우 제어 역전이 적용되어 있으니 큰 위험은 X.
- 이는 추후 데이터베이스 계층의 변경 등에도 유연.
- 또한 상위 레벨 패키지는 DDD의 바운디드 컨텍스를 대응 시키는 데도 적합.

## The Role of Dependency Injection

- 도메인에서 영속 레이어로 데이터가 흘러가지만,
- 의존은 역전 시킨 것에 대해 다시 한 번 설명.
- 그리고 이를 위해 의존성 주입이 필요함도.
- 의존성 주입이 없다면 제어의 역전은 일어나지 못함.

## How Does This Help Me Build Maintainable Software?

- 헥사고날 아키텍처를 위한 패키지 구조를 살펴봤음.
- 목표 아키텍처와 실제 코드 구조를 최대한 일치시킨 것.
- 아키텍처 요소들을 찾아가는 것은 패키지 구조를 따라가는 것으로 가능.
- 이는 커뮤니케이션과 개발과 유지보수를 도움.

# Implementing a Use Case

앞서 설명한 아키텍처를 코드로 살펴볼 차례.

## Implementing the Domain Model

- 계좌 간 돈을 보내는 유스 케이스를 구현.
- 객체지향 방식에서 이를 모델링하는 한 방식은, 입출금을 지원하는 `Account` 엔티티를 만드는 것.
- 코드는 [여기](https://github.com/thombergs/buckpal/blob/master/buckpal-application/src/main/java/io/reflectoring/buckpal/domain/Account.java) 참고.
- 이 엔티티는 계좌의 현재 스냅샷을 제공.
- 모든 입출금은 `Activity` 엔티티 안에 캡처 됨.
- 모든 activities를 매번 메모리에 로드하지 않고,
- `Account` 엔티티는 몇 일 또는 몇 주 동안의 activities만을 보유.
- 그리고 현재 잔고 계산을 위해 `baselineBalance`를 함께 가짐.
- 이는 윈도우 안의 첫 액티비티가 일어나기 직전의 잔고.
- 그리고 입출금을 위한 `withdraw`와 `deposit` 메서드를 가짐.
- 출금 시에는 초과 인출을 막기 위해 비즈니스 룰을 검사.

## A Use Case in a Nutshell

- 이제 유스 케이스를 살펴 볼 차례.
- 보통 아래 5가지 절차를 밟음.
    - Take input
    - Validates business rules
    - Manipulates the model state
    - Returns output
- 가장 먼저 incoming 어댑터로부터 입력을 받음.
- "Validate input"이 아니라 "Take input"이라 표현.
    - 유스 케이스 코드는 도메인 로직을 다뤄야지,
    - 입력 유효성 검증을 다루면 안 된다고 생각하기 때문이라고.
    - 입력 검증은 다른 어딘가에서 해야 함. (바로 뒤이어 다룸)
    - 대신, 비즈니스 룰을 검사할 책임을 가짐.
- 다음으로 비즈니스 룰 검사. 도메인 엔티티와 함께 수행.
- 비즈니스 룰이 만족되면, 입력에 기반해서 모델(보통은 도메인 객체)의 상태를 조작.
- 그리고 이 새로운 상태를 포트(영속 어댑터)로 전달. 또는 다른 outgoing 어댑터 호출.
- 호출한 어댑터로부터 반환 된 값을 출력 객체로 변환해서, 유스 케이스를 호출한 어댑터로 반환.
- 구현 모습은 아래 링크 참고.

https://github.com/thombergs/buckpal/blob/master/src/main/java/io/reflectoring/buckpal/application/service/SendMoneyService.java

## Validating Input

- 입력 값 검증이 유스 케이스의 역할이 아니라고 했지만,
- 여전히 애플리케이션 레이어에 속하는 일이라고 주장.
- 유스 케이스를 호출하는 어댑터가 입력 검증을 할 수도 있겠지만,
- 여러 어댑터가 잘못된 검증을 하거나 누락할 수도 있음.
- 따라서, input model이 이를 하도록 함.
- "Send Money" 유스 케이스에서 입력 모델은 `SendMoneyCommand` 클래스.
- 더 정확하게는 생성자 내에서 이 역할을 수행.

```java
@Value
@EqualsAndHashCode(callSuper = false)
class SendMoneyCommand extends SelfValidating<SendMoneyCommand> {

    @NotNull
    private final AccountId sourceAccountId;

    @NotNull
    private final AccountId targetAccountId;

    @NotNull
    private final Money money;

    public SendMoneyCommand(
            AccountId sourceAccountId,
            AccountId targetAccountId,
            Money money) {
        this.sourceAccountId = sourceAccountId;
        this.targetAccountId = targetAccountId;
        this.money = money;
        this.validateSelf();
    }
}
```

- 값 검증이 빠진 것 같지만 `Money` 클래스 안에 녹아 있음. [여기](https://github.com/thombergs/buckpal/blob/02256fa5439b90a665446c10621bf64feadf1a76/src/main/java/io/reflectoring/buckpal/domain/Money.java) 참고.
- 또한 `SelfValidating`을 열어 보면 아래와 같은 모습.

```java
public abstract class SelfValidating<T> {

  private Validator validator;

  public SelfValidating() {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
  }

  /**
   * Evaluates all Bean Validations on the attributes of this
   * instance.
   */
  protected void validateSelf() {
    Set<ConstraintViolation<T>> violations = validator.validate((T) this);
    if (!violations.isEmpty()) {
      throw new ConstraintViolationException(violations);
    }
  }
}
```

- 송금에 있어, 보내고 받는 각 계좌 ID가 필요하며, 금액 또한 필수.
- 따라서, 모든 값이 null이면 안 된다고 검증하고 있음.
- 또한 금액은 0보다 커야 함.
- 이 중 하나라도 위반되면 객체 생성을 중단하고 예외를 던짐.
- `SendMoneyCommand` 필드들을 final로 선언하여 불변으로 만듦.
- 따라서, 한 번 생성이 성공하면 상태가 유효하며 뭔가 잘못 변경될 일은 없다고 보장.
- `SendMoneyCommand`는 유스 케이스 API의 일부이므로, incoming 포트 패키지 안에 위치.
- 유스 케이스 코드를 더럽히지 않으면서 어플리케이션 핵심부에 남아있게 됨.
- 이런 검사를 좀 더 잘 할 수 있도록 Bean Validation API를 사용했고 `SelfValidating`를 만들어 재사용 (개인적으로는 상속 말고 객체를 인자로 받는 정적 메서드가 더 좋아 보임)

## The Power of Constructors

- 결론: 빌더의 사용은 피하고 생성자 활용하자.
- 지금은 파라미터가 3개지만 점점 늘어날 수 있음.
- 편의를 위해 빌더 패턴을 고려할 수도.
- 전체 인자를 받는 긴 생성자는 private으로 선언하고,
- 빌더의 `build` 메서드 안에서만 호출되도록 함.
- 그리고 여전히 private 생성자 안에서 유효성 검사를 진행.

```java
new SendMoneyCommandBuilder()
    .sourceAccountId(new AccountId(41L))
    .targetAccountId(new AccountId(42L))
    // ... initialize many other fields
    .build();
```

- 이 상태에서 필드가 추가된다고 해보자.
- 그러면 생성자와 빌더에 모두 반영해 주어야 함.
- 하지만 빌더에는 이 추가를 누락할 수 있음.
- 생성자만 사용한 경우와 다르게 컴파일 타임에 이 문제가 드러나지 않음.
- 단위 테스트로도 발견이 어려울 수 있음.
- IDE를 사용하면 긴 파라미터 목록도 예쁘게 단장할 수 있음.
- 생성자 사용을 적극 고려.

## Different Input Models for Different Use Cases

- 서로 다른 유스 케이스에 같은 입력 모델을 사용하고 싶을 수 있음.
- 예를 들어, "계좌 등록"과 "계좌 상세 수정" 유스 케이스가 있다고 해보자.
- 이름에서 알 수 있듯이 처음에는 거의 모든 입력이 동일할 수 있음.
- 차이점은 수정 유스 케이스는 대상 계좌의 ID가 필요하고,
- 등록 유스 케이스에는 계좌 보유자의 ID가 필요하다는 것.
- 이러한 차이 때문에 계좌 ID와 보유자 ID에 모두 null을 허용해야 함.
- 이렇게 되면 유효성 검증 로직이 유스 케이스 별로 각각 존재해야 하고,
- 이는 비즈니스 코드에 유효성 검증 관심사를 섞어버리는 일.
- 각 유스 케이스로 용도가 한정된 입력 모델은 유스 케이스를 좀 더 깨끗하게 유지시킴.
- 또한 다른 유스 케이스들과 결합도를 낮추어 의도치 않은 영향으로부터 보호.

## Validating Business Rules

먼저, 입력 유효성 검증과 비즈니스 규칙 검증의 차이 이야기.

- 입력 유효성 검증과 다르게 비즈니스 규칙 검증은 유스 케이스 역할.
- 입력 검증과 비즈니스 규칙 검증의 차이는,
- 도메인 모델의 현재 상태에 접근할 필요가 있느냐의 여부.
- 또한, 입력 검증은 `@NotNull`처럼 선언적 구현이 가능함.
- 이에 반해, 비즈니스 검증은 좀 더 많은 문맥이 필요.
- 입력 검증을 문법적 검증이라고 하고, 비즈니스 검증을 의미적 검증이라 부르기도.
- "원천 계좌는 초과 인출되면 안 된다"는 비즈니스 규칙 검증.
- 원천 계좌와 대상 계좌가 있는지, 그리고 현재 잔고가 얼마인지 등 도메인 모델의 현재 상태가 필요.
- "이체 금액은 0보다 커야 한다"는 단지 입력값만을 선언적으로 검사 가능.
- 이런 쉽고 일관된 규칙은 구현된 코드를 찾거나 추가할 위치를 찾는 데 도움이 됨.

다음으로, 비즈니스 규칙 검증을 어떻게 구현할지에 대한 이야기.

- 가장 좋은 방법은 도메인 엔티티에 맡기는 것.

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Account {

    public boolean withdraw(Money money, AccountId targetAccountId) {
		if (!mayWithdraw(money)) {
			return false;
		}

		// ...
	}
}
```

- 적당한 도메인 엔티티가 보이지 않는다면 유스 케이스에 맡길 수도.

```java
@RequiredArgsConstructor
@UseCase
@Transactional
public class SendMoneyService implements SendMoneyUseCase {

    @Override
	public boolean sendMoney(SendMoneyCommand command) {
        requireAccountExists(command.getSourceAccountId());
        requireAccountExists(command.getTargetAccountId());
        // ...
    }
}
```

## Rich versus Anemic Domain Model

- 도메인 로직을 엔티티에 많이 둘 것이냐,
- 아니면 엔티티는 단지 데이터만 가지는 얇은 "anemic"으로 두고,
- 유스 케이스에 도메인 로직을 많이 둘 것이냐 이야기.
- 여기에 어떤 제약은 없다면서 필요에 맞게 사용하라고.
- 참고로, 위에서 다룬 예제들은 "rich" 엔티티.

## Different Output Models for Different Use Cases

- 유스 케이스가 자신의 일을 완료했다면, 호출자에게 무엇을 넘겨줘야 할까?
- 입력과 마찬가지로 출력 역시 유스 케이스에 가능한 특화된 것이 좋음.
- 출력은 호출자에게 꼭 필요한 것들만을 담고 있어야 함.
- "Send Money" 유스 케이스에서는 `boolean`만을 반환.
- 만약, 호출자가 잔고에 관심 있어 한다면 어떨까?
- `Account`를 그대로 반환하고 싶을 수도 있음.
- 그러나 계좌를 조회하는 다른 유스 케이스를 만드는 것도 고려.
- 이는 다른 곳에서도 사용될 수도 있고 말이다.
- 정답은 없지만 유스 케이스를 가능한 특화시키는 것이 좋음.
- 의구심이 들 때는 최소한을 반환.
- 입력과 마찬가지로 출력 모델을 여러 유스 케이스에서 공유하게 되면,
- 유스 케이스 간의 불필요한 결합도를 낳음.
- 예컨대, 필드가 하나 추가된다면, 다른 유스 케이스에서는 관심 없더라도 알고 있어야 함.
- 공유된 모델은 시간이 지날수록 점점 커지는 여러 이유를 가짐.
- SRP.

## What about Read-Only Use Cases?

- "계좌의 잔고 확인"도 역시 구현이 필요한 하나의 유스 케이스.
- 하지만, 데이터 수정이 일어나는 유스 케이스와 구분되는, 단순한 쿼리로 바라볼 수도 있음.
- 이 경우에는 전용 "쿼리 서비스"를 만들면 됨.

```java
@RequiredArgsConstructor
class GetAccountBalanceService implements GetAccountBalanceQuery {

	private final LoadAccountPort loadAccountPort;

	@Override
	public Money getAccountBalance(AccountId accountId) {
		return loadAccountPort.loadAccount(accountId, LocalDateTime.now())
				.calculateBalance();
	}
}
```

- 단지 outgoing 포트로 쿼리를 전달하는 일만을 하고 있음.
- 이런 경우는 클라이언트가 outgoing 포트를 직접 호출하게 할 수도.
- 이는 뒤의 "Taking Shortcuts Consciously"에서 다룬다고 함.

# Implementing a Web Adapter

## Dependency Inversion

- 웹 어댑터 패키지의 컨트롤러가 유스 케이스를 직접 호출하지 않고,
- 애플리케이션 포트 in 패키지의 인터페이스를 호출. DIP.
- 왜 이렇게 한 번 거쳐가는 게 좋을까?
- 외부 세상과의 어떤 커뮤니케이션이 있는지 명확히 알 수 있음.
- 이는 레거시 코드베이스의 유지보수 주체에게는 의미 있는 정보.
- 뒤의 "Taking Shortcuts Consciously"에서도 다루겠지만, 유스 케이스를 직접 호출하기도.
- 하지만, 웹 소켓 처럼, 애플리케이션 코어에서 웹 어댑터로 데이터를 보내야 하는 경우도 존재.
- 이 경우에는 분명하게 포트가 필요.

## Respnsibilities of a Web Adapter

- Maps HTTP requests to Java objects
- Performs authorization checks
- Validates input
- Maps input to the input model of the use case
- Calls the use case
- Maps the output of the use case back to HTTP
- Returns an HTTP response
- 여기서의 입력 검증은 유스 케이스에서의 입력 검증, 그리고 비즈니스 규칙 검증과는 다름.
- 웹 어댑터의 입력 모델을 유스 케이스의 입력 모델로 변환할 수 있는지를 검증.

## Slicing Controllers

- 컨트롤러를 얼마나 잘게 나눌지에 대한 이야기.
- 저자는 가능한 적게 만들기 보다는 가능한 많이 나누라고 함.
- 가능한 적은 책임을 가진 좁은<sup>narrow</sup> 웹 어댑터를 만들라고.
- 일반적으로는 계좌에 관련된 모든 연산들을 담은 `AccountController`를 만들 것.
- 아래와 같은 식.

```java
@RestController
@RequiredArgsConstructor
class AccountController {
    @GetMapping("/accounts")
    List<AccountResource> listAccounts() {}

    @GetMapping("/accounts/id")
    AccountResource getAccount(@PathVariable("accountId") Long accountId)  {}

    @GetMapping("/accounts/{id}/balance")
    long getAccountBalance(@PathVariable("accountId") Long accountId) {}

    @PostMapping("/accounts") AccountResource createAccount(@RequestBody AccountResource account){}

    @PostMapping("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}") void sendMoney( @PathVariable("sourceAccountId") Long sourceAccountId, @PathVariable("targetAccountId") Long targetAccountId, @PathVariable("amount") Long amount) {}
}
```

- 하나의 클래스에서 계좌와 관련된 것만 다루고 있어 괜찮아 보임.
- 그러나 몇 가지 단점을 가짐.
- 커지면 커질수록 이해하기 힘듦.
- 테스트하기도 어려움. 테스트 코드는 프로덕션 코드보다 일반적으로 양도 더 많고 이해하기도 어려움. 
- 데이터 구조가 의도치 않게 재사용 될 가능성도 내포.
- 같은 클래스에서는 private 접근자도 여러 연산이 같이 사용 가능하니.
- 추가로, 여러 사람이 병렬 작업을 하더라도 충돌이 적음.
- 대신, 아래와 같은 구조를 언급.

```java
@WebAdapter
@RestController
@RequiredArgsConstructor
class SendMoneyController {

	private final SendMoneyUseCase sendMoneyUseCase;

	@PostMapping(path = "/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
	void sendMoney(
			@PathVariable("sourceAccountId") Long sourceAccountId,
			@PathVariable("targetAccountId") Long targetAccountId,
			@PathVariable("amount") Long amount) {

		SendMoneyCommand command = new SendMoneyCommand(
				new AccountId(sourceAccountId),
				new AccountId(targetAccountId),
				Money.of(amount));

		sendMoneyUseCase.sendMoney(command);
	}

}
```

- 서비스와 컨트롤러의 이름에 대해서도 깊게 생각해야 함.
- `CreateAccount` 보다 `RegisterAccount`가 더 나은 이름.
- 이 애플리케이션에서는 계좌를 생성하는 유일한 방법은 사용자가 이를 등록하는 것.
- 따라서, 이 이름이 더 의미를 잘 드러냄. (왜지? 예시는 와 닿지 않음)

# Implementing a Persistence Adapter

## Dependency Inversion

- 도메인 코드가 영속 관련 문제를 신경쓰지 않고도 변화할 수 있도록,
- 애플리케이션 서비스는 포트 인터페이스로 영속 기능을 호출.
- 그리고 이 인터페이스는 영속 어댑터 클래스가 구현.
- 이 어댑터를 헥사고날 아키텍처에서는 "driven" 또는 "outgoing"이라 부름.

## The Responsibilities of a Persistence Adapter

- Takes input
- Maps input into a database format
- Sends input to the database
- Maps database output into an application format
- Returns output
- 입력 모델은 도메인 엔티티가 될 수도 있고, 특정 작업에 특화된 객체일 수도.
- 그리고 이를 JPA 엔티티로 변환.
- 매번 매핑하는 것은 얻는 것에 비해 큰 작업일 수 있으므로,
- 뒤의 "Mapping between Boundaries"에서 매핑 없는 전략을 다룸.
- 한편, 꼭 ORM을 사용해야 하는 것은 아님.
- SQL로 변환해서 실행시킬 수도 있고,
- 파일에 데이터를 저장할 수도 있음.
- 핵심은 영속 어댑터가 핵심부에 영향 주지 않는 구조를 유지하는 것.

## Slicing Port Interfaces

- 포트 인터페이스는 어떻게 나누어야 할까.
- 너무 큰 포트 인터페이스는 불필요한 의존성을 가져옴.
- 불필요한 의존성은 코드를 이해하고 테스트하기 어렵게 만듦.
- 예를 들어, `RegisterAccountService` 단위 테스트를 만든다고 해보자.
- 이는 `AccountRepository` 인터페이스에 의존하는데, 수 많은 메서드 중 어떤 것을 mocking 해야 할까?
- 일단 찾아내는 데도 시간이 걸리는 게 문제.
- 그리고 일단 찾아서 mock으로 감쌌다고 하더라도,
- 다음 사람은 어느 정도까지 mocking 되어 있는지 파악해야 하고,
- 단위테스트가 동작하고 있기에 정상적으로 간주하고 mocking 안 된 부분을 사용할 수도 있음.
- ISP(Interface Segregation Principle)은 이 문제에 답을 제공.
- 인터페이스는 클라이언트가 오직 필요한 메서드만을 알 수 있도록 충분히 구체적이어야 한다고.

> Depending on something that carries baggage that you don't need can cause you troubles that you didn't expect."

## Slicing Persistence Adapters

- 하나의 단일 클래스로 모든 포트 인터페이스를 구현할 수도.
- 도메인 클래스, 그 중에서도 애그리거트를 기준으로 나눌 수도.
- 좀 더 나아가서 하나의 애그리거트지만, JPA 등의 ORM을 사용했냐 아니면 성능을 위해 평문 SQL을 사용했느냐로 나눌수도.
- "one persistence adapter per aggregate" 접근법을 시작점으로 잡는 것은 괜찮아 보임.
- 향후의 바운디드 컨텍스트로 세분화를 돕는 측면.
- 미리 나누어 두면 서로 의존할 가능성도 낮으니 분리가 좀 더 쉽기에.

## Example with Spring Data JPA

[Account](https://github.com/thombergs/buckpal/blob/f5a9be50771e77ca66a153bc83c383b32cab738e/src/main/java/io/reflectoring/buckpal/account/domain/Account.java)

- `Account`는 게터 세터를 갖는 단순한 데이터 클래스가 아니라,
- 가능한 불변이려는 노력을 하고 있고,
- 팩토리 메서드로만 유효한 상태의 엔티티를 생성할 수 있고,
- 상태를 조작하는 메서드는 잔고 확인 등의 유효성 검사를 항상 수행.
- 유효하지 않은 도메인 모델이 없도록 유지하는 것.
- `Account`를 JPA `@Entity`로 선언하는 것이 좋은지에 대해서는 뒤이은 "Mapping between Boundaries"에서 다룰 예정.
- 일단 분명한 건, JPA 엔티티가 되는 순간 도메인 모델에 제약이 가해진다는 것.
- 예컨대, JPA는 인자가 없는 기본 생성자를 가지는 것을 필요로 함.
- 또한 성능 관점에서 `ManyToOne`이 이치에 맞지만, 현재 도메인 모델의 요건에서는 관계가 반대로 되길 원함.
- 이러한 불일치나 타협으로부터 자유롭고자 한다면 도메인 모델과 영속 모델 간의 매핑은 필요할 것.

[AccountJpaEntity](https://github.com/thombergs/buckpal/blob/f5a9be50771e77ca66a153bc83c383b32cab738e/src/main/java/io/reflectoring/buckpal/account/adapter/out/persistence/AccountJpaEntity.java), [ActivityJpaEntity](https://github.com/thombergs/buckpal/blob/master/src/main/java/io/reflectoring/buckpal/account/adapter/out/persistence/ActivityJpaEntity.java)

- `Account`를 DB에 영속시키기 위해 영속 계층에서 만들어지는 JPA 엔티티들.
- 계좌 JPA 엔티티에는 단지 ID만이 있음. 추후 사용자 ID 정도가 추가될 수 있음.
- 대신, `ActivityJpaEntity`에 모든 계좌 활동이 담김.
- 이 둘은 서로 `@OneToMany` 등의 관계를 갖지 않음.
- 부하를 주는 쿼리가 발생할 수 있기 때문.

[UpdateAccountStatePort](https://github.com/thombergs/buckpal/blob/f5a9be50771e77ca66a153bc83c383b32cab738e/src/main/java/io/reflectoring/buckpal/account/application/port/out/UpdateAccountStatePort.java), [LoadAccountPort](https://github.com/thombergs/buckpal/blob/f5a9be50771e77ca66a153bc83c383b32cab738e/src/main/java/io/reflectoring/buckpal/account/application/port/out/LoadAccountPort.java), [AccountPersistenceAdapter](https://github.com/thombergs/buckpal/blob/master/src/main/java/io/reflectoring/buckpal/account/adapter/out/persistence/AccountPersistenceAdapter.java)

- 주어진 날짜 이전으로는 입출금 sum으로 잔고를 계산하고,
- 주어진 날짜 이후로는 `Activity` 목록으로 관리.
- `updateActivities`에서는 신규 활동만 저장해야 하므로 ID가 빈 것들만 저장.

## What abount Database Transactions?

- 영속 어댑터 입장에서는 어떤 다른 데이터들이 같은 유스 케이스에서 사용되는지 모름.
- 이 역할은 서비스가 오케스트레이션 할 수밖에 없음.
- 가장 쉬운 방법은 애플리케이션 서비스에 `@Transactional` 선언.
- 서비스가 좀 더 순수한 상태로 유지되길 원한다면 AOP를 사용할 수도.

# Testing Architecture Elements

특별한 내용은 없어서 기록 생략.

# Mapping Between Boundaries

- 레이어 모델 간의 매핑에 대한 이야기.
- 레이어 간에 동일한 모델을 사용하면 결합도가 올라가는 측면과,
- 레이어 간 매번 매핑을 하는 건 보일러플레이트 코드가 많이 늘어나는 측면을 함께 고려.

## The "No Mapping" Strategy

- 먼저, 매핑을 안 하는 전략부터 다룸.
- 예컨대, `AccountController`가 `SendMoneyUseCase`에게,
- `SendMoneyCommand` 대신 도메인 엔티티인 `Account`를 보내는 것.
- 영속 어댑터에도 `Account`를 보내고 있으니, 모든 레이어에서 `Account`를 재사용.
- 하지만 이 모델이 REST 응답으로도 사용되면 JSON으로 어떻게 시리얼라이즈 되는지에 대한 애노테이션도 담게 될 것.
- 또한 영속 레이어에서도 사용되니 데이터베이스 매핑을 위한 애노테이션들도 함께 담김.
- 이는 SRP 위반. `Account`가 서로 다른 이유(레이어의 서로 다른 니즈에 따라)로 바뀌기 때문.
- 이에 더해 각 레이어에서만 필요한 필드가 점점 추가될 수도. 도메인 모델의 파편화.
- 그럼 "no mapping" 전략을 무조건 가져가면 안 되는 걸까?
- 각 레이어에서 사용하는 필드가 모두 동일한, 단순한 CURD 유스 케이스에서는 매핑이 오히려 비용.
- 그리고 애노테이션의 변경에 대해서는 저자도(개인적으로도) 관대한 편.
- 개인적 경험으로는, 단순한 정보성 데이터나, 보수적인 성격을 띄어 잘 변하지 않는 곳에, "no mapping"이 주는 이점이 더 큼.

## The "Two-Way" Mapping Strategy

- 각 레이어가 각자의 모델을 가진 것을 저자는 "two-way" 매핑 전략이라 부름.
- 웹 레이어에서 도메인 모델로, 영속 레이어가 도메인 모델을 영속 모델로, 두 방향으로 변환하기 때문.
- 이 방식은 위에서 말한 것 처럼 한 모델에 여러 관심사가 섞이지 않게 됨.
- 각 레이어는 자신에게 최적화 된 모델을 사용.
- 한편, 이 방식이 "no mapping" 다음으로 단순한 방식.
- 하지만 보일러플레이트 코드가 늘어남. 자동으로 매핑해 주는 도구는 또 다른 단점을 가져옴.
- 또 다른 단점은, 도메인 모델이 레이어 간 경계에 사용된다는 것. 단지 레이어 간 호출의 필요에 의해 도메인 모델이 영향 받을 수 있음.

## The "Full" Mapping Strategy

- 각 연산마다 구분된 입출력 모델을 사용.
- 예제로 제공된 [SendMoneyService](https://github.com/thombergs/buckpal/blob/master/src/main/java/io/reflectoring/buckpal/account/application/service/SendMoneyService.java) 참고.
- 다만, 여기서 더 나아가 영속 어댑터 호출 시에도 별도의 입력 모델을 만드는 게 차이점.
- 저자는 영속 어댑터까지 이런 매핑을 가져가지는 않는다고 함.

## The "One-Way" Mapping Strategy

- 이 방식은 레이어간 동일한 인터페이스를 입력 모델로 사용하는 것.
- 하지만 구현체는 다름.
- 그림으로 보면 쉬운데 여기에 담을 수 없어 아쉬움.
- 개인적으로는 유스 케이스가 하나의 도메인 엔티티를 다루는 것이 아니기에, 별로 좋은 방식은 아니라고 생각됨.
- 그래서 저자도 read-only 연산 같이, 레이어 간 모델이 매우 유사한 경우에 사용하라고 제안하는 듯.

## When to Use Which Mapping Strategy?

- 결국 "It depends."
- 개인적으로는 성숙한 조직이라면 "no mapping"으로 시작하고,
    - 장단점을 알고 선택하는 것은 그렇지 않은 것과 다름.
    - 알고 선택한 경우는 변경해야 하는 시점도 인지할 수 있음.
    - 그리고 이 시점에 과감히 변경할 역량도 있고.
- 성숙도가 그리 높지 않다면 간소화 된 "full mapping"으로 시작하는 것이 좋다고 생각함.
    - 개인적 경험상 이 방식이 확률적으로 가장 편익을 많이 내는 방식이라 생각하기 때문.
- 저자는 아래와 같이 가이드.
- 수정 유스 케이스
    - 웹 -> 애플리케이션: "full mapping"
    - 애플리케이션 -> 영속: "no mapping"으로 시작 후 관심사가 멀어지기 시작하면 "two-way"
- 쿼리 유스 케이스
    - 모두: "no mapping"으로 시작 후 관심사가 멀어지기 시작하면 "two-way"
