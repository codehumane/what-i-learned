
# 카프카 핵심 가이드 2판

# 2장. 카프카 설치하기

## 2.1 환경 설정

별 내용은 없어서 일부 내용만 흐름 없이 기록.

- 대체로 리눅스 사용 권장.
- 카프카와 주키퍼는 자바 구현체 위에서 원활하게 작동.
- 카프카 클러스터의 메타데이터와 컨슈머 클라이언트 정보를 주키퍼에 저장.
- 주키퍼는 고가용성 보장을 위해 앙상블<sup>ensemble</sup>이라 불리는 클러스터 단위로 동작.
  - 이 때 쿼럼 때문에 홀수 개 서버 권장.
  - 그리고 보통 5개의 노드를 고려. 설정 변경 등의 이유로, 한 대의 노드를 정지할 수도 있기 때문(최소 3개만 있으면 동작하니, 1대는 작업 중이고 추가로 1대가 장애인 경우까지 고려하는 듯).
  - 그리고 9대 이상은 합의 프로토콜 특성상 성능 저하가 발생한다고 함.
  - 5대나 7대가 부하를 감당하기 어렵다면, 옵저버 노드를 추가해 읽기 전용 트래픽 분산을 고려할 수도.
  - 그 외에 모든 서버에 공통 설정 파일이 있어야 하고, myid 파일, initLimit과 syncLimit, client port와 peer port와 leader port 등의 얘기도.

## 2.3 브로커 설정하기

- 많은 설정 옵션이있으나, 기본 값을 바꿀 일은 잘 없다고 함.
- 내부 동작 이해 목적으로 살피면 될 듯.

### 2.3.1 핵심 브로커 매개변수

| 매개변수 | 설명 |
| -- | -- |
| broker.id | 브로커는 정수값 식별자를 가지며, 클러스터 안에서 고유해야 함. 호스터네임 활용 권장. |
| listeners | 수미표로 구분된 리스너 이름과 URI 목록. `PLAINTEXT://localhost:9092,SSL://:9091` 같은 형태. 호스트 이름을 0.0.0.0으로 잡으면 모든 네트워크 인터페이스로부터 연결을 받고, 값을 비우면 기본 인터페이스에 대해서만 연결 받음. |
| zookeeper.connect | 메타데이터 저장되는 주키퍼 위치. |
| log.dirs | 카프카는 모든 메시지를 로그 세그먼트 단위로 묶어 `log.dir` 디렉토리에 저장. 다수의 디렉토리를 사용하고자 한다면 `log.dirs` 사용. 다수의 경로가 지정되었다면, 브로커는 가장 적은 수의 파티션이 저장된 디렉토리에 새 파티션을 저장. 디스크 용량 사용량이 아니라, 저장된 파티션 수 기준으로 위치 배정. 따라서, 균등한 양의 데이터가 저장되지는 않음. |
| num.recovery.threads.per.data.dir | 카프카는 설정 가능한 스레드 풀을 사용해 로그 세그먼트를 관리. 브로커가 정상적으로 시작되면 각 파티션의 로그 세그먼트 파일을 열고, 브로커의 장애로 재시작 시 각 파티션 로그 세그먼트를 검사하고 잘못된 부분은 삭제하며, 브로커 종료 시 로그 세그먼트를 닫는 역할. 기본적으로는 하나의 로그 디렉토리에 하나의 스레드가 할당. 그러나 작업 병렬화를 위해 많은 스레드를 할당해 주는 게 좋고, 이는 파티션이 많을 수록 큰 차이를 가져옴. |
| auto.create.topics.enable | 프로듀서가 토픽에 메시지를 쓰기 시작하거나, 컨슈머가 토픽으로부터 메시지를 읽기 시작하거나, 클라이언트가 토픽 메타데이터를 요청할 때, 카프카는 토픽을 자동 생성한다고 함. 그러나, 토픽 생성을 명시적으로 관리하고자 한다면, 이 값을 false로 두기. |
| auto.leader.rebalance.enable | 모든 토픽 리더 역할이 하나의 브로커에 집중되면 클러스터 균형이 깨질 수 있음. 이 설정 활성화로 리더 역할을 가능한 균등하게 분산시킬 수 있음. 내부적으로는 파티션 분포 상태를 주기적으로 확인하는 백그라운드 프로세스가 시작. |
| delete.topic.enable | 토픽 임의 삭제를 막으려면 이 값을 false로 설정. |

