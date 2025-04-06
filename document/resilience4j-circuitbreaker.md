# CircuitBreaker

- 8년 전 이후로는 실무에서 잘 사용 안 했었는데, 얼마나 바뀌었는지 오랜만에 가볍게 다시 살펴봄.
- https://resilience4j.readme.io/docs/circuitbreaker

## Introduction

- 3가지 일반 상태와 3가지 특수 상태를 갖는, 유한 상태 머신으로 구현됨.
    - 일반 상태: CLOSED, OPEN, HALF_OPEN
    - 특수 상태: METRICS_ONLY, DISABLED, FORCED_OPEN

![](https://files.readme.io/39cdd54-state_machine.jpg)

- 슬라이딩 윈도우를 사용해 결과를 집계하고 저장.
- 슬라이딩 윈도우로는 카운트 기반과 타임 기반이 있음.
- 카운트 기반은 마지막 N개의 호출 결과를 집계.
- 타임 기반은 마지막 N초 동안의 호출 결과를 집계.

## Count-based sliding window

- N개의 측정값으로 구성된 원형 배열을 가짐.
- 윈도우 사이즈가 만약 10이라면, 원형 배열은 항상 10개의 측정값을 가짐.
- 새로운 호출 결과가 집계될 때 점진적으로 총집계가 업데이트 됨. 
- 가장 오래된 측정값이 evict 될 때, 총집계에서 제외되고, 버킷은 초기화(Subtract-on-Evict).
- 스냅샷은 윈도우 사이즈와 독립적이고 미리 집계되므로, 조회에 걸리는 시간은 상수 O(1).
- 저장 공간은 당연히 O(N)이 될 것.

## Time-based sliding window

- N개의 부분 집계(버킷)들로 이뤄진 원형 배열.
- 윈도우 사이즈가 10초라면, 원형 배열은 항상 10개의 부분 집계(버킷)를 가짐.
- 모든 버킷은 특정 초에 일어난 모든 호출 결과를 집계(부분 집계).
- 원형 배열의 헤드 버킷은 현재 초의 모든 호출 결과를 저장.
- 다른 부분 집계는 이전 초들의 호출 결과를 저장.
- 슬라이딩 윈도우는 호출 결과를 개별적으로 저장하지 않음.
- 대신, 점진적으로 부분 집계와 총집계를 업데이트함.
- 총집계는 새로운 호출 결과가 기록될 때 점진적 업데이트.
- 가장 오래된 버킷이 evict 되면 버킷의 부분 총집계는 총집계에서 제외되고 버킷은 초기화(Subtract-on-Evict).
- 스냅샷은 타임 윈도우 사이즈와 독립적이고 미리 집계되므로, 조회에 걸리는 시간은 상수 O(1).
- 호출 결과가 개별적으로 저장되는 게 아니므로 공간 복잡도는 O(N)에 가까움.
- N개의 부분 집계와 1개의 총집계가 생성될 뿐임.
- 부분 집계는 실패 호출 갯수를 집계하기 위해 3개의 integer를 가짐.
    - 실패 호출 횟수, 느린 호출 횟수, 전체 호출 횟수
- 그리고 호출 기간을 저장하기 위한 1개의 long을 가짐.
- 말이 좀 어려울 수 있는데, [SlidingTimeWindowMetrics](https://github.com/resilience4j/resilience4j/blob/master/resilience4j-core/src/main/java/io/github/resilience4j/core/metrics/SlidingTimeWindowMetrics.java) 코드 함께 보면 쉬움.
