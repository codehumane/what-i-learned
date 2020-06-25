
`@RabbitListener` 애노테이션으로 어떤 것까지 할 수 있을지 알아보기 위해, [Annotation-driven Listener Endpoints](https://docs.spring.io/spring-amqp/reference/html/#async-annotation-driven) 문서를 살펴봄. 기본적인 지식이 있어야 하기에, [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html)도 함께 살펴봄. 여기서는 후자의 문서를 기록.

# AMQP 0-9-1 Model Explained

AMQP(Advanced Message Queuing Protocol)는 RabbitMQ가 사용하는 메시징 프로토콜.

## What is AMQP 0-9-1?

메시지 브로커는 퍼블리셔(프로듀서라고 하기도)로부터 메시지를 받고(receive), 다시 컨슈머에게 라우팅(route). 브로커 안을 좀 더 들여다보면, 메시지는 먼저 익스체인지(exchange)로 보내지고(published), 익스체인지는 바인딩(binding)이라고 불리는 규칙을 사용해서 메시지 복사본을 큐에 분산(distribute). 그림으로 보면 아래와 같음.

### Model in Brief

!["Hello, world" example routing](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)

- 퍼블리셔는 다양한 메시지 퍼블리싱 시에 *메시지 속성(message attributes)*을 지정. 메시지 메타데이터.
- 일부 메타데이터는 브로커가 사용. 하지만 나머지는 완전히 opaque. 메시지 받는 애플리케이션에서만 관심.
- 네트워크 실패나 애플리케이션의 처리 실패 때문에, message acknowledgements 제공. 자동으로 브로커에게 알려줄 수도 있고, 개발자가 직접 결정할 수도 있음. 이 기능을 사용할 때는, 브로커가 통지를 받았을 때에만 큐에서 메시지를 제거함.
- 메시지가 라우팅 되지 않으면, 메시지는 퍼블리셔에게 반환되거나(returned), 제거되거나(dropped), DLQ로 이동되거나.

*queue, exchange, binding을 통칭하여 AMQP 엔티티라고 함.

### Programmable Protocol

- 프로그래밍 가능한 프로토콜
- 엔티티와 라우팅 스키마가 기본적으로 브로커 어드민이 아닌 애플리케이션에 의해 정의됨.
- 큐와 익스체인지를 선언하고, 이들 간의 바인딩을 정의하고, 큐 구독 등을 진행.
- 높은 자유도를 주긴 하지만 실수나 설정 충돌 등의 가능성을 가짐.

## Exchanges and Exchange Types

- 익스체인지는 AMQP 0-9-1의 엔티티이며,
- 메시지를 받은 뒤 0개 이상의 큐에 라우팅.
- 라우팅 알고리즘은 익스체인지 타입과 바인딩이라고 불리는 규칙에 따라 다름.
- AMQP 0-9-1 브로커는 아래의 4가지 익스체인지 타입을 제공.

| Exchange type    | Default pre-declared names              |
| :--------------- | :-------------------------------------- |
| Direct exchange  | (Empty string) and amq.direct           |
| Fanout exchange  | amq.fanout                              |
| Topic exchange   | amq.topic                               |
| Headers exchange | amq.match (and amq.headers in RabbitMQ) |

익스체인지 타입 외에도, 몇 가지 중요한 속성들이 있음.

- Name
- Durability (exchanges survive broker restart)
- Auto-delete (exchange is deleted when last queue is unbound from it)
- Arguments (optional, used by plugins and broker-specific features)

### Default Exchange

- 기본 익스체인지는 이름이 없는(빈문자열) direct 타입.
- 브로커에 의해 미리 선언되어 있음.
- 생성되는 모든 큐는 자동으로 이 익스체인지에 바운딩.
- 이 때의 라우팅 키는 큐의 이름과 동일함.
- 단순한 애플리케이션에 유용.

### Direct Exchange

- 메시지를 라우팅 키에 기반해서 큐에 전달.
- 메시지의 unicast 라우팅에 이상적.
- 큐는 라우팅 키 K와 함께 익스체인지에 바운딩.
- 라우팅 키 R과 함께 새로운 메시지가 direct 익스체인지에 도착하면,
- 익스체인지는 K = R 여부를 판단하고 큐로 이 메시지를 라우팅.
- 그리고 다수의 워커들(동일 애플리케이션의 인스턴스들)이 있으면,
- 작업(task)을 라운드 로빈 방식으로 분산시켜 줌.
- 큐가 아닌 컨슈머들에 대한 로드 밸런싱임에 유의.

![direct exchange](https://www.rabbitmq.com/img/tutorials/intro/exchange-direct.png)

### Fanout Exchange

- 메시지를 바운딩 된 모든 큐에 라우팅.
- 라우팅 키는 무시됨.
- N개의 큐가 fanout 익스체인지에 바운딩 되어 있으면,
- 새로운 메시지가 도착했을 때 이 메시지의 복제본을 모든 N개의 큐에 전달.
- 메시지의 broadcast 라우팅에 유리.

![fanout exchange](https://www.rabbitmq.com/img/tutorials/intro/exchange-fanout.png)

### Topic Exchange

- 1개 이상의 큐에 메시지 라우팅.
- 단, if '큐가 익스체인지에 바운딩 될 때 사용된 패턴' match '메시지의 라우팅 키'.
- 마찬가지로, 메시지의 multicast 라우팅에 주로 사용.

### Headers Exchange

- 라우팅 키 대신, 메시지 헤더에 있는 속성들을 기반으로 라우팅.
- 라우팅 키는 무시됨.
- 바인딩 시 지정된 값과 헤더의 값을 매칭하여 라우팅.
- `x-match` 인자의 값이 `all`일 때는 모든 헤더를 매칭하고,
- `x-match` 인자의 값이 `any`일 때는 1개 헤더만 매칭.
- 라우팅 키가 문자열일 필요가 없는(정수형이거나 딕셔너리 같은) 곳에서 direct 익스체인지로 사용될 수 있음.
- 참고로, `x-`로 시작하는 헤더는 매치에 사용되지 않음.
- [여기 예시](https://codedestine.com/rabbitmq-headers-exchange/)가 이해를 도움.
- 애플리케이션에서 해줘야 하는 일이 늘어나는 한편, 좀 더 자유로운 라우팅이 가능.

## Queues

메시지를 저장하고 애플리케이션에 의해 소비. 큐는 익스체인지와 프로퍼티 일부를 공유하지만, 부가적인 속성도 가지고 있음.

- Name
- Durable (the queue will survive a broker restart)
- Exclusive (used by only one connection and the queue will be deleted when that connection closes)
- Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes)
- Arguments (optional; used by plugins and broker-specific features such as message TTL, queue length limit, etc)

큐가 사용되려면 먼저 선언되어야 함. 큐가 선언되면, 이미 있는 이름의 큐인지를 확인 후 없으면 새로 생성. 이미 존재하는 이름이고 속성들이 같으면 아무 일도 일어나지 않음. 이름은 같은데 속성이 다르면 406 코드(`PRECONDITION_FAILED`)와 함께 채널 레벨의 예외가 발생함.

### Queue Names

TBD
