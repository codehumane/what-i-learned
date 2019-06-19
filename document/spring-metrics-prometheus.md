# Spring Metrics, Prometheus

> Spring Metrics의 Prometheus 문서를 보면서 간단히 기록

[https://docs.spring.io/spring-metrics/docs/current/public/prometheus](https://docs.spring.io/spring-metrics/docs/current/public/prometheus)

## Meters and Registries

일단, 아래는 meter의 정의. (메트릭 수집기 정도로 이해)

> A meter is the interface for collecting a set of measurements (which we individually call metrics) about your application.

`Meter` 인터페이스(문서에서는 primitives라고 부르는 것이 인상적)의 구현체인 `Timer`, `Counter`, `Gauge`, `DistributionSummary`, `LongTaskTimer`를 제공.

registry는 아래와 같이 정의.

> A registry creates and manages your application's set of meters.

그리고 Exporter는 이 meter registry를 이용해서 meter들을 순회(meter들의 metric 또한 순회). 그리고 시계열 데이터를 산출. registry로는 `SpectatorMeterRegistry`, `PrometheusMeterRegistr`, `SimpleMeterRegistry`가 제공됨. 

## Dimensions/Tags

- meter는 이름과 dimension들로 고유하게 식별됨.
- 여기서는 dimension과 tag라는 용어를 같은 의미로 사용.
- `spring-metrics`에서는 `Tag` 인터페이스를 사용. 더 짧기 때문.
- 이름은 피벗으로 사용될 수 있어야 함.
- 그리고 dimension은 특정 이름의 메트릭을 분할하여 자세히 들여다 볼 수 있는 용도로 사용.
- 아래는 예시. 스레드 풀 갯수와 데이터베이스 테이블 로우 수.

```java
// recommended
registry.counter("threadpool_size", "id", "server_requests")
registry.counter("db_size", "table", "users")

// bad
registry.counter("size", "class", "ThreadPool", "id", "server_requests");
registry.counter("size", "class", "Database", "table", "users");
```

### Metric and tag naming

- 언더스코어 네이밍을 사용하길 권장.
- 카멜케이스도 괜찮음.
- 대시(-)와 컴마(.)는 피할 것.
- 도구에 따라 뺄셈이나 계층 구조 등으로 인식할 수 있기 때문.

## Counters

- 단일 메트릭. 갯수.
- allow increment by a fixed amount
- 보통 카운터를 사용할 때는 특정 기간(interval)에 이벤트가 일어난 비율(rate)에 관심.
- 프로메테우스에서는 아래처럼 쿼리를 작성.

```
rate(counter[10s])
```

- 그리고 카운터를 사용하기 전에는 혹시 타이머(timer)가 필요한 것은 아닌지 고민.
- 타이머가 카운터를 포함한다고 생각하면 됨. 중복으로 만들 필요 X.

## Timers

- short-duration 동안의 지연이나 이벤트 빈도 측정에 유용.
- 타이머에 대한 base unit으로는 seconds를 사용하길 권장. 대신 double 형을 사용.
- `spring-metrics`에서는 혼란을 줄이기 위해 `TimeUnit`을 요구.
- 프로메테우스에서는 이 값을 아래 처럼 쿼리.

```
average latency = rate(timer_sum[10s]) / rate(timer_count[10s])
throughput(rps) = rate(timer_count[10s])
```

## Gauge

- 현재 값을 얻는 데 사용.
- 예) 컬렉션이나 맵의 크기, 실행 중인 스레드의 갯수 등.
- 메트릭이 샘플링 되므로 값이 관찰되지 않는 중간 중간에는 유실된다고 생각.
- 상한선을 모니터링 하는 데 유용.
- 요청 횟수와 같이 인스턴스 실행 중에 계속 증가하는 값들은 X.
- 감소를 허용하는! 카운터의 일반화 된 값

## Quantile Statistics

- timer, distribution summaries를 quantiles로 덧붙일 수 있음.
- 아래와 같이 사용. `@Timed` 사용할 때도 가능.

```java
Timer timer = meterRegistry.timerBuilder("my_timer")
				.quantiles(WindowSketchQuantiles.quantiles(0.5, 0.95).create())
				.create()
```

