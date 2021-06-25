# Isilon功能驗證
## 一、確認CDP節點能否ping到Isilon
## 二、確認Hadoop USERS and GROUPS已建立並且Enable
## 三、確認HDFS USERS家目錄已建立
<br></br>
## 確認CDP節點能否ping到Isilon
以我的測試環境為例，Isilon SmartConnect的FQDN為`isicdp.example.com`
```
ping -c4 isicdp.example.com
```
## 確認Hadoop USERS and GROUPS已建立並且Enable
1. 進入Isilon的OneFS UI
	- 網址為https://{ISILON_FQDN}:8080
 	- 預設帳號密碼是root與Isilon的root密碼
2. Access -> Member and roles
![](https://i.imgur.com/c53WN2G.png)
3. 確認Hadoop相關user and group的Account status為"Enabled"
	1. Current access zone選擇分配給HDFS的zone，我的測試環境是"zone1-cdp"
    2. 選擇Users頁籤（若是要查看Groups就改選擇Groups頁籤）
	3. Providers選擇分配給HDFS的zone，也是"zone1-cdp"
	4. 可以看到Users的Account status
![](https://i.imgur.com/JwooUFZ.png)
## 確認HDFS USERS家目錄已建立
以我的測試環境為例：
```
isi zone zones list
```
分配給HDFS使用的zone為`zone1-cdp`

![](https://i.imgur.com/lKlWUbV.png)
```		
ls -l /ifs/data/zone1/cdp/hadoop-root
```
查看`zone1-cdp`路徑`/ifs/data/zone1/cdp/hadoop-root`有無`user`目錄
![](https://i.imgur.com/AM0nL8Q.png)
```
ls -l /ifs/data/zone1/cdp/hadoop-root/user
```
接著查看`/ifs/data/zone1/cdp/hadoop-root/user`是否已經有Hadoop相關user的家目錄，例如`hdfs`, `yarn`, `hbase`等等
![](https://i.imgur.com/urHP63f.png)