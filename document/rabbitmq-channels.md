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

### Channels and Error Handling

- 애플리케이션에 의한 채널 닫힘 외에, 프로토컬 에외의 의한 닫힘도 있음.
- 몇 가지 시나리오는 회복 가능한("soft") 에러로 간주.
- 채널은 닫혔지만 애플리케이션은 다른 것을 열고 복구나 재시도를 할 수 있음.

```
Redeclaring an existing queue or exchange with non-matching properties will fail with a 406 PRECONDITION_FAILED error
Accessing a resource the user is not allowed to access will fail with a 403 ACCESS_REFUSED error
Binding a non-existing queue or a non-existing exchange will fail with a 404 NOT_FOUND error
Consuming from a queue that does not exist will fail with a 404 NOT_FOUND error
Publishing to an exchange that does not exist will fail with a 404 NOT_FOUND error
Accessing an exclusive queue from a connection other than its declaring one will fail with a 405 RESOURCE_LOCKED
```

- 클라이언트 라이브러리는 [예외 핸들러 등록](https://www.rabbitmq.com/client-libraries/java-api-guide#shutdown)을 통해 채널 예외를 관찰하고 반응할 수 있게 도와줌.
- 닫힌 채널에 대한 연산 시도는 예외를 맞음.
- RabbitMQ는 채널을 닫을 때, 클라이언트에게 비동기 프로토콜 메서드로 통지.
- 따라서, 연산에 대한 닫힘 예외를 즉시 만나지 않고, 조금 뒤에 닫힘 이벤트 핸들러에서 일어날 수 있음.
- 어떤 라이브러리들은 연산 응답을 동기적으로 기다림.
- 이 경우에는 채널 예외를 조금 다르게 처리해야 할 수도.

## Resource Usage

- 각 채널은 클라이언트에서 비교적 적은 메모리를 사용.
- 라이브러리 구현에 따라 다르지만, 전용 스레드 풀을 사용해서 컨슈머 연산들을 분배할 수도 있음.
- 클라이언트가 연결된 노드에서도 상대적으로 적은 메모리와 소수의 Erlang 프로세스를 사용.
- 노드가 여러 채널 커넥션을 지원하므로, 과도한 채널 사용이나 채널 누수의 영향은 주로 클라이언트가 아니라 RabbitMQ 노드 메트릭에 나타남.
- 이들 요소를 고려할 때 커넥션 당 채널 수의 제한은 매우 권장됨.
- 대부분의 애플리케이션들은 커넥션 당 한 자리 수의 채널을 유지하길 가이드.

### Maximum Number of Channels per Connection

- 커넥션 당 동시에 열릴 수 있는 채널 수는 서버와 클라이언트의 연결 시점에 협의됨.
- RabbitMQ와 클라이언트 모두에 설정 가능.
- 서버 사이드에서는 `channel_max`로 제한.

```
# no more 100 channels can be opened on a connection at the same time
channel_max = 100
```

- 이 수치를 넘으면 커넥션은 에러와 함께 닫힘.
- 클라이언트는 연결 당 더 적은 수로 설정.
- 이 수치를 넘으면 오류를 만나게 됨.

```java
ConnectionFactory cf = new ConnectionFactory();
// Ask for up to 32 channels per connection. Will have an effect as long as the server is configured
// to use a higher limit, otherwise the server's limit will be used.
cf.setRequestedChannelMax(32);
```
