# Consumer Acknowledgements and Publisher Confirms

[https://www.rabbitmq.com/confirms.html](https://www.rabbitmq.com/confirms.html)

## Overview

- [데이터 안정성](https://www.rabbitmq.com/reliability.html)에 관한 2가지 주제,
- consumer acknowledgements와 publisher confirms를 다룸.
- acknowledgement는 데이터 안정성에 있어서 컨슈머와 퍼블리서 측 모두에 중요함.
- 더 많은 주제는 [publisher](https://www.rabbitmq.com/publishers.html)와 [consumer](https://www.rabbitmq.com/consumers.html) 가이드 참고.

## The Basics

- 메시징 브로커를 사용하는 시스템들은 기본적으로 분산 환경.
- 메시지가 상대방에게 전달되고 성공적으로 처리 됐다는 보장을 할 수 없으므로,
- 퍼블리서와 컨슈머 모두 전송과 처리 confirmation 메커니즘이 필요.
- RabbitMQ와 같은 메시징 프로토콜들은 이런 기능들을 제공.
- AMQP 0-9-1의 기능들을 소개하긴 하지만, 다른 프로토콜들에서도 대개 유사함.
- 컨슈머에서 RabbitMQ로의 접수 통지가 메시징 프로토콜에서의 acknowledgements로 알려져 있음.
- 브로커에서 퍼블리서로의 접수 통지는 [publisher confirms](https://www.rabbitmq.com/confirms.html#publisher-confirms)라 불리는 프로토콜 확장.
- 이들 모두 신뢰성 있는 메시지 전달을 위해 필수.
- 다시 말해, 데이터 안정성을 위해 꼭 필요.
- RabbitMQ 노드 못지 않게 애플리케이션 역시 이에 대한 책임을 가짐.

## (Consumer) Delivery Acknowledgements

- RabbitMQ가 컨슈머에게 메시지를 전달할 때,
- 메시지 전달의 성공 기준을 뭘로 삼을지 알아야 함.
- 이는 어떤 시스템이냐에 따라 다름.
- 따라서, 근본적으로 애플리케이션의 결정임.
- AMQP 0-9-1에서는 `basic.consume` 메서드를 이용해서 컨슈머가 등록되거나,
- `basic.get` 메서드와 함께 메시지가 온 디멘드로 페치 될 때라고 함.
- (컨슈머 등록이 왜 delivery acknowledgements인지는 이해 못 하는 중)

## Delivery Identifiers: Delivery Tags

- 다른 주제를 다루기에 앞서, 어떻게 전달이 식별되는지 설명하는 것이 중요.
- 그리고 acknowledgements가 개별 전달을 어떻게 지칭하는지도.
- 컨슈머(구독)가 등록되면, RabbitMQ는 `basic.deliver` 메서드를 이용해서 메시지를 전달(푸시)함.
- 이 메서드는 delivery tag를 함께 전달.
- 이 태그가 채널 내의 전달을 고유하게 식별.
- 따라서 delivery tag는 채널 별로 scoped.
- delivery tag는 단순<sup>monotonically</sup> 증가하는 양수이며 클라이언트 라이브러리에 의해 제시됨.
- 메시지 전달을 acknowledge하는 클라이언트 라이브러리 메서드는 delivery tag를 인자로 받음.

## Consumer Acknowledgement Modes and Data Safety Considerations

컨슈머 acknowledgement 소개.

- 노드가 컨슈머에게 메시지를 전달할 때,
- 메시지가 다뤄졌는지 혹은 적어도 전달은 된 건지 판단해야 함.
- 실패는 곳곳에서 일어나므로, 이 결정은 데이터 안정성 관심사임.
- 메시징 프로토콜은 보통 confirmation 메커니즘을 제공함.
- 이는 컨슈머에 대한 접수 통지.
- 이 메커니즘을 사용할지 여부는 컨슈머 구독 시점에 결정됨.

컨슈머 acknowledgements 종류.

- acknowledgements를 사용하기로 했다면,
- 메시지가 보내지자 마자 성공이라고 간주될 수도 있고(TCP 소켓에 쓰여지면),
- 클라이언트가 명시적으로("manual") acknowledgements를 보낼 수도 있음.
- 수동 acknowledgement는 아래 프로토콜 메서드를 사용해서 positive 또는 negative로 설정.
  - `basic.ack`: positive acknowledgement.
  - `basic.nack`: negative acknowledgement.
  - `basic.reject`: negative acknowledgement인데, `basic.nack`에 비해 한 가지 제약을 가짐.
- positive acknowledgements는 단순히 RabbitMQ에게 메시지가 전달됐으며 버려도 된다고 기록하는 것.
- `basic.reject`와 함께 사용하는 negative acknowledgements는 같은 효과를 가짐.
- 버려도 되는 것은 같지만, 성공해서 버리냐(ack), 실패해서 버리냐(nack)의 시멘틱 차이일 뿐.

자동 acknowledgement의 안정성 주의점.

- 자동 acknowledgement 모드에서는, 메시지가 보내지고 난 직후에 전달이 성공했다고 여겨짐.
- 이는 높은 성능을 가져오지만, 전달과 컨슈머 프로세싱의 안정성이 줄어드는 트레이드 오프.
- 이 모드는 종종 "fire-and-forget"이라고 불림.
- 수동 모드와 다르게, 성공적으로 전달되기 전 컨슈머 TCP 커넥션이나 채널이 닫히면,
- 서버가 보낸 메시지는 유실될 수 있음.
- 안정성 측면에서 유의해야 함.

자동 acknowledgement의 부하 주의점.

- 또 한 가지 주의해야 할 것은, 자동 acknowledgement 모드가 컨슈머에게 부하가 될 수 있다는 것.
- 수동 acknowledgement 모드는 전형적으로 제한된 채널 prefetch와 함께 사용됨.
- 이는 채널의 처리되지 않은<sup>outstanding</sup> 메시지의 갯수를 제한하는 것.
- 하지만 자동 모드에서는 이런 제약이 없음.
- 컨슈머는 전달 속도에 의해 부하가 걸릴 수 있고,
- 메모리에 백로그를 계속 쌓아가서 힙을 고갈시키며 OS에 의해 프로세스가 중단 될 수도.
- 일부 클라이언트 라이브러리는 TCP 백 프레셔를 적용.

## Positively Acknowledging Deliveries

- delivery acknowledgement를 위한 API들은 보통 클라이언트 라이브러리의 채널 연산자로써 제공됨.
- Java 클라이언트들은 `Channel#basicAck`와 `Channel#basicNack`를 이용해 `basic.ack`와 `basic.nack`를 각각 수행.

```java
boolean autoAck = false;
channel.basicConsume(
  queueName,
  autoAck,
  "a-consumer-tag",
  new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag,
                               Envelope envelope,
                               AMQP.BasicProperties properties,
                               byte[] body) throws IOEXception {
      long deliveryTag = envelope.getDeliveryTag();
      // positively acknowledge a single delivery,
      // the message will be discarded
      channel.basicAck(deliveryTag, false);
    }
  }
);
```