### 2.3.2 토픽별 기본값

파티션 수 결정 고려 사항 먼저 정리.

1. 토픽 처리량은 얼마인가?
2. 단일 파티션에 대해 달성코자 하는 최대 읽기 처리량은? 하나의 파티션은 하나의 컨슈머만 읽을 수 있으며, 만약 컨슈머 애플리케이션의 DB 저장 속도가 느리다면 이는 컨슈머의 읽기 처리량으로 이어짐.
3. 프로듀서의 파티션 쓰기는 일반적으로는 고민 안 해도 됨. 보통 프로듀서가 컨슈머에 비해 훨씬 빠르기 때문.

키 값을 기준으로 파티션 분산을 한다면, 나중에 파티션을 추가하는 건 골치 아플 수 있음. 따라서, 현재 사용량이 아닌, 미래 사용량 예측값을 기준으로 처리량을 계산.

1. 브로커에 배치할 파티션 수, 브로커 별 사용 가능한 디스크 공간, 네트워크 대역폭 고려.
2. 과대 추산은 금지. 각 파티션은 브로커 메모리 등 여러 자원을 사용하고, 메타데이터 업데이트나 리더 변경 등의 지연도 늘어남.
3. 미러링 처리량 역시 고려해야 함. 큰 파티션은 미러링의 병목.
4. 파티션 수가 너무 많으면 병렬 처리로 클라우드의 IOPS 제한에 문제가 될 수도.

| 매개변수 | 설명 |
| -- | -- |
| num.partitions | 새로운 토픽 생성 시 몇 개의 파티션을 갖게 할지에 대한 설정이며, 주로 자동 토픽 생성 기능 활성화 시 사용됨. 기본값은 1. 많은 사용자들은 토픽당 파티션 수를 클러스터 내 브로커 수와 맞추거나 배수로 설정. 이렇게 하면 파티션이 브로커들에게 고르게 분산. 당연히 부하도 분산됨. 단, 이 방법 외에도 부하 분산 방법은 많아서, 이 방식이 필수는 아님. |
| default.replication.factor | 토픽의 리플리케이션 개수이며, 토픽 자동 생성 기능 사용 시 쓰임. `min.insync.replicas` 보다 최소 1 이상 크게 잡아야 함. 장애 내성이 더 높으려면 2만큼 더 큰 값으로 설정. 업데이트 등을 위해 일부러 레플리카를 하나 정지시키고, 의도치 않은 레플리카가 하나 더 발생해도, 적어도 1개가 남기 때문. |
| log.retention.ms | 카프카가 얼마나 오랫동안 메시지를 보존할지의 설정. 기본 값은 1주일. |
| log.retention.bytes | 메시지 만료의 또 다른 기준인 용량. |
| log.segment.bytes | 메시지는 파티션의 현재 로그 세그먼트 끝에 추가됨. 이 설정의 크기만큼 세그먼트가 커지면 브로커는 기존 것을 닫고 새로운 세그먼트를 염. 세그먼트는 닫히기 전까지는 만료나 삭제 대상이 되지 않음. 너무 작으면 디스크 쓰기의 효율성 감소. 그러나, 만약 메시지가 뜸하게 들어온다면, 예를 들어 하루에 100MB만큼 들어온다면, 기본 값을 기준으로 세그먼트 채우는 데 10일 소요됨. |
| log.roll.ms | 로그 세그먼트 파일이 닫히는 시점을 제어하는 또 하나의 방법으로, 파일이 닫힐 때까지 기다리는 시간을 지정할 수 있음. 기본값은 없으며, 따라서 이 때는 `log.segment.bytes` 값만이 세그먼트 닫힘에 사용됨. |
| min.insync.replicas | 이 값을 2로 잡으면 최소 2개 레플리카가 최신 상태로 동기화. 이를 사용하려면 프로듀서의 ack 설정을 `all`로 설정해야 함.  이는 리더가 쓰기 작업에 응답하고 장애가 발생했지만, 레플리카로 복제가 안 되고 새로운 리더가 선출되는 경우의 유실을 막아 줌. |
| message.max.bytes | 쓸 수 있는 메시지ㅡ이 최대 크기. 기본값은 1MB. 더 큰 크기의 메시지가 보내지면, 브로커는 이를 거부하고 에러를 반환. 이 값을 늘리면 요청 당 작업 시간도 늘어나고 I/O 처리량에도 영향. |

