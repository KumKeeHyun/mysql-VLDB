# Replication
동일한 데이터베이스를 여러 개 생성
- High Availability(HA)
- Read Scalability
- Pseudo Real-time Backup 

## Master/Slave
- 복제 방향이 있음 (마스터 -> 슬레이브)
- 마스터에 쓰고 슬레이브에서 읽기
    - 보통 슬레이브는 읽기만 허용(read-only)
- 보통 마스터 1대에 슬레이브 8-10 대 이하로 구성
    - 그 이상은 릴레이 노드로 구성(multi-layer)

## Level
- Asynchronous
    - 슬레이브에 내용이 반영되었는지 확인하지 않음
    - 속도가 가장 빠름
    - 슬레이브의 랙(lag) 현상이 존재
- Semi-synchronous
    - 슬레이브 중에 하나는 반영되는지 확인
- Synchronous
    - 모든 슬레이브가 반영되는지 확인
    - 가장 느리지만 신뢰도는 높음

## MySQL Replication
MySQL 은 기본적으로 비동기 복제 방식으로 작동
- 5.5 부터 반동기 복제로 설정 가능
- 동기 복제는 지원하지 않음
    - Gallera Cluster(별도 회사의 제품)은 지원해줌

- - -

- Asynchronous
![image](https://user-images.githubusercontent.com/44857109/100519285-598d5c00-31da-11eb-844c-6ac0d75037f6.png)
- Semi-synchronous
![image](https://user-images.githubusercontent.com/44857109/100519356-b25cf480-31da-11eb-9651-e11b2a90e393.png)

- - -

### Binlog Format
- Statement-based
    - 마스터에 들어온 쿼리를 그대로 슬레이브로 복사
    - 동일한 쿼리를 실행해도 다른 결과가 나오는 경우가 있음
        - timestamp, random number, uuid...
- Row-based
    - 쿼리의 실행결과를 슬레이브로 보냄
    - 동일한 결과를 보장하지만 트래픽이 많아지는 단점이 있음
    - **권장**
        - 락처리에 유리
        - 5.7 부터 기본으로 변경
- Mixed
    - 기본은 Statement-based 를 사용하고, 상활에 따라 Row-based 를 사용하는 방식

#### Log File
- Error Log
- General Log
    - 성능저하로 보통 사용하지 않음
- Slow Query Log
    - 실행시간이 긴 쿼리들에 대한 로그
- **Binary Log**
    - 복제용(마스터)
- **Relay Log**
    - 복제용(슬레이브)
- Audit Log
    - 유료버전 전용

- - -

### Process
![image](https://user-images.githubusercontent.com/44857109/100520026-ff42ca00-31de-11eb-8555-aae583d065a7.png)
- Master Thread
    - Slave Thread 의 서버
        - Slave에서 접속요청을 하기 때문에 Master에는 로그인을 위한 계정과 권한이 필요함
    - Binary Log 를 읽어서 Slave 로 전송
    - Slave Thread 당 하나의 Master Thread 가 대응되어 생성
- Slave I/O Thread
    - Master 로부터 연속적으로 수신한 데이터를 Relay Log 에 순차적으로 기록
- Slave SQL Thread
    - Relay Log 에 기록된 변경 데이터 내용을 읽어 스토리지 엔진을 통해 Replay
    - Replication 처리의 병목 지점이 될 수 있음
        - 5.7 부터 SQL Thread 가 병렬로 데이터베이스 갱신(Multi Thread Slave)을 수행할 수 있도록 개선됨

- - -

### construction
![image](https://user-images.githubusercontent.com/44857109/100520657-34045080-31e2-11eb-9669-93271fa2b95f.png)

## High Availability
서비스가 장애가 발생하더라도 계속 운영할 수 있도록 만드는 기술

- 복제 슬레이브가 죽는다면
    - 새로운 슬레이브를 생성하고 복제를 다시 설정하면 끝
- 복제 마스터가 죽는다면
    - 슬레이브 중에 하나를 마스터로 변경하고 다른 슬레이브들은 마스터의 주소를 변경하여 다시 복제를 진행

- - -

### Active Standby
- 마스터가 될 후보중 2개의 노드를 선정
- Active 마스터가 장애가 발생하면 Standby 를 마스터로 변경, 이후 새로운 Standby 선정

### Active Active
- 두 개 이상의 Active 노드 구성 허용
- 하나의 Active 노드의 장애가 발생해도 다른 Active 노드로 서비스
- Active 노드들끼리 의견이 다르다면 합의가 필요
    - Active 노드를 홀수로 구성 (3, 5, 7)
    - MySQL 에 Group Replication(마스터를 여러개 제어하는 기능, 합의와 비슷)가 있지만 Active-Active 구성이라 하기엔 부족함

- - -

### Solution
- MMM(MySQL Multi-master Replication Manager)
    - Active-Standby
    - Active 의 장애를 MMM Manager가 감지하고 Standby 를 Master 로 변경하는 기술
    - MMM Manager는 이중화 불가능
- MHA(MySQL HA)
- Group Replication
    - 마스터의 의견이 과반 수 이상일 경우 반영
- Galera Cluster
    - 모든 노드에 read/write 가능

## Backup
- 논리백업 vs. 물리백업
    - 논리백업
        - SQL 쿼리의 형태로 복사
        - 전체백업/부분백억/증분백업
    - 물리백업
        - 데이터를 가진 물리파일을 직접 복사
        - 전체백업이 기본
        - 논리백업보다 빠름
- 핫백업 vs. 콜드백업
    - 핫백업
        - 서비스를 동장하면서 백업
    - 콜드백업
        - 서버를 정지시키고 백업

### Dump(mysqldump)
- 논리백업
- 전체백업
- 글로벌락
```
// DBMS 전체 백업
$ mysqldump -uroot -p --all-databases > dump.sql

// employees database 백업
$ mysqldump -uroot -p employees > employees_db.sql

// employees table 백업
$ mysqldump -uroot -p employees employees > employees_table.sql
```

```
$ mysql -uroot -p < dump.sql

mysql> source dump.sql;
```

### EnterpriseBackup
- 유료버전
- 전체/증분/압축 백업
- 서비스 동작도중에도 백업 가능

### XtraBackup
- 무료이면서 성능이 좋음
### mariabackup
- mariadb:10.3 부터 Redo Log 포맷 변경으로 사용 권장