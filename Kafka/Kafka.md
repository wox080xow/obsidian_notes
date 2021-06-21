[Apache Kafka Series - Learn Apache Kafka for Beginners v2](https://www.udemy.com/course/apache-kafka/)
# 210619
decouple data streams

broker -> topic -> prartition -> offset

### offset
- 從0開始的ID
- 只能寫入與讀取，無法更改
- 預設一週後會刪除

### topic and partition
建立topic就要指定partition數量，指定後可以再增加
cf. Elasticsearch index & shard

### Leader of  partitions while there are replications
- Leader
- ISR, in-sync replica
- partition leader由ZooKeeper決定

## Producer
### message keys
- 有key就會固定送到該partition
- 可以用hashing處理
- 沒有key就使用round robin

### data write acknowledgements
1. ask=0
partition leader不用回報給Producer，可能掉資料
1. ask=1
partition leader要回報，較不會掉資料
1. ask=all
partition leader與partition ISR都要回報，不會掉資料

## Consumer
- 照offset ID順序讀取資料，partition間的offset的順序無法保證
- Consumer數量多於Broker會導致Comsumer對不到Broker而閒置
- 可以指定offset ID作為讀取資料的起始點（不必從0開始讀取）

### delivery semantics
1. at most once
當Consumer commit offset時，資料還沒寫進該offset，可能會沒讀到資料
3. at least once
必須確保Consumer不會應為讀取相同資料兩次以上就crash
4. exactly once
只能Kafka到Kafka

### bootstrap server
- 一次只能向外連一台
- 但是每台server都知道其他server的狀況——存了哪些topic的哪些partition——透過這些metadata將他轉介到其他server
- Kafka之外，Elasticsearch也是

## ZooKeeper
- Broker changes
- Topic changes

## Kafka保證
- 新增的資料絕對照順序儲存
- Consumer絕對照順序讀取資料
- replication factor = N，那就可以承受N-1台Brokers故障，所以當N = 3就能在一台Broker維護中的狀態下，再承受一個Broker意外故障
- 只要不新增partition，指定的key就會送到相同的partition

### Quiz
- Very important: you only need to connect to one broker (any broker) and just provide the topic name you want to write to. Kafka Clients will route your data to the appropriate brokers and partitions for you!
- Two consumers that have the same group.id (consumer group id) will read from mutually exclusive partitions

210620
我只是想在我的電腦裝JDK而已...
```
brew update
brew tap caskroom/version # deprecated
brew tap homebrew/cask-versions # Error: Unknown command: cask
xcode-select --install
brew install gcc # 460MB...
brew install brew-cask # not found...
brew install brew-cask-completion # Error: Unknown command: cask
brew install Caskroom/cask/seil # Error: Unknown command: cask
```
放棄`brew cask install java8`，改用`brew install openjdk@8`
```
brew install openjdk@8
sudo ln -sfn /usr/local/opt/openjdk@8/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-8.jdk
echo 'export PATH="/usr/local/opt/openjdk@8/bin:$PATH"' >> ~/.bash_profile
```

210621
用Homebrew安裝kafka
```
brew install kafka
```
![](https://i.imgur.com/JiQmC0i.png)
測試安裝JDK與Kafka後是否能運作 
```
cd kafka_2.13-2.8.0
bin/kafka-topics.sh
```
加到$PATH 
```
echo 'export PATH="$PATH:/Users/wox080xow/kafka_2.13-2.8.0/bin"' >>~/.bash_profile
```
修改ZooKeeper設定
```
cd kafka_2.13-2.8.0
mkdir -p data/zookeeper
vi config/zookeeper.properties
```
- 將`dataDir=/tmp/zookeeper`改為`dataDir=/Users/wox080xow/kafka_2.13-2.8.0/data/zookeeper`

開啟ZooKeeper
```
cd kafka_2.13-2.8.0
bin/zookeeper-server-start.sh config/zookeeper.properties
```
修改Kafaka設定
```
cd kafka_2.13-2.8.0
mkdir data/kafka
vi config/server.properties
```
- 將`log.dirs=/tmp/kafka-logs`改為`log.dirs=/Users/wox080xow/kafka_2.13-2.8.0/data/kafka`

開啟Kafka
```
cd kafka_2.13-2.8.0 
kafka-server-start.sh config/server.properties
```
創建topic
```
# Kafka 2.2.0以後，廢棄--zookeeper，改用--bootstrap-server
# Kafka 2.8.0直接過，不用指定partition與replication factor，預設均為一
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --create
```
![](https://i.imgur.com/fXXhE3O.png)
列出topics
```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --list
```
描述topic，查看partition與replication factor數量等資訊
```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --describe
```
刪除topic
```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --delete
```
改變topic的partition數量
```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --alter --partitions 3
```
## Producer & Consumer
寫入資料
```
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic malvin
```
從進入的時間點讀取
```
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic malvin
```
從頭讀到尾
```
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --from
-beginning
```
![](https://i.imgur.com/b0QcC6p.png)
Consumer Group
```
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --group my-first-application
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --from
-beginning --group my-first-application
```
- 同一個Group的各個Consumer分別讀取不同partition的資料，增刪Consumer會自動重新分配Partition
- 如果一個Group已經有某個Consumer從頭讀到尾，同Group的另一個Consumer就算設置`--from-beginning`也讀不到前面的資料（Kafka不會重複Commit相同的資料到同一個Group）
列出Groups
```
# Kafka 2.1以後不再列出單獨一個Consumer的Group
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list
```
描述Group
```
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --group my-fir
st-application --describe
```
重置offset
```
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --group my-first-application --reset-offsets --to-earliest --execute --topic malvin
```
Partition間offset順序無法保證：事後增加到3個Partition，很顯然Consumer讀取的順序是亂的，我第一個輸入的訊息應該是"This is my first try for using producer!"
![](https://i.imgur.com/gK7hUW3.png)
設定分隔符（好像只能隔一個，而不是CSV那種概念）
```
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic malvin --property parse.key=true --property key.separator=,
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic malvin --group my-first-application --from-beginning --property print.key=true --property key.separator=,
```
- Conduktor，由講師與其團隊開發的Kafka UI
- Kafkacat，另一種Kafka CLI
## Kafka JAVA Programming
Maven`pom.xml`
[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)
以Kafka 2.8.0為例
```
<!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.0</version>
</dependency>
```
[slf4j-simple](https://mvnrepository.com/artifact/org.slf4j/slf4j-simple)
以slf4j 1.7.30為例
```
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-simple -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.30</version>
    <scope>test</scope>
</dependency>
```
[Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)
[Consumer Configs](https://kafka.apache.org/documentation/#consumerconfigs)

如果`Duration.ofMills()`出現底線，要點擊"Set language level to 8"，把程式定調為JAVA8（會長在`pom.xml`），才能正常運作
![](https://i.imgur.com/baJaCcZ.png)

在同Group起兩個Consumer會Rebalancing
`ConsumerDemo`吃到Partition0與1
![](https://i.imgur.com/2E1Dean.png)
![](https://i.imgur.com/Qpgg8Rg.png)
`ConsumerDemoGroups`吃到Partition2
![](https://i.imgur.com/QggsHGQ.png)
![](https://i.imgur.com/Abu4vrG.png)

離開`ConsumerDemo`
![](https://i.imgur.com/sJQMHjY.png)
三個Partition全部Rebalancing到`ConsumerDemoGroup`
![](https://i.imgur.com/OEixIz2.png)

 ### Bi-directional Compability
- 自Kafka 0.10.2起，不管的Client版本新舊可以訪問所有版本的Broker
- 讓Client保持更新可以使用到最新功能

### Quiz
- I should block the `producer.send()` by adding `.get()` at the end. 
No! Never block `.send()`. It will kill your producer's performance. Instead, make sure to `.close()` your producer before shutting down your application.
- To allows consumers in a group to resume at the right offset, I need to set "gruop.id".
- When my consumers have the same group.id, they will read from mutually-exclusive partitions.
# 名詞解釋
## Avro
data format