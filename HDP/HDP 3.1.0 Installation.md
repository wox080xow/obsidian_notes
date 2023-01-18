## 環境
- HDP 3.1.0
- Ambari 2.7
- CentOS 7.6

# `ssh`無密碼登入
```
[root@malvin-hdp3-m1-2212 ~]# while read h;do sshpass -p asdf1234 ssh -o StrictHostKeyChecking=no $h "ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''" </dev/null; done <hostlist
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:6skpEb1AdOp1rWjjf9LOLgG+GUjhWa2fCxEeaIfPTvQ root@malvin-hdp3-m1-2212.example.com
The key's randomart image is:
+---[RSA 2048]----+
|   ..o..         |
|    *o* ..       |
|   +.@.=. .      |
|   .*.XoE.       |
|   ..B+=S.       |
|    oo*o+        |
|     .o= +       |
|    .oo++.o      |
|     .= .*+      |
+----[SHA256]-----+
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:43me9zlPjqSnuKuE4xUHj4lPxhCRar73NP/TWYAT9OM root@malvin-hdp3-m2-2212.example.com
The key's randomart image is:
+---[RSA 2048]----+
|      oo   ..    |
|      ..    ..   |
|     .. .    oo  |
|    o  + =  o... |
|   o  . S o  .E. |
|    .  * =      .|
|     .o O .  ..o.|
|    ...= = ooo== |
|     ...o.B=+=+oo|
+----[SHA256]-----+
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:auIdj2jd6bnl/VTK7TKrdw6ZZHYsL3Yj1Sc1yJltq7o root@malvin-hdp3-w1-2212.example.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|            . =  |
|             = +.|
|              .o+|
|        S     *o*|
|       .     =o@.|
|    ..+. ..  oX.+|
|   ..=.+o+ ..o==o|
|   .o o.=..E++o*o|
+----[SHA256]-----+
[root@malvin-hdp3-m1-2212 ~]# while read h; do sshpass -p asdf1234 ssh -o StrictHostKeyChecking=no $h "cat /root/.ssh/id_rsa.pub" </dev/null; done <hostlist >>/root/.ssh/authorized_keys
[root@malvin-hdp3-m1-2212 ~]# while read h; do sshpass -p asdf1234 scp /root/.ssh/authorized_keys $h:/root/.ssh/authorized_keys; done <hostlist
[root@malvin-hdp3-m1-2212 ~]# while read h; do ssh $h hostname </dev/null; done <hostlist
malvin-hdp3-m1-2212.example.com
malvin-hdp3-m2-2212.example.com
malvin-hdp3-w1-2212.example.com
```

# 外部資料庫
使用Ansible安裝PostgreSQL 9.6
```
malvin@malvin-ubuntu:~/hadoop-ansible-roles$ cat test_host
[test]
malvin-hdp3-m1-2212 ansible_ssh_host=172.16.1.71 ansible_ssh_port=22 ansible_user=root
```

```
malvin@malvin-ubuntu:~/hadoop-ansible-roles$ cat roles/external_database_installation/tasks/main.yml
---

- name: Install rsync (yum)
  yum:
    name:
      - rsync
  become: true
  when: ansible_distribution == "CentOS"

- name: Synchronization of src on the control machine to dest on the remote hosts
  ansible.posix.synchronize:
    src: files/
    dest: packages/

- name: Install PosgreSQL 9.6 (yum)
  yum:
    name:
      - packages/postgresql96-9.6.22-1PGDG.rhel7.x86_64.rpm
      - packages/postgresql96-libs-9.6.22-1PGDG.rhel7.x86_64.rpm
      - packages/postgresql96-server-9.6.22-1PGDG.rhel7.x86_64.rpm
  become: true
  when: ansible_distribution == "CentOS"
```

