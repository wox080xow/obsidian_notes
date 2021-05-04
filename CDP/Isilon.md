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
8.2.1語法更新
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
製作hadoop根目錄
```
mkdir -p /ifs/data/zone1/cdp/hadoop-root
```
串Isilon
```
isi hdfs settings modify --zone=zone1-cdp --root-directory=/ifs/data/zone1/cdp/hadoop-root
```
創建pool
> pool之間的IP不能重複
```
isi network pools create --id=groupnet0.subnet0.hadoop-pool-cdp --ranges=172.16.1.38-172.16.1.43 --access-zone=zone1-cdp --alloc-method=static --ifaces=1-2:ext-1 --sc-subnet=subnet0 --sc-dns-zone=cdp.zone1.malvin.example.com --description=hadoop
```
如果還沒設置SmartConnect
> 必須是現有pool之外的IP，若設定成多個IP是供故障轉移使用，沒有負載平衡功能
```
isi network subnets modify subnet0 --sc-service-addrs=172.16.1.36-172.16.1.37 --sc-service-name=cdp.zone1.malvin.example.com
```