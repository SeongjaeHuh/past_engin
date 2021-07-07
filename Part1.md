3.34.216.162	10.0.0.145	util1.com util

52.79.154.137	10.0.0.157	mn.com mn

3.34.6.203	10.0.0.19 		dn1.com dn1

3.34.68.165	10.0.0.158 	dn2.com dn2

3.34.75.86	10.0.0.159 	dn3.com dn3

## 1. Create a CDH Cluster on AWS

### a. Linux setup
####  Add the following linux accounts to [all nodes]

1번: github 5번
2번: findmnt 
3번: df -Th
4번: github 6번
5번: ipconfig
6번: 재작년github hostname 설정
7번: github 7번
8번: github 8번


### c. Install Cloudera Manager
CDH version 5.15.2

```
sudo yum install wget
```


**[all nodes]**
```
sudo wget http://ec2-3-34-114-205.ap-northeast-2.compute.amazonaws.com/cloudera-repos/cdh5/5.16.2/ -P /etc/yum.repos.d/
```
![](/img/1-10.PNG)

**[only util]**

#### rpc에 key 추가
```
sudo rpm --import http://ec2-3-34-114-205.ap-northeast-2.compute.amazonaws.com/cloudera-repos/cdh5/5.16.2/RPM-GPG-KEY-cloudera
```
#### cloudera install
```
# cloudera install
sudo yum install cloudera-manager-daemons cloudera-manager-server
```
![](/img/1-23.PNG)

### JDK Install
https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

jdk-8u291-linux-x64.rpm
```
sudo mv cloudera-manager.repo cloudera-manager.repo.bak
```
```
sudo yum install -y java-1.8.0-openjdk-devel
```
### d. Install a MySQl server
A command and output that reports the database server version

![](/img/추가1.PNG)

A command and output that lists all the databases in the server

![](/img/추가2.PNG)


#### Install a supported Oracle JDK [all nodes]
```
# 설치 가능한 jdk list 확인
sudo yum list oracle*

sudo yum install -y oracle-j2sdk1.7
```
![](/img/1-24.PNG)


#### java 경로 설정 [only util]  

```
vi ~/.bash_profile

# 아래 두줄 추가
# :$PATH가 뒤에 있어야 java -version 했을 때 정상적인 버전 확인 가능
export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
export PATH=/usr/java/jdk1.7.0_67-cloudera/bin:$PATH

source ~/.bash_profile

# java version 확인
java -version
```
![](/img/1-25.PNG)

  
#### jdbc connector 설치 [all nodes]

```
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
cd /usr/share/java/
sudo yum install -y mysql-connector-java
```
![](/img/1-26.PNG)


### e. maria db 설치 및 권한 설정 [only util]  

```
sudo yum install -y mariadb-server
sudo systemctl stop mariadb
```
configure mariadb server
```
sudo vi /etc/my.cnf
```

```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MariaDB, if you enable the binary log and do not set
#a server_id, MariaDB will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

start maria db
```
sudo systemctl enable mariadb
sudo systemctl start mariadb
# mariadb 상태 확인
sudo systemctl status mariadb
sudo /usr/bin/mysql_secure_installation

# 1. enter
# Y, root pw입력
# Y
# N
# Y
# Y
```

![](/img/1-27.PNG)

```
# 권한 설정 : 전체 Y 선택
sudo /usr/bin/mysql_secure_installation
```
![](/img/1-28.PNG)

#### Creating Databases for Cloudera Software

```
mysql -u root -p

# db, user 생성
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'password';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;
```
![](/img/1-30.PNG)

### f. Cloudera Manager Install

**[all nodes]**
```
#모든 Node에 비밀번호 설정 (*중요*)
sudo passwd centos
```
![](/img/1-31.PNG)

**[only util]**
```
#  set CM DB
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user password
# CM server start
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
![](/img/1-32.PNG)

#### Install a cluster and deploy CDH

```
http://util.com:7180
admin / admin
```
**CHD 클러스터 설치 시작**

![](/img/1-33.PNG)
![](/img/1-34.PNG)
![](/img/1-35.PNG)
```
jdk는 node에 직접 설치했기 때문에 uncheck
```
![](/img/1-36.PNG)
![](/img/1-37.PNG)
![](/img/1-38.PNG)
![](/img/1-39.PNG)
![](/img/1-40.PNG)
![](/img/1-41.PNG)
```
HDFS, YARN, ZooKeeper 먼저 설치
```
![](/img/1-42.PNG)
```
# HDFS
HttpFS 선택 x
NameNode - mn
SecondaryNameNode - utils
Balancer - dn1
DataNode - dn[1-3]
# CM
Telementry Publisher 선택 x
그 외 - all util
# YARN
ResourceManager - mn
JobHistory Server - dn1
Nodemanager - dn[1-3]
# ZooKeeper
Server - dn[1-2],mn
```
![](/img/1-43.PNG)
![](/img/1-44.PNG)
![](/img/1-45.PNG)
![](/img/1-46.PNG)

