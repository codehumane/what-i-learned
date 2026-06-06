
# [Shopify, We replaced Redis with MySQL for inventory reservations—and it scaled](https://shopify.engineering/scaling-inventory-reservations)

- 구매자가 "구매 완료" 버튼 누르면, 상품의 재고를 확인해야 함
- 2명이 1개 남은 상품 동시 구매, 재고가 있는데 품절 처리 모두 문제
- Shopify는 2025년 블랙 프라이데이에 분당 510만 달러 매출 기록
- 규모 있는 기업에서는 실패 여파가 빠르게 커짐
- 그래서 과잉 판매 방지 시스템 둠
- 결제 처리 중 재고를 일시 보류하는 방식
- 수년간 Redis를 사용해 왔음
- 그러나 일관성 문제가 있었음
- 그래서 통합 DB 전략으로 전환
- 여기서 어려운 점은 DB 설계가 아니었음
- 진짜 병목은 우리가 관찰하고 측정했던 것과 다른 곳에 있었음
- 이 글은 통합 DB 해결책, 그리고 해결 과정에서 얻은 교훈을 다룸

## The challenge

### 기능

과잉 판매 금지를 위해 아래 2개가 주요 기능.

- 예약<sup>reserve</sup> — 결제가 시작되면, 해당 상품을 예약 상태로 표시(몇 분 정도 유지)
- 청구<sup>claim</sup> — 결제가 완료되면, 재고 원장(source of truth)에서 해당 수량 영구 차감

### 비기능

- 정확성 최우선
  - 예약과 재고 간 일관성
  - oversell — 고객 불만
  - undersell — 손실
- 속도 (구매자 경험에 영향)
- 높은 처리량 (Shopify는 미국 전자상거래의 14% 이상을 점유)

### 레디스의 한계

- 기존 재고는 상품 별로 수량을 MySQL에 저장
- 그리고 재고 예약과 취소에 Redis `INCR`, `DECR` 사용
- 동시성 처리는 좋음
- 그러나 근본적으로 데이터 일관성 문제
- oversell 혹은 undersell 가능
- 클러스터 유지 비용도 추가

## The solution

### 이전 시도

- MySQL만으로 구현 시도
- 수량 열이 있는 단일 행으로는 경합 처리 X

### 해결 아이디어: SKIP LOCKED

