# Sharding using Spider
Spider only support mariadb... so using image mariadb:10.1 not mysql

## container
- spider
- shard1
- shard2

## spider
#### 1. Install spider plug-in
```
$ find / -name install_spider.sql
$ cd /usr/share/mysql

# connect to mariadb 

mysql> source install_spider.sql
# check successful install
mysql> show engines;
```

#### 2. Set shard server info
- shard-name을 통해 shard-db server를 구분
- DATABASE, USER : Spider(shard proxy), Shard에 모두 공통으로 설정해야 함
```
mysql> CREATE SERVER $(shard-name)
FOREIGN DATA WRAPPER mysql
OPTIONS(
  HOST '$(shard-ipaddress)',
  DATABASE '$(database-name)',
  USER '$(user-name)',
  PASSWORD '$(user-pw)',
  PORT 3306
);
```
```
# check shard server info
mysql> SELECT * FROM mysql.servers;
```

## spider, shard
#### 3. Create User, Database
Spider(shard proxy), Shard(모든 db 서버)에 공통으로 수행
```
mysql> CREATE DATABASE $(database-name);
mysql> CREATE USER '$(user-name)'@'%' IDENTIFIED BY '$(user-pw)';
# 편의를 위해 모든 권한 부여
mysql> GRANT ALL ON *.* TO '$(user-name)'@'%' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

## spider
#### 4. Create table for spider proxy
파티션에 등록하는 $(shard-server-name-1)는 #2에서 등록한 shard 서버 정보의 $(shard-name)을 사용
```
mysql> CREATE TABLE $(table-name)
(
  key-field     int(10)      NOT NULL AUTO_INCREMENT,
  some-field-1  char(120)    NOT NULL DEFAULT '',
  some-field-2  varchar(20)  NOT NULL,
  PRIMARY KEY (key-field)
) ENGINE=spider COMMENT='wrapper "mysql", table "$(table-name)"'
PARTITION BY KEY (key-field)
(
  PARTITION $(shard-server-name-1) COMMENT = 'srv "shard1"',
  PARTITION $(shard-server-name-2) COMMENT = 'srv "shard2"'
);
```

## shard
#### 5. Create table for shard
#4에서 등록한 테이블과 동일한 스키마의 테이블 등록. #2에서 등록한 shard 서버 정보의 $(database-name)에 table을 생성해야 함.
```
mysql> CREATE TABLE $(table-name)
(
  key-field     int(10)      NOT NULL AUTO_INCREMENT,
  some-field-1  char(120)    NOT NULL DEFAULT '',
  some-field-2  varchar(20)  NOT NULL,
  PRIMARY KEY (key-field)
) ENGINE=InnoDB
```

## spider
#### 6. Insert data into spider proxy table
```
mysql> INSERT INTO $(databse-anme).$(table-name) VALUES(...)
```