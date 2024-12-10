
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

final Consumer<String, String> consumer =
    new KafkaConsumer<>(properties);
```

- 위 4가지 프로퍼티는 필수.

## 4.3 토픽 구독하기

```
consumer.subscribe(Arrays.asList("accountTopic"));
```

- 여기서 토픽을 정규식으로 지정할 수도 있음.
- 정규식과 매칭되는 새로운 토픽이 생성되면 거의 즉시 리밸런싱.
- 다만, 토픽 필터링은 클라이언트 작업임을 유의해야 함.
- 정규식 지정 시 컨슈머는 전체 토픽과 파티션 정보를 브로커에 일정 간격으로 요청.
- 이 때, 토픽이나 파티션이나 컨슈머가 많으면, 브로커와 클라이언트 그리고 네트워크 전체에 상당한 오버헤드.

## 4.4 폴링 루프

- 컨슈머 API 핵심은 서버에 축 데이터가 들어왔는지 폴링하는 단순한 루프.
- spring-kafka의 [KafkaMessageListenerContainer](https://github.com/spring-projects/spring-kafka/blob/main/spring-kafka/src/main/java/org/springframework/kafka/listener/KafkaMessageListenerContainer.java#L1311)를 봐도 뼈대는 동일.
- [1684L](https://github.com/spring-projects/spring-kafka/blob/main/spring-kafka/src/main/java/org/springframework/kafka/listener/KafkaMessageListenerContainer.java#L1684)도 함께 보면 됨.

```kt
while (true) {
    val timeout = Duration.ofMillis(100)
    val records: ConsumerRecords<String, String> = consumer.poll(timeout)

    records.forEach {

        println(
            """
                topic = ${it.topic()}
                partition = ${it.partition()}
                offset = ${it.offset()}
                key = ${it.key()}
                value = ${it.value()}
            """
        )

    }
}
```

- 정해진 시간 내에 반복적으로 `poll`을 호출해야 살아 있는 컨슈머로 간주되고 그렇지 않으면 리밸런싱.
- `poll`에 넘겨진 매개변수는, 컨슈머 버퍼에 데이터가 없을 때 얼마나 블럭될 것인가를 결정.
- `poll`은 레코드들이 저장된 목록을 반환.
- 책에서는 List라고 되어 있으나 [코드](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/ConsumerRecords.java) 보면 아래와 같이 Iterable.

```java
public class ConsumerRecords<K, V> implements Iterable<ConsumerRecord<K, V>> {
    public static final ConsumerRecords<Object, Object> EMPTY = new ConsumerRecords<>(Collections.emptyMap());

    private final Map<TopicPartition, List<ConsumerRecord<K, V>>> records;

    ... 중략

