# Redis data types

https://redis.io/topics/data-types

## String

- 레디스 값 중에 가장 기본 종류.
- 레디스 문자열은 [binary-safe](https://en.wikipedia.org/wiki/Binary-safe).
- 어떤 종류의 데이터도 레디스 문자열에 담을 수 있음을 의미함.
- 예컨대, JPEG 이미지나 직렬화된 Ruby 객체 등.
- 최대 길이는 512mb.
- 문자열을 원자적 카운터로 사용할 수 있음([INCR](https://redis.io/commands/incr), [DECR](https://redis.io/commands/decr), [INCRBY](https://redis.io/commands/incrby) 커맨드를 이용).
- [APPEND](https://redis.io/commands/append) 명령어를 이용해 문자열 덧붙이기 가능.
- [GETRANGE](https://redis.io/commands/getrange), [SETRANGE](https://redis.io/commands/setrange)를 이용해 [random access](https://en.wikipedia.org/wiki/Random_access) vector로 사용도 가능.
- 많은 데이터를 작은 공간에 인코드하거나, [GETBIT](https://redis.io/commands/getbit)와 [SETBIT](https://redis.io/commands/setbit)를 이용해 블룸 필터<sup>Bloom Filter</sup>를 만들 수도.
- [HyperLogLogs](https://redislabs.com/redis-best-practices/counting/hyperloglog/)도 같이 참고.

## Lists

TBD