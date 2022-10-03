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
