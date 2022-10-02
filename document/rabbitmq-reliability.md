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