## 2.6 카프카 클러스터 설정하기

- 클러스터 구성의 가장 큰 이점은 부하 분산.
- 또한, 단일 시스템 장애 시와 달리 데이터 유실도 방지.
- 클라이언트 요청을 계속 처리하면서도 하단 시스템의 유지 관리 작업 수행도 가능.

### 2.6.1 브로커 개수

- 클러스터 크기 결정 요소: 디스크 용량, 브로커당 레플리카 용량, CPU, 네트워크.
- 일단, 클러스터가 10TB 데이터를 저장해야 한다면, 브로커가 최대 2TB 저장할 수 있을 때 최소 5개 브로커가 필요.
- 복제 팩터를 증가시킬 경우 저장 용량은 최소 100% 이상 증가함도 고려.
- 처리량도 고려해야 함. 만약, 10개 브로커가 있는데, 레플리카 수는 100개(예를 들어, 파티션이 50만 개고 복제 팩터가 2)이면, 각 브로커가 약 10만 개의 레플리카를 보유해야 함. 이는 부하가 됨.
- 과거에는, 브로커 당 4천 개 이하의 파티션 레플리카를, 클러스터 당 20만 개 이하의 레플리카를 공식적으로 권장.
- 현재는, 브로커 당 1.4만개, 클러스터당 100만개 이하로 파티션 레플리카 유지를 권장.
- CPU는 보통 주요 병목이 아니긴 하나, 하나의 브로커에 클라이언트 연결이 쏟아진다면 부하가 될 수도. 연결 수를 잘 관찰하다가 클러스터를 확장하면 됨.
- 네트워크 인터페이스 전체 용량은 얼마인지, 컨슈머가 많거나 트래픽이 많을 때도 잘 받아낼 수 있는지도 고려해야 함.
- 만약, 단일 브로커에 대해 네트워크 인터페이스 전체 용량이 80% 가량 사용된다면, 컨슈머가 2대일 때 1개의 브로커로는 부족.
- 복제 기능이 켜져 있다면 이 또한 데이터 컨슈머로써 고려가 필요.

### 2.6.2 브로커 설정

- 별 내용 없음.
- 모든 브로커들이 동일한 `zookeeper.connect` 값을 가져야 함.
- 여기에 주키퍼 앙상블과 경로를 지정. 이는 클러스터 메타데이터가 저장되는 위치를 나타냄.
- 두 번째로, 클러스터의 모든 브로커는 유일한 `broker.id`를 가져야 함.

# 3장. 카프카 프로듀서: 카프카에 메시지 쓰기

아래의 것들 다룬다고 함.

- 프로듀서의 디자인과 주요 요소의 전체적 모습.
- `KafkaProducer`, `ProducerRecord` 객체 생성.
- 카프카에 레코드를 어떻게 전송하는지.
- 카프카가 리턴할 수 있는 에러를 어떻게 처리할지.
- 프로듀서 작동 제어를 위한 설정들.
- 파티션 할당 방식을 정의하는 파티셔너.
- 객체의 형식을 정의하는 시리얼라이저.

