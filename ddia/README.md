# Designing Data-Intensive Application

## Data Models ans Query Languages

데이터 모델은 소프트웨어가 데이터를 어떻게 쓸지에 대한 것 뿐 아니라, 우리가 문제를 해결하는 방식에도 영향을 미침. 따라서, 애플리케이션 특성에 따라 적절한 솔루션을 도입하는 것이 중요. 여기서는 특히 관계형과 문서형 모델, 그리고 일부 그래프 기반 데이터 모델을 설명. 또한, 여러 질의 언어와 각 쓰임에 대해서도 비교. 참고로, 3장은 이들의 내부 동작 원리에 대해 알아본다.

### Relational Model Versus Document Model

TBD

## Storage and Retrieval

### Data Structures That Power Your Database

```bash
#!/bin/bash

db_set () {
    echo "$1,$2" >> database
}

db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

- 아주 간단한 데이터베이스.
- 매 라인마다 쉼표로 구성된 키-값 쌍의 텍스트 파일을 활용.
- 기존 값을 갱신하지 않고 계속 쌓기만 하는 append-only.
- 데이터 베이스의 로그가 바로 이런 형태.
- 로그가 너무 커지지 않게 디스크 공간 회수나 오류 처리, 동시성 제어 등의 문제 해결이 필요하긴 하지만, 기본적으로는 같은 원리.
- 하지만, 레코드가 많아지면 검색 성능이 나빠짐. 특정 키를 찾기 위해, 처음부터 끝까지 스캔해야 하기 때문. O(n).
- 키 찾기의 효율성을 위해 활용되는 것이 바로 인덱스.
- 하지만 쓰기 속도는 느려짐.
- 이런 트레이드 오프로 인해 데이터베이스는 모든 것을 인덱싱하지 않음. 사용자의 선택.

#### Hash Indexes

키-값 저장소를 해시 맵으로 인덱싱하기.

- [해시 맵 인덱스 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0301.png) 참고.
- 키를 데이터 파일의 바이트 오프셋에 매핑해 인메모리 해시 맵을 유지.
- 단순하지만 Bitcask 등 실제로 많이 사용됨.
- 고유키가 많지 않으면서도(메모리에 적재해야 하므로) 값의 갱신이 빈번한 경우에 유리.

만약, 디스크 상의 로그 구조화 파일에 계속 append만 되서 공간이 부족해 진다면?

- 로그를 특정 크기의 세그먼트<sup>segment</sup>로 나누기.
- 세그먼트가 특정 크기에 도달하면, 세그먼트를 닫고 이후의 쓰기는 새로운 세그먼트에 수행.
- 닫힌 세그먼트에 대해서는 일반적인 컴팩션<sup>compaction</sup> 수행.
- 당연히, 여러 개의 세그먼트에 대해서도 컴팩션 가능.

구현 시 주의 사항들

- CSV 형식은 로그에 부적합. 문자열 길이를 바이트 단위로 인코딩 후, 원시 문자열을 인코딩 하는 것이 빠르고 간편함. 이스케이핑도 필요 없어짐.
- 키 관련 값을 삭제하려면 특수한 삭제 레코드(tombstone이라고도 불림)을 추가. 세그먼트 병합 시 이 키의 이전 값들은 무시됨.

추가 전용 로그의 장단점

- 순차적 쓰기 작업으로, 무작위 쓰기에 비해 훨씬 빠름.
- 동시성과 고장 복구에 유리. 고장의 경우 별도의 파일에 추가한 뒤 병합하면 됨.
- 반면, 해시맵은 메모리에 저장해야 빠르므로, 너무 큰 인덱싱은 불가.
- 범위 질의가 비효율적. 해시 맵에서 일일이 모든 개별 키를 조회해야 함.

#### SSTables and LSM-Trees

먼저, 로그 구조화 저장소에 대한 간단한 요약.

- 로그 구조화 저장소 세그먼트는 [이 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0303.png)에서 보듯이 키-값 쌍의 연속.
- 이 쌍은 쓰여진 순서대로 나타남. 또한, 키 값이 같으면 나중의 값이 이전 값에 우선함.

SS 테이블의 등장.

- 만약, 세그먼트에서 일련의 키-값 쌍을 키로 정렬해야 한다면?
- [이 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0304.png)에서 보는 것 처럼 세그먼트 병합시 정렬 상태를 만들 수 있다.
- 이렇게 키로 정렬된 형식을 가리켜 정렬된 문자열 테이블<sup>Sorted String Table</sup>이라고 함. 또는 SS 테이블.

SS 테이블은 해시 인덱스를 가진 로그 세그먼트에 비해 몇 가지 장점을 가짐.

- 세그먼트 병합은 파일이 사용 가능한 메모리보다 크더라도 간단하고 효율적. 병합정렬 알고리즘과 유사하기 때문. (다만, 이게 왜 해시 인덱스를 가진 세그먼트의 병합 과정에 비해 장점인지는 잘 모르겠음)
- 더 이상 파일에서 특정 키를 찾기 위해 메모리에 모든 키의 인덱스를 유지할 필요가 없음. (DB 인덱스를 생각하면 금방 이해 됨)
- 레코드들을 블록으로 그룹화하고, 디스크에 쓰기 전에 압축할 수 있음. 인메모리 인덱스는 압축된 블록의 시작을 가리키면 됨. 이렇게 하면, 디스크 공간 절약과 I/O 대역폭 사용도 줄임.

##### Constructing and Maintaining SSTables

SSTables를 어떻게 유지할 수 있을까?

- 쓰기가 유입되면, in-memory balanced tree 데이터 구조(red-black tree와 같은)에 추가. 이런 인메모리 트리를 가리켜 종종 `memtable`이라고도 부름.
- memtable의 크기가 임계치를 넘어가면, SSTable 파일로 옮긴다. 트리가 이미 키로 정렬된 키-밸류 쌍이므로 효율적으로 수행 가능. 새로운 SSTable이 디스크에 작성되는 동안, 쓰기는 새로운 memtable 인스턴스에서 계속 됨.
- 읽기 요청을 처리하기 위해, 우선 memtable에서 키를 찾고, 다음으로 가장 최신의 디스크 세그먼트, 그리고 그 다음 세그먼트, ...를 찾는다.
- 종종 백그라운드에서 병합과 압축 과정을 수행. 세그먼트 파일을 합치면서 덮어씌워졌거나 삭제된 값을 버림.

하지만, 만약 데이터베이스가 고장나면?

- 최근의 쓰기들은 유실될 수 있음.
- 이를 피하기 위해 별도의 분리된 로그를 유지.
- 이 로그에는 쓰기가 발생하는 순서 그대로 즉각 기록.
- 고장시 복구를 위한 목적이므로 순서는 상관 없음.
- memtable이 SSTable로 옮겨질 때 마다, 대응되는 로그들은 버려짐.

##### Making an LSM-Tree Out Of SSTables

- 여기서 소개된 알고리즘은 LevelDB와 RocksDB 등에서 사용됨.
- Cassandra와 HBase에는 이와 유사한 저장소 엔진이 사용됨.
- 이런 인덱싱 구조는 원래 *Log-Structured Merge-Tree (LSM-Tree)*라는 이름으로 소개되었음.
- 정렬된 파일을 병합하고 압축하는 원칙에 기반하는 저장소 엔진들은 종종 LSM 저장소 엔진이라 불림.
- 루신(Elasticsearch와 Solr에서 사용되는 인덱싱 엔진)은 *term dictionary*를 저장하기 위해 비슷한 방식을 사용. 키는 단어(*a term*), 값은 이 단어가 포함된 모든 문서의 ID 목록(*the postings list*). 이 매핑이 SSTable 같은 정렬된 파일에 유지됨. 그리고 필요에 따라 백그라운드에서 병합됨.

#### B-Trees

- SS(Sorted String)테이블과 같이 정렬된 키-값 쌍을 유지.
- 따라서, 키-값 검색과 범위 질의에 효율적.
- 가변적 단위인 세그먼트 아니라, (일반적으로) 4KB라는 고정 크기의 블록이나 페이지 단위로 나눔.
- 한 페이지에서 참조하는 하위 페이지의 수를 가리켜 분기 계수<sup>branching factor</sup>라고 함. 보통 수백 개에 달함.
- 키 값의 갱신은 [여기 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0307.png) 참고.

##### Making B-Trees Reliable

- 로그 구조화 인덱스와 대조적으로 새로운 데이터를 기존 페이지에 덮어 씀.
- 일부 동작은 여러 다양한 페이지의 덮어쓰기가 수반됨. 고아 페이지에 유의.
- 고장 복구를 위해 디스크 상의 쓰기 전 로그<sup>write-ahead log</sup>를 쌓음. 재실행 로그<sup>redo log</sup>라고도 불림.
- 다중 스레드가 동일한 페이지 갱신에 유의. 보통 레치<sup>latch</sup> 즉, 가벼운 잠금으로 이를 보호함.

##### B-Tree Optimizations

- 페이지를 오버라이트하거나 WAL을 유지하는 대신, 일부 데이터베이스들은 copy-on-write 스키마를 사용. 동시성 제어에 유리. 일종의 READ-COMMITTED.
- 키 전체 대신 축약 버전을 저장. B+ 트리를 일컫는 것.
- 리프 페이지들은 디스크 상에서 연속된 순서로 나타나게끔 배치를 시도.
- 트리에 포인터 추가. 리프 페이지가 양쪽 형제들에 대한 참조를 가지면, 상위 페이지로 가지 않아도 순서대로 키 스캔이 가능.

#### Comparing B-Trees and LSM-Trees

경험적으로 LSM-트리가 쓰기에 빠르고, B-트리가 읽기에 빠름.

##### Advantages of LSM-Trees

- B-트리는 매번 적어도 두 벌의 쓰기를 해야 함. 하나는 write-ahead log, 다른 하나는 트리 페이지 그 자체. 때로는 페이지의 몇 바이트만을 바꾸기 위해 전체 페이지를 한 번에 기록해야 하기도.
- 로그 구조화 인덱스도 SSTable을 병합하고 압축하는 것을 반복하기 때문에 여러 번 데이터를 기록. 한 번의 쓰기가 데이터베이스의 생명주기 동안 여러 번의 쓰기를 야기할 수 있다는 점에서 쓰기 증폭<sup>write amplification</sup>. 
- 쓰기가 많은 애플리케이션에서는 디스크에 기록하는 속도가 성능의 병목이 됨. 따라서, 쓰기 증폭은 성능에 직접적인 비용이 됨. 저장소 엔진이 디스크에 많이 쓸수록, 디스크 대역폭은 줄어들기 때문.
- 제목은 LSM-트리의 장점인데 왜 이런 얘기가 계속 나오는지 의아함.
- 하지만, LSM-트리는 B-트리에 비해 일반적으로 더 좋은 성능을 보임. 종종 낮은 쓰기 증폭을 가지고, 트리에서 처럼 여러 페이지에 대해 덮어쓰기를 하기 보다는 차례대로 SSTable 파일들을 압축하기 때문. 순차적 쓰기가 랜돔 쓰기에 비해 훨씬 더 좋은 성능을 보이는 마그네틱 하드 드라이브에서 특히 중요한 차이.
- LSM-트리가 더 잘 압축되므로 더 작은 디스크 용량을 차지. 반면, B-트리는 파편화로 인해 사용되지 않는 디스크 공간이 남게 됨.

##### Downsides of LSM-Trees

- 압축 과정이 때때로 진행중인 읽기와 쓰기 성능에 간섭을 줄 수 있음.
- 압축 과정은 또한 initial write(memtable → 디스크 로깅과 플러시)가 일어날 때 성능 문제가 될 수 있음.
- 쓰기가 늘어나는 속도를 압축이 따라가지 못하면 디스크 공간이 부족해 질 수 있음.
- 로그 구조화 저장소 엔진은 중복된 키가 여러 세그멘트에 걸쳐 존재할 수 있어 트랜잭션 처리에 불리. 트랜잭션 격리는 키 범위에 대한 잠금을 이용해 구현됨.

#### Other Indexing Structures

기본키 인덱스와 보조 인덱스.

- 지금까지 키-값 인덱스를 살펴봄.
- 키-값 인덱스의 대표적인 예는 기본키 인덱싱.
- 보조 인덱스<sup>secondary index</sup>도 많이 사용됨.
  - 이는 효율적인 조인 수행에 결정적 역할.
  - RDB에서는 `CREATE INDEX` 명령을 사용.
  - 기본키와의 차이는 키가 고유하지 않을 수도 있다는 것.

##### Storing Values Within The Index

인덱스 값의 2가지 종류.

- 인덱스에서의 값은 2가지 중 하나.
- 실제 row(문서, 정점)이거나, 저장된 row를 가리키는 참조.
- 후자의 경우 row가 저장된 곳을 힙 파일<sup>heap file</sup>이라 부르고 순서 없이 데이터를 저장.
- 여러 보조 인덱스가 존재할 경우, 데이터 중복을 피할 수 있다는 점 때문에 후자의 접근 방식이 더 일반적.

힙 파일에서의 값 변경.

- 키의 변경 없이 값만 변경하는 경우라면, 힙 파일 접근법이 효율적.
- 새로운 값이 이전 값 보다 크지만 않다면 그 자리에서 대체될 수 있기 때문.
- 하지만, 새로운 값이 더 크다면, 새로운 공간에 저장되어야 하고, 모든 인덱스를 변경하거나, 포워딩 포인터를 오래된 힙 위치에 남겨두어야 함.

인덱스 된 로우를 인덱스 안에 저장하기.

- 익덱스 자체에 값을 넣어두는 것은 읽기의 성능을 높임.
- 클러스터드 인덱스<sup>clustered index</sup>라고 알려져 있음.
- MySQL의 InnoDB 저장소 엔진에서는 기본키가 항상 클러스터드 인덱스. 그리고 보조 인덱스가 기본키를 참조함.
- SQL Server에서는 테이블 별로 클러스터드 인덱스를 지정할 수 있음.

커버링 인덱스.

- 클러스터드 인덱스와 비 클러스터드 인덱스의 절충안으로, 테이블의 일부 컬럼을 인덱스에 저장할 수 있음.
- 이를 가리켜 커버링 인덱스<sup>covering index</sup> 또는 클러스터드 컬럼이 있는 인덱스<sup>index with included columns</sup>라고 부름.
- 오직 인덱스 만을 이용해서 쿼리에 응답할 수 있게 되는 것.
- 이런 경우를 cover 한다고 말한다. (캐시의 hit가 생각남)
- 읽기의 성능은 높이지만, 추가적인 저장 공간이 필요하고 쓰기의 부하를 더함.
- 또한 DB 입장에서는 트랜잭션을 보장하기 위한 추가적인 노력도 필요해짐.

##### Multi-Column Indexes

결합 인덱스concatenated index</sup>.

- 가장 흔한 다중 컬럼 인덱스 
- 단순히 몇 개의 필드를 연결해서 하나의 키로 만드는 것.
- 전화번호부에서 번호에 대한 (lastname, firstname) 인덱스를 제공하는 것이 한 예.
- lastname으로 번호를 찾거나, lastname-firstname 조합으로 번호를 찾을 때 유리하지만, firstname은 인덱스로 사용 불가.

다중 인덱스<sup>multi-dimensional indexes</sup>.

- 지리 데이터와 같이, 여러 컬럼에 대해 한 번에 질의를 보내는 방법.
- 예를 들면 아래와 같음.

```sql
SELECT *
FROM restaurants
WHERE latitude > 51.4946 AND latitude < 51.5079
	AND longitude > -0.1162 AND longitude < -0.1004;
