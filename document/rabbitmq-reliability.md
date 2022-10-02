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
