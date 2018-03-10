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