- 그래서 MySQL 8의 SKIP LOCKED 활용
- 품목당 한 행 X 재고 단위당 한 행 O
- 10개 단위 예약이면 10개 행을 가짐
- [37signals, DB 기반 부하 분산 방식](https://dev.37signals.com/introducing-solid-queue/)에서 영감
- 예약과 재고를 동일한 DB에 유지하므로 일관성 확보

```sql
-- Start transaction with READ COMMITTED isolation
BEGIN;

-- Select N units, skipping any locked by other transactions
SELECT id
FROM available_units
WHERE shop_id = ?
  AND inventory_item_id = ?
  AND inventory_group_id = ?
ORDER BY shop_id ASC, inventory_item_id ASC, inventory_group_id ASC, id ASC
LIMIT 3
FOR UPDATE SKIP LOCKED;

-- Move selected units to reserved_units table
INSERT INTO reserved_units (unit_id, cart_token, expires_at, ...)

-- Remove from available pool
DELETE FROM available_units
WHERE (shop_id, inventory_item_id, inventory_group_id, id) IN (?, ?, ?, ?);

COMMIT;
```

### 규모 스케일링 문제 보완

- 하지만, 재고 규모가 크면 문제
- 예컨대, 10개 지역 각각의 5만개 재고에는 50만개 행이 필요
- 예약 조회 시 이 행들을 모두 스캔해야 하므로 속도가 느려짐
  - [MySQL 문서 내용](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html) 함께 참고
  - 아마 중복된 값이 여러 행에 걸쳐 있기 때문인 듯
  - 여러 행에 락이 걸려 있으면 일일이 스캔하고 건너 뛰는 비용으로 추정됨
- 그래서 품목/지역 별로 최대 1,000개 행으로 제한된 풀을 유지
- 예약은 이 풀에서 행을 차감 & 재고 보충 프로세스가 재고 원장에서 행을 가져와 풀을 다시 채움
- 그런데 왜 1,000개인가?
- 수요 급증에도 재고 부족 일어나지 않을 만큼 충분히 큼
- 동시에 성능 저하도 초래하지 않는 적절한 수준
- 그래도 만약, 재고 풀이 고갈되면?
- 락 기능을 통해 한 번에 하나의 거래만 재고 보충 진행
- 락의 구체적 내용은 없어서 아쉬움

## Key technical decisions

### 1. 복합키: 행 락 감소

- 처음엔 기본키로 자동 증가 ID 사용
- 그리고 락 행위를 관찰해 봄(`SHOW ENGINE INNODB STATUS` 등)
- 그랬더니 예약 별로 1개가 아닌 2개의 행 락이 발생
- InnoDB에서 자동 증가 기본키를 사용하면, 2개의 락 발생 가능
  - WHERE 절에 있는 보조 인덱스
  - 클러스터형 인덱스(기본키)
- 그래서 복합 기본키로 변경
  - `shop_id, inventory_item_id, inventory_group_id, id`
- 이제 필터링에 사용되는 열들이 기본 키 일부가 됨
- 락은 1번으로 줄어듦
- 결국, 많은 예약을 동시에 수행할 수 있게 됨
- 요약: 이 정도 규모에서는, 인덱스와 기본키 설계가 락 수와 처리량에 직접적 영향

*글 읽으면서 함께 참고했던 MySQL 문서들

- [Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html)
- [InnoDB Locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)
- [Locking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)

### 2. READ COMMITED: 갭(상한) 락 피하기

- 데이터 보충을 위해 빈 테이블에 대해 `SELECT ... FOR UPDATE SKIP LOCKED` 실행할 때 있음
- 이 경우 갭 락<sup>gap locks</sup>(최상위 레코드를 포함) 발생
- 이는 데이터 보충 트랜잭션의 새로운 데이터 삽입을 막고 데드락 가능성도 가짐
- 그래서 트랜잭션 격리 레벨 변경: `REPEATABLE READ` → `READ COMMITED`
- InnoDB에서 `READ COMMITED`는 갭 락이 없음
- [A Comprehensive (and Animated) Guide to InnoDB Locking](https://jahfer.com/posts/innodb-locks/) 글이 도움 되었다고 함
- 해당 코드베이스에서는 기본 격리 레벨이 아닌 걸 쓰는 게 처음이었다고 함
- 트랜잭션 별로 격리 레벨을 설정하려면 프레임웍 지원이 필요

*함께 보면 좋을 MySQL 문서

- [Locking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)
- [InnoDB Locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)
- [A Comprehensive (and Animated) Guide to InnoDB Locking](https://jahfer.com/posts/innodb-locks/)

### 3. 일관된 락 순서: 데드락 피하기

- 예약과 청구가 2개의 테이블을 서로 다른 순서로 접근할 때 데드락 발생
- 예약은 `INSERT into reserved_quantities` 이후에 `DELETE from available_units` 수행
- 이 때 청구는 `DELETE from reserved_quantities` 수행 중
- 서로 다른 트랜잭션은 2개의 테이블을 각자 다른 순서로 잠그고 싸이클을 형성
- 해결은 순서 표준화
- 예약: 재고 테이블 DELETE → 예약 테이블 INSERT
- 청구: 예약 테이블만 접근
- 이제 동일한 순서로 락을 획득
- 더 이상의 순환 대기는 없어짐
- 사실, 위에 나온 설명만으로는, 대기는 있었을지언정 데드락이 발생했다고 판단하기는 어려움

*관련 MySQL 문서

- [Deadlocks in InnoDB](https://dev.mysql.com/doc/refman/9.7/en/innodb-deadlocks.html)
- [How to Minimize and Handle Deadlocks](https://dev.mysql.com/doc/refman/9.7/en/innodb-deadlocks-handling.html)

### 4. UNION ALL을 이용한 일괄 처리

- DB로의 왕복 통신에는 비용이 따름
- 장바구니의 여러 항목들을 한 번에 질의하면 도움
- 이 때 `UNION ALL`을 사용
- 응답 지연에도 효과

```sql
(SELECT id, inventory_item_id, inventory_group_id
 FROM available_units
 WHERE shop_id = 1 AND inventory_item_id = 100 AND inventory_group_id = 1
 ORDER BY shop_id, inventory_item_id, inventory_group_id, id
 LIMIT 2 FOR UPDATE SKIP LOCKED)
UNION ALL
(SELECT id, inventory_item_id, inventory_group_id
 FROM available_units
 WHERE shop_id = 1 AND inventory_item_id = 200 AND inventory_group_id = 1
 ORDER BY shop_id, inventory_item_id, inventory_group_id, id
 LIMIT 5 FOR UPDATE SKIP LOCKED)
```

## The real bottleneck: connections, not CPU

- 프로덕션에서 처리량이 목표치에 한참 미치지 못함
- 하지만, 예약 지연(p90 등)은 괜찮음, CPU도 최대치는 아님, 쿼리는 이미 최적화 됨
- 여러 체크아웃에 대해 일괄 처리 쿼리도 시도해 봄
- 부하 테스트는 괜찮았지만 복잡성 증가
- 또한 읽기 일부를 복제본으로 옮김
- 그래도 문제는 남아있었음

### 증상 따라가기

- 부하테스트에서 아래 3가지 발견
- ① MySQL 내 스레드 큐잉
- ② 큐에 담긴 작업이 실행될 때 CPU 스파이크
- ③ ProxySQL 레이어에서 MySQL 벡엔드에 대한 커넥션 고갈
- 그래서 어떤 비즈니스 프로세스가 DB 커넥션을 얼마나 오래 점유하는지 확인하기로 함
- 이를 위해 애플리케이션에서 모든 SQL에, 비즈니스 프로세스를 식별하는 주석 태그 추가
- `/* conn_tag:checkout_completion */`
- ProxySQL에서는 태그 파싱하고 호출자가 커넥션을 얼마나 오래 잡는지 측정
- 개인적으로는 이 방법이 다른 대안 대비 좋았을지는 의문

### 발견한 것

- 예약 외에도 결제 과정의 다른 곳에서 커넥션을 필요 이상으로 점유 중
- 커넥션은 유한하므로, 높은 처리량 위해서는, 짧은 트랜잭션이 필요
- 예약은 느리지 않았음
- 다른 기능에 의한 커넥션 부족이 예약에 영향 준 것
- 정리 작업으로, DB 읽기 작업은 50%, 트랜잭션은 33% 감소
- 또한 오래된 MySQL 구성도 재검토
- InnoDB 스레드 동시 실행 수가 적었음
- 과거의 보수적 설정이 계속 사용되고 있던 것
- 그래서 스레드 동시 실행 수를 늘려 병목 해결

### 전환

- 단순하게 Redis에서 MySQL로 전환한 게 아님
- "쉐도우 모드"라 불리는 방식으로, 두 시스템을 병렬로 실행함
- 모든 예약은 Redis와 MySQL 양쪽에 쓰임
- Redis가 여전히 source of truth
- 두 시스템 비교로 MySQL이 제대로 비즈니스 결과를 내는지 검증
- 또한 프로덕션 트래픽 속에서 성능 요구사항 만족되는지도 확인
- 두 시스템을 모두 운영했으므로 진행중인 예약의 마이그레이션도 필요 없음
- 정확성과 성능 검증 후 source of truth를 MySQL로 전환
- Redis는 여전히 동작 중이므로, 문제가 있으면 언제든 Redis로 다시 돌릴 수 있음
- 배포는 트래픽 적은 Pod부터 시작해 단계적으로 적용

## What we learned

- 이 프로젝트 통해 많은 걸 배움
- 그중 가장 중요한 것 2가지 이야기로 마무리

### 1. 과거 결정 재검토

- 5년 전엔 불가능했던 일이, `SKIP LOCKED` 같은 새로운 기능으로 가능해짐
- 스레드 제한처럼, 과거에 경험적으로 정해진<sup>rule of thumb</sup> 설정도, 운영 트래픽과 하드웨어의 변화에 맞춰 다시 점검해야 함
- 수치가 이상하다면(CPU 사용률 낮은데 큐잉 발생) 자세히 살펴볼 것

### 2. 작게 시작해서 관찰

- 간단한 프로토타입(작은 루비 스크립트와 MySQL)으로 많은 가치를 얻음
- DB 관찰(두 번째 터미널에서의 락 동작 확인)은 이론으로는 얻을 수 없는 것들을 알려줌
- 탐색 과정에서는 간단한 도구와 긴밀한 피드백 루프가 효과적
- MySQL은 이제 더 높은 처리량도 다룰 수 있음
- Redis, Kafka 필요 없을 수도
- 그리고 병목은 예상과 다른 곳에 있었음
- 몇 주를 쿼리와 락 최적화에 매달렸으나, 실제로 병목은 커넥션 사용에 있었음
- 해답은 엔진이 아니라 내부 구조에 있는 경우가 많음
