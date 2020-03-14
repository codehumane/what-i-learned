# MySQL 퍼포먼스 최적화

## MySQL의 특징

### MySQL은 전체적으로 어떻게 생겼나?

[여기 문서](https://www.rathishkumar.in/2016/04/understanding-mysql-architecture.html)에 전체 구조 그림이 있어 가져옴.

![](https://3.bp.blogspot.com/-WllJC9xfxqg/VxTbuAoMm4I/AAAAAAAAHzA/1tF7Sxx8Y34levMmR9fYPZfDQDHVdWzKwCLcB/s1600/MySQL%2BArchitecture.png)

- 서버 엔진: 쿼리 파싱, 쿼리 캐시, 옵티마이저, 접근 제어, 테이블 조인, 그룹핑, 정렬, 트리거, 프로시저 처리 등.
- 스토리지 엔진: 물리적 저장장치에서 데이터를 읽어 옴.

### MySQL에서 스토리지 엔진이란 무엇인가?

[Introduction to InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-introduction.html) 함께 참고.

- InnoDB 스토리지 엔진 주로 사용됨.
- 유일하게 트랜잭션 지원.
- MVCC와 행 단위 잠금 지원.
- 인덱스와 데이터 모두 메모리에 올림.
- 기본키는 클러스터 인덱스로 구성(기본키 순서대로 디스크 저장).
- 보조인덱스는 기본키를 값으로 가짐.

### MySQL은 데이터를 어떻게 처리할까?

- 모든 SQL을 단일 코어에서 처리. scale out 보다 up이 유리.
- 테이블 조인을 'Block' Nested Loop Join으로 처리. 버퍼 이용한 O(n^3)
- [StackOverflow의 Join. vs sub-query](https://stackoverflow.com/questions/2577174/join-vs-sub-query)도 함께 보기.


