# 一、[前情提要](#前情提要)
# 二、[設定SSH無密碼登入](#設定ssh無密碼登入)
# 三、[OS設定](#os設定)
# 四、[建置NTP Server](#建置ntp-server)
# 五、[Hadoop外部資料庫安裝建置 ](#hadoop外部資料庫安裝建置)
# 六、[建置YUM Local Repository](#建置yum-local-repository)
# 七、[安裝Cloudera Manager](#安裝cloudera-manager)
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
（以下以我的lab示範）
離線安裝，完全使用本地的Repo
盡量只在Master 1上操作

### OS
  - Red Hat Enterprise Linux 8.2
### 主要套件
  - Cloudera Manager 7.4.4
  - Cloudera Data Platform Private Cloud Base 7.1.7
  - PostgreSQL 12.5

### 節點資訊
|node|FQDN|IP|core|memory|
|-|-|-|-|-|
|Master 1|malvin-cdp-m1-2111.example.com|172.16.1.246|2|8|
|Master 2|malvin-cdp-m2-2111.example.com|172.16.1.247|2|8|
|Worker 1|malvin-cdp-s1-2111.example.com|172.16.1.248|2|8|

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
  172.16.1.246
  172.16.1.247
  172.16.1.248
  ```
### 手動填入每台主機的FQDN
```
vi hostlist
```
- 內容：
  ```
  malvin-hdp2-m1-2111.example.com
  malvin-hdp2-m2-2111.example.com
  malvin-hdp2-s1-2111.example.com
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
> (rhel8)
> 需要先一個一個`ssh`留下footprint並輸入密碼登入後，`sshpass`才能正常運作
> ![](https://i.imgur.com/kzi93oo.png)
```
while read h;do sshpass -p {password} ssh $h "ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''" </dev/null; done <hostlist
while read h; do sshpass -p {password} ssh $h "cat /root/.ssh/id\_rsa.pub" </dev/null; done <hostlist >>/root/.ssh/authorized_keys
while read h; do sshpass -p {password} scp /root/.ssh/authorized\_keys $h:/root/.ssh/authorized\_keys; done <hostlist >>/root/.ssh/authorized_keys
```

### 修改ssh設定檔`/etc/ssh/sshd_config`與`/etc/ssh/ssh_config`
修改`/etc/ssh/sshd_config`
```
vi /etc/ssh/sshd_config
```
- 修改內容：
  ```
  PermitRootLogin yes
  PubkeyAuthentication yes
  PasswordAuthentication yes
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
> (rhel8)
> 使用ssh檢查會有問題
> ![](https://i.imgur.com/QpaOHVP.png)
```
./sshall.sh 'locale'
```
### 停用SELinux
> 注意：
> 此設定須在reboot後確認是否生效

> (rhel8)
>  *設定檔路徑不一樣！！！*
>  `/etc/sysconfig/selinux`還留著，但是沒有效果
```
# 手動
vi /etc/selinux/config
```
- 修改內容：將`SELINUX`改為`disabled`
  ```
  SELINUX=disabled
  ```
```
./scpall.sh /etc/selinux/config
```
```
# 自動
./sshall.sh 'sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config'
```
```
./sshall.sh "cat /etc/selinux/config | grep 'SELINUX=disabled'"
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
./sshall.sh 'echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6'
./sshall.sh 'echo 1 > /proc/sys/net/ipv6/conf/default/disable_ipv6'
./sshall.sh "echo 'net.ipv6.conf.all.disable_ipv6 = 1' >> /etc/sysctl.conf"
./sshall.sh "echo 'net.ipv6.conf.default.disable_ipv6 = 1' >> /etc/sysctl.conf"
```
確認
```
./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable_ipv6; cat /proc/sys/net/ipv6/conf/default/disable_ipv6; cat /etc/sysctl.conf | grep disable_ipv6'
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
### 停用NUMA
```
vi /etc/default/grub
```
- 加上`numa=off`
  ```
  GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet numa=off"
  ```
```
./scpall.sh /etc/default/grub
```
確認
```
./sshall.sh 'grep -i numa /etc/default/grub'
````
產生新的GRUB設定檔
```
./sshall.sh 'grub2-mkconfig -o /etc/grub2.cfg'
```
### 重新啟動確認設定
```
./sshall.sh 'sh allhost_ping.sh'
./sshall.sh 'sh allhost_ssh.sh'
./sshall.sh 'cat success_ssh'
./sshall.sh locale
./sshall.sh sestatus
./sshall.sh 'systemctl is-active firewalld.service; systemctl is-enabled firewalld.service'
./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable_ipv6; cat /proc/sys/net/ipv6/conf/default/disable_ipv6; cat /etc/sysctl.conf | grep disable_ipv6'
./sshall.sh 'sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness'
./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'
./sshall.sh 'dmesg|grep -i numa'
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
cp /var/www/html/cloudera-repos/cm7/RPMS/x86_64/openjdk8-8.0+232_9-cloudera.x86_64.rpm .
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
```
./sshall.sh 'source /etc/profile'
```
登出root的shell，重新登入後確認`$JAVA_HOME`
```
./sshall.sh "echo $JAVA_HOME" # 必須要是雙引號
```
確認`java`與`javac`版本
> 需要找更好的方式確認，可能會command not found
```
./sshall.sh "export PATH=$PATH/bin;java -version;javac -version"
```
## 建置NTP Server
> 若未安裝ntp，則挪到YUM Local Repository建置後

> (rhel8)
> 沒有NTP Server了！`chronyd`取而代之

修改時區
```
./sshall.sh 'timedatectl set-timezone Asia/Taipei'
```
確認`chronyd`
```
./sshall.sh 'systemctl is-active chronyd; systemctl is-enabled chronyd'
```
### 設定Server
修改`/etc/chrony.conf`
```
vi /etc/chrony.conf
```
- 修改內容：Set its value to the **network** or **subnet** address from which the clients are allowed to connect.
- 例如：
  ```
  #pool 2.rhel.pool.ntp.org iburst # 註解預設的NTP Server pool
  server malvin-cdp-m1-2111.example.com
  local stratum 10 # 內網的NTP Server依賴自己主機的時間
  allow 172.16.1.0/24
  ```
### 設定Client
- 修改內容：註解預設用來校時的NTP Server，加上master 01的FQDN
  ```
  server {NTP_SERVER_FQDN}
  ```
- 例如：
  ```
  #pool 2.rhel.pool.ntp.org iburst
  server malvin-cdp-m1-2111.example.com
  ```

啟用
```
./sshall.sh 'systemctl restart chronyd'
```
確認
```
./sshall.sh 'systemctl is-active chronyd; systemctl is-enabled chronyd'
```
確認校時狀況
```
./sshall.sh 'chronyc sources'
./sshall.sh 'chronyc clients'
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
修改`/etc/yum.repos.d/oslocal.repo`
```
vi /etc/yum.repos.d/oslocal.repo
```
- 修改內容：
  ```
  [rhel8-base-local]
  name=Red Hat Enterprise Linux 8.2 BaseOS Local Repo
  baseurl=file:///cdrom/BaseOS
  enabled=1
  gpgcheck=1
  gpgkey=file:///cdrom/RPM-GPG-KEY-redhat-release
  
  [rhel8-appstream-local]
  name=Red Hat Enterprise Linux 8.2 AppStream Local Repo
  baseurl=file:///cdrom/AppStream
  enabled=1
  gpgcheck=1
  gpgkey=file:///cdrom/RPM-GPG-KEY-redhat-beta
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
yum install -y yum-utils createrepo
```
將ISO中的RPM同步到HTTP Server
> (rhel8)
> yum-utils-4.0
> OPTIONS重大不同！
> `-r`->`--repo`
```
# 先確認已經掛載ISO，並做好/etc/yum.repos.d/oslocal.repo
cd /var/www/html
reposync --repo rhel8-base-local
reposync --repo rhel8-appstream-local
```
將目錄改名
```
mv /var/www/html/rhel8-base-local/Packages /var/www/html/rhel8-base-local/RPMS
mv /var/www/html/rhel8-appstream-local/Packages /var/www/html/rhel8-appstream-local/RPMS
```
製作repodata
```
createrepo /var/www/html/rhel8-base-local/
createrepo /var/www/html/rhel8-appstream-local/
```
修改`/etc/yum.repos.d/oslocal.repo`
```
vi /etc/yum.repos.d/oslocal.repo
```
- 修改內容：
  ```
  [rhel8-base-local]
  name=Red Hat Enterprise Linux 8.2 BaseOS Local Repo
  baseurl=http://malvin-cdp-m1-2111.example.com/rhel8-base-local/
  enabled=1
  gpgcheck=0
  
  [rhel8-appstream-local]
  name=Red Hat Enterprise Linux 8.2 AppStream Local Repo
  baseurl=http://malvin-cdp-m1-2111.example.com/rhel8-appstream-local/
  enabled=1
  gpgcheck=0
  ```

將CDP安裝包複製到HTTP Server
```
mkdir -p /var/www/html/cloudera-repos/cm7
tar xvfz cm7.4.4-redhat8.tar.gz -C /var/www/html/cloudera-repos/cm7 --strip-components=1
chmod -R ugo+rX /var/www/html/cloudera-repos/cm7
chown -R root:root /var/www/html/cloudera-repos/cm7
```
修改`/etc/yum.repos.d/cloudera-manager.repo`
```
vi /etc/yum.repos.d/cloudera-manager.repo
```
- 修改內容：
  ```
  [cloudera-manager-7.4.4]
  name=Cloudera Manager 7.4.4
  baseurl=http://malvin-cdp-m1-2111/cm7.4.4
  enabled=1
  gpgcheck=1
  gpgkey=http://malvin-cdp-m1-2111/cm7.4.4/RPM-GPG-KEY-cloudera
  ```
  
將修改後的repo送到每台主機
```
./scpall.sh /etc/yum.repos.d/oslocal.repo
./scpall.sh /etc/yum.repos.d/cloudera-manager.repo
```
確認
```
./sshall.sh 'yum clean all'
./sshall.sh 'yum repolist'
./sshall.sh 'yum install bash' # BaseOS
./sshall.sh 'yum install gcc' # AppStream
./sshall.sh 'yum install cloudera-manager-server' # Cloudera Manager
```
# Hadoop外部資料庫安裝建置
> 注意：不同版本的PostgreSQL其套件名稱、服務名稱與設定檔的路徑各不相同，設定前需要確認清楚

安裝PostgreSQL12
> 注意：順序！！
```
yum install postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
yum install postgresql12-12.5-1PGDG.rhel8.x86_64.rpm
yum install postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm
```
初始化
```
/usr/pgsql-12/bin/postgresql-12-setup initdb
```
啟用
```
systemctl enable postgresql-12
systemctl start postgresql-12
```
修改`/var/lib/pgsql/12/data/pg_hba.conf`
```
vi /var/lib/pgsql/12/data/pg_hba.conf
```
- 修改內容：local連線直接trust，其他IPv4的連線改用md5驗證
  ```
  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  # "local" is for Unix domain socket connections only
  local   all             all                                     trust
  # IPv4 connections:
  host    all             all             0.0.0.0/0               md5
  ```

修改`/var/lib/pgsql/12/data/postgresql.conf`
```
vi /var/lib/pgsql/12/data/postgresql.conf
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
systemctl restart postgresql-12
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
安裝python2
> Cloudera Manager相依python2，rhel8預設只有安裝python3，因此要手動安裝

```
yum install -y python27
```
安裝
```
yum install -y cloudera-manager-server
```
如果沒抓到GPGKEY
```
rpm --import /var/www/html/cm7.4.4/RPM-GPG-KEY-cloudera
```
連結外部資料庫
```
/opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm
```
啟動
```
systemctl start cloudera-scm-server
```
> 因為服務肥大，所以要等待約兩分鐘才能啟動完成，過程中可以確認port有沒有開了：
> `lsof -nPi|grep 718`

使用瀏覽器查看`http://{MASTER_01_FQDN}:7180`
（確認開啟瀏覽器的主機是否能解析到{MASTER_01_FQDN}，否則改用IP訪問）
![](https://i.imgur.com/NEnqpE6.png)
使用預設的帳號admin與密碼admin登入，確認Cloudera Manager可不可以正常運作吧！

# Hadoop Stack
CMS
- Alert Publisher
- Event Server
- Host Monitor
- Service Monitor

|Component|Role|Master 1|Master 2|Worker 1|Worker2|
|-|-|-|-|-|-|
|Cloudera|CM Server|✓|-|-|-|
|Cloudera|CM Agent|✓|✓|✓|✓|
|Cloudera|CMS|✓|-|-|-|
|ZooKeeper|Server|✓|✓|✓|-|
|HDFS|NameNode|✓|-|-|-|
|HDFS|Secondary NameNode|-|✓|-|-|
|HDFS|JournalNode|✓|✓|✓|-|
|HDFS|DataNode|✓|✓|✓|✓|
|YARN|ResourceManager|-|✓|-|-|
|YARN|NodeManager|✓|✓|✓|✓|
|MapReduce|JobHistory Server|-|✓|-|-|
|Hive|Hive MetaStore|-|✓|-|-|
|Hive on Tez|Hive Server 2|-|✓|-|-|
|HUE|Server|-|✓|-|-|
|Spark|History Server|-|✓|-|-|
|Oozie|Server|-|✓|-|-|


# 套件
> 注意：如果是內網不能連外的環境，應事先下載與OS相容的套件RPM，避免安裝過程中還要花時間找套件

> \*標記為`rhel-8.2-x86_64-dvd.iso`沒有的RPM
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
- JDK，直接使用CM tarball附的JDK*
  ```
  yum install -y openjdk8-8.0+232_9-cloudera.x86_64.rpm
  ```
- external database，以PostgreSQL12為例*
  > 注意安裝順序

  ```
  yum install postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
  yum install postgresql12-12.5-1PGDG.rhel8.x86_64.rpm
  yum install postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm
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
- CM 7.4.4 RPMs
    > 需要有購買CDP License的CLOUDERA帳戶才能下載
    
    > 此tarball已經包含`allkeys.asc`與`repod.xml`，也有附上JDK
  ```
  wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz
  wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz.md5
  wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz.sha256

  ```
- CDH 7.1.7 Parcels
    > 需要有購買CDP License的CLOUDERA帳戶才能下載
  ```
  wget --user {cloudera_user} --password {cloudera_password} --recursive --no-parent --no-host-directories https://archive.cloudera.com/p/cdh7/7.1.7.0/parcels/
  ```
  ```
  # 一整包幾十GB，可以只挑需要的版本下載，以RedHat8為例
  wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel
  wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel.sha1
  wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel.sha256
  ```
- PostgreSQL 12
  ```
  wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm
  wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
  wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-devel-12.5-1PGDG.rhel8.x86_64.rpm
  ```
### 選用檔案
- OS ISO，以Red Hat Enterprise Linux 8.2 DVD為例（需要申請Red Hat帳號並登入才能下載）
  https://developers.redhat.com/content-gateway/file/rhel-8.2-x86_64-dvd.iso

