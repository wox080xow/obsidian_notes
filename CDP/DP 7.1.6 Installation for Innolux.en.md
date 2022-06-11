# 5. BoC[Emotional synopsis](#前情提要)

# Second, use fire[A code login was installed](#設定ssh無密碼登入)

# Third time[Yao, meat, catch](#os設定)

# Four, four[A set of official names for wooden rice was set up on the river](#建置ntp-server)

# 5. BoC[Installations are made from external repositories ](#hadoop外部資料庫安裝建置)

# Six, six[Build a water war](#建置yum-local-repository)

# Seven, seven[Start confirm settings again](#重新啟動確認設定)

# Eight, eight[Install the equipment](#安裝cloudera-manager)

# Appendix, Record, Liang duo[Suite](#套件)

# Appendix, Record, Liang duo[archive](#檔案)

> ### variable
>
> *   It's very dense
> *   path_to_jdk_rpm Things start with downloads
> *   path_to_jdk Look at the horse record with your eyes JAVA_HOME path
> *   \[That's NTP_SERVER_FQDN.]
> *   Zhang JDK_RPM_LOCATION the wooden oars carried by his men
> *   REPO_SERVER_FQDN \[that's it.]
> *   cloudera_user the name of the flag tent hired
> *   cloudera_password buying, buying, tightening, tightening arsenic

# Emotional synopsis

As my opinion
The front of the car is installed offline, using the local chaos completely
At most, look only in operation

### With badger oss

*   It's all wrapped around silk

### Main kit

*   \[Surname 癸.]
*   It's all wrapped around silk
*   \[The surname Is very well named.]

### Node qualifications

|| Transport | everywhere 〔|||
|-|-|-|-|-|
| |malvin-cdp-m1-2111.example.com||||
| |malvin-cdp-m2-2111.example.com||||
| |malvin-cdp-s1-2111.example.com|||||

# It is the number set by the emperor as the emperor and determined

### *Every mains machine*Changed to be closely proportionate

    passwd

## A code login was installed

### I copied and filled in the important matters of each competent authority

    vi iplist

*   Content: Content
        172.16.1.246
        172.16.1.247
        172.16.1.248

### Every star is agitated, and the drums will sound like horses

    vi hostlist

*   Content: Content
        malvin-hdp2-m1-2111.example.com
        malvin-hdp2-m2-2111.example.com
        malvin-hdp2-s1-2111.example.com

### Add each supervisor to join`/etc/hosts`

    cut hostlist -d '.' -f 1 > aliaslist
    paste -d ' ' iplist hostlist aliaslist > hostsupdate
    cat hostsupdate >> /etc/hosts
    hostname -f

### Installation

    yum install -y sshpass

### It is placed in each competent authority and inspired by each competent authority

> Note;
>
> 1.  If so`ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''`The private key of the production is from good evidence, but it is also not eaten`OPENSSH PRIVATE KEY`Reappointed him`ssh-keygen -f /root/.ssh/id_rsa -m pem -t rsa -b 2048 -N ''`output`RSA PRIVATE KEY`
> 2.  The units that are confirmed
> 3.  Each node head makes a center`/etc/hosts`

> At first, you can't use the combination of loops`sshpass`That's how it is done`sshd` The settings are to be asked
> Three squares:
>
> 1.  `ssh`With parameters`-o StrictHostKeyChecking=no`
> 2.  Just one first`ssh`Leave me merged into the Mithedon Cave
> 3.  Modify each node first`sshd`Set fixed
>     ![](https://i.imgur.com/kzi93oo.png)

    while read h;do sshpass -p {password} ssh -o StrictHostKeyChecking=no $h "ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''" </dev/null; done <hostlist
    while read h; do sshpass -p {password} ssh -o StrictHostKeyChecking=no $h "cat /root/.ssh/id_rsa.pub" </dev/null; done <hostlist >>/root/.ssh/authorized_keys
    while read h; do sshpass -p {password} scp /root/.ssh/authorized_keys $h:/root/.ssh/authorized_keys; done <hostlist

<!---->

    # 沒有sshpass，手動輸入密碼
    while read h;do ssh -o StrictHostKeyChecking=no $h "ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''" </dev/null; done <hostlist
    while read h; do ssh -o StrictHostKeyChecking=no $h "cat /root/.ssh/id_rsa.pub" </dev/null; done <hostlist >>/root/.ssh/authorized_keys
    while read h; do scp /root/.ssh/authorized_keys $h:/root/.ssh/authorized_keys; done <hostlist

"Confirm"

    while read h; do ssh $h hostname </dev/null; done <hostlist

### And I have roots behind me

    vi sshall.sh

*   Content: Content
        while read h
        do
          ssh $h "hostname;$1" </dev/null
        done <hostlist

<!---->

    chmod +x sshall.sh

### But there is no base for the host

    vi scpall.sh

*   Content: Content
        while read h
        do
          ssh $h hostname </dev/null
          scp $1 $h:$1 </dev/null
        done <hostlist

> Re-preferential treatment can be given to indoctrination: multiple files can be sent at once

    chmod +x scpall.sh

### Modified to a fixed bezel`/etc/ssh/sshd_config`Participate`/etc/ssh/ssh_config`

revise`/etc/ssh/sshd_config`

    vi /etc/ssh/sshd_config

*   Ouyang Xiu changed the content
        PermitRootLogin yes
        PubkeyAuthentication yes
        PasswordAuthentication yes

revise`/etc/ssh/ssh_config`

    vi /etc/ssh/ssh_config

*   Ouyang Xiu changed the content
        Host *
        StrictHostKeyChecking no

<!---->

    ./scpall.sh /etc/ssh/sshd_config
    ./scpall.sh /etc/ssh/ssh_config
    ./sshall.sh "service sshd restart"

### Test-taking script

    vi allhost_ping.sh

*   Content: Content
        while read h
        do
            ping -c 4 $h | grep -A 1 'ping statistics'
        done < hostlist

        while read i
        do
            ping -c 4 $i | grep -A 1 'ping statistics'
        done < iplist

<!---->

    chmod a+x allhost_ping.sh
    ./scpall.sh allhost_ping.sh
    ./sshall.sh 'sh allhost_ping.sh'

### Scripts from exams

    vi allhost_ssh.sh

*   Content: Content
        while read h
        do
            now=$(date +'%Y-%m-%d %H:%M:%S')
            ssh $h "echo $now login from $HOSTNAME >> /root/success_ssh" </dev/null
        done < /root/hostlist

<!---->

    chmod a+x allhost_ssh.sh
    ./scpall.sh allhost_ssh.sh
    ./sshall.sh 'sh allhost_ssh.sh'

## Yao, meat, catch

### Suppose he had said something nice

You can do it by hand, choose one to go

    # 手動
    vi /etc/profile

*   Ouyang Xiu changed the content: specifying the language on the end `en_US.UTF-8`
        export LANGUAGE=en_US.UTF-8
        export LANG=en_US.UTF-8
        export LC_ALL=en_US.UTF-8

<!---->

    ./scpall.sh /etc/profile

<!---->

    # 自動
    ./sshall.sh 'echo -e "export LANGUAGE=en_US.UTF-8\nexport LANG=en_US.UTF-8\nexport LC_ALL=en_US.UTF-8" >>/etc/profile'

one time`/etc/profile`

    ./sshall.sh 'source /etc/profile'

Climb the mountain and climb again to identify the back

> （8
> There is a problem to testify
> ![](https://i.imgur.com/QpaOHVP.png)

    ./sshall.sh 'locale'

### Deactivate fishing

> Note;
> The beard set here does have to recognize behind my back whether it is in effect or not

> （8
> *Setting up a fixed road is different!!!*
> `/etc/sysconfig/selinux`Came back to keep, but to no avail

    # 手動
    vi /etc/selinux/config

*   Ouyang Xiu changed the content: Generals`SELINUX`Changed to Li Wei`disabled`
        SELINUX=disabled

<!---->

    ./scpall.sh /etc/selinux/config

<!---->

    # 自動
    ./sshall.sh 'sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config'

<!---->

    ./sshall.sh "cat /etc/selinux/config | grep 'SELINUX=disabled'"

### Shut the door and lock to stop using as a firewall

Closed and deactivated

    ./sshall.sh 'systemctl stop firewalld.service; systemctl disable firewalld.service'

"Confirm"

    ./sshall.sh 'systemctl is-active firewalld.service; systemctl is-enabled firewalld.service'

### More wooden bridges are discontinued

Discontinue use

    ./sshall.sh 'echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6'
    ./sshall.sh 'echo 1 > /proc/sys/net/ipv6/conf/default/disable_ipv6'
    ./sshall.sh "echo 'net.ipv6.conf.all.disable_ipv6 = 1' >> /etc/sysctl.conf"
    ./sshall.sh "echo 'net.ipv6.conf.default.disable_ipv6 = 1' >> /etc/sysctl.conf"

"Confirm"

    ./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable_ipv6; cat /proc/sys/net/ipv6/conf/default/disable_ipv6; cat /etc/sysctl.conf | grep disable_ipv6'

Check if there is any`/etc/netconfig`

    ls /etc/netconfig

If something goes on

    vi /etc/netconfig

*   Ouyang Xiu changed the content: annotated two lines
        #udp6 tpi_clts v inet6 udp - -
        #tcp6 tpi_cots_ord v inet6 tcp - -

<!---->

    ./scpall.sh /etc/netconfig

### Modified

According to the official proposal, there will be a section intertwined`vm.swappiness`He should be reduced to phase`1`

    ./sshall.sh 'sysctl -w vm.swappiness=1 > /dev/null; echo 1 > /proc/sys/vm/swappiness; echo vm.swappiness=1 >> /etc/sysctl.conf'

"Confirm"

    ./sshall.sh 'sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness'

### Stop using the transparent_hugepage

Discontinue use

    ./sshall.sh 'echo never > /sys/kernel/mm/transparent_hugepage/defrag; echo never > /sys/kernel/mm/transparent_hugepage/enabled'

"Confirm"

    ./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'

Set "Disable after restart"

    ./sshall.sh "echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.d/rc.local; echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local; chmod +x /etc/rc.d/rc.local; tail -n 2 /etc/rc.d/rc.local"

### Deactivate the paddle

    vi /etc/default/grub

*   Add to it`numa=off`
        GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet numa=off"

<!---->

    ./scpall.sh /etc/default/grub

"Confirm"

    ./sshall.sh 'grep -i numa /etc/default/grub'

Drink and win

    [ -d /sys/firmware/efi ] && echo UEFI || echo BIOS

Full of flies to give birth to new ones

    ./sshall.sh 'grub2-mkconfig -o /etc/grub2.cfg'

Many difficulties and obstacles can be overlooked

    ./sshall.sh 'grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg'

### Start confirm settings again

    ./sshall.sh 'sh allhost_ping.sh'
    ./sshall.sh 'sh allhost_ssh.sh'
    ./sshall.sh 'cat success_ssh'
    ./sshall.sh locale
    ./sshall.sh sestatus
    ./sshall.sh 'systemctl is-active firewalld.service; systemctl is-enabled firewalld.service'
    ./sshall.sh 'cat /proc/sys/net/ipv6/conf/all/disable_ipv6; cat /proc/sys/net/ipv6/conf/default/disable_ipv6; cat /etc/sysctl.conf | grep disable_ipv6'
    ./sshall.sh 'sysctl -a 2>/dev/null | grep swappiness ; cat /proc/sys/vm/swappiness; cat /etc/sysctl.conf | grep swappiness'
    ./sshall.sh 'cat /sys/kernel/mm/transparent_hugepage/defrag; cat /sys/kernel/mm/transparent_hugepage/enabled'
    ./sshall.sh 'numactl --hardware'

> `./sshall.sh 'dmesg|grep -i numa'`At this meeting, what was the wrong judgment

### To load the car to roll and rock

Install furniture fully enshrined

    ./scpall.sh {path_to_jdk_rpm}

<!---->

    ./sshall.sh 'yum install -y {path_to_jdk_rpm}'

As usual:

    cp /var/www/html/cloudera-repos/cm7/RPMS/x86_64/openjdk8-8.0+232_9-cloudera.x86_64.rpm .
    ./scpall.sh openjdk8-8.0+232_9-cloudera.x86_64.rpm
    ./sshall.sh 'yum install -y openjdk8-8.0+232_9-cloudera.x86_64.rpm'

Wang did indeed identify the name of Wang XVIII's directory

    ls /usr/lib/jvm/
    ls /usr/java/

Set fixed`$JAVA_HOME`

    # 手動
    vi /etc/profile

*   Ouyang Xiu changed the content: in the future, the directory will be set as`$JAVA_HOME`
        export JAVA_HOME={path_to_jdk}
        PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin
        export PATH
*   As usual:
        export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_64
        PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin
        export PATH

    <!---->

        export JAVA_HOME=/usr/java/jdk1.8.0_232-cloudera
        PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin
        export PATH

<!---->

    ./scpall.sh '/etc/profile'

<!---->

    # 自動
    ./sshall.sh 'echo -e "export JAVA_HOME={path_to_jdk}\nPATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin\nexport PATH"'

<!---->

    ./sshall.sh 'source /etc/profile'

Climb the mountain and climb again to identify the back`$JAVA_HOME`

    ./sshall.sh "echo $JAVA_HOME" # 必須要是雙引號

"Confirm"`java`Participate`javac`version

> It should be made public, then it should be courtesy, and it should be adapted, so it can be regarded as equivalent to a number

    ./sshall.sh "export PATH=$PATH/bin;java -version;javac -version" # 必須是雙引號

## A set of official names for wooden rice was set up on the river

> If you don't change your clothes, you will see the preconceptions and see the shunshui behind you

> （8
> I didn't see the surname of the person with the surname Ofe`chronyd`Take it instead of it

Modify the time zone

    ./sshall.sh 'timedatectl set-timezone Asia/Taipei'

"Confirm"`chronyd`

    ./sshall.sh 'systemctl is-active chronyd; systemctl is-enabled chronyd'

### Set up and varied

revise`/etc/chrony.conf`

    vi /etc/chrony.conf

*   Modifications: Modifications: Everywhere **\[Has a very deep surname.]** This is also a big battle with me **\[It's Rotten Water Ridge.]** It's all wrapped up in the center
*   As usual:
        #pool 2.rhel.pool.ntp.org iburst # 註解預設的NTP Server pool
        server malvin-cdp-m1-2111.example.com
        local stratum 10 # 內網的NTP Server依賴自己主機的時間
        allow 172.16.1.0/24

### Set to fixed

*   Modification: Annotation preset time, \[add \[surname Ma surname,]
        server {NTP_SERVER_FQDN}
*   As usual:
        #pool 2.rhel.pool.ntp.org iburst
        server malvin-cdp-m1-2111.example.com

When enabled

    ./sshall.sh 'systemctl restart chronyd'

"Confirm"

    ./sshall.sh 'systemctl is-active chronyd; systemctl is-enabled chronyd'

Cai did admit the situation at that time

    ./sshall.sh 'chronyc sources'
    ./sshall.sh 'chronyc clients'

# Build a water war

> Faith: Now it is different from the public, with the courtesy, with the courtesy, with the courtesy

Spare equipment \[is the name of a wooden trough.]

    ./sshall.sh 'mkdir /etc/yum.repos.d/original_repos'
    ./sshall.sh 'mv /etc/yum.repos.d/* /etc/yum.repos.d/original_repos'

Create a directory for mounting

    mkdir /cdrom

Hang on the car

    mount CentOS-7-x86_64-Minimal-2009.iso /cdrom 

It is indeed a mounting feat

    ll /cdrom

revise`/etc/yum.repos.d/oslocal.repo`

    vi /etc/yum.repos.d/oslocal.repo

*   Ouyang Xiu changed the content
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

Installations are falling apart

    yum install -y httpd

When enabled

    systemctl start httpd
    systemctl enable httpd

Install wooden paddles to shake

    yum install -y yum-utils createrepo

It will be hit with fire by water with a meow \[water and fire].

> （8
> It was where the two of us came and went
> There is a big difference!
> `-r`->`--repo`

    # 先確認已經掛載ISO，並做好/etc/yum.repos.d/oslocal.repo
    cd /var/www/html
    reposync --repo rhel8-base-local
    reposync --repo rhel8-appstream-local

The directory will be renamed

    mv /var/www/html/rhel8-base-local/Packages /var/www/html/rhel8-base-local/RPMS
    mv /var/www/html/rhel8-appstream-local/Packages /var/www/html/rhel8-appstream-local/RPMS

The work of the surname 癸 is

    createrepo /var/www/html/rhel8-base-local/
    createrepo /var/www/html/rhel8-appstream-local/

revise`/etc/yum.repos.d/oslocal.repo`

    vi /etc/yum.repos.d/oslocal.repo

*   Ouyang Xiu changed the content
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

The water-filled basket is cooked with coarse wood

    mkdir -p /var/www/html/cloudera-repos/cm7
    tar xvfz cm7.4.4-redhat8.tar.gz -C /var/www/html/cloudera-repos/cm7 --strip-components=1
    chmod -R ugo+rX /var/www/html/cloudera-repos/cm7
    chown -R root:root /var/www/html/cloudera-repos/cm7

revise`/etc/yum.repos.d/cloudera-manager.repo`

    vi /etc/yum.repos.d/cloudera-manager.repo

*   Ouyang Xiu changed the content
        [cloudera-manager-7.4.4]
        name=Cloudera Manager 7.4.4
        baseurl=http://malvin-cdp-m1-2111/cm7.4.4
        enabled=1
        gpgcheck=1
        gpgkey=http://malvin-cdp-m1-2111/cm7.4.4/RPM-GPG-KEY-cloudera

Prepare to modify the style of the post-segment

    ./scpall.sh /etc/yum.repos.d/oslocal.repo
    ./scpall.sh /etc/yum.repos.d/cloudera-manager.repo

"Confirm"

    ./sshall.sh 'yum clean all'
    ./sshall.sh 'yum repolist'
    ./sshall.sh 'yum module list'
    ./sshall.sh 'yum install bash' # BaseOS
    ./sshall.sh 'yum install gcc' # AppStream
    ./sshall.sh 'yum install cloudera-manager-server' # Cloudera Manager

# Installations are made from external repositories

> Note: The views of different versions and the name of the suite, the name of the service and the way to set the file are different, if you need to confirm clearly before using the fixed

Install furniture \[surnamed 癸) is very smooth.)

> Note: Sequential!

    yum install -y postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
    yum install -y postgresql12-12.5-1PGDG.rhel8.x86_64.rpm
    yum install -y postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm

When I was first reformed

    /usr/pgsql-12/bin/postgresql-12-setup initdb

When enabled

    systemctl enable postgresql-12
    systemctl start postgresql-12

revise`/var/lib/pgsql/12/data/pg_hba.conf`

    vi /var/lib/pgsql/12/data/pg_hba.conf

*   Modifications: Shackles are connected to the line, and other aspects of the connection are used instead
        # TYPE  DATABASE        USER            ADDRESS                 METHOD
        # "local" is for Unix domain socket connections only
        local   all             all                                     trust
        # IPv4 connections:
        host    all             all             0.0.0.0/0               md5

revise`/var/lib/pgsql/12/data/postgresql.conf`

    vi /var/lib/pgsql/12/data/postgresql.conf

*   Ouyang Xiu changed the content
        listen_addresses = '*'
        shared_buffers = 256MB
        wal_buffers = 8MB
        checkpoint_completion_target = 0.9

\[Restart the water center.]

    systemctl restart postgresql-12

Use rice, koji, millet, kon, cantonese, and kon

    su - postgres
    psql

*   \[Those with the surname 癸 again go to suffocation.]
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

Back to the utensils of the people!

    exit

# Start confirm settings again

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

<!---->

    ./sshall.sh 'yum clean all'
    ./sshall.sh 'yum repolist'
    ./sshall.sh 'yum module list'
    ./sshall.sh 'yum install bash' # BaseOS
    ./sshall.sh 'yum install gcc' # AppStream
    ./sshall.sh 'yum install cloudera-manager-server' # Cloudera Manager

<!---->

    ./sshall.sh "echo $JAVA_HOME" # 必須要是雙引號
    ./sshall.sh "export PATH=$JAVA_HOME/bin; java -version; javac -version" # 必須是雙引號

<!---->

    ./sshall.sh 'chronyc sources'
    ./sshall.sh 'chronyc clients'

# Install the equipment

The installation is only intact

> They are entangled together, so you have to tidy up the pack by hand, so you have to tidy up the pack by yourself

    yum install -y python27

You install

    yum install -y cloudera-manager-server

If you don't catch the elderly

    rpm --import /var/www/html/cm7.4.4/RPM-GPG-KEY-cloudera

Collusion with external repositories

    /opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm

Launched a bit

    systemctl start cloudera-scm-server

> Because his service is fat, it takes two minutes to start to complete, and you can confirm whether it has been opened during the process
> `lsof -nPi|grep 718`

Use the Viewer to view`http://{MASTER_01_FQDN}:7180`
(MASTER\_01\_FQDN, otherwise you will have to use Hecha instead.)
![](https://i.imgur.com/NEnqpE6.png)
Using a preset tent, are you sure it's coming from the front?

# Sanssouci County

Use official grain in the water

*   Thick entanglement
*   \[Surname 癸.]
*   In smooth water
*   A rope that rolls with water

| 〔| There are | 〔| There are | | There are | of names
|-|-|-|-|-|-|
| Heart | Last name ||| || ||
| Heart | 〔| | | |||||||||||||
| Heart | Water |||||
| That's || | Last name | |||||
| Plume, wild food, wild fish | | ||||
| Plume, wild food, wild fish | It's all | |||||||||||||
| Plume, wild food, wild fish | |||| ||
| Plume, wild food, wild fish | || | | ||
| Wooden sticks, wooden | and wooden | | ||||||
| Wooden sticks, wooden | and wooden | |||||||
| There are | \[Go to || ||||||.]
| But | Yes ||||||
| Watts || Last name |||||
| Muddy water | Last name | ||| ||
| There are | \[Surname|||||||
| | Last name | ||| ||

# Suite

> If the three armies are not fought, the soldiers will not win

> Labeled `rhel-8.2-x86_64-dvd.iso`Use wooden fans, wooden fans, and wooden blocks

## A kit is required

*   \[It is of the upper grade.]
        yum install -y ntp

*   Yes originated from
        yum install -y yum-utils

*   Great for tangling together
        yum install -y createrepo

*   But I'm very surnamed
        yum install -y httpd

*   It was directly used as a backer, and directly used to devote himself to Shunshui
        yum install -y openjdk8-8.0+232_9-cloudera.x86_64.rpm

*   \[It is a precedent for 刔.]

    > in order to install, in order

        yum install postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
        yum install postgresql12-12.5-1PGDG.rhel8.x86_64.rpm
        yum install postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm

## Manufacturing techniques

*   Great place for both of us
        yum install -y ansible
*   There are two headscarves
        yum install -y sshpass
*   \[Originated from Tangshan.]
        yum install -y wget
*   Waited for a day
        yum install -y lsof
*   Waited for over a mile
        yum install -y telnet
*   It's all a mess
        yum install -y psmisc
*   The surname of the surname 癸
        yum install -y tree

# archive

> You can download the required files in advance to speed up the installation

### Cases must be handled well

*   Use officers and soldiers to open the set

    > You need to hire a boatman to carry it

    > This article is already contained in the heart`allkeys.asc`Participate`repod.xml`It is also possible to carve a water stick from the top

    ```
    wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz
    wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz.md5
    wget https://[username]:[password]@archive.cloudera.com/p/cm7/7.4.4/repo-as-tarball/cm7.4.4-redhat8.tar.gz.sha256

    ```
*   \[I came from a humble background.]

    > You need to hire a boatman to carry it

        wget --user {cloudera_user} --password {cloudera_password} --recursive --no-parent --no-host-directories https://archive.cloudera.com/p/cdh7/7.1.7.0/parcels/

    > Note: Don't miss it

        # 一整包幾十GB，可以只挑需要的版本下載，以RedHat8為例
        wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel
        wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel.sha1
        wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el8.parcel.sha256
        wget https://[username]:[password]@archive.cloudera.com/p/cdh7/7.1.7.0/parcels/manifest.json
*   \[The surname Is very well named.]
        wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-server-12.5-1PGDG.rhel8.x86_64.rpm
        wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-libs-12.5-1PGDG.rhel8.x86_64.rpm
        wget https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-8.2-x86_64/postgresql12-devel-12.5-1PGDG.rhel8.x86_64.rpm

### Select the case file

*   Use water hammers to form military trumpets to transport military use
    https://developers.redhat.com/content-gateway/file/rhel-8.2-x86\_64-dvd.iso
