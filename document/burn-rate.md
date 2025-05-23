
# Burn Rate Alert

https://docs.datadoghq.com/service_management/service_level_objectives/burn_rate/

## Summary

- SLO 소진율은 SLO 에러 예산의 소비 속도를 가리킴.
- 만약, 이 소비 속도가 임계치를 넘어 일정 기간 이상 지속되었다면 경고.
- 예를 들어, 30일 기준의 SLO에 대해, 지난 1시간 동안의 소진율이 14.4을 넘어선 게 최근 5분간 감지되면 경고.
- 좀 더 낮은 수준의 임계치(예컨대 7.2)에 대해서는 경고 수준의 알림을 보낼 수도 있음.

## How Burn Rate Alerts work

- 소진율은 SLO 목표 기간에 대해 상대적으로 얼마나 에러 예산을 빠르게 소비하고 있느냐를 나타냄.
- 30일 기간을 목표로 할 때, 소진율 1은 30일 동안 정확히 모든 예산을 딱 맞춰 쓰게 된다는 의미.
- 소진율 2는 15일 만에 다 쓴다는 것이고 3은 10일에 다 끝난다는 것.
- 공식으로 나타내면 아래와 같음.

```
length of SLO target (7, 30 or 90 days) / burn rate = time until error budget is fully consumed
```

- 소진율 경고는 최근 "에러 비율"을 사용함.
- 여기서의 에러 비율은 주어진 기간 동안 전체 동작에 대해 잘못된 동작의 비율을 의미.

```
error rate = (1 − good behavior during time period) / total behavior during time period
```

- "행위"는 SLO에 따라 다름.
- 메트릭 기반 SLO는 뭔가에 대한 발생 빈도(성공이나 실패 요청 수 등)를, 모니터 기반 SLO는 시간의 양을 측정(다운타임, 업타임 등).
- 목표 SLO(99.9% 같은)에 대한 에러 예산은 아래와 같이 계산.

```
error budget = 100% − SLO Target
```

- 다른 말로, 에러 예산은 지속하고 싶은 이상적 에러 비율.
- 따라서, 소진율은 이상적인 에러율에 대한 배수로 해석할 수 있음.
- 예를 들어, 30일간 99.9% SLO에 대해, 만약 소진율이 10이라면, 에러 예산은 3일만에 소진될 것이며, 관측된 에러 비율이 이상적인 것에 비해 10배 높다는 의미.

```
( burn rate ) ( ideal error rate ) = observed error rate
( 10 ) ( 0.001 ) = 0.01
```

- 이상적으로 소진율은 1을 유지하는 게 좋음.
- 책 설명과 달리 현실적으로 1 미만으로 유지해야 하는 게 좋다고 생각.
- 예산이라서 억지로 꼭 써야 하는 건 아님. 쓸 수 밖에 없는 상황일 때 한계치를 고려해야 하는 것.
- 실세계에서는 이슈나 장애가 발생하면 소진율이 급격히 올라가는 변동성을 가짐.
- 따라서, 소진율 기반의 알림은, 에러 예산을 빠르게 소모하는 문제가 발생했을 때, 그리고 이로 인해 SLO를 지키지 못할 가능성이 커질 때, 선제적으로 감지해서 대응할 수 있게 도움.

```
( long window error rate / (1 - SLO target) ≥ burn rate threshold ) ∧ ( short window error rate / (1 - SLO target) ≥ burn rate threshold ) = ALERT
```

- 소진율 알림을 설정할 때는, 소진율 임계치와 더불어, "long alerting window"와 "short alerting window"를 함께 설정해야 함.
- 긴 윈도우는 충분히 긴 기간 동안 측정하므로 심각한 이슈를 알려줌.
- 이는 사소한 이슈에 의한 불필요한 알림을 피할 수 있게 함.
- 한편 짧은 윈도우는 분 단위.
- 이슈가 종료되었을 때 모니터링 상태를 빠르게 복구해 줌.
- 구글에서는 짧은 윈도우를 긴 윈도우의 1/12로 하는 걸 추천.

## Maximum burn rate values

- 위에서 보았듯 긴 윈도우나 짧은 윈도우를 위한 소진율 공식은 아래와 같음.

