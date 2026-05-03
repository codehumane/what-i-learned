
# DORA’s software delivery performance metrics

https://dora.dev/guides/dora-metrics/

- 기술 중심 팀은 성과를 측정할 수 있어야 함
- 그래야 현재 하고 있는 것을 평가하고, 개선 우선순위를 조정하고, 프로세스를 검증할 수 있음
- DORA는 5가지 개발 성과 지표를 식별했고, 이를 통해 소프트웨어 개발 프로세스의 결과를 효과적으로 측정하게 함
- [DORA의 연구](https://dora.dev/research/)에 따르면, 이 지표는 더 나은 조직의 성과와 팀원들의 만족도 향상을 예측하는데 도움
- 이 지표는 선행 지표(시스템의 잠재적인 미래 변화 예고) 또는 후행 지표(과거의 성능과 결과를 반영) 모두로 볼 수 있음

## Throughput and instability

- DORA의 성능 지표는, 소프트웨어를 안전하고 빠르고 효율적으로 개발하는 팀의 역량에 중점을 둠
- 이는 크게 2가지로 나눌 수 있음 — ① 소프트웨어 변경의 처리량 ② 소프트웨어 변경의 불안정성

### Throughput

처리량은 일정 시간 동안 시스템이 얼마나 변하는지를 측정. 다음의 3가지로 측정.

1. 변경 리드 타임 — 버전 관리에 커밋이 프로덕션에 배포되기까지 걸린 시간
2. 배포 주기 — 주어진 기간 동안의 배포 횟수 혹은 배포 간 간격
3. 배포 실패 복구 시간 — 배포 실패로 즉각 조치가 필요한 상황에서 복구에 걸린 시간

### Instability

불안정성은 소프트웨어가 얼마나 잘 배포되는가에 대한 측정. 배포가 잘 되면, 팀은 더 자신감 있게 더 많은 변경을 프로덕션에 내보낼 수 있고, 사용자들은 배포 직후 이슈를 덜 겪게 됨. 아래 2가지로 이를 측정.

1. 변경 실패 비율 — 전체 배포 중 롤백이나 핫픽스 등의 즉각적인 조치가 필요한 배포의 비율
2. 배포 재작업<sup>rework</sup> 비율 — 예측하지 못한 프로덕션의 사고 결과에 대응하기 위한 배포의 비율

## Key insights

- DORA의 연구는 **속도와 안정성은 트레이드오프가 아님**을 거듭 증명함
- 사실은 이 지표들이 대부분의 팀들에서 상관관계를 보임
- 이 지표들은 어떤 기술 유형에도 동작
- 다만, 한 번에 하나의 서비스나 애플리케이션을 측정하는데 적합
- 그리고 맥락이 중요하다는 이야기도 언급
- 팀이 개발하는 서비스나 애플리케이션의 맥락에 맞게 지표들을 적용할 것
- 여러 팀이나 조직에 동일하게 지표를 적용하기는 어려움

> “…the real trade-off, over long periods of time, is between better software faster and worse software slower.” —Farley, D. (2021). [Modern Software Engineering: Doing what works to build better software faster](https://www.google.com/books/edition/Modern_Software_Engineering/rtnPEAAAQBAJ) (p. 154). Addison-Wesley.

## Common pitfalls

- 지표를 목표로 삼는 함정 — [굿하트<sup>Goodhart</sup>의 법칙](https://en.wikipedia.org/wiki/Goodhart%27s_law)을 무시하고, "모든 애플리케이션은 연말까지 하루에 여러 번 배포해야 함" 같은 포괄적 문장 만들기는, 팀이 지표를 편법으로 맞추려 할 가능성이 높음
- 하나의 지표로 모든 조직에 일괄 적용 — 여러 지표를 파악하고 지표들 간 적절한 균형을 유지해야 함 ([Choosing measurement frameworks to fit your organizational goals](https://dora.dev/insights/measurement-frameworks/) 참고)
- 업계의 관행을 개선의 방패막으로 삼기 — 규제 강한 산업의 일부 팀은 컴플라이언스 요구사항 때문에 현상 유지를 주장
- 서로 다른 것 비교하기 — 이 지표들은 애플리케이션이나 서비스 수준에 적용되도록 설계되었고, 서로 다른 것들을 비교하기에는 부적절
- 사일로화 된 오너십 갖기 — 개발, 운영, 릴리즈 팀 간 다섯 가지 핵심 지표를 모두 공유해야 협업 촉진과 소유권 강화
- 경쟁하기 — 목적은 [팀의 성과를 향상](https://dora.dev/guides/how-to-empower-software-delivery-teams/)시키는 것이고, 다른 팀과 조직과의 경쟁이 아님
- 개선이 아닌 측정에만 집중하기 — 지표에 필요한 데이터는 여러 곳에서 얻을 수 있으며, 정확한 데이터를 얻기 위한 시스템 구축은 투자 대비 효과가 떨어질 수 있으므로, 담당자와 대화를 먼저 나누거나, [DORA Quick Check](https://dora.dev/quickcheck/?v=2025)을 활용하거나, [source-available](https://dora.dev/resources/#source-available-tools) 혹은 상용 제품 사용을 고려

## Dive into the research

- DORA의 연구는 다섯 가지 지표를 넘어 고성능에 기여하는 다양한 역량을 탐구
- 이 역량과 소프트웨어 개발이 미치는 영향은 [역량 카탈로그](https://dora.dev/capabilities/)에서 자세히 확인

## Next steps

다섯 가지 지표를 개선하는 일반적인 접근법은 [변경의 크기를 줄이는 것](https://dora.dev/capabilities/working-in-small-batches/)

- 변경 크기가 작을수록 필요성과 영향을 파악하기 쉽고 배포하기도 쉬움
- 작은 변경은 실패 복구도 더 쉬움
- 더 안전하고 더 빠르게 변경할 수 있음
- 처리량<sup>throughput</sup> 그리고 안정성<sup>stability</sup> 모두 가능

애플리케이션의 우선순위 조정, 개발, 배포, 운영을 담당하는 팀을 목적 조직으로 모아 다음 단계를 진행

1. 기준선 설정 — [DORA Quick Check](https://dora.dev/quickcheck/?v=2025) 통해 현재 퍼포먼스 확인
2. 대화 — [배포 과정 도식화](https://dora.dev/guides/value-stream-management/) 등을 활용해, 개발/배포 단계에서의 문제점을 이야기
3. 개선을 다짐 — 목적 조직이 함께 가장 주요한 제약이나 병목에 대한 개선 게획
4. 다짐을 계획으로 전환 — 코드 검토 시간이나 테스트 품질 측정 등, 선행 지표 역할을 할 수 있는 몇 가지 구체적 측정 기준을 포함해 계획 수립
5. 실행 — 지름길은 없음
6. 점검 — 다시 DORA Quick Check을 사용하고 대화하고 회고하고 프로세스를 점검
7. 반복
