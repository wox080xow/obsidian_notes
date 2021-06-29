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
3 VMs
CentOS7.6

|node|FQDN|IP|core|memory|
|-|-|-|-|-|
|?|?|?|8|64G|
|?|?|?|8|16G|
|?|?|?|8|16G|

Meso建議先Master兼職Worker
# Linux 作業系統參數設定
### *每台主機*改成相同密碼
```
passwd
```
## 設定SSH無密碼登入
### 手寫填入每台主機的IP
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
vi /etc/ssh/ssh_config
```
- 修改內容：
  ```
  Host *
  StrictHostKeyChecking no
  ```
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
### ping測試腳本
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
### ssh測試腳本
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
## OS設定
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
### 修改vm.swappiness為1
按照官方建議值，將Linux Kernal Parameter`vm.swappiness`調降為`1`
```
./sshall.sh 'sysctl -w vm.swappiness=1 > /dev/null; echo 1 > /proc/sys/vm/swappiness; echo vm.swappiness=1 >> /etc/sysctl.conf'
```
確認
```
./sshall.sh 'sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness'
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
./sshall.sh "echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.d/rc.local; echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local; chmod +x /etc/rc.d/rc.local; tail -n 2 /etc/rc.d/rc.local"
```
### 安裝JDK8
安裝cloudera提供的JDK
```
./scpall.sh {path_to_jdk_rpm}
```
```
./sshall.sh 'yum install -y {path_to_jdk_rpm}'
```
例如：
```
cp cdp/CM/7.3.1/redhat7/yum/RPMS/x86_64/openjdk8-8.0+232_9-cloudera.x86_64.rpm .
./scpall.sh openjdk8-8.0+232_9-cloudera.x86_64.rpm
./sshall.sh 'yum install -y openjdk8-8.0+232_9-cloudera.x86_64.rpm'
```
確認JDK目錄名稱
```
ls /usr/lib/jvm/
ls /usr/java/
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
  export JAVA_HOME=/usr/java/jdk1.8.0_232-cloudera
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
確認`$JAVA_HOME`
```
./sshall.sh 'source /etc/profile'
./sshall.sh "echo $JAVA_HOME" # 必須要是雙引號
```
登出root的shell，重新登入後確認`java`與`javac`版本
> 需要找更好的方式確認，可能會command not found
```
./sshall.sh "export PATH=$PATH/bin;java -version;javac -verison"
```
## 建置NTP Server
> 群創若有自己的內網NTP Server，也可以直接使用他們的
> 若未安裝ntp，則挪到YUM Local Repository建置後

停用chronyd
```
./sshall.sh 'systemctl stop chronyd; systemctl disable chronyd'
```
確認
```
./sshall.sh 'systemctl is-active chronyd; systemctl is-enabled chronyd'
```
安裝NTP Server
```
./sshall.sh 'yum install -y ntp'
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
## 重啟後驗證設定正確
```
./sshall.sh 'sh /root/ping_allhosts.sh'
./sshall.sh 'locale'
./sshall.sh 'sestatus'
./sshall.sh 'systemctl is-active firewalld.service; systemctl is-enabled firewalld.service'
./sshall.sh 'ntpstat'
./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable_ipv6; cat /proc/sys/net/ipv6/conf/default/disable_ipv6; cat /etc/sysctl.conf | grep ipv6'
./sshall.sh 'sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness'
./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'
./sshall.sh 'java -version; javac -version'
```

# 建置YUM Local Repository
> 注意：如果需要透過外網安裝套件，建議此步驟挪至安裝Ambari前完成

備份原本的YUM Repository
```
./sshall.sh 'mkdir /etc/yum.repos.d/original_repos'
./sshall.sh 'mv /etc/yum.repos.d/* /etc/yum.repos.d/original_repos'
```
創建掛載用的目錄
```
mkdir /cdrom
```
掛載
```
mount CentOS-7-x86_64-Minimal-2009.iso /cdrom 
```
確認掛載成功
```
ll /cdrom
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

將CDP安裝包複製到HTTP Server
```
cp -r cdp/CM /var/www/html
cp -r cdp/CDH /var/www/html
```
製作repodata
```
createrepo /var/www/html/CM/7.3.1/
```
修改`/etc/yum.repos.d/cloudera-manager.repo`
```
vi /etc/yum.repos.d/cloudera-manager.repo
```
- 修改內容：
  ```
  [cloudera-manager]
  name=Cloudera Manager, Version 7.3.1
  baseurl=http://{REPO_SERVER_FQDN}/CM/7.3.1
  gpgcheck=1
  gpgkey=http://{REPO_SERVER_FQDN}/CM/7.3.1/redhat7/yum/RPM-GPG-KEY-cloudera
  ```
  
將PostgreSQL解壓縮至HTTP Server
```
tar xvf pgdg11.tar.gz -C /var/www/html/
```
製作repodata
```
createrepo /var/www/html/pgdg11/
```
製作
```
vi /etc/yum.repos.d/pgdg11.repo
```
- 修改內容：
  ```
  [pgdg11]
  name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch
  baseurl=http://{REPO_SERVER_FQDN}/pgdg11
  enabled=1
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
  ```
複製PostgreSQL的gpgkey
```
cp cdp/PostgreSQL/RPM-GPG-KEY-PGDG /etc/pki/rpm-gpg
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
```
將修改後的repo送到每台主機
```
./scpall.sh /etc/yum.repos.d/oslocal.repo
./scpall.sh /etc/yum.repos.d/cloudera-manager.repo
./scpall.sh /etc/yum.repos.d/pgdg11.repo
```
清空YUM快取並列出repolist
```
./sshall.sh 'yum clean all; yum repolist'
```
# PostgreSQL安裝建置 (Hadoop 外部資料庫)
> 注意：不同版本的PostgreSQL其套件名稱、服務名稱與設定檔的路徑各不相同，設定前需要確認清楚

