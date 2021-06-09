# 事前準備
1. ssh無密碼登入
2. OS設定
3. JAVA安裝

# 安裝外部資料庫與倒入舊資料
Destination安裝PostgreSQL
```
yum install -y postgresql11-server
```
Destination初始化PostgreSQL
```
/usr/pgsql-11/bin/postgresql-11-setup initdb
```
Destination開啟並啟用PostgreSQL
```
systemctl enable postgresql-11
systemctl start postgresql-11
```
Source倒出資料
```
su - postgres
pg_dumpall --file=/tmp/pg_dumpall.sql
exit
scp /tmp/pg_dumpall.sql 172.16.1.64:/tmp/pg_dumpall.sql
```
Source複製PostgreSQL設定
```
scp /var/lib/pgsql/11/data/pg_hba.conf 172.16.1.64:/var/lib/pgsql/11/data/pg_hba.conf
scp /var/lib/pgsql/11/data/postgresql.conf 172.16.1.64:/var/lib/pgsql/11/data/postgresql.conf
```
Destination重啟PostgreSQL
```
systemctl restart postgresql-11
```
Destination倒入資料
```
su - postgres
psql -f /tmp/pgdumpall.sql
```
 
 # 安裝Cloudera Manager與複製設定
Source複製`cloudera-manager.repo`
```
scp /etc/yum.repos.d/cloudera-manager.repo 172.16.1.64:/etc/yum.repos.d/cloudera-manager.repo
```
Destination重整YUM Repository
```
yum clean alll
yum repolist
```
Destination下載與匯入GPG-KEY
```
wget http://172.16.1.57/CM/7.3.1/redhat7/yum/RPM-GPG-KEY-cloudera
rpm --import RPM-GPG-KEY-cloudera
```
Destination安裝cloudear-manager-server
```
yum install cloudear-manager-server
```
Source複製Cloudera Manager相關設定
```
scp -r /var/lib/cloudera-scm-server/ 172.16.1.64:/var/lib/cloudera-scm-server/
```
Destination修改`/etc/cloudera-scm-server/db.properties`，可以跑腳本處理
```
/opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm
```
All Hosts修改`/etc/cloudera-scm-agent/config.ini`，把`server_host`改成Destination Host IP
```
vi /etc/cloudera-scm-agent/config.ini
```
- 範例：
  ```
  [General]
  # Hostname of the CM server.
  #server_host=172.16.1.57
  server_host=172.16.1.64
  ```
All Hosts重啟Agent
```
systemctl restart cloudera-scm-agent
```
Source關閉並停用cloudera-scm-server
```
systemctl stop cloudera-scm-server
systemctl disable cloudera-scm-server
```
Source複製相關descriptor，例如Isilon的`PowerScale-1.0.0.jar `
```
scp /opt/cloudera/cm/csd/PowerScale-1.0.0.jar 172.16.1.64:/opt/cloudera/cm/csd/PowerScale-1.0.0.jar
```
Destination廢棄無用descriptor，例如Isilon的`ISILON-7.3.1.jar `
```
mv /opt/cloudera/cm/csd/ISILON-7.3.1.jar /opt/cloudera/cm/csd/ISILON-7.3.1.jar.deprecated
```
Destination開啓並啟用cloudera-scm-server
```
systemctl start cloudera-scm-server
systemctl enable cloudera-scm-server
```
上瀏覽器查看Cloudera Manager是否正常運作