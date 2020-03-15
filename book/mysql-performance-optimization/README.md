# MySQL 퍼포먼스 최적화

책의 흐름을 따라가되, 기록은 주로 [dev.mysql.com](https://dev.mysql.com/) 내용을 기반으로 함.

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

## 쿼리 성능 진단은 최적화의 기초

### 쿼리 실행 계획에 대해서 알아보자

아래 2개 링크 함께 참고.

- [Optimizing Queries with EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/using-explain.html)
- [EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

링크 내용에서 몇 가지 기록.

- EXPLAIN은 MySQL이 어떻게 명령문을 수행할지에 대한 정보를 제공.
- 더 빨라지기 위해 어디에 인덱스를 추가해야 하는지를 파악할 수 있음.
- SELECT 명령에 사용된 테이블 정보들을 반환하며,
- MySQL이 명령문을 수행하기 위해 읽어들인 순서대로 테이블을 나열.
- nested-loop join을 사용한다는 얘기도 언급.

> This means that MySQL reads a row from the first table, and then finds a matching row in the second table, the third table, and so on.

- 책에서는 explain 결과를 읽는 순서 소개.
- 더불어, 실행 계획 각 항목(문서에서는 column으로 표현) 중 select_type, type, extra를 이야기.
- 여기서 type은 [EXPLAIN join Types](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types) 참고.
- [system](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_system) > [const](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) > [eq_ref](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref) > [ref](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref) > ... > [range](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_range) > [index](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_index) > [ALL](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_all)
- extra는 [Explain Extra Information](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-extra-information) 참고.
- "about how MySQL resolves the query"
- 그리고 성능이 괜찮은지를 보려면, using filesort와 using temporary가 있는지를 살펴보라고 함.
- 책에서도 `Using Filesort`, `Using Temporary` 상태라면 반드시 쿼리 튜닝을 권장.
- `Using Where`인 경우라고 하더라도 Type이 ALL 또는 Index이면 성능 좋지 않음.

### 쿼리 프로파일링을 이해하자

일단, [SHOW PROFILE Statement](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)을 보면, SHOW PROFILE이 deprecated이며, Performance Scheme를 사용하라고 하고 있음. 자세한 내용은 [Query Profiling Using Performance Schema](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-query-profiling.html)를 참고.

링크 내용의 예시로 어느 정도 설명이 가능. 먼저, 아래의 SQL을 수행했다고 가정.

```sql
mysql> SELECT * FROM employees.employees WHERE emp_no = 10001;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date |
+--------+------------+------------+-----------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
+--------+------------+------------+-----------+--------+------------+
```

위 SQL의 event_id가 무엇인지를 events_statements_history_long 테이블 질의를 통해 알 수 있음.

```sql
mysql> SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT
       FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like '%10001%';
+----------+----------+--------------------------------------------------------+
| event_id | duration | sql_text                                               |
+----------+----------+--------------------------------------------------------+
|       31 | 0.028310 | SELECT * FROM employees.employees WHERE emp_no = 10001 |
+----------+----------+--------------------------------------------------------+
```

그리고 해당 이벤트의 세부적 단계마다 소요된 시간 확인도 가능.

```sql
mysql> SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration
       FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=31;
+--------------------------------+----------+
| Stage                          | Duration |
+--------------------------------+----------+
| stage/sql/starting             | 0.000080 |
| stage/sql/checking permissions | 0.000005 |
| stage/sql/Opening tables       | 0.027759 |
| stage/sql/init                 | 0.000052 |
| stage/sql/System lock          | 0.000009 |
| stage/sql/optimizing           | 0.000006 |
| stage/sql/statistics           | 0.000082 |
| stage/sql/preparing            | 0.000008 |
| stage/sql/executing            | 0.000000 |
| stage/sql/Sending data         | 0.000017 |
| stage/sql/end                  | 0.000001 |
| stage/sql/query end            | 0.000004 |
| stage/sql/closing tables       | 0.000006 |
| stage/sql/freeing items        | 0.000272 |
| stage/sql/cleaning up          | 0.000001 |
+--------------------------------+----------+
```

## WHERE 조건 이해

> 책에 친절한 설명과 사례들이 담겨 있음.

### 묵시적 형변환 함정에 빠지지 말자

- 인덱스를 검색 조건으로 사용했어도, 묵시적 형변환에 의해 풀스캔이 일어날 수 있음.
- 따라서, 컬럼 타입에 맞는 조건절 값 사용을 강력히 권장.

### 편리한 함수, 잘못 쓰면 성능에 독이 된다

- WHERE 조건의 Left Value에 `DATE_FORMAT` 사용하는 사례를 들고 있음.
- DB 입장에서의 SQL 작성을 권장.

### LIKE 검색을 아무 때나 써야 하나?

- DB 자료가 인덱스 키 값 순서로 정렬되므로,
- LIKE '%문자열' 또는 LIKE '%문자열%'은 인덱스의 의미를 퇴색.
- '문자열%'이라고 하더라도, 데이터 selectivity가 10~15%(이 수치의 출처는 모르겠음)를 넘지 않아야.

## SQL 레벨에서의 접근법

> 4장도 MySQL 레퍼런스에 나오는 일반적인 이야기가 아니라서 키워드 위주로 기록.

### 데이터 흐름을 이해하자

- Temporary Table, Using filesort가 발생하더라도,
- 불필요한 데이터까지 불러들이지 않도록 함.
- 방법은 책 참고.
- 한편, 반드시 필요한 데이터만 조인 연산 대상으로 활용.
- 전자의 이야기가 불필요한 projection의 지양이었다면,
- 후자 이야기는 selection의 지양으로 보임.
- 사례는 책 참고.
- [MySQL 성능을 죽이는 잘못된 쿼리 습관](https://gywn.net/2012/05/mysql-bad-sql-type/)을 보는 것도 도움될 것.

### 불필요한 조인을 피하자

- 제목 그대로.

### Semi Join으로 인한 비효율을 제거하자

- 5.5 버전 이하일 경우에 한정된 얘기로 보임.
- [EXISTS vs IN - which one is better in MySQL 5.5 and MySQL 5.7?](https://stackoverflow.com/questions/48906772/exists-vs-in-which-one-is-better-in-mysql-5-5-and-mysql-5-7) 글 함께 참고.

### Outer Join이 반드시 필요한지 파악하자

- 무분별한 Outer Join은 성능 저하를 가져온다고 나옴.
- [Outer Join Simplification](https://dev.mysql.com/doc/refman/5.7/en/outer-join-simplification.html)에 아래 문구를 발견.

> When the optimizer evaluates plans for outer join operations, it takes into consideration only plans where, for each such operation, the outer tables are accessed before the inner tables. The optimizer choices are limited because only such plans enable outer joins to be executed using the nested-loop algorithm.

- 그래서 outer join의 대상을 미리 좁혀주는 것을 권장.
- Inner Join과 Outer Join 구분은 아래 그림 참고.

![SQL JOINS CHEATSHEET](https://camo.githubusercontent.com/26bcd184e10be85ee10d503f641706f630dd1f43/68747470733a2f2f662e636c6f75642e6769746875622e636f6d2f6173736574732f3333373831312f323336343536372f32303931376230612d613637652d313165332d386232312d6436653037313930376236352e706e67)

### 서브쿼리를 적극 활용하자

- 이것도 마찬가지로, 로드되는 데이터를 최소화하기 위한 노력.
- 목적이 중요. 당연히 무분별한 서브쿼리도 존재. 주의.

### 트랜잭션의 Isolation 레벨에서 테이블 잠금이 발생할 수 있음을 기억하자

- REPEATABLE-READ여도 아래 SQL은 테이블 잠금 유발 가능.
- `INSERT INTO SELECT..` 또는 `CREATE TABLE AS SELECT..`
- REPEATABLE-READ의 특성을 생각할 것.
