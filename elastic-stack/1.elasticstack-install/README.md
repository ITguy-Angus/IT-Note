---
description: ElasticStack 介紹與安裝
---

# 1.Elastic Stack 介紹與安裝

ELK Stack 簡介

     ELK是由Elasitc 公司開發並再2008年掛牌上市，ELK套件組成最初是由ElasticSearch、Logstash、Kibana 三個套件組成，後來又加入了Beat套件並改名為ELK Stack是一種數據處裡的框架，最先發名的組件為ElasticSearch是一個在2004為了收索食譜而發明收尋引擎，單當時他還叫做Compass就是了但在2010年就基於APACH Lucene 成為了開源軟體，後因ElasticSearch收到廣大歡迎催生了以下三個套件Logstash、Kibana、beat，並又針對商業應用開發了X-Pack擴展套件，下面就為各位列出各個套件分別不同的作用吧。

### 各組件應用

* Elasitcsearch  : 用於數據儲存與檢索，天然支持數據分片與複製，可輕鬆實現擴容，很多情況下被當成NoSql數據庫獨    立使用。
* Logstash : 用於數據清洗、數據傳輸，可以適配多種輸入和輸出數據源，通時提供了豐富的過濾器插件。
* Kibana : 用於數據可視化和數據分析，可看作是ElasticSearch的介面。
* Beat 分為各個不同的組件為各系統所應用
  1. Filebeat: Beats 組件中的一種，是一種安裝於宿主機的輕量級組件，核心作用就是將宿主機上指定的文件內容提取出來並發送到指定的目的地。



## ELK Stack 7.12 解壓安裝

### Elasticsearch 安裝

`cd /usr/local/`

`curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-linux-x86_64.tar.gz`

 `tar -xvf elasticsearch-7.12.0-linux-x86_64.tar.gz`

`rm elasticsearch-7.12.0-linux-x86_64.tar.gz`

`mv elasticsearch-7.12.0-linux-x86_64/elasticsearch`

`ln -s /usr/local/elasticsearch elasticsearch`

* 编辑 ./config/elasticsearch.yml

`vim /usr/local/elasticsearch-7.12.0/config/elasticsearch.yml`

```text
# 添加或修改
node.name: node-1
#network.host:0.0.0.0 代表開給外網連線
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

* 修改文件包含限制一个进程可以拥有的VMA\(虚拟内存区域\)的数量配置，编辑 /etc/sysctl.conf

`vim /etc/sysctl.conf`

```text
# 添加或修改
# sudo sysctl -p 使修改生效
vm.max_map_count = 262144
```

* 修改 ECS 安全组，放行 9200 端口

`ufw allow in 9200`

`ufw allow in 9300`

查看目前防火牆規則



```text
sudo ufw status # 查看目前防火牆規則
sudo ufw status numbered #以數字排列目前防火牆規則
```

*  創建帳戶

`adduser elasticsearch`

* 将对应的文件夹权限赋给该用户

`chown -R elasticsearch /usr/local/elasticsearch/elasticsearch-7.12.0`

* 切换至elasticsearch用户 

`su elasticsearch` 

* 进入启动目录启动 /usr/local/elasticsearch-7.12.0/bin 使用后台启动方式

`./elasticsearch -d`

* 測試是否成功啟動

 `curl -X GET "localhost:9200"`



### Kibana 安裝

`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz`

`tar -xvf kibana-7.12.0-linux-x86_64.tar.gz`

`rm kibana-7.12.0-linux-x86_64.tar.gz`

`mv kibana-7.12.0-linux-x86_64/ kibana`

`ln -s /usr/local/kibana kibana`

* 設定檔設定

`vim config/kibana.yml`

```text
server.port: 5601
server.host: "123.456.789.0"
server.name: "kibana-test"
elasticsearch.hosts: ["http://10.10.0.8:9200"]
# kibana会将部分数据写入es，这个是ex中索引的名字
kibana.index: ".kibana"
```

*  創建帳戶

`adduser kibana`

* 将对应的文件夹权限赋给该用户

`chown -R kibana /usr/local/kibana`

* 切换至kibana用户 

`su kibana`

* 进入启动目录启动 /usr/local/kibana/bin 使用后台启动方式

`./kibana &`

* 測試是否成功啟動

 `http://localhost:5601`

### Logstash 安裝

`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.12.0-linux-x86_64.tar.gz`

`tar -xvf logstash-7.12.0-linux-x86_64.tar.gz`

`rm logstash-7.12.0-linux-x86_64.tar.gz`

`mv logstash-7.12.0-linux-x86_64/ kibana`

`ln -s /usr/local/kibana kibana`

*  創建帳戶

`adduser kibana`

* 将对应的文件夹权限赋给该用户

`chown -R kibana/usr/local/kibana`

* 切换至kibana用户 

`su kibana`

* 进入启动目录启动 /usr/local/elasticsearch-7.12.0/bin 使用后台启动方式

`./kibana -d`

* 測試是否成功啟動

 `curl -X GET "localhost:5601"`

### Filebeat 安裝

`mkdir /usr/local/filebeat`

`cd /usr/local/filebeat`

`curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.0-linux-x86_64.tar.gz`

 `tar -xvf filebeat-7.12.0-linux-x86_64.tar.gz`

`rm filebeat-7.12.0-linux-x86_64.tar.gz`

 `cd ./filebeat-7.12.0/`





`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz`

`tar -xvf kibana-7.12.0-linux-x86_64.tar.gz`

`rm kibana-7.12.0-linux-x86_64.tar.gz`

`mv kibana-7.12.0-linux-x86_64/ kibana`

 `cd ./kibana/`

`ln -s /usr/local/kibana kibana`

*  創建帳戶

`adduser kibana`

* 将对应的文件夹权限赋给该用户

`chown -R kibana/usr/local/kibana`

* 切换至kibana用户 

`su kibana`

* 进入启动目录启动 /usr/local/elasticsearch-7.12.0/bin 使用后台启动方式

`./kibana -d`

* 測試是否成功啟動

 `curl -X GET "localhost:5601"`



