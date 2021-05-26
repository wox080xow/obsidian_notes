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

從FQDN改用IP
```
sed -i 's/malvin-hdp2-m1.example.com/172.16.1.61/g' /etc/yum.repos.d/*.repo
```