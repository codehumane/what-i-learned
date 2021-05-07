# 레거시 코드 활용 전략

# 1부. 코드 변경의 메커니즘

## 소프트웨어 변경

### 소프트웨어 코드를 변경하는 네 가지 이유

- 이런 저런 내용이 나오지만, 결국 어떤 동작을 변경할 때 다른 동작들에는 영향이 없어야 한다는 이야기.
- 그리고 안전한 변경을 위해 영향 범위를 정확히 이해하는 것이 중요.

### 위험한 변경

> 클래스와 메서드를 새로 생성하지 않으면, 기존의 클래스와 메서드는 갈수록 비대해져서 결국 코드를 이해하기 어려운 지경에 이른다. 대규모 시스템에서 변경 작업을 할 때는 작업 대상을 이해하기 위해 어느 정도 시간이 걸릴 것을 감안해야 한다. 이때 좋은 시스템과 나쁜 시스템의 차이는 좋은 시스템일 경우 시스템에 익숙해질수록 마음이 편안해지고 변경 작업에 확신을 갖게 되지만, 나쁜 시스템의 경우 시스템을 알게 될수록 코드 변경 작업이 마치 호랑이를 피하기 위해 절벽에서 뛰어내리는 것 같은 느낌을 받게 된다는 점이다.

## 피드백 활용

### 시스템 변경의 방식

- 시스템 변경의 방식을 크게 2가지로 나눔.
- 편집 후 기도하기(Edit and Pray)
- 보호 후 수정하기(Cover and Modify)
- 편집 후 기도하기가 일반적.
- 이 방식에서 매우 신중하게 계획을 세우고, 대상 코드를 충분히 이해하는 등의 노력이 포함됨.
- 그러나 아무리 신중하게 주의를 기울여도 그에 비례해서 안정성이 높아진다는 보장은 없음.
- 이를 극복하기 위해 먼저 보호하는 것이 필요.

### 보호망

- 이 보호망은 다름 아닌 테스트.
- 작업 결과의 정확성을 보여주기 위함이기도 하며,
- 변경된 부분을 발견하기 위함이기도 함.
- 변경하고자 하는 부분만을, 의도대로 변경했는지 확인하는 것.

### 회귀 테스트

- 보통 회귀 테스트가 이를 위한 도구.
- 그러나 회귀 테스트는 애플리케이션 수준의 테스트.
- 피드백 주기가 길고, 빠른 문제 디버깅이 어려움.
- 커버리지도 높지 않은 것이 일반적.
- 그럼에도 불구하고 이런 상위 수준의 테스트는 다수 클래스들의 동작을 한 번에 확인할 수 있는 이점이 있음.

### 단위 테스트

- 단위테스트는 이 단점들을 보완.
- 실행 속도가 빠름.
- 좀 더 빠른 오류 위치 파악에 도움이 됨.

### 레거시 코드를 변경하는 순서

1. 변경 지점을 식별한다.
2. 테스트 루틴을 작성할 위치를 찾는다.
3. 의존 관계를 제거한다.
4. 테스트 루틴을 작성한다.
5. 변경 및 리팩토링을 수행한다.

### 테스트 작성 전 의존 관계의 제거

- 변경에 앞서 테스트를 작성해야 하는데,
- 깊고 거대한 의존성을 테스트 작성을 어렵게 함.
- 그래서 이 의존성을 제거하기 위한 리팩토링을 먼저 진행.
- 기본 타입 매개변수(primitive parameter)와 인터페이스 추출(extract interface)라 부르는 방법 사용.
- 이 같은 리팩토링을 테스트 없이 수행해도 충분히 안전하다고 저자가 말하는 게 인상적.

## 감지와 분리

- 테스트 시 (mock이 아닌) fake 클래스 만들어 감지와 분리의 과정을 보여줌.
- 테스트 하려는 클래스는 보통 다른 클래스들에 의존하기 마련.
- 이 의존 클래스가 테스트 환경에서 실행이 어려울 수 있음.
- 이 때, 이 의존 클래스를 가짜로 대체해서, 실제 실행 없이도 약속된 값을 반환하도록 하는 것이 '감지'.
- 한편, 이 의존 클래스는 또 다른 클래스, 그리고 그 클래스는 또 다른 클래스를 의존하고, 이런 식으로 확장되다 보면, 테스트 코드에 거의 모든 클래스를 정의해야 할 수도 있음.
- 이런 의존 관계를 끊어서 원하는 테스트에 집중할 수 있게 하는 것이 '분리'.
- 설명이 어려울 뿐, 우리가 흔히 하는 일들에 지나지 않음.
- 그리고 아래 내용이 인상적.

