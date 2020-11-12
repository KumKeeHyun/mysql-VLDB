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
mysql> select * from mysql.servers;
```

## spider, shard
#### 3. Create User, Database
Spider(shard proxy), Shard 모든 db 서버에 공통으로 수행
```
mysql> create database $(database-name);
mysql> create user '$(user-name)'@'%' identified by '$(user-pw)';
# 편의를 위해 모든 권한 부여
mysql> grant all on *.* to '$(user-name)'@'%' with grant option;
mysql> flush privileges;
```