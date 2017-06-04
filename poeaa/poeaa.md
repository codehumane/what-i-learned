# 책소개

- [엔터프라이즈 애플리케이션 아키텍처 패턴](http://wikibook.co.kr/peaa/)
- 마틴 파울러 지음
- 너무 많은 책에서 인용되고 있어 거부할 수 없었음. 오랜만에 다시 정독 중.

# 계층화

## 계층화의 의미

먼저, 계층화란 무엇일까?

- 복잡한 소프트웨어 시스템을 분할하는 일반적인 기법.
- 컴퓨터 아키텍처의 경우 프로그래밍 언어로 운영체제를 호출하면, 다음의 계층으로 전달됨.
  - 장치 드라이버, CPU 명령어 집합, 칩 내의 논리 게이트

계층화는 아래의 규칙을 따른다.

- 하위 계층은 상위 계층을 보지 못하며, 계층을 건너뛰지 못함.
- 하지만 최근에는 계층의 역전만을 경계할 뿐임.

## 장단점

이것이 가져다 주는 이점은,

- 다른 계층 없이도 단일 계층을 하나의 일관된 계층으로 이해
  - 이더넷 몰라도 TCP 기반의 FTP 서비스 구축 가능함.
  - AWS의 S3의 경우도 SDK를 이용하면 모든 것을 구현하지 않고 사용이 가능함.
- 동일한 기본 서비스를 가진 대안 구현으로 계층 대체 가능
- 계층간의 의존성 최소화
- 계층은 표준화하기 좋은 위치
- 구축된 계층은 여러 다른 상위 서비스에서 재사용 가능

단점도 존재한다.

- 전체가 효과적으로 캡슐화 되는 것은 아님. UI에 표시될 데이터가 추가되야 하는 필드는 데이터베이스에도 영향을 미치므로, 중간의 모든 계층에 영향을 주게 된다.
- 성능이 저하됨. 계층간의 전송 데이터는 서로 변환이 필요하기 때문이다.

책의 내용으로는 다소 부족한듯 싶어, Martin Folwer 아저씨의 글, [PresentationDomainDataLayering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)을 찾아보았고 도움이 많이 되어 함께 기록한다. 글의 제목만 보면 책 내용과 다를 것 같지만, 책에서 실제로 다루는 계층은 `프레젠테이션-도메인-데이터`이다. 엔터프라이즈 아키텍처 패턴에서의 계층화는 어쩌면 더 잘 설명하고 있다.

여기서 설명하는 가장 큰 계층화의 이점은 한 번에 한가지만 신경쓸 수 있게 해준다는 점이다.

> It's biggest advantage (for me) is that it allows me to reduce the scope of my attention by allowing me to think about the three topics relatively independently. (중략) I narrow the scope of my thinking in each piece, which makes it easier for me to follow what I need to do.

집중의 범위를 줄여주어 3가지 계층을 (상대적으로) 독립적으로 생각하게 해준다는 것이다. 또한, 아래의 인용처럼, 변경사항이 하나의 레이어로만 제한되는 것이 아님에도(오히려 같이 변경되는 것을 일반적이라고 받아들인다), 계층화된 것이 변화를 더 쉽게 만들어준다고 한다.

> but when refining the UX I need to change the domain which necessitates a change to the data layer. But even with that kind of cross-layer iteration, I find it easier to focus on one layer at a time as I make changes.

그 이외에도 몇 가지 장점을 이야기하고 있다. 그 중 하나는, 구현 모듈을 다른 것으로 대체할 수 있다는 점이다. 이는 여러 개의 프리젠테이션에서 특정 구현을 반복하는 대신, 도메인 로직을 재사용할 수 있도록 도와준다.

> Another reason to modularize is to allow me to substitute different implementations of modules. This separation allows me to build multiple presentations on top of the same domain logic without duplicating it.

또 한가지는 테스트 용이성이다. UI 코드는 종종 테스트하기 어려우며, 따라서 가능한 많은 로직을 테스트가 용이한 도메인 레이어에 두기도 하고, 데이터 접근이 필요한 곳에는 [TestDoubles](https://martinfowler.com/bliki/TestDouble.html)를 두어 좀 더 쉽고 반응성 있는 테스트를 할 수 있다.

> Modularity also supports testability, (중략) UI code is often tricky to test, so it's good to get as much logic as you can into a domain layer which is easily tested without having to do gymnastics to access the program through a UI [1]. Data access is often slow and awkward, so using TestDoubles around the data layer often makes 
> domain logic testing much easier and responsive.

하지만 무엇보다 줄어든 집중의 범위가 가져오는 이점을 다시 한 번 강조하고 있다.

## 세 가지 주요 계층

책에서 주로 다루는 세개의 계층은 프레젠테이션, 도메인, 데이터원본이다. 사실 이들이 지금에 와서 그렇게 큰 의미를 가지지는 않는다고 생각한다. 지금은 더 많은 발전을 이뤘기 때문이다. 하지만 후반의 내용들을 이해하기 위해, 같이 스터디를 하는 사람들과의 커뮤니케이션을 위해 한번 정리가 필요하다고 생각한다. 각각을 하나씩 살펴보자. 

### 프레젠테이션Presentation

>“사용자와 소프트웨어 간 상호작용을 처리한다. … 리치 클라이언트 그래픽 UI 또는 HTML 기반 브라우저 UI인 경우가 많다. 프레젠테이션 계층의 주 역할은 사용자에게 정보를 표시하고 사용자가 내린 명령을 도메인과 데이터 원본에서 수행할 작업으로 해석하는 것이다.”

### 데이터 원본<sup>Data Source</sup>

Source를 왜 원본이라고 번역했을까? 여기서 말하는 데이터 원본의 대표주자는 다름 아닌 데이터베이스이다. 데이터 소스라고 그대로 번역했어야 했다. 이어지는 책 내용에서 이 용어를 만나면 재빨리 데이터 소스라고 받아들이자. 실제로 기록은 전부 '소스'를 사용했다. 어쨋든 정의는 다음과 같다.

> “애플리케이션을 대신해 다른 시스템과 통신한다. 여기서 다른 시스템은 트랜잭션 모니터, 다른 애플리케이션, 메시징 시스템 등일 수 있다. 대부분의 엔터프라이즈 애플리케이션에서 가장 큰 데이터 소스 논리는 지속성 데이터를 저장하는 데이터베이스다.”

### 도메인 논리<sup>domain logic</sup>

비즈니스 논리라고 불리기도 한다.

> “애플리케이션이 수행해야 하는 도메인과 관련된 작업이다. 이러한 작업에는 입력과 저장된 데이터를 바탕으로 하는 계산, 프레젠테이션에서 받은 데이터의 유효성 검사, 프레젠테이션에서 받은 명령을 기준으로 작업 대상이 될 데이터 소스 논리를 결정하는 등의 작업이 포함된다.”

### 각 계층의 조합

그리고 한가지 생각해볼만한 부분이 있어 기록한다.

> "때로는 도메인 계층이 데이터 소스를 프레젠테이션으로부터 완전히 감추도록 계층이 구성된 경우도 있다. 그런데 이보다는 프레젠테이션이 데이터 저장소에 접근하는 경우가 더 많다. 이 방식은 순수하지는 않지만 실제로는 아주 잘 작동한다. 이 경우 프레젠테이션은 사용자의 명령을 해석하고 데이터 소스을 통해 데이터베이스에서 해당하는 데이터를 가져온 다음 도메인 논리에 데이터 처리를 맡긴 후 처리된 데이터를 사용자에게 제공한다."

DDD에 있어 어플리케이션 레이어와 유사하다고 느껴지며, 순수함(혹은 원칙이라고 표현할 수도 있겠다)과 효율성 사이의 고민이 인상깊은 부분이다.

### 프레젠테이션과 데이터 소스의 구분

프레젠테이션과 데이터 소스를 구분하는 차이점으로, 아래와 같은 이야기도 한다.

> 프레젠테이션은 복잡한 사용자 인터페이스 또는 간단한 원격 프로그램 인터페이스이든 관계없이 시스템이 다른 사람에게 제공하는 서비스에 대한 외부 인터페이스다. 반면 데이터 소스는 프로그램에 제공하는 서비스에 대한 외부 인터페이스이다.

데이터 소스를 인터페이스로 바라본다는 점이 기존에 생각치 못하던 부분이다. 분명 위에서 데이터 소스를 "어플리케이션을 대신해 다른 시스템과 통신"하는 것으로 정의했지만 잘 와닿지 않았는데, 다른 프로그램에 제공하는 또 하나의 외부 인터페이스라고 하니 단번에 와닿는다.

### 도메인 논리의 구분

도메인 논리를 다른 것과 구분하는 것이 어려운 경우가 많은데, 이 경우 완전히 다른 종류의 인터페이스를 추가를 상상해 본다고 한다.

> 웹 애플리케이션에 명령줄 인터페이스를 추가하는 것과 같이 근본적으로 다른 계층을 애플리케이션에 추가한다고 가정해보곤 한다. 계층을 추가하기 위해 복제해야 하는 기능이 있다면 도메인 논리가 프레젠테이션으로 유출됐다는 신호다.

"계층을 추가하기 위해 복제해야 하는 기능이 있다면" 이라는 부분을 통해 잘 알수 있는데, 개인적으로도 사용하는 방법이긴 하지만 다른 이에게 설명하기에는 무척 곤란한 방법이다. 같이 일하는 사람들에게 좀 더 명확한 정의나 방법을 알려주면, 계층이 좀 더 잘 나눠질 수 있을텐데 말이다.

## 계층이 실행될 위치 선택

계층이 실행될 위치를 선택하는 내용이라기보다, 실행될 코드를 어느 계층에서 관리하는 것이 좋을까에 대한 내용이다. 아래 문장이 핵심을 잘 표현한다고 생각한다.

> 서버에서 실행하는 것이 유지 관리를 생각할 때 최선의 방법이다. 비즈니스 논리를 클라이언트로 옮기는 이유는 응답성을 개선하거나 비연결 작업을 지원하기 위해서다.

여기서 '비즈니스 논리'라고 하는 부분은 '모든 것'이라고 바꿔도 문제 없다.

# 도메인 논리 구성

## 소개

도메인 논리를 저장하는 3가지 방식으로, 트랜잭션 스크립트, 도메인 모듈, 테이블 모듈을 소개하고 있다.

먼저 트랜잭션 스크립트란, 

- 프레젠테이션에서 받은 입력에 대한 유효성 검사와 계산을 수행하고,
- DB에 데이터를 저장하고, 
- 다른 시스템을 호출하는 프로시저를 가리킴.
- 이해하기 쉬운 반면 도메인 논리가 늘어나면서 복잡도가 함께 상승함.

도메인 모델이란,

- 한 루틴이 사용자 작업의 논리를 모두 담당하는 대신,
- 여러 객체가 역할을 나누어 처리하는 것을 가리킴.
- 이는 복잡한 논리를 체계적으로 관리할 수 있도록 해줌.
- 요구사항이 추가되면 전략 객체를 새로 추가하는 방식 등으로 말이다.

마지막으로 테이블 모듈이란,

- 객체가 역할을 나눠가지긴 하지만, 도메인 논리(레코드 단위가 객체가 되는)와 다르게 테이블 단위가 객체가 됨.
- 아키텍처의 나머지 부분과 잘 맞는다는 장점이 있지만,
- 상속이나 전략, 그 외 다른 객체지향 패턴을 사용하기 어려움.

## 선택

- 트랜잭션 스크립트, 테이블 모듈, 도메인 모듈로 갈수록 복잡한 도메인에 대응하기 쉬워짐.
- 필자의 경우 개발팀 수준이 높으면 도메인 모델을 선호하지만,
- 이 세가지 패턴들은 서로 상호배타적 관계는 아니라고 함.
- 일부 도메인에서는 트랜잭션 스크립트를 사용하고, 다른 곳에서는 테이블 모듈이나 도메인 모듈을 쓰기도 함.

## 서비스 계층

- 도메인 모듈이나 테이블 모듈의 경우, 도메인 논리를 처리할 때 일반적으로 도메인 계층을 둘로 나눔.
- 이 때 나오는 것이 서비스계층이며, 트랜잭션 제어나 보안 등의 기능을 담당하고, 도메인 모델이나 테이블 모듈 위에서 동작함.
- 서비스 계층에 얼마나 많은 동작을 넣을지에 따라, 서비스 계층을 단지 파사드로만 바라볼수도 있음.
- 필자의 경우 일반적으로는 서비스 계층을 사용하지 않고, 꼭 필요하다면 간소화된 서비스 계층 즉, 파사드로 사용한다고 함.
- 물론 그렇다고 해서 비즈니스 논리가 서비스 계층에 많이 담기는 것을 반대하지는 않음.

# 관계형 데이터베이스 매핑

## 아키텍처 패턴

효과적인 SQL쿼리와 명령을 정의하는 데 어려움을 겪는 개발자가 많기 때문에, SQL 접근을 도메인 논리와는 별도로 분리하고 개별 클래스에 배치하는 것이 좋다고 이야기한다. 이 경우 테이블당 클래스 하나(테이블에 대한 게이트웨이가 됨)를 가지기를 권장하고 있으며, 이를 위한 방법은 또다시 행 데이터 게이트웨이와 테이블 데이터 게이트웨이로 나뉜다. 하지만 뒤에서는 게이트웨이의 사용보다, 엑티브레코드(도메인 간단한 경우)나 데이터매퍼(도메인 복잡한 경우)를 사용하길 권장한다.

각 패턴과 관련된 글은 아래를 참고

- [Row Data Gateway](https://martinfowler.com/eaaCatalog/rowDataGateway.html)
- [Table Data Gateway](https://martinfowler.com/eaaCatalog/tableDataGateway.html)
- [Active Record](https://martinfowler.com/eaaCatalog/activeRecord.html)
- [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html)

### 행 데이터 게이트웨이<sup>Row Data Gateway</sup>

> An object that acts as a Gateway (466) to a single record in a data source. There is one instance per row.

- 쿼리가 반환하는 각 행마다 인스턴스 하나를 만듬
- 객체지향적 사고방식에 잘 어울림
- 도메인 모델에서 사용

| Person Row Data Gateway
| --------------
| lastname
| firstname
| numberOfDependents
| insert
| update
| delete
| findForCompany(companyId)

### 테이블 데이터 게이트웨이<sup>Table Data Gateway</sup>

> An object that acts as a Gateway (466) to a database table. One instance handles all the rows in the table.

- 각 테이블마다 클래스가 하나 존재함
- 데이터베이스를 쿼리하고 레코드 집합을 반환
- 레코드 집합<sup>Record Set</sup>이란, 데이터베이스의 테이블식 특성을 흉내 낸 테이블과 행의 범용 자료구조
- 다양한 환경에서 폭넓게 지원함
- 테이블 모듈에서 사용

| Person Table Data Gateway
| --------------
| find (id) : RecordSet
| findWithLastName(String) : RecordSet
| update (id, lastname, firstname, numberOfDependents)
| delete (id)

### 엑티브 레코드<sup>Active Record</sup>

> An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data.

- 간단한 애플리케이션에서의 도메인 모델은 엑티브 레코드로 표현할 수 있음
- 엑티브 레코드는 테이블에 대응하는 도메인 객체가 직접 데이터베이스 로드 및 저장을 수행
- 하지만, 도메인 논리가 복잡해질수록 활용이 어려워짐
- 도메인 논리를 작은 클래스로 리팩토링하기 시작하면 클래스:테이블 1:1 대응이 깨지기 시작
- 관계형 데이터베이스는 상속 미지원이므로, 다양한 전략과 객체 지향 패턴 적용 어려움

| Person Active Record
| --------------
| load(ResultSet)
| findWithLastName(String) : RecordSet
| delete
| insert
| update
| checkCredit
| sendBills

### 데이터 매퍼<sup>Data Mapper</sup>

> A layer of Mappers (473) that moves data between objects and a database while keeping them independent of each other and the mapper itself.

- 도메인 모델이 복잡해질수록 간접성을 원하게 됨
- 데이터 매퍼란, 도메인 객체와 데이터베이스 테이블 간의 매핑을 간접 계층을 통해 처리하는 것
- 복잡하지만 두 계층의 완전한 분리라는 이점을 제공함

## 동작 문제

ORM 사용시 주목해야할 부분은 객체와 테이블이 연관되는 방법이 아니라, ORM의 아키텍처와 동작 측면이라고 한다. 어렵기 때문이다. 여기서 말하는 동작 문제는 다음의 2가지이다.

- 다수의 객체를 메모리에 로드시, 수정한 객체를 데이터베이스에 제대로 반영하기 위해 추적이 필요함
- 객체를 읽고 수정하는 동안 데이터베이스의 상태를 일관되게 유지. 객체를 읽어서 작업할 때는 다른 프로세스에 의해 수정되지 않도록 격리해야 하는 동시성 문제가 발생함

작업단위<sup>Unit of Work</sup>는 이를 해결하기 위해 꼭 필요한 패턴이라고 하는데, [UnitOfWork](https://martinfowler.com/eaaCatalog/unitOfWork.html)에 따른 작업단위의 설명은 다음과 같다. 요컨대, 읽어들인 객체를 추적하여 데이터베이스에 잘 반영되도록 하는 것이다.

> Maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems. (중략) A Unit of Work keeps track of everything you do during a business transaction that can affect the database. When you're done, it figures out everything that needs to be done to alter the database as a result of your work.

한편, 같은 객체를 두 번 로드하지 않도록 주의해야 한다. 둘 이상의 객체를 추적하는 것이 어렵기 때문인데, 이를 위한 한가지 방법은 [식별자 맵<sup>Identity Map</sup>](https://martinfowler.com/eaaCatalog/identityMap.html)이다. JPA에서의 first level of cache가 이에 해당되는 것으로 보인다. 추적을 용이하게 하는 것도 목적이지만, 불필요한 데이터베이스 호출을 줄이는 효과도 있다. JPA를 쓸때는 개인적으로 후자의 효과를 위해 사용되는 메커니즘으로만 알았다.

마지막으로 [지연로드<sup>Lazy Load</sup>](https://martinfowler.com/eaaCatalog/lazyLoad.html)에 대해서 언급하고 있다. 데이터베이스로부터 객체 로드시 연관된 객체가 모두 함께 로드될때, 막대한 규모의 객체 그래프가 만들어지는 것을 차단하기 위한 방법이다.

## 데이터 읽기

SQL select 문을 메서드 구조의 인터페이스로 래핑하는 것을 여기서는 검색기<sup>finder</sup> 메서드라고 하고 있다. 이 검색기를 넣을 위치는 다양한데,

- 테이블 기반 클래스의 경우 해당 클래스에 검색기 메서드 결합할 수 있으나, 행 기반 클래스의 경우는 결합 불가.
- 행 기반인 경우 검색기 메서드를 정적으로 만들수도 있긴 하지만, 테스트시 데이터베이스를 [서비스 스텁](https://www.martinfowler.com/eaaCatalog/serviceStub.html)으로 대체하기 어려움.
- 이 경우 검색기 객체를 별도로 만들기를 권장함.

그 외에 다음과 같은 성능 이슈들도 언급하고 있다.

- 가급적 여러 행을 한 번에 읽는다. 특히 같은 테이블에서 여러 행을 읽는 경우.
- 비관적 동시성 제어 사용시에는 행이 너무 많이 잠기지 않도록 주의.
- 조인을 사용해 여러 테이블을 한 번에 가져오는 것도 하나의 성능 개선 방법.
- 다만, 조인 사용시 데이터베이스가 쿼리당 최대 3~4개의 조인에 최적화 되어 있음에 주의.
- 그 외에도 데이터 클러스터링, 세심한 인덱스 사용, 메모리 캐시 활용등이 있음.
- 마지막으로 반드시 애플리케이션 프로파일링 할 것을 강조함.

## 구조적 매핑 패턴

### 관계 매핑

객체와 관계형 데이터베이스가 연결을 처리하는 방법에는 2가지 차이점이 존재한다. 하나는 참조 방법의 차이, 다른 하나는 컬렉션을 처리하는 방법의 차이이다.

- 참조 방법의 차이 문제
    - 객체는 참조를 저장하는 방법으로 연결을 처리하는 반면,
    - 관계형 데이터베이스는 다른 테이블에 대한 키를 생성해 연결을 처리함.
- 컬렉션을 처리하는 방법의 차이 문제
    - 객체는 컬렉션을 사용해 단일 필드로 여러 참조를 처리하는 반면,
    - 데이터베이스는 모든 연관 링크가 단일 값을 가져야함.
    - 즉, 컬렉션을 소유하는 대신 컬랙션의 각 요소가 소유자를 외부키를 통해 참조해야 함.

이들의 차이를 극복하기 위한 방법도 제시하고 있다.

- 참조 방법의 차이 문제
    - 기본적으로 [식별자 필드<sup>Identity Field</sup>](https://martinfowler.com/eaaCatalog/identityField.html)를 사용함.
    - 테이블에서 외래키가 나올 경우는 [외래 키 매핑<sup>Foreign Key Mapping</sup>](https://martinfowler.com/eaaCatalog/foreignKeyMapping.html)을 사용해서 객체 간 참조를 적절히 구성.
    - 기본적으로 식별자 맵에 키가 있는지 확인하고 없으면, 데이터베이스에서 가져오거나 지연 로드를 사용함.
- 컬렉션을 처리하는 방법의 차이 문제
    - 객체에 컬렉션이 있는 경우 원본 객체의 ID와 연결된 모든 행을 찾기 위해 다른 쿼리를 수행(혹은 지연 로드)해야 함.
    - 컬렉션 저장시에는 컬렉션 객체 저장하고 원본 객체의 외래 키를 넣는 과정을 거침.
    - 이 과정은 컬렉션에 추가되거나 제거된 객체를 감지하기 어렵게 만듬.
    - 컬렉션 객체가 컬렉션 소유자 범위 바깥에서 사용되지 않았다면 [의존 매핑<sup>Dependent Mapping</sup>](https://martinfowler.com/eaaCatalog/dependentMapping.html)을 활용할 수 있음

그 외에도 여러 가지 문제와 해결책을 설명하고 있다.

- 다대다 관계를 처리할 때는 [연관 테이블 매핑<sup>Association Table Mapping</sup>](https://martinfowler.com/eaaCatalog/associationTableMapping.html) 사용.
- 데이터베이스에 컬렉션 저장시 순서 유지 어려움. 객체에서 컬렉션 저장시 순서 없는 집합 사용을 고려하거나, 컬렉션 쿼리시 마다 정렬 순서를 지정(하지만 성능 저하 가능성 존재)
- 참조 무결성이 업데이트를 복잡하게 하는 문제는, 무결성 검사를 트랜잭션의 끝으로 연기하거나, 데이터베이스가 모든 쓰기 작업을 검사하거나, 업데이트를 위상 정렬<sup>topological sort</sup>하거나, 테이블 기록 순서를 코드에 직접 명시할 수 있다.
- [값 객체<sup>Value Object</sup>](https://martinfowler.com/eaaCatalog/valueObject.html)는 외래 키와 식별자 필드를 활용하기 보다, 연결된 객체의 [포함 값<sup>Embedded Value</sup>](https://martinfowler.com/eaaCatalog/embeddedValue.html)으로 넣는다. 또한 값 객체는 식별자 맵을 사용할 필요도 없다.
- 객체 클러스터가 규모가 큰 경우 테이블의 한 열에 [직렬화 LOB<sup>Serialized LOB</sup>](https://martinfowler.com/eaaCatalog/serializedLOB.html)으로 저장할 수도 있다. 다만 저장된 구조를 부분적으로 쿼리할 수 없음에 유의(XML로 저장하고 XPath 쿼리 식을 사용할수 있으나 대부분 비표준으로 취급됨)

내용에서 언급된 각각의 패턴들 설명은 아래와 같다.

| 패턴        | 설명                                       |
| --------- | ---------------------------------------- |
| 식별자 필드    | Saves a database ID field in an object to maintain identity between an in-memory object and a database row. |
| 외래 키 매핑   | Maps an association between objects to a foreign key reference between tables. |
| 의존 매핑     | Has one class perform the database mapping for a child class. |
| 연관 테이블 매핑 | Saves an association as a table with foreign keys to the tables that are linked by the association. |
| 값 객체      | A small simple object, like money or a date range, whose equality isn't based on identity. |
| 포함 값      | Maps an object into several fields of another object's table. |
| 직렬화 LOB   | Saves a graph of objects by serializing them into a single large object (LOB), which it stores in a database field. |

### 상속

구조적 매핑 중 관계 매핑과 관련된 문제를 책에서는 '부품 트리와 같은 구성적<sup>compositional</sup> 계층'의 문제라고 표현하고 있다. 이러한 구성적 문제 이외에도 상속으로 연결된 클래스들을 데이터베이스로 매핑해주는 문제가 남아있다. 이를 처리하는 방법으로 3가지가 소개됨.

[클래스 테이블 상속<sup>Class Table Inheritance</sup>](https://martinfowler.com/eaaCatalog/classTableInheritance.html)

> Represents an inheritance hierarchy of classes with one table for each class.

- 클래스와 테이블이 각각 그대로 대응된다는 점에서 단순함
- 일반적으로 성능이 낮음 (단일 객체 로드를 위해 여러 번 조인 수행해야 함)

[구현 테이블 상속<sup>Concrete Table Inheritance</sup>](https://martinfowler.com/eaaCatalog/concreteTableInheritance.html)

> Represents an inheritance hierarchy of classes with one table per concrete class in the hierarchy.

- 조인 없이 한 객체를 한 테이블에서 가져올 수 있음
- 상위 클래스 테이블에서의 잠금 경합 감소
- 변경에 취약함
    - 상위 클래스가 변경되면 모든 테이블과 매핑 코드를 함께 변경해야 함
    - 계층 자체를 변경할 경우 더 많은 부분을 변경해야 함
    - 상위 클래스 테이블이 없기 때문에 키 관리가 불편(?)하고 참조 무결성을 유지하기도 쉽지 않음

[단일 테이블 상속<sup>Single Table Inheritance](https://martinfowler.com/eaaCatalog/singleTableInheritance.html)

> Represents an inheritance hierarchy of classes as a single table that has columns for all the fields of the various classes.

- 모든 정보를 한 곳에서 관리하므로 수정이 쉽고 조인 없어도 됨
- 빈 공간이 많이 발생함
    - 하위 클래스를 위한 필드가 상위 클래스에서는 쓰이지 않을 수 있으므로
    - 그러나 대부분의 데이터베이스가 낭비된 테이블 공간을 잘 압축해줌
- 크기 때문에 접근 시 병목현상 발생할 수 있음

## 매핑

한 번 읽기는 했지만 기록할만한 가치는 없다고 판단하여 생략함.

## 메타데이터 사용

- [메타데이터 매핑<sup>Metadata Mapping</sup>](https://www.martinfowler.com/eaaCatalog/metadataMapping.html)이란, 데이터베이스의 열이 객체의 필드에 매핑되는 구체적인 방법을 메타데이터 파일에 기록하는 것
- [P of EAA](https://www.martinfowler.com/eaaCatalog/index.html)에서는 단순히 ORM의 세부사항을 메타데이터로 보관하는 것이라고 설명하고 있음
- 메타데이터를 일단 만든 후에는 코드 생성이나 리플렉션을 통해 반복적 코드를 대신할 수 있음
- 책에 나오진 않았지만, 관례를 이용해 매핑을 위한 단순박복 코드를 줄일수 있기도 하다.
- [쿼리 객체<sup>Query Object</sup>](https://www.martinfowler.com/eaaCatalog/queryObject.html)는 이러한 메타데이터를 이용해 SQL을 만들어낸다.
- 이를 좀 더 발전시켜 [리포지토리<sup>Repository</sup>](https://www.martinfowler.com/eaaCatalog/repository.html)를 사용할 수도 있음
- 아래는 P of EAA의 정의이며, 책에서의 2부 코드 예제를 보면 좀 더 명확하게 이해할 수 있음
- 개인적으로는 쿼리 객체와 메타데이터 매핑의 추상화, 그리고 Criteria 같은 선언적 특징을 핵심이라고 생각함 (두 번째가 다소 의문)

> Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.

## 데이터베이스 연결

- 쿼리가 반환하는 [레코드 집합<sup>Record Set</sup>](https://martinfowler.com/eaaCatalog/recordSet.html)을 연결된 레코드 집합과 끊긴 레코드 집합으로 나누고 있음
- 연결을 만드는 데 들어가는 비용을 줄이기 위한 연결 풀 이야기도 언급
- 연결 풀은 "열기"와 "닫기", "얻기"와 "반환하기"를 추상화함

# 웹 프레젠테이션

## 모델 뷰 컨트롤러

> Splits user interface interaction into three distinct roles.

- [모델 뷰 컨트롤러](https://martinfowler.com/eaaCatalog/modelViewController.html)는 사용자 인터페이스의 상호작용을 3가지 역할(뷰, 모델, 컨트롤러)로 나눈것
- 모델을 웹 프레젠테이션으로부터 완전히 분리하기 위함임
- 프레젠테이션의 수정 추가가 쉬워지고, 트랜잭션 모델 등을 테스트하기도 쉬워짐
- 다만, 여기서의 컨트롤러는 자주 오해되는 개념임.
- 혼동을 피하고자 입력 컨트롤러<sup>input controller</sup>라는 용어를 사용함
- 입력 컨트롤러의 역할은 아래와 같음
    - 요청을 받고, 요청에서 정보를 꺼냄
    - 비즈니스 논리를 적절한 모델 객체로 전달함 (Q. 비즈니스 논리를 전달한다고?)
    - 모델로부터 제어권을 넘겨 받으면서, 응답에 필요한 데이터도 함께 받음
    - 응답할 뷰를 결정하고 응답 데이터와 함께 제어권을 넘겨줌
- 한편, [어플리케이션 컨트롤러](https://martinfowler.com/eaaCatalog/applicationController.html)도 함께 소개됨
    - 2가지 역할 수행: 컨트롤러에 들어온 요청으로부터 실행할 도메인 논리 결정, 응답을 표시할 뷰 결정
    - 이 2가지 역할이 복잡해지고, 여러 곳에서 중복이 발생할 수 있음
    - 이 역할을 모아둔 곳이 어플리케이션 컨트롤러임
    - [Application Controller PoEAA 패턴 설명](https://martinfowler.com/eaaCatalog/applicationController.html)의 설명과 그림 참고

> A centralized point for handling screen navigation and the flow of an application.

## 뷰패턴

- [템플릿 뷰<sup>template view</sup>](https://martinfowler.com/eaaCatalog/templateView.html), [변환 뷰<sup>transform view</sup>](https://martinfowler.com/eaaCatalog/transformView.html), 1단계 뷰, [2단계 뷰<sup>two-step view</sup>](https://martinfowler.com/eaaCatalog/twoStepView.html)를 소개하고 있음

> A view that processes domain data element by element and transforms it into HTML.

- [변환뷰]()는 


# 동시성

이 장에서는 동시성 문제가 무엇인지, 그리고 아래의 2가지 문제는 어떻게 해결할 수 있는지 소개함.

1. 오프라인 동시성<sup>offline concurrency</sup>: 여러 데이터베이스 트랜잭션에 걸쳐 조작되는 데이터에 대한 동시성 제어를 가리킴
2. 어플리케이션 서버 동시성: 다중 스레드가 발생시키는 동시성 제어를 가리킴

## 동시성 문제

동시성의 핵심 문제는 다음의 2가지다.

1. 손실된 업데이트<sup>lost update</sup>
2. 일관성 없는 읽기<sup>inconsistent read</sup>

간단한 내용이므로 설명을 아래 그림으로 대체함.

![KakaoTalk_Photo_2017-05-30-18-31-42_97.jpeg](quiver-image-url/5409EEF64F46A0C97A556D99D000A0EA.jpg)

다만 위 2가지 문제의 발생 조건은 "둘 이상의 사용자가 동일한 데이터를 사용하려 하는 경우"이다. 만약, 데이터에 대한 동시 사용을 막으면 일관성(책에서는 정확성 또는 안정성이라고 표현)을 유지할 수 있다. 다만 동시성(책에서는 활동성<sup>liveness</sup>이라고 표현)을 포기해야 한다. 이와 반대로, 동시성을 보장하고 싶다면 일관성을 어느 정도 포기해야 한다. 결국 어플리케이션에서의 동시성 문제는, 동시성과 일관성의 균형을 어떻게 잘 유지할 것인가가 된다.

## 실행 컨텍스트

동시성 이해를 위해, 실행 컨텍스트라는 개념을 소개하고 있다. 먼저, 외부 세계와 상호작용할 때 발생하는 2가지 컨텍스트를 소개함.

1. 요청<sup>request</sup>: 소프트웨어가 작업하고 선택적으로 응답을 보내야 하는 외부 세계로부터의 단일 호출
2. 세션<sup>session</sup>: 클라이언트와 서버 간에 오랫동안 실행되는 상호작용

그 다음으로, 운영체제와 관련된 2가지 컨텍스트 소개.

1. 프로세스<sup>process</sup>: 사용하는 내부 데이터에 대한 다단계 격리를 제공하는 대규모 실행 컨텍스트
2. 스레드<sup>thread</sup>: 한 프로세스 내에서 여러 스레드로 작동할 수 있게 구성된 소규모의 활성 에이전트

스레드는 메모리를 공유하기 때문에 동시성 문제를 유발할 수 있음. 위에서 언급한 동시성 문제 발생 조건을 생각해보면 이해하기 쉬움. 물론 프로세스라고 하더라도 동일 DB를 사용하면 메모리를 공유하는 것처럼, 동시성 문제 발생 조건이 충족됨. 마지막으로 데이터베이스를 처리할 때의 컨텍스트인, 트랜잭션을 이야기함.

1. 시스템 트랜잭션
2. 비즈니스 트랜잭션

## 격리와 불변성

- 동시성 문제의 조건은 "둘 이상의 사용자가 동일한 데이터를 사용하려 하는 경우"라고 했음
- 만약 한번의 한 사용자만 동일 데이터에 접근할 수 있게하거나,
- 데이터를 수정할 수 없게한다면 문제를 피할 수 있음
- 전자는 격리, 후자는 불변성을 가리킴

Q. 동시성과 관련한 이슈들을 공유하면 도움될 듯

## 낙관적 동시성 제어와 비관적 동시성 제어

그러나 만약 격리할 수 없는 변경 가능한 데이터가 있다면 어떻게 해야 할까? 이럴때 아래 2가지 방법이 도움이 될 수 있다.

1. 낙관적 잠금<sup>optimistic locking</sup>: 충돌 감지에 해당함. 둘 이상의 사용자가 동시 수정은 가능하지만, 완료 시점에 변경 내용 충돌여부 감지해서 사용자에게 알림
2. 비관적 잠금<sup>pessimistic locking</sup>: 충돌 예방에 해당함. 이미 누군가 수정중인 데이터라면 다른 사용자의 편집을 불가능하게 만듬

일반적으로는 낙관적 잠금이 선호된다고 함. 그리고 비관적 잠금에 비해 사용자가 더 원활하게 작업할 수 있음. 수정이 일어난다고 하더라도 데이터를 읽을 수 있기 때문임. git 등의 VCS에서 머지하는 경우가 낙관적 잠금에 해당함. 만약, 충돌이 사용자에게 미치는 영향이 크고 중요하다면 다소 불편하더라도 비관적 잠금이 유리함. 충돌의 영향이 크지 않다면 더 나은 동시성을 제공하는 낙관적 잠금을 선택. 어쨋든 이 2가지 방법은 격리나 불변성 같이, 문제를 원천적으로 차단한 것이라기 보다는 어느 정도 타협된 해결책이다.

### 일관성 없는 읽기 예방

비관적 잠금의 읽기 잠금과 쓰기 잠금을 활용하여 해결할 수 있다.

- 데이터를 읽으려면 읽기 잠금이 필요하고,
- 데이터를 쓰려면 쓰기 잠금이 필요함
- 여러 사용자가 동시에 읽기 잠금을 가질 수 있지만,
- 읽기 잠금을 가진 사용자가 하나라도 있으면 누구도 쓰기 잠금을 가질 수 없음

낙관적 잠금의 충돌 감지 기능에 대해서도 설명한다.

- 타임스탬프나 순차 카운터 등의 버전 표식을 이용함
- 업데이트하려는 데이터의 버전과 공유 데이터의 버전을 비교하여 충돌 감지
- Q. 일관성 없는 읽기를 감지하는 것도 이와 비슷하다고 하는데, 구체적인 내용은 없어서 궁금함
- Q. 데이터를 읽고 있는 중에 수정이 발생하면, 읽고 있는 데이터에 변경사항이 있다고 표기를 해주는 걸까?

데이터를 제어 대상과 그렇지 않은 대상으로 구분하는 것이 중요함도 이야기한다.

- 모든 데이터를 제어하려고 할 경우 대기 시간이 너무 길어질 수 있기 때문임
- 잠금 방식을 무엇을 사용하느냐와 상관없이 이 작업은 중요함

임시 읽기<sup>temporal read</sup>도 일관성 없는 읽기 예방의 한 방법이다.

- 읽은 모든 데이터에 일종의 타임스탬프나 읽기 전용 라벨을 붙이고,
- DB는 이 시간이나 라벨을 기준으로 데이터를 반환함
- Q. 임시 읽기 자체에 대한 설명은 더 없어서 정확한 이해는 어렵다. 대략적인 추정만 가능할뿐. PostgreSQL의 MVCC가 이를 가리키는 게 아닐까?

### 교착 상태

먼저 교착상태<sup>deadlock</sup>에 대해 설명함

- 비관적 잠금에서 발생할 수 있는 상태
- (가) 사용자가 A를 수정하고 있고, (나) 사용자가 B를 수정하고 있음. (가)의 작업은 B도 수정해야 끝나는데, (나)가 B의 잠금을 획득한 상태이고, (나)는 A를 수정해야 작업을 완료할 수 있는데, (가)가 A에 대한 잠금을 획득하고 있음

그리고 나서 해결책을 언급

1. 교착 상태 발생시 희생자를 선정하고, 희생자의 잠금을 포기하도록 만듦 (감지 차원)
2. 모든 잠금에 제한 시간을 두어, 시간 초과시 희생자로 만듦 (감지 차원)
3. 작업 시작시 사용자가 필요로 하는 모든 잠금을 얻게함 (예방 차원)
4. 잠금에 대한 순서를 지정 (예방 차원)
5. 이미 획득중인 잠금을 다른 사용자가 얻으려고 하면 그 사용자를 희생자로 맘듦 (예방 차원)

보수적인 방법을 선호한다면 
