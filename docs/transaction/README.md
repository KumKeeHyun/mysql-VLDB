# 전굽기

## MySQL
![image](https://user-images.githubusercontent.com/44857109/100318672-c4d00600-3001-11eb-89c1-d69155dc4cf2.png)

- MySQL Server : MySQL Engine + Storage Engine
    + MySQL Engine
        - SQL Interface
        - Parser
        - Optimizer
        - Cache/Buffer
    + Storage Engine
        - MyISAM
        - InnoDB(defualt)
        - Memory
        - 등등등

### Storage Engine
#### 1. MyISAM
- 가장 오래된 엔진
- 트랜잭션, 외래키 X
- 테이블락
- 풀텍스트 인덱스, R-Tree 인덱스 지원
#### 2. InnoDB
- 트랜잭션, 외래키 지원
- 레코드락
- MVCC(Multi-Version Concurrency Control) 지원
- 풀텍스트 인덱스(5.6), R-Tree 인덱스(5.7) 지원
#### 3. Memory
- in-momory 테이블
- 트랜잭션 안정성, 외래키 X
- 테이블락
#### 4. NDB Cluster
- Network Database Cluster
- 트랜잭션 지원
- 외래키 X
- 레코드락

## Lock
여러 개의 트랜잭션/스레드가 공유자원(데이터베이스/테이블/레코드)에 동시에 접근을 시도할 때 접근제한
1. Global Lock
- 특정한 명령이 수행되면 다른 모든 명령어가 수행이 정지
- mysqldump
2. Table Lock
- 특정 트랜잭션이 해당 테이블을 수정하고 있으면 다른 트랜잭션은 해당 테이블을 접근하지 못함
3. Record Lock
- 특정 트랜잭션이 해당 레코드를 수정하고 있으면 다른 트랜잭션은 해당 테이블을 접근하지 못함

# Transaction

## Usage
InnoDB Engine은 기본설정이 트랜잭션 미지원 상태이기 때문에 설정을 변경해야 함
```sql
mysql> set autocommit=0;
mysql> show variables like '%commit&';
```

- MySQL Shell
```sql
mysql> some queries 1 ...

mysql> rollback; //if error occured

mysql> some queries 2 ...

mysql> commit; //successful
```

```sql
mysql> some queries 1 ..

mysql> savepoint A;

mysql> some queries 2 ..

mysql> rollback to A;
```

- JDBC
```java
try {
    conn.setAutoCommit(false);
    // set Statement
    // execute
    conn.commit();
} catch (SQLException se) {
    conn.rollback();
} finally {
    conn.setAutoCommit(true);
}
```

## Atomicity
### Undo/Redo Log
- Undo Log
    - 실행 취소시 사용하는 로그
        - 트랜잭션의 롤백 지원
        - 트랜잭션의 격리수준 지원
        - MVCC 지원
    - 트랜잭션이 진행중일 때 생성/수정/삭제가 되면 기록됨
- Redo Log
    - 다시 실행이 필요한 경우 사용하는 로그
    - 주로 크래시 복원용


### Global Transaction(분산 트랜잭션)
- 2개 이상의 시스템간의 트랜잭션
- 일반적으로 원자성을 보장하기 위해 2PC(2단계 커밋) 알고리즘을 사용

## Isolation
여러 트랜잭션이 동시에 동일한 데이터에 접근할 때 적용하는 규칙
1. Read Uncommitted
1. Read Committed
1. Repeatable Read
1. Serializable

### Read Uncommitted
- commit 여부와 관계없이 현재 레코드값을 리턴
- 가장 낮은 데이터 안정성, 가장 높은 성능
- Dirty Read 발생
    - 커밋되지 않은 데이터를 읽는 경우
### Read Committed
- commit 된 마지막 레코드값을 리턴
- 일반적으로 가장 많이 사용하는 격리수준
- **Non-Repeatable Read** 발생
    - 동일한 트랜잭션 안에서 동일한 쿼리의 결과가 다른 경우
- Oracle 의 기본 격리수준

```
+------+-----+
| name | age |
+------+-----+
| sjha | 50  |
+------+-----+

mysql> update user set age=51 where name='sjha';

mysql> select age from user where name='sjha';
51 //Read Uncommitted
50 //Read Committed

mysql> commit;
```

### Repeatable Read
- 동일한 트랜잭션에서는 한 쿼리의 값은 항상 일정함
- **Phantom Read** 발생
    - 새로운 쿼리를 실행하면 이전에는 없던 데이터가 읽히는 경우
    - 데이터 **변경**은 불가능하지만 추가/삭제는 가능하기 때문에
- 백업이나 복제와 같이 상대적으로 긴 시간동안 동작하는 트랜잭션에서 안정적인 실행보장이 필요할 경우 사용 (Long Transaction)
- MySQL 의 기본 격리수준

```
// age : 50

    <Transaction 1>                 <Transcation 2>
$                               $ start backup(long transaction)
$ update age 50 -> 51, commit   $
$                               $ read age -> 50
$ update age 51 -> 52, commit   $
$                               $ read age -> 50
$                               $ end backup (backup result : 50)
$                               $ read age -> 52
```

### Serializable
- 두 개 이상의 트랜잭션이 동시에 수행되지 않음
- 가장 높은 데이터 안정성, 가장 낮은 성능

### Summary
|레벨|Dirty Read|Non-Repeatable Read|Phantom Read|
|:-:|:--------:|:------------------:|:---------:|
|Read Uncommitted|O|O|O|
|Read Committed|X|O|O|
|Repeatable Read|X|X|O|
|Serializable|X|X|X|

- Read Uncommitted vs. Read Committed
    - 현재 데이터값 vs. Undo Log 에서 마지막으로 커밋된 값
- Read Committed vs. Repeatable Read
    - Undo Log 에서 마지막으로 커밋된 값 vs. Undo Log 에서 커밋된 값들 중 해당 트랜잭션의 커밋값

### MVCC (Multi-Version Concurrency Control)
- 락 없이 읽기 성능을 증가시키는 기술
    - Lock-free Read Scalability
    - 두 개의 트랜잭션이 동일한 레코드를 읽으려 할 때 각 트랜잭션의 Isolation Level 이 다르다면 락이 필요없음
    - 동일한 격리수준이라도 Undo Log 를 동시에 여러개 복제해서 읽기 락이 발생될 확률을 낮춤
- 구현방식 Undo Segment vs. Pessimisitc Lock 기반의 MGA
    - Oracle, MySQL 은 Undo Segment 기반

## Consistency
트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 상태로 유지하는 것
- 만약 복제로 구성된 수만개의 DB가 있을 경우 일관성이 깨질 수 있음
    - 특정 서버에는 업데이트되고 특정 서버에는 업데이트가 안되어 있는 상황
    - 하지만 일시적인 현상이고 복제가 모두 이루어지면 다시 일관성을 유지
    - 일반적으로 RDBMS는 Large-scale 복제를 가정하지 않음

<br>

- cf. 무결성 : 데이터가 깨지는 상황
