

# 一、[前置作業](#前置作業)
# 二、[CDP啟用Kerberos](#cdp啟用kerberos)
# 三、[事後整合確認](#事後整合確認)
# 前置作業
### 安裝與設定Kerberos Server
1. #### 安裝Kerberos Server
	 - 選定一台主機安裝
	 - Cloudera Manager需要openldap-clients
	```
	yum -y install krb5-server krb5-libs openldap-clients
	```
1. #### 設定`/etc/krb5.conf`
	 - 以下範例假設安裝Kerberos Server的主機FQDN為`cfori-m1.us-central1-a.c.eternal-ruler-310501.internal`
	 - Kerberos的realm一定要是大寫的，如`US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL`
	 - 每行前面有無空白只是排版美觀，不會影響設定
	```
	[libdefaults]
	default_realm = US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	dns_lookup_kdc = false
	dns_lookup_realm = false
	ticket_lifetime = 86400
	renew_lifetime = 604800
	forwardable = true
	default_tgs_enctypes = aes256-cts-hmac-sha1-96
	default_tkt_enctypes = aes256-cts-hmac-sha1-96
	permitted_enctypes = aes256-cts-hmac-sha1-96
	udp_preference_limit = 1
	kdc_timeout = 3000

	[realms]
	US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL = {
	kdc = cfori-m1.us-central1-a.c.eternal-ruler-310501.internal
	admin_server = cfori-m1.us-central1-a.c.eternal-ruler-310501.internal
	}

	[domain_realm]
	.us-central1-a.c.eternal-ruler-310501.internal = US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	us-central1-a.c.eternal-ruler-310501.internal = US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL

	[logging]
	kdc = FILE:/var/log/krb5kdc.log
	admin_server = FILE:/var/log/kadmin.log
	default = FILE:/var/log/krb5lib.log
	```
1. #### 設定`/var/kerberos/krb5kdc/kdc.conf`
	```
	default_realm = US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNA

	[kdcdefaults]
	v4_mode = nopreauth
	kdc_ports = 0

	[realms]
	 US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL = {
	  kdc_ports = 88
	  admin_keytab = /etc/kadm5.keytab
	  master_key_type = aes256-cts
	  database_name = /var/kerberos/krb5kdc/principal
	  acl_file = /var/kerberos/krb5kdc/kadm5.acl
	  max_life = 10h 0m 0s
	  max_renewable_life = 7d 0h 0m 0s
	  supported_enctypes = arcfour-hmac:normal des3-hmac-sha1:normal des-cbc-crc:normal des:normal des:v4 des:norealm des:onlyrealm des:afs3
	  default_principal_flags = +preauth
	 }
	 ```

1. #### 設定`/var/kerberos/krb5kdc/kadm5.acl`
	```
	*/admin@CW.COM	    *
	```

### 安裝JCE POLICY
CDP要求Kerberos要使用aes256加密，這的加密法需要另外下載
1. #### 下載jce_policy-8.zip
	 - 需要登入ORACLE手動下載解壓縮，在傳到安裝Kerberos的主機
	 	https://www.oracle.com/webapps/redirect/signon?nexturl=https://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip
	 - **注意！使用CLI可以下載，但是檔案不完整無法解壓縮...**
1. #### 把`jar`檔放到正確位置
	```
	mkdir $JAVA_HOME/lib/security
	cp local_policy.jar US_export_policy.jar $JAVA_HOME/lib/security
	```

### 建立Kerberos的database
```
kdb5_util create -s
```
### 建立Kerberos的管理員principal
1. #### 進入本地的kadmin shell
	```
	kadmin.local
	```
1. #### 建立Kerberos的管理員principal，要記得自己設定的密碼！
	```
	kadmin.local:  addprinc root/admin
	```
1. #### 離開kadmin shell（進出kadmin shell的步驟下面省略）
	```
	kadmin.local:  exit
	```
 ### 建立Cloudera Manager的principal
- 密碼設定為cloudera-scm
- @後面是Kerberos的realm
```
kadmin.local:  -pw cloudera-scm cloudera-scm/admin@US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
```
### 啟動KDC(Key Distribution Center)
接下來可以啟動KDC了！
```
systemctl start krb5kdc.service
systemctl start kadmin.service
systemctl enable krb5kdc.service
systemctl enable kadmin.service
```

### 安裝Kerberos Client
1. #### 安裝Kerberos Client
	 - 叢集裡所有主機都要安裝
	```
	while read h;do ssh $h "hostname;yum -y install krb5-workstation krb5-libs" </dev/null;done < hostlist
	```
