# Redis data types

https://redis.io/topics/data-types

## String

### 개념

- 레디스 값 중에 가장 기본 종류.
- 레디스 문자열은 [binary-safe](https://en.wikipedia.org/wiki/Binary-safe).
- 어떤 종류의 데이터도 레디스 문자열에 담을 수 있음을 의미함.
- 예컨대, JPEG 이미지나 직렬화된 Ruby 객체 등.
- 최대 길이는 512mb.

### 활용

- 문자열을 원자적 카운터로 사용할 수 있음([INCR](https://redis.io/commands/incr), [DECR](https://redis.io/commands/decr), [INCRBY](https://redis.io/commands/incrby) 커맨드를 이용).
- [APPEND](https://redis.io/commands/append) 명령어를 이용해 문자열 덧붙이기 가능.
- [GETRANGE](https://redis.io/commands/getrange), [SETRANGE](https://redis.io/commands/setrange)를 이용해 [random access](https://en.wikipedia.org/wiki/Random_access) vector로 사용도 가능.
- 많은 데이터를 작은 공간에 인코드하거나, [GETBIT](https://redis.io/commands/getbit)와 [SETBIT](https://redis.io/commands/setbit)를 이용해 블룸 필터<sup>Bloom Filter</sup>를 만들 수도.
- [HyperLogLogs](https://redislabs.com/redis-best-practices/counting/hyperloglog/)도 같이 참고.

## Lists

### 개념

- 레디스 리스트는 단지 문자열의 목록.
- 삽입된 순서로 정렬되어 있음.
- 새로운 원소를 head나 tail로 넣는 것도 가능.
- [LPUSH](https://redis.io/commands/lpush)와 [RPUSH](https://redis.io/commands/rpush) 참고.
- 이 두 연산이 수행될 때, 주어진 키가 존재하지 않으면, 새로운 리스트를 생성함.
- 반대로, 위 연산이 수행된 결과가 빈 리스트라면, 키 스페이스로부터 키 삭제됨.
- 키가 존재하는지 여부를 따로 판단하지 않아도 되므로 편리함.
- 최대 길이는 2^32 - 1.
- head와 tail 근처의 원소를 상수 시간에 삽입하고 삭제함. 수백만 개의 아이템이 삽입된 경우라고 하더라도 말이다.
- 목록의 양 끝쪽에 접근하는 것은 매우 빠른 한편, 매우 큰 목록의 중간에 접근하는 것은 느림. O(N) 연산이기 때문.

### 활용

- 소셜 네트워크 타임라인 모델링. [LPUSH](https://redis.io/commands/lpush)와 [RPUSH](https://redis.io/commands/rpush)로 사용자 타임라인에 새로운 엘리먼트를 추가하고, [LRANGE](https://redis.io/commands/lrange)를 통해 최근 삽ㅇ비된 아이템들을 불러옴.
- [LPUSH](https://redis.io/commands/lpush)와 [RPUSH](https://redis.io/commands/rpush)와 [LTRIM](https://redis.io/commands/ltrim)을 이용해서, 주어진 길이를 초과하지 않는 리스트 생성 가능. 다만, 최근 N개의 엘리먼트라는 것에 유의. 참고로, `LTRIM`은 O(1).
- 메시지 전달 수단으로도 활용. 백그라운드 잡을 생성하기 위한 루비 라이브러리 [Resque](https://github.com/resque/resque) 참고.
- [BLPOP](https://redis.io/commands/blpop)을 이용한 블럭킹 동작을 활용할 수도.

## Sets

### 개념

- 정렬되지 않은 문자열 컬렉션.
- O(1) 시간에 추가, 삭제, 키 존재 여부 확인(원소 갯수 상관 없음).
- 당연한 얘기지만 중복 허용 X.
- 삽입 시 중복 체크도 필요 없음.
- union(합집합), intersection(교집합), difference(차집합) 연산 빠른 시간 내에 수행.
- 최대 길이는 2^32 - 1.

### 활용

- 사이트 고유 방문자 수 등의 고유 여부 추적. [SADD](https://redis.io/commands/sadd) 활용.
- 관계를 표현하는 데에도 유용. 예컨대 태깅. 각 블로그 글의 아이디를 키로 하여 해당 태그들을 셋에 넣어둠. [SINTER](https://redis.io/commands/sinter)를 이용해서, 임의의 3가지 태그를 가진 글들만을 추려낼 수 있음.
- [SPOP](https://redis.io/commands/spop)과 [SRANDMEMBER](https://redis.io/commands/srandmember)를 이용해서 무작위로 원소 추출 가능.

## Hashes

### 설명

- 문자열 필드와 문자열 값들 간의 맵<sup>map</sup>
- 따라서, 객체를 표현하기에 적합한 데이터 타입.

```
HMSET user:1000 username antirez password P1pp0 age 34
HGETALL user:1000
HSET user:1000 password 12345
HGETALL user:1000
```

- 얼마 안 되는 필드를 가진 해시는 매우 적은 공간을 차지.
- 여기서의 '얼마 안 되는'은 100개 정도까지를 나타냄.
- 작은 레디스 인스턴스에 수백만 객체를 저장할 수 있는 정도.
- 모든 해시는 2^32 - 1 쌍의 필드-값 짝을 가질 수 있음.

### 활용

- 기본적으로 객체를 표현하는 데 사용.
- 여러 요소들을 담을 수 있는 데이터 타입이므로 다른 여러 방식으로도 활용 가능.

## Sorted sets

### 개념

- 중복을 허용하지 않는 점에서 레디스 셋과 유사.
- 하지만, Sorted Set 모든 멤버는 점수가 매겨져 있음.
- 이 점수는 정렬을 위해 사용됨. 작은 값에서 큰 값 순으로.
- 멤버는 고유하지만, 점수는 반복될 수 있음.
- 원소의 추가, 삭제, 수정이 매우 빠르게 수행됨.
- 원소 수의 로그에 비례하는 시간이라고 함(왜 O(N)이라고 안 하고 이렇게 길게 표현했을까)
- 점수나 랭킹(포지션)으로 범위 내의 원소를 조회하는 것도 빠름.
- 그 이유를 아래와 같이 설명하고 있는데 처음엔 이해하기 어려웠음.

> Since elements are taken in order and not ordered afterwards, ...

- 관련 내용을 좀 더 찾아봤고 [이 글](https://scalegrid.io/blog/introduction-to-redis-data-structures-sorted-sets/)이 위 문장을 설명해 주는 듯.
    - Sorted set은 내부적으로 이중 데이터 구조체.
    - 하나는 hash, 다른 하나는 skip list.
    - hash는 객체들을 점수에 대응.
    - skip list는 점수를 객체에 대응.
- [여기](http://redisgate.kr/redis/configuration/internal_skiplist.php) 글에서는 zip list에 대한 설명도 나와 있음. skip list에 대한 친절한 그림도 포함.
- 중간 원소에 접근하는 것도 매우 빠름.

### 활용

- 대규모 온라인 게임의 리더보드. 새로운 점수가 제출될 때마다 [ZADD](https://redis.io/commands/zadd)를 이용해서 업데이트. [ZRANGE](https://redis.io/commands/zrange)를 통해 상위 사용자들을 추출할 수도 있음. [ZRANK](https://redis.io/commands/zrank)를 이용하면 사용자 이름을 키로 하여 랭킹을 알 수도. ZRANK와 ZRANGE를 함께 이용하면 비슷한 점수 대의 사용자들을 매우 빠르게 가져올 수도. 
- 레디스에 저장된 데이터에 대한 인덱스로도 많이 사용. 예를 들어, 사용자를 표현하는 여러 해시를 가지고 있다면, 사용자의 나이를 키로 하고 사용자의 ID를 값으로 하는 sorted set을 관리할 수도. 그리고 [ZRANGEBYSCORE](https://redis.io/commands/zrangebyscore)를 통해 주어진 범위의 사용자들을 조회.

## Bitmaps and HyperLogLogs

- 이런 게 있다고만 언급.
- 좀 더 자세한 내용은 [An introduction to Redis data types and abstractions](https://redis.io/topics/data-types-intro) 참고.
