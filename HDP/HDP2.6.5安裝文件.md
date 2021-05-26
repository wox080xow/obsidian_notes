# 一、[前情提要](#前情提要)
# 二、[設定SSH無密碼登入](#設定ssh無密碼登入)
# 三、[OS設定](#os設定)
# 四、[建置NTP Server](#建置ntp-server)
# 五、[建置Ambari外部資料庫](#建置ambari外部資料庫)
# 六、[建置YUM Local Repository](#建置yum-local-repository)
# 七、[安裝Ambari](#安裝ambari)
# 附錄、[套件](#套件)
# 附錄、[檔案](#檔案)
> ### 變數:
> - {password} 確認每台主機的root密碼一致
> - {path_to_jdk_rpm} 事先下載的JDK
> - {path_to_jdk} JDK目錄作為$JAVA_HOME路徑
> - {NTP_SERVER_FQDN} 就是master 01的FQDN
> - {JDK_RPM_LOCATION} 自己下載的JDK RPM
> - {REPO_SERVER_FQDN} 就是master 01的FQDN
> - {cloudera_user} 有購買HDP License的CLOUDERA帳號
> - {cloudera_password} 有購買HDP License的CLOUDERA密碼

# 前情提要
### 基本資訊
4 VMs on ESXi
CentOS7.9(CentOS-7-x86_64-Minimal-2009.iso)

|node|FQDN|IP|
|-|-|-|
|master 01|malvin-hdp2-m1.example.com|172.16.1.61|
|master 02|malvin-hdp2-m2.example.com|172.16.1.62|
|worker 01|malvin-hdp2-s1.example.com|172.16.1.63|
|worker 02|malvin-hdp2-s2.example.com|172.16.1.64|


### *每台VM*設定hostname與IP
修改hostname
```
hostnamectl set-hostname malvin-hdp2-m1
```
確認
```
hostname
```
修改IP
```
vi /etc/sysconfig/network-scripts/ifcfg-ens192
```
確認
```
ip addr
```
重啟後再確認一次
```
reboot
hostname
ip addr
```
> 注意：以下均使用*master 01*進行設定
# 設定SSH無密碼登入
### 手寫填入個台主機的IP
```
vi iplist
```
- 內容：
  ```
  172.16.1.61
  172.16.1.62
  172.16.1.63
  172.16.1.64
  ```
### 手動填入每台主機的FQDN
```
vi hostlist
```
- 內容：
  ```
  malvin-hdp2-m1.example.com
  malvin-hdp2-m2.example.com
  malvin-hdp2-s1.example.com
  malvin-hdp2-s2.example.com
  ```
