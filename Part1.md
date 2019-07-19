### 1. Create a CDH Cluster on AWS

#### a. Linux setup
i. Add the following linux accounts to [all nodes]
1. User training with a UID of 3800
2. Set the password for user “training” to “training”
3. Create the group skcc and add training to it
4. Give training sudo capabilities

```
# 계정 생성
sudo useradd training
sudo usermod -u 3800 training
sudo passwd training

sudo groupadd skcc
cat /etc/passwd | grep training
sudo usermod -aG skcc training

# 계정 수도권한추가
sudo usermod -aG wheel training
```
![](/img/1-14.PNG)
![](/img/1-15.PNG)
![](/img/1-16.PNG)

ii. List the your instances by IP address and DNS name

```
sudo vi /etc/hosts

# 아래와 같이 내용 추가 private ip <모든 node>
172.31.43.223 util.com util
172.31.43.162 mn.com mn
172.31.37.124 dn1.com dn1
172.31.47.50 dn2.com dn2
172.31.33.50 dn3.com dn3
```
![](/img/1-1.PNG)
```
# 각각의 node (노드 명은 약어 말고 full name)
sudo hostnamectl set-hostname <노드명>
예) sudo hostnamectl set-hostname util.com

# 변경된 hostname 확인
hostname
```
![](/img/1-2.PNG)
```
sudo yum install bind-utils net-tools -y

nslookup [도메인명]

getent hosts
```
![](/img/1-18.PNG)
![](/img/1-19.PNG)

iii. List the Linux release you are using
```
linux version 확인

[centos@ip-172-31-43-223 ~]$ grep . /etc/*-release
```
![](/img/1-11.PNG)

iv. List the file system capacity for the first node (master node)
```
[centos@ip-172-31-43-162 ~]$ df -Th
```
![](/img/1-12.PNG)

v. List the command and output for yum repolist enabled
```
yum repolist all
```
![](/img/1-13.PNG)


vi. List the /etc/passwd entries for training (only in master name node)
vii. List the /etc/group entries for skcc (only in master name node)
```
cat /etc/passwd | grep training
cat /etc/group | grep skcc
```

![](/img/1-21.PNG)

viii. List output of the flowing commands:
```
getent group skcc

getent passwd training
```
![](/img/1-22.PNG)

#### 추가 setting

```
sudo vi /etc/ssh/sshd_config
# PasswordAuthentication -> yes 로 변경 후 저장

# sshd 재시작 및 상태확인
sudo systemctl restart sshd.service
sudo systemctl status sshd.service
```
![](/img/1-3.PNG)
![](/img/1-5.PNG)
```
sudo yum update
sudo yum install -y wget

# 전체 y 선택
```
![](/img/1-7.PNG)

**ntp setting**
```
sudo yum install ntp

sudo vi /etc/ntp.conf
# 내용 변경 : 위에 네줄 주석처리, 아래 세줄 추가
#server 0.centos.pool.ntp.org
#server 1.centos.pool.ntp.org
#server 2.centos.pool.ntp.org
#server 3.centos.pool.ntp.org
server kr.pool.ntp.org
server time.bora.net
server time.kornet.net

sudo chkconfig ntpd on
sudo systemctl start ntpd
# 성공 확인
ntpq -p
```
![](/img/1-8.PNG)
![](/img/1-9.PNG)


### c. Install Cloudera Manager
CDH version 5.15.2

[all nodes]
```
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/

# base url 수정
sudo vi /etc/yum.repos.d/cloudera-manager.repo
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.15.2/
```
![](/img/1-10.PNG)

[only util]

rpc에 key 추가
```
sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
```
cloudera install
```
# cloudera install
sudo yum install cloudera-manager-daemons cloudera-manager-server
```
![](/img/1-23.PNG)



### b. Install a MySQl server
1. A command and output that shows the hostname of your database server

2. A command and output that reports the database server version

![](/img/추가1.PNG)
3. A command and output that lists all the databases in the server
![](/img/추가2.PNG)

이미지 추가 1,2

[all nodes]
```
# 설치 가능한 jdk list 확인
sudo yum list oracle*

sudo yum install -y oracle-j2sdk1.7
```
![](/img/1-24.PNG)

[only util]
* java 경로 설정

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

[all nodes]  
* jdbc connector 설치

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

[only util]
* maria db 설치 및 권한 설정

```
sudo yum install -y mariadb-server

sudo systemctl enable mariadb
sudo systemctl start mariadb
# mariadb 상태 확인
sudo systemctl status mariadb
```
![](/img/1-27.PNG)

```
# 권한 설정 : 전체 Y 선택
sudo /usr/bin/mysql_secure_installation
```
![](/img/1-28.PNG)

* Creating Databases for Cloudera Software

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

#### Cloudera Manager Install

[all nodes]
```
#모든 Node에 비밀번호 설정 (*중요*)
sudo passwd centos
```
![](/img/1-31.PNG)

[only util]
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
####
iv. In you cluster, create a user named “training” with password “training”
make sure user “training” has both a linux and HDFS home directory
```
hue - training 계정 생성
```
![](/img/1-67.PNG)
~~~
user/training 폴더 생성 확인
~~~
![](/img/1-68.PNG)

### 2. In MySQL create the sample tables that will be used for the rest of the test

#### a. In MySQL, create a database and name it “test”
```
mysql -u root -p

create database test;
```
test 데이터베이스 생성 확인  
![](/img/1-69.PNG)

#### b. Create 2 tables in the test databases: authors and posts.
i. You will use the authors.sql and posts.sql script files that will be provided for you to generate the necessary tables

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

#### c. Create and grant user “training” with password “training” full access to the test database. (It is ok if you give training full access to the entire MySQL database)

```
create user 'training'@'%' identified by 'training';

grant all privileges on *.* to 'training'@'%';
```
![](/img/1-73.PNG)

### 3. Extract tables authors and posts from the database and create Hive tables.

#### a. Use Sqoop to import the data from authors and posts

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

#### b. For both tables, you will import the data in tab delimited text format

#### c. The imported data should be saved in training’s HDFS home directory
i. Create authors and posts directories in your HDFS home directory
ii. Save the imported data in each

#### d. In Hive, create 2 tables: authors and posts. They will contain the data that you imported from Sqoop in above step.

#### e. You are free to use whatever database in Hive.

#### f. Create authors as an external table.

#### g. Create posts as a managed table.

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

### 4. Create and run a Hive/Impala query. From the query, generate the results dataset that you will use in the next step to export in MySQL.

#### a. Create a query that counts the number of posts each author has created.
i. The id column in authors matches the author_id key in posts.

#### b. The output of the query should provide the following information:

```
select a.id Id, a.first_name fname, a.last_name lname, count(*) num_posts
from authors a, posts p
where a.id = p.author_id
group by a.id, a.first_name, a.last_name;
```

![](/img/1-80.PNG)

#### c. The output of the query should be saved in your HDFS home directory.

i. Save it under “results” directory

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

### 5. Export the data from above query to MySQL

#### a. Create a MySQL table and name it “results”
i. Make sure it has the necessary columns of matching type as the results of your query from above

#### b. The table should be created under the database “test”

```
create table results
( Id int, fname varchar(500), lname varchar(500), num_posts int);
```

#### c. Finally, export into MySQL the results of your query

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