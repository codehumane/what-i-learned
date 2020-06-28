
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

TBD


