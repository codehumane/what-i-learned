# 08/29

## NewRelic Custom Instrumentation
- NewRelic이 지원하는 프레임워크가 아닌 경우에는 [Custom Instrumentation](https://docs.newrelic.com/docs/agents/manage-apm-agents/agent-data/custom-instrumentation)이 필요함.
- 사용중인 프레임워크는 [Spring Zuul](https://github.com/Netflix/zuul) 이었으며, 이 때 [ZuulProperties](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/zuul/filters/ZuulProperties.java)가 코딩량 줄이기에 도움이 됨.
- 프레임워크 내부 코드 열어보는 것은 거의 매번 도움이 됨.

## AWS Elastic Beanstalk Leader Instance
- EB 인스턴스 leader 종종 실종. AutoScalingGroup에서 termination policy가 OldestInstance이기 때문임.
- 운영 환경의 특성상 NewestInstance로 바꾸면 해결됨.

# 08/30

## AWS Beanstalk Auto Scaling Trigger Setting
- Scaling Trigger 설정을 통해 불필요한 인스턴스 비용 절약
- 평소 관리하는 성능 테스트 지표가 도움이 됨
- `Measurement period`,  `Breach duration`, `Upper threshold`, `Lower threshold`의 의미와 관계 이해 및 값 변경
- 참고
  - [AWS Developer Forums: Autoscaling Configuration in Elastic ...](https://forums.aws.amazon.com/thread.jspa?threadID=61883)
  - [Configuring Auto Scaling with Elastic Beanstalk - AWS Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.as.html)

## AWS RDS Configuration
- RDS의 높은 CPU Utilization의 원인을 추정함. 실험 후 결과 궁금.
- RDS 설정(`slow_query_log`, `long_query_time`, `log_output option`) 이해. 좀 더 나은 모니터링을 위해 설정 변경 예정. 궁금함.
- `SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 300;`

## Java 8 Optional map Performance
- [forEach](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html)가 특정 경우에 performance 저하(상황에 따라 유의미하기도 함)를 가져왔던 것과 다르게, [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)의 map은 유의미한 차이를 가져오지는 않음. 코드 배포함. 덕분에 간결하고 읽기 쉬움. (나만?)

# 09/02

## Trivial Java Tip
- selection을 통해 median을 찾는 알고리즘 작성하다가 실수한 것들.
- `int ary = new int[3];`을 하면 `ary`의 원소들은 0으로 채워짐.
- `new Random().nextInt()`은 음수가 반환될 수 있음.
- `new Random().nextInt(bound)`로 하면, 0~bound 사이의 값을 반환해 줌. 그래서인지 0 이하를 지정하는 경우 예외를 던지는 코드가 내부에 존재함.

# 09/04

## TDD & PP
- TC를 얼마나 작성해야 하느냐? 사례가 일반화 된 수 만큼. FizzBuzz의 경우는 8개 정도. 그런 면에서, TDD가 도움이 됨.
- FizzBuzz 구현을 TDD로 아주 세밀한 단계까지 나누어 수행. 이 정도까지 나눌 필요는 없지만, 이 정도까지 나눌 수 있다는 게 중요해 보임. 문제의 난이도에 따라 도움이 됨. Baby Step 이라고도 부름. 라이브 코딩 면접 시 안정감을 주기도 함.
- 요구사항이 추가되거나 수정되는 것은 별로 문제가 되지 않음. 이보다는 인터페이스가 바뀌는 경우가 어려움. 예컨대, 문자열이 아닌 다른 자료구조를 반환해야 한다면, TC에서 전체를 바꿔야 함. 한 가지 해결책은 헬퍼 메소드. 의존 라이브러리를 래퍼<sup>wrapper</sup> 객체를 두어 사용하는 느낌. 결국은 의존성을 한 곳으로 모는 것.
- TDD & PP를 하면서 코드를 **연구**한다는 느낌을 받음. 중요 부분에는 TDD & PP를 고려.

# 09/05

## Spring Boot Externalized Configuration
- [@ConfigurationProperties vs. @Value](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-vs-value) 문서에 따르면, `@Value`는 [Relaxed binding](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-relaxed-binding)을 지원하지 않는다고 나옴. 삽질한 적 있음. 주의.
- `@ConfigurationProperties` 사용시에는 주로 `@EnableConfigurationProperties` 함께 사용됨.

## AWS Beanstalk Deploy
- `Failed to deploy application.` 외에 별다른 메시지도 없이 deploy 실패함.
- `/var/log/` 파일들에 특별히 남겨지는 로그도 없음.
- 이전에는 Rebuild Environment를 했었음. 만약, Live 장비라면?
- Auto Scaling Group에서 특정 EC Instance를 제거함과 동시에 투입하는 방법을 사용.

## AWS Beanstalk Auto Scaling Group Termination Policy
- 기본으로 설정된 값은 `Default`임.
- 기존에는 `Default` == `OldestInstance`라고 오해했음.
- `Default`에 대한 동작 방식은 [Controlling Which Instances Auto Scaling Terminates During Scale In - Auto Scaling](http://docs.aws.amazon.com/autoscaling/latest/userguide/as-instance-termination.html) 문서에 잘 나와 있음.

## ETC
- [BFF](http://samnewman.io/patterns/architectural/bff/) 용도로 사용하고 있는 API Gateway의 Aggregate 필요성에 대해 다시 한번 고민하게 됨. 조심.
- 스프링 부트가 개발자를 바보로 만든다는 비판. 하지만 잘 사용한다면 좋은 측면이 더 많음. 오히려 실수도 줄여주고, 부가적 기능도 설정만으로 간단하게 추가하는 등. Back to the Basic이라는 말은 남용되면 안됨.


# 09/05

## Zuul Service
- [Zuul](https://github.com/Netflix/zuul) 인증 라우팅에 커넥션 타임아웃을 늘리자는 고민을 같이 하다보니,
- Zuul에 명시하는 의존 서비스는 논리적 단위.
- 반드시 물리적 서버(하나의 인스턴스 혹은 인스턴스 그룹)에 대응할 필요 없음.
- 같은 물리적 서버로의 라우팅이지만, 독립적인 정책을 가져가야 한다면, 서비스를 구분.

## ETC
- 단순화. Queue를 사용하지 않고, 동기적 호출에서 발생할 수 있는 문제를 극복하는 것을 봄. 이런 저런 생각을 하게 됨.

# 09/13

## Ribbon (Client Side Load Balancer)

- Discovery를 사용하지 않는다면, [@LoadBalanced](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalanced.java) 보다 [LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java)를 더 선호. 더 명시적이고, 단순함의 수준은 비슷.
- [AbstractRibbonCommand](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/zuul/filters/route/support/AbstractRibbonCommand.java#L90)는 [HystrixCommand](https://github.com/Netflix/Hystrix/blob/master/hystrix-core/src/main/java/com/netflix/hystrix/HystrixCommand.java)의 [HystrixCommandGroupKey](https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommandGroupKey.html) 값으로 "RibbonCommand"를 명시.
- 결국, Ribbon을 통한 Hystrix는 기본적으로 격리 X. 반대로, 스레드 풀 등을 공유해서 사용.
- 한편, Ribbon에서의 [HystrixCommandKey](https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommandKey.html)는 의존 서비스 아이디를 값으로 함.
- 그룹과 커맨드의 단위가 이게 좋은가에 대해서는 의문.

## Java

- [Consumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html), [Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html), [Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html)에 대해 다시 한 번 살펴봄.

## Non-Blocking, Asynchronous

- Blocking/NonBlocking: 호출/응답 사이에 다른 일을 할 수 있는지 여부
- Synchronous/Asynchronous: 호출/응답의 시작/종료 시간이 동일한지 여부
- 참고 자료
    - [Blocking-NonBlocking-Synchronous-Asynchronous](http://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/).
    - [Async & Spring](https://docs.com/toby-lee/6428)


# 09/14

## Git Merge Commit Revert

- `git revert`의 `-n(--no-commit)`: revert 시 자동 수행되는 commit을 하지 않음.
- `git revert`의 `-m(--mainline)`: merge commit revert 시 mainline을 지정함.
- Revert of Revert (참고: [Undoing Merges](https://git-scm.com/blog/2010/03/02/undoing-merges.html))

## Back to the Basic

- 문제 생겼을 때 빠르게 확인할 수 있는 TC 혹은 replay의 필요성에 대해 다시 한번 절실히 느낌.
- 얽히고 섥혀 있는 시스템을 보며, 기본적인 OCP도 지키지 못하는 시스템의 고달픔을 다시 한번 절실히 느낌.
- `getXXX`류의 함수를 열었는데, 내부에서 몰래 값을 할당하고 있는 시스템에 당혹감을 느끼며, SCP가 이렇게 중요하구나를 또다시 체감함.

# 09/15

## Hystrix Thread Timeout

- 다수의 Hystrix 비동기/병렬 호출 시, 별도의 스레드를 사용하는데, 분리된 스레드에서의 타임아웃은?
- Hystrix에 의해 감싸진 녀석이 `InteruptedException`을 해석하지 않으면 멈추지 않음.
- 출처: https://github.com/Netflix/Hystrix/wiki/How-it-Works#6-hystrixobservablecommandconstruct-or-hystrixcommandrun

> Please note that there's no way to force the latent thread to stop work - the  best Hystrix can do on the JVM is to throw it an InterruptedException. If the work wrapped by Hystrix does not respect InterruptedExceptions, the thread in the Hystrix thread pool will continue its work, though the client already received a TimeoutException. This behavior can saturate the Hystrix thread pool, though the load is 'correctly shed'. Most Java HTTP client libraries do not interpret InterruptedExceptions. So make sure to correctly configure connection and read/write timeouts on the HTTP clients.

## RestTemplate

- [RestTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)의 기본 타임아웃은 무려 `0`(무한).
- `setErrorHandler`, `setInterceptor`  등의 인터페이스가 존재함.
- `HttpComponentsClientHttpRequestFactory`를 통해 Pooling, Timeout 등의 설정 가능
- 호스트 별로 여러개의 `RestTemplate` 인스턴스를 두어 사용할수도.


# 09/25

## Endpoint Version Design

- 서비스 엔드포인트의 versioning은 불변 서비스<sup>immutable service</sup>의 의미
- 서비스 내의 단위 기능 수준보다, 서비스 수준에서 버전을 적용하는 것이 언제나 더 간단하다는 이야기에 여러 가지 생각.

## Planning & Analisys

- 하루 종일 플래닝, 설계를 진행. 이것 저것 따져가며 리스크를 찾아내는데, 자꾸만 조금만 더, 조금만 더, 하는 욕구가 생겨남.
- 집에 오는 길에 읽고 있던 <호모데우스, 유발하라리> 내용 중 아래 문구가 눈에 들어옴.

> 당신이 추구하는 어떤 이상이 애초에 결함을 품고 있다면, 대게 그 이상의 실현 단계에 와서야 그러한 결함을 알게 된다.

## Run TID like a Machine

- 성장하기 위해, 단순히 어떤 일을 반복하기보다는 [의도적 수련<sup>deliberate practice</sup>](https://en.wikipedia.org/wiki/Practice_(learning_method)#Deliberate_practice)이 필요.
- (매일 같이 하지는 못하지만, 그리고 이제 막 시작했지만) TID는 스스로 얼마나 나아가는지를 돌아보는 피드백이 됨.
- 아래는 이와 관련하여 떠오른 [Management Priciples, Ray Dailo](http://www.businessinsider.com/ray-dalios-bridgewater-management-principles-2014-11)의 그림과 내용 일부.

> 1) understanding how well your people and designs are operating to achieve your goals, and 2) constantly improving them.

![improvement-machine](improvement-machine.jpg)

