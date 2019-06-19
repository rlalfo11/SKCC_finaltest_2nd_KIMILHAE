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
	ip-172-31-2-76.ap-northeast-2.compute.internal
	ip-172-31-11-108.ap-northeast-2.compute.internal
	ip-172-31-5-178.ap-northeast-2.compute.internal
	ip-172-31-2-186.ap-northeast-2.compute.internal
	ip-172-31-14-114.ap-northeast-2.compute.internal
	----

### iii.
	$ cat /etc/*release*
	----
	CentOS Linux release 7.6.1810 (Core)
	Derived from Red Hat Enterprise Linux 7.6 (Source)
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
	/dev/nvme0n1p1 104846316 1156260 103690056   2% /
	devtmpfs         7872836       0   7872836   0% /dev
	tmpfs            7895692       0   7895692   0% /dev/shm
	tmpfs            7895692   16736   7878956   1% /run
	tmpfs            7895692       0   7895692   0% /sys/fs/cgroup
	tmpfs            1579140       0   1579140   0% /run/user/1000
	----
	
### v.	
	$ yum repolist all
	----
	Loaded plugins: fastestmirror
	Determining fastest mirrors
	epel/x86_64/metalink                                     | 7.5 kB     00:00
	 * base: mirror.kakao.com
	 * epel: d2lzkl7pfhq30w.cloudfront.net
	 * extras: mirror.kakao.com
	 * updates: mirror.kakao.com
	base                                                     | 3.6 kB     00:00
	cloudera-manager                                         |  951 B     00:00
	epel                                                     | 4.7 kB     00:00
	extras                                                   | 3.4 kB     00:00
	updates                                                  | 3.4 kB     00
	----

### vi.		
	$ cat /etc/passwd
	----
	training:x:1001:1001::/home/training:/bin/bash
	----

### vii.		
	$ cat /etc/group
	----
	skcc:x:1002:training
	----

### viii.		
	$ getent group skcc
	----
	skcc:x:1002:training
	----
	$ getent passwd training
	----
	training:x:1001:1001::/home/training:/bin/bash
	----
	
## 1.b
	ii.
	1. 
	$ hostname
	----
	nd1.team5.com
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
	$ mysql -u root -p test < /Users/smartwork/Desktop/big-data-final/authors.sql
	$ mysql -u root -p test < /Users/smartwork/Desktop/big-data-final/posts.sql	
## 2.c
	$ mysql -u root -p
	GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'training';
	FLUSH PRIVILEGES;	
	
# 3. Extract tables authors and posts
## 3.a~c
	- sqoop import
	sqoop import --connect jdbc:mysql://localhost/test --username training --password training --table authors --target-dir /authors
	sqoop import --connect jdbc:mysql://localhost/test --username training --password training --table posts --target-dir /posts
## 3.d~g
	- sqoop import with hive direct
	sqoop import --connect jdbc:mysql://localhost/test --username training --password training --table authors --target-dir /authors --hive-import --create-hive-table --hive-table default.authors
	sqoop import --connect jdbc:mysql://localhost/test --username training --password training --table posts --target-dir /posts --hive-import --create-hive-table --hive-table default.posts
	
# 4. Create and run a Hive/Impla query.
## 4.a~c
	- 작성자와 작성자가 작성한 게시물의 개수를 쿼링하여 /results 에 저장
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
	sqoop export \
	--connect jdbc:mysql://localhost/test \
	--username training \
	--password training \
	--table results \
	--export-dir /results
	--input-fields-terminated-by '\t'
	