```

- 표준 B-트리나 LSM-트리는 이런 질의에 효율적으로 응답하기 어려움.
- 한가지 대안은 이차원 위치를 공간-채움 곡선<sup>space-filling curve</sup>(?)를 이용해 1개의 숫자로 변환하여 사용하는 것.
- 좀 더 일반적인 방법은 R-트리처럼 specialized spatial index를 이용하는 것.
- 다중 인덱스는 지리적 위치 이외에도, 커머스에서의 제품 색상 검색(red, green, blue)이나, 날짜 관측을 위한 검색(date, temperature) 등에도 사용.
- 1차원 색인은 한 가지 값으로 필터링 후, 2번째 값으로 추가적인 필터링을 하는 반면, 다차원 인덱스는 주어진 값으로 동시에 범위를 줄일 수 잇음.

##### Full-Text Search and Fuzzy Indexes

오타처럼 유사한 키에 대한 검색을 허용해 주는 퍼지<sup>fuzzy</sup> 질의에 대한 이야기.

- 동의어로 질의를 확장해 주거나,
- 단어의 문법적 변형은 무시하거나,
- 동일 문서 내에 서로 인접해 나타난 단어를 검색해주거나,
- 언어학적인 분석을 통한 여러 다양한 기능들을 지원함.
- 루씬은 편집 거리<sup>edit distance</sup> 범위 내의 허용된 단어를 검색하기도 함.

##### Keeping Everything in Memory

인메모리 데이터베이스의 현실화.

- 디스크는 메인 메모리와 비교해 다루기 어렵다고 함. 그럼에도 불구하고 지속성<sup>durability</sup>과 RAM 대비 비용이 저렴하기 때문에 계속 사용됨.
- 하지만 최근에는 RAM 비용이 점점 낮아지면서 인메모리 데이터베이스가 점점 현실성을 갖게 됨.

내구성<sup>durability</sup>.

- Memcached와 같은 일부 키-값 저장소는 캐싱 용도로만 사용됨. 머신의 재시작으로 데이터가 유실될 수 있음을 인정.
- 하지만 지속성을 추구하는 인메모리 데이터베이스도 존재. 특수 하드웨어를 사용하거나, 디스크에 변경 로그를 남기거나, 디스크에 주기적으로 스냅샷을 남기거나, 인메모리의 상태를 다른 머신으로 복제하거나 하는 방법들을 사용함.
- 인메모리 데이터베이스가 재시작되면, 디스크나 네트워크를 통해 레플리카로부터 상태를 재적재. 단지 지속성을 위해 디스크에 append-only 로그를 남길 뿐. 읽기는 완전히 메모리를 통해 이루어짐.
- 디스크를 함께 사용하는 것은 추가적인 이점을 안겨줌. 외부 도구들을 사용해, 백업하고 검사하고 분석하기 쉬움.
- 참고로, Redis와 Couchbase는 디스크에 비동기로 쓰기 때문에 상대적으로 약한 지속성을 제공.

왜 빠른가.

- 직관과 다르게, 디스크로부터 읽지 않아서 빠른 것은 아님.
- OS가 최근 사용된 디스크 블럭들을 메모리에 적재하기 때문에, 충분한 메모리가 있다면 디스크 기반의 저장소 엔진들도 메모리로부터 데이터를 읽어 들이는 것.
- 인메모리 데이터페이스가 빠른 이유는 디스크에 쓰기 위한 인코딩 오버헤드로부터 자유롭기 때문.

다양한 데이터 모델.

- 디스크 기반의 인덱스로는 구현하기 어려운 데이터 모델들을 제공.
- 예컨대, Redis는 우선순위 큐나 셋 같은 다양한 데이터 구조체를 데이터베이스 같은 인터페이스를 통해 제공. 모든 데이터가 메모리에 있기 때문에, 상대적으로 구현하기 쉽다.

### Transaction Processing or Analytics?

시작하면서 재밌는 내용이 나옴.

> In the early days of business data processing, a write to the database typically corresponded to a *commercial transaction* taking place: making a sale, placing an order with a supplier, paying an employee’s salary, etc. As databases expanded into areas that didn’t involve money changing hands, the term *transaction* nevertheless stuck, referring to a group of reads and writes that form a logical unit.

위 언급된 내용처럼, 데이터베이스는 OLTP(Online Transaction Processing)이었지만, 데이터 분석 등과 같은 OLAP(Online Analytic Processing)으로도 발전되고 사용됨. 명확한 구분은 어렵지만 주요 차이로는 아래와 같은 것들이 있음. 특별한 내용은 아니지만 표현이 재미있음.

| Property             | OLTP                                              | OLAP                                      |
| -------------------- | ------------------------------------------------- | ----------------------------------------- |
| Main read pattern    | Small number of records per query, fetched by key | Aggregate over large number of records    |
| Main write pattern   | Random-access, low-latency writes from user input | Bulk import (ETL) or custom stream        |
| Primarily used by    | End user/customer, via web application            | Internal anayst, for decision support     |
| What data represents | Latest state of data (curret point in time)       | History of events that happened over time |

처음에는 이 두 가지 용도로 동일한 데이터베이스가 사용되었지만(이런 면에서 꽤나 유연하다), 점차 별도의 분석용 데이터베이스를 사용하기 시작함. 이를 가리켜 데이터 웨어하우스라고 부름.

#### Data Warehousing

OLTP 데이터베이스는 읽기 전용의 복제(OLTP 로부터의) 데이터를 가짐. ETL(Extract-Transform-Load)라고 불림. 이 과정을 잘 나타낸 그림은 [여기](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0308.png)를 참고. 앞에서 살펴 본 인덱싱 알고리즘을 보면 어느 정도 알 수 있듯이, OLTP는 분석적 질의에는 응답하기에 좋지 않음. 반면, 데이터 웨어하우스의 저장소 엔진은 이런 일에 최적화 되어 있음.

#### Stars and Snowflakes: Schemas for Analytics

많은 데이터 웨어하우스는 star schema(dimensional modeling이라고도 알려져 있음)라는 데이터 모델을 주로 사용함. [여기](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0309.png)를 보면 예시 스키마가 나와 있음. [Amazon Redshift의 샘플 데이터베이스](https://docs.aws.amazon.com/ko_kr/redshift/latest/dg/c_sampledb.html)와 유사함. 여기서의 테이블은 크게 2가지로 나뉨. fact와 dimension. fact는 이벤트를 표현하고, dimensions는 이벤트에 대한 who, what, where, when, how, 그리고 why를 나타냄.

이 dimension에는 날짜와 시간만이 담긴 테이블도 존재함. 이 테이블의 존재 이유는 아래와 같이 설명함.

> Even date and time are often represented using dimension tables, because this allows additional information about dates (such as public holidays) to be encoded, allowing queries to differentiate between sales on holidays and non-holidays.

[여기](https://docs.aws.amazon.com/ko_kr/redshift/latest/dg/r_datetable.html)의 예시를 가져오면 다음과 같음.

| 열 이름 | 데이터 형식 | 설명                                                         |
| ------- | ----------- | ------------------------------------------------------------ |
| DATEID  | SMALLINT    | 각 행의 고유 ID 값을 나타내는 기본 키입니다. 각 행마다 연중 날짜를 나타냅니다. |
| CALDATE | 날짜        | 날짜(**2008-06-24** 등)입니다.                               |
| 일      | CHAR)       | 짧은 형식의 주중 요일(**SA** 등)입니다.                      |
| 주      | SMALLINT    | 주의 수(**26** 등)입니다.                                    |
| Month   | CHAR)       | 짧은 형식의 월 이름(**JUN** 등)입니다.                       |
| QTR     | CHAR)       | 분기 수(**1**~**4**)입니다.                                  |
| YEAR    | SMALLINT    | 4자리 연도(**2008** 등)입니다.                               |
| holiday | 부울        | 공휴일(미국 기준) 여부를 나타내는 플래그입니다.              |

참고로, "star schema"라고 이름 지어진 이유는 테이블들의 관계가 마치 별 같기 때문. fact 테이블이 가운데에 있고, 이를 dimension 테이블들이 둘러싸는 모습.

이 템플릿의 변형으로는 snowflake schema가 있음. dimension들이 subdimension들로 좀 더 쪼개진 것. 예를 들어, 상품 테이블이 브랜드나 카테고리와 같이 좀 더 분리되는 것. snowflake schema가 좀 더 정규화 되었지만, star schema가 좀 더 사용하기에 편리해서 더 자주 쓰인다고 함.

데이터 웨어하우스에서의 fact 테이블들은 종종 컬럼을 100개 이상씩 소유한다고 함. 때로는 수백개. dimension 테이블들도 마찬가지. 예를 들어, `dim_store`에는 in-store baekery가 있는지, 상점 면적은 얼만지, 개점일은 언제인지, 마지막 리모델링 일자는 언제인지, 공공 도로에서 얼마나 먼지 등을 모두 담고 있음.

### Column-Oriented Storage

row-oriented는 한 로우의 값을 연속되게 저장함. document 데이터베이스도 마찬가지. 수백개의 컬럼으로 구성된 fact 테이블에서 단지 3개의 컬럼 데이터만을 필요로 한다면 비효율이 발생할 수 있는 구조인 것. 3개의 컬럼만이 필요한데, 수백개의 속성을 모두 메모리로 불러들여야 할 수도 있음.

한편, column-oriented storage는 아래와 같은 방식으로 데이터를 저장한다.

> don’t store all the values from one row together, but store all the values from each *column* together instead. If each column is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work.

#### Column Compression

필요한 컬럼만을 디스크에 질의할 수 있는 것 외에, column-oriented는 데이터를 압축하는데도 유리함. 한 가지 방법은 bitmap encoding이고, 이를 설명하는 그림은 [여기](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0311.png)를 참고. 방식을 간단히 설명하면 다음과 같음. 참고로, distinct 값의 갯수는 로우 갯수에 비해 적다는 점을 전제로 함. 예를 들어, 상품은 여러 번 판매될 수 있지만, 대상 상품은 몇 개 없을 수 있음.

1. Bitmap for each possible value
   - n개의 distinct 값을 n개의 구분된 비트맵으로 변환.
   - 값 별로 어느 로우에 속하는지를 0과 1로 표기.
2. Run-length encoding
   - 9,1 = 9 zeros, 1 one, rest zeros
   - 0,4 = 0 zeros, 4 ones

#### Sort Order in Column Storage

컬럼 저장소에서는 로우가 추가되는 순서 대로 컬럼 파일들에 데이터를 덧붙이는 게 가장 쉬움. 하지만, 앞서 SSTable에서 했던 것 처럼 순서를 부과하고 인덱싱 메커니즘을 활용할 수도 있음.

- 정렬은 전체 로우 단위로 이뤄짐.
- 정렬 기준은 쓰임새에 따라 컬럼을 선정. 복수 개를 선정할 수도 있음.
- 예를 들어, 주로 날짜를 기준으로 테이블 데이터가 정렬된다면 날짜를 기준 컬럼으로 선정.
- 2차 컬럼으로 상품 아이디 등을 활용할 수도 있음.
- 이렇게 정렬된 컬럼은 압축에도 유리.
- 첫 번째 정렬 컬럼에만 효과가 크겠지만, 그럼에도 불구하고 전체적인 편익은 더 크다고 함.

##### Several Different Sort Orders

이 아이디어의 확장된 버전도 있음. C-Store에 소개되고, Vertica라는 사용 웨어하우스에 도입되었다고 함. 간단히 소개하면 다음과 같음.

> so why not store the same data in *several diffrent ways*?

- 같은 데이터지만 다르게 정렬된 버전을 여러 개 가지고 있는 것.
- 데이터 유실 방지를 생각한다면, 여러 머신에 복제되어 저장되기도 해야 함.
- 여러 기준으로 정렬된 질의 결과도 받고, 데이터 유실도 방지하는 것.

row-oriented에서도 보조 인덱스를 가질 수 있지만, 이 때는 데이터가 오직 한 곳에만 저장되고, 보조 인덱스는 로우의 위치를 가리킬 뿐. column-oriented는 포인터가 없다. 오직 값을 가지는 컬럼들 만이 있을 뿐.

#### Writing to Column-Oriented Storage

하지만 단점도 존재. 바로 쓰기의 어려움. 정렬된 테이블의 중간에 데이터를 삽입해야 한다면, 모든 컬럼 파일들을 대부분 재작성해야 할 것.

LSM-트리는 좋은 해결 방안이 됨. 모든 쓰기는 최초에 메모리 저장소로 향함. 여기서 정렬된 형태를 유지하며 디스크에 쓰여지길 기다림. 충분한 양이 쌓이면 디스크 상의 컬럼 파일들에 대량 머지됨. 실제로 Vertica가 그렇게 한다고 한다. 다만, 질의는 디스크와 메모리 모두에 대해 이뤄져야 함. 그리고 결과를 조합. 물론, 사용자가 하는 일은 아니겠지만 말이다.

#### Aggregation: Data Cubes and Materialized Views

데이터 웨어하우스의 또 다른 중요한 요소 중의 하나는 materialized aggregates. COUNT, SUM, AVG, MIN, MAX 등의 애그리거트 함수가 종종 사용되는데, 매번 이를 원천 데이터에 대해 수행하는 것은 낭비가 될 수 있음. 자주 사용되는 횟수나 합계를 캐시해 두는 것은 어떨까.

이런 캐시를 생성하는 방법 중의 하나가 바로 materialized view. 이는 쿼리 결과를 복사해서 디스크에 저장해둔 것. 관련된 데이터가 바뀔 때는 이 뷰가 함께 갱신되야 함. 이 비용은 만만치 않으나 읽기가 주로 발생하고 또 중요한 데이터 웨어하우스에서는 비용이 상쇄됨.

materialized view의 한 예는 data cube. 설명은 아래와 같고, [그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0312.png)을 보면 바로 대략적인 이해는 바로 될 것.

> It is a grid of aggregates grouped by different dimensions.