> 어떤 사람들은 가짜 객체를 사용하는 테스트를 가리켜서 "진짜 테스트가 아니다."라고 말하곤 한다. 어쨌든 실제 화면에 정말로 표시되는 내용을 보여주는 것은 아니기 때문이다. 따라서 금전 등록기 디스플레이 소프트웨어의 일부가 잘못 동작해도 우리는 이 테스트를 통해 소프트웨어의 오동작을 알아낼 수 없다. 물론 이것은 사실이지만, 그렇다고 해서 이 테스트가 실제 테스트가 아니라는 뜻은 아니다. 설령 실제 금전 등록기 화면과 정확히 동일한 픽셀로 구성되는 테스트를 설계할 수 있다고 한들, 그것이 모든 하드웨어에서의 동작을 의미할까? 그렇지는 않다. 하지만 이것 역시 실제 테스트가 아니라는 의미는 아니다. 우리는 테스트 루틴을 작성할 때 분할 후 정복(divide and conquer) 접근법을 취해야 한다. 이 테스트의 목적은 Sale 객체가 디스플레이에 어떤 영향을 미치는지 알려주는 것에 그치지만, 그렇다고 중요하지 않은 테스트라고 말할 수 없다. 적어도 버그의 원인이 Sale 클래스에 있는지 여부를 확인할 수는 있기 때문이다. 이러한 테스트를 통해 오류의 원인이 발생한 위치를 좁혀 나가고 디버깅 시간을 크게 절약할 수 있다.

## 봉합 모델

- 봉합이라는 단어가 참 어색.
- 테스트를 위해 감지와 분리를 해야 하는데,
- 이를 위해 설정 등으로 의존 대상을 갈아끼울 수 있어야 함을 의미.
- 책에서는 아래와 같이 정의.

> 봉합은 코드를 편집하지 않고도 동작을 변화시킬 수 있는 위치를 말한다.

# 2부. 소프트웨어 변경

## 고칠 것은 많고 시간은 없고

- 코드 변경을 위해 의존 관계를 제거하고 테스트를 작성하는 것은 분명 시간이 드는 일.
- 하지만 '결국은' 개발 시간과 시행착오를 줄여준다고.
- 그런데 여기서 결국은 언제일까?
- 최악의 경우에는 몇 년 후가 될 수도.
- 그러나 일반적으로는 코드가 변경되는 위치는 시스템의 특정 부분에 집중됨.
- 오늘 바뀐 코드의 주변은 가까운 시일 내에 변경될 가능성이 높음.

### 발아 메서드

- 원서에서는 Sprout Method라고 표기.
- 먼저 아래 코드가 있음.

```java
public class TransactionGate {
  public void postEntries(List entries) {
    for (Iterator it = entries.iterator(); it.hasNext(); ) {
      Entry entry = (Entry) it.next();
      entry.postDate();
    }
    transactionBundle.getListManager().add(entries);
  }
}
```

- 이제, 신규 항목인 경우에만 transactionBundle에 추가해야 한다면?

```java
public class TransactionGate {
  public void postEntries(List entries) {
    List entriesToAdd = new LinkedList();
    for (Iterator it = entries.iterator(); it.hasNext(); ) {
      Entry entry = (Entry) it.next();
      if (!transactionBundle.getListManager().hasEntry(entry)) {
        entry.postDate();
        entriesToAdd.add(entry);
      }
    }
    transactionBundle.getListManager().add(entriesToAdd);
  }
}
```

- 이렇게 변경하는 것은 무리가 있음.
- 제대로 변경됐는지를 확인하기가 어렵게 때문.
- 또한, 날짜 설정과 중복 검사라는 2개의 동작이 섞임.
- 그리고 임시 변수가 추가되었는데(항상 나쁜 것은 아님), 이후에 추가 될 동작도 이 임시 변수에 대해 이뤄질 가능성이 높고, 따라서 코드가 점점 더 복잡해 질 수 있음.
- 이 대신, sprout method 방식에서는 아래와 같이 함.

