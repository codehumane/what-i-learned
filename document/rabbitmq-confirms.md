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
