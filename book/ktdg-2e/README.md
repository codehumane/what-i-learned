
# 카프카 핵심 가이드 2판

# 4장. 카프카 컨슈머: 카프카에서 데이터 읽기

## 4.1 카프카 컨슈머: 개념

책에서는 개념으로 컨슈머 그룹, 리밸런싱, 정적 그룹 멤버십 이야기. 그런데, 컨슈머 자체에 개념은 더 없나 궁금해서 [여기](https://docs.confluent.io/platform/current/clients/consumer.html) 내용 간단 기록.

- 컨슈머는 "fetch" 요청을 브로커에게 보냄.
- 각 요청 로그에는 컨슈머의 오프셋이 명시 되어 있음.
- 컨슈머는 오프셋부터 시작한 토픽의 모든 메시지를 청크 로그로 응답 받음.
- 컨슈머는 이 포지션에 대한 막대한 권한을 가지며, 필요할 경우 되감기로 데이터를 다시 소비할 수도.

### 4.1.1 컨슈머와 컨슈머 그룹

- 확장<sup>scale</sup> 목적.
- 토픽을 파티션으로 나누고, 각 파티션에 컨슈머 할당.
- 토픽 생성 시 파티션 수를 크게 잡아주면 확장에 유리.
- 참고로, 파티션과 컨슈머의 관계는 N:1.
- 만약, 파티션보다 컨슈머 수가 더 많으면 유휴 컨슈머 발생.
- 여기서 나아가 토픽에 대해 컨슈머 그룹 자체를 추가할 수도.
- 이쯤 되면 브로커의 push가 아니라 컨슈머의 pull 임을 예상할 수 있음.
- [아파치 카프카 문서](https://kafka.apache.org/documentation/#design_pull) 보면 pull이라고 나와 있음.

> data is pushed to the broker from the producer and pulled from the broker by the consumer. ... 중략 ... However, a push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. The goal is generally for the consumer to be able to consume at the maximum possible rate; unfortunately, in a push system this means the consumer tends to be overwhelmed when its rate of consumption falls below the rate of production.

- 폴링이 문제가 되어 푸시로 바꾸는 경우도 많지만, 카프카에서는 위 이유로 pull이며, 언제나 트레이드 오프.
- 그 외에 컨슈머가 자신의 상황에 맞게 (문서에서는 공격적이라 표현) 데이터를 일괄 처리할 수 있음. 퍼블리셔가 지금 메시지를 즉각 보내야 할지, 기다렸다가 모아서 보내야 할지 판단하기는 어려우며, 이는 지연으로 이어질 수 있음.
- 한편, 가져갈 데이터가 없는 데 자꾸 폴링하는 건 비용이라, 데이터가 도착할 때까지 기다리게 하는 파라미터를 함께 제공.
- 토픽, 컨슈머, 컨슈머 그룹의 관계 그림은 [여기](https://medium.com/javarevisited/kafka-partitions-and-consumer-groups-in-6-mins-9e0e336c6c00) 잘 나와 있음.

### 4.1.2 컨슈머 그룹과 파티션 리밸런스

- 리밸런싱이란, 컨슈머에 할당된 파티션을 다른 컨슈머에게 할당해주는 작업.
- 그룹에 컨슈머 추가나 삭제, 혹은 운영자가 토픽에 새로운 파티션을 추가한 경우 발생.
- high availability, scalability가 목적.
- 하지만 달갑지 않은 부분도 존재.

#### 조급한<sup>eager</sup> 리밸런스

책보다는 [여기](https://www.confluent.io/blog/debug-apache-kafka-pt-3/) 설명이 더 나음. 그리고 기억하기 더 쉽게 Stop-the-World라 부르고 있음. 전체 과정은 아래와 같음.

![](https://images.ctfassets.net/8vofjvai1hpv/Pfql9YfYW5ILCJK3Avotd/38032f581fa4c6b95544ed0d1dcde626/stop-the-world-rebalancing.png)

1. 이미 그룹에 컨슈머들이 읽기를 하고 있었다고 가정.
2. 새로운 컨슈머가 토픽에 대한 JoinGroup 요청을 그룹 코디네이터에게 전달.
3. 코디네이터는 현재의 모든 멤버들에게 JoinGroup 요청을 보내라면서 리밸런싱을 시작.
4. 이 명령은 heartbeat 응답에 담겨 컨슈머들에게 전달됨.
5. 멤버들은 하던 처리를 마무리하고 코디네이터에게 JoinGroup을 보내며 stop the world.
6. 코디네이터는 그룹에 속한 멤버들과 그룹이 참여한 토픽들을 알게 되고, 코디네이터는 멤버들에게 JoinResponse을 보내며, [리더](https://stackoverflow.com/questions/42015158/what-is-the-difference-in-kafka-between-a-consumer-group-coordinator-and-a-consu)를 선출해 파티션 할당을 위임.
7. 모든 멤버는 이에 대한 반응으로 SyncGroup 요청을 보내고, 여기에 그룹 리더는 파티션 할당을 담아서 보냄.
8. 코디네이터는 SyncResponse를 컨슈머에게 보내며 토픽 파티션 할당을 확정.
9.  컨슈머들은 할당을 받아들이고 읽기 작업을 재개. stop the world는 종료.

#### 협력적<sup>cooperative</sup> 리밸런스

![](https://cdn.confluent.io/wp-content/uploads/cooperative_rebalancing_protocol-1536x716.png)

incremental rebalance라고도 부름. 조급한 리밸런스에서의 1~6 단계까지는 동일. 그 이후가 다르며 아래와 같음.

1. 조급한 리밸런스에서의 1~4 단계를 동일하게 진행.
2. 멤버들은 하던 처리를 마무리하고 코디네이터에게 JoinGroup 요청을 보내는데, 이 때 새로운 할당을 기다리며 stop the world를 하는 대신, 컨슈머들은 추가적인 통지가 있을 때까지 기존에 할당된 토픽 파티션 처리를 계속 진행.
3. JoinGroup 요청을 받은 코디네이터는 그룹에 속한 멤버들과 그룹이 참여한 토픽들을 알게 되고, 코디네이터는 JoinResponse를 멤버들에게 보내며, 리더를 선출해 파티션 할당을 위임.
4. 모든 멤버는 이에 대한 반응으로 SyncGroup 요청을 보내고, 여기에 그룹 리더는 파티션 할당을 담아서 보냄.
5. 여기서 코디네이터는 새로운 할당과 과거의 할당을 비교해 변경이 필요한 컨슈머에게만 SyncResponse 응답.
6. 그리고 영향을 받는 컨슈머만 리밸런싱을 위해 중단.
7. 이후의 과정 설명은 생략 된 것 같고, 위 이미지를 참고해서 해석함.

그룹에 속한 컨슈머가 많을 경우 리밸런싱이 오랜 시간 소요될 수 있음. 이 때 점진적 협력적 리밸런싱으로 전체 중단을 막을 수 있음.

### 4.1.3 정적 그룹 멤버십

- 컨슈머가 그룹을 떠나면, 혹은 다시 참여하면 리밸런싱이 기본.
- 그러나 컨슈머에 `group.instance.id`를 할당하면 정적 멤버가 됨.
- 떠났다가 다시 돌아오더라도, `session.timeout.ms` 시간을 넘지 않았다면, 이전 파티션 할당.
- 책에서는, 파티션 데이터로 로컬 상태나 캐시를 구축해야 하고, 컨슈머 재시작으로 이 작업을 반복하고 싶지 않을 때 유용하다고 함.
- [Dynamic vs. Static Consumer Membership in Apache Kafka](https://www.confluent.io/blog/dynamic-vs-static-kafka-consumer-rebalancing/)에서는 롤링 업그레이드 환경을 위한 것이라고 함.
- 리밸런싱이 컨슈머 수의 2배 만큼 발생하므로, 롤링 업그레이드 시간이 오래 걸림.

## 4.2 카프카 컨슈머 생성하기

```java
// create consumer configs
final Properties properties = new Properties();
properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "broker1:9092,broker2:9092");
properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "groupId");
properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

final Consumer<String, String> consumer =
    new KafkaConsumer<>(properties);
```