```java
public class TransactionGate {
  public void postEntries(List entries) {
    
    // 여기가 sprout method
    List entriesToAdd = uniqueEntries(entries);

    for (Iterator it = entries.iterator(); it.hasNext(); ) {                
      entry.postDate();
      entriesToAdd.add(entry); 
    }

    transactionBundle.getListManager().add(entriesToAdd);
  }
}
```

- 하지만 기존의 `TransactionGate`가 테스트하기에는 너무 의존성이 심한 클래스일 수 있음.
- 이 때는 새로운 발아 메서드를 public static으로 선언하기도 한다고 함.
- 이렇게 static 메서드가 몇 개가 추가되고, 만약 공유되는 변수들이 발견되면 새로운 클래스로 추출.

### 발아 클래스

- 기존 클래스가 의존성이 커서 테스트하기 어렵다면 생각해 볼 수 있는 방식.
- 새로운 메서드와 함께 클래스도 새로 생성하는 것.
- 하지만, 작은 변경에도 클래스를 무작정 새로 생성되는 것은 문제가 될 수도.
- 충분히 의미가 있는 클래스인지 고민해 보고, 또는 어쩔 수 없는 경우에 사용.

### 포장 메서드

- wrap method
- 먼저 아래와 같은 코드가 있음.

```java
public class Employee {
  ...
  public void pay() {
    Money amount = new Money();
    for (Iterator it = timecards.iterator(); it.hasNext(); ) {
      Timecard card = (Timecard) it.next();
      if (payPeriod.contains(date)) {
        amount.add(card.getHours() * payRate);
      }
    }
    payDispatcher.pay(this, date, amount);
  }
  ...
}
```

- 이제, 금액 지불 시, 직원 이름의 파일도 갱신해야 한다고 해보자.
- 아래와 같이 기존 `pay` 메서드를 `dispatchPayment`로 바꾸고,
- `pay` 메서드에서 `dispatchPayment`와 더불어 새 요구사항을 수행하는 `logPayment`를 함께 호출.

```java
public class Employee {
  
  // ...
  public void pay() {
    logPayment();
    dispatchPayment();
  }
  
  public void dispatchPayment() {
    Money amount = new Money();
    for (Iterator it = timecards.iterator(); it.hasNext(); ) {
      Timecard card = (Timecard) it.next();
      if (payPeriod.contains(date)) {
        amount.add(card.getHours() * payRate);
      }
    }
    payDispatcher.pay(this, date, amount);
  }

  private void logPayment() { 
    ...
  }
}
```

- 한편 아래와 같이 할 수도.

```java
public class Employee {
  ...
  public void makeLoggedPayment() {
    logPayment();
    pay();
  }
  
  public void pay() {
    ...
  }

  private void logPayment() { 
    ...
  }
}
```

- 신규 기능을 추가할 수 있는 좋은 방법.
- 발아 메서드/클래스는 기존 메서드가 변경되어야 하지만,
- 포장 방식에서는 기존 것을 바꾸지 않아도 되기 때문.
- 하지만, 포장 메서드에 다소 부적절한 이름을 짓기가 쉬움.

### 포장 클래스

- 데코레이터 생각하면 됨.
- 포장 클래스는 엄격한 조건하에서만 고려하라고 주장.
- 첫 번째로, 추가하려는 동작이 완전히 독립적이며, 기존 클래스가 상관 없는 동작으로 오염되게 하고 싶지 않을 때.
- 두 번째로, 기존 클래스가 너무 비대한 경우.

## 어떻게 기능을 추가할까?

- 많은 레거시 코드가 테스트 코드를 갖지 않음.
- 심지어 테스트 코드 추가가 어려울 수도.
- 이 경우 앞서 소개한 발아/포장 기법을 적용할 수 있으나,
- 이는 기존 코드를 많이 수정하지 않기에 당장은 코드 개선 효과를 보기 어려움.
- 또한, 새로 추가되는 코드가 기존 코드와 중복된 부분이 있으면 잠재적 위험.
- 그리고 큰 개선이 없을 거라는 두려움과 이것이 반복되어 생기는 체념도 이야기.

### 테스트 주도 개발

- 예시와 함께 TDD 소개.
- 이렇게 하면, 위 문제들을 극복할 수 있음.
- 레거시 코드에 TDD를 하고자 한다면, 기존 코드 주위에 테스트 루틴을 작성하는 것이 중요.

### 차이에 의한 프로그래밍

