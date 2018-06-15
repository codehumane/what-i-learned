# 머리말

## 마가릿 해밀턴의 일화

머릿말에 마가릿 해밀턴<sup>Margaret Hamilton</sup>의 일화가 소개됨. 내용은 [여기](https://www.wired.com/2015/10/margaret-hamilton-nasa-apollo/)의 "That would never happen" 부분에서도 찾아볼 수 있음. 아래는 이 일화에 관한 마가렛의 이야기.

>  "시스템이 어떻게 동작하는지를 완벽하게 이해한다고 해서 사람의 실수를 예방할 수는 없다."



# 소개

## 지속적 엔지니어링

-  SRE팀에 티켓, 전화 응대, 수작업 등 소위 운영 업무에 최대 50%의 시간만 투입.
-  서비스를 안정적으로 운영할 수 있는 상태를 유지하기 위한 시간 확보가 목적.
-  이를 위해 시간 측정은 물론, 상한선 초과시 티켓 할당이나 전화 담당 업무를 개발팀에게 넘김.

## 에러 예산

-  더 이상 '무정지 시스템' 같은 목표를 세우지 않음.
-  사용자 노트북, 가정 WIFI, ISP 등 다른 시스템의 불가용성 존재. 사용자는 결국 100% 가용성을 누릴 수 없음.
-  그래서 허용 가능한 불가용성을 설정(예컨대 1 - 99.99)하고 이를 에러 예산<sup>error budget</sup>으로 칭함.
-  예산이라는 말 그대로, 제품 개발팀은 기능의 출시 속도를 극대화하기 위해 이를 활용함.

## 모니터링

-  사람이 알림을 읽고 대응 여부를 결정하는 것은 문제.
-  장애 판단은 소프트웨어가 수행. 특정 대응이 필요한 경우에만 사람에게 알림이 도착해야 함.
-  모니터링 결과를 3가지로 구분하고 있음.
-  알림<sup>alerts</sup>: 사람이 즉각적으로 대응해야 함.
-  티켓<sup>tickets</sup>: 사람의 대응이 필요하지만 즉각적일 필요는 없음.
-  로깅<sup>logging</sup>: 사람이 정보를 확인할 필요는 없음. 하지만 향후 분석이나 조사를 위해 기록.

## 긴급 대응

-  신뢰성이란, MTTF(Mean Time To Failure)와 MTTR(Mean Time To Repaire)을 의미.
-  이 중 긴급 대응의 효율성은 MTTR으로 표현됨.
-  시스템의 MTTR 대비 사람의 그것은 3배 이상.
-  사람의 개입이 불가피한 경우는 행동 지침을 바탕으로.
-  유능한 엔지니어보다 행동지침 바탕으로 훈련된 엔지니어의 성과가 더 큼.

## 변화 관리

-  장애 상황의 70%가 시스템의 변화로 인한 것.
-  따라서 아래 3가지를 자동화하는 것이 효과적.
-  제품의 단계적 출시, 문제를 빠르고 정확하게 도출, 문제 발생 시 안전하게 이전 버전으로 롤백.

## 그 외

-  `SW 엔지니어링 자격 요건에 근접한 인력 = 요구되는 기술 항목의 85~99% 만족`으로 수치화 하는 부분이 흥미로움.
-  알림을 발생시키지 않은 건에 대한 포스트모텀을 중요하게 여김.
-  수용 계획은 자연적 성장과 인위적 성장을 함께 고려해야 함. +수요 예측 이야기. 뒷 장들에서 어떻게 풀어나갈지 기대.
-  책에서는 프로비저닝<sup>provisioning</sup>을 변화 관리와 수용 계획을 합한 개념으로 소개.

# SRE 관점에서 바라본 구글의 프로덕션 환경

## 개발 환경

-  개발자라서 어쩔 수 없나 보다. 가장 와 닿는 부분.
-  일부 오픈 소스 저장소 그룹을 제외하면, 모든 엔지니어들은 하나의 공유 저장소를 바탕으로 업무 수행.
-  프로젝트 외부의 컴포넌트에서 문제 발생 시, 코드를 수정하고 CL(changelist)을 소유자에게 보내 리뷰 요청.
-  승인된 변경사항은 제출되고 직/간접적으로 영향 받는 모든 소프트웨어에 대해 테스트 수행됨.
-  문제가 있는 경우 변경사항을 제출한 소유자에게 알림.
-  문제 없다면 (일부 프로젝트의 경우) 자동으로 프로덕션에 배포. 즉, [푸시-온-그린(push-on-green)](https://en.wikipedia.org/wiki/Push_on_green).

## 요청의 흐름

-  아래 그림은 요청의 흐름 예시.
-  GSLB에 대한 의존성이 커서 엄격한 테스트, 배포 정책, 사전 장애 복구 정책등을 적용.

![request flow](02-request-flow.png)

## 작업과 데이터의 조직화

-  부하 테스트를 통해 백엔드 서버가 100QPS를 처리할 수 있음을 알게됐다고 함.
-  100이라는 수치로 이런 저런 생각에 잠기는 것도 잠시. 아래 내용도 흥미로움.
-  특정 환경에서의 A 서비스가 최대 3,470QPS가 발생될 수 있음을 알게 됨.
-  이 때 필요한 태스크의 최소 수는 35개가 아닌 37개 혹은 N+2개.
-  업데이트 혹은 머신 장애가 일어날 수 있기 때문임.
-  `35개 혹은 N이라는 수치` = `단지 최대 부하를 견디기 위해 필요한 최소한의 태스크 수`
-  하지만 비용 절감 목적으로 지연응답을 감수하기도 함.
-  또한, 사용자 트래픽은 전 세계에 분산되어 있음.
-  지연 방지를 위해 데이터 저장소도 역시 분산되어야 함.
-  이를 위해 지역별로 데이터 복제. 서버 장애 대응은 물론, 데이터 접근에 대한 응답 지연도 함께 해소.


# 위험 요소 수용하기

## 위험과 혁신의 균형

- 앞에서 언급된 것 처럼, 업타임<sup>uptime</sup> 극대화 대신, 빠른 혁신과의 균형을 꾀한다는 이야기.
- 패리티 코드<sup>parity code</sup>라는 용어를 사용하는 것이 재밌음. [Parity Bit](https://en.wikipedia.org/wiki/Parity_bit)를 생각하면 됨.

## 서비스 위험 측정

- 의도되지 않은<sup>unplanned</sup> 다운타임 대신 요청 성공률로 가용성을 정의함.
- 여러 지역을 활용한 분산 서비스를 운영해서 장애 분리가 되기 때문.
- 기존의 가용성 공식이 `가용성 = 업타임 / (업타임 + 다운타임)`이라면,
- 구글의 가용성 공식은 `가용성 = 성공한 요청 수 / 전체 요청 수`임.
- 다운 타임을 [SLI<sup>Service Level Indicator</sup>](https://en.wikipedia.org/wiki/Service_level_indicator)에서 제외했다는 것이 재밌음. 언제나 목적이 중요.

## 위험 수용도

- 서비스의 위험 수용도<sup>risk tolerance</sup>를 소비자 대상 서비스와 인프라스트럭처 서비스를 구분하여 정의.
- 또한, 장애의 종류에 따라 위험도를 다르게 설정하기도. 유/무료 여부, 경쟁자의 가용성, 수익과의 직접적인 연관성 등도 고려.
- 비용을 측정하는 부분도 재미있음. 예전엔 과연 이 지표가 의미 있을까 싶었는데, 생각이 긍정적으로 바뀜.

## 에러 예산의 활용

- SLI, [SLO](https://en.wikipedia.org/wiki/Service_level_objective)(, 혹은 나아가 SLA)를 정했다면, 이제 이를 어떻게 활용하는지에 대한 이야기.
- 분기별로 SLO 설정하고 결과를 비교. 이로 인해 얻게 된 `에러 예산`이 있다면, 새로운 릴리즈를 허용.
- 경직성<sup>inflexibility</sup>과 느린 혁신을 드러나게 하는 도구라고 표현하기도.

# Service Level Objectives

https://landing.google.com/sre/book/chapters/service-level-objectives.html

>  It’s impossible to manage a service correctly, let alone well, without understanding which behaviors really matter for that service and how to measure and evaluate those behaviors.

## Service Level Terminology

### SLI(Service Level Indicator)

-  측정하려는 항목.
-  주요 지표로는 Request Latency, Error Rate, System Throughput.
-  Availability 또한 주요 지표. Yield(수율?)이라고 불리기도 함. 데이터 스토리지에서는 Durability.

### SLO(Service Level Objectives)

-  SLI의 목표 수준.
-  SLI ≤ target 혹은 lower bound ≤ SLI ≤ upper bound 등의 형태로 표현.
-  시작할 때부터 설정할 필요는 없음.
-  SLO를 고객에게 공개한다는 점은 인상적.
-  고객이 서비스 동작에 대한 예측을 할 수 있음.
-  SLO가 없다면 고객, 운영자, 경영자들은 서로 다른 기대치로 서비스를 바라보게 됨.
-  또한, 의존 서비스들에게 적절한 행위를 취하게 할 수도. 너무 많은 의존성은 피하게 하거나 폴백 등의 마련.

### SLA(Service Level Aggremenet)

-  explicit or implicit contract with your users
-  that includes consequences of meeting(or missing) the SLOs they contain
-  consequences로는 rebate나 penalty 등.
-  따라서, SRE는 SLA의 결정에는 참여하지 않는다고.
-  "SLO를 달성하지 못하면 어떻게 될 것인가?"가 SLO와 SLA의 차이점을 잘 나타냄.

## Indicators in Practice

### What Do You and Your Users Care About? 

-  사용자와의 접점 시스템들은 가용성, 응답 시간, 처리량이 중요.
-  저장소 시스템은 응답 시간, 가용성, 내구성.
-  데이터 처리 파이프라인 등은 처리량과 종단 간 응답 시간. 하지만 성격에 따라 두 지표는 배타적이기도 함.
-  놓치지 않아야 할 것은 정확성<sup>correctness</sup>. SRE가 관여하지는 않음.

### Collecting Indicators

-  Bormon, Prometheus 등.

### Aggregation

-  단순함, 유용함을 위해 데이터 합산하기도 하지만, 이 경우 주의가 필요.
-  예컨대, `1분간 요청의 평균값`은 짝수초 마다 200개, 홀수초 마다 0개의 요청이 발생한 경우를 파악할 수 없음.
-  대부분의 지표들은 평균보다 분포<sup>distribution</sup>이 중요.
-  예를 들어, 단순 평균은 아웃 라이어(책에서는 Tail Latency라고 표현)를 인지하기 어려움.
-  보통 이를 해결하기 위해 백분위수<sup>percentile</sup>를 사용하는데, 책에서도 같은 이야기를 한다.

### Standarlize Indicators

-  표준화된 척도는 많은 노력을 절감시켜줌.
-  의사소통 비용이나 시스템 간의 비교 등이 용이해 질 것으로 보임.

## Objectives in Practice

-  서비스 간의 통신이 늘어나면서, 서비스 지표와 목표를 설정하는 것이 중요해 짐을 느낌.
-  지표와 목표가 있다면 이에 의존하는 서비스들은 Fail Fast, Timeout, Bulkhead 등을 적절하게 설정할 수 있음.
-  에러 예산 같은 개념도 활용할 수 있고 말이다. 즉 적절한 균형을 잡을 수 있는 것.
-  앞에서 언급된 것 처럼 서로의 기대치를 어느 정도 일치시킬 수도 있고 말이다.
-  서비스의 상태 추이를 알 수 있으므로 잠재적인 문제에 어느 정도 대응이 가능하기도.

다음은 목표치 선택에 도움이 되는 팁들.

1. Don’t pick a target based on current performance. 제목과 다르게 현재 시스템의 수준을 고려해야 한다는 이야기.
2. Keep it simple. 복잡하면 이해하기도 파악하기도 어려움.
3. Avoid absolutes. 자기 만족에 너무 지나친 수준을 설정하지 말 것.
4. Have as few SLOs as possible. 의외이다. SLO가 사용자의 만족을 정의하는 것의 한계도 한 몫 함.
5. Perfection can wait.

>  SLOs are a massive lever.

# Eliminating Toil

> "If a human operator needs to touch your system during normal operations, you have a bug."
>
> \- Carla Geisser, Google SRE

## Toil Defined

- 사전적 의미의 Toil은 "work extremely hard or incessantly"
- 책에서의 의미는 "running a production service that tends to be manual, repetitive, automatable, tactical, devoid of enduring value, and that scales linearly as a service grows"
  - Manual: 스크립트로 수행 vs. 사람의 실행
  - Repetitive: 한 두번이 아닌, 계속되는 반복 작업
  - Automatable: 머신을 통해 사람이 하는 일을 대체할 수 있는 경우. 하지만 인간의 판단이 필요한 경우는 제외.
  - Tactical: strategy-driven & practive vs. interrupt-driven & reactive
  - No enduring value: 서비스의 상태가 작업 후에도 별반 차이가 없음. 반대로 서비스의 영구적 개선을 가져온다면, 너저분한 작업이 필요하다고 하더라도 Toil X.
  - O(n) with service growth: 서비스의 크기나 트래픽 양에 비례하여 선형적으로 작업량이 늘어남.

## Why Less Toil Is Better

- SRE 팀은 Toil 같은 운영 작업을 업무의 50% 이내로 유지하려 함.
- 나머지는 서비스 피처를 더하거나, 미래의 toil을 줄이는 작업 등에 투자.
- SRE 조직이 서비스 크기에 부선형적으로<sup>sublinearly</sup> 스케일 업 되기 위함.

## What Qualifies as Engineering?

- 엔지니어링 작업은 새로우면서도 흥미롭고<sup>novel</sup> 사람의 판단을 필요로 하는 일들임.
- 이를 통해 동일한 인력으로 더 큰 서비스 혹은 더 많은 서비스를 관리할 수 있음.
- SRE 활동은 다음처럼 4개로 분류됨.
  - 소프트웨어 엔지니어링: 코드 작성이나 관련 디자인 혹은 문서화 작업.
  - 시스템 엔지니어링: 프로덕션 시스템 설정 조정이나 문서화. 개발팀에게 서비스 출시 등 조언 등.
  - 삽질: 서비스 운영과 직접 관련된 반복 수작업.
  - 부하: 운영과 직접 관련되지 않은 업무들. 채용, 팀/회사 회의, 버그 큐 치우기, 업무 보고, 동료 평가 등.

## Is Toil Always Bad?

- Toil이 매번 모든 사람을 행복하지 않게 만드는 것은 아님.
- 또한, 어느 정도의 삽질은 필수 불가결.
- 하지만, 일정량 이상으로 늘어나면 아래와 같은 문제들을 유발.
  - **Career stagnation**
  - **Low morale**: 개인 별 한계를 넘는 경우 burnout, boredom, discontent.
  - **Creates confusion**: SRE 조직은 엔지니어링 조직이라는 것을 인식시키기 위해 노력함. 하지만 Toil이 많은 경우, 스스로 조직의 역할에 대한 의구심. 타 조직과의 의사소통이 어려워지기도.
  - **Slows progress**: Less productive. Feature velocity will slow.
  - **Sets precedent**: 좋지 않은 선례로 인해, 더 많은 단순 반복 운영 업무들을 떠안게 될지도.
  - **Promotes attrition**: start looking elsewhere for a more rewarding job.
  - **Cuases breach of faith**: 기대와 다른 업무로 팀 동료들의 신뢰 문제가 발생할 수도.

# Monitoring Distributed System

모니터링과 알림 시스템을 성공적으로 구축하는 방법에 대해 소개. 특히, 사람이 반드시 개입되어야 하는 이슈와, 그렇지 않은 이슈는 어떻게 다루어야 하는지를 가이드.

## Definitions

- **Monitoring**: 시스템에 대한 실시간 정량적 데이터를 실시간 수집, 가공, 종합, 그리고 표기. QOS, 처리 시간, 에러 수와 타입 등.
- **White-box monitoring**: 시스템 내부 지표들의 모니터링. 로그, JVM 프로파일링 인터페이스, HTTP 핸들러를 통해 얻은 통계들
- **Black-box monitoring**: 사용자가 보게 되는 외부에 가시적인 행위들을 테스트.
- **Dashboard**: 서비스의 핵심 지표들을 요약 뷰로 제공하는 어플리케이션. 필터, 선택자 등을 가짐. 티켓 큐의 길이나 우선순위 높은 버그들, 현재 On-Call 엔지니어 등을 보여줌.
- **Alert**: 사람이 읽도록 의도된 통지. 이 알림들은 티켓, 이메일 알림, 호출 등으로 분류됨.
- **Root cause**: 만약 수정되면, 다시는 같은 방식으로 일어나지 않을, 소프트웨어나 휴먼 시스템의 결함. 예컨대, 다음 3가지의 결합이 Root Cause 일 수 있음. 불충분한 프로세스 자동화, 조작된 입력에 의한 소프트웨어 충돌, 그리고 설정 생성을 위해 사용된 스크립트를 충분히 테스트 하지 못함.
- **Node and machine**: 물리 서버, 가상 머신, 컨테이너 등에서 동작하는 단일 커널 인스턴스. 단일 머신 위에 모니터링 해야 하는 여러 개의 서비스가 있을 수 있음.
- **Push**: 실행 중인 소프트웨어나 설정에 대한 모든 변경 사항.

## Why Monitor?

- **Analyzing long-term trends**: DB의 크기는 얼마나 빠르게 증가하는가? 사용자 수는?
- **Comparing over time or experiment groups**: 예를 들어, 노드를 추가했을 때 memcache의 hit rate는 얼마나 올라가는가?
- **Alerting**: 뭔가 잘못된 것을 감지하고 수정. 혹은 곧 문제가 생길 것 같아서 대비.
- **Building dashboards**: 서비스에 대한 기본적인 질문들에 대한 답변. 일반적으로 [네 가지 결정적 신호<sup>The Four Golden Signals</sup>](https://landing.google.com/sre/book/chapters/monitoring-distributed-systems.html#xref_monitoring_golden-signals)를 가짐.
- **Conducting ad hoc retrospective analysis (i.e., debugging)**: 응답 지연이 발생했다. 같은 시간에 다른 일들이 발생한 것이 있는가?

## Setting Reasonable Exceptions for Monitoring

- 견고하고 자동화된 모니터링 시스템을 갖추고 있음에도, 항상 최소 한 명씩은 모니터링 작업을 수행함. 그 정도로 매우 중요한 업무.
- 더 나은 사후 분석<sup>post hoc analysis</sup> 도구를 통해, 더 빠르고 간단한 모니터링 시스템이 만들어짐.
- 하지만 스스로 임계치를 학습하고 문제를 감지해내는 마법과 같은 시스템은 피하려고 함.

## Symptoms Versus Causes

- 무엇이 망가졌는가(what's broken)와 왜 망가졌는지(why)에 대한 구분.
- 예컨대, HTTP 500 또는 404를 반환하는 것은 증상, DB 서버가 커넥션을 거부한 것은 원인.
- 최대의 신호와 최소한의 노이즈<sup>maximum signal and minimum noise</sup>를 위해 중요한 구분.

## Black-Box Versus White-Box

- heavy use of white-box + modest but critical uses of black-box monitoring
  - black-box
    - symptom-oriented and represents active(predicted: X) problems
    - "The system isn't working correctly, right now."
  - white-box
    - instarumentation와 함께 로그나 HTTP 엔드포인트.
    - 문제가 발생하려 하는 징후나, 재시도 후에도 실패한 것 등을 감지할 수 있음.
- white-box 모니터링은 symptom-oriented 이면서 cause-oriented 이기도.
  - 예컨대, DB 지연은 그 자체로도 문제 증상이지만 웹사이트가 느려지는 원인이기도 함.
  - 이는 white-box가 얼마나 정보가 많으냐에 따라 달림.
- 디버깅을 위해 원격 측정 지표들을 수집한다면, white-box 모니터링은 필수.
  - 예를 들어, DB에 부하를 주는 요청에 의해 웹 서버가 느려지는 것 같다면,
  - DB 처리 속도는 얼마인지와 웹 서버가 DB 결과를 얼마나 빠르게 인지했는지 알아야,
  - DB 자체가 문제인지 아니면 DB와 웹 서버 사이의 네트워크 문제인지를 인지할 수 있음.
- black-box를 통한 알림은 문제가 이미 발생한 경우 효과적이지만, 곧 발생할 것 같은 문제에는 무의미.


## The Four Golden Signals

단 4가지만 측정할 수 있다면 latency, traffic, errors, saturation을 측정하라.

### Latency

- "The time it takes to service a request."
- 성공 응답과 실패 응답의 구별은 중요.
- 예를 들어, DB나 주요 백엔드 연결 실패에 의한 500 에러는 종종 빠르게 반환됨.
- 만약 전체 응답 지연에 이들을 포함시키면 잘못된 계산 결과를 얻을 수도.
- 한편, 느린 에러는 빠른 에러보다 훨씬 심각한 문제.
- 단지 에러를 구별하는 것에 그치지 않고 이런 에러 응답 지연을 추적하는 것이 중요.

### Traffic

- "A measure of how much demand is being placed on your system."
- 웹 서비스의 경우는 초당 HTTP 요청 수. 그리고 요청 성질에 따라 동적이냐 정적 컨텐츠냐 등으로 나뉠 수 있음.
- 오디오 스트리밍 시스템의 경우는 네트워크 I/O 비율이나 동시 세션에 집중.
- 키-밸류 저장 시스템은 초당 트랜잭션 혹은 조회<sup>retrievals</sup> 수.

### Errors

- "The rate of request that fail."
- 500 응답과 같이 명시적일 수도 있고, 200 응답이지만 잘못된 컨텐츠가 포함된 암묵적일 수도 있음.
- 프로토콜 응답 코드로 실패 조건을 표현하기 어렵다면, 보조(내부) 프로토콜이 필요함.
- 로드 밸런서에서 500을 잡아내는 것에 더해, 종단 시스템 테스트가 필요할 수도.

### Saturation

- "How 'full' your service is."
- 시스템에서 주요한 자원(메모리 혹은 I/O)의 가용성을 가리킴.
- 이들 지표는 100%가 되기 전에 성능 저하가 발생하므로 적절한 목표 설정이 중요.
- 복잡한 시스템에서는 좀 더 상위 수준의 부하 지표로 보완하기도.
  - "두 배의 트래픽을 감당할 수 있는가?"
  - "10% 많은 트래픽은?"
  - "현재보다 트래픽이 적어진다면?"
- 하지만 CPU 이용률이나 네트워크 대역폭 같은 간접적 신호도 함께 사용해야 함.
- latency의 증가는 종종 saturation의 선행 지표.
- 다시 말해, 99th 백분위수 응답 시간 등의 측정은 saturation의 초기 지표가 될 수 있음.

## Worrying About Your Tail (or, Instrumentation and Performance)

- 평균 값을 지표로 삼는 것의 위험성에 대해 이야기.
- 지연 시간을 예로 들고 있음.
  - 초당 1,000개 요청을 받을 때 평균 지연 시간이 100ms라면,
  - 1%의 요청은 5초가 걸릴 수 있음.
  - 만약 특정 페이지 렌더링이 여러 웹 서비스에 의존한다면,
  - 백엔드 1개의 99 백분위수가 프론트엔드의 평균 응답시간이 되기 쉽상.
- 느린 평균 지연과 매우 느린 "tail" 요청을 구분되어야 함.
- 가장 쉬운 방법은 아래와 같이 질문하는 것.
- 더불어, 히스토그램의 경계를 지수(위 질문에서는 대략 3)적으로 나누는 것이 쉽다고도 이야기.

> How many requests did I serve that took between 0 ms and 10 ms, between 10 ms and 30 ms, between 30 ms and 100 ms, between 100 ms and 300 ms, and so on?

## Choosing an Appropriate Resolution for Measurements

시스템은 각각의 관점과 세분화 수준으로 측정되어야 함.

- CPU 부하의 1분 단위 측정은 심지어 꽤 긴 시간 동안의 상승<sup>spikes</sup>도 드러내지 못할 수 있음.
- 1년 동안 총 9시간 미만의 다운타임을 목표로 하는 웹 서비스의 경우, 매 1분마다 200 성공 응답 여부를 조사하는 것은 너무 빈번한 것. 
- 비슷하게, 99.9%의 가용성을 목표로 하는 서비스에 대해, 하드 드라이브 여유 공간을 검사하는 것도 불필요 할 수 있음.

## As Simple as Possible, No Simpler

소프트웨어와 마찬가지로 모니터링 또한 복잡해질 수 있으며, 따라서 깨지기 쉽고 변경하기 어려우며 유지 부담이 높아질 수 있음. 아래는 무엇을 모니터링할지를 결정할 때의 가이드라인.

- 가장 자주 발생하는 사고를 감지하는 규칙은 가능한 쉽고, 예측 가능하고, 신뢰할 수 있어야 함.
- 드물게 사용되는 데이터 수집, 종합, 알림 설정은 제거.
- 수집은 되었지만 대시보드에 노출되지 않거나 알림으로 사용되지 않은 신호는 제거 후보.

## Trying These Principles Together

모니터링과 알림을 위한 규칙을 만들 때, false positives와 pager burnout을 막기 위해 생각해 볼 만한 질문들. 요즘의 고민거리.

- [Zero-redundancy](https://en.wikipedia.org/wiki/N%2B1_redundancy)와 같이, 시급하고, 대처할 수 있으며, 사용자에게 금방 드러날 상황을 알아챌 수 있는 규칙인가?
- 유해하지 않다면 무시할 수 있는 알림인가? 언제, 왜 무시해야 하고, 어떻게 하면 이런 상황 자체를 피할 수 있나?
- 사용자에게 부정적 영향을 미치는지를 분명하게 알려주는 알림인가? drained traffic 혹은 테스트 배포와 같이 사용자에게 부정적 영향을 미치지 않는, 따라서 걸러져야 하지는 않는가?
- 대응할 수 있는 알림인가? 그 대응은 시급한가, 아니면 아침까지 기다릴 수 있는가? 그 대응은 안전하게 자동화 될 수 있는가? 장기적인 수정이 될 것인가, 아니면 단지 단기적인 우회책이 되는가?
- 이미 다른 사람들이 이슈에 대해 호출을 받아서, 불필요한 페이지 렌더링이 적어도 1개 이상 발생하지는 않았는가?

이러한 질문들은 호출<sup>page</sup>과 호출기<sup>pager</sup>에 대한 철학을 반영.

- 호출이 일어날 때 마다 긴장감을 갖고 대응할 수 있어야 함. 하루에 단 몇 번만 이런 긴장감을 가질 수 있으며, 그렇지 않으면 피로가 발생.
- 모든 호출은 대응이 가능해야 함.
- 모든 호출 응답에는 어느 정도의 사고<sup>intelligence</sup>가 필요함. 단순히 로봇의 응답으로 가능하다면, 호출되지 말아야 한다.
- 호출은 새로운 문제 혹은 지금까지 보지 못한 이벤트에 관한 것이어야 한다.


## Monitoring for the Long Term

### Bigtable SRE: A Tale of Over-Alerting

남의 얘기 같지가 않음.

> SLO에 근접해지면 이메일 알림이 발송되었고 SLO를 초과하면 호출기 알림이 발송되었다. 이 두 가지 알림이 너무 빈번하게 발생해서 엔지니어링 시간을 너무 많이 소모했다. 팀은 실제로 대응이 필요한 알림을 선별하는 데 너무 많은 시간을 할애했으며, 실제로 사용자가 영향을 받는 경우에 대한 알림이 극히 적어 종종 놓치는 경우도 많았다. 호출의 상당 부분은 인프라스트럭처에 발생하는 문제점들에 대한 것이었는데, 이 문제점들은 이미 잘 알려져 있는 것이거나, 사실 그렇게 급하게 처리하지 않아도 되는 문제들이어서 이미 정해진 대응책이 있거나 혹은 대응이 필요하지 않은 것들이었다.

그래서 3가지 방안을 도입했다고 함.

1. 빅테이블의 성능을 개선하는 동시에,
2. SLO 목표치를 일시적으로 75% 요청 지연으로 하향 조정.
3. 너무 많이 발송돼 대응에 많은 시간이 소모되던 이메일 알림 중단.

호출기가 울리는 일이 줄어들고, 크고 작은 단기적 문제들을 수정하는 대신, 장기적 문제들을 수정해 나갔던 것. 즉, 단기 가용성 목표와 장기적 관점의 목표 간에 적절한 트레이드 오프를 선택.

### The Long Run

아래 내용도 인상 깊어서 전체를 기록.

>  A common theme connects the previous examples of Bigtable and Gmail: a tension between short-term and long-term availability. Often, sheer force of effort can help a rickety system achieve high availability, but this path is usually short-lived and fraught with burnout and dependence on a small number of heroic team members. Taking a controlled, short-term decrease in availability is often a painful, but strategic trade for the long-run stability of the system. It's important not to think of every page as an event is isolation, but to consider whether the overall level of paging leads toward a healthy, appropriately available system with a healty, visible team and long-term outlook.

# The Evolution of Automation at Google

## The Value of Automation

여러 요소를 나열하고 있긴 하지만, 이것은 많은 트래픽이나 서버를 관리하지 않는 입장에서는 크게 와닿지 않을 수도 있다. 하지만 조금만 상상해 보면 금방 느껴볼 수도 있는 일이다. 서버 3,000대가 있고 모두 업데이트를 해야 한다면? 1초에 들어 오는 트래픽이 상당해서 조금만 회복이 지체 되어도 손실 금액이 크다면?

그런데 말이다. 이를 바꿔 말하면, 규모가 작은 시스템에서는 자동화의 이점이 그리 크지 않을 수도 있다는 얘기가 된다. 모든 작성되는 코드 또한 읽기나 변경 등의 유지 보수 비용이 발생한다. 무작정 자동화가 필요하다고 말하는 대신, 한 번쯤 자동화로 인한 트레이드 오프를 고려해야 하는 것 아닐까.

### Consistency

> For a start, any action performed by a human or humans hundreds of times won’t be performed the same way each time: even with the best will in the world, very few of us will ever be as consistent as a machine.

- 일관성의 부족은 실수와 부주의, 데이터 품질, 신뢰성 등의 문제를 수반함.
- 어떤 강한 의지도 기계 만큼의 일관성을 보장하지는 못함.

### A Platform

- 잘 설계되고 수행되는 자동화 시스템은 플랫폼으로써 가치가 있음.
- 즉, 확장되고, 더 많은 시스템에 적용되고, 더 오랫동안 지속됨.
- 또한, 실수를 중앙화. 잘못된 코드를 한 번 고치는 것으로 모든 곳에 영원히 적용.
- 좀 더 자주, 좀 더 지속적으로 수행할 수 있기도.
- 작업 성능에 대한 메트릭을 추출할 수 있음.

### Faster Repairs

- 공통된 실수에 대해 MTTR이 감소하는 효과를 볼 수도 있음.
- 실제 운영 환경에서의 수정은 시간과 돈 측면에서 높은 비용임. 자동화에 의해 MTTR이 감소되는 것은 이들 비용을 낮춰주는 것.
- 그리고 이 자동화 된 작업이 진행되는 동안 사람들은 더 중요한 것에 시간 투자.
- 뒤이어 나오는 Faster Action과 Time Saving과 겹치는 부분이 많지만, '빠른 수정에 의한 비용 절감'을 그만큼 강조하고 싶은 것으로 보임.

### Faster Action

- 사람이 기계 만큼 빠르지 않다는 이야기.
- failover나 traffic switching 등과 같은 작업이 한 예.
- 구글에서의 서비스들은 이미 수동 운영으로는 관리 가능한 수준을 벗어남.

### Time Saving

- 가장 많이 언급되는 자동화의 이유.
- 하지만 당장 수작업을 하는 것과 자동화 하는 데 필요한 노력 중 어느 것이 더 큰지 계산하기 어려울 때가 있음.
- 간과하지 말아야 할 것은 한 번 자동화 된 것은 다른 누구도 사용할 수 있다는 것.
- 즉, 자동화의 이점은 다수가 누릴 수 있음.
- 이런 면에서 operator와 operation 사이의 디커플링은 매우 강력.

## The Use Cases for Automation

자동화는 소프트웨어에 대해 동작하는 소프트웨어라는 점에서 일종의 "메타 소프트웨어"라고 말함. 아래는 자동화의 예시들. 참고로, 책에서는 non-exhaustive list라는 얘기를 굳이 하고 있음.

개인적으로 하고 있는 것들은 체크해 봄.

- [ ] User account creation
- [x] Cluster turnup and turndown for services
- [ ] Software or hardware installation preparation and decommissioning
- [x] Rollouts of new software changes
- [x] A special case of runtime config changes: changes to your depedencies

### Google SRE's Use Cases for Automation

- 인프라스트럭처를 오가는 데이터의 품질을 관리하기 보다는,
- 새로운 클러스터에 서비스를 배포하는 등의 시스템 생명 주기를 관리하는 운영적 측면을 자동화 함.
- Puppet, Chef는 상위 수준의 추상화를 제공. Perl은 POSIX-level 수준. 
- 이 둘간의 선택은 트레이드 오프. 추상화 된 도구는 사용하기 쉬움. 하지만 "Leaky Abstraction"에 취약.
- 예컨대, 새로운 바이너리를 클러스터에 추가하는 것을 atomic으로 간주하기 쉬움. 새로운 버전으로 배포 되거나, 옛날 버전이 유지되거나.
- 하지만 실제로는 더 복잡. 클러스터의 네트워크가 한 쪽 방향으로만 실패할 수도 있고, 머신이 실패하거나, 클러스터를 관리 레이어에서의 커뮤니케이션이 실패할 수도 있다. 새로운 바이너리가 스태이징만 되고 푸시되지 않을 수도 있고, 푸시는 되었지만 재시작 되지 않았을 수도, 재시작 되더라도 검증되지 않았을 수도 있다.
- 이런 수준의 결과를 모델링하눈 추상화는 거의 없으며, 이런 경우 대부분 작업이 중단되고 사람의 개입을 필요로 함.

### A Hierarachy of Automation Classes

1. No automation: 마스터 DB를 수동으로 failover.
2. Externally maintained system-specific automation: failover 스크립트를 작성하여 개인이 소유.
3. Externally maintained generic automation: 모두가 사용할 수 있는 "generic failover" 스크립트에 이 failover 기능을 추가.
4. Internally maintained system-specific automation: DB에 이 failover 스크립트를 포함시킴.
5. system that don't need any automation: DB가 직접 문제를 감지하고, 사람의 개입 없이 자동으로 failover를 수행.

## Automate yourself Out of a Job: Automate ALL the Things!

- Google Ads 제품은 데이터의 높은 신뢰성을 필요로 하므로 MySQL을 사용했었음.
- 많은 것들을 자동화 했지만 다음의 2가지를 위해 Borg(Google의 클러스터 스케쥴링 시스템)로 이전하기로.
  - 머신/레플리카 유지보수의 **완전한** 제거. Borg는 새로운 작업의 설정과 깨진 작업의 재시작을 자동으로 수행.
  - 다수의 MySQL바이너리 패킹을 동일한 물리 머신에 두기. Borg는 컨테이너를 활용해서 머신 리소스를 효율적으로 사용함.
- 위 이점을 얻을 수 있었지만 failover가 불가피 했고, 처음에는 수동으로 진행 되었으며, 걸리는 시간도 상당했음.
- 사람의 리소스도 절감하고 에러 예산을 넘지 않기 위해 이 failover를 자동화.
- failover 시간이 95%의 비율로 30초 이내 수행 가능하게 되었지만, 이에 대한 트레이드오프로 실패가 발생하는 것을 허용해야 했음.
- 따라서, JDBC와 같은 MySQL 클라이언트들은 이전보다 더 실패에 대한 내성을 키우는 로직을 필요로 하게 됨.
- 하지만 전체적으로 보면 Borg로의 이전이 더 큰 이득이 됨.

 



















