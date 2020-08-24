
# An introduction to Redis data types and abstractions

레디스는 *plain key-value store*가 아니라, 여러 종류의 값들을 지원하는 *data structures server*. 전통적인 키-값 저장소에서는 문자열 키를 문자열 값들에 대응 시켰음. 레디스에서는 여기에 국한되지 않고, 더 복잡한 데이터 구조체를 유지할 수 있음. 아래는 레디스가 지원하는 데이터 구조체들.

- Binary-safe strings.
- Lists
    - 문자열 원소들의 집합.
    - 삽입된 순서대로 정렬.
    - 기본적으로는 링크드 리스트.
- Sets
    - 고유한, 정렬되지 않은 문자열 원소들의 집합.
- Sorted Sets
    - 기본적으로는 Set과 유사하지만,
    - 모든 문자열 원소들이 부동소수점<sup>floating</sup> 숫자 값에 대응.
    - 이 숫자는 스코어<sup>score</sup>라고 불림.
    - 모든 원소들은 항상 자신의 스코어 대로 정렬되어 있음.
    - 최상위 10개와 같은 특정 범위의 원소들을 조회할 수 있음.
- Hashes
    - 값들에 대응된 필드로 구성된 맵.
    - 필드와 값 모두 문자열.
    - Ruby나 Python의 해시와 매우 비슷.
- Bit arrays(또는 단순히 bitmaps)
    - 특수 커맨드들을 이용하면, 문자열 값을 비트의 배열로 다룰 수 있음.
    - 각 비트에 값을 할당하고 제거.
    - 1이 할당된 비트 갯수를 세거나, 1이 설정/해제된 가장 앞 쪽의 비트를 찾는 등의 일들이 가능.
- HyperLogLogs
    - 집합의 카디널리티를 추정하기 위한 확률적 데이터 구조체.
- Streams
    - 맵 같은 엔트리의 추가만 가능한<sup>append-only</sup> 집합.
    - 로그 데이터 타입의 추상화를 제공.

## Redis keys

레디스 키는 binary safe. 어떤 바이너리 시퀀스도 키로 사용할 수 있음을 의미. "foo" 같은 단순한 문자열부터 JPEG 파일 컨텐츠까지 다양. 빈 문자열 또한 유효한 키. 아래는 키에 대한 몇 가지 규칙.

- 매우 긴 키는 권장 X. 예컨대, 1024 바이트의 키 사용은 메모리 사용 측면에서 비효율. 또한, 데이터 셋 안에서의 키의 검색<sup>lookup</sup>에 값 비싼 키 비교 비용이 들어감. 단지 큰 값이 존재여부를 매칭하는 일이라고 하더라도, 메모리와 대역폭 관점에서, 이를 해싱해서 사용하는 것이 좋음.
- 매우 짧은 키 역시 나쁜 선택일 수도. 예를 들어, `user:1000:followers` 대신 `u1000flw`를 키로 사용하는 것은 별로 의미가 없음. 전자가 더 가독성이 좋고, 약간의 공간 사용 증가는 마이너한 요소. 더 짧은 키가 더 적은 비트를 사용하겠지만, 균형을 잡는 것이 우리의 몫.
- 스키마를 지키자. `object-type:id`와 같은 형태를 사용. 그리고 마침표나 대시를 사용해서 여러 단어로 이뤄진 필드에 사용. `comment:1234:reply.to` 또는 `comment:1234:reply-to`와 같이.
- 키 사이즈의 최대치는 512 MB.

## Redis Strings

- 레디스 키에 대응해서 사용하는 가장 단순한 타입.
- Memcahced에서의 유일한 데이터 타입.
- 레디스 키가 문자열이므로, 값으로도 문자열을 사용하면, 문자열을 다른 문자열에 매핑하는 것.
- 문자열 타입은 HTML 프래그먼트나 페이지 캐싱 등의 여러 경우에 유용.

```
> set mykey somevalue
OK
> get mykey
"somevalue"
```

- `SET`과 `GET` 커맨드를 이용해서 문자열 값을 설정하고 불러옴.
- 이미 있는 키에 대한 `SET`이면 값을 덮어 씀.
- 기존 값이 문자열이 아닌 값이라고 하더라도 대체 됨에 유의.
- 어떤 종류의 문자열(바이너리 데이터 포함)이든 값으로 저장할 수 있음.
- 예컨대 JPEG 이미지도 저장 가능.
- 값은 512 MB를 넘을 수 없음.
- `SET` 옵션 중에 `NX`는 키가 없는 경우에만,
- `XX`는 값이 이미 있는 경우에만 값을 설정함을 가리킴.

