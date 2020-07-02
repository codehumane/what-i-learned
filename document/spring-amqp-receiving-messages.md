
`@RabbitListener` 애노테이션으로 어떤 것까지 할 수 있을지 알아보기 위해, Spring AMQP 문서의 [Annotation-driven Listener Endpoints](https://docs.spring.io/spring-amqp/reference/html/#async-annotation-driven) 부분을 살펴봄. 읽다 보니 [Receiving Messages](https://docs.spring.io/spring-amqp/reference/html/#receiving-messages) 내용이 좋아 전체적으로 기록.

AMQP의 기본적 개념은 [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html) 문서 참고.

# Receiving Messages

- 메시지 수신은 늘 발송에 비해 좀 더 복잡하다고 함.
- 수신 방법은 두 가지.
- 좀 더 간단한 한 가지는 폴링.
- 메서드 호출을 통해 한 번에 한 개의 `Message`만 가져오는 것.
- 좀 더 복잡하고 더 흔한 방법은 리스너 등록.
- 리스너는 온디멘드로, 그리고 비동기로 `Message`를 받음.

## Polling Consumer

[AMQP 0-9-1 Model Explained의 Consumers](https://www.rabbitmq.com/tutorials/amqp-concepts.html#consumers) 부분에 보면 Polling("pull API")을 아래와 같이 설명하고 있음.

> this way is highly inefficient and should be avoided in most cases

그래서 간단히만 기록.

- `AmqpTemplate`으로 `Message` 폴링.
- 메시지가 없다면 `null` 반환.
- 즉, 블럭킹이 없음.
- 1.5 버전 이후로는 `receiveTimeout`과 함께 블럭 가능.
- 1.6 이후로는 매 호출마다 타임아웃을 지정하는 `receive` 제공.
- 시그니처들만 이 내용은 기록하고 마무리.

```java
Message receive() throws AmqpException;
Message receive(String queueName) throws AmqpException;
Message receive(long timeoutMillis) throws AmqpException;
Message receive(String queueName, long timeoutMillis) throws AmqpException;

Object receiveAndConvert() throws AmqpException;
Object receiveAndConvert(String queueName) throws AmqpException;
Object receiveAndConvert(long timeoutMillis) throws AmqpException;
Object receiveAndConvert(String queueName, long timeoutMillis) throws AmqpException;

<R, S> boolean receiveAndReply(
  ReceiveAndReplyCallback<R, S> callback
) throws AmqpException;

<R, S> boolean receiveAndReply(
  String queueName,
  ReceiveAndReplyCallback<R, S> callback
) throws AmqpException;

<R, S> boolean receiveAndReply(
  ReceiveAndReplyCallback<R, S> callback,
  String replyExchange,
  String replyRoutingKey
) throws AmqpException;

<R, S> boolean receiveAndReply(
  String queueName,
  ReceiveAndReplyCallback<R, S> callback,
  String replyExchange,
  String replyRoutingKey
) throws AmqpException;

<R, S> boolean receiveAndReply(
  ReceiveAndReplyCallback<R, S> callback,
  ReplyToAddressCallback<S> replyToAddressCallback
) throws AmqpException;

<R, S> boolean receiveAndReply(
  String queueName,
  ReceiveAndReplyCallback<R, S> callback,
  ReplyToAddressCallback<S> replyToAddressCallback
) throws AmqpException;
```

## Asynchronous Consumer

`@RabbitListener`를 이용한 비동기 컨슈머 설정이 현재로써 가장 편리한 방식이라 하여 이 내용은 생략.

## Batched Messages

- 프로듀서에 의해 배치로 묶인 메시지들은 리스너 컨테이너에 의해 자동으로 풀려서 전달됨(`SpringBatchFormat` 메시지 헤더를 사용).
- 배치 중 어느 메시지 하나라도 거절되면 전체가 거절되는 것에 유의.
- 그 외 여러가지 이야기를 하는데, 보내는 측과 받는 측의 이런 저런 설정이 필요하다는 내용.

## Consumer Events

- 리스너(컨슈머)가 특정 실패를 겪으면 애플리케이션 이벤트를 발행.
- 이벤트는 `ListenerContainerConsumerFailedEvent`이며 몇 가지 속성을 가짐.
    - `container`: 어떤 리스너 컨테이너가 겪었는지.
    - `reason`: 실패 이유
    - `fatal`: 실패가 심각한지 여부. 심각하지 않은 경우에는 컨테이너가 `recovertyInterval`이나 `recoveryBackoff` 또는 `monitorInterval`에 따라 컨슈머를 재시작 시킨다고 함.
    - `throwable`: 말 그대로 `Throwable`.
- 그 외에도 `AsyncConsumerRestartedEvent` 등의 여러 가지 컨테이너 라이프사이클 단계를 나타내는 이벤트들이 발행됨.

## Consumer Tags

- 컨슈머 태그는 컨슈머의 고유 식별자를 가리킴.
- 아래 인터페이스를 통해 컨슈머 태그를 생성할 수 있음.

```java
public interface ConsumerTagStrategy {
  String createConsumerTag(String queue);
}
```

## Annotation-driven Listener Endpoints

- 비동기로 메시지를 받는 가장 쉬운 방법.
- 애노테이션으로 된 리스너 엔드포인트 인프라를 이용하는 것.

일단 가장 간단한 예시부터 살펴봄.

```java
@Component
public class MyService {
  @RabbitListener(queues = "myQueue")
  public void processOrder(String data) {
    ...
  }
}
```

위 코드에 대한 설명은 아래와 같음.

1. `myQueue`라는 이름의 큐에 메시지를 사용할 수 있으면<sup>available</sup>,
2. `processOrder` 메서드가 적절히(여기서는 메시지의 페이로드와 함께) 호출됨.
3. 애노테이션이 달린 각 메서드 별로 메시지 리스너 컨테이너가 뒷단에서 생성됨.
4. 이 때 `RabbitListenerContainerFactory`가 사용됨.
5. 위 예시에서는 `myQueue`가 반드시 미리 존재해야 함.
6. 그리고 익스체인지에 연결되어 있어야 함.
7. `RabbitAdmin`이 애플리케이션 컨텍스트에 존재하면 자동으로 큐가 선언되고 연결될 수 있음.

댜음으로 3가지 예시 나옴. 자세한 내용은 원문과 [RabbitListener](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/annotation/RabbitListener.html) 참고.

```java
@Component
public class MyService {

  @RabbitListener(bindings = @QueueBinding(
    value = @Queue(value = "myQueue", durable = "true"),
    exchange = @Exchange(value = "auto.exch", ignoreDeclarationExceptions = "true"),
    key = "orderRoutingKey")
  )
  public void processOrder(Order order) {
    ...
  }

  @RabbitListener(bindings = @QueueBinding(
    value = @Queue,
    exchange = @Exchange(value = "auto.exch"),
    key = "invoiceRoutingKey")
  )
  public void processInvoice(Invoice invoice) {
    ...
  }

  @RAbbitListener(queuesToDeclare = @Queue(name = "${my.queue}", durable = "true))
  public String handleWithSimpleDeclare(String data) {
    ...
  }

}
```

2.0 이후부터는 여러 라우팅 키로 큐를 익스체인지에 연결할 수 있다고 함.

```java
key = { "red", "yellow" }
```

`@QueueBinding`을 이용해 큐에 인자를 지정할 수도 있음.

```java
@RabbitListener(
  bindings = @QueueBinding(
    value = @Queue(
      value = "auto.headers",
      autoDelete = "true",
      arguments = @Argument(
        name = "x-message-ttl",
        value = "10000",
        type = "java.lang.Integer"
      )
    ),
    exchange = @Exchange(
      value = "auto.headers",
      type = ExchangeTypes.HEADERS,
      autoDelete = "true"
    ),
    arguments = {
      @Argument(name = "x-match", value = "all"),
      @Argument(name = "thing1", value = "somevalue"),
      @Argument(name = "thing2")
    }
  )
)
public String handleWithHeadersExchange(String foo) {
  ...
}
```

간단히 기록하면 아래와 같음.

1. `x-message-ttl` 지정할 때 10초를 지정.
2. 인자 타입이 문자열이 아니므로 `Integer` 타입을 따로 명시.
3. 만약, 같은 이름의 큐가 이미 존재한다면, 선언된 인자들이 기존 큐의 것과 모두 일치해야 함.
4. 헤더 익스체인지를 사용하기 위해, 바인딩 인자들을 지정함.
5. `thing1` 헤더 값은 `somevalue`여야 하며, `thing2` 헤더는 어느 값이든 존재하기만 하면 됨.
6. `x-match`가 `all`이므로 모든 인자 조건이 만족해야 함.

### Meta-annotations

여러 리스너에 대해 동일한 설정을 사용하고 싶을 때가 있음. 보일러플레이트 설정을 줄이고자 할 때, meta-annotations를 사용.

```java
@Target({
  ElementType.TYPE,
  ElementType.METHOD,
  ElementType.ANNOTATION_TYPE
})
@Retention(RetentionPolicy.RUNTIME)
@RabbitListener(
  bindings = @QueueBinding(
    value = @Queue,
    exchange = @Exchange(
      value = "metaFanout",
      type = ExchangeTypes.FANOUT
    )
  )
)
public @interface MyAnonFanoutListener {
}

public class MetaListener {

  @MyAnonFanoutListener
  public void handle1(String foo) {
    ...
  }

  @MyAnonFanoutListener
  public void handle2(String foo) {
    ...
  }

}
```

프로퍼티 오버라이딩을 위한 `@AliasFor`, 하나의 메서드에 대해 여러 컨테이너를 지정하도록 돕는 `@Repeatable` 설명도 하고 있음.

### Enable Listener Endpoint Annotations

```java
@Configuration
@EnableRabbit
public class AppConfig {

  @Bean
  public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory());
    factory.setConcurrentConsumers(3);
    factory.setMaxConcurrentConsumers(10);
    factory.setContainerCustomizer(container -> /* customize the container */);
    return factory;
  }
}
```

- `@RabbitListener` 애노테이션을 위해 `@EnableRabbit` 선언.
- `SimpleRabbitListenerContainerFactory` 대신 2.0부터는 `DirectMessageListenerContainerFactory` 사용도 가능.
- 둘의 차이는 [Choosing a Container](https://docs.spring.io/spring-amqp/reference/html/#choose-container) 참고.
  - 기존 버전은 큐의 각 컨슈머 별로 스레드가 할당 되었음.
  - 다수의 동시 전송이 불가능한 구조.
  - 새로운 버전은 동시성 지원.
  - 리스너는 RabbitMQ Client 스레드에서 직접 호출됨.
  - 따라서 더 간단하고 효율성도 증대.
  - 하지만 이로 인한 한계들도(기능적 한계도 포함).
- `setContainerCustomizer`를 통해 `ContainerCustomizer` 구현체 제공 가능. 컨테이너 팩토리에서 할 수 없던 프로퍼티들의 조작이 가능.
- 인프라스트럭처(라고 계속 표현하는 게 잼있음)는 `rabbitListenerContainerFactory` 빈을 먼저 찾고, 이를 메시지 리스너 컨테이너 생성하는 데 사용.
- 이렇게 되면 RabbitMQ 인프라스트럭처 셋업은 무시됨.
- 위 예시에서는 `processOrder` 메서드가 3~10개의 스레드 풀에서 호출.
- `MessagePostProcessor` 추가가 가능하고, 이는 메시지를 받고난 뒤(하지만 리스너 호출 전) 그리고 replies를 보내기 전에 적용.
- `RetryTemplate`과 `RecoveryCallback` 추가도 가능. 그런데 이는 replies를 보내는 데 사용된다고 함.
- `@RabbitListener`는 `concurrency` 프로퍼티를 가지는데, `DirectMessageListenerContainer`가 사용될 때는 `consumerPerQueue` 프로퍼티가 설정되고, `SimpleRabbitListenerContainer`가 사용될 때는 `concurrentConsumers` 프로퍼티가 설정됨.
- `autoStartup`과 `taskExecutor`, `ackMode` 프로퍼티 설정도 가능.

### Message Conversion for Annotated Methods

### Programmatic Endpoint Registration

### Annotated Endpoint Method Signature

### Listening to Multiple Queues

### Reply Management

### Multi-method Listeners

### `@Repeatable` `@RabbitListener`

### Proxy `@RabbitListener` and Generics

### Handling Exceptions

### Container Management

### 