    @Override
    public Iterator<ConsumerRecord<K, V>> iterator() {
        return new ConcatenatedIterable<>(records.values()).iterator();
    }
```

- `poll`은 데이터를 가져오는 것 외에도 여러가지 일을 함.
- 컨슈머가 GroupCoordinator를 찾아 컨슈머 그룹에 참가하고 파티션 할당 받음.
- 리밸런스 역시 관련 콜백들과 함께 여기서 처리됨.
- 따라서, 컨슈머나 콜백에서 뭔가 잘못된 것들은 전부 `poll`에서 예외로 던져짐.
- `max.poll.interval.ms`에 지정된 시간보다 오랫동안 `poll`이 호출해야 살아 있다고 간주됨.
- 그래서 폴링 루프 안에서 예측 불가능한 시간 동안 블럭되는 작업을 수행하는 건 위험.
- `KafkaConsumer#poll` 주석을 보면 [아래 설명들](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/KafkaConsumer.java#L1168) 적혀 있음.
  - 두 번째 문단 마지막에 마침표 누락 @_@
  - 컨르톨 레코드가 뭔가 하니, 트랜잭션을 사용할 때 begin/end/abort 등의 메시지라고 함.

> Fetch data for the topics or partitions specified using one of the subscribe/ assign APIs. It is an error to not have subscribed to any topics or partitions before polling for data.
> 
> On each poll, consumer will try to use the last consumed offset as the starting offset and fetch sequentially. The last consumed offset can be manually set through seek(TopicPartition, long) or automatically set as the last committed offset for the subscribed list of partitions
> 
> This method returns immediately if there are records available or if the position advances past control records or aborted transactions when isolation. level=read_committed. Otherwise, it will await the passed timeout. If the timeout expires, an empty record set will be returned. Note that this method may block beyond the timeout in order to execute custom ConsumerRebalanceListener callbacks.

## 4.6 오프셋과 커밋

- 앞서 봤듯 `poll()`은 아직 읽지 않은 레코드를 반환.
- 컨슈머가 아직 읽지 않은 메시지인지 판단할 때 오프셋을 활용.
- 각 파티션 메시지를 고유하게 식별하는 증분 ID가 오프셋.
- [Kafka Offset Explained](https://kontext.tech/diagram/1159/kafka-offset-explained)에 나온 그림과 같이 컨슈머와 프로듀서 모두 파티션 별 오프셋이 있음.

![](https://kontext.tech/api/flex/diagram/diagram-1159)

- 다른 JMS 큐들과 달리 카프카는 컨슈머가 데이터를 읽어 들임.
- 컨슈머가 파티션의 메시지를 어디까지 읽어 들였는지 기록하는 게 오프셋 커밋.
- 컨슈머가 파티션에 할당 받으면 마지막으로 커밋된 오프셋으로부터 데이터 읽어 들임.
- 만약, 파티션을 컨슈머 그룹이 처음 읽는 거라면 `auto.offset.reset` 설정에 따라 행동.
- [Kafka Documentation의 auto.offset.reset](https://kafka.apache.org/documentation/#consumerconfigs_auto.offset.reset) 함께 참고.
- 오프셋 커밋은 전통적 메시지 큐와 달리 개별 레코드 커밋이 아니라 처리한 메시지의 마지막 위치를 커밋.
- 그 앞의 메시지들은 모두 성공적으로 처리한 것으로 간주.
- 오프셋 커밋은 카프카의 특수 토픽인 `__consumer_offsets`에 기록.
- 만약 리밸런싱이 일어나 파티션에 다른 컨슈머가 할당되면 여기서 오프셋을 읽고 처리 재개.
- `실제 처리한 메시지 오프셋 > 커밋된 메시지 오프셋`이면 이 사이의 메시지들은 중복 처리.
- `실제 처리한 메시지 오프셋 < 커밋된 메시지 오프셋`이면 이 사이의 메시지들은 처리 누락.

![](https://newrelic.com/sites/default/files/wp_blog_inline_files/events_lost2-1024x547.jpg)

![](https://newrelic.com/sites/default/files/wp_blog_inline_files/events_duplicated2.jpg)

- 참고로, `poll`에서 받은 마지막 메시지의 오프셋이 아니라, 실제로는 그 다음 오프셋을 커밋한다고 함.
- 그래서 수동 오프셋 커밋 등에 유의해야 함.

### 4.6.1 자동 커밋

- `enable.auto.commit`을 설정하면, poll을 통해 받은 마지막 메시지의 오프셋을 주기적으로 자동 커밋.
- `auto.commit.interval.ms`를 함께 설정해서, 오토 커밋을 얼마나 자주 일으킬지 결정(기본 5s).
- 참고로, [CONFLUENT 문서](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiAx9q6BhCDARIsACwUxu46n05vuk2S2xtJdZ2tf0TgZpzs2zRmTYldmZ0ej1GSfwALOjt0VBIaAsmIEALw_wcB#enable-auto-commit)에서는 백그라운드에서 주기적 커밋이라고 되어 있음.
- 그리고 책에서는 매 `poll` 마다 필요 여부 판단 후 오프셋 커밋한다고 함.
- 자동 커밋 사용 시, 주기적으로 커밋되므로, 리밸런싱이 일어난 뒤 메시지 중복 발생 가능.
- 주기를 아무리 줄여도 중복을 완전히 제거할 순 없음.
- 또한, 다음 `poll`에서 커밋이 일어날 수 있으니, 이전에 받은 메시지 처리를 그 전에 끝내야 함.
- 자동 커밋은 편리하지만 중복 트레이드오프.

### 4.6.2 현재 오프셋 커밋하기

- 대부분의 개발자들은 오프셋 커밋을 직접 제어하려 한다고 함.
- 유실과 중복 가능성을 제거하기 위해.
- 이를 위해 우선 `enable.auto.commit=false` 설정.
- 그리고 `commitSync()` API 활용.
- 여기서는 어떤 오프셋을 커밋할지 명시하지 않고 있으며,
- poll에서 리턴된 마지막 메시지를 커밋함에 유의.
- 뒤에서 파라미터 지정하는 이야기도 다룸.
- [KafkaConsumer](https://github.com/apache/kafka/blob/trunk/clients/src/main/java/org/apache/kafka/clients/consumer/KafkaConsumer.java) 코드에 나온 예시는 아래와 같음(코틀린 버전으로 바꿈).

```kt
val props = Properties().apply {
    setProperty("bootstrap. servers", "localhost:9092")
    setProperty("group. id", "test")
    setProperty("enable. auto. commit", "true")
    setProperty("auto. commit. interval. ms", "1000")
    setProperty("key. deserializer", "org. apache. kafka. common. serialization. StringDeserializer")
    setProperty("value. deserializer", "org. apache. kafka. common. serialization. StringDeserializer")
}

val consumer = KafkaConsumer<String, String>(props)
consumer.subscribe(mutableListOf("foo", "bar"))

while (true) {
    val records = consumer.poll(Duration.ofMillis(100))

    for (record in records) {
        println("offset = ${record.offset()}, key = ${record.key()}, value = ${record.value()}")
    }
}
```

- `commitSync`에 오프셋을 넘기는 예제도 있음.

```kt
try {
    while (isRunning) {
        val records = consumer.poll(Duration.ofMillis(Long.MAX_VALUE))

        for (partition in records.partitions()) {
            val records = records.records(partition)

            for (record in records) {
                println(record.offset().toString() + ": " + record.value())
            }

            val lastOffset = records[records.size - 1].offset()
            val commitOffset = OffsetAndMetadata(lastOffset + 1)
            consumer.commitSync(Collections.singletonMap(partition, commitOffset))
        }
    }
} finally {
    consumer.close()
}
```

### 4.6.3 비동기적 커밋

- 수동 커밋 단점 중 하나가 커밋 일어나는 동안의 애플리케이션 블럭킹.
- 물론 덜 자주 커밋해서 처리량을 올릴 순 있음. 그러나 리밸런싱에 의한 메시지 중복은 더 커짐.
- 또 하나의 방법은 `commitAsync()` API를 이용한 비동기 커밋.
- 요청만 보내고 응답은 기다리지 않고 다른 처리를 이어감.
- 다만, 비동기 커밋은 동기 커밋과 달리 재시도가 없음.
- 순서 꼬임 문제가 발생할 수 있기 때문.
- 만약, 비동기 커밋 시 지정하는 콜백에서 재시도 유혹이 든다면 많은 고민이 필요.
- 재시도 시 현재 커밋을 불러와 비교하는 것도 방법.

### 4.6.4 동기적 커밋과 비동기적 커밋을 함께 사용하기

- 재시도 없는 커밋이 간혹 실패한다고 큰 문제가 되진 않음.
- 뒤이은 커밋이 결국엔 일어나기 때문.
- 그러나 컨슈머 종료 전, 혹은 리밸런싱 전이라면 다를 수도.
- 그래서 일반적인 경우엔 비동기 커밋을 사용하고, 예외가 발생해 컨슈머를 종료해야 하는 경우엔 동기적 커밋.

```kt
try {
    while (running) {
        consumer.poll(duration)
        ...
        consumer.commitAsync()
    }
} catch(e: Exception) {
    consumer.commitSync()
} finally {
    consumer.close()
}
```

## 4.7 리밸런스 리스너

- 컨슈머 종료나 리밸런싱 전에 정리 작업 필요할 수도.
- 이를 위해 컨슈머 `subscribe()`에 `ConsumerRebalanceListener`를 전달.
  - `onPartitionsAssigned`
  - `onPartitionsRevoked`
  - `onPartitionsLost` (협력적 리밸런싱에서 예외적 상황에만 호출, 일반적으로는 revoked 호출)
- 책에서는 `onPartitionsRevoked`에서 파티션 해제 전 마지막 오프셋 커밋하는 예제 소개함.

## 4.8 특정 오프셋의 레코드 읽어오기

- 파티션의 원하는 오프셋부터 데이터 읽어들이기도 가능.
- `seekToBeginning(Collection<TopicPartition> partitions)`: 파티션 처음부터 전부 읽기
- `seekToEnd(Collection<TopicPartition> partitions)`: 파티션 마지막부터 새로 읽기
- `seek`로 특정 오프셋을 지정할 수도 있음.
- API 문서 설명은 아래와 같음.
- 다음 `poll` 동작 시 오프셋이 반영되는 것.

> Overrides the fetch offsets that the consumer will use on the next poll(timeout). If this API is invoked for the same partition more than once, the latest offset will be used on the next poll(). Note that you may lose data if this API is arbitrarily used in the middle of consumption, to reset the fetch offsets

- 오프셋 변경은 아래 같은 경우에 활용 고려.
- 애플리케이션에 문제가 있어 시간에 민감한 데이터 처리가 늦어져서 건너뛰어야 할 때.
- 컨슈머 측 데이터 유실로 전체 복구를 해야 할 때.

## 4.9 폴링 루프를 벗어나는 방법

- 컨슈머는 무한 루프(스프링 KafkaMessageListenerContainer에서는 isRunning을 따지긴 함).
- 만약 컨슈머를 종료하고자 한다면, 다른 스레드에서 `consumer.wakeup()`을 호출하면 됨.
- 이 때 컨슈머가 `poll()`에서 블럭킹 되어 있다면 `WakeupException`을 발생시키며 즉각 탈출.
- 혹은 메시지를 처리 중이라면 다음 `poll()`에서 예외 발생.
- 이 예외를 받고 스레드를 종료하기 전 `consumer.close()` 호출해야 함.
- 그러면, 코디네이터에게 그룹을 떠난다는 메시지를 전송하고 필요 시 오프셋 커밋.
- `KafkaConsumer#close(Duration timeout)`를 보면 아래와 같이 설명.

> Tries to close the consumer cleanly within the specified timeout. This method waits up to timeout for the consumer to complete pending commits and leave the group. If auto-commit is enabled, this will commit the current offsets if possible within the timeout. If the consumer is unable to complete offset commits and gracefully leave the group before the timeout expires, the consumer is force closed. Note that wakeup() cannot be used to interrupt close.