1. #### 上傳設定檔`/etc/krb5.conf`
	 - 叢集裡所有主集都要有Kerberos這台主機的`etc/krb5.conf`
	```
	while read h;do scp /etc/krb5.comf $h:/etc/krb5.conf;done < hostlist
	```



# CDP啟用Kerberos
### 啟用Kerberos
 - #### 若是「在建立CDP前」啟用Kerberos
	新增CDP叢集時可以看到啟用Kerberos的提示
	![Kerberos](https://i.imgur.com/jDRDhQh.png)
	使用Wizard啟用Kerberos
	1. #### Getting Started
		 - 選擇MIT KDC
		 - 勾選已經完成上述步驟
		![Step1](https://i.imgur.com/DokepsQ.png)
	1. #### Enter KDC Information
		 - Kerberos Encryption Type：使用aes256，比較安全
		 - Kerberos Security Realm：安裝Kerberos Server的主機，不用hostname，英文全部大寫
		 - KDC Server Host：安裝Kerberos Server的主機FQDN
		 - KDC Admin Server Host：安裝Kerberos Server的主機FQDN
		 - 其他參數基本上不用填寫或修改
		![Step2](https://i.imgur.com/fWmWNp7.png)
	1. #### Manage krb5.conf
		 - 勾選管控krb5.conf，其他參數基本上不用填寫或修改
		![Step3](https://i.imgur.com/yFOnG8n.png)
	1. #### Enter Account Credentials
		輸入「[[#建立Cloudera Manager的principal]]」的步驟中的
		帳號密碼（markdown錨點有問題）
		![Step4](https://i.imgur.com/Aw6CoJ3.png)
	1. #### Command Details
		等待指令跑完就完成了！
 - #### 若是「建立好CDP後」啟用Kerberos
	1. 進入建立好的叢集（Cluster）頁面，點擊叢集名稱右邊的「三的點點」，下拉式選單中有Enable Kerberos，進入後按照步驟填寫相關參數
		![Kerberos](https://i.imgur.com/b6TtVQb.png)
- #### 其他設置
	 - #### Kerberos參數
		Administration > Settings
		再利用關鍵字搜尋
	 - #### Kerberos Credentials相關
		左欄Adminisration > Security，進到Security頁面點擊"Kerberos Credentials"
		![path](https://i.imgur.com/IIb7goM.png)
		 - #### 輸入Kerberos管理員帳號密碼
			輸入「[[#建立Cloudera Manager的principal]]」的步驟中的
			帳號密碼（markdown錨點有問題）
			![account manager](https://i.imgur.com/5KV3MK4.png)
		 - #### 製作缺漏的Kerberos Principal（Hadoop core system users, such as `hdfs`）
			![principal](https://i.imgur.com/UbF7xlr.png)
### 為HDFS建立Superuser principal
1. #### 修改HDFS的superuser group
   例如設定為`supergroup`
   ![superuser](https://i.imgur.com/Rg5v5jW.png)
3. #### 在Gateway或任一台主機建立新user
	```
	useradd supergroup
	```
1. #### 建立Kerberos的principal
	```
	kadmin:  addprinc supergroup@US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	```
1. #### 初始化principal
	初始化之後代表由這個principal向KDC索取憑證，這樣才能正常使用HDFS功能
	```
	kinit supergroup@US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	```
	
# 事後整合確認
### 確認Kerberos整合成功
1. #### 在Gateway或任一台主機建立新user
	```
	useradd malvin
	```
1.  #### 建立Kerberos的principal
	初始化root/admin管理員principal，使用管理員來新增principal
	```
	kinit root/admin
	```
	進入遠端的kadmin shell：需要輸入密碼
	```
	kadmin
	```
	建立principal
	```
	kadmin:  addprinc malvin@US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	```
	離開kadmin shell：與本地的shell一樣，exit離開
	```
	kadmin:  exit
	```
1. #### 為新user在HDFS建立個人目錄
	 **注意：這裡要使用「屬於HDFS的superuser group」的user才有權限在指定目錄創建新目錄**
	```
	hdfs dfs -mkdir /user/malvin
	hdfs dfs -chown malvin /user/malvin
	```
1. #### 初始化principal
	```
	kinit malvin@US-CENTRAL1-A.C.ETERNAL-RULER-310501.INTERNAL
	```
1. #### 嘗試執行一個MapReduce程式（跑pi）
	```
	hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-0.20-mapreduce/hadoop-examples.jar pi 10 10000
	```



