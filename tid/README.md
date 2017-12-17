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

# 09/26

## API Endpoint Design

- API 설계에 있어서도 [Uniform Access Design](https://martinfowler.com/bliki/UniformAccessPrinciple.html)은  마찬가지 아닐까.

# 09/27

## Code Smell (Vulnerability)

- 아래 코드는 여러 가지 문제점을 가짐.
- 그 중의 한 가지는, 주로 값객체로 사용되는 mutable 객체를 그대로 반환한다는 것.

```
@Data
public class Hello {
  private Date date;
}
```

## Gradle FindBugs

- [Gradle의 FindBugsExtension 문서](https://docs.gradle.org/4.1/dsl/org.gradle.api.plugins.quality.FindBugsExtension.html)가 도움이 됨.

# 09/28

## try-with-resource

- 이미 예전부터 제공되던 기능
- [The try-with-resource Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)

```java
final FileOutputStream outputStream = new FileOutputStream(file);
try {
  workbook.write(outputStream);
} finally {
  outputStream.close();
}
```

```java
try (FileOutputStream outputStream = new FileOutputStream(file)) {
  workbook.write(outputStream);
}
```

- Java9에서는 더 나아진다(고 생각됨).

```
try {
  workbook.write(outputStream);
}
```

# 10/05

## Maven BOM

- Maven을 안 쓴지 꽤 되었지만, BOM(Bill Of Material)에 대해서 이제야 알게 됨.
- [Introduction to the Depedency Mechanism 문서](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)의 ["Importing Depdencies"]() 항목에 잘 설명됨.
- [spring-boot-starter-parent](https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-parent/1.2.5.RELEASE/spring-boot-starter-parent-1.2.5.RELEASE.pom)는 일종의 BOM.
- gradle에서는 `spring-boot` 플러그인으로 이런 역할을 수행. [여기](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-gradle-plugin.html#build-tool-plugins-gradle-dependency-management)를 참고.


# 10/06

## Spring Context

- [@Lazy](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Lazy.html)라는 녀석이 존재함. 얼마나 유용할지는 모르겠음.

## Gradle Multi Module Project

- 오랜만에 다시 한 번 Multi Module 관련 문서를 읽음.
- [Gradle User Guide > Multi Project Build](https://docs.gradle.org/current/userguide/multi_project_builds.html)
- [Srping Getting Started > Multi Module](https://spring.io/guides/gs/multi-module/)


# 10/12

## Spring ObjectProvider

- Spring에서 제공되는 [ObjectProvider](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/ObjectProvider.html)라는 것이 있음.
- `getIfAvailable`이나 `getIfUnique` 등의 편의 기능들이 제공됨. 코딩량을 꽤나 줄이는데 도움이 됨.
- 아주 오래된 기능인 줄 알았으나, [Spring Framework 4.3이 되어서야 도입된 것이라고 함](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3).
- [Implicit constructor injection for single-constructor scenarios](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3#implicit-constructor-injection-for-single-constructor-scenarios)도 이때 생긴 것.

## Spring Database Popluation

- [ResourceDatabasePopulation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/init/ResourceDatabasePopulator.html)과 [DatabasePopulatorUtils](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/datasource/init/DatabasePopulatorUtils.html)를 이용하면, 좀 더 자유로운 DB 준비 작업(DDL, DML 등)이 가능함.
- Auto Configuration의 하나인 [DataSourceInitializer](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/datasource/init/DataSourceInitializer.html)는 좋은 예시가 됨.

## Meta-Annotation Support for Testing

- 중복된 테스팅 환경 설정 선언들을, 커스텀 애노테이션을 생성하여 재사용 할 수 있음.
- 반드시 인과관계가 있는 것도 아니고 부산물이긴 하지만, 테스트 컨텍스트 캐시에도 유리해 보임.
- 내용은 [여기](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#integration-testing-annotations-meta)를 참고.


# 10/15

## Complex Number

- 켤레 복소수, 복소수의 절대값, 복소평면(가우스 평면)의 개념.
- 복소수의 극형식(극좌표 표시) `z = r(cosθ + i·sinθ)`
- 편각 `θ = arg(z)`
- 복소수의 곱과 나눗셈. 극형식과 [삼각함수의 덧셈정리](https://ko.wikipedia.org/wiki/%EC%82%BC%EA%B0%81%ED%95%A8%EC%88%98%EC%9D%98_%EB%8D%A7%EC%85%88%EC%A0%95%EB%A6%AC)를 활용.
- 예컨대, i의 극형식은 `cos(\frac{\pi}{2}) + i \cdot sin(\frac{\pi}{2})`이므로,
- '복소수 z에 i를 곱한다'는 것은 90˚ 회전시킨다는 것과 같음.
- [드무아브르의 공식](https://ko.wikipedia.org/wiki/%EB%93%9C%EB%AC%B4%EC%95%84%EB%B8%8C%EB%A5%B4%EC%9D%98_%EA%B3%B5%EC%8B%9D)은 수학적 귀납법을 이용.
- 이 때, `cos2θ = cos^2θ - sin^2θ`와 `sin2θ = 2sinθcosθ`  성질을 이용함.

# 10/22

## Java Set Interface Bulk Operations

- [Set Interface](https://docs.oracle.com/javase/7/docs/api/java/util/Set.html)를 살펴보니, 교집합, 부분집합, 합집합, 여집합에 대한 처리가 잘 되어 있음.
- [Oracle Tutorior](https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html)에서도 이들이 집합 대수<sup>set-algebraic</sup> 연산을 잘 수행한다고 명시되어 있음.
- 처음에 사용하려 했던 비트 연산보다, 여러모로 좋은 선택으로 보임.

| Set-algebraic Operation | Set Interface                            |
| ----------------------- | ---------------------------------------- |
| subset                  | [containsAll](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html#containsAll-java.util.Collection-) |
| union                   | [addAll](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html#addAll-java.util.Collection-) |
| intersection            | [retainAll](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html#retainAll-java.util.Collection-) |
| complement              | [removeAll](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html#removeAll-java.util.Collection-) |
| cartesian product       | 없는 것 같다.                                 |

## Java HTTP Connection Close

- HTTP 응답을 처리하는 데 있어, `close`가 명시적으로 필요할 수 있음.
- 예컨대, [Ribbon RestClient의 618라인](https://github.com/Netflix/ribbon/blob/master/ribbon-httpclient/src/main/java/com/netflix/niws/client/http/RestClient.java#L618)에서는,
- 결국 아래의 [Jersey WebResource](http://javadox.com/com.sun.jersey/jersey-bundle/1.18/com/sun/jersey/api/client/WebResource.html) 코드를 호출하게 되는데,
- 이 때 class type이 `ClientResponse`이므로 `close`되지 않고 그대로 반환됨.

```java
private <T> T handle(Class<T> c, ClientRequest ro) throws UniformInterfaceException, ClientHandlerException {
    setProperties(ro);
    ClientResponse r = getHeadHandler().handle(ro);
    if (c == ClientResponse.class) return c.cast(r);
    if (r.getStatus() < 300) return r.getEntity(c);
    throw new UniformInterfaceException(r,
            ro.getPropertyAsFeature(ClientConfig.PROPERTY_BUFFER_RESPONSE_ENTITY_ON_EXCEPTION, true));
}
```

- 참고로, 위 코드에서 `r.getEntity(c)` 내부에서는 알아서 `close`가 호출됨.
- Jersey의 `ClientResponse` 코드를 보다보면, `Closeable` 녀석들은 close가 필요하다고 함.
- 다른 문서나 코드에서도 비슷한 내용들을 찾을 수 있음.

> ```
> If the entity is not an instance of Closeable then this response
> * is closed (you cannot read it more than once, any subsequent
> * call will produce {@link ClientHandlerException}).
> ```

# 10/31

TID인데, 1주일 간격으로 작성 중. 반성.

## Collecting Parameter

-  함수 파라미터로 객체를 넘겨주고, 함수는 이 객체에게 질의도 하고 값 설정도 하게 만듦.
-  효율성을 이유로 이렇게 만들었으나, 아무래도 다른 사람이 보기에 좀 더 쉽게 작성할 수는 없을지 고민하게 됨.
-  마침 "[Move accumulation to Collecting Parameter](https://www.industriallogic.com/xp/refactoring/accumulationToCollection.html)"가 생각남.
-  [리팩토링](http://www.yes24.co.kr/24/goods/267290) 책에 나왔던 것 같음. 출처는 확인 필요.
-  옆 동료와 한참을 이야기 함. 그리고 파라미터로 넘어온 객체의 상태를 바꾸는 것을 허용하기로 함.
-  참고로 [여기](http://wiki.c2.com/?CollectingParameter)서는 이것이 일반적으로는 안티 패턴이라고 소개함.

>  A variation on this pattern occurs when the [CollectingParameter](http://wiki.c2.com/?CollectingParameter) is not a collection, but is an object with various properties. As the object is passed around, various actors get and set properties on the [ParameterObject](http://wiki.c2.com/?ParameterObject). I think this version of the pattern is generally an [AntiPattern](http://wiki.c2.com/?AntiPattern), but may be useful when refactoring from the use of globals.

## The Expert Beginner

-  [How Developers Stop Learning: Rise of the Expert Beginner](https://www.daedtech.com/how-developers-stop-learning-rise-of-the-expert-beginner/)에 대한 [번역글](https://medium.com/@jwyeom63/%EB%8D%94-%EC%9D%B4%EC%83%81-%EB%B0%B0%EC%9A%B0%EB%A0%A4-%ED%95%98%EC%A7%80-%EC%95%8A%EB%8A%94-%EA%B0%9C%EB%B0%9C%EC%9E%90-expert-beginner%EC%9D%98-%EB%93%B1%EC%9E%A5-dd40)을 보게 됨.
-  아래 그림이 인상 깊어서 기록함.

![novice-to-expert](novice-to-expert.jpeg)

# 11/01

## Java Access Modifier

| Modifier    | Class | Package | Subclass | World |
| ----------- | ----- | ------- | -------- | ----- |
| public      | Y     | Y       | Y        | Y     |
| protected   | Y     | Y       | Y        | N     |
| no modifier | Y     | Y       | N        | N     |
| private     | Y     | N       | N        | N     |

-  출처는 오라클의 [Controlling Access to Members of a Class](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html) 문서.
-  default 접근자를 진짜 많이 사용하는데, 참 모르고 (혹은 까먹고) 쓰고 있었구나를 느낌.
-  `protecetd`가 `no modifier`보다 더 개방적.

## Java NavigableMap

-  https://docs.oracle.com/javase/7/docs/api/java/util/NavigableMap.html
-  [SortedMap](https://docs.oracle.com/javase/7/docs/api/java/util/SortedMap.html)의 확장 인터페이스이며, [navigableKeySet](https://docs.oracle.com/javase/7/docs/api/java/util/NavigableMap.html#navigableKeySet()) 등의 편의 API들을 상당수 제공함.

## API Endpoint Versioning

-  1개의 서비스에서 20개의 API 엔드포인트를 제공한다고 가정.
-  이 엔드포인트들은 여러 서비스에서 사용됨.
-  그런데, 특정 엔드포인트가 변경되야 한다고 하자.
-  이 엔드포인트의 버전만 올려야 할까, 아니면 전체 엔드포인트의 버전을 함께 올려야 할까?
-  혹은, 얼마만큼의 변경이 있을때 버전을 올리는가?
-  그러니까, 버전이 뭔가? 왜 하는가?
-  많은 접근법들이 있구나 알게 됨.

## API GW, Shared Client Library

-  [Mastering Chaos, A Netflix Guide to Microservices](https://www.youtube.com/watch?v=OczG5FQIcXw&feature=youtu.be)
-  위 영상의 20분 즈음에는 GW가 또 다른 레거시가 되는 이야기가 나옴.
-  물론, GW 자체의 문제라기보다 공유 라이브러리에 의한 것. 재미있음.
-  여러 서비스 간의 공유 라이브러리에 대한 실제 문제 사례가 이런저런 생각을 하게 만듦.

# 11/02

## Divide-and-Conquer

-  짧지 않은 시간 동안 레거시를 몇 개의 서비스로 나누어 감. 주요했던 교훈은, 배포의 단위를 얼마나 잘게 쪼갤 수 있느냐.
-  다시 말하면, 얼마나 리스크를 분산시킬 수 있느냐. 잘못된 상태로 너무 많이 나아가지 않을 수 있는가(혹은 다시 돌아올 수 있는가).
-  일종의 분할 정복<sup>Divide-and-Conquer</sup>? 돌다리?
-  마차의 바퀴를 몽땅 갈아치우고 싶은 욕심에, 달리는 마차를 멈출 수는 없는 노릇.
-  그런 의미에서 [Strangler Application Pattern](https://www.martinfowler.com/bliki/StranglerApplication.html)은 단순하지만 강력한 접근법. ([Big-Bang](https://en.wikipedia.org/wiki/Big_bang_adoption)은 노노)
-  [Feature Toggle](https://martinfowler.com/articles/feature-toggles.html)이나, 데이터 이중화 등이 도움이 됨.
-  하지만 차후 사용되지 Feature들의 정리는 또다른 부담. 역시나 좋은 습관의 중요성.

>  "Leave the campground cleaner than you found it." - Clean Code

## Value

-  이런 저런 이유로 떠올랐던 오늘의 문장. 출처는 [이 곳](http://pragmaticstory.com/attachment/cfile25.uf@1654D0124BCFBE0C2F6182.pdf).

>  "수술은 성공했습니다. 하지만 환자는 죽었습니다. 무엇이 성공한 것입니까"

# 11/03

## Depedencies - Ordering, Filtering

사용자가 상품들을 장바구니에 담았음. 그리고 이제 배송일자를 골라야 함. 이 때, 각종 정책에 의해 불가능한 날짜를 제거하고, 가능한 날짜들만 사용자에게 알려줘야 함.

그래서 만든 것들 중 주요한 것은 아래 2개

- `DeliveryDateChecker` : 불가능한 날짜 제거하는 인터페이스. 그리고 이들의 구현체가 10개 정도 존재.
- `DeliveryCalendarService` : 배송 가능한 날짜들로 캘린더 만들어 주는 녀석. 구현은 아래와 같음.

```java
@Serivce
class DeliveryCalendarService {

  private final List<DeliveryDateChecker> checkers;

  DevlieryCalendarService(List<DeliveryDateChecker> checkers) {
    this.checkers = checkers;
  }
  
  DeliveryCalendar get(Set<Product> products, LocalDate startDate) {
    return checkers
      .stream()
      .map(c -> c.check(products, startDate))
      .reduce(toCalendar());
  }
}
```

그런데, `OrderConfirmationService` 라는 녀석을 만들어야 함. 사용자가 선택한 날짜들로, 다시 한 번 배송 가능한 지 확인하는 녀석. 그런데, `DeliveryCalendarService`와 "**의존성**" 관련 요구사항이 조금 다름.

1. 모든 `DeliveryAvailabilityChecker` 구현체가 아닌, 2개의 구현체만 필요함.
2. `DeliveryAvailabilityChecker`를 호출하는 순서가 달라야 함.

조금 더 풀어서 설명하면, 다음과 같다.

> `DeliveryCalendarService`에서는 `DeliveryAvailabilityChecker`의 구현체인 A, B, C, D를 차례로 호출한 반면, `OrderConfirmationService`에서는 D, C를 차례로 호출해야 함.

여러 방법들이 있음. 그러나, 어떻게 하는 게 가장 좋을까?

# 11/07

-  [I interviewed at five top companies in Silicon Valley in five days, and luckily got five job offers](http://www.looah.com/article/view/2070)
-  위 글에서 아래 두 문장이 마음 깊은 곳을 찌름.
-  꿈보다 해몽인가. 참으로 평범한 문장들임에도 불구하고 말이다.

>  Understand the requirements first, then lay out the high-level design, and finally drill down to the implementation details. Don’t jump to the details right away without figuring out what the requirements are.

>  There are no perfect system designs. Make the right trade-off for what is needed.

# 11/08

-  [통계적 유의성](https://ko.wikipedia.org/wiki/%ED%86%B5%EA%B3%84%EC%A0%81_%EC%9C%A0%EC%9D%98%EC%84%B1)의 표준은 관행상 p값([유의 수준](https://ko.wikipedia.org/wiki/%EC%9C%A0%EC%9D%98_%EC%88%98%EC%A4%80))을 0.05로 잡는다고 함.
-  [틀리지 않는 법 (수학적 사고의 힘)](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=74921141)에서는 이를 비판하는 내용이 상당히 많이 나옴.
-  아래는 책에서 인용된 [xkcd](https://xkcd.com/882/)의 한 그림.

![green-jelly-beans-linked-to-acne](green-jelly-beans-linked-to-acne.png)

# 11/11

## Merge Sort

- 오랜만에 작성해 본 [병합정렬](https://en.wikipedia.org/wiki/Merge_sort).
- 생각보다 오래 걸렸고, 시간 복잡도도 높게 나옴.
- 약간의 공간복잡도를 포기하면 될 것을, 무의식적으로 모 아니면 도라고 생각한 듯.
- 잘못된 구현은 [여기](https://github.com/codehumane/learn-algorithm-in-java/commit/5db9aacaf333996c1379a0673ba23c815f689065)에 기록함.
- 약간의 공간 복잡도 포기한, 하지만 시간 복잡도는 O(n · ln(n)) 혹은 Θ(n · ln(n))을 달성한 구현. 여기에 병렬 처리까지 추가한 코드는 [이 곳](https://github.com/codehumane/learn-algorithm-in-java/commit/b498cac6b324a25d6707bb94035a541ddb5682f6)을 참고.

## Merge Sort Time Complexity

- 병합정렬의 내부는 분할, 정복, 병합으로 이루어짐.
- 여기서 분할, 병합의 실행 시간은 n + 1이며, cn으로 표기 가능함(c는 상수).
- 그리고 아래와 같이 트리 그림으로 표현하면, 모든 트리의 깊이에서 n의 실행시간을 가진다는 것을 알 수 있음.
- 자꾸 잊어버려서, [블로그에 기록](http://codehumane.github.io/2017/11/11/%EB%B3%91%ED%95%A9%EC%A0%95%EB%A0%AC-%EC%8B%9C%EA%B0%84%EB%B3%B5%EC%9E%A1%EB%8F%84-%EA%B5%AC%ED%95%98%EA%B8%B0/)함.

![merge-sort-complexity-tree](merge-sort-complexity-tree.png)

# 11/12

## Median Find

- 오늘은 [중위수 찾기 알고리즘을 구현함](https://github.com/codehumane/learn-algorithm-in-java/commit/fba07a0bedac08998ee62d77885ce7ddfdd249f8).
- 쉽게 생각하면, 병합 정렬로 숫자를 정렬한 뒤, 가운데 값을 추출하면 됨.
- 이 때의 시간 복잡도는 당연히 O(n · ln(n))가 됨.
- 하지만, 선택<sup>selection</sup>을 사용하면 좀 더 쉽고 빠르다.
- [여기](https://brilliant.org/wiki/median-finding-algorithm/)에 설명이 잘 나와 있다고 생각함.
- 수행 시간에 대한 기댓값은 T(n) ≤ T(3n/4) + O(n).
- 따라서, 평균적으로 O(n)의 수행시간을 가짐.

# 11/13

## Spring Cloud Stream & AWS

-  스프링에서 [SQS](https://aws.amazon.com/ko/sqs/), [SNS](https://aws.amazon.com/ko/sns/)를 사용할 일이 있어 [Spring Cloud Stream](https://cloud.spring.io/spring-cloud-stream/)을 떠올렸으나,
-  [Binder Implementation](https://github.com/spring-cloud/spring-cloud-stream#binder-implementations)을 살펴보니 아직까지는 [Kafaka](https://github.com/spring-cloud/spring-cloud-stream-binder-kafka)와 [RabbitMQ](https://github.com/spring-cloud/spring-cloud-stream-binder-rabbit)만을 지원함.
-  대신, [Spring Cloud AWS](https://cloud.spring.io/spring-cloud-aws/)가 존재함.
-  SQS, ElastiCache, SNS, CloudFormation, RDS, S3를 위한 편의 기능들을 제공.
-  레퍼런스는 [여기](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/)를 참고.

## When the Optimizer Considers a Full Table Scan

-  index가 걸려 있는 컬럼이라고 하더라도, SELECT 범위가 크다면 Sequential(Full) Scan이 발생할 수 있음.
-  [Full table scan](https://en.wikipedia.org/wiki/Full_table_scan), [Why does PostgreSQL perform sequential scan on indexed column?](https://stackoverflow.com/questions/5203755/why-does-postgresql-perform-sequential-scan-on-indexed-column) 문서 참고.

# 11/14

## Spring Cloud AWS Messaging

>  ... enables developers to receive and send messages with the [Simple Queueing Service](https://aws.amazon.com/sqs/) for point-to-point communication. Publish-subscribe messaging is supported with the integration of the [Simple Notification Service](https://aws.amazon.com/sns/).

\- [Spring Cloud AWS Reference](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/)

## AWS SQS, SNS

-  [SQS](https://aws.amazon.com/ko/sqs/)는 대기열 서비스. 메시지 전달 안정성(fault tolerant). 2가지 대기열 (표준, FIFO)
-  [SNS](https://aws.amazon.com/sns/?nc1=h_ls)는 [토픽](http://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html) 사용. 발신자 수신자 간의 결합도 낮춤. [Fanout](https://en.wikipedia.org/wiki/Fan-out_(software)). 폴링 처리 제거. ([Pub/Sub Messaging Basics](https://aws.amazon.com/pub-sub-messaging/))
-  [Common Amazon SNS Scenarios](http://docs.aws.amazon.com/sns/latest/dg/SNS_Scenarios.html)에 보면 Fanout을 개발 환경에서 프로덕션 환경의 데이터를 받는 용도로도 사용함.
-  [What is the difference between Amazon SNS and Amazon SQS?](https://stackoverflow.com/questions/13681213/what-is-the-difference-between-amazon-sns-and-amazon-sqs)

| SQS                              | SNS                                   |
| -------------------------------- | ------------------------------------- |
| queueing system                  | pub/sub system (topic)                |
| polling (+ process, delete)      | push                                  |
| some latency                     | send time-critical (immediate)        |
| point-to-point (one receiver)    | fanout (multiple reciever)            |
| concurrency decoupling (persist) | publisher decoupling from subscribers |

-  [SQS Queues and SNS Notification - Now Best Friends](https://aws.amazon.com/blogs/aws/queues-and-notifications-now-best-friends/) 문서에서는 이 둘을 "glue" 컴포넌트라고 표현.

## Deployment

-  Snowflake vs. [Phoenix Server](https://www.thoughtworks.com/insights/blog/moving-to-phoenix-server-pattern-introduction), [Immutable Server](https://martinfowler.com/bliki/ImmutableServer.html)
-  [Blue/Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html), [Canary Release](https://martinfowler.com/bliki/CanaryRelease.html), [A/B Testing](https://en.wikipedia.org/wiki/A/B_testing)

## AWS SDK Credentials Configuration

-  [Simple credentials configuration](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_simple_credentials_configuration)
-  [Instance profile configuration](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_instance_profile_configuration) with [Using Instance Profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
-  [결합된 방식](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_mixing_both_security_configurations)을 권장하기도.


# 11/15

##  AWS Instance Metadata

-  [EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
-  [Instance Metadata Categories](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories)

### Spring Cloud AWS Region Configuration

-  [Automatic Region Configuration](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_automatic_region_configuration)
-  EC2 instance에서는 Automatic Configuration을 **사용할 수 있다**가 아니라, **사용해야 한다**구나.

>  "If the application context is started inside an EC2 instance, then the region can automatically be fetched from the [instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) and therefore must not be configured statically."

## Spring Cloud Instance Metadata Retrieve

-  [Retrieving Instance Metadata](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_retrieving_instance_metadata)
-  property placeholder 등을 통한 인스턴스 메타데이터 접근 가능. HTTP Service 호출 X
-  EC2에서 동작하는 부트의 경우에는 자동으로 사용 가능함.
-  [Tags](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_using_instance_tags), [User Data](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_using_instance_user_data)에도 접근 가능.

## Spring Cloud Messaging

- SQS의 몇 가지 제약사항. [SQS Support](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_sqs_support) 참고.
  - String payload only.
  - No transaction support. 메시지는 두 번 폴링 될 수 있음.
  - 256kb 크기 제한.
- SQS에 대한 [Message Reply](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.2.1.RELEASE/#_message_reply) 기능도 제공.

```java
@SqsListener("treeQueue")
@SendTo("leafsQueue")
public List<Leaf> extractLeafs(Tree tree) {
	// ...
}
```

# 11/16

- [새빨간 거짓말, 통계 (책)](https://github.com/codehumane/what-i-learned/blob/master/how-to-lie-with-statistics/README.md)
- [언제나 의심스러운 여론조사](https://github.com/codehumane/what-i-learned/tree/master/how-to-lie-with-statistics#언제나-의심스러운-여론조사)
- [평균은 하나가 아니다](https://github.com/codehumane/what-i-learned/tree/master/how-to-lie-with-statistics#평균은-하나가-아니다)
- [통계도 논리이다](https://github.com/codehumane/what-i-learned/tree/master/how-to-lie-with-statistics#숫자로-장난치기)
- [백분율 더하기](https://github.com/codehumane/what-i-learned/tree/master/how-to-lie-with-statistics#백분율-더하기)
- [백분율, 백분율점, 백분위수](https://github.com/codehumane/what-i-learned/tree/master/how-to-lie-with-statistics#백분율-백분율점-백분위수)


# 11/20

## Workflow Subscriptions & Event Broker Terminology

[여기](https://docs.vmware.com/en/vRealize-Automation/7.0/com.vmware.vra.extensibility.doc/GUID-594E5327-E3B4-467A-AFB8-839450798161.html)서 설명하는 토픽과 이벤트의 관계, 그리고 메시지/페이로드/이벤트의 구분이 기억에 남음.

| 용어          | 설명                                       |
| ----------- | ---------------------------------------- |
| Event Topic | 이벤트 집합. 모든 이벤트는 이벤트 토픽의 한 인스턴스.          |
| Event       | 상태의 변화. 이벤트 발생에 대한 정보를 기록하고 있는 엔티티.      |
| Payload     | 이벤트 데이터.                                 |
| Message     | 관련자들끼리 주고받는, 이벤트에 대한 운송 정보.              |
| 그 외         | Event Broker Service, Subscription, Subscriber, Provider, Producer |

## AWS SNS Protocol - Application

-  [SNS Topic Subscription](http://docs.aws.amazon.com/ko_kr/sns/latest/dg/SubscribeTopic.html)의 가능한 프로토콜 중에 `Application`이라는 것이 있음.
-  이게 뭔가 했는데, [여기](https://stackoverflow.com/questions/33488224/what-is-application-protocol-of-sns-subscription)를 보니, 모바일 앱과 디바이스를 위한 것.

>  delivery of JSON-encoded message to an EndpointArn for a mobile app and device.

## AWS SNS Retry Policy

-  http://docs.aws.amazon.com/ko_kr/sns/latest/dg/DeliveryPolicies.html
-  다른 프로토콜과 다르게, HTTP와 HTTPS는 전송 정책을 제공함.
-  먼저, 전송 실패에 대한 기본적인 정의가 있음.
-  재시도는 최대 100회, 최대 1시간 이내에만 허용됨.
-  단계는 총 4가지
   -  Retries with no delay, Pre-Backoff Phase, Backoff Phase, Post-Backoff Phase
-  최대 수신 속도 또한 지정 가능함.

## Spring Composite Injection

-  Bean 주입에 Composite을 적용.
-  [Spring: Injecting Lists, Maps, Optionals and getBeansOfType() Pitfalls](https://dzone.com/articles/spring-injecting-lists-maps) 참고함.

```java
@Component
@Primary
public class CompositeRunner implements Runnable {
  
  private final List<Runnable> runnables;  
  CompositeRunner(List<Runnable> runnables) {
    this.runnables = runnables;
  }
 
  @Override
  public void run() {
    return runnables
      .forEach(Runnable::run);
    }
}
```

## Articles

-  [내 멋대로 구현한 이벤트 드리븐](http://www.popit.kr/%EB%82%B4-%EB%A9%8B%EB%8C%80%EB%A1%9C-%EA%B5%AC%ED%98%84%ED%95%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%93%9C%EB%A6%AC%EB%B8%90/)
-  [[MySQL] 인덱스 정리 및 팁](http://jojoldu.tistory.com/243)


# 11/23

## Book

- Spring Microservices
  - [Autoscaling Microservices](https://github.com/codehumane/what-i-learned/tree/master/spring-ms#autoscaling-microservices)
  - [Spring Microservices with Spring Cloud](https://github.com/codehumane/what-i-learned/tree/master/spring-ms#scaling-microservices-with-spring-cloud-1)
  - [Understanding the concept of autoscaling](https://github.com/codehumane/what-i-learned/tree/master/spring-ms#understanding-the-concept-of-autoscaling)
  - [Autoscaling Approaches](https://github.com/codehumane/what-i-learned/tree/master/spring-ms#autoscaling-approaches)
- 도메인 주도 설계 구현
  - [11장. 팩토리](https://github.com/codehumane/what-i-learned/tree/master/dddi#11장-팩토리)


# 11/25

## Book

- 도메인 주도 설계 구현
  - [도메인 이벤트](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#8장-도메인-이벤트)
  - [언제 그리고 왜 도메인 이벤트를 사용할까](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#언제-그리고-왜-도메인-이벤트를-사용할까)
  - [이벤트 모델링](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#이벤트-모델링)
  - [도메인 모델에서 이벤트를 발행하기](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#도메인-모델에서-이벤트를-발행하기)
  - [원격 바운디드 컨텍스트로 이벤트 전파하기](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#뉴스를-원격-바운디드-컨텍스트로-전파하기)
  - [이벤트 저장소](https://github.com/codehumane/what-i-learned/blob/master/dddi/README.md#이벤트-저장소)

# 11/28

## AWS SNS, SQS

- AWS JDK로 큐 메시지 수신하는 방법에 대해서는 [여기](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/examples-sqs-messages.html) 참고.
- Spring의 [@SqsListener](https://github.com/spring-cloud/spring-cloud-aws/blob/master/spring-cloud-aws-messaging/src/main/java/org/springframework/cloud/aws/messaging/listener/annotation/SqsListener.java)로 수신하는 방법에 대해서는 [여기](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.1.0.RELEASE/#_annotation_driven_listener_endpoints) 참고.
- 이 녀석의 동작 방식을 이해하는 데는 [SimpleMessageListenerContainer](https://github.com/spring-cloud/spring-cloud-aws/blob/master/spring-cloud-aws-messaging/src/main/java/org/springframework/cloud/aws/messaging/listener/SimpleMessageListenerContainer.java), [SimpleMessageListenerContainerFactory](https://github.com/spring-cloud/spring-cloud-aws/blob/master/spring-cloud-aws-messaging/src/main/java/org/springframework/cloud/aws/messaging/config/SimpleMessageListenerContainerFactory.java) 코드와 [Spring Cloud AWS 문서](http://cloud.spring.io/spring-cloud-static/spring-cloud-aws/1.1.0.RELEASE/#_annotation_driven_listener_endpoints)가 도움이 됨. 한 예로, 아래의 문장.

> By default the `SimpleMessageListenerContainer` creates a `ThreadPoolTaskExecutor` with computed values for the core and max pool sizes. The core pool size is set to twice the number of queues and the max pool size is obtained by multiplying the number of queues by the value of the `maxNumberOfMessages` field. If these default values do not meet the need of the application, a custom task executor can be set with the `task-executor` attribute (see example below).

- SNS에서 SQS로 전송 시, 메시지 유실 가능성은 존재하는 것으로 보임. [여기](https://stackoverflow.com/questions/30750033/amazon-sns-delivery-retry-policies-for-sqs) 참고.
- [SNS Monitor](http://docs.aws.amazon.com/ko_kr/sns/latest/dg/MonitorSNSwithCloudWatch.html) 등을 통한 유실 모니터링 필요.

# 11/30

## AWS SNS, SQS Message Loss

SNS, SQS로 메시징 인프라 구축 시, 메시지가 유실될 수 있는 구간은 총 3개.
1. Application to SNS
2. SNS to SQS
3. SQS to Application

가장 쉽게 떠올랐고 일단 실행하고 있는 유실 대응 방안은 다음과 같음.

1. 일단, 1번 구간은 애플리케이션이 보장.
   - SNS 호출 실패 시 트랜잭션 전체를 롤백하거나,
   - retry를 위해 Queue를 이용해서 잠시 후 다시 시도 등
2. SNS로 전달된 모든 메시지는 영속화.
3. 2번 구간과 3번 구간은 메시지 유실을 모니터링.
4. 메시지 유실 시, 영속된 메시지를 다시 SNS로 전달(replay).
5. 단, 모든 수신 애플리케이션은 멱등성을 보장.

# 12/01

## JOOQ Record to POJO Mapping

- Type-Safe SQL 작성에 JOOQ를 사용 중.
- JOOQ의 select로 얻어낸 데이터를 POJO(도메인 모델)로 변환해서 사용하는데,
- JOOQ 또는 써드 파티들이 제공하는 것들이 많은 한계를 가짐.
- (가능한 피하는 방식이긴 하지만) 결국 직접 매핑 라이브러리를 작성.
- 잘 사용하고 있었으나, 성능 문제로 Performance Critical 부분에는 적용 X.
- 적용이 필요한 시점이 되어, 결국 성능 개선 작업을 진행. 약 2배 정도의 TPS 향상.
- 배포는 하되 [Feature Toggle](https://martinfowler.com/articles/feature-toggles.html)을 적용. 언제든 기존 알고리즘이 새로운 알고리즘을 대체할 수 있도록 함.
- [관련된 내용은 블로그에 정리](http://codehumane.github.io/2017/12/04/JOOQ-to-POJO-Mapping/).

# 12/04

## Book

- 알고리즘
  - 그래프의 분할(decompositions of graph), [왜 그래프인가](https://github.com/codehumane/what-i-learned/blob/master/algorithm/decompositions-of-graph.md)

# 12/05

## URI, URL, URN

-  매번 헷갈리는 대상. 설명은 [여기](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn) 잘 나와 있음.
-  더불어, URI의 구성요소도 궁금했고, 간단히 아래처럼 코드로 확인.

```java
@Test
public void uri() throws Exception {
    final URI uri = URI.create("https://www.aaa.com:8080/bbb?ccc=ddd#eee");
    log.info(uri.getAuthority()); // www.aaa.com:8080
    log.info(uri.getFragment()); // eee
    log.info(uri.getHost()); // www.aaa.com
    log.info(uri.getPath()); // /bbb
    log.info(uri.getQuery()); // ccc=ddd
    log.info(uri.getScheme()); // https
    log.info(uri.getPort()); // 8080
}
```

# 12/07

## Zuul-Ribbon ReadTimeout

-  오늘 A라는 서비스를 배포함.
-  이 배포에는 새로 추가된 API 엔드포인트가 있었는데, 기존의 것들에 비해 느렸음. 
-  부하 테스트를 통해 인지하고 있었지만, 영향범위 분석해서 감안하고 배포한 것.
-  하지만 API Gateway(Zuul)의 A 서비스에 대한 `ReadTimeout`을 고려하지 못함.
-  이따금 이 임계치를 넘는 요청 지연이 있었고, 이로 인해 `503(Service Unavailable)`이 발생.
-  즉시 대응으로, 새로운 기능에 대해 Feature Toggle을 Off 시킴.
-  그리고 `ReadTimeout` 늘린 후, 다시 Feature Toggle On.
-  향후 대응으로, 배포를 준비하고 있던 성능 개선 작업을 가능한 빠르게 배포할 예정.
-  그럼에도 불구하고 기존 엔드포인트만큼의 성능을 보장하지 못한다면, A 서비스에 대한 Zuul의 논리적 의존성을 분리.
-  이런 저런 것들을 많이 느낀 배포였음.


# 12/10

## Graph Depth-First Search

- [오랜만에 다시 구현](https://github.com/codehumane/learn-algorithm-in-java/commit/584463868529cde41f8001f7096b2aba2a124c80).
- 그리고 기본적인 것 이외에 여러 가지 활용이 있다는 것을 알게 됨.
- 그 활용은 [여기 작성](https://github.com/codehumane/learn-algorithm-in-java/commit/5d495acaa75c0f6baa460d973321f40533141975)한 `previsit`, `postvisit` 처리로 가능함.
- 그리고 [이건](https://github.com/codehumane/learn-algorithm-in-javascript/blob/master/test/algorithm/graph.spec.js) 예전에 작성해 본 JavaScript 버전.
- [유향 그래프<sup>Directed Graph</sup>에 대한 깊이 우선 탐색도 작성](https://github.com/codehumane/learn-algorithm-in-java/commit/940fde15d8b475a99bd8c0de10315bcd2c2038ea)해 봄.

## Java Map Interfaces

- [putIfAbsent](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#putIfAbsent-K-V-), [computeIfAbsent](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#computeIfAbsent-K-java.util.function.Function-), [computeIfPresent](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#computeIfPresent-K-java.util.function.BiFunction-) 인터페이스를 오랜만에 다시 살펴봄.
- 키와 값의 매핑을 `associated`라고 표현하는 것이 재미있음.
- `Map`의 값 초기화+추가에 사용하던 `if`를 제거할 수 있어 좋음.

```java
edges.computeIfAbsent(from, k -> new LinkedHashSet<>()).add(to);
```

- 그리고 아래 내용도 문서를 통해 쉽게 확인이 가능한 거였구나를 알게 됨. Collection 인터페이스가 문서에 굉장히 잘 설명돼 있었던 것을 볼 때와 같은 느낌.

> The default implementation makes no guarantees about synchronization or atomicity properties of this method. Any implementation providing atomicity guarantees must override this method and document its concurrency properties. In particular, all implementations of subinterface [`ConcurrentMap`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html) must document whether the function is applied once atomically only if the value is not present.

# 12/11

## Directed Graph

- [유향 그래프에서의 순환<sup>cycle</sup>이 있는지 여부 감지하기](https://github.com/codehumane/learn-algorithm-in-java/commit/3c5ea76f0aad303a87c61f05263c1bd0e3b867de)
- [유향 비순환 그래프(DAG<sup>Directed Acyclic Graph</sup>)에서의 위상 정렬<sup>topological sort</sup>](https://github.com/codehumane/what-i-learned/blob/master/algorithm/decompositions-of-graph.md#위상-정렬)

# 12/15

> *If* I *have seen* further, it is by standing on the *shoulders* of giants. \- Isaac Newton

## Synchronization

```java
return cache.get(key).orElseGet(() -> {
  val generated = generateMappings();
  cache.put(key, generated);
  return generated;
});
```

- 위와 같은 코드를 작성하고, 리뷰를 받으며 잠시 동시성에 대해 이야기 나눔.
- `cache#get`과 `cache#put` 사이의 코드가 얼마든지 여러 스레드에서 동시에 실행될 수 있음.
- 이로 인해 불필요한 `generateMappings`와 `cache#put`이 발생할 수 있음.
- 지금은 불필요한 수준에서 그치지만, 추후 코드 변경하는 사람이 동시성을 고려 못하면, 버그가 만들어지지 않을까 고민.
- 어디까지를 보호해 줄 것인가 한참 논의. 결론은 보호하지 않기로 함.
- 연산을 캐시(memoize)한다는 것은 연산이 멱등성을 보장한다는 얘기고,
- 멱등성을 더 이상 보장하지 못하는 변경이 아닌 한, 버그 수반 가능성은 낮다고 판단함.
- 혹여나, 멱등성을 더 이상 보장하지 못하는 변경이라고 하더라도, 이는 캐시 사용도 어려운 경우임.
- 따라서, 동시성 문제의 배경이 된 데이터 자체가 사라질 것.
- 부담스러운 `synchronized` 사용도 하지 말고, 코드도 간결하게 유지하기로 함.
- 불필요한 연산이 발생 가능한 것도, 캐싱이 수행되는 최초 한 번에 한해서일 뿐.

# 12/17

## Book

- [[알고리즘] 그래프의 순환 - 유향 그래프의 강한 연결 성분](https://github.com/codehumane/what-i-learned/blob/master/algorithm/decompositions-of-graph.md)

