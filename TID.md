# 08/29

## NewRelic Custom Instrumentation
- NewRelic이 지원하는 프레임워크가 아닌 경우에는 [Custom Instrumentation](https://docs.newrelic.com/docs/agents/manage-apm-agents/agent-data/custom-instrumentation)이 필요함.
- 사용중인 프레임워크는 [Spring Zuul](https://github.com/Netflix/zuul) 이었으며, 이 때 [ZuulProperties](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/zuul/filters/ZuulProperties.java)가 코딩량 줄이기에 도움이 됨.

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
