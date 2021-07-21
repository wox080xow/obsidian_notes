# Phoenix Client
1. 確認JDK版本與路徑
2. 確認JDBC之JAR檔路徑
3. 確認Phoenix的FQDN與Port
4. 串接
5. 測試增刪改查
	```
	create table if not exists tmp1(id varchar primary key, cf1.database varchar);
	upsert into tmp1 values ('1','MySQL');
	upsert into tmp1 values ('2','ORACLE');
	upsert into tmp1 values ('3','PostgreSQL');
	!tables
	select * from tmp1;
	drop table tmp1;
	```

# HBase Backup
## 功能測試
1. 調整YARN Queue
	- 已知default queue佔20% 
2. 測試YARN Queue是否生效
	- 跑Pi
	```
	find /usr/hdp/ -name "*examples.jar"
	# hadoop jar /usr/hdp/2.6.5.0-292/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi -Dmapred.job.queue.name=queue 10 10
	```
	- 查看RM UI
3. 選定table
	- PDATA_EDA.MEA_EDC_T3 
	- 210604大小紀錄
	  
	  |Isilon|HDFS|
	  |--|--|
	  |486GB|323.9GB|
4. 分批匯出
	- 時間格式轉換
     ```
     date -d 150101 +%s
	 date -d @14200884000
     ```
   
      |date|timestamp (ms)|
      |--|--|
      |150101|1420088400000|
      |160101|1451624400000|
      |170101|1483246800000|
      |180101|1514782800000|
      |190101|1546318800000|
      |200101|1577854800000|
      |210101|1609477200000|
      |210701|1625068800000|
	  
   尋找最早資料
   ```
   # hbase shell 
   import org.apache.hadoop.hbase.filter.FirstKeyOnlyFilter;
   #count 'PDATA_EDA.MEA_EDC_T3',{FILTER=>FirstKeyOnlyFilter.new(),TIMERANGE => [1, 1622093995391], LIMIT => 1}
   ```
   匯出desc
   ```
   echo "desc 'PDATA_EDA.MEA_EDC_T3'"|hbase shell -n >desc.list.tmp
   hdfs dfs -put desc.list.tmp /tmp
   distcp hdfs://pdhdpfs:8020/tmp/desc.list.tmp hdfs://tnisilonh500.cminl.oa/tmp/
   ```
   分批匯出
   ```  
   # part1
   #hbase org.apache.hadoop.hbase.mapreduce.Export -Dmapred.job.queue.name=queue PDATA_EDA.MEA_EDC_T3 hdfs://tnisilonh500.cminl.oa:8020/tmp/exprot-PDATA_EDA.MEA_EDC_T3_TMP-part1 1 1451624400000
   
   # part2
   #hbase org.apache.hadoop.hbase.mapreduce.Export -Dmapred.job.queue.name=queue PDATA_EDA.MEA_EDC_T3 hdfs://tnisilonh500.cminl.oa:8020/tmp/exprot-PDATA_EDA.MEA_EDC_T3_TMP-part2 1451624400000 1625068800000
   ```
6. 分批匯入與Merge
   ```
   hdfs dfs -cat /tmp/desc.list.tmp
   ```
   ```
   # hbase shell
   #create 'PDATA_EDA.MEA_EDC_T3_TMP', ...
   #create 'PDATA_EDA.MEA_EDC_T3', ...
   ```
   ```
   # part1
   hbase org.apache.hadoop.hbase.mapreduce.Import PDATA_EDA.MEA_EDC_T3_TMP hdfs://tnisilonh500.cminl.oa:8020/tmp/exprot-PDATA_EDA.MEA_EDC_T3_TMP-part1
   hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=PDATA_EDA.MEA_EDC_T3 PDATA_EDA.MEA_EDC_T3_TMP
   
   # 清除tmp table資料
   echo "truncate 'PDATA_EDA.MEA_EDC_T3_TMP'"|hbase shell -n
   
   # part2
   hbase org.apache.hadoop.hbase.mapreduce.Import PDATA_EDA.MEA_EDC_T3_TMP hdfs://tnisilonh500.cminl.oa:8020/tmp/exprot-PDATA_EDA.MEA_EDC_T3_TMP-part2
   hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=PDATA_EDA.MEA_EDC_T3 PDATA_EDA.MEA_EDC_T3_TMP
   ```
10. 確認備份狀態
	```
	# HDP
	hbase org.apache.hadoop.hbase.mapreduce.RowCounter -Dmapred.job.queue.name=queue PDATA_EDA.MEA_EDC_T3_TMP
	```
	```
	# CDP
	hbase org.apache.hadoop.hbase.mapreduce.RowCounter PDATA_EDA.MEA_EDC_T3_TMP
	```
11. 紀錄花費時間

## 腳本測試
腳本需調整部分：
- 匯出腳本`export.sh`
	- `srchdfs`改為`hdfs://pdhdpfs:8020/tmp`
	- `desthdfs`改為`hdfs://tnisilonh500:8020/tmp`
	- `hbase org.apache.hadoop.hbase.mapreduce.RowCounter`加上option`-Dmapred.job.queue.name=queue`
	- `hbase org.apache.hadoop.hbase.mapreduce.Export`加上option`-Dmapred.job.queue.name=queue`

1. 跑匯出
	```
	time sh export.sh 210701 210702
	time sh export.sh 210702 210703
	```
1. 跑匯入
	```
	time sh export.sh 210701 210702
	time sh export.sh 210702 210703
	```
3. 跑Merge
	```
	time sh export.sh 210701 210702
	time sh export.sh 210702 210703
	```
5. 確認備份狀態
	```
	#vi OMNI_TMP_FILES/rc.table.list-...
	```
1. 紀錄花費時間
	使用time語法輸出的`real`
1. 推估總花費時間
	一次跑一天的資料，預計跑兩年半的資料，總花費時間約為`real*900`