```
malvin@malvin-ubuntu:~/hadoop-ansible-roles$ ansible-playbook -i test_host extdb_hdp3.yml

PLAY [Before Installation of HDP] **********************************************

TASK [Gathering Facts] *********************************************************
ok: [malvin-hdp3-m1-2212]

TASK [EXT DB installation] *****************************************************

TASK [external_database_installation : Install rsync (yum)] ********************
changed: [malvin-hdp3-m1-2212]

TASK [external_database_installation : Synchronization of src on the control machine to dest on the remote hosts] ***
changed: [malvin-hdp3-m1-2212]

TASK [external_database_installation : Install PosgreSQL 9.6 (yum)] ************
changed: [malvin-hdp3-m1-2212]

PLAY RECAP *********************************************************************
malvin-hdp3-m1-2212        : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

# Ambari

```
[root@malvin-hdp3-m1-2212 ~]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
==============================================================================
Enter choice (1): 2
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.352.b08-2.el7_9.x86_64
Validating JDK on Ambari Server...done.
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)?
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 4
Hostname (localhost): malvin-hdp3-m1-2212.example.com
Port (5432):
Database name (ambari):
Postgres schema (ambari):
Username (ambari):
Enter Database Password (bigdata):
Re-enter password:
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL directly from the database shell to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-Postgres-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
ambari-admin-2.7.4.14.1.jar
....
Ambari repo file doesn't contain latest json url, skipping repoinfos modification
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```

```
[root@malvin-hdp3-m1-2212 ~]# ambari-server start
Using python  /usr/bin/python
Starting ambari-server
Ambari Server running with administrator privileges.
Organizing resource files at /var/lib/ambari-server/resources...
Ambari database consistency check started...
Server PID at: /var/run/ambari-server/ambari-server.pid
Server out at: /var/log/ambari-server/ambari-server.out
Server log at: /var/log/ambari-server/ambari-server.log
Waiting for server start.....................................
Server started listening on 8080

