# Case Study: Phenomenal Cosmic Powers, Itty-Bitty Living Space

새로운 사이트를 만들어서 운영하기 시작. 첫 추수 감사절은 잘 지나감. 다음 날인 블랙 프라이데이가 되었고, 시스템 모니터링 지표들을 살펴봄. 주문 수는 전날보다 늘었지만, 페이지 레이턴시는 여전히 250ms 이하를 유지하고 있었음. 그런데 어느 순간, 모든 DRP에 빨간색이 표기되고, DRP의 롤링 재시작은 즉각 실패하고 있었음. 아래는 그 당시의 몇 가지 증상.

1. 세션 수가 전날보다도 높음.
2. 네트워크 대역폭 사용량이 높았지만 한계치에 도달한 건 아님.
3. 애플리케이션 서버 페이지 레이턴시가 높음.
4. 웹, 애플리케이션, 데이터베이스 CPU 사용량은 매우 낮음.
5. 검색 서버(종종 우리의 범인이었던)는 잘 응답 중. 시스템 통계는 건강해 보임.
6. 요청을 다루는 스레드들이 모두 바쁨. 상당수의 스레드가 5초 이상 자신의 일을 처리하는 중.

사실, 페이지 레이턴시는 문제를 잘 드러내지 못했음. 오래 걸리는 것들은 타임아웃이 걸리고, 통계는 성공적으로 요청을 처리한 경우만을 계산한 결과이기 때문. (그렇다고 실패하는 응답을 포함하기도 어려움. 실패하는 요청은 일반적으로 성공한 경우보다 빠른 시간 안에 끝나기 때문. 잘못된 응답 지연을 만들어 내는 것은 마찬가지. 목적을 생각해 보자) 문제가 있다는 전화를 받은지 거의 90분이 지나고 있었음. SLA를 어기기 직전의 상황.

프론트엔드 애플리케이션 서버의 스레드 덤프는 다음과 같은 모습을 보여주고 있었음.

1. 일부 스레드가 백엔드 호출을 위해 바쁜 상태였고,
2. 대부분의 스레드는 백엔드 연결을 기다리는 중.
3. 백엔드에 대한 연결 리소스 풀에는 타임아웃이 X.
4. 백엔드가 응답을 주지 않으면 프론트엔드는 계속 기다려야만 하는 상황.

다음으로 주문 관리 시스템을 스레드 덤프로 확인.

1. 외부 연동 지점 호출에 450개의 스레드가 사용되고,
2. 나머지 스레드는 이 호출이 끝나기를 기다리는 중.
3. 이 외부 연동 지점은 배송을 위한 스케쥴링 서버.
4. 보통 4대의 인스턴스가 가동되지만,
5. 유지보수를 위해 휴일 동안 2대를 중단시켰으며,
6. 남은 인스턴스 중 1대는 알 수 없는 이유로 이상 동작을 보이고 있었음.

