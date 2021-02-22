---
marp: true
---

# Hexagonal Architecture

![](https://reflectoring.io/assets/img/posts/spring-hexagonal/hexagonal-architecture.png)
https://reflectoring.io/spring-hexagonal/

---

# Hexagonal Architecture

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

---

# 오늘 다룰 얘기는?

- 유스 케이스(Use Case)
- 위 hexagonal 패키지 구조에서는 `application`
- DDD의 애플리케이션 서비스 (도메인 서비스 X)
- 수마트라에서는 굳이 꼽자면 `Service` 클래스들

---

# 유스 케이스에서 하는 일은?

---

# 유스 케이스에서 하는 일?

1. Take input
2. Validate business rules
3. Manipulates the model state
4. Returns output

---

# Take input

- "Validate input" X
- "Take input" O
- 입력 검증은 다른 곳의 책임
- 대신, 비즈니스 책임을 가져야 함
- 예) "Validate business rules" (뒤에서 다룰 내용)

---

# Validate input

- 유스 케이스의 역할은 아님
- 하지만, 여전히 애플리케이션 레이어의 책임
- 유스 케이스를 호출하는 어댑터(컨트롤러 등)가 할 수도 있음
- 그러나 잘못되거나 누락 문제를 겪을 수 있음
- 그렇다면?

---

# Validate input

- input model이 하면 됨
- "Send Money" 유스 케이스 호출을 위해,
- `SendMoneyCommand` 입력 모델을 넘겨준다고 하면,
- `SendMoneyCommand` 스스로 수행하는 것

---

# Validate input

```java
@Value
@EqualsAndHashCode(callSuper = false)
class SendMoneyCommand extends SelfValidating<SendMoneyCommand> {

    @NotNull private final AccountId sourceAccountId;
    @NotNull private final AccountId targetAccountId;
    @NotNull private final Money money;

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

---

# Validate input

```java
public abstract class SelfValidating<T> {

  private Validator validator;

  public SelfValidating() {
    validator = Validation
        .buildDefaultValidatorFactory()
        .getValidator();
  }

  protected void validateSelf() {
    final Set<ConstraintViolation<T>> violations = validator.validate((T) this);

    if (!violations.isEmpty()) {
      throw new ConstraintViolationException(violations);
    }
  }
}
```

---

# Validate input

- 객체 생성과 동시에 유효성 검증
- 하나라도 위배되면 객체 생성을 중단하고 예외를 던짐
- 모든 필드는 `final` (불변)
- 객체가 한 번 생성되면 상태가 유효하고,
- 생명주기 동안 뭔가 잘못된 일은 없다고 보장

---

# Validate input

- input model이 하면 됨
- 유스 케이스를 더럽히지 않으면서도,
- 애플리케이션 코어에 남아 있게 됨

---

# The Power of Constructors

- 파라미터가 점점 늘어날 수 있음
- 편의를 위해 `Builder`를 고려해 봐야 하나?
- 전체 인자를 받는 긴 생성자는 `private` 선언하고,
- `Builder#build()`에서 이를 호출하면 유효성 검사도 유지됨

---

# The Power of Constructors

```java
new SendMoneyCommandBuilder()
    .sourceAccountId(new AccountId(41L))
    .targetAccountId(new AccountId(42L))
    .money(new Money(300))
    // ... initialize many other fields
    .build();
```

---

# The Power of Constructors

- 위 상태에서 필드가 추가되면?
- 생성자와 빌더 모두에 반영해 줘야 함
- 생성자만 사용하면 추가가 누락된 문제가 컴파일 타임에 드러남
- 빌더까지 사용한 경우엔 런타임에 발견됨
- 단위 테스트로도 발견 어려울 수 있음
- IDE 포맷팅과 값 객체의 활용도 고려

---

# Different Input Models for Different Use Cases

- 같은 input model을 여러 유스 케이스에 재사용 할 수 있음
- "계좌 등록"과 "계좌 수정" 유스 케이스가 있다고 해보자
- 등록 시에는 계좌 보유자 ID, 수정 시에는 계좌 ID가 필요
- 보유자 ID나 계좌 ID에 `null`을 허용하게 됨
- 결국 유스 케이스에 유효성 검증 관심사가 섞임
- 또한 유스 케이스들 간의 결합도를 높이고 의도치 않은 영향 야기
- `AgreementChannelInquire`

---

# Validating Business Rules

- 입력 유효성 검증과 비즈니스 규칙 검증의 차이는?

---

# Validating Business Rules

| 차이 | 입력 검증 | 비즈니스 규칙 검증 |
| --- | ------- | ------------- |
| 도메인 모델 현재 상태에 접근 | X | O |
| 선언적 구현 | O (`@NotNull`) | X |
| 호칭 | 문법적 검증 | 의미적 검증 |

사례 1. "원천 계좌는 초과 인출되면 안 된다"
사례 2. "이체 금액은 0보다 커야 한다"

---

# Validating Business Rules

- 엔티티에 위임해서 처리할 수도 있고
- 적당한 엔티티가 없다면 유스 케이스에서 수행

---

# Validating Business Rules

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Account {

    public boolean withdraw(Money money, AccountId targetAccountId) {
        if (!mayWithdraw(money)) return false;
        // ...
    }
}
```

---

# Validating Business Rules

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

---

# Different Output Models for Different Use Cases

유스 케이스가 자신의 일을 완료한 뒤 호출자에게 무엇을 넘겨줘야 할까?

---

# Different Output Models for Different Use Cases

- input과 마찬가지로 output model 역시 유스 케이스에 특화된 것이 좋음
- 호출자에게 꼭 필요한 것만을 담아야 함
- "Send Money" 유스 케이스에서는 `boolean` 만을 반환

---

# Different Output Models for Different Use Cases

- 만약, 호출자가 잔고에 관심 있다면?
- `Account`를 그대로 반환하고 싶을 수도 있음
- 그러나 계좌를 조회하는 다른 유스 케이스를 만드는 것을 고려
- 의구심이 들 때는 최소한을 반환
- 불필요한 결합도를 줄이는 일
- SRP

> 공유된 모델은 시간이 지날수록 점점 커지는 여러 이유를 갖게 된다
