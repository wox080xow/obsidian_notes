# 事前準備：安裝Apache Maven
尋找並安裝Apache Maven
```
yum whatprovides 'mvn'
yum install -y maven
```
![](https://i.imgur.com/XONRf1n.png)

> hbck2需要maven-3.3.3以上
![](https://i.imgur.com/30uZbMm.png)
![](https://i.imgur.com/cn5Dw37.png)

尋找其他版本
```
yum --showduplicates list maven
```
可惜只有3.0.5

![](https://i.imgur.com/ZNzaLMK.png)

移除舊版
```
yum remove -y maven
```
使用TAR檔安裝指定版本
```
cd ~
wget https://downloads.apache.org/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
mkdir /usr/local/maven
tar xvf apache-maven-3.3.9-bin.tar.gz -C /usr/local/maven/
alternatives --install /usr/bin/mvn mvn /usr/local/maven/apache-maven-3.3.9/bin/mvn 1
alternatives --config mvn
```
![](https://i.imgur.com/l8IC1JU.png)

# 安裝HBase hbck2
下載並安裝hbck2
```
git clone https://github.com/apache/hbase-operator-tools.git
cd hbase-operator-tools/
mvn clean install -DskipTests
```
應該有跑了十分鐘

![](https://i.imgur.com/gaJoTpV.png)

安裝好hbck2後，使用hbase指令執行
```
hbase hbck -j /root/hbase-operator-tools/hbase-hbck2/target/hbase-hbck2-1.2.0-SNAPSHOT.jar
```
長長一串使用說明

![](https://i.imgur.com/yUBO8Yu.png)