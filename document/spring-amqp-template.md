
# AmqpTemplate

https://docs.spring.io/spring-amqp/reference/html/#amqp-template

- 다른 스프링 프레임워크들처럼, 중심 역할을 수행하는 "template"을 제공.
- 주요 연산들을 정의한 인터페이스는 `AmqpTemplate`.
- 주요 연산이라 함은 메시지들을 보내고 받는 일반적 행위들.
- 이름이 AMQP인 만큼 특정 구현체가 아닌 AMQP 프로토콜에 대응.
- 이 구현체 중 하나가(그리고 현재까지 하나뿐인) `RabbitTemplate`.

## Adding Retry Capabilities

- 브로커 연결 문제 등을 다루는데 `RetryTemplate`이 도움.
- 이 역시 [spring-retry](https://github.com/spring-projects/spring-retry)에 기반.

```java
@Bean
public RabbitTemplate rabbitTemplate() {
    var template = new RabbitTemplate(connectionFactory());
    var retryTemplate = new RetryTemplate();

    var backOffPolicy = new ExponentialBackOffPolicy();
    backOffPolicy.setInitialInterval(500);
    backOffPolicy.setMultiplier(10.0);
    backOffPolicy.setMaxInterval(10000);

    retryTemplate.setBackOffPolicy(backOffPolicy);
    template.setRetryTemplate(retryTemplate);
    return template;
}
```

- 1.4 버전부터는 `retryTemplate` 프로티에 더해 `recoveryCallback`도 지원.
- `RetryTemplate.execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback)`.

```java
retryTemplate.execute(
    new RetryCallback<Object, Exception>() {

        @Override
        public Object doWithRetry(RetryContext context) throws Exception {
            context.setAttribute("message", message);
            return rabbitTemplate.convertAndSend(exchange, routingKey, message);
        }
    },
    new RecoveryCallback<Object>() {

        @Override
        public Object recover(RetryContext context) throws Exception {
            Object message = context.getAttribute("message");
            Throwable t = context.getLastThrowable();
            // ...
            return null;
        }
    }
)
```