安裝PostgreSQL11
```
yum install -y postgresql11-server
```
初始化
```
/usr/pgsql-11/bin/postgresql-11-setup initdb
```
啟用
```
systemctl enable postgresql-11
systemctl start postgresql-11
```
修改`/var/lib/pgsql/11/data/pg_hba.conf`
```
vi /var/lib/pgsql/11/data/pg_hba.conf
```
- 修改內容：local連線直接trust，其他IPv4的連線改用md5驗證
  ```
  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  # "local" is for Unix domain socket connections only
  local   all             all                                     trust
  # IPv4 connections:
  host    all             all             0.0.0.0/0               md5
  ```

修改`/var/lib/pgsql/11/data/postgresql.conf`
```
vi /var/lib/pgsql/11/data/postgresql.conf
```
- 修改內容：
  ```
  listen_addresses = '*'
  shared_buffers = 256MB
  wal_buffers = 8MB
  checkpoint_completion_target = 0.9
  ```

重啟PostgresSQL
```
systemctl restart postgresql-11
```
建立USER與DATABASE
```
su - postgres
psql
```
- PostgreSQL Shell
  ```
  CREATE ROLE scm LOGIN PASSWORD 'scm';
  CREATE DATABASE scm OWNER scm ENCODING 'UTF8';

  CREATE ROLE rman LOGIN PASSWORD 'rman';
  CREATE DATABASE rman OWNER rman ENCODING 'UTF8';

  # 以下視使用到的Component建立
  CREATE ROLE hue LOGIN PASSWORD 'hue';
  CREATE DATABASE hue OWNER hue ENCODING 'UTF8';

  CREATE ROLE hive LOGIN PASSWORD 'hive';
  CREATE DATABASE metastore OWNER hive ENCODING 'UTF8';

  CREATE ROLE oozie LOGIN PASSWORD 'oozie';
  CREATE DATABASE oozie OWNER oozie ENCODING 'UTF8';

  CREATE ROLE das LOGIN PASSWORD 'das';
  CREATE DATABASE das OWNER das ENCODING 'UTF8';

  CREATE ROLE schemaregistry LOGIN PASSWORD 'schemaregistry';
  CREATE DATABASE schemaregistry OWNER schemaregistry ENCODING 'UTF8';

  CREATE ROLE smm LOGIN PASSWORD 'smm';
  CREATE DATABASE smm OWNER smm ENCODING 'UTF8';
  
  \q
  ```
回到root使用者！
```
exit
```

# 安裝Cloudera Manager
安裝
```
yum install -y cloudera-manager-server
```
如果沒抓到GPGKEY
```
rpm --import /var/www/html/CM/7.3.1/redhat/yum/RPM-GPG-KEY-cloudera
```
連結外部資料庫
```
/opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm
```
啟動
```
systemctl start cloudera-scm-server
```
> 因為服務肥大，所以要等待約兩分鐘才能啟動完成，使用瀏覽器查看`http://{MASTER_01_FQDN}:7180`（確認開啟瀏覽器的主機是否能解析到{MASTER_01_FQDN}，否則改用IP訪問），確認Cloudera Manager可不可以正常使用，並用預設的帳號admin與密碼admin登入吧！
# 套件
> 注意：如果是內網不能連外的環境，應事先下載與OS相容的套件RPM，避免安裝過程中還要花時間找套件

> \*標記為`CentOS-7-x86_64-Everything-2009.iso`沒有的RPM
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
  - oracle JDK*
    > 注意：需要事先下載JDK的RPM
    ```
    yum install -y {path_to_jdk_rpm}
    ```
- 外部database，以PostgreSQL9.6為例*
  ```
  yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
  yum install -y postgresql96-server
  ```
- Ambari*
  > 注意：需要事先建置Local Repository
  ```
  yum install -y ambari-server
  ```
 
## 方便工具
- ansible*
  ```
  yum install -y ansible
  ```
- sshpass*
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
- telnet
  ```
  yum install -y telnet
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
- CM 7.3.1 RPMs
  ```
  wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/cm7/7.3.1/repo-as-tarball/cm7.3.1-redhat7.tar.gz
  ```
- CDH 7.1.6 RPMs
  ```
  wget --user {cloudera_user} --password {cloudera_password} --recursive --no-parent --no-host-directories https://archive.cloudera.com/p/cdh7/7.1.6.0/parcels/
  
  # 一整包幾十GB，可以只挑需要的版本下載，以RedHat7為例
  wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/cdh7/7.1.6.0/parcels/CDH-7.1.6-1.cdh7.1.6.p0.10506313-el7.parcel
  wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/cdh7/7.1.6.0/parcels/CDH-7.1.6-1.cdh7.1.6.p0.10506313-el7.parcel.sha1
  wget --user {cloudera_user} --password {cloudera_password} https://archive.cloudera.com/p/cdh7/7.1.6.0/parcels/CDH-7.1.6-1.cdh7.1.6.p0.10506313-el7.parcel.sha256
  ```
  > 需要有購買CDP License的CLOUDERA帳戶才能下載
### 選用檔案
- OS ISO，以CentOS 7.9 Minimal為例
  ```
  wget http://ftp.twaren.net/Linux/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso
  ```

