# Living in Production

- feature complete != production ready
- crash, hang, lose data, violate privacy, lose money, crazy real uses, globe-spanning taffic, virus writing mobs, …
- 최선의 계획을 세우더라도 나쁜 일이 일어난다는 것을 받아들여야 함. 따라서, 할 수 있는 것들을 최대한 하되, 어떤 심각한 상황에서도 전체 시스템이 회복될 수 있어야 함.

## Aiming for the Right Target

- "고객의 첫 번째와 마지막 이름은 필수고, 중간은 선택이에요"와 같은 테스트를 통과하기 위함이 아님.
- 대신, "design for production"

## The scope of the Challenge

- 사용자는 늘어나고, 더 높은 수준의 가용성이 요구됨.
- 빌드 비용이 저렴하고, 사용자에게 유용하고, 운영 비용이 적은 소프트웨어를 빠르게 만들어내야 함.
- 조그마한 워드프레스에 적절한 설계는 대규모 확장과, 트랜잭션, 분산 시스템 등의 요구에 실패할 것.

## A Million Dollars Here, a Million Dollars There

- 설계와 아키텍처 결정은 또한 재정적 결정.
- 이런 결정들은 구현 비용과 더불어 후속 비용(가용성이나 배포 비용 등의 운영을 가리키는 듯)도 함께 고려해야 함.
- 기술적 그리고 재정적 관점은 이 책에서 반복적으로 다루는 중요한 테마.

## Use the Force

- "Use the Force"가 무슨 말인가 했더니, 스타워즈에 나오는, 앞을 내다보라는 식의 말인 것 같다.
- 저자는 열성적인 애자일 지지자이긴 하지만, 초기 결정의 중요성에 대해 말하고 있음.
- 이 결정이 시스템의 궁극적 모습에 지대한 영향을 미치기도 하고, 때로는 돌이킬 수 없기도 함.
- 앞으로 책에서, 어떤 문제에 대한 대안을 소개하고, 이들이 선택된 이후에 어떤 영향을 미치는지 살펴본다고 함. 궁금.

## Pragmatic Architecture

- 저자는 아키텍트를 2개로 분류하는데, 하나는 상아탑<sup>ivory towel</sup>이고, 다른 하나는 실용주의.
- 높은 추상화를 추구하기 보다는 현실적 문제를 다룸. 메모리 사용량, CPU 요건, 대역폭, 하이퍼스레딩과 CPU 바인딩의 이점과 단점 등을 논의.
- 특별한 이야기는 아님. 개인적으로는 지면이 아까운..