## 3.1 프로듀서 개요

카프카에 데이터 전송 시 내부적으로 일어나는 일들 소개.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*YlIYjzZcc5pd1V5fBn91ng.png)

- 메시지 전송은 `ProducerRecord` 생성과 함께 시작.
- 이 때, 토픽과 값 지정은 필수, 키와 파티션 지정은 선택.
- 그리고 이 레코드를 전송하는 API를 호출하면, 키와 값 객체를 직렬화해서 바이트 배열로 변환.
- 만약, 파티션을 명시적으로 지정하지 않았다면, 데이터를 파티셔너에게 보냄.
- 파티셔너는 파티션 결정에 레코드 키 값 사용.
- 다음으로, 레코드 배치(같은 파티션으로 보낼 데이터를 모아둔)로 레코드 전달.
- 별도의 스레드가 이 레코드 배치를 브로커에게 전송.
- 메시지 저장이 성공하면, RecordMetadata 객체(파티션과 오프셋이 담긴) 반환.
- 저장 실패 시에는 에러 반환.
- 프로듀서는 에러를 받았을 때 재시도를 할 수도 있음.

## 3.2 카프카 프로듀서 생성하기

프로듀서 생성에는 3가지 필수 속성 필요.

- `bootstrap.servers`: 브로커의 연결 정보인데, 모든 브로커를 포함할 필요는 없음. 첫 연결을 시도한 뒤, 추가 정보를 받아 오기 때문. 1개가 문제 있을 수 있으니 최소 2개 이상 지정 권장.
- 그 외에 `key.serializer`, `value.serializer` 필요.

