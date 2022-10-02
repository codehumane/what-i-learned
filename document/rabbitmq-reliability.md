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
