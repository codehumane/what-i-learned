# Building Microservices, 2nd Edition

두 번 읽었던 책인데 2판 나왔길래 아래 챕터들만 다시 읽어보는 것으로.

- CH1. What Are Microservices?
- CH2. How to Model Microservices?
- CH3. Splitting The Monolith
- CH6. Workflow
- CH10. From Monitoring to Observability
- CH12. Resiliency
- CH13. Scaling
- CH14. User Interfaces
- CH15. Organizational Structures

# Chapter 1. What Are Microservices?

## Microservices at a Glance

기본 개념.

- 독립적으로 배포 가능한 서비스.
- 이 서비스는 비즈니스 도메인을 모델링.
- 기능들을 캡슐화하며, 다른 서비스들이 네트워크를 통해 접근.
- 이들이 모여 전체 시스템을 구성.

SOA와의 관계.

- 물론 서비스 경계를 그리는 것에 대해 논란이 있고,
- 독립적 배포가 핵심이긴 하나,
- SOA의 한 형태.
- 특정 기술에 구애받지 않는 것이 한 가지 주요 이점.

캡슐화와 DB.

- 기능은 캡슐화되어 있고,
- 이 기능은 네트워크 엔드포인트를 통해 노출.
- 구현 세부 사항은 숨겨져 있다는 의미이며,
- 공유 DB가 없는 형태를 주로 가짐.
- 필요하면 각자 DB를 가지고 캡슐화.

정보 은닉과 변경.

- 외부 인터페이스로 노출하는 것을 제외하고는 정보를 가능한 숨김.
- 외부 인터페이스에 영향을 주지 않는 내부의 변경은 쉬우며,
- 이는 배포 독립성에 있어 핵심.
- 따라서 명확하고 안정적인 서비스 경계가 중요.
- 높은 응집도와 낮은 결합도.
- 이를 위한 한 가지 방법으로 헥사고날 아키텍처 언급.

## Key Concepts of Microservices

### Independent Deployability

- 다른 서비스의 배포 없이도,
- 한 서비스를 수정하고 배포할 수 있는 것.
- 이를 위해 서비스 간 낮은 결합도를 유지해야 함.
- 이는 서비스 간의 계약이 명시적이고, 잘 정의되어 있으며, 안정적이어야 함을 의미.
- 공유 DB 등의 선택은 이를 어렵게 만듦.

### Modeled Around a Business Domain

- 2개 이상의 마이크로서비스를 동시에 수정해야 하는 기능 출시는 비용이 큼.
- 관련 팀들이 같이 협업해야 하고, 새로운 버전의 배포 순서 등도 고려해야 함.
- 따라서 한 번에 한 곳만 바뀌는 경계 설정이 중요.
- 종종 3티어 아키텍처(presentation/business logic/data)를 만나게 되는데,
- 이 3티어 계층이 모두 같이 바뀌는 경우가 많음.
- 계층이 복잡해 질수록 문제는 더 심각.
- 기술적 응집력보다는 비즈니스적 응집력을 선호.

### Owning Their Own State

- 사람들이 가장 힘들어 하는 것이 공유 DB를 안 쓰는 것.
- 데이터의 소유자에게 데이터를 제공 받아야 함.
- 이런 식으로 숨겨야 할 것과 공유해야 하는 것을 잘 나누면,
- 하위호환성을 위한 비용을 절감할 수 있음.
- 앞에서 언급했든 같은 비즈니스 기능을 가진 것끼리 모아야 함.
- 비즈니스 관련 변경의 비용을 줄이는 것이 목적.
- DB 데이터 역시 마찬가지.

### Size

- 얼마나 작아야 하는지는 자주 나오는 질문.
- 하지만 실제 마이크로서비스에서는 그다지 관심사 주제가 아님.
- 그리고 크기 측정은 또 어떻게 할 것인지가 모호.
- 라인 수, 인터페이스의 수, 이해 가능한 정도 등 모두 한계가 있음.
- 이보다는 아래 2가지가 더 중요.
- 먼저, 마이크로서비스를 얼마나 많이 다룰 수 있는 역량이 되느냐(많을수록 복잡성도 증대).
- 다음으로, 서로 강하게 결합되지 않는 경계를 어떻게 나눌 것인가.

### Flexibility

> microservices buy you options.

- 서비스를 나누면 유연성이 올라감.
- 조직적, 기술적, 확장적, 강건적 유연성.
- 하지만 비용/복잡성도 함께 올라감.
- 따라서 선택의 문제.
- 저자는 점진적 적용을 권장.
- 가고 있는 길의 영향을 좀 더 판단하기 쉽고,
- 멈추거나 돌아오기도 쉽기 때문.

### Alignment of Architecture and Organization

> Organizations which design systems...are constrained to produce designs which are copies of the communication structures of these organizations.

- 콘웨이 법칙 이야기.
- 과거와 지금은 다름.
- 사일로 현상과 핸드오프를 줄이고,
- 더 빠르고 잦은 소프트웨어 출시를 바람.
- 기술 계층으로 팀을 구분하기보다,
- 비즈니스 경계로 팀을 구성.
- 다른 팀과의 커뮤니케이션 비용 없이도,
- 자율적으로 빠르게 원하는 기능을 결정하고 출시.
- 14, 15장에서 좀 더 자세히 다룸.
