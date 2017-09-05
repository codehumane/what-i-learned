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
