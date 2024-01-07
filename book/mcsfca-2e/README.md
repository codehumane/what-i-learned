# Multi-Cloud Strategy for Cloud Architects - Second Edition

# CH1. Introduction to Multi-Cloud

- 멀티 클라우드의 선택 이유로, 안정성이 아닌 사업 민첩성을 언급.
- 사업의 필요에 맞는 클라우드 서비스를 선택하는 것.
- 이 첫 문단을 보니 내가 기대한 내용이 아니겠다 싶음.
- 당연한 얘기지만, 요구사항의 수집이, 멀티 클라우드로의 전환에 앞서 가장 중요.
- 그래서 VOC를 수집하는 도구들 얘기도 다룬다고 함.

## Understanding multi-cloud concepts

https://www.techopedia.com/definition/33511/multi-cloud-strategy

> Multi-cloud refers to the use of two or more cloud computing systems at the same time. The deployment might use public clouds, private clouds, or some combination of the two. Multi-cloud deployments aim to offer redundancy in case of hardware/software failures and avoid vendor lock-in.

## Multi-cloud⎯more than just public and private

- 하이브리드 IT와 멀티 클라우드의 차이점 중 하나는,
- 하이브리드는 homogeneous고 멀티 클라우드는 heterogeneous라는 것.
- 하이브리드는 클라우드 솔루션이 한 스택에 속함(Azure 퍼블릭 클라우드 + Azure Stack 온프레미스).
- 퍼블릭 클라우드는 예를 들어 Azure와 AWS의 조합.
- 하이브리드는 매우 일반적인 배포 모델이며, 클라우드의 미래 모델로 꼽히는 대상.
- 장점은 보안성과 지연(온프레미스가 가진 장점이기도 한).
- 하지만 퍼블릭 클라우드에서의 개발이 보다 민첩.
- 그래서 보안성과 지연이 중요한 것은 온프레미스에,
- 민첩한 개발이 필요한 개발은 퍼블릭 클라우드에서 진행.
- 이것이 하이브리드 IT의 탄생.
- 그러나 책에서 우리는 하이브리드 IT와 멀티 클라우드를 구분할 것.

## Introducing the main players in the field

생략

## Evaluating cloud service models

클라우드가 제공하는 다양한 서비스 모델들 이야기.

### IaaS

- 기업들은 클라우드 프로바이더로 마이그레이션 시 IaaS부터 시작.
- 인프라를 가리킴: 네트워크, 스토리지, compute, 가상화 레이어.

### PaaS

- 리소스에서 나아가 OS와 미들웨어까지 포함.
- 한 예로 MySQL이나 PostgreSQL 등의 데이터베이스 플랫폼.

### SaaS

- 일반적으로 미래 모델로 받아들여지는 서비스.
- 소프트웨어 스택의 모든 것을 클라우드 제공자가 관리.
- 애플리케이션과 데이터까지.

### FaaS

- 서버리스 컴퓨팅 애플리케이션을 만들고 관리할 수 있게 하는 클라우드 서비스.
- 정확히 필요한 만큼의 CPU나 메모리 등의 자원을 사용.

### CaaS

- Container as a Service.
- 컨테이너 클러스터를 쉽게 설정할 수 있게 도와줌.

### XaaX

- Anything as a Service.
- 모든 것을 서비스로 제공할 수 있다는 아이디어를 표현한 것.
- Hardware as as Service, Desktop as a Service, Database as a service, ...

## Setting out a real strategy for multi-cloud

- 클라우드 전략은 비즈니스 목표로부터 나옴.
- 예컨대, 브랜드 인지도 높이기, 시장 출시 속도 높이기, 순이익 늘리기.
- 수익을 창출하고 올리는 게 궁극적 목표.
- 이 목표가 책을 계속 읽게 하는 가장 큰 동인. 궁금.
- 하지만 아쉽게도 구체적인 내용이 이 절엔 없음.
- 90년대 이후로 IT가 회사에 있어 핵심 활동 중 하나가 됐고,
- 이런 상황에서 멀티 클라우드는 조직에 유연성과 선택의 자유를 늘려줌.
- 다만, 이는 리스크도 가져옴. 집중력이 분산되고 관리의 복잡성을 늘림.
- 그래서 전략이 필요하다는 이야기.
- "cloud-first"가 아닌 "cloud-fit" 전략.
- 적합한 것을 골라야.

## Gathering requirements for multi-cloud

- 클라우드 환경을 설계하고 구축하기에 앞서 요구사항 수집해야 함.
- "클라우드 기술을 사용하면 비즈니스에 어떤 가치를 더 일으킬 수 있지?"
- 민첩성, 속도, 유연성, 재정적 측면 등을 고려.
- 능력이 있어서 클라우드로 가는 게 아님.
- 분명한 이점을 가져다 주기에 움직이는 것.
- 기술은 마지막.

### Using TOGAF for requirements management

- TOGAF는 요구사항 수집에 도움.
- 보통 아래의 활동들을 함.
  - 요구사항 식별.
  - 요구사항들의 기준선 생성.
  - 기준선을 기반으로 요구사항 변경점들 식별.
  - 변경점들의 영향도 평가.
  - 요구사항 리포지토리 만들고 관리.
  - 요구사항 구현.
  - 제품 명세와 요구사항 사이의 차이점 분석 수행.
- 클라우드를 위한 비즈니스 요구사항들은 아래 유형들로 분류 가능.
  - interoperability
  - configurability
  - performance
  - discoverability
  - robustness
  - portability
  - usability
- 보안의 중요성도 강조.
- 클라우드를 이용해 비즈니스가 얻고자 하는 것들은 주로 3가지.
  - scalability
  - reliability
  - ease of use

기대했던 내용들이 아니고, 대략적인 내용들도 어느 정도 파악됨. 중간은 생략하고, 아래 2개 정도만 추가로 보기.

1. Service Designs For Multi-Cloud
2. Designing Applications For Multi-Cloud
