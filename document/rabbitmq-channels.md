# Channels

https://www.rabbitmq.com/docs/channels

## Overview

채널은 커넥션 없이는 존재할 수 없음. [커넥션 가이드](https://www.rabbitmq.com/docs/connections) 먼저 익숙해지기를 권장. 여기서는 아래 내용 다룸.

- The basics of channels
- Channel lifecycle
- Channel exceptions (errors) and what they mean
- Channel resource usage
- Monitoring and metrics related to channels and how to identify common problems
- Flow control

## The Basics

- 일부 애플리케이션은 브로커에 대한 다수의 논리적 커넥션이 필요.
- 하지만, 많은 TCP 커넥션을 동시에 열어 두지 못할 수도.
- 이는 많은 리소스를 소비하고, 방화벽 설정도 어렵게 만들기 때문.
- AMQP 0-9-1은 채널을 통해 커넥션을 멀티플렉싱.
- "단일 TCP 여결을 공유하는 경량 커넥션"로 생각하면 됨.
- 모든 프로토콜 연산은 클라이언트의 단일 채널에서 수행.
- 한 채널에서의 통신은 다른 채널과는 완전히 분리되어 있음.
- 이를 위해 모든 프로토콜 메서드는 채널은 채널 ID를 수반.
- 채널은 커넥션 맥락에서만 존재. 커넥션이 닫히면 채널도 마찬가지.

## Channel Lifecycles

### Opening Channels

- 애플리케이션은 커넥션을 성공적으로 맺으면 바로 채널을 연다.
- 아래는 커넥션 맺고 채널까지 생성하며, 자동으로 할당된 채널 ID를 부여 받는 과정.

```java
ConnectionFactory cf = new ConnectionFactory();
Connection conn = cf.createConnection();

Channel ch = conn.createChannel();
// ... use the channel to declare topology, publish, consume
```

- 채널도 커넥션과 마찬가지로 오래 유지되기를 기대.
- 즉, 매 작업마다 채널을 여는 건 비효율.
- 채널을 여는 작업은 네트워크 왕복이 발생하기 때문.

### Closing Channels

- 채널이 필요하지 않으면 닫아야 함.
- 채널을 닫으면 사용할 수 없게 되고 채널 리소스 반환이 예약됨.

```java
Channel ch = conn.createChannel();
// do some work

// close the channel when it is no longer needed
ch.close();
```

- 채널의 커넥션이 닫혀도 채널은 닫힘.
- 닫힌 채널에 대한 연산 시도는 예외를 맞게 됨.
- 컨슈머가 [ack](https://www.rabbitmq.com/docs/confirms)를 응답하고 채널을 즉시 닫으면,
- ack가 큐에 전달되지 않을 수도 있음.
- (아마도 ack 전달이 비동기라서 그럴 것으로 추정)
- pending ack는 큐에 자동으로 재적재 될 수 있음.
- 이 현상은 수명이 짧은 채널에서 주로 발생.
- 수명이 긴 채널을 사용하고, 소비자들이 재전달에도 대응할 수 있게 설계하면, 위의 현상을 줄일 수 있음.
- 긴 수명의 채널은 더 나은 성능에도 영향.
- 재전달된 메시지는 [명시적으로 표시](https://www.rabbitmq.com/docs/consumers#message-properties) 된다는 점 참고.
