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

