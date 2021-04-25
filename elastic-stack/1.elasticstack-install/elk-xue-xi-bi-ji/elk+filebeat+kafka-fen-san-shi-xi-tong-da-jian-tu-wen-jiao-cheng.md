# ELK+FileBeat+Kafka分散式系統搭建圖文教程

### &lt;/&gt; 工作流程

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229b9f4b71a8781e97b7e48f1249bf33b47c5e207f914a55b6b2ec464a87fe49de8b.jpg)

filebeat收集需要提取的日誌檔案， 將日誌檔案轉存到kafka叢集中，logstash處理kafka日誌，格式化處理，並將日誌輸出到elasticsearch中，前臺頁面通過kibana展示日誌。

使用kafka叢集做快取層， 而不是直接將filebeat收集到的日誌資訊寫入logstash，讓整體結構更健壯，減少網路環境，導致資料丟失。filebeat負責將收集到的資料寫入kafka，logstash取出資料並處理。

### &lt;/&gt;  硬體條件支援

#### 一共使用了4臺伺服器：

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b6f59aae95221362d26d6b32ad4a0f7c2fc8be98abf802440fb0407937ae294af.jpg)

#### 每臺伺服器都需要安裝jdk，配置環境變數。

修改全域性配置檔案 ，作用於所有使用者：

```text
sudo vi /etc/profileexport JAVA_HOME=JDK安裝路徑export PATH=$JAVA_HOME/bin:$PATH
```

#### 系統調優

```text
vim /etc/sysctl.conffs.file-max=65536vm.max_map_count = 262144
vim /etc/security/limits.conf* soft nofile 65535* hard nofile 131072* soft nproc 2048* hard nproc 4096
```

#### 使用的軟體版本

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229b186e4068e681aaaf08ec7af818d4303bc79c045503c2cc69a1b6bb1d32f809e3.jpg)

### &lt;/&gt;  應用安裝

#### kafka、zookeeper叢集搭建

在10.16.10.113、10.16.10.114、10.16.8.187伺服器中搭建kafka叢集，關閉防火牆

關閉防火牆命令：

```text
systemctl stop firewalld
```

檢視防火牆狀態：

```text
systemctl status firewalld
```

**zookeeper搭建**

本次直接使用kafka自帶的zookeeper，不需要單獨下載zookeeper

解壓安裝包到/usr/local/kafka目錄下

```text
vim config/zookeeper.properties
```

修改配置內容：

```text
clientPort=2181maxClientCnxns=100tickTime=2000initLimit=10syncLimit=5dataDir=/usr/local/kafka/zookeeper/datadataLogDir=/usr/local/kafka/zookeeper/logserver.1=10.16.10.113:12888:13888server.2=10.16.10.114:12888:13888server.3=10.16.8.187:12888:13888
```

注意：dataDir、dataLogDir檔案目錄需要手動建立。

三臺伺服器配置內容一致，需要在dataDir目錄下建立myid檔案，檔案的內容必須與zookeeper.properties中的編號保持一致。

**kafka搭建**

```text
vim config/server.properties
```

修改配置內容：

```text
broker.id=1prot = 9092host.name = 10.16.10.113num.network.threads=3num.io.threads=8socket.send.buffer.bytes=102400socket.receive.buffer.bytes=102400socket.request.max.bytes=104857600log.dirs=/usr/local/kafka-logsnum.partitions=16num.recovery.threads.per.data.dir=1offsets.topic.replication.factor=1transaction.state.log.replication.factor=1transaction.state.log.min.isr=1log.retention.hours=168log.segment.bytes=1073741824log.retention.check.interval.ms=300000zookeeper.connect=10.16.10.113:2181,10.16.10.114:2181,10.16.8.187:2181zookeeper.connection.timeout.ms=6000group.initial.rebalance.delay.ms=0
```

注意:每臺伺服器除broker.id 和 host.name 兩個屬性需要修改之外，其他屬性保持一致。

**驗證**

啟動zookeeper

```text
nohup sh zookeeper-server-start ../config/zookeeper.properties &
```

啟動kafka

```text
nohup sh kafka-server-start ../config/server.properties &
```

建立topic

```text
/usr/local/kafka/bin/kafka-topics.sh --create --zookeeper 10.16.10.113:2181,10.16.10.114:2181,10.16.8.187:2181 --replication-factor 1 --partitions 2   --topic  testtopic
```

檢視topic

```text
/usr/local/kafka/bin/kafka-topics.sh --zookeeper  10.16.10.113:2181,10.16.10.114:2181,10.16.8.187:2181  --list
```

寫入訊息:

命令：

```text
/usr/local/kafka/bin/kafka-console-producer.sh --broker-list 10.16.10.113:9092 --topic testtopic
```

消費訊息：

命令：

```text
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server   10.16.10.113:9092  --from-beginning --topic testtopic
```

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af9839366b4f17d7b85bd32ce1c061fb416c317322e59d54dc623971b052c3d6c0072c6.jpg)

