## OneFS基本設定
確認版本
> OneFS須為8.1.2以後
```
isi version
```
添加License
```
isi license add --evaluation=HDFS
```
列出LIcense
```
isi license licenses list
isi license licenses view HDFS
```
8.2.1語法
```
isi license list
isi license view HDFS
```
啟用HDFS
```
isi service hdfs enable
```
更新補丁清單
```
isi upgrade patches list
```
安裝補丁
```
isi upgrade patches install patch-<patch-ID>.pkg --rolling=false
```

## OneFS目錄與網路
```
isi zone zones create --name=zone1-cdp --path=/ifs/data/zone1/cdp/hadoop-root --create-path
```
8.2.1語法
```
isi zone zones create zone1-cdp /ifs/data/zone1/cdp/hadoop-root --create-path
```
製作hadoop根目錄(上一個指令有帶--create-path就不用了)
```
mkdir -p /ifs/data/zone1/cdp/hadoop-root
```
限定HDFS的區域與根目錄
```
isi hdfs settings modify --zone=zone1-cdp --root-directory=/ifs/data/zone1/cdp/hadoop-root
```
創建pool
> pool之間的IP不能重複！
```
isi network pools create --id=groupnet0.subnet0.hadoop-pool-cdp --ranges=172.16.1.43-172.16.1.56 --access-zone=zone1-cdp --alloc-method=static --ifaces=1-2:ext-1 --sc-subnet=subnet0 --sc-dns-zone=malvin-isi.example.com --description=hadoop
```
如果還沒設置SmartConnect
> SmartConnect就是一台DNS Server，因此必須是現有pool之外的IP，若設定成多個IP是供故障轉移使用，沒有負載平衡功能
```
isi network subnets modify subnet0 --sc-service-addrs=172.16.1.41-172.16.1.42 --sc-service-name=isicdp.example.com
```
> CDP Cluster必須要加上SmartConnect這個DNS Server，以RHEL7為例，需要修改`/etc/resolve.conf`，示範如下：
>  ```
>  domain isicdp.example.com # Isilon Cluster FQDN
>  nameserver 172.16.1.41 # SmartConnet的IP
>  search isicdp.example.com # 指定要透過172.16.1.41解析FQDN
>  ```

誤刪subnet重建
```
isi network subnets create groupnet0.subnet0 ipv4 24 --description=System --gateway=172.16.1.254 --gateway-priority=20 --mtu=1500 --sc-service-addrs=172.16.1.41
```
## OpenFS使用者
創建相應的group（如果使用小工具就不用手動下指令創建了！）
```
isi auth groups create hdfs --zone zone1-cdp --provider local --gid 986
```
創建相應的user
```
isi auth users create hdfs --primary-group hdfs --zone zone1-cdp --provider local --home-directory /ifs/data/zone1/hadoop/user/hdfs --uid 989
```

```
isi hdfs settings view --zone=zone1-cdp
```
建立角色
```
isi auth roles create --name=HdfsAccess --description="Bypass FS permissions" --zone=zone1-cdp
```
授權
```
isi auth roles modify HdfsAccess --add-priv=ISI\_PRIV\_IFS\_RESTORE --zone=zone1-cdp
```
授權
```
isi auth roles modify HdfsAccess --add-priv=ISI\_PRIV\_IFS\_BACKUP --zone=zone1-cdp
```

```
isi auth roles modify HdfsAccess --add-user=hdfs --zone=zone1-cdp
```
> **不要**使用FreeBSD指令建立group & user！
> https://docs.freebsd.org/zh_TW/books/handbook/users-synopsis.html
> ```
> adduser
> pw groupadd hdfs
> ```

製作user與目錄自己手刻可能有漏，造成安裝Component不順，建議使用小工具：
- https://github.com/claudiofahey/isilon-hadoop-tools
工具比較舊，需要手動調整CDP hosts上uid
```
./isilon_create_users.sh --dist cdh --zone zone1-cdp
./isilon_create_users.sh --dist cdh --zone zone1-cdp --fixperm
```
查看建立好的目錄，owner是UID或名字都沒關係，最後有mapping到CDP上的user就好了
```
ls -l /ifs/data/zone1/cdp/hadoop-root/user
```
![](https://i.imgur.com/Q89UzZS.png)
- https://github.com/Isilon/isilon_hadoop_tools
最新的工具，可是不知道Isilon怎麼裝python...

> **做好user之後一定要到Isilon OneFS Web UI確認由沒有"Enable"！**