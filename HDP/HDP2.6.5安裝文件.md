> ### 變數:
> - {password} 確認每台主機的root密碼一致
> - {JDK_RPM_LOCATION} 事先下載的JDK
> - {path_to_jdk}作為$JAVA_HOME路徑
> 


# 前情提要
4 VMs on ESXi

|node|FQDN|IP|
|-|-|-|
|master|malvin-hdp2-m1.example.com|172.16.1.61|
|master|malvin-hdp2-m2.example.com|172.16.1.62|
|worker|malvin-hdp2-s1.example.com|172.16.1.63|
|worker|malvin-hdp2-s2.example.com|172.16.1.64|

# SSH無密碼登入
### *每台VM*設定hostnam與IP
```
hostnamectl set-hostname malvin-hdp2-m1
hostname
vi /etc/sysconfig/network-scripts/ifcfg-ens192
ip addr
reboot
```
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
內容：
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
內容：
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

```
./scpall.sh /etc/profile
```

```
# 自動
./sshall.sh 'echo -e "export LANGUAGE=en_US.UTF-8\nexport LANG=en_US.UTF-8\nexport LC_ALL=en_US.UTF-8" >>/etc/profile'
```
```
./sshall.sh 'source /etc/profile'
```
登出root的shell，重新登入後確認
```
./sshall.sh 'locale'
```
### 停用SELinux
> 注意：此設定需在後面步驟reboot確認是否生效
```
# 手動
vi /etc/sysconfig/selinux
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
### 停用 transparent_hugepage
停用
```
./sshall.sh 'echo never > /sys/kernel/mm/transparent_hugepage/defrag; echo never > /sys/kernel/mm/transparent_hugepage/enabled'
```
確認
```
./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'
```
永久停用
```
./sshall.sh 'echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.d/rc.local; echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local; chmod +x /etc/rc.d/rc.local; tail -n 2 /etc/rc.d/rc.local'
```
### 安裝JDK8
自己預備或使用open JDK皆可，擇一
```
# 使用自己下載的RPM
./sshall.sh 'yum install -y {JDK_RPM_LOCATION}'
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
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_6
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
# 建置外部Database
> 注意：不同版本的PostgreSQL其套件名稱、服務名稱與設定檔的路徑各不相同，設定前需要確認清楚
```
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql96-server
/usr/pgsql-9.6/bin/postgresql96-setup initdb
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
# 建置YUM Local Repository
> 注意：如果需要透過外網安裝套件，建議此步驟挪至安裝Ambari前完成
```
yum install -y httpd
```
```
yum install -y yum-utils createrepo
```
從FQDN改用IP
```
sed -i 's/malvin-hdp2-m1.example.com/172.16.1.61/g' /etc/yum.repos.d/*.repo
```
# 安裝Ambari
```
yum install -y ambari-server
```

# 套件
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
- httpd
  ```
  yum install -y httpd
  ```
- java-1.8.0-openjdk-devel.x86_64
  ```
  yum install -y java-1.8.0-openjdk-devel.x86_64
  ```
- oracle JDK
  ```
  yum install -y
  ```
- postgresql96-server
  ```
  yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
  yum install -y postgresql96-server
  ```
- ambari-server
  ```
  yum install -y ambari-server
  ```
  > 注意：需要建置Local Repository
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
```
   37  yum -y install ansible
   46  yum -y install sshpass
  113  yum -y install wget
  122  yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redha
t-repo-latest.noarch.rpm
  123  yum install -y postgresql96-server
  156  ./sshall.sh 'yum -y install ntp'
  214  yum install -y yum-utils createrepo
  215  yum install httpd
  301  yum install java-1.8.0-openjdk-devel.x86_64
  304  ./sshall.sh 'yum install -y java-1.8.0-openjdk-devel.x86_64'
  328  yum install -y ambari-server
```

# 檔案
- postgresql-42.2.20.jar
  ```
  wget https://jdbc.postgresql.org/download/postgresql-42.2.20.jar
  ```
https://jdbc.postgresql.org/download.html