- 클래스를 직접 수정하지 않고 기능을 추가하는 또 다른 방법.
- 바로 상속 이야기.
- 물론 상속의 남용으로 인한 문제들은 경계.
- LSP 얘기를 하며 상속이 주는 문제점을 예시로 보여주기도.
- 하지만 일단 테스트를 빠르게 통과시키고 목적지로 가기 위한 임시처로는 괜찮다고.
- 이것을 `MessageForwarder` 예시로 보여줌.
- 기능 추가를 하며 `MailingConfiguration`이 추가되고,
- 이것이 다시 `MailingList`로 변경되는 전체적인 과정이 재미있음.

## 뚝딱! 테스트 하네스에 클래스 제대로 넣기

- 기존 코드에 테스트를 추가할 때 몇 가지 문제들이 있음.
- 클래스의 객체 생성이 쉽지 않음.
- 클래스를 포함한 테스트를 쉽게 빌드하기 어려움.
- 반드시 사용해야 하는 생성자가 부작용을 일으킴.
- 생성자 내부에서 상당량의 처리가 일어나며 이 내용을 알아야 함.

### 성가신 매개변수

- 테스트 할 클래스의 객체를 만들 때,
- 생성자가 여러 매개변수들을 요구할 수 있음.
- 예를 들어, DB 연결이 완료된 커넥션 객체 같은 것.
- 하지만 테스트에서는 DB 연결이 불필요할 수 있고, 이 객체를 만들기가 번거롭거나 위험하기도.
- 이 때는 이 번거로운 의존 클래스로부터 인터페이스를 추출하고 페이크 클래스 등을 선언하여 테스트를 돕게 할 수도.
- 상황에 따라서는 그냥 `null`을 넘기기도. (다만 테스트 환경에 한정)

### 숨겨진 의존 관계

- 생성자 내에서 다른 객체를 생성하는 경우가 있음.
- 그런데 이 객체 생성이 테스트 환경에서는 하면 안 되거나 실행이 불가능하기도.
- 그래서 이 객체를 생성자를 통해 주입 받도록 변경.
- 필요하면 기존 생성자도 함께 유지하여 클라이언트 코드에 영향을 줄일 수도.

### 복잡한 생성자

- 너무 많은 의존 관계를 생성자로 주입 받게 될 수도 있음.
- 이 때는 팩토리 메서드 추출과 재정의 기법을 적용.
- 또는 setter를 통해 객체를 주입(선호 X).

### 까다로운 전역 의존 관계

- 테스트 하려는 코드 내에 싱글톤이 있을 경우의 이야기.
- 싱글톤에 대한 비판 한참 한 뒤,
- `getInstance`에 대응하는 `setInstance`를 만들어 테스트에 활용하라고.
- 또는 인터페이스 추출과 상속을 통한 우회.

## 테스트 하네스에서 이 메서드를 실행할 수 없다