能正常的寫入訊息、消費訊息，kafka叢集完成。

### &lt;/&gt;  ELK 搭建

在10.16.10.113、10.16.10.114、10.16.3.165搭建elasticsearch,，注意關閉防火牆，修改系統的配置，注意es的啟動必須是非root使用者啟動。

**安裝elasticsearch**

解壓檔案到/usr/local/目錄下,以10.16.3.165為主節點master

```text
vim elasticsearch/config/elasticsearch.yml
```

修改配置：

```text
###配置解釋# cluster.name 叢集名稱# node.name 節點主機名# node.master 是否參與主節點競選# node.data:true  指定該節點是否儲存索引資料，預設為true。本例沒配置，所有節點都儲存
```

包括主節點

```text
# discovery.zen.ping.unicast.hosts 配置上elasticsearch 叢集除本機外其他機器# cluster.initial_master_nodes 引導啟動叢集的機器IP或者主機名# http.port http埠，kibana中會用到 。# transport.tcp.port 設定節點間互動的tcp埠，預設是9300。cluster.name: elkmasternode.name: 10.16.3.165node.master: truepath.logs: /usr/local/data/log/network.host: 10.16.3.165http.port: 9200discovery.zen.ping.unicast.hosts: ["10.16.10.113","10.16.10.114"]cluster.initial_master_nodes: ["10.16.3.165"]
```

注意：其他兩臺伺服器，作為solver,需要修改cluster.name、node.name、network.host為自身的配置node.naster: false。最後兩個屬性根據伺服器內容進行修改。

**安裝kibana**

在10.16.3.165伺服器上安裝kibana

解壓檔案到/usr/local/目錄下

```text
vim kibana/config/kibana.yml
```

修改配置：

```text
# i18n.locale: "zh-CN"  web介面中文# server.port  監聽埠server.port: 5601server.host: "10.16.3.165"elasticsearch.hosts: ["http://10.16.3.165:9200"]i18n.locale: "zh-CN"
```

**驗證**

啟動之前切換為非root使用者。

啟動elasticsearch:

命令：

```text
nohup sh elasticsearch &# 正確的用法是下面/bin/elasticsearch -d
```

**啟動kibana**

命令：

```text
nohup sh kibana &
```

三臺伺服器訪問地址ip:9200，出現如圖結果說明elasticsearch啟動成功

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b22e92b145958c985fe613677df563f5b2e2393d7695629eb272b74d45e453e00.jpg)

10.16.3.165訪問10.16.3.165:5601，出現如下結果說明kibana啟動成功

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229bc3a34585ac7b7fe628452cf56b09fc310cf9ee2797a2722bee6f3efe2e77881a.jpg)

#### filebeat安裝

在10.16.3.166伺服器搭建filebeat服務

解壓檔案到/usr/local/目錄下

```text
vim filebeat/filebeat.yml
```

修改配置：

```text
filebeat.inputs:- type: log  enabled: true  paths:    - /data/home/app/domains/cpay_domain/logs/cpay-tms-gate.log output.kafka:    enable: true    hosts: ["10.16.8.187:9092"]    topic: es-tmslogs    compression: gzipmax_message_bytes: 100000
```

注意：paths表示需要提取的日誌的路徑，將日誌輸出到kafka中，建立topic

啟動filebeat:

命令：

```text
./filebeat -e -c filebeat.yml
```

#### logstash安裝

在10.16.3.165伺服器搭建logstash服務

解壓檔案到/usr/local/目錄下，建立用於本次處理的配置檔案logstashfortms.conf

```text
vim  logstash/config/logstashfortms.conf
```

配置內容：

```text
input{    kafka{        bootstrap_servers => "10.16.10.113:9092,10.16.10.114:9092,10.16.8.187:9092"        topics => ["es-tmslogs"]        codec => json    }}output{    elasticsearch {        hosts => ["10.16.3.165:9200"]        index => "logstash-%{+YYYY.MM.dd}"    }}
```

啟動logstash:

命令：

```text
nohup sh logstash -f  ../config/logesforcpay.conf &
```

### kibana頁面操作

安裝完成以後訪問10.16.3.165:5601頁面，選擇紅框中的按鈕

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b8e2c0e22043936d2c0443d886e9c7a848275b10123b23c6fab8987f8ded6d6cf.jpg)

進入索引建立模式

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af9839386ab3a3a48a61d5a6e1b0de95108fda3935c85ebb5630b19b8bd0be747f7e1e2.jpg)

建立索引之後，點選紅框內按鈕，即可展示日誌的資訊,服務搭建完成。

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af983931585df491bf30bc1d1e4ca44708c75f24e38fa7dfe171150224ff4354378483e.jpg)

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b8a88951680f567b2ad35697f70f1fdd0a81578a2c4e0483ebdea250ba84c1842.jpg)

