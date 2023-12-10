# Multi-Cloud Strategy for Cloud Architects - Second Edition

# CH1. Introduction to Multi-Cloud

- 멀티 클라우드의 선택 이유로, 안정성이 아닌 사업 민첩성을 언급.
- 사업의 필요에 맞는 클라우드 서비스를 선택하는 것.
- 이 첫 문단을 보니 내가 기대한 내용이 아니겠다 싶음.
- 당연한 얘기지만, 요구사항의 수집이, 멀티 클라우드로의 전환에 앞서 가장 중요.
- 그래서 VOC를 수집하는 도구들 얘기도 다룬다고 함.

## Understanding multi-cloud concepts

https://www.techopedia.com/definition/33511/multi-cloud-strategy

> Multi-cloud refers to the use of two or more cloud computing systems at the same time. The deployment might use public clouds, private clouds, or some combination of the two. Multi-cloud deployments aim to offer redundancy in case of hardware/software failures and avoid vendor lock-in.

## Multi-cloud⎯more than just public and private

- 하이브리드 IT와 멀티 클라우드의 차이점 중 하나는,
- 하이브리드는 homogeneous고 멀티 클라우드는 heterogeneous라는 것.
- 하이브리드는 클라우드 솔루션이 한 스택에 속함(Azure 퍼블릭 클라우드 + Azure Stack 온프레미스).
- 퍼블릭 클라우드는 예를 들어 Azure와 AWS의 조합.
- 하이브리드는 매우 일반적인 배포 모델이며, 클라우드의 미래 모델로 꼽히는 대상.
- 장점은 보안성과 지연(온프레미스가 가진 장점이기도 한).
- 하지만 퍼블릭 클라우드에서의 개발이 보다 민첩.
- 그래서 보안성과 지연이 중요한 것은 온프레미스에,
- 민첩한 개발이 필요한 개발은 퍼블릭 클라우드에서 진행.
- 이것이 하이브리드 IT의 탄생.
- 그러나 책에서 우리는 하이브리드 IT와 멀티 클라우드를 구분할 것.