결국, [여기 그림](https://learning.oreilly.com/library/view/release-it-2nd/9781680504552/images/case_study_living_space/scheduling.png)에서 보듯 시스템 크기의 큰 불균형이 발생했던 것. 그런데, 남아 있던 스케쥴링 서버가 1대였으면, 부하가 발생했을 텐데, 왜 모니터링이 되지 않았을까?

1. 알림을 받긴 했지만,
2. 평소에도 일시적인 CPU 스파이크 알림이 있었고,
3. 이 스파이크는 거짓 경보였기에,
4. 서버가 1대만 남아 생긴 부하 알림까지 무시하게 된 것.

여러가지 방법을 찾던 중, 문제가 생기면 커넥션 풀 관리 컴포넌트를 재시작(복구)하기로 함. 구체적으로는 커넥션 관리 컴포넌트의 `max` 프로퍼티를 적절히 설정하고, `stopService`와 `startService`를 호출하는 것. 책에서는 이게 *recovery-oriented computing*의 핵심 개념이라고 함. ROC 제안의 수준은 아니겠지만, 전체를 재시작하는 대신, 일부만 복구하는 것.

참고로, ROC 원칙은 다음과 같음.

1. Failures are inevitable, in both hardware and software.
2. Modeling and analysis can never be sufficiently complete. A *priori* prediction of all failure modes is not possible.
3. Human action is a major source of system failures.

# Foundations

## Networking in the Data Center and the Cloud

데이터 센터와 클라우드에서 네트워킹을 한다는 것은 단순히 소켓 열기를 넘어, 별도의 노력과 보안이 요구되는 것.

### NICS AND NAMES

호스트네임에 대한 오해들이 있음. *hostname*이 완전히 별개인 2가지로 정의될 수 있기 때문.

1. OS가 자기 자신을 인식하기 위해 사용하는 이름. 호스트네임과 "[default search domain](https://apple.stackexchange.com/questions/286395/what-does-the-search-domain-in-the-network-preferences-do)"이 합쳐져 FQDN(Fully Qualified Domain Name)이 됨.
2. 외부 시스템에 대한 이름. DNS 리졸브 대상.

이로 인해, 장비가 가진 FQDN이, DNS가 IP 주소를 위해 가지고 있는 FQDN과 같은 것인지를 보장할 수 없음. 예컨대, "spock.example.com"에 대한 장비의 FQDN 집합(?)을 가지면서, "mail.example.com"과 "www.example.com"과 같은 DNS 매핑을 가질 수 있음. 장비의 호스트네임은 장비 식별에 사용되는 반면, DNS 네임은 IP 주소 식별에 사용. 많은 유틸리티와 프로그램은 장비 스스로에게 할당한 FQDN을 합법적인 DNS 이름으로 간주.

그 외에도 다양한 복잡함이 존재. "DNS 네임 -> IP 주소"는 다대다 관계. 단일 장비는 다수의 NIC를 가짐(멀티호밍<sup>multihoming</sup>). 이더넷 포트, Wi-Fi, 루프백 NIC, ... 그리고 데이터 센터는 다른 목적으로 멀티호밍을 함. 관리와 모니터링을 위한 네트워크를 서로 분리하여 보안을 강화하는 것. 또한, 백업 같이 높은 트래픽이 필요한 네트워크와 프로덕션 트래픽을 분리하기도 함. [본딩과 티밍](https://zetawiki.com/wiki/%EB%B3%B8%EB%94%A9,_%ED%8B%B0%EB%B0%8D)도 언급.

### PROGRAMMING FOR MULTIPLE NETWORKS

다수의 인터페이스는 애플리케이션에도 영향을 미침. 언어들이 지원하는 "쉬운" 버전의 소켓 리스닝을 피할 것.

```
// 안 좋은 접근법
ln, err := net.Listen("tcp", ":8080")

// 좋은 접근법
ln, err := net.Listen("tcp", "spock.example.com:8080")
```

어떤 인터페이스에 바인딩을 할 것인지 결정하기 위해서는, 애플리케이션이 장비의 이름과 IP 주소들을 알아야 함. 멀티호밍 된 장비에서 `getLocalHost` 등의 호출은 서버 내부의 호스트네임에 대응하는 IP 주소를 반환. 또한, 로컬 네이밍 컨벤션에 따라 얼마든지 달라질 수 있음. 따라서, 소켓을 리슨해야 하는 서버 애플리케이션은 어떤 인터페이스를 바인딩할 것인지를 결정할 수 있도록 설정 가능한 프로퍼티를 제공해야 함.

## Physical Hosts, Virtual Machines, and Containers

모든 장비들은 어느 정도 공통점을 갖긴 하지만 서로 다른 부분도 많음. 물리적 데이터 센터 환경에서 잘 동작하는 설계가 컨테이너화 된 클라우드 환경에서는 많은 비용을 초래할 수도.

### PHYSICAL HOSTS

기록할 만한 내용 없음.

### VIRTUAL MACHINES IN THE DATA CENTER

가상화는 리소스를 효율적으로 사용할 수 있게 도와주지만, 종종 성능 문제를 야기함. `물리적 장비가 가진 리소스 < 물리적 장비 위에 올라간 가상화 장비들의 리소스 합`이 일반적이기 때문. 따라서, 가상화 장비에서 동작하는 애플리케이션은, 언제든 리소스 부족이나 성능 저하가 일어날 수 있음을 고려해야 함. 시계 문제도 언급. OS 시계는 믿지 않되, 실제 시간이 중요하다면 로컬 NTP 서버 같은 외부 소스를 사용.

### CONTAINERS IN THE DATA CENTERS

> “I’ll never again have to ask if production matches QA.”

컨테이너의 수명은 짧고, 따라서 인스턴스 기반의 설정은 피해야 함. 예컨대, Nagios 같은 오래된 모니터링 시스템은 장비가 추가되거나 재시작할 때마다 재설정이 필요. 또한, 로컬 저장소가 충분치 않음. 외부 스토리지 활용이 필요. 가장 어려운 부분은 네트워크 설정. 자주 사용되는 방법 중 하나는 *overlay network*를 구성하는 것. [Docker Guides의 Use overlay networks](https://docs.docker.com/network/overlay/) 함께 참고.

> The `overlay` network driver creates a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it (including swarm service containers) to communicate securely. Docker transparently handles routing of each packet to and from the correct Docker daemon host and the correct destination container.

그 외에도 컨테이너의 *control plane software*(k8s, Mesos, Docker Swarm, ...), 설정 외부화(민감 정보, 호스트네임, 포트 등), 빠른 시작 시간(총 시작 시간이 1초가 되는 것을 목표로 하라고 함), 컨테이너 안에서 동작하는 디버깅의 어려움 등을 함께 언급.

### VIRTUAL MACHINES IN THE CLOUD

> Any individual virtual machine in the cloud has worse availability than any individual physical machine.

클라우드의 (데이터센터에 비해 상대적으로) 낮은 가용성, 자주 바뀌는 장비 ID와 IP 주소, 클러스터에 인스턴스를 투입하는 것이 컨트롤러에 의한 것이 아닌 인스턴스의 자원<sup>volunteer</sup>에 의해 이뤄져야 한다는 것, 기본적으로 제공되는 네트워크 인터페이스가 너무 간소화 되어 있다는 것(1개의 NIC와 비공개 IP 주소) 등을 언급.

### CONTAINERS IN THE CLOUD

클라우드 VM에 컨테이너를 올리는 것은 클라우드와 VM의 어려움을 함께 맞이하게 되는 것.

# Processes on Machines

이전 장에서는 소프트웨어가 배포되는 환경(다양한 네트워크와 물리적 장비)에 대해 알아봤다면, 이번에는 개별 인스턴스에 대해 이야기.  좀 더 구체적으로는, 코드, 설정, 연결<sup>connection</sup>에 대한 내용. 그 전에 몇 가지 용어 정리.

1. **Service** 일련의 기능을 제공하기 위해 협력하는 프로세스들의 집합. 서로 다른 *executable*로부터의 *process*로 구성될 수 있음. 예컨대, 애플리케이션 코드 + 데이터베이스.
2. **Instance** 단일 장비 위의 *installation*. *service*와 다르게, 같은 *execuatble*로부터의 *process*.
3. **Executable** 장비 위에서 *process*로 실행될 수 있는 artifact를 가리킴. 컴파일 언어에서는 바이너리, 인터프리터 언어에서는 소스.
4. **Process** 장비 위에서 실행되는 OS 프로세스. 
5. **Installation** 장비 위에 존재하는 *executable* + 부수적인 디렉토리 + 설정 파일 + 기타 리소스.
6. **Deployment** 장비 위에 *installation*을 만드는 행위.

별 것 아닌 것 같지만 이 정의는 중요하다고 함. "서버를 재시작 해"라고 했을 때, 우리는 무엇을 해야 한다는 얘기일까? 단어에 대해 서로 다르게 이해한다면, 단일 프로세스를 죽이라는 것인지, 혹은 전제 장비를 말하는 것인지 확신하기 어려움.

다시 돌아와서, 이제부터 인스턴스가 필요로 하는 코드, 설정, 커넥션에 대해 알아볼 예정.