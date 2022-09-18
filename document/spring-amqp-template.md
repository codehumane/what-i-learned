
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

## Publishing is Asynchronous - How to Detect Successes and Failures

- 메시지 퍼블리싱은 비동기 메커니즘.
- 기본적으로 라우팅되지 못한 메시지는 RabbitMQ에 의해 drop 됨.
- 성공적인 퍼블리싱을 위해 비동기 confirm을 받을 수 있음.
- 아래의 2가지 실패 시나리오를 생각해보자.

```
1. 익스체인지로 퍼블리싱 했는데 매칭되는 목적지 큐가 하나도 없음.
2. 존재하지 않는 익스체인지로 퍼블리싱.
```

- 첫 번째 케이스는 퍼블리셔의 반환으로 대응이 가능(뒤에서 자세히 다룸).
- 두 번째 케이스는 메시지가 drop되고 아무런 반환도 되지 않음.
- 관련된 채널은 예외와 함께 닫힘.
- 기본적으로 이 예외는 로그에 남지만,
- `ChannelListener`를 `CachingConnectionFactory`에 등록해서 이 이벤트를 통지 받을 수 있음.

```
this.connectionFactory.addConnectionListener(new ConnectionListener() {

    @Override
    public void onCreate(Connection connection) {
    }

    @Override
    public void onShutDown(ShutdownSignalException signal) {
        ...
    }

})
```

- signal의 `reason` 프로퍼티에서 문제를 좀 더 확인할 수 있음.
- 보내는 측 스레드에서 이 예외를 감지하고 싶다면,
- `RabbitTemplate`에 `setChannelTransacted(true)`를 하고,
- `txCommit()`에서 예외를 받음.
- 하지만 트랜잭션은 성능을 크게 저하시킴.
- 따라서 트랜잭션 활성화는 신중히 고려할 것.