### 將每台主機加入`/etc/hosts`
```
cut hostlist -d '.' -f 1 > aliaslist
paste -d ' ' iplist hostlist aliaslist > hostsupdate
cat hostsupdate >> /etc/hosts
hostname -f
```
### 安裝sshpass
```
yum install -y sshpass
```
> 這有bug...
> ```
> while read h;do sshpass -p {password} ssh $h echo "$(hostname -i) $(hostname -f) $(hostname)" </dev/null; done < iplist
> ```
> ![](https://i.imgur.com/psk5Wxu.png)
### 製作每台主機的ssh金鑰，並將ssh公鑰發到每台主機
```
while read h;do sshpass -p {password} ssh $h "ssh-keygen -f /root/.ssh/id_rsa -t rsa -N''" </dev/null; done <hostlist
while read h; do sshpass -p {password} ssh $h "cat /root/.ssh/id\_rsa.pub" </dev/null; done <hostlist >>/root/.ssh/authorized_keys
while read h; do sshpass -p {password} scp /root/.ssh/authorized\_keys $h:/root/.ssh/authorized\_keys; done <hostlist >>/root/.ssh/authorized_keys
```

### 修改ssh設定檔`/etc/ssh/sshd_config`與`/etc/ssh/ssh_config`
修改`/etc/ssh/sshd_config`
手自動皆可，擇一進行
```
# 手動
vi /etc/ssh/sshd_config
```
- 修改內容：
  ```
  PermitRootLogin yes
  PubkeyAuthentication yes
  PasswordAuthentication yes
  ```
```
# 自動
sed -i "s/PermitRootLogin no/PermitRootLogin yes/; s/PubkeyAuthentication yes/PubkeyAuthentication no/; s/PasswordAuthentication no/PasswordAuthentication yes/" /etc/ssh/sshd_config
```
修改`/etc/ssh/ssh_config`
```
# 手動
vi /etc/ssh/ssh_config
```
- 修改內容：
  ```
  Host *
  StrictHostKeyChecking no
  ```

```
# 自動
sed -i 's/# Host */Host */; s/#   StrictHostKeyChecking ask/  StrictHostKeyChecking no/' /etc/ssh/ssh_config
```
> 可以再優化：
>  - 抓關鍵字
>  - 針對不符合預期設定的關鍵字整行修改
```
service sshd restart
```
### ssh到所有主機的腳本
```
vi sshall.sh
```
- 內容：
  ```
  while read h
  do
    #echo $h
    #echo $1
    ssh $h "hostname;$1" </dev/null
    #sshpass -p asdf1234 ssh $h "hostname;$1" </dev/null
  done <hostlist
  #done <iplist
  ```
### scp到所有主機的腳本
```
vi scpall.sh
```
- 內容：
  ```
  while read h
  do
    ssh $h hostname </dev/null
    #sshpass -p asdf1234 ssh $h hostname </dev/null
    scp $1 $h:$1 </dev/null
    #sshpass -p asdf1234 scp $1 $h:$1 </dev/null
  done <hostlist
  #done <iplist
  ```
> 可以再優化：一次傳送多份檔案
### ping測試
```
vi allhost_ping.sh
```
- 內容：
  ```
  while read h
  do
      ping -c 4 $h | grep -A 1 'ping statistics'
  done < hostlist
  
  while read i
  do
      ping -c 4 $i | grep -A 1 'ping statistics'
  done < iplist
  ```
```
chmod a+x allhost_ping.sh
./scpall.sh allhost_ping.sh
./sshall.sh 'sh allhost_ping.sh'
```
### ssh測試
```
vi allhost_ssh.sh
```
- 內容：
  ```
  while read h
  do
      now=$(date +'%Y-%m-%d %H:%M:%S')
      ssh $h "echo $now login from $HOSTNAME >> /root/success_ssh" </dev/null
  done < /root/hostlist
  ```
```
chmod a+x allhost_ssh.sh
./scpall.sh allhost_ssh.sh
./sshall.sh 'sh allhost_ssh.sh'
```
# OS設定
### 設定語言
手自動皆可，擇一進行

```
# 手動
vi /etc/profile
```
- 修改內容：在檔尾加上指定語言為 `en_US.UTF-8`
  ```
  export LANGUAGE=en_US.UTF-8
  export LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  ```
```
./scpall.sh /etc/profile
```

```
# 自動
./sshall.sh 'echo -e "export LANGUAGE=en_US.UTF-8\nexport LANG=en_US.UTF-8\nexport LC_ALL=en_US.UTF-8" >>/etc/profile'
```
source一下`/etc/profile`
```
./sshall.sh 'source /etc/profile'
```
登出root的shell，重新登入後確認
```
./sshall.sh 'locale'
```
### 停用SELinux
> 注意：此設定須在reboot後確認是否生效
```
# 手動
vi /etc/sysconfig/selinux
```
- 修改內容：將`SELINUX`改為`disabled`
  ```
  SELINUX=disabled
  ```
```
./scpall.sh /etc/sysconfig/selinux
```
```
# 自動
./sshall.sh 'sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux''
```
> 單引號可以包單引號！
```
./sshall.sh 'cat /etc/sysconfig/selinux | grep 'SELINUX=disabled'
```
### 關閉及停用防火牆
關閉及停用
```
./sshall.sh 'systemctl stop firewalld.service; systemctl disable firewalld.service'
```
確認
```
./sshall.sh 'systemctl is-active firewalld.service; systemctl is-enabled firewalld.service'
```
### 停用IPv6
停用
```
./sshall.sh 'echo 1 > /proc/sys/net/ipv6/conf/all/disable\_ipv6; echo 1 > /proc/sys/net/ipv6/conf/default/disable\_ipv6; echo 'net.ipv6.conf.all.disable\_ipv6 = 1' >> /etc/sysctl.conf; echo 'net.ipv6.conf.default.disable\_ipv6 = 1' >> /etc/sysctl.conf'
```
確認
```
./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable\_ipv6; cat /proc/sys/net/ipv6/conf/default/disable\_ipv6; cat /etc/sysctl.conf | grep ipv6'
```
檢查有無`/etc/netconfig`
```
ls /etc/netconfig
```
若有上述檔案則進行修改
```
vi /etc/netconfig
```
- 修改內容：註解udp6與tcp6兩行
  ```
  #udp6 tpi_clts v inet6 udp - -
  #tcp6 tpi_cots_ord v inet6 tcp - -
  ```
```
./scpall.sh /etc/netconfig
```
> 注意：若需要從外網安裝套件，有些OS預設使用IPv6辨認DNS Server，所以需要修改`/etc/resolv.conf`
>  ```
>  vi /etc/resolv.conf
>  ```
>  - 修改內容：加上Google的DNS Server
>    ```
>    nameserver 8.8.8.8
>    nameserver 8.8.4.4
>    ```
### 修改vm.swappiness為1
按照官方建議值，將Linux Kernal Parameter`vm.swappiness`調降為`1`
```
./sshall.sh 'sysctl -w vm.swappiness=1 > /dev/null; echo 1 > /proc/sys/vm/swappiness; echo vm.swappiness=1 >> /etc/sysctl.conf'
```
確認
```
sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness
```
### 停用transparent_hugepage
停用
```
./sshall.sh 'echo never > /sys/kernel/mm/transparent_hugepage/defrag; echo never > /sys/kernel/mm/transparent_hugepage/enabled'
```
確認
```
./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'
```
設定「重啟後停用」
```
./sshall.sh 'echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.d/rc.local; echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local; chmod +x /etc/rc.d/rc.local; tail -n 2 /etc/rc.d/rc.local'
```
### 安裝JDK8
自己預備或使用open JDK皆可，擇一
```
# 使用自己下載的RPM
./sshall.sh 'yum install -y {path_to_jdk_rpm}'
```
```
# 使用open JDK
yum install -y java-1.8.0-openjdk-devel.x86_64
```
確認
```
./sshall.sh 'java -version; javac -version'
```
確認JDK目錄名稱
```
ls /usr/lib/jvm/
```
設定`$JAVA_HOME`
```
# 手動
vi /etc/profile
```
- 修改內容：將JDK目錄設定為`$JAVA_HOME`
  ```
  export JAVA_HOME={path_to_jdk}
  PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin
  export PATH
  ```
- 例如：
  ```
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_64
  PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin
  export PATH
  ```
```
./scpall.sh '/etc/profile'
```
```
# 自動
./sshall.sh 'echo -e "export JAVA_HOME={path_to_jdk}\nPATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin\nexport PATH"'
```
```
./sshall.sh 'source /etc/profile'
./sshall.sh 'echo $JAVA_HOME'
```
# 建置NTP Server
停用chronyd
```
systemctl stop chronyd; systemctl disable chronyd
```
確認
```
systemctl is-active chronyd; systemctl is-enabled chronyd
```
安裝NTP Server
```
yum install -y ntp
```
修改`/etc/ntp.conf`
```
vi /etc/ntp.conf
```
- 修改內容：註解預設用來校時的NTP Server，加上master 01的FQDN
  ```
  server {NTP_SERVER_FQDN}
  ```
- 例如：
  ```
  #server 0.centos.pool.ntp.org iburst
  #server 1.centos.pool.ntp.org iburst
  #server 2.centos.pool.ntp.org iburst
  #server 3.centos.pool.ntp.org iburst
  server malvin-hdp2-m1.example.com
  ```
傳送給每台VM
```
./scpall.sh /etc/ntp.conf
```
再一次修改`/etc/ntp.conf`
```
vi /etc/ntp.conf
```
- 修改內容：註解預設用來校時的NTP Server，改用本機的NTP Server校時
  ```
  restrict 10.128.0.0 mask 255.255.255.0 nomodify notrap
  #server 0.centos.pool.ntp.org iburst
  #server 1.centos.pool.ntp.org iburst
  #server 2.centos.pool.ntp.org iburst
  #server 3.centos.pool.ntp.org iburst
  server 127.127.1.0
  fudge 127.127.1.0 stratum 8
  ```

啟用
```
./sshall.sh 'systemctl restart ntpd; systemctl enable ntpd'
```
確認
```
./sshall.sh 'systemctl is-active ntpd; systemctl is-enabled ntpd'
```
確認校時狀況
```
./sshall.sh 'ntpstat'
```
> 通常需要稍待一下才會開始校時
# 建置Ambari外部資料庫
> 注意：不同版本的PostgreSQL其套件名稱、服務名稱與設定檔的路徑各不相同，設定前需要確認清楚

下載Postgres的YUM遠端倉庫清單
```
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```
安裝PostgreSQL9.6
```
yum install -y postgresql96-server
```
初始化
```
/usr/pgsql-9.6/bin/postgresql96-setup initdb
```
啟用
```
systemctl enable postgresql-9.6
systemctl start postgresql-9.6
```
修改`/var/lib/pgsql/9.6/data/pg_hba.conf`
```
vi /var/lib/pgsql/9.6/data/pg_hba.conf
```
- 修改內容：local連線直接trust，其他IPv4的連線改用md5驗證
  ```
  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  # "local" is for Unix domain socket connections only
  local   all             all                                     trust
  # IPv4 connections:
  host    all             all             0.0.0.0/0               md5
  ```

修改`/var/lib/pgsql/9.6/data/postgresql.conf`
```
vi /var/lib/pgsql/9.6/data/postgresql.conf
```
- 修改內容：將localhost的IP改為`*`
  ```
  listen_addresses = '*'
  ```

重啟PostgresSQL
```
systemctl restart postgresql-9.6
```
建立USER與DATABASE
```
su - postgres
psql
```
- PostgreSQL Shell
  ```
  CREATE DATABASE ambari;
  CREATE USER ambari WITH PASSWORD 'ambari';
  GRANT ALL PRIVILEGES ON DATABASE ambari TO ambari;
  \connect ambari;
  CREATE SCHEMA ambari AUTHORIZATION ambari;
  ALTER SCHEMA ambari OWNER TO ambari;
  ALTER ROLE ambari SET search\_path to 'ambari', 'public';
  
  CREATE DATABASE hive;
  CREATE USER hive WITH PASSWORD 'hive';
  GRANT ALL PRIVILEGES ON DATABASE hive TO hive;
  \q
  ```
回到root使用者！
```
exit
```
# 建置YUM Local Repository
> 注意：如果需要透過外網安裝套件，建議此步驟挪至安裝Ambari前完成

備份原本的YUM Repository
```
mkdir /etc/yum.repos.d/original_repos
mv /etc/yum.repos.d/* /etc/yum.repos.d/original_repos
```
下載OS ISO
```
wget http://ftp.twaren.net/Linux/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso
```
創建掛載用的目錄
```
mkdir /cdrom
```
掛載
```
mount CentOS-7-x86_64-Minimal-2009.iso /cdrom 
```
確認有CentOS的gpgkey
```
ll /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
修改`/etc/yum.repos.d/oslocal.repo`
```
vi /etc/yum.repos.d/oslocal.repo
```
- 修改內容：
  ```
  [oslocal]
  name=oslocal
  baseurl=file:///cdrom
  enabled=1
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  ```
安裝HTTP Server
```
yum install -y httpd
```
啟用
```
systemctl start httpd
systemctl enable httpd
```
安裝YUM Repository小工具
```
yum installl -y yum-utils createrepo
```
將ISO中的RPM同步到HTTP Server
```
reposync -r /var/www/html/oslocal
```
將目錄改名
```
mv /var/www/html/oslocal/Packages /var/www/oslocal/RPMS
```
製作repodata
```
createrepo /var/www/html/oslocal/
```
修改`/etc/yum.repos.d/oslocal.repo`
```
vi /etc/yum.repos.d/oslocal.repo
```
- 修改內容：
  ```
  [oslocal]
  name=oslocal
  baseurl=http://{REPO_SERVER_FQDN}/oslocal
  enabled=1
  gpgcheck=0
  ```

下載Ambari與HDP的RPM
```
wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/ambari/2.x/2.6.2.59/centos7/ambari-2.6.2.59-7-centos7.tar.gz
wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/HDP/2.x/2.6.5.0/centos7/HDP-2.6.5.0-centos7-rpm.tar.gz
wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/HDP-UTILS/1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz
```
解壓縮至HTTP Server
```
tar xvf ambari-2.6.2.59-7-centos7.tar.gz -C /var/www/html/
mkdir /var/www/html/hdp
tar xvf HDP-2.6.5.0-centos7-rpm.tar.gz -C /var/www/html/hdp
tar xvf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/hdp
```
修改`/etc/yum.repos.d/ambari.repo`
```
vi /etc/yum.repos.d/ambari.repo
```
- 修改內容：
  ```
  [ambari-2.6.2.59]
  name=ambari-2.6.2.59
  baseurl=http://{REPO_SERVER_FQDN}/ambari/centos/2.6.2.59-7
  enabled=1
  gpgcheck=0
  ```
  
修改`/etc/yum.repos.d/hdp.repo`
```
vi /etc/yum.repos.d/hdp.repo
```
- 修改內容：
  ```
  [HDP-2.6.5.0]
  name=HDP-2.6.5.0
  baseurl=http://{REPO_SERVER_FQDN}/hdp/HDP/centos7/2.6.5.0-292/
  enabled=1
  gpgcheck=0
  
  [HDP-UTILS-1.1.0.22]
  name=HDP-UTILS-1.1.0.22
  baseurl=http://{REPO_SERVER_FQDN}/hdp/HDP-UTILS/centos7/1.1.0.22/
  enabled=1
  gpgcheck=0
  ```
將修改後的repo送到每台主機
```
./scpall.sh /etc/yum.repos.d/oslocal.repo
./scpall.sh /etc/yum.repos.d/ambari.repo
./scpall.sh /etc/yum.repos.d/hdp.repo
```
# 安裝Ambari
安裝
```
yum install -y ambari-server
```
下載JDBC
```
wget https://jdbc.postgresql.org/download/postgresql-42.2.20.jar
```
使用JDBC串接PostgreSQL
```
ambari-server setup --jdbc-db=postgres --jdbc-driver=/root/postgresql-42.2.10.jar
```
匯入Ambari使用到的SCHEMA
```
psql -U ambari
```
- PostgreSQL Shell
  ```
  \connect ambari;
  \i /var/lib/ambari-server/resources/Ambari-DDL-Postgres-CREATE.sql;
  \q
  ```
  
設定Ambari Server
```
ambari-server setup
```
Interactive介面，範例如下：
```
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
Do you want to change Oracle JDK [y/n] (n)? y
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1): 3
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_64
Validating JDK on Ambari Server...done.
Checking GPL software agreement...
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
Enter choice (4): 4
Hostname (malvin-hdp2-m1): malvin-hdp2-m1.example.com
Port (5432): 5432
Database name (ambari): ambari
Postgres schema (ambari): ambari
Username (ambari): ambari
Enter Database Password (ambari):
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL against the database to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-Postgres-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
............
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```
啟動
```
ambari-server start
```
> 成功啟動後，趕快用瀏覽器查看`http://{MASTER_01_FQDN}:8080`，確認Ambari可不可以正常使用，並用預設的帳號admin與密碼admin登入吧！
# 套件
> 注意：如果是內網不能連外的環境，應事先下載與OS相容的套件RPM，避免安裝過程中還要花時間找套件
## 必要套件
- ntp
  ```
  yum install -y ntp
  ```
