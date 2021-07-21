# CDP Private Cloud Base
CDP Private Cloud Base是Cloudera Data Platform的本地版本。

這款新產品結合了Cloudera Enterprise Data Hub和Hortonworks Data Platform Enterprise，並增強既有功能與添加新功能。 此軟體套件是一個可擴展且可自定義的平台，您可以安全地運行多種類型的工作。

圖1. CDP Private Cloud Base架構
![](https://docs.cloudera.com/cdp-private-cloud/latest/overview/images/cdppvc-architecture.png)

CDP Private Cloud Base支持多種混合解決方案，其中計算任務與資料存儲分離，並且可以從遠端叢集存取資料，包括使用CDP Private Cloud創建工作。 這種混合方法通過管理存儲、資料表設計、身份驗證、授權和治理為容器化應用程式提供了基礎。

CDP Private Cloud Base由各種組件組成，如Apache HDFS、Apache Hive3、Apache HBase和Apache Impala，以及用於專門工作的許多其他組件。 您可以選擇這些服務的任意組合來創建滿足業務需求和工作的叢集。 一些預配置的服務包也可用於常見工作。 這些包括：

## 基本叢集
### 資料工程
程序開發，並提供預測模型。
服務包括：HDFS、YARN、YARN Queue Manager、Ranger、Atlas、Hive、Hive on Tez、Spark、Oozie、Hue和Data Analytics Studio
### 資料市集
以交互方式瀏覽、查詢和探索資料。
服務包括：HDFS、Ranger、Atlas、Hive和Hue
### 作業資料庫
為現代資料驅動型業務提供實時見解。
服務包括：HDFS，Ranger，Atlas和HBase
### 客製化服務
選擇您自己的服務，所選服務所依賴的服務將自動包含在內。

## 計算叢集
### 資料工程
程序開發，並提供預測模型。
服務包括：Spark、Oozie、Hive on Tez、Data Analytics Studio、HDFS、YARN和YARN Queue Manager
### Apache Spark
Spark運算
包括的服務：Core Configuration、Spark、Oozie、YARN和YARN Queue Manager
### 資料市集
用於計算的Impala
服務包括：Core Configuration、Impala和Hue
### 流式消息傳遞（簡單）
用於流式消息傳遞的簡單Kafka叢集
服務包括：Kafka、Schema Registry和Zookeeper
### 流式消息傳遞（完整）
進階Kafka叢集，具有用於流式消息傳遞的監控和複製服務
包括的服務：Kafka、Schema Registry、Streams Messaging Manager、Streams Replication Manager、Cruise Control和Zookeeper
### 客製化服務
選擇您自己的服務，所選服務所依賴的服務將自動包含在內。

安裝CDP Private Cloud Base叢集時，您需要安裝一個名為Cloudera Runtime的單個Parcel，其中包含所有組件。 有關包含組件的完整列表，請參閱[Cloudera Runtime組件版本](https://docs.cloudera.com/cdp-private-cloud-base/7.1.6/runtime-release-notes/topics/rt-pvc-runtime-component-versions.html)。

除了Cloudera Runtime組件外，CDP Private Cloud Base還包含強大的工具，可幫助管理、治理和保護您的叢集。

## CDP Private Cloud Base工具
### Cloudera Manager
CDP Private Cloud Base使用Cloudera Manager管理一個或多個叢集及其配置，並監控叢集性能。 您還可以使用Cloudera Manager管理安裝、升級、維護工作流、加密、存取控制和資料複製。 您還可以使用Cloudera Manager來管理Cloudera Enterprise CDH叢集。 您可以使用Cloudera Manager創建一個虛擬專用叢集，使您能夠將計算資源與資料存儲分開，並在計算資源之間共享資料存儲。

### Apache Atlas
CDP Private Cloud Base中還包含Apache Atlas，用於為您的資料提供治理。 Apache Atlas作為通用元資料存儲，旨在交換Hadoop stack內外的元資料。 Atlas與Apache Ranger的緊密整合使您能夠在Hadoop stack的所有組件中一致地定義、管理和管理安全和合規性策略。 對於熟悉Cloudera Enterprise的客戶，Apache Atlas取代了Cloudera Navigator Metadata Server。它提供以下功能：
- 靈活的元資料模型
- 使用模型屬性、分類（標籤）和自由文本進行實體搜索
- 基於應用於實體的流程跨實體沿襲

### Apache Ranger
Apache Ranger為您的CDP Private Cloud Base叢集提供稽核、身份驗證和授權功能。

Apache Ranger提供了一個集中式框架，用於收集存取稽核歷史記錄和報告資料，包括對各種參數進行過濾。 Ranger增強了從Hadoop組件獲得的稽核資訊，並通過這種集中式報告功能提供見解。

Apache Ranger還通過使用者界面管理存取控制，確保跨CDP Private Cloud Base組件進行一致的策略管理。 安全管理員可以在資料庫、表、欄和文件級別定義安全策略，並且可以管理特定基於LDAP的組或單個使用者的權限。 基於動態條件（如時間或地理位置）的規則也可以添加到現有策略規則中。 Ranger授權模型是可插拔的，可以使用基於服務的定義輕鬆擴展到任何資料源。

對於熟悉Cloudera Enterprise的客戶，Apache Ranger取代了Sentry和Navigator Audit Server，還提供了以下功能：

- 更好的細粒度存取控制：
	- 動態列篩選
	- 動態欄遮罩
	- 基於屬性的存取控制
- SparkSQL細粒度存取控制
- 豐富的策略功能
	- 允許/拒絕構造，自定義策略條件/上下文增強器，有時限策略，Atlas整合（基於標記的策略）
- 使用豐富的事件元資料進行廣泛的存取稽核