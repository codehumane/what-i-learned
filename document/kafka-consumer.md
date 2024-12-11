
# 카프카 컨슈머

## 기본 개념

https://www.confluent.io/ko-kr/blog/tutorial-getting-started-with-the-new-apache-kafka-0-9-consumer-client/

### 파티션과 컨슈머 그룹

- 토픽은 파티션이라 불리는 로그 집합으로 나뉨.
- 프로듀서는 로그들의 꼬리에 쓰고, 컨슈머는 자신의 속도에 맞게 읽음.
- 카프카는 파티션을 컨슈머 그룹에 분배함으로써 토픽 소비를 스케일링.
- 컨슈머 그룹이란, 동일한 `group.id`를 공유하는 컨슈머 집합.

![](https://cdn.confluent.io/wp-content/uploads/2016/08/New_Consumer_figure_1.png)

### 읽기

- 그룹이 처음 만들어지면, 컨슈머들은 각 파티션의 earliest 또는 latest 오프셋부터 읽기를 시작.
- 이후에는 각 파티션 로그를 순차적으로 읽음.
- 그리고 컨슈머는 성공적으로 처리한 메시지의 오프셋을 커밋.
- 아래 그림에서 컨슈머는 6번까지 읽었고 1번까지 커밋.

![](https://cdn.confluent.io/wp-content/uploads/2016/08/New_Consumer_Figure_2.png)

- 리밸런싱 이후 컨슈머가 새로운 파티션에 할당되면, 마지막으로 커밋된 파티션의 오프셋부터 읽어 들임.
- 위 그림 상태라면, 1번부터 읽어 들이게 되고, 6번까지 중복으로 재처리가 일어남.
- 참고로, 맨 우측 마지막 오프셋은 가장 최근 쓰여진 메시지.
- high watermark는 레플리카에 복제가 성공한 마지막 로그.
- 컨슈머는 high watermark까지만 데이터를 읽을 수 있음에 유의.

### 그룹 코디네이터

- 그룹 관리를 위해서는 카프카에 내장된 그룹 코디네이션 프로토콜 사용.
- 각 그룹에 대해 브로커 중 하나가 그룹 코디네이터로 선정.
- 코디네이터는 그룹의 상태를 관리하는 책임을 가짐(e.g. 컨슈머 리더 선정).

![](https://images.ctfassets.net/gt6dp23g0g38/6smQqDfuZgjNPFeB7iNHLb/d237de3390dcb0cada914c06f4142053/Kafka_Internals_065.png)

- 각 코디네이터는 `__consumer_offsets`라는 내부 토픽을 두고 여기에 그룹 메타데이터 관리.
- 예를 들어, 현재 오프셋 포지션이 저장되고, 리밸런싱 후 컨슈머들이 여기서 오프셋을 가져감.
- 자세한 내용은 [Consumer Group Protocol](https://developer.confluent.io/courses/architecture/consumer-group-protocol/)에 잘 나와 있음.

## 핵심 코드

### 폴링 루프

컨슈머 API 핵심은 서버에 데이터가 들어왔는지 폴링하는 단순한 루프.

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

`poll` 내부에서는 많은 일이 일어나므로 잠깐 설명.

- `poll`은 저장된 레코드 목록(`Iterable`)을 반환.
- 또한, `poll` 호출 시 그룹 코디네이터를 찾아 컨슈머 그룹에 참여하며 파티션 할당 받음.
- `poll`에 넘겨진 매개변수 = 컨슈머 버퍼에 데이터가 없을 때 얼마나 블럭킹 할 것인가.
- 리밸런싱 등의 콜백들도 여기서 처리되며, 콜백에서 잘못되면 `poll`에서 예외로 던져짐.
- 정해진 시간(`max.poll.interval.ms`) 내에 반복적으로 `poll` 호출해야, 살아 있는 컨슈머로 간주되고 리밸런싱 피할 수 있음.
- 따라서, 폴링 루프 안에서 일어나는 작업 시간이 예측 불가능하다면 위험.
- 오프셋에 관련된 처리도 여기서 일어나는데 세부 동작은 뒤에서 다시 설명.
- [`KafkaConsumer#poll`](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/KafkaConsumer.java#L1168) 문서 보면 아래 설명 적혀 있음 참고.

> Fetch data for the topics or partitions specified using one of the subscribe/ assign APIs. It is an error to not have subscribed to any topics or partitions before polling for data.
> 
> On each poll, consumer will try to use the last consumed offset as the starting offset and fetch sequentially. The last consumed offset can be manually set through seek(TopicPartition, long) or automatically set as the last committed offset for the subscribed list of partitions
> 
> This method returns immediately if there are records available or if the position advances past control records or aborted transactions when isolation. level=read_committed. Otherwise, it will await the passed timeout. If the timeout expires, an empty record set will be returned. Note that this method may block beyond the timeout in order to execute custom ConsumerRebalanceListener callbacks.

### 컨슈머 레코드

레코드는 [ConsumerRecords](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/ConsumerRecords.java) 구현에서 보듯 파티션 데이터이고 `Iterable`.

```java
public class ConsumerRecords<K, V> implements Iterable<ConsumerRecord<K, V>> {
    public static final ConsumerRecords<Object, Object> EMPTY = new ConsumerRecords<>(Collections.emptyMap());

    private final Map<TopicPartition, List<ConsumerRecord<K, V>>> records;

    ... 중략

    @Override
    public Iterator<ConsumerRecord<K, V>> iterator() {
        return new ConcatenatedIterable<>(records.values()).iterator();
    }

    ...
```

### 컨슈머 생성과 토픽 구독

```kt
// 아래 4가지 프로퍼티는 필수
val properties = Properties().apply {
    
    setProperty(
        BOOTSTRAP_SERVERS_CONFIG,
        "broker1:9092,broker2:9092"
    )
    
    setProperty(
        KEY_DESERIALIZER_CLASS_CONFIG,
        StringDeserializer::class.java.name
    )

    setProperty(
        VALUE_DESERIALIZER_CLASS_CONFIG,
        StringDeserializer::class.java.name
    )
    
    setProperty(
        GROUP_ID_CONFIG,
        "groupId"
    )
}

val consumer = KafkaConsumer<String, String>(properties)
consumer.subscribe(mutableListOf("accountTopic"))
```

## 설계

### 푸시 vs 풀

- RabbitMQ와 달리 컨슈머가 pull.
- [아파치 카프카 문서](https://kafka.apache.org/documentation/#design_pull)에서는 아래와 같이 이야기.

> data is pushed to the broker from the producer and pulled from the broker by the consumer. ... 중략 ... However, a push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. The goal is generally for the consumer to be able to consume at the maximum possible rate; unfortunately, in a push system this means the consumer tends to be overwhelmed when its rate of consumption falls below the rate of production.

- push와 pull은 서로 트레이드 오프.
- push 시스템은, 브로커가 데이터 전송 속도를 제어하므로, 다양한 컨슈머를 다루기 어려움.
- 예를 들어, 소비 속도가 생산 속도보다 낮아질 때, 백오프 포로토콜 등으로 제어해야 하나, 소비자의 완전한 처리량을 활용하기는 까다로움.
- pull 방식에서는 메시지 생성 속도의 부담으로부터 자유로움.
- 또한, pull 방식에서는 컨슈머가 데이터를 공격적으로 읽어들일 수 있음.
- push에서는 데이터를 브로커가 즉각 즉각 보내거나 혹은 어느 정도 모아서 보내야 하는데 뭘 선택하더라도 지연은 불가피.
- 한편, 데이터가 없는데 자꾸 폴링하는 건 비용이라, 데이터가 도착할 때까지 기다리게 하는 파라미터를 함께 제공하고 있음.
- [카프카 문서](https://kafka.apache.org/24/documentation.html)에 나온 아래 그림이 이해를 도움.

![](https://kafka.apache.org/24/images/log_consumer.png)

### 파티션과 컨슈머

- 기본적으로 토픽은 1개의 파티션과 함께 생성됨.
- 그리고 확장을 고려한다면, 파티션 수를 크게 잡아주는 게 유리.
- 파티션이 여러 개라면 여러 개의 컨슈머를 할당할 수 있음.
- 참고로, 컨슈머와 파티션의 관계는 1:N.
- 만약, 컨슈머가 파티션 수보다 많다면 유휴 컨슈머 발생.
- 그리고, 하나의 토픽에 그룹 자체를 여럿 둘 수도 있음.
- 처음 시작할 땐 아래 모습일 수 있음.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*sPGKO_Z-rKbM00dJd0QV7w.png)

- 이후에 처리량을 늘리고자 컨슈머 추가 투입.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*zTRQR9H_Glv1xbPWl_yY6w.png)

- 그러나 컨슈머만 계속 늘린다고 처리량이 늘어나지는 않음.
- 파티션 수도 함께 고려해야 함.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*INW2vHXkN7v47-WvrgbhbA.png)

- 여기서 나아가 토픽에 그룹 자체를 추가할 수도.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*uH_krFrkQgJYpHYjKXLyJw.png)

그림 출처는 [여기](https://medium.com/javarevisited/kafka-partitions-and-consumer-groups-in-6-mins-9e0e336c6c00).

### 리밸런스

- 리밸런싱이란, 컨슈머에 할당된 파티션을 다른 컨슈머에게 할당하는 작업.
- 그룹에 컨슈머 추가나 삭제(또는 의도치 않은 중단), 혹은 운영자가 토픽에 새로운 파티션을 추가한 경우 발생.
- 목적은 high availability, scalability.
- 하지만 달갑지 않은 부분도 존재하며 주의해야 함.

#### 조급한<sup>eager</sup> 리밸런스

[여기](https://www.confluent.io/blog/debug-apache-kafka-pt-3/)에서는 조급한 리밸런스를 Stop-the-World라 부르고 있음. 전체 과정은 아래와 같음.

![](https://images.ctfassets.net/8vofjvai1hpv/Pfql9YfYW5ILCJK3Avotd/38032f581fa4c6b95544ed0d1dcde626/stop-the-world-rebalancing.png)

1. 이미 그룹에 컨슈머들이 읽기를 하고 있었다고 가정.
2. 새로운 컨슈머가 토픽에 대한 JoinGroup 요청을 그룹 코디네이터에게 전달.
3. 코디네이터는 현재의 모든 멤버들에게 JoinGroup 요청을 보내라면서(주기적으로 일어나는 heartbeat의 응답을 통해 요구) 리밸런싱을 시작.
4. 멤버들은 하던 처리를 마무리하고 코디네이터에게 JoinGroup을 보내며 Stop-the-World.
5. 코디네이터는 그룹에 속한 멤버들과 그룹이 참여한 토픽들을 알게 되고, 코디네이터는 멤버들에게 JoinResponse을 보내며, [리더](https://stackoverflow.com/questions/42015158/what-is-the-difference-in-kafka-between-a-consumer-group-coordinator-and-a-consu)를 선출해 파티션 할당을 위임.
6. 모든 멤버는 이에 대한 반응으로 SyncGroup 요청을 보내고, 여기에 그룹 리더는 파티션 할당을 담아서 보냄.
7. 코디네이터는 SyncResponse를 컨슈머에게 보내며 토픽 파티션 할당을 확정.
8.  컨슈머들은 할당을 받아들이고 읽기 작업을 재개. Stop-the-World는 종료.

#### 협력적<sup>cooperative</sup> 리밸런스

incremental rebalance 혹은 incremental cooperative rebalance 라고도 부름. 조급한 리밸런스에서의 1~3 단계까지는 동일한데 그 이후가 다름.

![](https://cdn.confluent.io/wp-content/uploads/cooperative_rebalancing_protocol-1536x716.png)

1. 조급한 리밸런스에서의 1~3 단계를 동일하게 진행.
2. 멤버들은 하던 처리를 마무리하고 코디네이터에게 JoinGroup 요청을 보내는데, 이 때 새로운 할당을 기다리며 Stop-The-World를 하는 대신, 컨슈머들은 추가적인 통지가 있을 때까지 기존에 할당된 토픽 파티션 처리를 계속 진행.
3. JoinGroup 요청을 받은 코디네이터는 그룹에 속한 멤버들과 그룹이 참여한 토픽들을 알게 되고, 코디네이터는 JoinResponse를 멤버들에게 보내며, 리더를 선출해 파티션 할당을 위임.
4. 모든 멤버는 이에 대한 반응으로 SyncGroup 요청을 보내고, 여기에 그룹 리더는 파티션 할당을 담아서 보냄.
5. 여기서 코디네이터는 새로운 할당과 과거의 할당을 비교해 변경이 필요한 컨슈머에게만 SyncResponse 응답.
6. 그리고 영향을 받는 컨슈머만 리밸런싱을 위해 중단.
7. 일련의 과정을 이번에는 파티션 할당을 위해 다시 진행.

Stop-The-World는 사라지지만, 그룹에 속한 컨슈머가 많을 경우 리밸런싱이 오래 걸릴 수 있음. [Dynamic vs. Static Consumer Membership in Apache Kafka](https://www.confluent.io/blog/dynamic-vs-static-kafka-consumer-rebalancing/) 문서에서는 롤링 업그레이드에서 동적 멤버십이 문제가 되기도 한다고 설명.

[Eager vs. cooperative: Who wins?](https://www.confluent.io/blog/cooperative-rebalancing-in-kafka-streams-consumer-ksqldb/#eager-vs-cooperative)에는 조급한 리밸런스와 협력적 리밸런스의 벤치마크도 있음.

![](https://cdn.confluent.io/wp-content/uploads/throughput-vs-time_eager.png)

Throughput (records/s) vs. time (s) for a 10-instance eager rebalancing Streams app going through a rolling bounce.

![](https://cdn.confluent.io/wp-content/uploads/throughput-vs-time_cooperative.png)

Throughput (records/s) vs. time (s) for a 10-instance cooperative rebalancing app going through a rolling bounce.

#### 정적 그룹 멤버십

- 컨슈머가 그룹을 떠나면, 혹은 다시 참여하면 리밸런싱이 기본.
- 그러나 컨슈머에 `group.instance.id`를 할당하면 정적 멤버가 됨.
- 떠났다가 다시 돌아오더라도, `session.timeout.ms` 시간을 넘지 않았다면, 이전 파티션 할당.
- 책에서는, 파티션 데이터로 로컬 상태나 캐시를 구축해야 하고, 컨슈머 재시작으로 이 작업을 반복하고 싶지 않을 때 유용하다고 함.
- [Dynamic vs. Static Consumer Membership in Apache Kafka](https://www.confluent.io/blog/dynamic-vs-static-kafka-consumer-rebalancing/)에서는 롤링 업그레이드 환경을 위한 것이라고 함.
- 동적 멤버십에서는, 롤링 업그레이드 시 리밸런싱이 컨슈머 수의 2배 만큼 발생하므로 느림.

### 오프셋

- `poll()`은 아직 읽지 않은 레코드 반환.
- 여기서, 아직 읽지 않았는지 여부는 오프셋으로 판단.
- 오프셋이란, 각 파티션 메시지를 고유하게 식별하는 증분 ID.
- [Kafka Offset Explained](https://kontext.tech/diagram/1159/kafka-offset-explained)에 나온 그림과 같이 컨슈머와 프로듀서 모두 파티션 별 오프셋이 있음.

![](https://kontext.tech/api/flex/diagram/diagram-1159)

- 컨슈머는 어느 오프셋의 메시지까지를 읽었는지 기록하며 이를 오프셋 커밋이라 함.
- 오프셋 커밋은 전통적 메시지 큐와 달리 개별 레코드 커밋이 아니라 처리한 메시지의 마지막 위치를 커밋.
- 오프셋은 카프카의 특수 토픽인 `__consumer_offsets`(그룹 코디네이터가 가진)에 저장.
- 리밸런싱으로 인해 컨슈머가 새로운 파티션에 할당되면, 어디부터 데이터를 읽을지 알기 위해 여기서 값을 불러옴.
- `실제 처리한 메시지 오프셋 > 커밋된 메시지 오프셋`이면 이 사이의 메시지들은 중복 처리.
- `실제 처리한 메시지 오프셋 < 커밋된 메시지 오프셋`이면 이 사이의 메시지들은 처리 누락.
- 참고로, 컨슈머 그룹이 처음 파티션을 읽을 땐, 커밋된 오프셋이 없으므로 [`auto.offset.reset`](https://kafka.apache.org/documentation/#consumerconfigs_auto.offset.reset) 설정에 따라 행동.


![](https://newrelic.com/sites/default/files/wp_blog_inline_files/events_lost2-1024x547.jpg)

![](https://newrelic.com/sites/default/files/wp_blog_inline_files/events_duplicated2.jpg)

- 주의점: `poll`에서 받은 마지막 메시지의 오프셋이 아니라, 실제로는 그 다음 오프셋을 커밋한다고 함. 수동 오프셋 커밋 시 유의.

### 오프셋 자동 커밋

- `enable.auto.commit=true`이면 읽어 들인 마지막 메시지의 오프셋을 주기적으로 자동 커밋.
- `auto.commit.interval.ms`로 자동 커밋 주기 설정(기본 5s).
- 자동 커밋 사용 시, 주기적으로 커밋되므로, 리밸런싱이 일어난 뒤 메시지 중복 발생 가능.
- 주기를 아무리 줄여도 중복을 완전히 제거할 순 없음.
- 또한, 다음 `poll`에서 커밋이 일어날 수 있으니, 이전에 받은 메시지 처리를 그 전에 끝내야 함.
- 편리 vs. 중복 트레이드오프.

### 오프셋 수동 커밋

- <카프카 핵심 가이드> 책에서는 대부분의 개발자들이 오프셋 커밋을 직접 제어한다고 함.
- 유실과 중복 가능성 제거가 목적.
- 이를 위해 우선 `enable.auto.commit=false` 설정.
- 그리고 `commitSync()` API 활용.
- 파라미터를 명시하지 않을 땐, `poll()`에서 리턴된 마지막 메시지를 커밋.
- [KafkaConsumer](https://github.com/apache/kafka/blob/trunk/clients/src/main/java/org/apache/kafka/clients/consumer/KafkaConsumer.java) Javadoc에 나온 예시를 살짝 바꾼 코드는 아래와 같음.

```kt
val buffer = mutableListOf<ConsumerRecord<String, String>>()
val minBatchSize = 200

while (true) {
    val records = consumer.poll(Duration.ofMillis(100))
    records.forEach(buffer::add)

    if (buffer.size >= minBatchSize) {
        persist(buffer)
        consumer.commitSync()
        buffer.clear()
    }
}
```

- `commitSync`에 오프셋을 넘기는 예제도 있음.

```kt
val records = consumer.poll(Duration.ofMillis(Long.MAX_VALUE))

// 컨슈머:파티션 = 1:N
for (partition in records.partitions()) {
    val partitionRecords = records.records(partition)

    partitionRecords.forEach {
        println("offset: ${it.offset()}, value: ${it.value()}")
    }

    val offset = OffsetAndMetadata(partitionRecords.last().offset() + 1)
    consumer.commitSync(mapOf(partition to offset))
}
```

### 오프셋 비동기 커밋

- 수동 커밋 단점 중 하나가 커밋 일어나는 동안의 애플리케이션 블럭킹.
- 물론 덜 자주 커밋해서 처리량을 올릴 순 있음. 그러나 리밸런싱에 의한 메시지 중복이 늘어남.
- 또 하나의 방법은 `commitAsync()` API를 이용한 비동기 커밋.
- 요청만 보내고 응답은 기다리지 않고 다른 처리를 이어감.
- 다만, 비동기 커밋은 동기 커밋과 달리 재시도가 없음.
- 순서 꼬임 문제가 발생할 수 있기 때문.
- 아래와 같이 동기와 비동기를 혼합해 사용하기도.

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

- 재시도 없는 커밋이 간혹 실패한다고 큰 문제가 되진 않음. 뒤이은 커밋이 결국엔 일어나기 때문.
- 그러나 컨슈머 종료 전, 혹은 리밸런싱 전에는, 오프셋 중요할 수 있으니 동기적 커밋.

### 원하는 오프셋부터 읽기

- 파티션의 원하는 오프셋부터 데이터 읽어들이기도 가능.
- `seekToBeginning(Collection<TopicPartition> partitions)`: 파티션 처음부터 전부 읽기
- `seekToEnd(Collection<TopicPartition> partitions)`: 파티션 마지막부터 새로 읽기
- `seek`로 특정 오프셋을 지정할 수도 있음.
- API 문서 설명은 아래와 같으며, 다음 `poll` 동작 시 오프셋이 반영되는 것임에 유의.

> Overrides the fetch offsets that the consumer will use on the next poll(timeout). If this API is invoked for the same partition more than once, the latest offset will be used on the next poll(). Note that you may lose data if this API is arbitrarily used in the middle of consumption, to reset the fetch offsets

- 애플리케이션에 문제가 있어 시간에 민감한 데이터 처리가 늦어져서 건너뛰어야 하거나, 컨슈머 측 데이터 유실로 전체 복구 시 고려.

## 설정

알아 두면 좋을만한 설정들.

| 설정 | 설명 |
| -- | -- |
| [bootstrap.servers](https://kafka.apache.org/documentation/#producerconfigs_bootstrap.servers) | 클러스터 연결 정보 |
| [group.id](https://kafka.apache.org/documentation/#producerconfigs_bootstrap.servers) | 컨슈머 그룹 식별자 |
| [key.deserializer](https://kafka.apache.org/documentation/#consumerconfigs_key.deserializer) | |
| [value.deserializer](https://kafka.apache.org/documentation/#consumerconfigs_value.deserializer) | |
| [fetch.min.bytes](https://kafka.apache.org/documentation/#consumerconfigs_fetch.min.bytes) | 브로커로부터 레코드 얻어올 때 받는 최소 데이터량이며, 충분한 양이 쌓이면 레코드를 전송. 읽어 올 데이터가 별로 없는데 CPU를 많이 쓰거나, 컨슈머가 많아 브로커 부하를 줄여야 할 때 고려. 지연이 생기는 건 트레이드 오프. |
| [fetch.max.wait.ms](https://kafka.apache.org/documentation/#consumerconfigs_fetch.max.wait.ms) | min.bytes 지정 시 지연이 너무 길어질 수 있으니, 최대 대기 시간으로 지연 줄일 때 사용.|
| [fetch.max.bytes](https://kafka.apache.org/documentation/#consumerconfigs_fetch.max.bytes) | 대량 디스크 읽기와 네트워크 전송에 따른 부하 발생 시 고려. |
| [max.poll.records](https://kafka.apache.org/documentation/#consumerconfigs_max.poll.records) | `poll()`이 반환할 최대 레코드. |
| [session.timeout.ms](https://kafka.apache.org/documentation/#consumerconfigs_session.timeout.ms) | 컨슈머가 그룹 코디네이터에게 하트비트를 보내야 하는 타임아웃. |
| [heartbeat.interval.ms](https://kafka.apache.org/documentation/#consumerconfigs_heartbeat.interval.ms) | 하트비트의 전송 주기이며, 온프레미스 환경이면 보통 `session.timeout.ms`의 1/3으로 설정. |
| [max.poll.interval.ms](https://kafka.apache.org/documentation/#consumerconfigs_max.poll.interval.ms) | `poll()` 스레드는 죽었는데 백그라운드에서 하트비트는 계속 전송될 수 있음. 그래서 최대 폴링 타임아웃 지정으로, 죽은 컨슈머 찾아내는데 함께 활용. |
| [request.timeout.ms](https://kafka.apache.org/documentation/#brokerconfigs_request.timeout.ms) | 클라이언트의 브로커 응답 대기 타임아웃. 시간이 지나면 연결을 끊고 재연결. |
| [auto.offset.reset](https://kafka.apache.org/documentation/#consumerconfigs_auto.offset.reset) | 커밋 오프셋이 없거나 유효치 않을 때의 동작. earliest, latest, none |
| [enable.auto.commit](https://kafka.apache.org/documentation/#consumerconfigs_enable.auto.commit) | 자동 커밋 사용 여부 |
| [partition.assignment.strategy](https://kafka.apache.org/documentation/#consumerconfigs_partition.assignment.strategy) | 파티션 할당 전략. RangeAssignor, RoundRobinAssignor, StickyAssignor, CooperativeStickyAssignor |
| [client.rack](https://kafka.apache.org/documentation/#consumerconfigs_client.rack) | 데이터센터나 가용 영역이 다수인 경우 같은 영역에 있는 레플리카로부터 메시지를 읽어 오게 하는 설정. |
| [group.instance.id](https://kafka.apache.org/documentation/#consumerconfigs_group.instance.id) | 정적 멤버십 사용할 때 지정하는 컨슈머 식별자. |

## 참고 문서

1. 카프카 핵심 가이드 2판
2. https://kafka.apache.org/documentation
3. https://www.confluent.io/blog/debug-apache-kafka-pt-3/
4. https://www.confluent.io/blog/dynamic-vs-static-kafka-consumer-rebalancing/
5. https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiAx9q6BhCDARIsACwUxu46n05vuk2S2xtJdZ2tf0TgZpzs2zRmTYldmZ0ej1GSfwALOjt0VBIaAsmIEALw_wcB#enable-auto-commit
6. https://medium.com/javarevisited/kafka-partitions-and-consumer-groups-in-6-mins-9e0e336c6c00
7. https://kontext.tech/diagram/1159/kafka-offset-explained
8. https://stackoverflow.com/questions/42015158/what-is-the-difference-in-kafka-between-a-consumer-group-coordinator-and-a-consu
9.  https://docs.confluent.io/platform/current/clients/consumer.html
10. https://github.com/spring-projects/spring-kafka/blob/main/spring-kafka/src/main/java/org/springframework/kafka/listener/KafkaMessageListenerContainer.java
11. https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/KafkaConsumer.java
12. https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/consumer/ConsumerRecords.java
