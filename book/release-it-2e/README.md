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

# Case Study: The Exception That Grounded an Airline

- 정기적인 DB 유지보수가 있었고,
- 이 때, A라는 DB로부터 B라는 DB로 페일오버가 예정됨.
- 서비스에 이상이 없는지를 확인할 수 있는 모니터링과 함께,
- 무중단 DB 페일오버를 수동으로 진행.
- 문제 없이 완료 후 작업은 모두 끝남.
- 하지만 몇 시간 후 항공사 시스템에 전면 장애가 발생.
- 장애 발생 시의 즉각적인 대응과 포스트 모텀 등을 이야기 한 뒤(이런 부분들이 잼있었음),
- 문제의 원인이 된 코드도 보여줌. CF라는 시스템은 아래의 코드만으로 이뤄진 애플리케이션이 실행되고 있었다고 함.

```java
public class FlightSearch implements SessionBean {

    private MonitoredDataSource connectionPool;

    public List lookupByCity(...) throws SQLException, RemoteException {
        Connection conn = null;
        Statement stmt = null;

        try {
            conn = connectionPool.getConnection();
            stmt = conn.createStatement();

            // ...
        } finally {
            if (stmt != null) {
                stmt.close();
            }
            if (conn != null) {
                conn.close();
            }
    }
}
```

- `stmt.close()`가 SQLException을 던지기 때문.
- 데이터베이스의 페일오버 시 연결이 끊겼고, 이 때 `stmt.close()`의 호출이 예외를 발생시켰으며, 이로 인해 커넥션이 종료되지 않고, 리소스가 누수되어서 발생한 문제.

# Stabilize Your System

1. 엔터프라이즈 소프트웨어는 냉소적이어야 함.
    - 나쁜 것이 일어날 것을 예상하고, 그것이 일어났을 때 놀라지 않는 것.
    - 스스로를 또한 믿지 않기 때문에 장애 시의 대비책도 마련.
    - 다른 시스템과 너무 밀접<sup>intimate</sup>해지는 것을 거부. 다치는 것을 염려하기 때문.
    - 앞선 항공사 사례에서의 예외는 충분히 회의적이지 않았음.
2. 낮은 안정성은 많은 실질적 비용을 초래. 수익은 물론, 평판도 마찬가지.
3. 좋은 안정성에 꼭 많은 비용이 들일 필요는 없음.
    - 아키텍처나 설계, 저수준 구현 시 많은 안정성을 위한 레버리지 포인트들이 존재.
    - 이 레버리지는 기능적 요구사항과 상충되는 길이 아니며,
    - 안정적이지 못한 것과 안정적인 것을 만드는데 들어가는 시간은 동일하다고.
    - (하지만 갑자기 자본론이 떠오름. 아쉬움)

## Defining Stability

대부분의 사람들은 "안정성<sup>stability</sup>"을 아래와 같이 정의한다고 함.

> 강건한 시스템은 일시적 충격<sup>impulse</sup>이나, 지속적인 스트레스<sup>stress</sup>, 컴포넌트의 실패에 상관 없이, 트랜잭션 처리를 지속함.

여기서의 트랜잭션과 시스템의 의미는 각각 다음과 같음.

- 트랜잭션: 시스템에 의해 처리되는 일의 단위를 추상화한 것. DB 트랜잭션 X. "customer places order"와 같이 다른 시스템(카드사 등)과의 통합이 포함될 수도.
- 시스템: 사용자의 트랜잭션을 처리하기 위한 하드웨어, 애플리케이션, 서비스의 상호 의존적이고 완전한 집합.

## Extending Your Life Span

1. 시스템 수명에 대한 주요 위험 요소는 메모리 누수와 데이터 성장이라고 함.
2. 이 둘은 모두 테스트 기간에는 잡히기 어려운 것들. (주로 긴 시간을 두고 운영할 때 생기는 문제이기 때문)
3. 테스트에 관해서는 머피의 법칙을 따르자. 테스트 안 한 것이 문제가 됨. 42시간이 지나야 생기는 메모리 누수, 자정이 지나야 생기는 충돌 문제 등.
4. 그런데 이런 장기적(?) 버그는 어떻게 찾을 수 있을까.
5. 개발 장비를 하나 마련해서 JMeter, Marathon 등으로 장기 버그를 테스트. 심한 부하를 주기 보다는, 적정한 양의 요청을 지속적으로 보내고, 이따금 일정 기간 동안에는 적은 양의 요청을 보내게 하라(커넥션 풀이나 방화벽 시간 제한 등의 문제를 잡을 수 있다고 함).
6. 돈이 문제가 된다면 적어도 중요한 부분만이라도 하라.
7. 이런 테스트가 없다면, 프로덕션 환경 = 장기 테스트 환경

## Failure Modes

1. 갑작스런 충격이나 과도한 긴장<sup>strain</sup>(스트레스가 가해진 모습을 가리킴)은 재앙적 실패<sup>catastrophic failure</sup>를 불러올 수 있음.
2. 일부 컴포넌트에 장애가 생기고, 이것에 의존하는 다른 컴포넌트에 급속도로 장애가 전파되는 것을 가리킴.
3. 다양한 고장 모드는 반드시 일어남. 충격을 수용하고 나머지 시스템을 보호하는 안전한 고장 모드를 만들어야 함.
4. 이런 종류의 자가 보호가 전체 시스템의 회복 탄력성<sup>resilience</sup>을 결정.
5. Chiles라는 사람은 이를 가리켜 "crackstoppers"라고 부름.

## Stopping Crack Propagation