- 잠시 test harness의 정의를 다시 돌아봄.
- 쉬운 설명으로는 [여기 StackOverflow의 답변](https://stackoverflow.com/questions/11625351/what-is-test-harness/11628329)을 참고.

### 숨어 있는 메서드

- 메서드를 변경해야 하는데(그리고 테스트 해야 하는데) 그 메서드가 private일 수도.
- private 메서드를 테스트해야 한다면, 이는 public으로 만들어야 한다는 신호.
- 혹시 public으로 만들기 꺼려진다면 클래스가 너무 많은 책임을 갖기 때문일 것이며 분리를 고려해야.
- 좋은 설계 = 테스트 가능한 설계
- 다만, 무작정 public으로 바꾸기엔 위험한 경우도 있음(책 예시: `setSnapRegion`).
- 그리고 기한이나 주변 위험 요소 등을 고려할 때 리팩토링이 어려운 경우도.
- 이 때는 대상 메서드를 protected로 만들고 상속을 이용해서 테스트 하기도.
- 캡슐화 위반 등의 문제가 있지만 임시 거처로는 괜찮음.

### 탐지 불가능한 부작용

- 리턴값이 void인 경우 테스트를 어떻게 해야 할까.
- 책에서는 아래의 `AccountDetailFrame#actionPerformed` 예시로 설명. ([코드](https://gist.github.com/codehumane/885be1735d45f86fb7ce0c8e8e36eb50))
- 가장 먼저, 이벤트에 대한 처리와 그렇지 않은 것을 분리. ([코드](https://gist.github.com/codehumane/305f0bf00a8bbba0048d2e536c57f4b7))
- 다음으로 프레임(`DetailDisplay`)에 접근하는 코드 부분을 메서드로 추출. ([코드](https://gist.github.com/codehumane/c14a9ede77e84f0b58308f120979d305))
- 다음으로 `AccountDetailFrame`의 컴포넌트에 접근하는 코드를 추출. ([코드](https://gist.github.com/codehumane/e986da876d4ce4c9807074baf2f63f14))
- 그리고 서브클래스화와 메서드 재정의를 이용해 테스트. ([코드](https://gist.github.com/codehumane/214b81d837ff0b256a34e6a8f217912e))
- 이제 `AccountdetailFrame`은 5개의 메서드를 가짐. 이는 임시 단계.
  - `performAction`
  - `performCommand`
  - `getAccountSymbol`
  - `setDisplayText`
  - `setDescription`
- 결국엔 `detailDisplay` 변수만을 사용하는 `getAccountsymbol`, `setDescription`은 `SymbolSource`라는 새로운 클래스로 추출하고,
- `display`만을 사용하는 `setDisplayText`는 `AccountDetailDisplay`라는 새로운 클래스로 추출.

## 코드를 변경해야 한다. 어느 메서드를 테스트해야 할까?

- Reasoning About Effects, Reasoning Forward, Effect Propagation 언급.
- 흔히 변경의 영향 범위 파악할 때의 과정을 글로 나타낸 것.
- 참고로, Reasoning About Effects와 Reasoning Forward의 차이는 아래와 같이 설명.
- previous가 가리키는 게 Reasoning About Effetcs.

> In the previous example, we tried to deduce the set of objects that affect values at a particular point in code. When we are writing characterization tests, we invert this process. We look at a set of objects and try to figure out what will change downstream if they stop working.

- 전자는 영향 주는 객체들을 조사한 것이고, 후자는 영향 받는 객체들을 조사한 것.

## 클래스 의존 관계, 반드시 없애야 할까?

### 교차 지점

- 변경에 의한 영향을 감지할 수 있는 프로그램 내의 위치.
- 말은 어렵지만 이 역시 우리가 흔히 하는 일에 대한 이야기.
- 책에서는 `getValue`가 여기에 해당.

### 상위 수준의 교차 지점

- 한 가지 변경을 위해 몇 개의 클래스를 같이 고치는 대신,
- 상위 수준에서 한 번에 테스트를 작성할 수도 있음을 이야기.
- 이 지점이 변경의 영향을 검출하기 위한 유일한 곳일 수도 있음.
- 이 때는 이 상위 교차 지점을 저자는 조임 지점이라고 부른다고 함.
- 하지만 이것이 개별 단위 테스트를 대체하는 것은 아님.
- 단지 변경/리팩토링과 개별 단위 테스트 작성을 위한 첫 단계.

### 조임 지점을 이용한 설계 판단

- 조임 지점은 자연적인 [캡슐화](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming))의 경계이기도 함.
- 같이 바뀌는 것들의 접점이며, 이 접점의 호출부는 바뀌는 것들의 세부 사항은 몰라도 됨.

### 조임 지점의 함정

- 단위 테스트가 소규모 통합 테스트로 점점 커질 수 있음.
- 테스트 규모가 커지면 수행 시간이 오래 걸림.
- 가능한 좀 더 작은 단위로 테스트를 분리하는 것이 바람직.

## 변경해야 하는데, 어떤 테스트를 작성해야 할지 모르겠다

### 문서화 테스트

- 책 중간 중간 문서화 테스트<sup>characterization test</sup>가 종종 언급됐는데 이제야 설명이 나옴.
- 예상한 대로, 시스템이 어떻게 동작하고 있는지를 알려주는 테스트.
- 코드를 이해하기 위해 작성하는 테스트 루틴이기도.
- 저자는 "기존 동작 유지에 필요한 테스트"라고 말하고 있음.
- 레거시 시스템의 메서드를 사용하기 전 그 메서드의 테스트 루틴이 없다면 먼저 루틴을 작성하라는 당부도.

### 정해진 목표가 있는 테스트

- 변경하려는 코드에 대해 테스트 루틴이 있는지 확인하고 없으면 추가.
- 그리고 실패해야 할 테스트가 성공되는 일이 없도록 테스트 입력 값을 잘 선택하라는 얘기도.
