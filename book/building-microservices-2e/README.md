# Building Microservices, 2nd Edition

두 번 읽었던 책인데 2판 나왔길래 앞부분만 빠르게 다시 읽어보는 것으로. 읽어 보려는 챕터는 아래와 같음.

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
