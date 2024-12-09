
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