**Hive 서비스 추가**

![](/img/1-49.PNG)
```
Gateway - dn[1-3], util
Hive Metastore Server - util
HiveServer2 - util
```
![](/img/1-50.PNG)
![](/img/1-51.PNG)
![](/img/1-52.PNG)

**Sqoop 서비스 추가**
```
Sqoop2 Server - util
```
![](/img/1-54.PNG)
![](/img/1-55.PNG)

**Impala 서비스 추가**  

```
Impala Catalog Server - util
Impala State Store- util
Impala EDaemon - dn[1-3]
```

![](/img/1-56.PNG)
![](/img/1-57.PNG)

**Oozie 서비스 추가**
![](/img/1-58.PNG)
~~~
Oozie Server - util
~~~
![](/img/1-59.PNG)
![](/img/1-60.PNG)
![](/img/1-61.PNG)

**Hue 서비스 추가**
![](/img/1-63.PNG)
```
Hue Server - util
Load Balancer - util
```
![](/img/1-64.PNG)
![](/img/1-65.PNG)

**모든 서비스 설치 완료!**
![](/img/1-66.PNG)

#### In you cluster, create a user named “training” with password “training”

```
hue - training 계정 생성
```
![](/img/1-67.PNG)
#### make sure user “training” has both a linux and HDFS home directory

![](/img/1-68.PNG)

## 2. In MySQL create the sample tables that will be used for the rest of the test

### a. In MySQL, create a database and name it “test”
```
mysql -u root -p

create database test;
```
test 데이터베이스 생성 확인  
![](/img/1-69.PNG)

### b. Create 2 tables in the test databases: authors and posts.
#### You will use the authors.sql and posts.sql script files that will be provided for you to generate the necessary tables

```
# file copy local > util

scp skcc.pem authors.sql.zip training@15.164.144.197:.
scp skcc.pem authors.sql.zip training@15.164.144.197:.
```
![](/img/1-70.PNG)

```
sudo yum install -y unzip

unzip authors.sql.zip
unzip posts.sql.zip
```
![](/img/1-71.PNG)

```
mysql -u root -p

use test;

# sql
source authors-23-04-2019-02-34-beta.sql
source posts23-04-2019 02-44.sql
```
![](/img/1-72.PNG)

### c. Create and grant user “training” with password “training” full access to the test database.

```
create user 'training'@'%' identified by 'training';

grant all privileges on *.* to 'training'@'%';
```
![](/img/1-73.PNG)

## 3. Extract tables authors and posts from the database and create Hive tables.

```
# training 계정으로 접속
su training
```

```
# sqoop 수행 [util]
sqoop import \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table posts \
--fields-terminated-by "\t" \
--target-dir /user/training/posts
```

```
sqoop import \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table authors \
--fields-terminated-by "\t" \
--target-dir /user/training/authors
```
![](/img/1-74.PNG)
![](/img/1-75.PNG)


```
create external table authors
(
id int,
first_name string,
last_name string,
email string,
birthdate date,
added timestamp
)
row format delimited
fields terminated by '\t'
location '/user/training/authors/.'
```
![](/img/1-77.PNG)
```
create table posts
(
id int,
author_id int,
title string,
description string,
content string,
date date
)
row format delimited
fields terminated by '\t'
location '/user/training/posts/.'
```
![](/img/1-79.PNG)

## 4. Create and run a Hive/Impala query. From the query, generate the results dataset that you will use in the next step to export in MySQL.

```
select a.id Id, a.first_name fname, a.last_name lname, count(*) num_posts
from authors a, posts p
where a.id = p.author_id
group by a.id, a.first_name, a.last_name;
```

![](/img/1-80.PNG)

### The output of the query should be saved in your HDFS home directory.

```
insert overwrite directory '/user/training/results'
row format delimited
fields terminated by '\t'
select a.id Id, a.first_name fname, a.last_name lname, count(*) num_posts
from authors a, posts p
where a.id = p.author_id
group by a.id, a.first_name, a.last_name;
```
![](/img/1-81.PNG)

## 5. Export the data from above query to MySQL

### Create a MySQL "results" table under the database "test"

```
create table results
( Id int, fname varchar(500), lname varchar(500), num_posts int);
```

### export into MySQL the results of your query

```
sqoop export \
--connect jdbc:mysql://util.com/test \
--username training \
--password training \
--table results \
--fields-terminated-by '\t' \
--export-dir hdfs://mn.com/user/training/results
```
![](/img/1-83.PNG)

