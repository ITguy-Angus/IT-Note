# X-Pack security

Elasticsearch Xpack認證

 阿新 • 來源：網路 • 發佈：2020-12-15  


**說明**

　　ES（在本文中，ES即Elasticsearch簡稱）的xpack一直都是要收費的，然而在6.8版本開始，ES開放了部分xpack認證功能，但一些高階功能仍舊收費（如單點登入以及對欄位級和文件級安全性的Active Directory / LDAP身份驗證仍然是付費功能）。

詳細可參看[官方說明（這個版本主要也是對認證進行了調整）。](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/release-highlights-6.8.0.html)

**概述**

　　本文主要針對ES 6.8.0 安全認證部署進行講解

**一、單機ES安全認證部署**

（1）下載[ES 6.8.0 linux 安裝包](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.0.tar.gz)

（2）上傳至伺服器並解壓

```text
# 解壓
sudo tar -zxvf elasticsearch-6.8.0.tar.gz
# 將 es 目錄授予 elasticsearch 使用者，因為 es 不允許使用 root 使用者啟動
sudo chown -R elasticsearch elasticsearch-6.8
```

（3）修改 ES config/elasticsearch.yml 配置，新增如下配置

```text
 xpack.security.audit.enabled: true
 xpack.security.enabled: true
 xpack.security.transport.ssl.enabled: true
```

（4）啟動ES

```text
# 切換使用者
su elasticsearch
# 啟動ES
./bin/elasticsrearch
```

（5）驗證

　　頁面訪問 ${IP}:9200，會彈出輸入使用者名稱密碼的視窗。ES有一些預設的使用者名稱（elastic、kibana、logstash等），但是需要自己設定密碼，否則登入不進去。

（6）給預設使用者設定密碼

```text
./bin/elasticsearch-setup-passwords interactive
```

設定完成之後選擇其中一個使用者再去登入即可訪問到ES節點資訊

**二、ES叢集安全認證部署**

（1）基於單機拷貝兩份作為另外兩個ES節點。一個master，兩個worker

```text
cp elasticsearch-6.8.0 elasticsearch-6.8.0-worker0
cp elasticsearch-6.8.0 elasticsearch-6.8.0-worker1
```

（2）三個節點ES配置檔案

```text
# 節點角色
node.name: master
node.master: true
node.data: false
# 資料以及日誌目錄
path.data: /opt/elasticsearch-6.8.0/data
path.logs: /opt/elasticsearch-6.8.0/logs
# http以及tcp埠,http用於API訪問，tcp用於叢集內部訪問
http.port: 9200
transport.tcp.port: 9300
# 叢集節點
discovery.zen.ping.unicast.hosts: ["xxx:9300","xxx:9301","xxx:9302"]
# xpack 訪問認證
 xpack.security.audit.enabled: true
 xpack.security.enabled: true
 xpack.security.transport.ssl.enabled: true
# xpack 叢集認證
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certificates.p12
xpack.security.transport.ssl.truststore.path: certificates.p12
```

```text
# 節點角色
node.name: worker0
node.master: false
node.data: true
# 資料以及日誌目錄
path.data: /opt/elasticsearch-6.8.0-worker0/data
path.logs: /opt/elasticsearch-6.8.0-worker0/logs
# http以及tcp埠,http用於API訪問，tcp用於叢集內部訪問
http.port: 9201
transport.tcp.port: 9301
# 叢集節點
discovery.zen.ping.unicast.hosts: ["xxx:9300","xxx:9301","xxx:9302"]
# xpack 訪問認證
 xpack.security.audit.enabled: true
 xpack.security.enabled: true
 xpack.security.transport.ssl.enabled: true
# xpack 叢集認證
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certificates.p12
xpack.security.transport.ssl.truststore.path: certificates.p12
```

```text
# 節點角色
node.name: worker1
node.master: false
node.data: true
# 資料以及日誌目錄
path.data: /opt/elasticsearch-6.8.0-worker1/data
path.logs: /opt/elasticsearch-6.8.0-worker1/logs
# http以及tcp埠,http用於API訪問，tcp用於叢集內部訪問
http.port: 9202
transport.tcp.port: 9302
# 叢集節點
discovery.zen.ping.unicast.hosts: ["xxx:9300","xxx:9301","xxx:9302"]
# xpack 訪問認證
 xpack.security.audit.enabled: true
 xpack.security.enabled: true
 xpack.security.transport.ssl.enabled: true
# xpack 叢集認證
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certificates.p12
xpack.security.transport.ssl.truststore.path: certificates.p12
```

（3）配置祕鑰檔案

　　- 生成ca檔案（在任何一個節點目錄下執行都行）

```text
./bin/elasticsearch-certutil ca
```

　　- 生成節點祕鑰檔案

```text
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

　　- 複製祕鑰檔案到各節點的指定目錄下（根據配置複製到具體目錄下，這裡是複製到config目錄下）

　　- 在各個節點執行以下命令設定密碼

```text
./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

（4）啟動各個節點並登陸驗證

**三、kibana ES**

（1）[下載kibana](https://artifacts.elastic.co/downloads/kibana/kibana-6.8.0-linux-x86_64.tar.gz)

（2）kibana 配置

```text
server.name: "xxxx"
# 連線es master節點
elasticsearch.url: "http://${ip}:9200"
# 認證賬號密碼
elasticsearch.username: "kibana"
elasticsearch.password: "xxxx"
# kibana預設訪問埠
server.port: 5601
server.host: "0.0.0.0"
# 監控（此配置開啟的話，會造成cpu監控為N/A，改成false即可）
xpack.monitoring.ui.container.elasticsearch.enabled: true
# 開啟認證
xpack.security.enabled: true
```

（3）啟動kibana

```text
./bin/kibana
```

（4）訪問

　${ip}:5601

**四、logstash ES**

（1）[下載logstash安裝包](https://artifacts.elastic.co/downloads/logstash/logstash-6.8.0.tar.gz)

（2）logstash 的 logstash.yml 配置

```text
# xpack 認證
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: "logstash"
xpack.monitoring.elasticsearch.password: "xxx"
xpack.monitoring.elasticsearch.url: ["${ip}:9200"]
```

（3）logstash 的 pipeline.yml 管道配置

```text
- pipeline.id: xxx-pipeline
  path.config: "/usr/share/logstash/pipelines/xxx.conf"
  pipeline.workers: 10
  pipeline.batch.size: 500
```

（4）logstash 管道連線 ES

```text
elasticsearch {
  action => "index"
  hosts => ["${ip}:9200"]
  index => "xxx"
　# 認證
  user => "logstash"
  password => "xxx"
  document_type => "xxx"
  document_id => "xxx"
  template => "/usr/share/logstash/es-template/xxx.json"
  template_name => "xxx"
  template_overwrite => true
}
```

（5）執行 logstash

```text
# 無需指定配置檔案，預設走pipelines.yml的配置
./bin/logstash
```

