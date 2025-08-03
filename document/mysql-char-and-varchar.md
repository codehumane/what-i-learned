https://dev.mysql.com/doc/refman/9.3/en/char.html

# 13.3.2 The CHAR and VARCHAR Types

## 소개

- CHAR와 VARCHAR는 매우 유사.
- 하지만, 저장되고 조회되는 방식이 다름.
- 또한, 최대 길이도 다르고, 트레일링 스페이스를 포함하는지 여부도 다름.

## 길이

- 둘 다 최대 문자열 수를 나타내는 길이와 함께 정의됨.
- 예를 들어, `CHAR(30)`은 30개 문자를 담을 수 있음.

### CHAR 길이
- CHAR 컬럼은 고정 길이.
- 값은 0에서 255 중 하나.
- 저장 시에는 지정된 길이만큼 스페이스가 우측 패딩.
- 조회 시에는 트레일링 스페이스가 제거됨.
  - PAD_CHAR_TO_FULL_LENGTH 모드가 비활성화 상태에 한함.

```
mysql> CREATE TABLE vc (v VARCHAR(4), c CHAR(4));
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO vc VALUES ('ab  ', 'ab  ');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT CONCAT('(', v, ')'), CONCAT('(', c, ')') FROM vc;
+---------------------+---------------------+
| CONCAT('(', v, ')') | CONCAT('(', c, ')') |
+---------------------+---------------------+
| (ab  )              | (ab)                |
+---------------------+---------------------+
1 row in set (0.06 sec)
```

### VARCHAR 길이

- VARCHAR 컬럼은 가변 길이 문자열.
- 0에서 65,535 길이 중 하나를 지정.
- [단, 로우의 최대 크기도 65,535 바이트고 캐릭터 셋에 따라서 실제 최대 크기는 다를 수 있음](https://dev.mysql.com/doc/refman/9.3/en/column-count-limit.html).
- CHAR와 달리 VARCHAR는 1바이트나 2바이트의 길이 접두사가 함께 저장됨.
- 데이터의 바이트가 255를 넘지 않으면 접두사로 1바이트를, 255가 넘으면 2바이트 사용.
- strict SQL 모드가 비활성화 상태라면, 최대 길이를 넘는 데이터는 잘리게 되고 경고 메시지가 생성됨.
- strict SQL 모드가 활성화 되어 있다면, 최대 길이를 넘는 데이터에 대해 오류가 발생하고 저장은 실패함.
- 단, 최대 길이를 넘는 문자가 스페이스라면 SQL 모드에 상관 없이 경고와 함께 자동 제거됨.

| Value | CHAR(4) | Storage Required | VARCHAR(4) | Storage Required |
| -- | -- | -- | -- | -- |
| '' | '    ' | 4 bytes | '' | 1 byte |
| 'ab' | 'ab  ' | 4 bytes | 'ab' | 3 bytes |
| 'abcd' | 'abcd' | 4 bytes | 'abcd' | 5 bytes |
| 'abcdefgh' | 'abcd' | 4 bytes | 'abcd' | 5 bytes |

- 위 표는 접두사의 의미를 나타낸 것.
- 이 중에서 마지막 데이터만 strict SQL 모드 사용 중일 때 저장 실패하고 오류.
