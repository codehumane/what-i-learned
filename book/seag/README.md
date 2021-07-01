# Software Engineering at Google

# Thesis

# 1. What is Software Engineering?

프로그래밍과 소프트웨어 엔지니어링의 3가지 중요한 차이.

1. 시간
   - 수명이 길어질수록 변경이 많음.
   - 비즈니스, OS, 하드웨어, 언어 버전 등.
   - 이에 대응할 수 있는 것이 중요해짐.
   - 수명이 짧은 코드는 단지 프로그래밍 문제에 가까움.
2. 스케일
   - 프로그래밍 작업은 보통 개인의 작업.
   - 소프트웨어 엔지니어링은 팀의 노력.
   - 팀 협업은 개인 프로그래밍에 없던 여러 가지 새로운 문제를 안겨줌.
   - 하지만 혼자할 때에 비해, 좋은 시스템을 만들 가능성이 큼.
   - 제품, 조직, 개발 워크플로우의 규모가 커짐에 따른 비용을 잘 유지하고 관리해야 함.
3. 트레이드 오프
   - 종종 부정확한 가치 메트릭을 기반으로,
   - 상위 수준의 이해관계 결과물을 고려하며,
   - 여러 길 사이의 트레이드 오프를 고려한 복잡한 결정을 내려야 함.

## Time and Change

- 모바일 앱이나 스타트업의 코드 수명은 비교적 짧음.
- 하지만 구글 검색이나 Apache HTTP 프로젝트의 수명은 예측할 수 없음.
- 내부적으로는 indefinitely로 간주한다고 함. 
- 이런 수명이 긴 프로젝트에서는 업그레이드 문제가 중요.
- 이런 업그레이드를 고려하지 않다가 한 번에 업그레이드 하려는 경우 그 비용은 상당.
- 이 비용을 겪고 나면 후속 업그레이드도 크다고 느껴, 코드를 재작성하거나 아예 업그레이드 자체를 피하기도.
- 하지만 첫 번째 큰 업그레이드 이후에도 지속적으로 나아가는 것이 프로젝트의 장기 유지가능성의 핵심.
- 물론, 업그레이드가 제공하는 가치와 비용, 그리고 프로젝트의 기대 수명을 같이 고려해야.

### Hyrum's Law

다른 엔지니어들에게 사용되는 프로젝트를 맡고 있다면, Hyrum's Law가 "it works"와 "it is maintainable"의 차이에 대한 중요한 교훈이 됨.

> With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody.

### Why Not Just Aim for "Nothing Changes"?

- 새로운 기능을 지속적으로 제공해야 하고
- 보안 취약점이 새로 발견되고
- 버그도 계속 발견됨
- CPU 등의 변경에 따라 최적화도 계속 달라짐
- 수명이 긴 프로젝트라면 변경을 피하기 어려움
