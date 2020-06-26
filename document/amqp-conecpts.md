
`@RabbitListener` 애노테이션으로 어떤 것까지 할 수 있을지 알아보기 위해, [Annotation-driven Listener Endpoints](https://docs.spring.io/spring-amqp/reference/html/#async-annotation-driven) 문서를 살펴봄. 기본적인 지식이 있어야 하기에, [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html)도 함께 살펴봄. 여기서는 후자의 문서를 기록.

# AMQP 0-9-1 Model Explained

## What is AMQP 0-9-1?

AMQP(Advanced Message Queuing Protocol)는 RabbitMQ가 사용하는 메시징 프로토콜.

### Model in Brief

메시지 브로커는 퍼블리셔<sup>publisher</sup>(프로듀서<sup>producer</sup>라고 하기도)로부터 메시지를 받고<sup>receive</sup>, 다시 컨슈머에게 라우팅<sup>route</sup>. 브로커 안을 좀 더 들여다보면, 메시지는 먼저 익스체인지<sup>exchange</sup>로 보내지고<sup>published</sup>, 익스체인지는 바인딩<sup>binding</sup>이라고 불리는 규칙을 사용해서 메시지 복사본을 큐에 분산<sup>distribute</sup>. 그림으로 보면 아래와 같음.

!["Hello, world" example routing](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)

- 퍼블리셔는 메시지를 보낼 때 다양한 *메시지 속성*<sup>message attributes</sup>(메시지 메타데이터)을 지정.
- 일부 메타데이터는 브로커가 사용.
- 나머지는 완전히 opaque. 메시지 받는 애플리케이션에서만 관심.
- 네트워크 실패나 애플리케이션의 처리 실패 때문에, message acknowledgements 제공. 자동으로 브로커에게 알려줄 수도 있고, 개발자가 직접 결정할 수도 있음. 이 기능을 사용할 때는, 브로커가 통지를 받았을 때에만 큐에서 메시지를 제거함.
- 메시지가 라우팅 되지 않으면, 메시지는 퍼블리셔에게 반환되거나<sup>returned</sup>, 제거되거나<sup>dropped</sup>, DLQ로 이동됨.

*큐<sup>queue</sup>, 익스체인지<sup>exchange</sup>, 바인딩<sup>binding</sup>을 통칭하여 AMQP 엔티티라고 함.

### Programmable Protocol

- 프로그래밍 가능한 프로토콜
- 즉, 엔티티와 라우팅 스키마가 기본적으로 브로커 어드민이 아닌 애플리케이션에 의해 정의됨.
- 큐와 익스체인지를 선언하고, 이들 간의 바인딩을 정의하고, 큐 구독 등을 진행.
- 높은 자유도를 주긴 하지만 실수나 설정 충돌 등의 가능성을 가짐.

## Exchanges and Exchange Types

- 익스체인지는 AMQP 0-9-1의 엔티티.
- 메시지를 받고, 0개 이상의 큐에 라우팅.
- 라우팅 알고리즘은 익스체인지 타입과 바인딩이라고 불리는 규칙에 따라 다름.
- AMQP 0-9-1 브로커는 아래의 4가지 익스체인지 타입을 제공.

| Exchange type    | Default pre-declared names              |
| :--------------- | :-------------------------------------- |
| Direct exchange  | (Empty string) and amq.direct           |
| Fanout exchange  | amq.fanout                              |
| Topic exchange   | amq.topic                               |
| Headers exchange | amq.match (and amq.headers in RabbitMQ) |

익스체인지 타입 외에도, 몇 가지 중요한 속성들이 있음.

- Name
- Durability (exchanges survive broker restart)
- Auto-delete (exchange is deleted when last queue is unbound from it)
- Arguments (optional, used by plugins and broker-specific features)

### Default Exchange

- 기본 익스체인지는 이름이 없는(빈문자열) direct 타입.
- 브로커에 의해 미리 선언되어 있음.
- 생성되는 모든 큐는 자동으로 이 익스체인지에 바운딩.
- 이 때의 라우팅 키는 큐의 이름과 동일함.
- 단순한 애플리케이션에 유용.

### Direct Exchange

- 메시지를 라우팅 키에 기반해서 큐에 전달.
- 메시지의 unicast 라우팅에 이상적.
- 큐는 라우팅 키 K와 함께 익스체인지에 바운딩.
- 라우팅 키 R과 함께 새로운 메시지가 direct 익스체인지에 도착하면,
- 익스체인지는 K = R 여부를 판단하고 큐로 이 메시지를 라우팅.
- 그리고 다수의 워커들(동일 애플리케이션의 인스턴스들)이 있으면,
- 작업<sup>task</sup>을 라운드 로빈 방식으로 분산시켜 줌.
- 큐가 아닌 컨슈머들에 대한 로드 밸런싱임에 유의.

![direct exchange](https://www.rabbitmq.com/img/tutorials/intro/exchange-direct.png)

### Fanout Exchange

- 메시지를 바운딩 된 모든 큐에 라우팅.
- 라우팅 키는 무시됨.
- N개의 큐가 fanout 익스체인지에 바운딩 되어 있으면,
- 새로운 메시지가 도착했을 때 이 메시지의 복제본을 모든 N개의 큐에 전달.
- 메시지의 broadcast 라우팅에 유리.

![fanout exchange](https://www.rabbitmq.com/img/tutorials/intro/exchange-fanout.png)

### Topic Exchange

- 1개 이상의 큐에 메시지 라우팅.
- 단, '큐가 익스체인지에 바운딩 될 때 사용된 패턴'과 '메시지의 라우팅 키'가 매칭될 때만.
- 마찬가지로, 메시지의 multicast 라우팅에 주로 사용.

### Headers Exchange

- 라우팅 키는 무시됨.
- 대신, 메시지 헤더에 있는 속성들을 기반으로 라우팅.
- '큐가 연결<sup>bound</sup>될 때 지정된 값'과 '헤더의 값'을 매칭하여 라우팅.
- `x-match` 인자의 값이 `all`일 때는 모든 헤더를 매칭하고,
- `x-match` 인자의 값이 `any`일 때는 1개 헤더만 매칭.
- 라우팅 키가 문자열일 필요가 없는(정수형이거나 딕셔너리 같은) 곳에서 direct 익스체인지를 사용하기도.
- 참고로, `x-`로 시작하는 헤더는 매치에 사용되지 않음.
- [여기 예시](https://codedestine.com/rabbitmq-headers-exchange/)가 이해를 도움.
- 애플리케이션에서 해줘야 하는 일이 늘어나는 한편, 좀 더 자유로운 라우팅이 가능.

## Queues

- 메시지를 저장하고 애플리케이션에 의해 소비.
- 큐는 익스체인지와 프로퍼티 일부를 공유하지만,
- 아래의 부가적인 속성도 가지고 있음.

| 속성 | 설명 |
| :-- | :-- |
| Name | |
| Durable | the queue will survive a broker restart |
| Exclusive | used by only one connection and the queue will be deleted when that connection closes |
| Auto-delete | queue that has had at least one consumer is deleted when last consumer unsubscribes |
| Arguments | optional; used by plugins and broker-specific features such as message TTL, queue length limit, etc |

- 큐가 사용되려면 먼저 선언되어야 함.
- 큐가 선언되면, 이미 있는 이름의 큐인지를 확인 후 없으면 새로 생성.
- 이미 존재하는 이름이고 속성들이 같으면 아무 일도 일어나지 않음.
- 이름은 같은데 속성이 다르면 406 코드(`PRECONDITION_FAILED`)와 함께 채널 레벨의 예외가 발생함.

### Queue Names

- 애플리케이션은 큐 이름을 고르거나, 브로커에게 만들어 달라고 요청할 수도.
- 큐 이름으로는 UTF-8로 255 바이트까지 사용 가능.
- 큐 이름 인자를 빈문자열로 전달하면 브로커가 애플리케이션을 대신해 고유한 큐 이름을 생성함. 이렇게 생성된 이름은 클라이언트에게 큐 선언에 대한 응답으로 반환됨.
- `amq.`은 예약어. 브로커 내부의 사용 때문. 이를 사용하려고 하면 채널 레벨에서 403 코드(ACCESS_REFUSED) 예외가 일어남.

### Queue Durability

- 큐는 druable 또는 transient로 선언 가능.
- durable 큐의 메타데이터는 디스크에 저장됨.
- transient 큐의 것은 메모리에 저장.
- 이 구분은 발송 시점의 메시지에 대해서도 마찬가지.
- [여기](https://www.rabbitmq.com/queues.html#durability)에 따르면, 애플리케이션이 durable 큐를 사용해야 할 뿐만 아니라 메시지도 persisted로 표기해서 보내야 함.

## Bindings

- 바인딩은 익스체인지가 메시지를 큐에 라우팅하기 위해 사용하는 규칙.
- 익스체인지 E가 큐 Q에게 메시지를 라우팅하기 위해서는, Q가 E에 연결되어<sup>bound</sup> 있어야 함.
- 바인딩은 선택적으로 *라우팅 키* 속성을 가질 수 있음. 일부 익스체인지 타입에서 사용.
- 라우팅 키의 역할은 메시지를 필터링하는 것.
- 바인딩으로 인해 일종의 레이어가 추가된 건데, 이를 통해 퍼블리서가 큐에 바로 연결되면 하기 어려운 일들을 가능하게 해주고(연결을 여러 개 둘 수도 있으니 그런 듯) 애플리케이션 개발의 중복된 작업도 덜어줌.
- 만약 일치하는 바인딩이 없어 어떤 큐에도 메시지가 전달될 수 없다면, 제거되거나<sup>dropped</sup> 퍼블리셔에게 다시 반환됨. 이는 [퍼블리셔가 설정한 메시지 속성](https://www.rabbitmq.com/publishers.html#unroutable)에 따름.
- [여기 스프링 가이드](https://spring.io/guides/gs/messaging-rabbitmq/)를 보면 간단한 예시가 나옴.

```java
@Bean
Binding binding(Queue queue, TopicExchange exchange) {
  return BindingBuilder
      .bind(queue)
      .to(exchange)
      .with("foo.bar.#");
}
```

이에 대한 설명은 아래와 같이 하고 있음.

> In this case, we use a topic exchange, and the queue is bound with a routing key of `foo.bar.#`, which means that any messages sent with a routing key that begins with `foo.bar.` are routed to the queue.

## Consumers

- 애플리케이션의 소비가 있어야 큐에 메시지 저장하는 것이 의미 있음.
- 소비에는 2가지 방법이 존재. push API와 poll API.
- 그런데, pull을 가리켜 대부분의 경우에 "highly inefficient and should be avoided"라고 함.
- 그 동안 써왔던 것은 push API와 prefetch를 같이 두었던 건데,
- 사실상 pull과 같은 것으로 생각하고 있었고,
- 메시지 소비와 생산의 속도를 일치시키지 않아도 되기 때문에 오히려 유리한 것 아니었나?
- 별다른 후속 설명이 없어 의문만 남기고 일단은 넘어감.
- push API 사용 시에는 애플리케이션이 특정 큐의 메시지 소비를 하겠다고 명시해야 함.
- 이를 가리켜 "register a conumser" 또는 "subscribe to a queue"라고 표현.
- 독점적 컨슈머<sup>exclusive consumer</sup> 등록도 가능하고 하나의 큐에 대해 여러 컨슈머 등록도 가능.
- 각 컨슈머는 컨슈머 태그<sup>consumer tag</sup>라고 하는 식별자를 가짐.
- 이 식별자는 단순 문자열이며, 구독 취소 등에 사용.

### Message Acknowledgements

- 컨슈머 애플리케이션은 실패할 수 있음.
- 네트워크 이슈로 문제가 생길 수도 있음.
- 브로커는 언제 큐에서 메시지를 제거해야 하는가에 대한 질문으로 귀결.
- 이를 위해 2가지 acknowledgement 모드를 제공함.
- 하나는 애플리케이션에 메시지를 전달한 뒤.
- 다른 하나는 애플리케이션이 acknowledgement를 응답한 뒤.
- 전자는 자동 acknowledgement 모델.
- 후자는 명시적 acknowledgement 모델.
- 전자는 `basic.deliver` 또는 `basic.get-ok` 메서드 이용.
- 후자는 `basic.ack` 메서드 이용.
- 만약, acknowledgement를 보내지 못하고 컨슈머가 죽으면, 브로커는 다른 컨슈머에게 재전달. 다른 컨슈머가 없다면 새로운 컨슈머가 등록될 때까지 기다림.

### Rejecting Messages

- 애플리케이션은 메시지 처리가 실패했음을 메시지 거절<sup>rejecting a message</sup>을 통해 브로커에게 알려줄 수 있음.
- 이 때, 메시지를 버릴지<sup>discard</sup> 다시 큐에 적재<sup>requeue</sup> 할지를 브로커에게 요청할 수 있음.
- 큐에 컨슈머가 하나 뿐일때는, 거절과 재적재로 인한 무한 루프가 생기지 않도록 유의.

### Negative Acknowledgements

- 메시지 거절에는 `basic.reject` 메서드가 사용됨.
- 이 메서드에는 한 가지 제약이 있음.
- 한 번에 다수의 메시지에 대해 거절할 수 없는 것.
- 하지만 acknowledgement 메서드에는 `multiple` 필드를 가질 수 있고,
- 이를 통해 다수에게 acknowledgement를 보낼 수 있는데,
- `basic.nack`(negative acknowledgement)이 이 필드를 가짐.
- 과거에 만들어진 `basic.reject`는 불가.
- 좀 더 자세한 내용은 [여기](https://www.rabbitmq.com/confirms.html#consumer-acks-multiple-parameter) 참고.

### Prefetching Messages

- 설명이 어려워서 [Channel Prefetch Setting (QoS)](https://www.rabbitmq.com/confirms.html#channel-qos-prefetch)와 [Consumer Prefetch](https://www.rabbitmq.com/consumer-prefetch.html) 문서를 참고.
- 요약하면, "limit the number of unacknowledged messages on a channel".
- pull이 아닌 push와 같이 사용됨에 유의.
- 사용 목적은 주로 컨슈머 측의 unbounded buffer 문제를 피하기 위함.
- `basic.qos` 메서드를 통해 "prefetch count" 설정함.

## Message Attributes and Payload

- 메시지는 *속성*<sup>attribute</sup>을 가짐.
- 몇몇 속성은 자주 사용되어 AMQP 명세가 아예 정의를 해 두었음.
- 문서에 몇 가지 예시를 들었는데, 이보다는 [Message Properties](https://www.rabbitmq.com/publishers.html#message-properties)를 참고하는 게 도움될 것.
- 일부는 브로커에 의해 사용되지만, 대부분은 메시지를 받는 애플리케이션에서 사용되는 것들.
- 일부 속성들은 선택적이며, 헤더라고 알려져 있음.
- 메시지가 보내질 때 속성들이 설정됨.
- 당연하게도 메시지는 페이로드를 가짐.
- 브로커는 이를 불투명한 바이트 배열로 간주.
- 보지도 않고 수정하지도 않는 것.
- 속성만 가지고 페이로드는 없는 메시지도 가능.

끝.
