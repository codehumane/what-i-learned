
# AmqpTemplate

https://docs.spring.io/spring-amqp/reference/html/#amqp-template

- 다른 스프링 프레임워크들처럼, 중심 역할을 수행하는 "template"을 제공.
- 주요 연산들을 정의한 인터페이스는 `AmqpTemplate`.
- 주요 연산이라 함은 메시지들을 보내고 받는 일반적 행위들.
- 이름이 AMQP인 만큼 특정 구현체가 아닌 AMQP 프로토콜에 대응.
- 이 구현체 중 하나가(그리고 현재까지 하나뿐인) `RabbitTemplate`.
