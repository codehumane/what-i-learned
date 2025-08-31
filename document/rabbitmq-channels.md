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

- 일부 애플리케이션은 브로커에 대한 다수의 논리적 연결이 필요.
- 하지만, 많은 TCP 연결을 동시에 열어 두지 못할 수도.
- 이는 많은 리소스를 소비하고, 방화벽 설정도 어렵게 만들기 때문.
- AMQP 0-9-1은 채널을 통해 연결을 멀티플렉싱.
- "단일 TCP 여결을 공유하는 경량 연결"로 생각하면 됨.
- 모든 프로토콜 연산은 클라이언트의 단일 채널에서 수행.
- 한 채널에서의 통신은 다른 채널과는 완전히 분리되어 있음.
- 이를 위해 모든 프로토콜 메서드는 채널은 채널 ID를 수반.
- 채널은 연결 맥락에서만 존재. 연결이 닫히면 채널도 마찬가지.