```
error rate / (1 − SLO target)
```

- 에러 비율의 최대값은 1 (동작의 100%가 에러)
- 따라서, 최대 소진율은 아래와 같은 공식을 가짐.

```
1 / (1 - SLO target)
```

- SLO 목표가 낮을수록, 최대 소진율도 낮아짐.
- 따라서, 이보다 높은 소진율 임계치를 설정하면 알림이 절대 발생하지 않음.

## Picking burn rate values

- 소진율 값을 고르는 건, SLO의 목표와 시간 윈도우에 따라 다름.
- 소진율 알림을 설정할 때 소진율 임계치와 긴 시간 윈도우 설정에 집중해야 함.
- 짧은 시간 윈도우는 보통 긴 시간 윈도우의 1/12로 시작.
- 위에서 언급한 최대치를 넘지 않는 것도 중요.

### Approach #1: Time to error budget depletion

```
length of SLO target (7, 30 or 90 days) / burn rate = time until error budget is fully consumed
```

- 소진율을 계산한 뒤, 에러 예산이 전부 소진되면 심각한 이슈가 될 만한 기간을 고르기.
- 예를 들어, 3일 만에 소진이 다 되면 이건 진짜 심각해라고 한다면 소진율은 `30일 / 3일 = 10`으로 잡으면 됨.
- 긴 알림 윈도우는, 사소한 일시적 문제가 아니라, 정말로 이슈라고 여겨질 정도로 소진율이 지속되는 기간을 잡기.
- 소진율 값이 높을수록 긴 윈도우는 짧게 잡는 게 좋음(심각한 이슈를 빠르게 발견할 수 있게).

### Approach #2: Theoretical error budget consumption

- 한편, 다른 접근법도 있음.
- 소진율과 긴 윈도우의 조합을 이론적 에러 예산 소비로 보는 것.

```
burn rate = length of SLO target (in hours)  *  percentage of error budget consumed / long window (in hours)  * 100 %
burn rate = (SLO 목표 기간(시간) × 소비된 에러 버짓 비율) / 긴 윈도우(시간) × 100%
```

- 예를 들어, 7일 기간의 SLO에 대해, 1시간 동안 에러 예산을 10% 이상 소비했을 때 알림을 받겠다면, 소진율은 아래와 같이 계산.
- 그러니까, "에러 버짓을 얼마나 빠르게 쓰고 있는가?"를 정량적으로 계산할 수 있는 방법.
- 특정 시간 안에 몇 퍼센트를 소모하면 심각하다고 볼 것인지를 기준으로, 소진율을 역산해서 결정할 수 있음.

```
burn rate = (7 days * 24 hours * 10 % error budget consumed) / 1 hour * 100 % = 16.8
burn rate = (7일 × 24시간 × 10%) / 1시간 × 100% = 16.8
```

## Examples

- DataDog에서 7, 30, 90일 SLO에 대한 추천 값을 표로 제공.
- 이 예시들은 99.9%를 목표로 한다고 가정한 것.
- 먼저 7일.

| Burn Rate | Long Window | Short Window | Theoretical Error Budget Consumed |
|-----------|-------------|---------------|-----------------------------------|
| 16.8      | 1 hour      | 5 minutes     | 10%                               |
| 5.6       | 6 hours     | 30 minutes    | 20%                               |
| 2.8       | 24 hours    | 120 minutes   | 40%                               |

- 다음은 30일.

| Burn Rate | Long Window | Short Window | Theoretical Error Budget Consumed |
|-----------|-------------|---------------|-----------------------------------|
| 14.4      | 1 hour      | 5 minutes     | 2%                                |
| 6         | 6 hours     | 30 minutes    | 5%                                |
| 3         | 24 hours    | 120 minutes   | 10%                               |

- 마지막으로 90일.

| Burn Rate | Long Window | Short Window | Theoretical Error Budget Consumed |
|-----------|-------------|---------------|-----------------------------------|
| 21.6      | 1 hour      | 5 minutes     | 1%                                |
| 10.8      | 6 hours     | 30 minutes    | 3%                                |
| 4.5       | 24 hours    | 120 minutes   | 5%                                |