- yum-utils
  ```
  yum install -y yum-utils
  ```
- createrepo
  ```
  yum install -y createrepo
  ```
- HTTP Server
  ```
  yum install -y httpd
  ```
- open JDK與oracle JDK擇一
  - java-1.8.0-openjdk-devel.x86_64
    ```
    yum install -y java-1.8.0-openjdk-devel.x86_64
    ```
  - oracle JDK
> 注意：需要事先下載JDK的RPM
    ```
    yum install -y {path_to_jdk_rpm}
    ```
- 外部database，以PostgreSQL9.6為例
  ```
  yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
  yum install -y postgresql96-server
  ```
- Ambari
> 注意：需要事先建置Local Repository
  ```
  yum install -y ambari-server
  ```
 
## 方便工具
- ansible
  ```
  yum install -y ansible
  ```
- sshpass
  ```
  yum install -y sshpass
  ```
- wget
  ```
  yum install -y wget
  ```
- lsof
  ```
  yum install -y lsof
  ```
- pstree
  ```
  yum install -y psmisc
  ```
- tree
  ```
  yum install -y tree
  ```
# 檔案
> 可以事先下載需要的檔案，加快安裝速度
### 必要檔案
- Ambari 2.6.2.59 RPMs
- HDP 2.6.5 RPMs
```
wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/HDP/2.x/2.6.5.0/centos7/HDP-2.6.5.0-centos7-rpm.tar.gz
wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/HDP-UTILS/1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz
```
> 以上都需要購買HDP License才能下載
### 選用檔案
- OS ISO，以CentOS 7.9 Minimal為例
  ```
  wget http://ftp.twaren.net/Linux/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso
  ```
- oracle JDK
  > 需要註冊ORACLE會員才能下載
- JDBC for PostgreSQL
  > 不同版本的JDK要使用不同的JDBC來串接PostgreSQL
  > 參考：https://jdbc.postgresql.org/download.html
  ```
  wget https://jdbc.postgresql.org/download/postgresql-42.2.20.jar
  ```