DB configs consistency check: no errors and warnings were found.
Ambari Server 'start' completed successfully.
```
### Installer -> 8. Review -> Blueprint
**Admin Name** : admin

**Cluster Name** : HDP3ofMalvin

**Total Hosts** : 3 (3 new)

**Repositories**:

-   redhat7 (HDP-3.1):  
    http://172.16.1.64/hdp/HDP/centos7/3.1.0.0-78
    
-   redhat7 (HDP-UTILS-1.1.0.22):  
    http://172.16.1.64/hdp/HDP-UTILS/centos7/1.1.0.22
    

**Services:**

-   _**HDFS**_
    -   DataNode : 1 host
    -   NameNode : malvin-hdp3-m1-2212.example.com
    -   NFSGateway : 0 host
    -   SNameNode : malvin-hdp3-m2-2212.example.com
-   _**YARN + MapReduce2**_
    -   Timeline Service V1.5 : malvin-hdp3-m2-2212.example.com
    -   NodeManager : 1 host
    -   ResourceManager : malvin-hdp3-m2-2212.example.com
    -   Timeline Service V2.0 Reader : malvin-hdp3-m2-2212.example.com
    -   Registry DNS : malvin-hdp3-m2-2212.example.com
-   _**HBase**_
    -   Master : malvin-hdp3-m2-2212.example.com
    -   RegionServer : 1 host
    -   Phoenix Query Server : 1 host
-   _**ZooKeeper**_
    -   Server : 3 hosts
-   _**Ambari Metrics**_
    -   Metrics Collector : malvin-hdp3-m1-2212.example.com
    -   Grafana : malvin-hdp3-m1-2212.example.com
-   _**SmartSense**_
    -   Activity Analyzer : malvin-hdp3-m1-2212.example.com
    -   Activity Explorer : malvin-hdp3-m1-2212.example.com
    -   HST Server : malvin-hdp3-m1-2212.example.com
# 功能確認
### HDFS
```
[root@malvin-hdp3-m1-2212 ~]# sudo -u hdfs hdfs dfs -mkdir /user/root
[root@malvin-hdp3-m1-2212 ~]# sudo -u hdfs hdfs dfs -chown root:root /user/root
[root@malvin-hdp3-m1-2212 ~]# sudo -u hdfs hdfs dfs -ls -d /user/root
drwxr-xr-x   - root root          0 2022-12-07 17:12 /user/root
```
### YARN
```
[root@malvin-hdp3-m1-2212 ~]# yarn jar `find /usr/hdp/*/hadoop-mapreduce/ -name "*mapred*example*jar"|head -n 1` pi 10 10
Number of Maps  = 10
Samples per Map = 10
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Starting Job
22/12/07 17:12:43 INFO client.RMProxy: Connecting to ResourceManager at malvin-hdp3-m2-2212.example.com/172.16.1.72:8050
22/12/07 17:12:43 INFO client.AHSProxy: Connecting to Application History server at malvin-hdp3-m2-2212.example.com/172.16.1.72:10200
22/12/07 17:12:44 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /user/root/.staging/job_1670403671240_0003
22/12/07 17:12:44 INFO input.FileInputFormat: Total input files to process : 10
22/12/07 17:12:44 INFO mapreduce.JobSubmitter: number of splits:10
22/12/07 17:12:45 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1670403671240_0003
22/12/07 17:12:45 INFO mapreduce.JobSubmitter: Executing with tokens: []
22/12/07 17:12:45 INFO conf.Configuration: found resource resource-types.xml at file:/etc/hadoop/3.1.0.0-78/0/resource-types.xml
22/12/07 17:12:46 INFO impl.YarnClientImpl: Submitted application application_1670403671240_0003
22/12/07 17:12:46 INFO mapreduce.Job: The url to track the job: http://malvin-hdp3-m2-2212.example.com:8088/proxy/application_1670403671240_0003/
22/12/07 17:12:46 INFO mapreduce.Job: Running job: job_1670403671240_0003
22/12/07 17:12:54 INFO mapreduce.Job: Job job_1670403671240_0003 running in uber mode : false
22/12/07 17:12:54 INFO mapreduce.Job:  map 0% reduce 0%
22/12/07 17:13:10 INFO mapreduce.Job:  map 20% reduce 0%
22/12/07 17:13:11 INFO mapreduce.Job:  map 30% reduce 0%
22/12/07 17:13:19 INFO mapreduce.Job:  map 40% reduce 0%
22/12/07 17:13:21 INFO mapreduce.Job:  map 60% reduce 0%
22/12/07 17:13:29 INFO mapreduce.Job:  map 70% reduce 0%
22/12/07 17:13:30 INFO mapreduce.Job:  map 80% reduce 0%
22/12/07 17:13:32 INFO mapreduce.Job:  map 90% reduce 0%
22/12/07 17:13:37 INFO mapreduce.Job:  map 100% reduce 0%
22/12/07 17:13:41 INFO mapreduce.Job:  map 100% reduce 100%
22/12/07 17:13:42 INFO mapreduce.Job: Job job_1670403671240_0003 completed successfully
22/12/07 17:13:42 INFO mapreduce.Job: Counters: 53
	File System Counters
		FILE: Number of bytes read=226
		FILE: Number of bytes written=2560008
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2860
		HDFS: Number of bytes written=215
		HDFS: Number of read operations=45
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=3
	Job Counters
		Launched map tasks=10
		Launched reduce tasks=1
		Data-local map tasks=10
		Total time spent by all maps in occupied slots (ms)=97271
		Total time spent by all reduces in occupied slots (ms)=14840
		Total time spent by all map tasks (ms)=97271
		Total time spent by all reduce tasks (ms)=7420
		Total vcore-milliseconds taken by all map tasks=97271
		Total vcore-milliseconds taken by all reduce tasks=7420
		Total megabyte-milliseconds taken by all map tasks=99605504
		Total megabyte-milliseconds taken by all reduce tasks=15196160
	Map-Reduce Framework
		Map input records=10
		Map output records=20
		Map output bytes=180
		Map output materialized bytes=280
		Input split bytes=1680
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=280
		Reduce input records=20
		Reduce output records=0
		Spilled Records=40
		Shuffled Maps =10
		Failed Shuffles=0
		Merged Map outputs=10
		GC time elapsed (ms)=2695
		CPU time spent (ms)=12150
		Physical memory (bytes) snapshot=8257257472
		Virtual memory (bytes) snapshot=31865405440
		Total committed heap usage (bytes)=7325876224
		Peak Map Physical memory (bytes)=806940672
		Peak Map Virtual memory (bytes)=2817052672
		Peak Reduce Physical memory (bytes)=207564800
		Peak Reduce Virtual memory (bytes)=3733016576
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=1180
	File Output Format Counters
		Bytes Written=97
