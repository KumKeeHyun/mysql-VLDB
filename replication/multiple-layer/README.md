# Replication - Multiple Layer
using mysql:5.6

## container
- first (second's master)
- second (first's slave, third's master)
- third (second's slave)

first <= second <= third

## config
### first
single layer의 master와 동일
```
[mysqld]
log-bin=mysql-bin
server-id=1
```

### second
- log-bin, **log-slave-updates** : 슬레이브를 다시 마스터로 사용하기 위해 설정
- read-only : second layer또한 외부에서 쓰기 동작 수행을 제한
```
[mysqld]
log-bin=mysql-bin
log-slave-updates=true
server-id=2
read-only=1
```

### third
single layer의 slave 동일
```
[mysqld]
server-id=3
read_only=1
```

# Run
## first <= second
single-layer에서의 과정과 동일

## second <= third
single-layer에서의 과정과 동일

