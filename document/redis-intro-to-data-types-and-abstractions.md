
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

