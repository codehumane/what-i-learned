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
