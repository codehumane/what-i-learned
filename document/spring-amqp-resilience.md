# Resilience: Recovering from Errors and Broker Failures

아래 문서의 간단한 정리.

https://docs.spring.io/spring-amqp/reference/html/#_resilience_recovering_from_errors_and_broker_failures

- Spring AMQP는 프로토콜 에러나 브로커 장애 시 복구와 자동 재연결 기능을 추상화 된 형태로 제공.
- `CachingConnectionFactory`가 이런 재연결 기능을 제공.
- `RabbitAdmin`을 사용한 자동-선언<sup>auto-declaration</sup> 기능도 유용.
- 전달<sup>delivery</sup>의 보장에는 `RabbitTemplate`의 `channelTransacted` 플래그, `SimpleMessageListenerContainer`, `SimpleMessageListenerContainer`의 `AcknowledgeMode.AUTO`(또는 직접 acks)가 활용됨.

## Automatic Declaration of Exchanges, Queues, and Bindings

- `RabbitAdmin` 컴포넌트는 애플리케이션 구동 시 익스체인지, 큐, 바인딩을 선언할 수 있음.
- `ConnectionListener`를 통해 lazy하게 이를 수행.
- 따라서, 구동 시에는 브로커가 없어도 됨.
- `Connection`이 처음으로 사용될 때(예컨대 메시지 보낼 때),
- 리스너가 점화되고(fire) 어드민 기능이 적용됨.
- 자동 선언의 또 다른 이점은 커넥션이 어떤 이유로든 끊어졌을 때(dropped), 재연결 때 다시 적용된다는 것.

## Failures in Synchronous Operations and Options for Retry

- `RabbitTemplate`을 이용해 동기 처리를 하는 중에 브로커와의 연결이 끊어지면,
- Spring AMQP는 `AmqpException`(보통 `AmqpIOException`)을 발생시킴.
- 이 예외를 받으면 연결 유실을 의심하고 연산 재시도를 해볼 수 있음.
- 직접 해도 되고 Spring Retry(선언적이거나 명령형인 방식 모두 제공)를 이용해도.
- Spring AMQP에서는 Spring Retry 인터셉터를 편리하게 생성하도록 돕는 팩토리 빈들을 제공.
- `StatefulRetryOperationsInterceptor`, `StatelessRetryOperationsInterceptor` 참고.
- 참고로, 상태 없는 재시도는 트랜잭션이 없거나, 트랜잭션이 재시도 콜백 안에서 일어나도 되는 곳에 적절.
- 상태 없는 재시도가 설정도 쉽고 분석에도 용이.
- 하지만 진행 중인 트랜잭션이 있고, 연결 실패로 인해 롤백 되어야 하는 경우에는 상태 있는 재시도가 필요.
- 이 때는 메시지를 고유하게 식별하는 등의 노력이 필요(`MessageKeyGenerator` 등 참고).
- 아래는 인터셉터를 구성하는 예시.

```java
@Bean
public StatefulRetryOperationsInterceptor interceptor() {
	return RetryInterceptorBuilder.stateful()
			.maxAttempts(5)
			.backOffOptions(1000, 2.0, 10000) // initialInterval, multiplier, maxInterval
			.build();
}
```

## Retry with Batch Listeners

- producer-created 배치에서만 사용.
- 이 배치에서는 실패한 메시지가 오직 하나이므로 복구가 가능.
- 하지만 consumer-created 배치에서는 어떤 메시지가 실패했는지 알 수 없음.
- [Sending Messages의 Batching](https://docs.spring.io/spring-amqp/reference/html/#template-batching) 함께 참고.

## Message Listeners and the Asynchronous Case

비즈니스 실패와 끊어진 연결에 의한 실패를 구분하고 서로 다르게 대응.

- `MessageListener`가 비즈니스 예외로 실패하면, 이 예외의 처리는 메시지 리스너 컨테이너에 넘어가고, 다시 다른 메시지 리스닝.
- 끊어진 연결에 의한 실패라면, 리스너를 위해 메시지를 수집하는 컨슈머가 취소되고 재시작 됨. `SimpleMessageListenerContainer`가 이를 매끄럽게 처리.
- 실패에 의한 컨슈머의 재시작은 무한정 반복. 아주 심각한 경우에만 이를 멈춤.

비즈니스 실패에 대한 처리가 좀 더 복잡하며 나머지 부분은 이에 대한 이야기.

- 비즈니스 예외 핸들링에는 좀 더 많은 고민과 커스텀 설정이 필요.
- 트랜잭션이나 컨테이너 acks를 사용할 때 특히 더 그러함.
- DLQ 지원이 없던 시절에는, 거부되거나 롤백된 메시지라고 하더라도 무한정 재전달 되었음.
- 재전달에 대한 횟수 제한의 한 가지 방법은 리스너 어드바이스 체인에서 `StatefulRetryOperationsInterceptor`를 사용하는 것.
- 인터셉터는 데드 레터 행위에 대한 커스텀 구현 콜백을 가질 수 있음.
- 또 다른 한 가지 방법은 컨테이너의 `defaultRequeueRejected` 프로퍼티를 `false`로 설정.
- 실패한 메시지를 모두 버리는 것.
- `AmqpRejectAndDontRequeueException`을 던지는 것도 재적재를 피하는 방법.
- 이는 `defaultRequeueRejected` 프로퍼티와 독립적.
- `ImmediateRequeueAmqpException`도 2.1 버전 이후부터 제공되기 시작. 이름 그대로 반대 기능.

위에서 설명한 방법들을 종종 아래와 같이 조합해서 사용.

- 어드바이스 체인에서 `StatefulRetryOperationsInterceptor`와 `MessageRecoverer`를 함께 사용.
- `MessageRecoverer`는 재시도가 모두 실패한 경우에 호출되며,
- 여기서 `AmqpRejectAndDontRequeueException`을 던지도록 함.
- `RejectAndDontRequeueRecoverer`가 정확히 이 일을 함.
- 기본 `MessageRecoverer` 구현체는 단지 WARN 로깅만 함.
- 1.3 버전에서부터는 `RepublishMessageRecoverer`를 제공.
- 이는 재시도가 모두 실패한 메시지를 퍼블리시 하도록 도와줌.
- 퍼블리시 할 때는 스택트레이스, 원래의 익스체인지, 라우팅 키 등을 메시지 헤더에 담는다.
- 헤더가 단일 프레임에 잘 들어맞도록 너무 긴 스택트레이스는 일부 잘릴 수 있음.
- `frameMaxHeadroom` 속성을 통해 조정 가능.
- `ImmediateRequeueAmqpException` 예외를 던져 바로 리스너에게 바로 재적재를 요청하는 `ImmediateRequeueMessageRecoverer`도 2.1 버전에서 추가됨.
