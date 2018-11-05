# Redis cluster tutorial

https://redis.io/topics/cluster-tutorial 내용 읽고 간단 정리. 자세한 내용은 [Redis Cluster specification](https://redis.io/topics/cluster-spec)를 참고.

### Redis Cluster 101

1. 자동으로 데이터 샤딩 (automatically sharded across multiple Redis nodes)
2. 일부 노드의 장애나 통신 실패에도 지속적인 운영이 가능 (some degree of availability during partitions)

### Redis Cluster TCP ports

1. 총 2개의 TCP 포트를 사용.
2. command port ― 클라이언트의 요청을 받는 포트.
3. cluster bus port ― 노드 간의 통신을 위해 사용되는 포트.
4. bus 포트는 command 포트 값에 무조건 10,000을 더한 값을 사용.

### Redis Cluster and Docker

1. Redis Cluster는 NATted 환경과 IP 주소나 TCP 포트가 다시 매핑되는 환경을 지원하지 않음.
2. Docker의 [host networking mode](https://docs.docker.com/network/host/)를 사용해야 함.

### Redis Cluster data sharding

1. RC는 consistent hashing을 사용하지 않음.
2. 대신, 16384개의 해시 슬롯<sup>hash slot</sup>이 있고, 주어진 키를 해싱하여 어디로 보낼지를 결정.
3. `crc16(key) % 16384`
4. 그리고 모든 노드는 이런 해시 슬롯들의 부분 집합.
5. 새로운 노드가 추가되면, 각 노드에서 일부 해시 슬롯들을 새로운 노드로 이동시킴.
6. [Fixed Number of Partitions](https://github.com/codehumane/what-i-learned/blob/master/book/ddia/Distributed-Data.md#fixed-number-of-partitions)와 동일.
7. 참고로, 해시 슬롯을 옮기는 것은 다운타임 없이 운영 중에 가능.
8. RC는 multiple key operation들을 지원.
9. 단, 키로 찾으려는 값들이 같은 해시 슬롯에 속해 있어야 함.
10. [해시 태그<sup>hash tags</sup>](https://redis.io/topics/cluster-spec#keys-hash-tags)를 통해 이 문제를 어느 정도 극복할 수 있음.

1. 가용성 확보
  - 일부 마스터 노드가 장애이거나,
  - 다른 대다수의 노드와 통신을 못하는 등의 상황에서의.
2. 1 master to N replicas

### Redis Cluster consistency guarantees

1. 강한 일관성<sup>strong consistency</sup>는 보장하지 못함.
2. 즉, 특정 상황에서 (클라이언트에게 성공했다고 응답했음에도 불구) 데이터를 유실할 수 있음.
3. 이런 이유 중의 하나는 비동기 레플리케이션.
  - 클라이언트가 마스터 B에게 씀.
  - 마스터 B는 OK를 응답.
  - B는 쓰기를 슬레이브 B1, B2에게 전달.
  - 하지만, 이 전달을 끝내지 못하고 B가 죽음.
  - B1, B2 중에 하나가 마스터로 승격되어도 데이터는 유실.
4. MySQL의 `innodb_flush_log_at_trx_commit` 설정 이슈와 유사한 문제.
5. 성능을 포기하고 일관성을 높이고자 동기 레플리케이션을 사용([WAIT](https://redis.io/commands/wait) 이용)해도 유실을 완벽히 막을 순 없음.
6. 아래는 또 다른 데이터 유실의 사례.
  - 마스터 노드 A,B,C와 슬레이브 A1,B1,C1, 그리고 Z라는 클라이언트가 존재.
  - 네트워크 단절로 인해 A,C,A1,B1,C1과 B,Z1의 파티션 발생.
  - Z1은 여전히 B에게 쓸 수 있음.
  - 일정 시간이 지나서 B1이 마스터로 승격.
  - B에 쓰여졌던 데이터는 유실.
7. 이런 이유로 maximum window, node timeout 설정은 중요.
