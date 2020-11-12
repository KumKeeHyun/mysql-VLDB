# Replication - Single Layer
using mysql:5.6

## container
- master
- slave1
- slave2

## config
mysql container의 /etc/mysql/conf.d를 설정해야 함. 'config_file.cnf' 파일명으로 파일 생성 후 volume으로 이어줌.
### master
server-id는 정수형으로 지정 (ex: 1 or 2 ...). 중복되지 않아야함.
```
[mysqld]
log-bin=mysql-bin
server-id=$(id)
```

### slave
```
 [mysqld]
 server-id=$(id)
 read_only=1
```

### check server-id
```
mysql> select @@server_id;
```

## master
#### 1. Check binary-log info
```
mysql> show master status;
| $(bin-log-filename) | $(bin-log-offset) | 
(ex: | mysql-bin.000004 | 120 | )
```

#### 2. Create User
```
mysql> CREATE USER '$(user-name)'@'%' IDENTIFIED BY '$(user-pw)';
mysql> GRANT REPLICATION SLAVE ON *.* TO '$(user-name)'@'%';
```

## slave
#### 3. Set master info
#1에서 확인한 master의 binary-log 정보, #2에서 설정한 user 정보를 입력
```
mysql> CHANGE MASTER TO MASTER_HOST='$(master-ipaddress)', 
  MASTER_USER='$(user-name)', MASTER_PASSWORD='$(user-pw)',
  MASTER_LOG_FILE='$(bin-log-filename)', MASTER_LOG_POS=$(bin-log-offset);
```

#### 4. Start slave for trace master's bin-log
```
mysql> start slave;
mysql> show slave status\G
```
'Slave_IO_Running', 'Slave_SQL_running' 두 값이 모두 Yes 인지 확인

만약 문제가 생긴다면 
```
mysql> stop slave;
```
이후 #3, #4 재수행