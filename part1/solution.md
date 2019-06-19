# 1. Create a CDH Cluster on AWS
## 1.a

### i.
	$ sudo useradd training -u 3800
	$ sudo passwd training
	$ sudo groupadd skcc
	$ sudo usermod -a -G skcc training
	
### ii.
	IP정보 조회 :  ip addr
	호스트명 조회 : hostname
	----
	cm.rlalfo.com cm
	m1.rlalfo.com m1
	d1.rlalfo.com d1
	d2.rlalfo.com d2
	d3.rlalfo.com d3
	----

### iii.
	$ cat /etc/*release*
	----
	CentOS Linux release 7.6.1810 (Core)
	Derived from Red Hat Enterprise Linux 7.6 (Source)
	cat: /etc/lsb-release.d: Is a directory
	NAME="CentOS Linux"
	VERSION="7 (Core)"
	ID="centos"
	ID_LIKE="rhel fedora"
	VERSION_ID="7"
	PRETTY_NAME="CentOS Linux 7 (Core)"
	ANSI_COLOR="0;31"
	CPE_NAME="cpe:/o:centos:centos:7"
	HOME_URL="https://www.centos.org/"
	BUG_REPORT_URL="https://bugs.centos.org/"

	CENTOS_MANTISBT_PROJECT="CentOS-7"
	CENTOS_MANTISBT_PROJECT_VERSION="7"
	REDHAT_SUPPORT_PRODUCT="centos"
	REDHAT_SUPPORT_PRODUCT_VERSION="7"

	CentOS Linux release 7.6.1810 (Core)
	CentOS Linux release 7.6.1810 (Core)
	cpe:/o:centos:centos:7
	----
	
	
### iv.	
	$ df
	----
	Filesystem     1K-blocks    Used Available Use% Mounted on
	/dev/nvme0n1p1 104846316 3157176 101689140   4% /
	devtmpfs         7872408       0   7872408   0% /dev
	tmpfs            7895688       0   7895688   0% /dev/shm
	tmpfs            7895688   16884   7878804   1% /run
	tmpfs            7895688       0   7895688   0% /sys/fs/cgroup
	tmpfs            1579140       0   1579140   0% /run/user/1000
	tmpfs            1579140       0   1579140   0% /run/user/997
	tmpfs            1579140       0   1579140   0% /run/user/0
	cm_processes     7895688       0   7895688   0% /run/cloudera-scm-agent/process
	----
	
### v.	
	$ yum repolist all
	----
	base/7/x86_64                          CentOS-7 - Base                                 enabled: 10,019
	cloudera-manager                           Cloudera Manager, Version 5.15.2                enabled:      7
	extras/7/x86_64                            CentOS-7 - Extras                               enabled:    417
	updates/7/x86_64                           CentOS-7 - Updates                              enabled:  2,089
	----

### vi.		
	$ cat /etc/passwd
	----
	training:x:3800:3800::/home/training:/bin/bash
	----

### vii.		
	$ cat /etc/group
	----
	training:x:3800:
	skcc:x:3801:training
	----

### viii.		
	$ getent group skcc
	----
	skcc:x:3801:training
	----
	$ getent passwd training
	----
	training:x:3800:3800::/home/training:/bin/bash
	----
	
## 1.b
	ii.
	1. 
	$ hostname
	----
	cm.rlalfo.com
	----
	2.
	mysql --version
	----
	mysql  Ver 15.1 Distrib 5.5.60-MariaDB, for Linux (x86_64) using readline 5.1
	----
	3. MariaDB [(none)]> show databases;
	----
	+--------------------+
	| Database           |
	+--------------------+
	| information_schema |
	| amon               |
	| hue                |
	| metastore          |
	| mysql              |
	| nav                |
	| navms              |
	| oozie              |
	| performance_schema |
	| rman               |
	| scm                |
	| sentry             |
	+--------------------+
	12 rows in set (0.00 sec)
	----
	
## 1.c	
	MariaDB [(none)]> GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
	----
	Query OK, 0 rows affected (0.00 sec)
	----

	MariaDB [(none)]> FLUSH PRIVILEGES;
	----
	Query OK, 0 rows affected (0.00 sec)
	----
	
	cloudera manager img
	![photo.PNG](https://github.com/rlalfo11/bigdata_final/blob/master/CM_Manager.PNG?raw=true)
	
	
	
# 2. create sample table	
## 2.a
	$ mysql -u root -p
	CREATE DATABASE test;
	EXIT;
## 2.b
   	FTP를 활용해 쿼리 파일 cm 노드로 이동 후 명령어 입력 진행
    
	$ mysql -u root -p test < ./authors.sql
	$ mysql -u root -p test < ./posts.sql

	MariaDB [(none)]> use test;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	MariaDB [test]> show tables;
	+----------------+
	| Tables_in_test |
	+----------------+
	| authors        |
	| posts          |
	+----------------+
	2 rows in set (0.00 sec)
	
## 2.c
	$ mysql -u root -p
	GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'training';
	Query OK, 0 rows affected (0.00 sec)

	FLUSH PRIVILEGES;	
	Query OK, 0 rows affected (0.00 sec)

# 3. Extract tables authors and posts
## 3.a~c
	- sqoop import
	sqoop import --connect jdbc:mysql://cm.rlalfo.com/test --username training --password training --table authors --target-dir /authors_2
	sqoop import --connect jdbc:mysql://cm.rlalfo.com/test --username training --password training --table posts --target-dir /posts_2

## 3.d~g
	- sqoop import with hive direct
	sqoop import --connect jdbc:mysql://cm.rlalfo.com/test --username root --password password --table authors --target-dir /authors --hive-import --create-hive-table --hive-table default.authors
	sqoop import --connect jdbc:mysql://cm.rlalfo.com/test --username root --password password --table posts --target-dir /posts --hive-import --create-hive-table --hive-table default.posts
	
# 4. Create and run a Hive/Impla query.
## 4.a~c
	insert overwrite directory '/results'
	row format delimited fields terminated by '\t'
	select A.id,
		   A.first_name AS fname,
		   A.last_name AS lname
		   B.num_posts AS num_posts
	from authors A
	inner join ( select author_id, count(author_id) AS num_posts
					from posts P
				   group by author_id ) B
	on A.id = B.author_id

# 5. Export the data form above query to MySQL
## 5.a	
	CREATE TABLE `results` (
	  `id` int NOT NULL,
	  `fname` varchar(255) default NULL,
	  `lname` varchar(255) default NULL,
	  `num_posts` int default 0
	);

## 5.b	
	sqoop export --connect jdbc:mysql://localhost/test --username training --password training --table results --export-dir /results --input-fields-terminated-by '\t'
