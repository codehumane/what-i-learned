# Reliability Guide

## Overview

- 데이터 안정성<sup>data safety</sup>에 관한 기능들 간략 소개.
- 이들은 애플리케이션이 신뢰성 있는 전달<sup>reliable delivery</sup>을 달성하도록 도움.
- 즉, 각종 실패를 만나더라도, 메시지가 항상 전달되도록 보장.
- 데이터 안정성은 RabbitMQ 노드들과 퍼블리서, 컨슈머 모두의 책임.
- 아래는 데이터 안정성과 회복성에 관한 주제들 상세.
- [Acknowledgements and Confirms](https://www.rabbitmq.com/confirms.html)
- [Clustering](https://www.rabbitmq.com/clustering.html)
- [Replicated Queues](https://www.rabbitmq.com/quorum-queues.html) and [Streams](https://www.rabbitmq.com/streams.html)
- [Publishers](https://www.rabbitmq.com/publishers.html)
- [Consumers](https://www.rabbitmq.com/consumers.html)
- [Resource Alarms](https://www.rabbitmq.com/alarms.html)
- [Monitoring, Metrics and Health Checks](https://www.rabbitmq.com/monitoring.html)

## What Can Fail?

- 메시징 기반 시스템은 분산 환경이며, 여러 이유로 때로는 모호한 방식으로 실패할 수 있음.
- 네트워크 연결 문제나 혼잡이 주요한 실패 이유.
- 방화벽이 연결을 유휴 상태라 판단하고 방해할 수도.
- 그리고 이런 네트워크 실패는 탐지하는 데 시간이 걸림.
- 한편, 서버와 클라이언트 애플리케이션은 하드웨어 실패를 겪기도.
- 소프트웨어가 고장날 수도 있음.
- 또는 로직 에러로 채널이나 커넥션 실패를 일으킬 수도.
- omission failure (예상된 시간 내에 응답 실패하는 것).
- 성능 저하.
- 자원 고갈.

## Connection Failures

- RabbitMQ 노드와 클라이언트 간 네트워크 연결 실패 시에는,
- 클라이언트가 브로커로 새로운 연결을 수립해야 함.
- 이미 연결되어 열린 상태의 채널들도 자동으로 닫히고 다시 열려야 함.
- 일반적으로 커넥션이 실패하면, 커넥션이 예외를 일으켜서 클라이언트에 통지.
- 대부분의 클라이언트 라이브러리들은 커넥션 실패 시 자동 복구하는 기능들을 제공.
- 이것이 적절치 않은 경우라면, 연결 실패 이벤트 핸들러 정의를 통해 자체적 복구 메커니즘 마련.

## Acknowledgements and Confirms

acknowledgements 필요성 이야기.

- 연결이 실패하면, 메시지는 클라이언트와 서버 사이에 전송 중일 수 있음.
- 클라이언트나 서버에서 디코드나 인코딩 중일 수도 있고,
- TCP 스택 버퍼에 머물거나, 와이어를 통해 전송 중일 수도.
- 이런 메시지는 전달되지 않을 가능성이 큼.
- 재전송을 고려해야 함.
- [acknowledgements](https://www.rabbitmq.com/confirms.html)이 도움이 됨.

acknowledgements 방향과 각각의 용어.

- acknowledgements는 양쪽에서 사용될 수 있음.
- 컨슈머가 메시지를 받았고 처리했음을 서버에게 알려줄 수 있고(consumer acknowledgements),
- 서버가 퍼블리서에게 알려줄 수도 있음(publisher confirms).

TCP 전달 보장과의 차이점.

- TCP는 패킷이 연결 상대에게 전달됨을 보장.
- 그리고 이것이 될 때까지 재전송.
- 하지만, 이는 네트워크 레이어의 실패만을 다룬 뿐임.
- acknowledgements와 confirms는 메시지를 받았고,
- 상대 어플리케이션이 이에 대해 처리를 마쳤다는 것까지 알려줌.

acknowledgements의 시멘틱.

- 컨슈머 애플리케이션은 전달 받은 메시지에 대해,
- 필요한 작업을 다 끝내기 전까지 acknowledge를 하면 안 됨.
- 비슷하게, 브로커는 메시지에 대한 책임을 다 수행하면 메시지를 confirm.

at least once, at most once.

- acknowledgements의 사용은 at least once 전달을 보장.
- 이것이 없으면 퍼블리시와 컨슘 사이에 메시지 유실 가능하고,
- 따라서 at most once 정도만을 보장.

## Detecting Dead TCP Connections with Heartbeats

- 고장난 TCP 연결은 OS가 감지하는 데 오래 걸릴 수 있음.
- 이는 패킷 손실로 이어지기도 함.
- AMQP 0-9-1은 [heartbeat feature](https://www.rabbitmq.com/heartbeats.html)를 제공.
- 이를 통해 애플리케이션 레이어가 고장난 연결을 즉각 찾아낼 수 있도록 함.
- 유휴 TCP 연결을 종료시키는 네트워크 장비에 대한 대응이 될 수 있음.

## Data Safety on the RabbitMQ Side

- RabbitMQ 측의 메시지 손실을 피하기 위해서는,
- 큐와 메시지가 RabbitMQ 노드 재시작이나 노드와 하드웨어 고장을 다룰 수 있어야 함.
- 이를 위해 [durability of queues and messages](https://www.rabbitmq.com/queues.html#durability),
- [published as persistent by publishers](https://www.rabbitmq.com/publishers.html#message-properties)를 활용.
