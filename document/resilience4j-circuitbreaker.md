# CircuitBreaker

- 8년 전 이후로는 실무에서 잘 사용 안 했었는데, 얼마나 바뀌었는지 오랜만에 가볍게 다시 살펴봄.
- https://resilience4j.readme.io/docs/circuitbreaker

## Introduction

- 3가지 일반 상태와 3가지 특수 상태를 갖는, 유한 상태 머신으로 구현됨.
    - 일반 상태: `CLOSED`, `OPEN`, `HALF_OPEN`
    - 특수 상태: `METRICS_ONLY`, `DISABLED`, `FORCED_OPEN`

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
- 부분 집계는 실패 호출 개수를 집계하기 위해 3개의 integer를 가짐.
    - 실패 호출 횟수, 느린 호출 횟수, 전체 호출 횟수
- 그리고 호출 기간을 저장하기 위한 1개의 long을 가짐.
- 말이 좀 어려울 수 있는데, [SlidingTimeWindowMetrics](https://github.com/resilience4j/resilience4j/blob/master/resilience4j-core/src/main/java/io/github/resilience4j/core/metrics/SlidingTimeWindowMetrics.java) 코드 함께 보면 쉬움.

## Failure rate and slow call rate thresholds

`CLOSED` -> `OPEN` 전환.

- 실패율이 설정된 임계치보다 같거나 크면,
- 써킷 브레이커 상태가 `CLOSED`에서 `OPEN`으로 바뀜.
- 기본적으로 모든 예외는 실패로 카운팅.
- 혹은 실패로 카운팅 할 예외 목록 정의도 가능.
- 그 외 예외는 성공으로 카운팅 하거나, 실패도 성공도 아닌 것으로 무시할 수도 있음.
- 실패 호출 말고 느린 호출 비율도 임계치를 넘어서면 `OPEN` 상태로 전이.
- 이는 외부 시스템이 무응답이 되기 전 부하 줄이기에 도움.

최소 호출 조건도 있음.

- 실패와 느린 호출 비율 계산은 최소 호출 수가 만족한 경우에만 일어남.
- 만약, 최소 수를 10으로 설정했다면, 9개 호출이 일어나고 9개 모두 실패여도, `OPEN` 상태가 되지 않음.

`OPEN` -> `CLOSED` 전환은 아래와 같이 일어남.

- 써킷 브레이커는 `OPEN` 상태가 되면 `CallNotPermittedException` 오류와 함께 호출을 거부.
- 일정 시간이 지나면 상태가 `OPEN`에서 `HALF_OPEN`으로 바뀜.
- 이 상태에서는 설정된 만큼의 호출을 허용하고 외부 시스템의 가용성을 확인.
- 호출 허용된 요청들이 완료될 때까지 나머지 호출은 `CallNotPermittedException` 예외로 거부.
- 실패 또는 느린 호출 비율이 임계치보다 낮으면 다시 `CLOSED` 상태로 전환.

특수 상태들도 있음.

- `METRICS_ONLY`: 항상 접근 허용. 상태 전환을 제외한 모든 이벤트가 여전히 일어나고 메트릭도 기록되지만 회로가 열리지는 않음.
- `DISABLED`: 항상 접근 허용. 이벤트도 일어나지 않고, 메트릭도 기록되지 않음.
- `FORCED_OPEN`: 항상 접근 차단. 이벤트와 메트릭 모두 `DISABLED`와 마찬가지로 일어나지 않음.

스레드 세이프함.

- 모든 상태는 `AtomicReference`에 저장됨.
- 부수 효과 없는 함수들과 함께 원자적 연산으로 값을 변경.
- 슬라이딩 윈도우의 기록과 스냅샷 읽기는 `synchronize`.
- 원자성은 보장되어야 하고, 한 번에 하나의 스레드만 상태나 슬라이딩 윈도우를 업데이트.
- 스레드 세이프 설명에 부합하는 코드 찾아보니 아래와 같음.

```java
private class ClosedState implements CircuitBreakerState {
  // ... 중략
  private void checkIfThresholdsExceeded(Result result) {
    if (Result.hasExceededThresholds(result) && isClosed.compareAndSet(true, false)) {
      publishCircuitThresholdsExceededEvent(result, circuitBreakerMetrics);
      transitionToOpenState();
    }
  }
}

public class SlidingTimeWindowMetrics implements Metrics {
  // ... 중략
  public synchronized Snapshot record(long duration, TimeUnit durationUnit, Outcome outcome) {
    totalAggregation.record(duration, durationUnit, outcome);
    moveWindowToCurrentEpochSecond(getLatestPartialAggregation())
            .record(duration, durationUnit, outcome);
    return new SnapshotImpl(totalAggregation);
  }

  public synchronized Snapshot getSnapshot() {
    moveWindowToCurrentEpochSecond(getLatestPartialAggregation());
    return new SnapshotImpl(totalAggregation);
  }
}
```

비즈니스 로직은 동기화하지 않음.

> But the CircuitBreaker does not synchronize the function call.

- 위와 같이 써져 있음.
- 아마도 써킷 브레이커로 감싼 애플리케이션 코드에 대해 `synchronize` 하지 않는다는 의미일 것.
- 이는 너무나 큰 성능 저하로 이어질 수 있음.
- 슬라이딩 윈도우 사이즈가 15이고, 동시에 20개의 스레드에서 함수 실행 요구를 했다면, 20개 모두 실행됨.
- 슬라이딩 윈도우 사이즈가 동시에 최대 실행 가능한 작업 개수를 의미하지 않음.
- 만약 이런 기능이 필요하다면 Bulkhead 기능을 사용하라고 함.

3개 스레드가 동시에 함수 실행을 할 때 일어나는 예시 그림을 제공하고 있음.

![](https://files.readme.io/8d10418-Multiplethreads.PNG)

## Create and configure a CircuitBreaker

- `ConcurrentHashMap`에 기반한 인메모리 `CircuitBreakerRegistry`를 사용해서 `CircuitBreaker` 인스턴스를 생성하고 조회.
- 모든 `CircuitBreaker` 인스턴스에 대해 전역 디폴트 `CircuitBreakerConfig`를 가진 `CircuitBreakerRegistry`를 생성하려면 아래와 같이 하면 됨. 

```java
CircuitBreakerRegistry circuitBreakerRegistry = 
  CircuitBreakerRegistry.ofDefaults();
```

- 그리고 CHM을 어디에 어떻게 쓰는가 했더니, 설정값들을 CHM으로 관리하고 있음.

```java
public class AbstractRegistry<E, C> implements Registry<E, C> {

    // 중략
    protected final ConcurrentMap<String, C> configurations;

```