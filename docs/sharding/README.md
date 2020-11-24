# Sharding
데이터를 여러 대의 시스템에 분할하여 저장
- 쓰기 성능 향상
    - 샤딩 개수가 증가하면 할수록 쓰기 성능 향상
    - 수평 샤딩
        - 모든 샤드가 동일한 스키마
    - 수직 샤딩 
        - 샤드별로 다른 스키마를 가짐
- 장애가 났을 경우 해당 샤드는 접근 안됨
    - 파티션 내성(Partition Tolerance)이 있음
        - 시스템의 일부가 망가져도 시스템이 계속 동작할 수 있는 것
- DBMS 자체에서 지원하지 않고 외부의 별도 SW 지원 필요
    - 미들웨어나 샤딩 프레임워크/플랫폼을 사용
    - MySQL CGE 는 자체 내장
- cf.
    - 복제 : 동일한 데이터를 여러 대에 저장
    - 파티셔닝 : 데이터를 한 시스템에서 여러 테이블에 분할하여 저장 

## Solution
- Middleware
    - Spider(MariaDB 기본내장)
        - 단순한 로드밸런서/취합 기능
        - 중간에 샤드 변경시 자동으로 Global Relocation 해주지 않음
    - Spock Proxy(MySQL Proxy 기반)
    - Gizzard(Twitter)
- DBMS 자체 지원
    - nStore
    - MongoDB
- Application 자체 지원
    - Hibernate Shards

## Sharding + Replication
먼저 샤딩을 적용한 후에 각 샤드를 복제
