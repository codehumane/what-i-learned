
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
