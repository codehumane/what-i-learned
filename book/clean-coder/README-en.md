# Clean Coder

마음 다잡고자 영문판으로 다시 읽기 시작.

# CH1. Professionalism

## Be Careful What You Ask For

- 버그를 만들어 천만원 손해가 났다면,
- 내가 이 비용을 지불하는 것이,
- 프로페셔널이라 생각하면 됨.

## Taking Responsibility

> Upon reflection I realized that shipping without testing the routine had been irresponsible. The reason I neglected the test was so I could say I had shipped on time. It was about me saving face. I had not been concerned about the customer, nor about my employer. I had only been concerned about my own reputation. I should have taken responsibility early and told Tom that the tests weren’t complete and that I was not prepared to ship the software on time. That would have been hard, and Tom would have been upset. But no customers would have lost data, and no service managers would have called.

## First, Do No Harm

- 우리 역량을 선한 곳에 쓰는 것을 책임이라 이야기.
- 개발자로써 해를 끼칠 수 있는 것으로는 소프트웨어의 기능과 구조를 꼽음.

### Do No Harm to Function

- 버그를 만들지 않기.
- 소프트웨어가 복잡해서 버그가 생길 수도 있긴 하나,
- 더 복잡한 사람의 몸을 다루는 의사 사례로 반박.
- 어렵다는 핑계로 버그를 만들 수 있다는 생각은 위험.
- 물론, 완벽할 순 없음.
- 하지만 불완전함에 책임져야 함.
- 오류가 0일 순 없으나,
- 오류가 0에 도달하도록 노력하는 것이,
- 우리의 책임.

### QA Should Find Nothing

- QA가 결함을 찾을 수 없어야 함.
- 버그 찾기를 QA에 의존하면 X
- 제대로 작동하는지 확신 못 하면 X.
- 만약 발견되면 사과하고,
- 나는 왜 찾아내지 못했을지 검토하고,
- 예방을 위해 노력.

### You Must Know It Works

- 코드가 제대로 동작하는지 어떻게 알까?
- 테스트하면 됨. 이런 저런 방법으로 테스트.
- 근데 시간이 걸린다고?
- 그럼 자동화 하면 됨.
- 그리고 100% 커버리지 요구.
- 테스트하기 어렵고, 만약 구조적인 이유다?
- 구조를 테스트하기 쉽게 만들 것.

### Automated QA

- 전체 QA 절차는,
- 단위 테스트와 인수 테스트.
- 몇 분이면 끝남.
- 물론 품질이 매우 중요한 곳에는,
- 자동화 테스트로 충분치 않을 수 있으나,
- 품질의 충분 요건이 아닐 뿐 필수.

### Do No Harm to Structure

- 구조를 희생하고 기능을 전달하는 것은 바보.
- 보통 이는 소프트웨어는 변경이 쉽다는 잘못된 가정에 기반.
- 항상 변경이 쉽게 만들어야 함.

## Work Ethic

- 당신의 커리어는 당신의 책임.
- 회사는 의무 없음.
- 시장성, 훈련, 책 구매 모두.
- 만약 이를 돕는다면 그것은 호의.
- 주 40시간은 고용주의 문제에 쓰여야 함.
- 그리고 나머지 20시간을 당신을 위해 사용해야.
- 독서, 연습, 배움 등 경력을 발전시킬 것에.
- 때로는 회사의 일이 경력에 도움되기도.
- 이 경우라면 20시간을 회사에 써도 됨.
- 그래도 이 20시간을 당신을 위한 것임을 잊지 말아야.

### Know Your Field

- 아래 질문들로 시작.
- Nassi-Schneiderman 차트를 아는가?
- Mealy와 Moore의 상태 머신은 아는가?
- 모른다면 왜 모르는가?
- 퀵소트를 혼자 작성해 볼 수 있는가?
- "Transform Analysis"의 의미는 무엇인가?
- 데이터 플로우 다이어그램으로 함수 분해를 할 수 있나?
- "Tramp Data"의 의미는?
- "Conascence"는 들어 봤나?
- Pernas Table은 뭔가?
- 이것이 과거의 데이터이지만,
- 많은 것이 빠르게 변한 반면,
- 여전히 많은 지식들은 여전히 필요로 함.
- `if`와 `while` 문이 아직 쓰이는 것처럼.
- 과거를 기억하지 못하면 이를 반복하기 마련.
- 아래의 것들은 최소한으로 알고 있어야 하는 지식들.
- 디자인 패턴 - GOF, POSA.
- 디자인 원칙 - SOLID, 컴포넌트 원칙
- 방법론 - XP, 스크럼, 린, 칸반, 워터폴, 구조적 분석, 구조적 설계.
- descipline - TDD, OOD, 구조적 프로그래밍, CI, 페어 프로그래밍.
- artifact - UML, DFD, 스트럭처 차트, Petri Nets, 상태 전이 다이어그램과 테이블, 플로우 차트, 의사결정 테이블.
