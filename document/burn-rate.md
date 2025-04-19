
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

```math
length of SLO target (7, 30 or 90 days) / burn rate = time until error budget is fully consumed
```

- 소진율 경고는 최근 "에러 비율"을 사용함.
- 여기서의 에러 비율은 주어진 기간 동안 전체 동작에 대해 잘못된 동작의 비율을 의미.

```math
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
