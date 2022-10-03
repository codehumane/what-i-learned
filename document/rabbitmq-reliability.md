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

publisher duplication.

- 참고로 이건 consumer 입장이며,
- 뒤에서 다루는 data safety on the publisher side에 따르면,
- publisher confirm 사용할 경우,
- 메시지 중복이 발생할 수 있음을 설명.

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

## Clustering and Queue Content Replication

- [clusters of nodes](https://www.rabbitmq.com/clustering.html)는 redundancy를 제공하며 단일 노드 장애에 내성을 가짐.
- RabbitMQ 클러스터에서는 모든 요소(익스체인지, 바인딩, 사용자 등)가 전체 클러스터에 복제.
- [quorum queues](https://www.rabbitmq.com/quorum-queues.html)와 [streams](https://www.rabbitmq.com/streams.html)도 여러 클러스터 노드에 복제.
- 한 노드가 리더, 나머지 노드들이 팔로워.
- 리더가 장애이면 팔로워 중 하나가 새로운 리더로 선출.
- 과거에는 [classic queue mirroring](https://www.rabbitmq.com/ha.html)이 큐 리플리케이션의 유일 선택지.
- 하지만 이제는 deprecated 되고 [quorum queues](https://www.rabbitmq.com/ha.html#interstitial)와 [streams](https://www.rabbitmq.com/streams.html) 사용이 권장됨.
- [exclusive queues](https://www.rabbitmq.com/queues.html#exclusive-queues)는 커넥션의 생애주기에 묶이므로,
- 리플리케이션 되지 않으며 노드 재시작에도 살아 남지 못함.
- 연결된 노드의 고장에 대해서는 컨슈머가 평소처럼 복구 절차 밟아야 함.
- 새로운 리더 선출 시, 다른 노드에 연결되어 있던 컨슈머는, RabbitMQ에 의해 자동으로 재등록 됨.
- 이 경우엔 컨슈머들이 재연결이나 재구독 등의 복구를 수행할 필요 없음.

## Data Safety on the Publisher Side

- confirms를 사용할 때,
- 채널이나 커넥션 고장으로부터 복구중인 프로듀서는,
- 브로커로부터 acknowledgement를 받지 못한 메시지를 재전송해야 함.
- 여기서 메시지 중복이 발생할 수 있음.
- 브로커는 confirmation을 보냈는데 프로듀서에게 전달되지 않았기 때문.
- 따라서 컨슈머 애플리케이션은 중복 제거를 하거나,
- 들어온 메시지를 멱등성 있게 처리해야 함.

## Ensuring that Messages are Routed

- 메시지가 큐로 라우팅 됐는지를 보장받는 게, 프로듀서 입장에서 중요할 때가 있음.
- 이미 알고 있는 단일 큐로 라우팅하려면, 프로듀서는 단지 목적지 큐를 선언하고 곧장 발행하면 됨.
- 만약 좀 더 복잡한 상황에서 적어도 1개 이상의 큐로 메시지가 도달했는지 알고 싶으면,
- `basic.publish`에 `mandatory` 플래그를 설정해서,
- 만약 적절한 큐로 바운딩 되지 못했다면 `basic.return`가 클라이언트에 돌아오게 하면 됨.
- 자세한 내용은 [Publishers guide](https://www.rabbitmq.com/publishers.html) 참고.
- 클러스터링 된 노드로 메시지 발행 시에는 미러링 때문에,
- 노드 간 네트워크 장애 시 지연이 발생할 수 있음.
- 자세한 내용은 [inter-node heartbeat guide](https://www.rabbitmq.com/nettick.html) 참고.

## Data Safety on the Consumer Side

- 네트워크나 노드 장애 상황에서, 메시지는 재전송 될 수 있고,
- 컨슈머는 이미 과거에 봤던 메시지를 다시 다루는 일에 대비해야 함.
- 컨슈머 구현은 멱등성을 가지도록 설계하는 것을 권장.
- 만약 메시지가 컨슈머에게 전달된 후 다시 큐에 적재되면,
- RabbitMQ는 `redelivered` 플래그를 설정하고 다시 재전달.
- 메시지를 이전에 봤을 수도 있다고 컨슈머에게 힌트를 주는 것.
- 물론, 당연하게도 이미 봤다는 것이 보장은 안 됨.
- 원래의 메시지 전달이 네트워크나 컨슈머 애플리케이션 문제로 인해 실패했을 수도 있으니.
- 하지만, `redelivered` 플래그가 설정되지 않으면 이 메시지가 이전에 봤던 것이 아님은 보장.
- 따라서, 메시지 중복 제거 비용이 크다면, `redelivered` 플래그가 있는 경우에만 수행하는 것을 고려.

## Unprocessable Deliveries

- 만약 컨슈머가 메시지를 처리할 수 없다면,
- `basic.reject`나 `basic.nack`를 이용해서 거절할 수 있음.
- 이를 받은 서버는 메시지를 재적재 하거나, DLQ로 보내는 등의 처리.
