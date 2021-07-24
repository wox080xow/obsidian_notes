# 是否有免費版？
有單機試用版，可以試用60天

參考連結：[https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-trial-download-information.html](https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-trial-download-information.html)
# CDP Private Cloud Base安裝最低規格？
## 作業系統
CDP Private Cloud Base支援的作業系統

|Operating System|Version|
|--|--|
|RHEL/CentOS/Oracle|7.6, 7.7, 7.8, 7.9|
|SLES|12 SP5|
|Ubuntu|18.04|

## 硬體
### CDP Private Cloud Base基本叢集
-   叢集包含儲存與計算
-   節點數量建議至少五個，兩台主機作為主節點，負責協調工作節點，並設置高可用性，三台主機作為工作節點，負責儲存與計算
    
主節點硬體規格：

|**項目**|**說明**|
|--|--|
|CPU|24 cores+|
|Memory|32 GB+|
|Storage|200 GB+|

工作節點硬體規格：

|**項目**|**說明**|
|--|--|
|CPU|24 cores+|
|Memory|64 GB+|
|Storage|建議區別系統碟與資料碟<br><ol><li>系統碟 200 GB+</li><li>資料碟</li><ul><li>不必設置RAID</li><li>視預期資料量決定容量大小，保留兩成緩衝容量</li><li>每顆硬碟不要大於 8 TB</li><li>總容量不要大於 100 TB</li></ul></ol>|
	
	
### CDP Private Cloud Base計算叢集

-   叢集僅計算功能，儲存使用其他解決方案，例如DellEMC Isilon等
    
-   節點數量建議至少五個，兩台主機作為主節點，負責協調工作節點，並設置高可用性，三台主機作為工作節點，負責計算工作
    

主節點硬體規格：

|**項目**|**說明**|
|--|--|
|CPU|16 cores+|
|Memory|32 GB+|
|Storage|200 GB+|

工作節點硬體規格：

|**項目**|**說明**|
|--|--|
|CPU|16 cores+|
|Memory|64 GB+|
|Storage|200 GB+|

參考連結：

[https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-os-requirements.html](https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-os-requirements.html)

[https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-hardware-requirements.html](https://docs.cloudera.com/cdp-private-cloud/latest/release-guide/topics/cdpdc-hardware-requirements.html)

# 為什麼需要訂閱Cloudera企業版？

Cloudera企業版，包括對不同系統和資料管理軟體的訪問，8x5或24x7支援，以及賠償，與企業Data Hub的永續發展至關重要。 此外，所有Cloudera License需要每年續訂，因此Cloudera必須不斷證明其價值。

# 是什麼讓Cloudera的產品與眾不同？

Cloudera的平台具有使其獨特的原因，包括:

-   與其他商業替代方案的差異：
	Cloudera提供不同的功能
	1.  Production等級的互動式SQL與Hadoop搜索
	1.  全面的系統管理，包括滾動升級、自動災難恢復、中心化的安全策略、主動健康檢查和多集群管理
	1.  通過細粒度稽核與訪問控制來簡化資料管理的過程
-   與Apache Hadoop社群版的差異
	雖然Cloudera的平台可以在開源的Hadoop生態系中找得到相同程式碼，但是Cloudera每季會為其平台的用戶提供錯誤修復和穩定措施（也會貢獻給上游的開源程式碼）。因此，Cloudera客戶可以定期改進平台，這些修正經由嚴格測試保證與上游兼容。