```
> set mykey newval nx
(nil)
> set mykey newval xx
OK
```

- 문자열에 대해 `INCR`, `INCRBY`, `DECR`, `DECRBY`를 사용하기도.
- 이들 커맨드는 모두 원자적<sup>atomic</sup>.

```
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

- `GETSET` 커맨드는 새로운 값을 할당하면서 기존의 값을 반환.
- 이 커맨드의 사용 예시는 다음과 같음. 웹 사이트에 매번 새로운 방문자가 들어올 때마다 `INCR` 명령어로 카운팅을 하고, 한 시간에 한 번씩 `GETSET`을 통해 갯수를 수집하고 동시에 값을 초기화.
- `MSET`, `MGET`은 여러 값을 한 번에 설정하고 조회.

```
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

## Altering and querying the key space

- 어떤 키의 타입에 대해서도 쓰일 수 있는,
- 키 스페이스와 상호작용하기 위한 커맨드들이 있음.
- 특정 타입에 대해 정의된 것은 아님.
- 예를 들어 [EXISTS](https://redis.io/commands/exists) 커맨드는 키의 존재 여부를 1과 0으로 알려줌.
- 한편 [DEL](https://redis.io/commands/del) 커맨드는 값(어떤 값이든 상관 없음)과 함께 키를 지워줌.

```
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
```

- 위 예시에서 `DEL`이 키 존재 여부에 따라 1과 0을 반환함을 확인할 수 있음.
- [TYPE](https://redis.io/commands/type) 커맨드는 주어진 키의 값이 어떤 타입인지를 반환.

```
> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```

## Redis expires: keys with limited time to live

- Redis expires.
- 이 또한 값의 타입에 상관 없는 기능.
- 키에 대해 타임아웃(살아 있는 시간을 제한)을 지정하는 것.
- 약속된 시간이 지나면 키는 자동으로 없어짐.
- `DEL` 커맨드를 호출한 것과 동일한 결과.
- 초나 밀리세컨드 단위로 만료 시간 지정 가능.
- 하지만 만료 시간 체크는 1 밀리세컨드 단위로 이루어짐.
- 만료 정보는 디스크에 복제되고 영속됨.

```
> set key some-value
OK
> expire key 5
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
```

- 위 예시에서 두 번의 `GET` 사이에 키는 사라짐.
- 이를 위해 `EXPIRE` 커맨드 사용.
- 만료 시간 업데이트를 위해, 만료 시간이 이미 지정된 키에 대해서도 다시 `EXPIRE` 사용 가능.
- `PERSIST`는 만료 시간 지정을 무효화.
- `SET`의 `ex` 옵션처럼, 다른 커맨드에서도 타임아웃 지정 가능.

```
> set key 100 ex 10
OK
> ttl key
(integer) 9
```

- 위 예시에서 `TTL` 커맨드는 남아 있는 TTL 시간을 확인.
- 밀리세컨드 단위로 만료를 지정하고 확인하고 싶다면, `PEXPIRE`와 `PTTL` 커맨드 사용.

## Redis Lists

- 매우 일반적인 관점에서 보자면, 리스트는 단지 정렬된 요소들의 시퀀스.
- 하지만 구현체가 연결 리스트냐 배열이냐에 따라 많은 속성들이 달라짐.
- 레디스는 연결 리스트 사용.
- 수백만 개의 원소가 리스트에 들어 있더라도, 앞뒤로의 원소 추가는 상수 시간 내에 끝남을 의미.
- 한편, 특정 인덱스의 원소 접근은 배열 만큼 빠르지 않음을 의미.
- 레디스가 연결 리스트를 선택한 이유는 데이터베이스 시스템에 있어 매우 긴 리스트에도 빠른 원소의 추가가 중요하기 때문이라고 함.
- 또 다른 주요 이점도 얘기하는 데 이해 안 됨. 일단 원문 그대로 가져옴.

> Another strong advantage, as you'll see in a moment, is that Redis Lists can be taken at constant length in constant time.

- 만약, 규모가 큰 컬렉션에서 중간 원소의 접근이 빨라야 한다면, sorted set 구조체 사용을 권장.

## First steps with Redis Lists

- `LPUSH`는 왼쪽(head)에, `RPUSH`는 오른쪽(tail)에 원소 삽입하는 커맨드.
- 두 커맨드 모두 *variadic*. 삽입할 원소를 한 번에 여러 개 지정할 수 있음을 의미.
- `LRANGE`는 목록에서 특정 범위 내의 원소들 추출.
- `LRANGE`의 두 인자는 모두 음수일 수 있음. -1은 마지막 원소. -2는 끝에서 두 번째.
- 레디스 리스트에서 중요한 연산 하나는 `pop`.
- 이는 리스트에서 원소를 제거함과 동시에 조회하는 것.

## Common use cases for lists

- 아래의 2가지가 리스트의 대표적 사용 사례.
- 소셜 네트워크에서 사용자의 최근 업데이트를 기억.
- 컨슈머-프로듀서 패턴을 이용한, 프로세스들 간의 커뮤니케이션.
- Ruby 라이브러리인 [resque](https://github.com/resque/resque)와 [sidekiq](https://github.com/mperham/sidekiq) 참고.
- 트위터의 [takes the latest tweets] 영상도 참고.

## Capped lists

- 리스트를 사용해서 최신 아이템들을 저장하는 경우가 많음.
- 소셜 네트워크 업데이트나 로그 등.
- 레디스를 이런 capped 컬렉션을 지원.
- 즉, 최근 N개의 아이템만을 기억하고, 오래된 나머지는 버리는 것.
- [LTRIM](https://redis.io/commands/ltrim) 커맨드를 이용.
- `LRANGE`와 유사하지만, 단지 범위 내의 원소들을 보여주는 것이 아니라, 이 범위의 원소들을 새로운 리스트 값으로 설정. 범위 밖의 원소들은 버려짐.

```
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

주로 아래의 명령어 짝으로 사용.

```
LPUSH mylist <some element>
LTRIM mylist 0 999
```

## Blocking operations on lists

- `LPUSH`와 `RPOP`을 통해 프로듀서-컨슈머를 위한 큐를 만들 수 있음.
- 하지만 종종 큐가 비어 있을 수 있음.
- 이 때 `RPOP`은 NULL 반환.
- 그러면 컨슈머는 일정 시간 대기한 뒤 `RPOP`을 재시도.
- 이는 폴링 방식.
- 2가지 면에서 안 좋음.
    - 큐가 비어 있는 동안 레디스와 클라이언트가 불필요한 커맨드를 반복적으로 처리.
    - 지연이 더 길어짐. NULL이 반환되면 어느 정도 시간을 두고 대기해야 하기 때문.
    - 지연을 줄이려면 불필요 커맨드가 늘어나고, 불필요 커맨드를 줄이려면 대기가 늘어남.
- 그래서 `BRPOP`과 `BLPOP`을 제공.
- `RPOP`과 `LPOP`과 기본적으로 동일.
- 하지만 리스트가 비어 있는 경우 블럭킹.
- 혹은 주어진 타임아웃 만큼 블럭킹. (타임아웃이 0이면 무제한)
- 그리고 명령어 인자로 여러 리스트를 지정할 수 있음.
- 이렇게 하면 주어진 리스트 목록 중 비어 있지 않은 가장 첫 번째 리스트의 원소 반환. [BLPOP non-blocking behavior](https://redis.io/commands/blpop#non-blocking-behavior) 참고.
- 만약 모든 리스트가 비어 있다면 대기하다가 리스트 중 하나라도 새로운 원소가 생기면 바로 반환. [BLPOP blocking behavior](https://redis.io/commands/blpop#blocking-behavior) 참고.
- [RPOPLPUSH](https://redis.io/commands/rpoplpush)라는 커맨드도 있음.
- 블럭킹 버전인 [BRPOPLPUSH](https://redis.io/commands/brpoplpush)도.

## Automatic creation and removal of keys

- 기본적으로는 빈 리스트를 생성하거나 제거할 필요가 없음.
- 원소 제거 시 빈 목록이면 자동으로 지워지거나, 원소 추가 시 리스트가 없으면 자동으로 생성되기 때문.
- 이는 리스트에만 국한된 특징이 아니라, 여러 원소로 이루어지는 모든 레디스 데이터 타입에 대해서도 마찬가지.
- Stream, Set, Sorted Set, Hash 말이다.

```
// 존재하지 않는 키에 대한 삽입 연산
> del mylist
(integer) 1
> lpush mylist 1 2 3
(integer) 3

// 이미 존재하는 키에 대한 연산들
> exists mylist
(integer) 1
> lpop mylist
"3"
> lpop mylist
"2"
> lpop mylist
"1"

// 아래는 존재하지 않는 키에 대한 연산들
> exists mylist
(integer) 0
> del mylist
(integer) 0
> llen mylist
(integer) 0
> lpop mylist
(nil)
```

## Redis Hashes

- 필드와 값의 짝으로 구성된, "hash"라는 말 그대로의 모습.

```
> hmset user:1000 username antirez birthyear 1977 verified 1
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

- 해시가 객체 표현에 유용하긴 하지만,
- 해시에 넣을 수 있는 필드의 수에는 특별한 제한이 없음.
- 따라서, 애플리케이션에서 해시를 다양한 방식으로 사용할 수 있음.
- `HSET`, `HGET`, `HMGET` 등의 커맨드 사용.

```
> hmget user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```

- `HINCRBY`와 같은 개별 필드에 대한 커맨드도 있음.

```
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```

- 작은 해시(몇 안 되는 갯수의 원소와 작은 값들)는 특수한 방식으로 인코딩 되므로 메모리 효율에 유리.

## Redis Sets

- 순서 없는 문자열 집합.
- `SADD` 커맨드로 새로운 원소 추가.
- 그 외에 값이 존재하지는지 여부 확인이나, 여러 셋에 대한 교집합, 합집합, 차집합 등의 연산도 가능.

```
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 3
2. 1
3. 2
```

- 아래와 같이 멤버십을 확인하는 커맨드도 존재.

```
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```

- 셋은 객체들 간의 관계를 표현하기에 적합.
- 예를 들어, 태그 기능 구현을 위해 셋을 사용할 수도.
- 아래 예시는 ID가 1000인 뉴스에 1, 2, 5, 77 태그가 달려 있음을 표현.

```
> sadd news:1000:tags 1 2 5 77
(integer) 4
> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```

- 혹은 반대 방향으로 관계를 표현할 수도.

```
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1
```

- 참고로, 위 예시들에서는 각 ID에 대응하는 객체들이 해시에 저장되어 있으리라 가정.
- 다음은 태그 1, 2, 10, 27에 관련된 객체들을 모두 반환하는 예시.

```
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```

- 원소를 추출하는 `SPOP`도 유용한 경우가 있음.
- 예컨대, 웹 기반의 포커 게임에서, 셋으로 덱<sup>deck</sup>을 표현하고, `SPOP`을 통해 임의의 카드를 추출. 아래 예시에서 각 문자는 카드의 앞 글자임. (C)lubs, (D)iamonds, (H)earts, (S)pades.

```
>  sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
   (integer) 52
```

- 하지만, `deck`에 대해 직접 추출을 하면, 다음 게임에 카드의 덱을 다시 만들어야 함.
- 따라서, 게임을 시작할 때 `deck`을 복사하는 것이 좋음.
- 매 게임의 `deck` 키는 `game:1:deck` 같은 식으로 명명.
- 이 때 [SUNIONSTORE](https://redis.io/commands/sunionstore) 활용.
- 여러 셋에 대한 합집합을 구하는 연산이고, 그 결과를 다른 셋에 저장.
- 하지만, 여기서는 단일 셋이므로 단지 복사하는 용도.

```
> sunionstore game:1:deck deck
(integer) 52
```

- 이제 아래와 같은 식으로 카드 추출.

```
> spop game:1:deck
"C6"
> spop game:1:deck
"CQ"
> spop game:1:deck
"D1"
> spop game:1:deck
"CJ"
> spop game:1:deck
"SJ"
```

- [SCARD](https://redis.io/commands/scard)는 원소의 갯수를 반환.
- 셋 이론의 문맥에서 *cardinality of a set*이라는 용어가 사용되므로 `SCARD`라 명명.
- 52개에서 5개가 추출되었으므로 47을 반환.

```
> scard game:1:deck
(integer) 47
```

## Redis Sorted sets

- 정렬된 셋<sup>sorted set</sup>은 셋<sup>set</sup>과 해시<sup>hash</sup>를 섞어 놓은 것과 비슷.
- 셋과 유사하게 고유한, 반복되지 않은 문자열 원소로 구성.
- 하지만 정렬된 셋 안의 원소들은 모두 부동소수점 점수 값에 대응.
- 이 점수는 스코어<sup>score</sup>라고 불림.
- 모든 원소들이 어떤 값에 대응되어 있다는 점에서 해시랑 유사.
- 게다가, 정렬된 셋 안의 원소들은 모두 정렬되어 있는 상태를 유지.
- 정렬 규칙은 아래와 같음.
    - A와 B가 서로 다른 점수를 가지고, 만약 A.점수 > B.점수라면 A > B이다.
    - A와 B가 같은 점수를 가지고, A가 B에 대해 사전 상으로<sup>lexicographically</sup> 앞선다면 A > B이다.
    - A와 B는 절대 같을 수 없음. 정렬된 셋은 고유한 원소들을 가지기 때문.

```
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer) 1
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```

- [ZADD](https://redis.io/commands/zadd)는 `SADD`와 유사하지만 한 가지 인자를 더 받음.
- 바로 점수.
- `ZADD`는 *variadic*이므로 여러 스코어-값 짝을 지정할 수 있음.
- 구현 노트.
    - 정렬된 셋은 두 개의 데이터 구조체로 이뤄져 있음.
    - 하나는 스킵 리스트, 나머지 하나는 해시 테이블.
    - 따라서, 원소 추가에 O(log(N)) 소요됨.
    - 대신, 이미 정렬되어 있으므로 정렬된 목록 반환시에는 별다른 일을 하지 않음.

```
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```

- [ZRANGE](https://redis.io/commands/zrange) 대신 [ZREVRANGE](https://redis.io/commands/zrevrange)를 사용해서 반대 순으로도 반환할 수 있음.

```
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```

- `WITHSCORES` 인자로 점수를 같이 반환할 수도.

```
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```

## Operating on ranges

- 정렬된 셋은 범위에 대한 연산도 제공.
- 아래 예시는 [ZRANGEBYSCORE](https://redis.io/commands/zrangebyscore)를 이용해 출생년도가 1950년 이하인 대상만을 추출한 것.

```
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```

- 좀 더 정확히 말하면, negative infinity와 1950 사이 점수의 원소들을 반환한 것.
- [ZREMRANGEBYSCORE](https://redis.io/commands/zremrangebyscore)를 이용해 범위에 대한 제거도 가능.

```
> zremrangebyscore hackers 1940 1960
(integer) 4
```

- get-rank 연산도 매우 유용. 이는 원소가 몇 번째에 위치했는지를 질의.
- [ZREVRANK](https://redis.io/commands/zrevrank)도 함께 참고.

```
> zrank hackers "Anita Borg"
(integer) 4
```

## Lexicographical scores

- 레디스 2.8부터 도입된 기능.
- 점수가 같을 때 사전적 순서로 원소를 반환.
- 주요 명령어는 [ZRANGEBYLEX](https://redis.io/commands/zrangebylex), [ZREVRANGEBYLEX](https://redis.io/commands/zrevrangebylex), [ZREMRANGEBYLEX](https://redis.io/commands/zremrangebylex), [ZLEXCOUNT](https://redis.io/commands/zlexcount)가 있음.
- 일단 아래와 같이 점수를 0으로 해서 유명 해커 목록을 만듦.

```
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
  "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
  0 "Linus Torvalds" 0 "Alan Turing"
```

- 정렬된 셋의 정렬 규칙에 따라 이들은 사전적 순서로 정렬되어 있음.

```
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```

- `ZRANGEBYLEX`를 이용해서 사전적 순서로 범위 질의 가능.

```
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds"
```

- 범위는 (첫 번째 문자가 무엇이냐에 따라) inclusive 또는 exclusive일 수 있음.
- 그리고 +와 - 부호로 무한 범위 지정 가능.
- 이 기능이 중요한 이유는 정렬된 셋을 제너릭 인덱스로 사용할 수 있게 해주기 때문.
- 예를 들어, 원소들을 부호가 없는 128 비트 정수형 인자로 인덱스를 하고 싶다고 가정.
- 이를 위해 정렬된 셋에 동일한 스코어로 원소들을 추가.
- 단, 빅 엔디안 128 비트 숫자를 원소의 접두사로 사용해야 함.
- 빅 엔디안 방식이므로 사전적 순서로 정렬되는 것이 실질적으로 숫자 정렬과 같음.
- 그리고 128 비트 공간에 대해서 범위 질의. 그리고 나서 접두사를 버리고 실제 원소 값을 사용.

## Updating the score: leader boards

- 다음 주제로 넘어가기에 앞서 마지막 정리.
- 정렬된 셋의 점수는 언제든 업데이트 될 수 있음.
- 이미 존재하는 원소에 대해 `ZADD`를 호출하면 됨.
- 점수와 함께 위치도 변경됨.
- 시간 복잡도는 O(log(N)).
- 따라서, 많은 업데이트에도 괜찮음.
- 이런 특성으로 리더 보드를 만드는 데 유용.
- 페이스북 게임은 높은 점수 순으로 사용자들을 정렬된 상태로 유지하고, get-rank 연산을 통해 사용자의 순위를 알려줌.
