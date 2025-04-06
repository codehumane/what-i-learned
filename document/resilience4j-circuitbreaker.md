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