[`KafkaProducer`](https://github.com/a0x8o/kafka/blob/master/clients/src/main/java/org/apache/kafka/clients/producer/KafkaProducer.java) javadoc에 예제 잘 나와 있음.

```java
Properties props = new Properties();
props. put("bootstrap. servers", "localhost:9092");

Producer<String, String> producer = new KafkaProducer<>(
    props,
    new StringSerializer(),
    new StringSerializer()
);
```

메시지 전송 방법도 설명함. 3가지 있음.

1. fire and forget: 전송만 하고 성공이나 실패 결과는 신경 쓰지 않음. 보통은 가용성이 높고 재시도가 일어나기에 유실은 잘 없음. 알 수 없는 에러 또는 타임아웃 시 유실 가능.
2. synchronous send: 기술적으로는 전부 비동기. 여기서는 Future 객체를 반환함을 가리킴.
3. asynchronous send: send에 콜백 함수 담아서 비동기 호출.

## 3.3 카프카로 메시지 전달하기

```java
ProducerRecord<String, String> record = 
    new ProductRecord<>(
        "CustomerCountry",
        "Precision Products",
        "France"
    );

try {
    producer.send(record);
} catch (Exception e) {
    e.printStackTrace();
}
```

1. 우선 레코드가 필요. `ProducerRecord` 생성자는 여러 개인데 이 중 하나 사용. 여기서는 토픽 이름, 키, 밸류를 지정.
2. 다음으로, `send` 메서드 호출. 이 메시지는 버퍼에 저장되었다가 별도 스레드가 브로커로 전달. 반환 값으로는 `RecordMetadata`가 담긴 `Future`인데 여기서는 무시.
3. 메시지 전송 결과를 무시하더라도, 메시지 직렬화 실패로 `SerializationException`이 발생할 수도 있고, 버퍼가 가득 차서 `TimeoutException`이 발생하거나, 전송 작업 수행하는 스레드에 인터럽트가 걸린 경우 `InterruptException`이 발생할 수도 있음에 유의.

### 3.3.1 동기적으로 메시지 전송하기

- 동기적 코드는 성능 저하가 커서, 예제 코드만 많을 뿐, 실전에서는 사용 잘 안 함.
- 방법은 간단. 반환 받은 `Future`의 `get()`을 호출해 결과를 기다리면 됨.

### 3.3.2 비동기적으로 메시지 전송하기

- 엄청난 성능 이점으로 대부분 비동기로 메시지 전송.
- 이 때, 전송 성공한 경우는 보통 관심이 없으나, 실패에 대한 사후 처리는 중요.

```java
private class DemoProducerCallback implements Callback {
    @Override
    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
        if (e != null) {
            e.printStackTrace();
        }
    }
}

ProducerRecord<String, String> record =
    new ProducerRecord<>(
        "CustomerCountry",
        "Biomedical Materials",
        "USA"
    );

producer.send(record, new DemoProducerCallback());
```

## 3.4 프로듀서 설정하기

| 설정 | 설명 |
| -- | -- |
| [client.id](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#client-id) | 프로듀서의 논리적 식별자. ip/port로는 빠른 식별/추적이 어려우므로 이를 사용. |
| [acks](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#acks) | 얼마나 많은 파티션 레플리카가 레코드를 받아야 쓰기를 성공으로 간주할지. 기본 값은 리더만 받아도 성공. `0`, `1`, `all`이 존재함. 1인 경우에는, 리더 충돌로 복제 끝나기 전 새 리더 선출 일어나면 유실. |
| [max.block.ms](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#max-block-ms) | 메시지 전송의 각 단계인 `send()`, `partitionsFor()`, `initTransactions()`, `sendOffsetsToTransaction()`, `commitTransaction()`, `abortTransaction()`에서 얼마까지 대기를 기다릴지. 책이랑 카프카 문서랑 좀 달라서, 문서 보는 게 좋아 보임. |
| [delivery.timeout.ms](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#delivery-timeout-ms) | `send()` 이후 성공 또는 실패 결과를 전달받기까지의 타임아웃 시간. 재시도 중 이 타임아웃이 넘어가면, 가장 최근 에러 결과를 콜백에 전달. 레코드 배치 전송 기다리는 중에 타임아웃 되면, 타임아웃 예외와 함께 콜백 호출. |
| [request.timeout.ms](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#request-timeout-ms) | 요청에 대한 응답을 얼마나 기다릴지. 이 시간을 넘으면 재시도를 하거나, 재시도 횟수가 끝나면 TimeoutException을 콜백에 전달. `replica.lag.time.max.ms` 시간보다 길어야 함에 유의. |
| [retries](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#retries), [retry.backoff.ms](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#retry-backoff-ms) | 재시도 횟수, 그리고 재시도 사이의 간격. 책에서는 기본 값 조정을 권장하지 않는다고 함. 대신, `delivery.timeout.ms`를 설정하라고. 세부적 정책보다, 카프카 클러스터가 복구될 수 있는 충분한 시간을 지정하는 것. |
| [linger.ms](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#linger-ms) | 기본적으로는 메시지 전송할 스레드가 있으면 메시지 전송. 그러나 이 값을 설정하면 일정 시간 기다렸다가 메시지를 모아서 전달. 그러나, 배치가 가득 찬 경우엔 이 시간을 무시. |
| [batch.size](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#batch-size) | 같은 파티션에 대한 메시지를 모아서 전송하는데, 이 때의 배치에 들어 있는 메시지 메모리의 양을 결정(개수가 아니라 바이트 단위). 이 값은 배치 사이즈의 한계값이지, 이 만큼 무조건 기다리는 게 아님에 유의. 기다림을 가져가고 싶다면 `linger.ms` 사용. |
| [max.in.flight.requests.per.connection](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#max-in-flight-requests-per-connection) | 설정 이름 그대로의 역할. 책 보다는 문서 설명이 더 와닿음. "The maximum number of unacknowledged requests the client will send on a single connection before blocking". 값을 2로 설정할 때 처리량이 최대를 기록하고, 기본 값인 5를 사용해도 비슷하다고 설명. 그리고 만약, 이 값이 1보다 크고 `enable.idempotence=false`라면 재시도 과정에서 순서 보장 안 될 수 있음. 또한, 멱등성 설정을 사용하려면, 이 in-flight 값은 5 이하여야 함. |
| [max.request.size](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#max-request-size) | 한 번에 보낼 수 있는 메시지의 최대 개수. 하나의 요청이 너무 큰 것을 방지. 브로커의 `message.max.bytes` 설정과 동일하게 맞추는 것이 좋음. |
| [enable.idempotence](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.dsa_mt.dsa_rgn.apac_lng.eng_dv.all_con.docs&utm_term=&creative=&device=c&placement=&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8N6Ei3xbpzKhkIEqjfIy6Pwy0w2PJAQvjmpRogc0s27Vz44N88oncaAiIZEALw_wcB#enable-idempotence) | 스트림에 각 메시지가 정확히 한 번 쓰이는 것을 보장. 이 값이 제대로 동작하려면, `max.in.flight.requests.per.connection` 값은 5 이하여야 하고, `retries`는 0보다 커야 하며, `acks`는 `all`이어야 함. `acks=all`이고 `delivery.timeout.ms`가 꽤 큰 값일 때는 정확히 한 번이 아니라 최소 한 번 쓰여짐. 예를 들어, 전송 요청을 리더 브로커가 받아 레플리카 복제는 성공했는데, 프로듀서에게 응답을 보내기 전 크래시가 났다면, 프로듀서는 재시도를 하게 되고 이는 새로 선출된 리더에게 전달 되어 메시지가 중복 저장 됨. 멱등성 설정은 이를 막아줌. 내부적으로는 프로듀서가 레코드를 보낼 때 순차적 번호를 붙여서 보내고, 브로커는 값 중복 시 하나만 저장하며, 프로듀서는 DuplicateSequenceException 받아 이를 인지. |

## 3.6 파티션

- `ProduceRecord`에서 파티션과 값만 필수.
- 키는 선택적. 하지만 추가 정보를 나타내며, 파티션을 결정하는 기준점 역할도 함.
- 키가 없으면 파티션을 랜덤으로 지정. 이 때는 라운드 로빈 사용.
- sticky 처리 얘기도 나오는데 재밌음. 결국 효율을 위한 것.
- 키 값이 있고 기본 파티셔너를 사용한다면, 키를 해시한 결과로 파티션 지정.
- 만약 특정 파티션에 장애가 있다면 (드문 경우지만) 해당 파티션에 데이터 쓰기 시 에러 발생.
- 파티셔너는 파티션 상태를 고려하지 않음을 이야기하는 것. 언제나 모든 파티션을 대상으로 분배 알고리즘을 태움.
- 기본 파티셔너 외에 RoundRobinPartitioner, UniformStickyPartitioner가 있음.
- 키 분포가 불균형하다면 부하가 한쪽으로 몰릴 수 있는데 이 때 UniformStickyPartitioner를 사용하면 균등 분포를 위한 파티션 할당이 이뤄짐.
- 키와 파티션 대응은 파티션 수가 변하지 않는 한 변하지 않음.

### 3.6.1 커스텀 파티셔너 구현하기

```kt
class BananaPartitioner : Partitioner {

    override fun configure(configs: MutableMap<String, *>?) = Unit

    override fun close() = Unit

    override fun partition(
        topic: String,
        key: Any?,
        keyBytes: ByteArray?,
        value: Any?,
        valueBytes: ByteArray?,
        cluster: Cluster
    ): Int {

        val partitions = cluster.partitionsForTopic(topic)
        if ((keyBytes == null) || (key == null) || (key !is String)) {
            throw InvalidRecordException("Customer name is required as message key")
        }

        return if (key == "Banana") {
            partitions.size - 1;
        } else {
            abs(Utils.murmur2(keyBytes) % partitions.size - 1)
        }
    }

}
```

## 3.7 헤더

- 레코드는 파티션, 키, 값 외에 헤더 가질 수 있음.
- 헤더가 있으면 메시지 파싱 필요 없이 헤더로 메시지 라우팅이나 출처 추적이 가능.
- 키는 무조건 String 타입이어야 하고, 값은 직렬화 된 객체도 상관 없음.

## 3.8 인터셉터

- 주로 모니터링, 정보 추적, 표준 헤더 삽입 등에 인터셉터가 활용.
- 특히 메시지가 생성된 위치에 대한 정보를 심어 메시지 전달 경로를 추적하거나 민감 정보 삭제 처리 등에 사용.
- `ProducerInterceptor`는 2개의 메서드를 가짐.
- `ProducerRecord<K, V> onSend(ProducerRecord<K, V> record)`: 브로커로 레코드 보내기 전, 직렬화 직전에 호출. 레코드 정보를 보기도 하고 수정도 가능.
- `void onAcknowledgement(RecordMetadata metadata, Exception exception)`: 브로커가 보낸 응답을 클라이언트가 받았을 때 호출. 데이터를 변경할 수는 없고 읽을 수만 있음.
- 아래와 같은 인터셉터를 만들고 의존성으로 추가하고 설정 파일을 애플리케이션 실행 시 명시해주면 됨.

```kt
class CountingProducerInterceptor<K, V> : ProducerInterceptor<K, V> {

    private val sentCount = AtomicInteger(0)
    private val ackedCount = AtomicInteger(0)
    private val executorService = Executors.newSingleThreadScheduledExecutor()

    override fun configure(configs: MutableMap<String, *>) {
        val windowSize = (configs["counting.interceptor.window.size.ms"] as String).toLong()

        executorService.scheduleAtFixedRate(
            { printStatistics() },
            windowSize,
            windowSize,
            TimeUnit.MILLISECONDS
        )
    }

    private fun printStatistics() {
        val sent = sentCount.getAndSet(0)
        val acked = ackedCount.getAndSet(0)
        println("sent: $sent, acked: $acked")
    }

    override fun close() {
        executorService.shutdownNow()
    }

    override fun onAcknowledgement(metadata: RecordMetadata?, exception: Exception?) {
        ackedCount.incrementAndGet()
    }

    override fun onSend(record: ProducerRecord<K, V>): ProducerRecord<K, V> {
        sentCount.incrementAndGet()
        return record
    }

}
```

## 3.9 쿼터, 스로틀링

- 브로커에 쓰기/읽기 속도를 제한할 수 있는 기능이 쿼터<sup>quota</sup>.
- 3가지 쿼터 타입 있음.
  - produce quota
  - consume quota
  - request quota: 브로커의 요청 처리 제한
- 기본값을 설정할 수도 있고, 특정 `client.id`에 대해 지정할 수도 있으며, 사용자 단위도 가능.
- 단, 사용자 단위 설정은 보안 기능과 클라이언트 인증 기능이 활성화 된 클라이언트에 대해서만 작동.
- **브로커 설정**에 `quota.producer.default=2M`이라고 명시하면, 각 프로듀서의 초당 쓰기 데이터를 2M로 제한.
- 만약 클라이언트가 쿼터를 다 채우면, 브로커는 스로틀링을 시작.
- 구체적으로는, 클라이언트 요청에 대한 응답을 늦게 보내준다는 의미.
- 클라이언트에는 `max.in.flight.requests.per.connection` 설정이 있으니 요청 속도를 줄이게 되고 쿼터를 넘지 않게 됨.
- 만약 이를 무시하는 클라이언트의 요청이 게속 들어온다면, 해당 커뮤니케이션 채널을 일시적으로 무시해서 브로커를 보호.
- JMX 메트릭으로 스로틀링 작동 여부도 확인 가능.

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

# 6장. 카프카 내부 메커니즘

사용자 입장에서 알아두면 좋을 카프카 내부 작동 방식 이야기.

- 카프카 컨트롤러
- 복제 작동 방식
- 프로듀서와 컨슈머가 요청을 처리하는 방식
- 파일 형식이나 인덱스 등 카프카가 저장을 처리하는 방식

## 6.1 클러스터 멤버십

- 각 브로커는 고유한 식별자를 가짐.
- 식별자는 브로커 설정 파일에 저장되어 있거나 자동 생성됨.
- 그리고 브로커 목록은 아파치 주키퍼에 등록되어 관리.
- 브로커 프로세스는 시작 시 주키퍼에 [Ephemeral 노드](https://zookeeper.apache.org/doc/r3.4.6/zookeeperOver.html#Nodes+and+ephemeral+nodes) 형태로 ID 등록.
- 카프카 컨트롤러 등은 주키퍼의 `/brokers/ids` 경로를 구독해 브로커의 추가/제거를 알림 받음.
- 브로커와 주키퍼 연결이 끊어지면 브로커가 생성한 Ephemeral 노드는 주키퍼에서 자동으로 삭제.
- 브로커 목록의 구독자는 이를 인지하게 됨.
- 브로커 정지 시 브로커를 나타내는 ZNode는 삭제되지만, 브로커 ID는 다른 자료구조에 여전히 남아 있음.
- 예를 들어, 각 토픽의 레플리카 목록에는 브로커 ID가 남아 있고, 동일 ID의 브로커를 새로 투입하면, 유실된 브로커 자리를 대체해 이전 브로커의 토픽과 파티션을 할당받음.

## 6.2 컨트롤러

- 컨트롤러는 브로커 기능에 더해 파티션 리더 선출 역할을 추가로 수행.
- 클러스터에서 가장 먼저 시작하는 브로커가 주키퍼의 `/controller`에 Ephemeral 노드를 생성해 컨트롤러가 됨.
- 컨트롤러 변경을 감지하기 위해 다른 브로커들은 이 노드를 와치.
- 컨트롤러가 멈추거나 주키퍼와 연결이 끊어지면 이 Ephemeral 노드는 삭제됨.
- `zookeeper.session.timeout.ms` 설정 값보다 더 오래 주키퍼에 하트비트 전송하지 않는 것도 이에 해당.
- 컨트롤러가 없어짐을 감지한 다른 브로커들은 주키퍼에 컨트롤러 노드 생성 요청.
- 가장 먼저 성공한 브로커가 컨트롤러가 됨.
- 새로운 컨트롤러는 주키퍼의 conditional increment 연산에 의해 증가한 epoch 값을 가짐.
- 브로커들은 현재 컨트롤러의 epoch 값을 알고 있고, 만약 더 낮은 예전 값을 가진 컨트롤러에게 메시지를 받으면 무시함.
- 브로커가 컨트롤러가 되면 주키퍼로부터 최신 레플리카 상태 맵을 읽어옴.
- 그리고 클러스터 메타데이터 관리와 리더 선출을 시작.
- 브로커가 클러스터를 나갔음을 컨트롤러가 알게 되면, 브로커가 리더를 맡고 있던 모든 파티션에 대해 새로운 브로커를 결정.
- 그리고 새로운 상태를 주키퍼에 쓴 뒤, 파티션의 레플리카를 가진 모든 브로커에게 `LeaderAndISR` 요청을 보냄.
- 여기에는 파티션들에 대한 새로운 리더와 팔로워 정보가 담겨 있음.
- 마지막에 스플릿 브레인<sup>split brain</sup> 용어도 잠시 언급됨.
- 아까 얘기했던, 스스로 컨트롤러라 생각하는 브로커가 2개 이상 존재하는 것. 이를 막기 위해 epoch 번호를 사용.
