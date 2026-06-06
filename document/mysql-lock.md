
# MySQL Lock

MySQL 락 관련, 공식 문서의 몇 가지 내용들 모음

[Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html)의 몇 가지 내용 기록.

1. Typically, the clustered index is synonymous with the primary key.
2. Indexes other than the clustered index are known as secondary indexes.
3. In InnoDB, each record in a secondary index contains the primary key columns for the row, as well as the columns specified for the secondary index.
4. InnoDB uses this primary key value to search for the row in the clustered index.
5. If the primary key is long, the secondary indexes use more space, so it is advantageous to have a short primary key.

[InnoDB Locking](https://dev.mysql.com/doc/refman/9.7/en/innodb-locking.html)에 나온 supremum pseudo-record 설명.

1. For the last interval, the next-key lock locks the gap above the largest value in the index and the “supremum” pseudo-record having a value higher than any value actually in the index.
2. The supremum is not a real index record, so, in effect, this next-key lock locks only the gap following the largest index value.

이번에는 같은 문서 내 레코드/인덱스 락 이야기.

1. A record lock is a lock on an index record.
2. Record locks always lock index records, even if a table is defined with no indexes.

[Locks Set by Different SQL Statements in InnoDB](https://dev.mysql.com/doc/refman/8.4/en/innodb-locks-set.html)에는 아래의 락 내용도 나옴.

1. A locking read, an UPDATE, or a DELETE generally set record locks on every index record that is scanned in the processing of an SQL statement.
2. It does not matter whether there are WHERE conditions in the statement that would exclude the row.
3. InnoDB does not remember the exact WHERE condition, but only knows which index ranges were scanned.
4. The locks are normally next-key locks that also block inserts into the “gap” immediately before the record.

문서를 읽게 된 계기가 된 내용은 아래 문장임.

> If a secondary index is used in a search and the index record locks to be set are exclusive, InnoDB also retrieves the corresponding clustered index records and sets locks on them.

[Locking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)에도 같은 내용 나옴.

1. For index records the search encounters, locks the rows and any associated index entries, the same as if you issued an UPDATE statement for those rows.
2. Other transactions are blocked from updating those rows, from doing SELECT ... FOR SHARE, or from reading the data in certain transaction isolation levels.

[InnoDB Locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)에는 갭 락이 "purely inhibitive"라는 설명도 나와 있음. 이는 데드락이 발생하는 배경.

1. Gap locks in InnoDB are “purely inhibitive”, which means that their only purpose is to prevent other transactions from inserting to the gap. Gap locks can co-exist.
2. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap.
3. There is no difference between shared and exclusive gap locks.
4. They do not conflict with each other, and they perform the same function.

갭 락을 끌 수 있다는 얘기도 뒤이어 설명됨.

1. Gap locking can be disabled explicitly.
2. This occurs if you change the transaction isolation level to READ COMMITTED.
3. In this case, gap locking is disabled for searches and index scans and is used only for foreign-key constraint checking and duplicate-key checking.

[Deadlocks in InnoDB](https://dev.mysql.com/doc/refman/9.7/en/innodb-deadlocks.html)에서 설명하는, 데드락 발생 조건과 해결 방식.

1. A deadlock can occur when transactions lock rows in multiple tables (through statements such as UPDATE or SELECT ... FOR UPDATE), but in the opposite order.
2. A deadlock can also occur when such statements lock ranges of index records and gaps, with each transaction acquiring some locks but not others due to a timing issue.
3. To reduce the possibility of deadlocks, use transactions rather than LOCK TABLES statements;
4. keep transactions that insert or update data small enough that they do not stay open for long periods of time;
5. when different transactions update multiple tables or large ranges of rows, use the same order of operations (such as SELECT ... FOR UPDATE) in each transaction;
6. create indexes on the columns used in SELECT ... FOR UPDATE and UPDATE ... WHERE statements.
7. The possibility of deadlocks is not affected by the isolation level, because the isolation level changes the behavior of read operations, while deadlocks occur because of write operations.

[How to Minimize and Handle Deadlocks](https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks-handling.html) 내용도 함께 읽으면 좋음. 데드락이 그렇게 위험한 건 아니라는 얘기도.

1. Deadlocks are a classic problem in transactional databases, but they are not dangerous unless they are so frequent that you cannot run certain transactions at all.
2. Normally, you must write your applications so that they are always prepared to re-issue a transaction if it gets rolled back because of a deadlock.