Job Finished in 59.329 seconds
Estimated value of Pi is 3.20000000000000000000
[root@malvin-hdp3-m1-2212 ~]#
```
### HBase
```
[root@malvin-hdp3-m1-2212 ~]# hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/phoenix/phoenix-5.0.0.3.1.0.0-78-server.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/hadoop/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
For Reference, please visit: http://hbase.apache.org/2.0/book.html#shell
Version 2.0.2.3.1.0.0-78, r, Thu Dec  6 12:27:45 UTC 2018
Took 0.0033 seconds
hbase(main):001:0> status
1 active master, 0 backup masters, 1 servers, 0 dead, 2.0000 average load
Took 0.7483 seconds
hbase(main):002:0> list
TABLE
0 row(s)
Took 0.0185 seconds
=> []
hbase(main):003:0> create 't1','cf'
Created table t1
Took 1.4060 seconds
=> Hbase::Table - t1
hbase(main):004:0> put 't1','r1','cf:c1','v1'
Took 0.2346 seconds
hbase(main):005:0> scan 't1'
ROW                   COLUMN+CELL
 r1                   column=cf:c1, timestamp=1670404757494, value=v1
1 row(s)
Took 0.0595 seconds
hbase(main):006:0> exit
[root@malvin-hdp3-m1-2212 ~]#
```

### Phoenix
```
[root@malvin-hdp3-m1-2212 ~]# phoenix-sqlline
Setting property: [incremental, false]
Setting property: [isolation, TRANSACTION_READ_COMMITTED]
issuing: !connect jdbc:phoenix: none none org.apache.phoenix.jdbc.PhoenixDriver
Connecting to jdbc:phoenix:
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/phoenix/phoenix-5.0.0.3.1.0.0-78-client.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/hadoop/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
22/12/07 17:19:55 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Connected to: Phoenix (version 5.0)
Driver: PhoenixEmbeddedDriver (version 5.0)
Autocommit status: true
Transaction isolation: TRANSACTION_READ_COMMITTED
Building list of tables and columns for tab-completion (set fastconnect to true to skip)...
133/133 (100%) Done
Done
sqlline version 1.2.0
0: jdbc:phoenix:> !tables
+------------+--------------+-------------+---------------+----------+---------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE_NA |
+------------+--------------+-------------+---------------+----------+---------+
|            | SYSTEM       | CATALOG     | SYSTEM TABLE  |          |         |
|            | SYSTEM       | FUNCTION    | SYSTEM TABLE  |          |         |
|            | SYSTEM       | LOG         | SYSTEM TABLE  |          |         |
|            | SYSTEM       | SEQUENCE    | SYSTEM TABLE  |          |         |
|            | SYSTEM       | STATS       | SYSTEM TABLE  |          |         |
+------------+--------------+-------------+---------------+----------+---------+
0: jdbc:phoenix:> create table sc.t1 (id integer primary key, name varchar);
No rows affected (1.395 seconds)
0: jdbc:phoenix:> !tables
+------------+--------------+-------------+---------------+----------+---------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE_NA |
+------------+--------------+-------------+---------------+----------+---------+
|            | SYSTEM       | CATALOG     | SYSTEM TABLE  |          |         |
|            | SYSTEM       | FUNCTION    | SYSTEM TABLE  |          |         |
|            | SYSTEM       | LOG         | SYSTEM TABLE  |          |         |
|            | SYSTEM       | SEQUENCE    | SYSTEM TABLE  |          |         |
|            | SYSTEM       | STATS       | SYSTEM TABLE  |          |         |
|            | SC           | T1          | TABLE         |          |         |
+------------+--------------+-------------+---------------+----------+---------+
0: jdbc:phoenix:> upsert into sc.t1 values (1, 'Malvin');
1 row affected (0.118 seconds)
0: jdbc:phoenix:> select * from sc.t1;
+-----+---------+
| ID  |  NAME   |
+-----+---------+
| 1   | Malvin  |
+-----+---------+
1 row selected (0.041 seconds)
0: jdbc:phoenix:> !q
Closing: org.apache.phoenix.jdbc.PhoenixConnection
[root@malvin-hdp3-m1-2212 ~]#
```