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
