# EBLK之2---EBLK



## EBLK之2---EBLK

阿新 • 來源：網路 • 發佈：2021-01-03  


## EBLK <a id="eblk"></a>

目錄

* [EBLK](https://www.796t.com/article.php?id=214345#eblk)
* [第1章 傳統日誌分析需求](https://www.796t.com/article.php?id=214345#%E7%AC%AC1%E7%AB%A0-%E5%82%B3%E7%B5%B1%E6%97%A5%E8%AA%8C%E5%88%86%E6%9E%90%E9%9C%80%E6%B1%82)
* [第2章 EBLK介紹](https://www.796t.com/article.php?id=214345#%E7%AC%AC2%E7%AB%A0-eblk%E4%BB%8B%E7%B4%B9)
  * [EBLK介紹](https://www.796t.com/article.php?id=214345#eblk%E4%BB%8B%E7%B4%B9)
  * [EBLK日誌收集流程](https://www.796t.com/article.php?id=214345#eblk%E6%97%A5%E8%AA%8C%E6%94%B6%E9%9B%86%E6%B5%81%E7%A8%8B)
* [第3章 EBL部署](https://www.796t.com/article.php?id=214345#%E7%AC%AC3%E7%AB%A0-ebl%E9%83%A8%E7%BD%B2)
  * [Elasticsearch單節點部署](https://www.796t.com/article.php?id=214345#elasticsearch%E5%96%AE%E7%AF%80%E9%BB%9E%E9%83%A8%E7%BD%B2)
  * [Kibana部署](https://www.796t.com/article.php?id=214345#kibana%E9%83%A8%E7%BD%B2)
  * [Elasticsearch-head部署](https://www.796t.com/article.php?id=214345#elasticsearch-head%E9%83%A8%E7%BD%B2)
* [第4章 收集普通格式的nginx日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC4%E7%AB%A0-%E6%94%B6%E9%9B%86%E6%99%AE%E9%80%9A%E6%A0%BC%E5%BC%8F%E7%9A%84nginx%E6%97%A5%E8%AA%8C)
  * [nginx部署](https://www.796t.com/article.php?id=214345#nginx%E9%83%A8%E7%BD%B2)
  * [nginx生成訪問日誌](https://www.796t.com/article.php?id=214345#nginx%E7%94%9F%E6%88%90%E8%A8%AA%E5%95%8F%E6%97%A5%E8%AA%8C)
  * [filebeat部署](https://www.796t.com/article.php?id=214345#filebeat%E9%83%A8%E7%BD%B2)
  * [filebeat啟動並檢視日誌驗證](https://www.796t.com/article.php?id=214345#filebeat%E5%95%9F%E5%8B%95%E4%B8%A6%E6%AA%A2%E8%A6%96%E6%97%A5%E8%AA%8C%E9%A9%97%E8%AD%89)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第5章 收集Json格式的Nginx日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC5%E7%AB%A0-%E6%94%B6%E9%9B%86json%E6%A0%BC%E5%BC%8F%E7%9A%84nginx%E6%97%A5%E8%AA%8C)
  * [不足與期望](https://www.796t.com/article.php?id=214345#%E4%B8%8D%E8%B6%B3%E8%88%87%E6%9C%9F%E6%9C%9B)
  * [Nginx改進](https://www.796t.com/article.php?id=214345#nginx%E6%94%B9%E9%80%B2)
  * [Filebeat改進](https://www.796t.com/article.php?id=214345#filebeat%E6%94%B9%E9%80%B2)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第6章 自定義Elasticsearch索引名稱](https://www.796t.com/article.php?id=214345#%E7%AC%AC6%E7%AB%A0-%E8%87%AA%E5%AE%9A%E7%BE%A9elasticsearch%E7%B4%A2%E5%BC%95%E5%90%8D%E7%A8%B1)
  * [不足與期望](https://www.796t.com/article.php?id=214345#%E4%B8%8D%E8%B6%B3%E8%88%87%E6%9C%9F%E6%9C%9B)
  * [Filebeat改進](https://www.796t.com/article.php?id=214345#filebeat%E6%94%B9%E9%80%B2)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第7章 按日誌型別定義索引名稱](https://www.796t.com/article.php?id=214345#%E7%AC%AC7%E7%AB%A0-%E6%8C%89%E6%97%A5%E8%AA%8C%E5%9E%8B%E5%88%A5%E5%AE%9A%E7%BE%A9%E7%B4%A2%E5%BC%95%E5%90%8D%E7%A8%B1)
  * [不足與期望](https://www.796t.com/article.php?id=214345#%E4%B8%8D%E8%B6%B3%E8%88%87%E6%9C%9F%E6%9C%9B)
  * [Filebeat改進](https://www.796t.com/article.php?id=214345#filebeat%E6%94%B9%E9%80%B2)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第8章 使用ES-ingest 節點轉換Nginx普通日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC8%E7%AB%A0-%E4%BD%BF%E7%94%A8es-ingest-%E7%AF%80%E9%BB%9E%E8%BD%89%E6%8F%9Bnginx%E6%99%AE%E9%80%9A%E6%97%A5%E8%AA%8C)
  * [Ingest 節點介紹](https://www.796t.com/article.php?id=214345#ingest-%E7%AF%80%E9%BB%9E%E4%BB%8B%E7%B4%B9)
  * [Grok 介紹](https://www.796t.com/article.php?id=214345#grok-%E4%BB%8B%E7%B4%B9)
  * [預處理日誌流程](https://www.796t.com/article.php?id=214345#%E9%A0%90%E8%99%95%E7%90%86%E6%97%A5%E8%AA%8C%E6%B5%81%E7%A8%8B)
  * [ES匯入pipeline規則](https://www.796t.com/article.php?id=214345#es%E5%8C%AF%E5%85%A5pipeline%E8%A6%8F%E5%89%87)
  * [Nginx配置](https://www.796t.com/article.php?id=214345#nginx%E9%85%8D%E7%BD%AE)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第9章 使用filebeat模組收集Nginx普通日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC9%E7%AB%A0-%E4%BD%BF%E7%94%A8filebeat%E6%A8%A1%E7%B5%84%E6%94%B6%E9%9B%86nginx%E6%99%AE%E9%80%9A%E6%97%A5%E8%AA%8C)
  * [Filebeat模組介紹](https://www.796t.com/article.php?id=214345#filebeat%E6%A8%A1%E7%B5%84%E4%BB%8B%E7%B4%B9)
  * [Filebeat工作流程](https://www.796t.com/article.php?id=214345#filebeat%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [Nginx配置](https://www.796t.com/article.php?id=214345#nginx%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
  * [建立監控視圖面板](https://www.796t.com/article.php?id=214345#%E5%BB%BA%E7%AB%8B%E7%9B%A3%E6%8E%A7%E8%A6%96%E5%9C%96%E9%9D%A2%E6%9D%BF)
    * [建立一個柱狀圖](https://www.796t.com/article.php?id=214345#%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%80%8B%E6%9F%B1%E7%8B%80%E5%9C%96)
    * [建立一個URL圖](https://www.796t.com/article.php?id=214345#%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%80%8Burl%E5%9C%96)
    * [建立一個餅狀圖](https://www.796t.com/article.php?id=214345#%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%80%8B%E9%A4%85%E7%8B%80%E5%9C%96)
    * [建立一個儀表盤](https://www.796t.com/article.php?id=214345#%E5%BB%BA%E7%AB%8B%E4%B8%80%E5%80%8B%E5%84%80%E8%A1%A8%E7%9B%A4)
* [第10章 使用模組收集MySQL慢日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC10%E7%AB%A0-%E4%BD%BF%E7%94%A8%E6%A8%A1%E7%B5%84%E6%94%B6%E9%9B%86mysql%E6%85%A2%E6%97%A5%E8%AA%8C)
  * [mysql日誌介紹](https://www.796t.com/article.php?id=214345#mysql%E6%97%A5%E8%AA%8C%E4%BB%8B%E7%B4%B9)
  * [MySQL 5.7二進位制部署](https://www.796t.com/article.php?id=214345#mysql-57%E4%BA%8C%E9%80%B2%E4%BD%8D%E5%88%B6%E9%83%A8%E7%BD%B2)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第11章 收集tomcat的json日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC11%E7%AB%A0-%E6%94%B6%E9%9B%86tomcat%E7%9A%84json%E6%97%A5%E8%AA%8C)
  * [Tomcat日誌介紹](https://www.796t.com/article.php?id=214345#tomcat%E6%97%A5%E8%AA%8C%E4%BB%8B%E7%B4%B9)
  * [Tomcat部署](https://www.796t.com/article.php?id=214345#tomcat%E9%83%A8%E7%BD%B2)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第12章 收集Java多行日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC12%E7%AB%A0-%E6%94%B6%E9%9B%86java%E5%A4%9A%E8%A1%8C%E6%97%A5%E8%AA%8C)
  * [介紹](https://www.796t.com/article.php?id=214345#%E4%BB%8B%E7%B4%B9)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第13章 使用Redis快取](https://www.796t.com/article.php?id=214345#%E7%AC%AC13%E7%AB%A0-%E4%BD%BF%E7%94%A8redis%E5%BF%AB%E5%8F%96)
  * [使用Redis快取的日誌收集流程](https://www.796t.com/article.php?id=214345#%E4%BD%BF%E7%94%A8redis%E5%BF%AB%E5%8F%96%E7%9A%84%E6%97%A5%E8%AA%8C%E6%94%B6%E9%9B%86%E6%B5%81%E7%A8%8B)
  * [Nginx日誌改為json格式](https://www.796t.com/article.php?id=214345#nginx%E6%97%A5%E8%AA%8C%E6%94%B9%E7%82%BAjson%E6%A0%BC%E5%BC%8F)
  * [Redis部署](https://www.796t.com/article.php?id=214345#redis%E9%83%A8%E7%BD%B2)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [Logstash部署](https://www.796t.com/article.php?id=214345#logstash%E9%83%A8%E7%BD%B2)
  * [多Redis主備配置](https://www.796t.com/article.php?id=214345#%E5%A4%9Aredis%E4%B8%BB%E5%82%99%E9%85%8D%E7%BD%AE)
* [第14章 使用kafka快取](https://www.796t.com/article.php?id=214345#%E7%AC%AC14%E7%AB%A0-%E4%BD%BF%E7%94%A8kafka%E5%BF%AB%E5%8F%96)
  * [使用kafka為快取的日誌收集流程](https://www.796t.com/article.php?id=214345#%E4%BD%BF%E7%94%A8kafka%E7%82%BA%E5%BF%AB%E5%8F%96%E7%9A%84%E6%97%A5%E8%AA%8C%E6%94%B6%E9%9B%86%E6%B5%81%E7%A8%8B)
  * [所有節點配置hosts和金鑰](https://www.796t.com/article.php?id=214345#%E6%89%80%E6%9C%89%E7%AF%80%E9%BB%9E%E9%85%8D%E7%BD%AEhosts%E5%92%8C%E9%87%91%E9%91%B0)
  * [zookeeper叢集部署](https://www.796t.com/article.php?id=214345#zookeeper%E5%8F%A2%E9%9B%86%E9%83%A8%E7%BD%B2)
  * [kafka叢集部署](https://www.796t.com/article.php?id=214345#kafka%E5%8F%A2%E9%9B%86%E9%83%A8%E7%BD%B2)
  * [Filebeat配置](https://www.796t.com/article.php?id=214345#filebeat%E9%85%8D%E7%BD%AE)
  * [Logstash配置](https://www.796t.com/article.php?id=214345#logstash%E9%85%8D%E7%BD%AE)
  * [檢查收集結果](https://www.796t.com/article.php?id=214345#%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第15章 EBLK安全認證配置](https://www.796t.com/article.php?id=214345#%E7%AC%AC15%E7%AB%A0-eblk%E5%AE%89%E5%85%A8%E8%AA%8D%E8%AD%89%E9%85%8D%E7%BD%AE)
  * [1.EBLK認證介紹](https://www.796t.com/article.php?id=214345#1eblk%E8%AA%8D%E8%AD%89%E4%BB%8B%E7%B4%B9)
  * [2.Elasticsearch配置賬號密碼](https://www.796t.com/article.php?id=214345#2elasticsearch%E9%85%8D%E7%BD%AE%E8%B3%AC%E8%99%9F%E5%AF%86%E7%A2%BC)
  * [3.修改filebeat配置檔案](https://www.796t.com/article.php?id=214345#3%E4%BF%AE%E6%94%B9filebeat%E9%85%8D%E7%BD%AE%E6%AA%94%E6%A1%88)
  * [4.修改logstash配置檔案](https://www.796t.com/article.php?id=214345#4%E4%BF%AE%E6%94%B9logstash%E9%85%8D%E7%BD%AE%E6%AA%94%E6%A1%88)
  * [5.修改kibana配置檔案](https://www.796t.com/article.php?id=214345#5%E4%BF%AE%E6%94%B9kibana%E9%85%8D%E7%BD%AE%E6%AA%94%E6%A1%88)
  * [6.通過kibana設定不同使用者的許可權](https://www.796t.com/article.php?id=214345#6%E9%80%9A%E9%81%8Ekibana%E8%A8%AD%E5%AE%9A%E4%B8%8D%E5%90%8C%E4%BD%BF%E7%94%A8%E8%80%85%E7%9A%84%E8%A8%B1%E5%8F%AF%E6%AC%8A)
  * [7.檢查實驗效果是否符合期望效果](https://www.796t.com/article.php?id=214345#7%E6%AA%A2%E6%9F%A5%E5%AF%A6%E9%A9%97%E6%95%88%E6%9E%9C%E6%98%AF%E5%90%A6%E7%AC%A6%E5%90%88%E6%9C%9F%E6%9C%9B%E6%95%88%E6%9E%9C)
* [第16章 使用EBK收集k8s的pod日誌](https://www.796t.com/article.php?id=214345#%E7%AC%AC16%E7%AB%A0-%E4%BD%BF%E7%94%A8ebk%E6%94%B6%E9%9B%86k8s%E7%9A%84pod%E6%97%A5%E8%AA%8C)
  * [1.k8s日誌收集流程介紹](https://www.796t.com/article.php?id=214345#1k8s%E6%97%A5%E8%AA%8C%E6%94%B6%E9%9B%86%E6%B5%81%E7%A8%8B%E4%BB%8B%E7%B4%B9)
  * [2.k8s環境快速搭建部署](https://www.796t.com/article.php?id=214345#2k8s%E7%92%B0%E5%A2%83%E5%BF%AB%E9%80%9F%E6%90%AD%E5%BB%BA%E9%83%A8%E7%BD%B2)
  * [3.k8s交付EBK](https://www.796t.com/article.php?id=214345#3k8s%E4%BA%A4%E4%BB%98ebk)
  * [4.k8s交付邊車模式的NginxPOD](https://www.796t.com/article.php?id=214345#4k8s%E4%BA%A4%E4%BB%98%E9%82%8A%E8%BB%8A%E6%A8%A1%E5%BC%8F%E7%9A%84nginxpod)
  * [5.創造訪問日誌並檢查收集結果](https://www.796t.com/article.php?id=214345#5%E5%89%B5%E9%80%A0%E8%A8%AA%E5%95%8F%E6%97%A5%E8%AA%8C%E4%B8%A6%E6%AA%A2%E6%9F%A5%E6%94%B6%E9%9B%86%E7%B5%90%E6%9E%9C)
* [第17章 EBK最大架構](https://www.796t.com/article.php?id=214345#%E7%AC%AC17%E7%AB%A0-ebk%E6%9C%80%E5%A4%A7%E6%9E%B6%E6%A7%8B)

## 第1章 傳統日誌分析需求 <a id="&#x7B2C;1&#x7AE0;-&#x50B3;&#x7D71;&#x65E5;&#x8A8C;&#x5206;&#x6790;&#x9700;&#x6C42;"></a>

```text
1.找出訪問排名前十的IP,URL
2.找出10點到12點之間排名前十的IP,URL
3.對比昨天這個時間段訪問情況有什麼變化
4.對比上個星期同一天同一時間段的訪問變化
5.找出搜尋引擎訪問的次數和每個搜尋弓|擎各訪問了多少次
6.指定域名的關鍵連結訪問次數,響應時間
7.網站HTTP狀態碼情況
8.找出攻擊者的IP地址,這個IP訪問了什麼頁面，這個IP什麼時候來的，什麼時候走的共訪問了多少次
9.5分鐘內告訴為結果
```

## 第2章 EBLK介紹 <a id="&#x7B2C;2&#x7AE0;-eblk&#x4ECB;&#x7D39;"></a>

### EBLK介紹 <a id="eblk&#x4ECB;&#x7D39;"></a>

```text
E   Elasticsearch    java
B   Filebeat         Go
L   Logstash         java
K   Kibana           java
```

### EBLK日誌收集流程 <a id="eblk&#x65E5;&#x8A8C;&#x6536;&#x96C6;&#x6D41;&#x7A0B;"></a>

## 第3章 EBL部署 <a id="&#x7B2C;3&#x7AE0;-ebl&#x90E8;&#x7F72;"></a>

### Elasticsearch單節點部署 <a id="elasticsearch&#x55AE;&#x7BC0;&#x9EDE;&#x90E8;&#x7F72;"></a>

```text
rpm -ivh elasticsearch-7.9.1-x86_64.rpm
cat > /etc/elasticsearch/elasticsearch.yml << 'EOF'    
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 127.0.0.1,10.0.0.51
http.port: 9200
discovery.seed_hosts: ["10.0.0.51"]
cluster.initial_master_nodes: ["10.0.0.51"]
EOF
systemctl daemon-reload
systemctl start elasticsearch.service
netstat -lntup|grep 9200
curl 127.0.0.1:9200
```

> 有問題，停止，清除，啟動
>
> ```text
> systemctl stop elasticsearch.service
> rm -rf /var/lib/elasticsearch/*
> systemctl start elasticsearch.service
> ```

### Kibana部署 <a id="kibana&#x90E8;&#x7F72;"></a>

```text
rpm -ivh kibana-7.9.1-x86_64.rpm
cat > /etc/kibana/kibana.yml << 'EOF'
server.port: 5601
server.host: "10.0.0.51"
elasticsearch.hosts: ["http://10.0.0.51:9200"]
kibana.index: ".kibana"
EOF
systemctl start kibana
```

> 有問題，停止，清除，啟動
>
> ```text
> systemctl stop kibana.service
> rm -rf /var/lib/kibana/*
> # ES刪除kibana索引
> systemctl start kibana
> ```

### Elasticsearch-head部署 <a id="elasticsearch-head&#x90E8;&#x7F72;"></a>

google瀏覽器 --&gt; 更多工具 --&gt; 擴充套件程式 --&gt; 開發者模式 --&gt; 選擇解壓縮後的外掛目錄

## 第4章 收集普通格式的nginx日誌 <a id="&#x7B2C;4&#x7AE0;-&#x6536;&#x96C6;&#x666E;&#x901A;&#x683C;&#x5F0F;&#x7684;nginx&#x65E5;&#x8A8C;"></a>

### nginx部署 <a id="nginx&#x90E8;&#x7F72;"></a>

```text
cat > /etc/yum.repos.d/nginx.repo <<'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=0
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
EOF

yum install nginx -y
systemctl start nginx
```

### nginx生成訪問日誌 <a id="nginx&#x751F;&#x6210;&#x8A2A;&#x554F;&#x65E5;&#x8A8C;"></a>

```text
for i in `seq 10`;do curl -I 127.0.0.1 &>/dev/null ;done
```

### filebeat部署 <a id="filebeat&#x90E8;&#x7F72;"></a>

```text
rpm -ivh filebeat-7.9.1-x86_64.rpm
cp /etc/filebeat/filebeat.yml /opt/
cat > /etc/filebeat/filebeat.yml << EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
EOF
```

> 有問題，停止，清除，啟動
>
> ```text
> systemctl stop filebeat.service
> rm -rf /var/lib/filebeat/*
> systemctl start filebeat.service
> ```

### filebeat啟動並檢視日誌驗證 <a id="filebeat&#x555F;&#x52D5;&#x4E26;&#x6AA2;&#x8996;&#x65E5;&#x8A8C;&#x9A57;&#x8B49;"></a>

```text
systemctl start filebeat
tail -f /var/log/filebeat/filebeat
```

* filebeat匯入的索引：`filebeat-7.9.1-2020.12.29-000001`
* 索引生命週期管理的索引：`ilm-history-2-000001`

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;"></a>

1. 訪問http://10.0.0.51:5601/
2. [Connect to your Elasticsearch index](http://10.0.0.51:5601/app/management/kibana/indexPatterns)
3. **Index pattern name**
4. **選擇時間順序**
5. **檢視日誌：主頁 --&gt; Discover**

## 第5章 收集Json格式的Nginx日誌 <a id="&#x7B2C;5&#x7AE0;-&#x6536;&#x96C6;json&#x683C;&#x5F0F;&#x7684;nginx&#x65E5;&#x8A8C;"></a>

[官方文件](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html)

### 不足與期望 <a id="&#x4E0D;&#x8DB3;&#x8207;&#x671F;&#x671B;"></a>

**當前日誌收集方案的不足**：

* 所有日誌都儲存在message的value裡,不能拆分單獨顯示
* 要想單獨顯示，就得想辦法把日誌欄位拆分開，變成json格式

**我們期望的日誌收集效果**：可以把日誌所有欄位拆分出來

```text
{
    "time_local": "24/Dec/2020:09:43:45 +0800",
    "remote_addr": "127.0.0.1",
    "referer": "-",
    "request": "HEAD / HTTP/1.1",
    "status": 200,
    "bytes": 0,
    "http_user_agent": "curl/7.29.0",
    "x_forwarded": "-",
    "up_addr": "-",
    "up_host": "-",
    "upstream_time": "-",
    "request_time": "0.000"
}
```

### Nginx改進 <a id="nginx&#x6539;&#x9032;"></a>

1. **修改配置檔案**

```text
vi /etc/nginx/nginx.conf
```

```text
log_format json '{ "time_local": "$time_local", '
                          '"remote_addr": "$remote_addr", '
                          '"referer": "$http_referer", '
                          '"request": "$request", '
                          '"status": $status, '
                          '"bytes": $body_bytes_sent, '
                          '"http_user_agent": "$http_user_agent", '
                          '"x_forwarded": "$http_x_forwarded_for", '
                          '"up_addr": "$upstream_addr",'
                          '"up_host": "$upstream_http_host",'
                          '"upstream_time": "$upstream_response_time",'
                          '"request_time": "$request_time"'
    ' }';
    access_log  /var/log/nginx/access.log  json;
```

1. **檢查並重啟nginx，清空舊日誌，建立新日誌，檢查是否為json格式**

```text
nginx -t
systemctl restart nginx
> /var/log/nginx/access.log
for i in `seq 10`;do curl -I 127.0.0.1 &>/dev/null ;done
cat /var/log/nginx/access.log
```

### Filebeat改進 <a id="filebeat&#x6539;&#x9032;"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml << EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
EOF
```

1. **清除之前filebeat匯入ES的資料**
2. **重啟filebeat生效，重新將JSON日誌匯入ES**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-1"></a>

1. **訪問**[http://10.0.0.51:5601/](http://10.0.0.51:5601/)
2. [Connect to your Elasticsearch index](http://10.0.0.51:5601/app/management/kibana/indexPatterns)
3. **移除之前建立的`filebeat-7.9.1`**
4. **建立新的索引模式（Index patterns），流程同上**
5. **可以篩選日誌所有欄位**

## 第6章 自定義Elasticsearch索引名稱 <a id="&#x7B2C;6&#x7AE0;-&#x81EA;&#x5B9A;&#x7FA9;elasticsearch&#x7D22;&#x5F15;&#x540D;&#x7A31;"></a>

### 不足與期望 <a id="&#x4E0D;&#x8DB3;&#x8207;&#x671F;&#x671B;-1"></a>

當前日誌收集方案的不足：`filebeat-7.9.1-2020.12.29-000001`

* 雖然日誌可以拆分了，但是索引名稱還是預設的，根據索引名稱並不能看出來收集的是什麼日誌
* 每天建立一個索引

我們期望的日誌收集效果：`nginx-7.9.1-2020.12`

* 自定義索引名稱
* 每月的日誌存在一個索引中

### Filebeat改進 <a id="filebeat&#x6539;&#x9032;-1"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  index: "nginx-%{[agent.version]}-%{+yyyy.MM}"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-2"></a>

1. **清除之前filebeat匯入ES的資料**
2. **nginx生成新的訪問日誌，稍等**

```text
for i in `seq 10`;do curl -I 127.0.0.1 &>/dev/null ;done
```

## 第7章 按日誌型別定義索引名稱 <a id="&#x7B2C;7&#x7AE0;-&#x6309;&#x65E5;&#x8A8C;&#x578B;&#x5225;&#x5B9A;&#x7FA9;&#x7D22;&#x5F15;&#x540D;&#x7A31;"></a>

### 不足與期望 <a id="&#x4E0D;&#x8DB3;&#x8207;&#x671F;&#x671B;-2"></a>

當前日誌收集方案的不足：只有訪問日誌，沒有錯誤日誌

我們期望的日誌收集效果：

* `nginx-access-7.9.1-2020.12`
* `nginx-error-7.9.1-2020.12`

### Filebeat改進 <a id="filebeat&#x6539;&#x9032;-2"></a>

1. **修改配置檔案**
   * 以指定項，作為某類索引名的篩選條件
   * 以自定義標籤項，作為某類索引名的篩選條件

```text
cat > /etc/filebeat/filebeat.yml << EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/access.log"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/error.log"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

```text
cat > /etc/filebeat/filebeat.yml << EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
  
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-3"></a>

1. **nginx生成新的錯誤日誌，稍等**

```text
for i in `seq 10`;do curl -I 127.0.0.1/1 &>/dev/null ;done
```

## 第8章 使用ES-ingest 節點轉換Nginx普通日誌 <a id="&#x7B2C;8&#x7AE0;-&#x4F7F;&#x7528;es-ingest-&#x7BC0;&#x9EDE;&#x8F49;&#x63DB;nginx&#x666E;&#x901A;&#x65E5;&#x8A8C;"></a>

### Ingest 節點介紹 <a id="ingest-&#x7BC0;&#x9EDE;&#x4ECB;&#x7D39;"></a>

ingest 節點可以看作是ES資料前置處理轉換的節點，支援 pipeline管道 設定，可以使用 ingest 對資料進行過濾、轉換等操作，類似於 logstash 中 filter 的作用，功能相當強大。

Ingest 節點是 Elasticsearch 5.0 新增的節點型別和功能。

Ingest 節點的基礎原理，是：節點接收到資料之後，根據請求引數中指定的管道流 id，找到對應的已註冊管道流，對資料進行處理，然後將處理過後的資料，按照 Elasticsearch 標準的 indexing 流程繼續執行。

### Grok 介紹 <a id="grok-&#x4ECB;&#x7D39;"></a>

Grok 轉換語法

```text
127.0.0.1 							==> %{IP:clientip}
- 									==> -
- 									==> -
[08/Oct/2020:16:34:40 +0800] 		==> \\[%{HTTPDATE:nginx.access.time}\\]
"GET / HTTP/1.1" 					==> "%{DATA:nginx.access.info}"
200 								==> %{NUMBER:http.response.status_code:long} 
5 									==> %{NUMBER:http.response.body.bytes:long}
"-" 								==> "(-|%{DATA:http.request.referrer})"
"curl/7.29.0" 						==> "(-|%{DATA:user_agent.original})"
"-"									==> "(-|%{IP:clientip})"
```

Sample Data 通過 Grok Pattern 轉換為JSON格式

* Sample Data

```text
127.0.0.1 - - [29/Dec/2020:16:30:48 +0800] "HEAD /1 HTTP/1.1" 404 0 "-" "curl/7.29.0" "-"
```

* Grok Pattern

```text
%{IP:clientip} - - \[%{HTTPDATE:nginx.access.time}\] \"%{DATA:nginx.access.info}\" %{NUMBER:http.response.status_code:long} %{NUMBER:http.response.body.bytes:long} \"(-|%{DATA:http.request.referrer})\" \"(-|%{DATA:user_agent.original})\"
```

### 預處理日誌流程 <a id="&#x9810;&#x8655;&#x7406;&#x65E5;&#x8A8C;&#x6D41;&#x7A0B;"></a>

filebaet輸出日誌，先到ingest節點，通過 pipeline 指令碼匹配 Grok Pattern 將普通格式轉換為 Json格式，再存入ES。

### ES匯入pipeline規則 <a id="es&#x532F;&#x5165;pipeline&#x898F;&#x5247;"></a>

[官方文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/handling-failure-in-pipelines.html)

```text
GET _ingest/pipeline
PUT _ingest/pipeline/pipeline-nginx-access
{
  "description" : "nginx access log",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{IP:clientip} - - \\[%{HTTPDATE:nginx.access.time}\\] \"%{DATA:nginx.access.info}\" %{NUMBER:http.response.status_code:long} %{NUMBER:http.response.body.bytes:long} \"(-|%{DATA:http.request.referrer})\" \"(-|%{DATA:user_agent.original})\""]
      }
    },{
      "remove": {
        "field": "message"
      }
    }
  ]
}
```

### Nginx配置 <a id="nginx&#x914D;&#x7F6E;"></a>

1. **修改配置檔案**

```text
vi /etc/nginx/nginx.conf
```

```text
    access_log  /var/log/nginx/access.log  main;
```

1. **重啟nginx，清空舊日誌，建立新日誌，檢查是否為普通格式**

```text
systemctl restart nginx
> /var/log/nginx/access.log
> /var/log/nginx/error.log
for i in `seq 10`;do curl -I 127.0.0.1/1 &>/dev/null ;done
cat /var/log/nginx/access.log
cat /var/log/nginx/error.log
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml << EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

processors:
  - drop_fields:
      fields: ["ecs","log"]

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]

  pipelines:
    - pipeline: "pipeline-nginx-access"
      when.contains:
        tags: "access"

  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-4"></a>

1. **清除之前filebeat匯入ES的資料**
2. **nginx生成新的訪問日誌，稍等**

## 第9章 使用filebeat模組收集Nginx普通日誌 <a id="&#x7B2C;9&#x7AE0;-&#x4F7F;&#x7528;filebeat&#x6A21;&#x7D44;&#x6536;&#x96C6;nginx&#x666E;&#x901A;&#x65E5;&#x8A8C;"></a>

### Filebeat模組介紹 <a id="filebeat&#x6A21;&#x7D44;&#x4ECB;&#x7D39;"></a>

```text
[root@node-1 soft]# ls /etc/filebeat/modules.d/
activemq.yml.disabled       haproxy.yml.disabled    nginx.yml.disabled
apache.yml.disabled         ibmmq.yml.disabled      o365.yml.disabled
auditd.yml.disabled         icinga.yml.disabled     okta.yml.disabled
aws.yml.disabled            iis.yml.disabled        osquery.yml.disabled
azure.yml.disabled          imperva.yml.disabled    panw.yml.disabled
barracuda.yml.disabled      infoblox.yml.disabled   postgresql.yml.disabled
bluecoat.yml.disabled       iptables.yml.disabled   rabbitmq.yml.disabled
cef.yml.disabled            juniper.yml.disabled    radware.yml.disabled
checkpoint.yml.disabled     kafka.yml.disabled      redis.yml.disabled
cisco.yml.disabled          kibana.yml.disabled     santa.yml.disabled
coredns.yml.disabled        logstash.yml.disabled   sonicwall.yml.disabled
crowdstrike.yml.disabled    microsoft.yml.disabled  sophos.yml.disabled
cylance.yml.disabled        misp.yml.disabled       squid.yml.disabled
elasticsearch.yml.disabled  mongodb.yml.disabled    suricata.yml.disabled
envoyproxy.yml.disabled     mssql.yml.disabled      system.yml.disabled
f5.yml.disabled             Mysql.yml.disabled      tomcat.yml.disabled
fortinet.yml.disabled       nats.yml.disabled       traefik.yml.disabled
googlecloud.yml.disabled    netflow.yml.disabled    zeek.yml.disabled
gsuite.yml.disabled         netscout.yml.disabled   zscaler.yml.disabled
```

### Filebeat工作流程 <a id="filebeat&#x5DE5;&#x4F5C;&#x6D41;&#x7A0B;"></a>

參考文件：[Filebeat](https://www.cnblogs.com/cjsblog/p/9495024.html)

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-1"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.config.modules:
  path: \${path.config}/modules.d/*.yml
  reload.enabled: enable

filebeat.modules:
- module: nginx

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/access.log"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/error.log"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **啟用filebeat模組：nginx**

```text
filebeat modules list
filebeat modules enable nginx
filebeat modules list
```

> 啟用實質上就是重新命名：
>
> ```text
> mv /etc/filebeat/modules.d/nginx.yml.disabled /etc/filebeat/modules.d/nginx.yml
> ```

1. **配置filebeat模組nginx：配置日誌路徑**

```text
cat > /etc/filebeat/modules.d/nginx.yml << 'EOF'
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log"]
EOF
```

1. **一個BUG：刪除多餘的規則**

```text
rm -rf /usr/share/filebeat/module/nginx/ingress_controller
```

> filebeat模組`nginx`下有三套`pipeline.yml`規則：
>
> ```text
> [root@node-1 ~]# tree /usr/share/filebeat/module/nginx
> /usr/share/filebeat/module/nginx
> |-- access
> |   |-- config
> |   |   `-- nginx-access.yml
> |   |-- ingest
> |   |   `-- pipeline.yml
> |   `-- manifest.yml
> |-- error
> |   |-- config
> |   |   `-- nginx-error.yml
> |   |-- ingest
> |   |   `-- pipeline.yml
> |   `-- manifest.yml
> |-- ingress_controller
> |   |-- config
> |   |   `-- ingress_controller.yml
> |   |-- ingest
> |   |   `-- pipeline.yml
> |   `-- manifest.yml
> `-- module.yml
>
> 9 directories, 10 files
> ```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### Nginx配置 <a id="nginx&#x914D;&#x7F6E;-1"></a>

1. **修改配置檔案**

```text
vi /etc/nginx/nginx.conf
```

```text
    access_log  /var/log/nginx/access.log  main;
```

1. **重啟nginx，清空舊日誌，建立新日誌，檢查是否為普通格式**

```text
systemctl restart nginx
> /var/log/nginx/access.log
> /var/log/nginx/error.log
for i in `seq 10`;do curl -I 127.0.0.1/1 &>/dev/null ;done
cat /var/log/nginx/access.log
cat /var/log/nginx/error.log
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-5"></a>

1. **檢視ES資料**
2. **訪問**[http://10.0.0.51:5601/](http://10.0.0.51:5601/)
3. [Connect to your Elasticsearch index](http://10.0.0.51:5601/app/management/kibana/indexPatterns)
4. **移除之前建立的`filebeat-7.9.1`，建立新的索引模式（Index patterns），流程同上**

### 建立監控視圖面板 <a id="&#x5EFA;&#x7ACB;&#x76E3;&#x63A7;&#x8996;&#x5716;&#x9762;&#x677F;"></a>

#### 建立一個柱狀圖 <a id="&#x5EFA;&#x7ACB;&#x4E00;&#x500B;&#x67F1;&#x72C0;&#x5716;"></a>

訪問排名前十的IP

#### 建立一個URL圖 <a id="&#x5EFA;&#x7ACB;&#x4E00;&#x500B;url&#x5716;"></a>

#### 建立一個餅狀圖 <a id="&#x5EFA;&#x7ACB;&#x4E00;&#x500B;&#x9905;&#x72C0;&#x5716;"></a>

#### 建立一個儀表盤 <a id="&#x5EFA;&#x7ACB;&#x4E00;&#x500B;&#x5100;&#x8868;&#x76E4;"></a>

監控儀表盤：資料實時更新，點選過濾，+ Add filter 排除

## 第10章 使用模組收集MySQL慢日誌 <a id="&#x7B2C;10&#x7AE0;-&#x4F7F;&#x7528;&#x6A21;&#x7D44;&#x6536;&#x96C6;mysql&#x6162;&#x65E5;&#x8A8C;"></a>

### mysql日誌介紹 <a id="mysql&#x65E5;&#x8A8C;&#x4ECB;&#x7D39;"></a>

| 型別 | 檔案 |
| :--- | :--- |
| 查詢日誌 | general\_log |
| 慢查詢日誌 | log\_slow\_queries |
| 錯誤日誌 | log\_error， log\_warnings |
| 二進位制日誌 | binlog |
| 中繼日誌 | relay\_log |

慢查詢：執行時間超出指定時長的查詢。

### MySQL 5.7二進位制部署 <a id="mysql-57&#x4E8C;&#x9032;&#x4F4D;&#x5236;&#x90E8;&#x7F72;"></a>

1. **下載並安裝**

```text
cd /data/soft
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
tar xf mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz -C /opt
ln -s /opt/mysql-5.7.30-linux-glibc2.12-x86_64 /usr/local/mysql
```

1. **初始化資料，修改配置檔案：啟用慢日誌**

```text
mkdit -p /data/3306/data
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/3306/data
cat > /etc/my.cnf <<EOF
[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/3306/data
slow_query_log=1 
slow_query_log_file=/data/3306/data/slow.log
long_query_time=0.1
log_queries_not_using_indexes
port=3306
socket=/tmp/mysql.sock
[client]
socket=/tmp/mysql.sock
EOF
```

1. **加入systemctl服務管理，啟動並開機自啟**

```text
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
systemctl enable mysqld
systemctl start mysqld
```

1. **創造mysql慢日誌並檢視**

```text
mysql -e "select sleep(2);"
tail -f /data/3306/data/slow.log
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-2"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.config.modules:
  path: \${path.config}/modules.d/*.yml
  reload.enabled: enable

filebeat.modules:
- module: mysql

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  indices:
    - index: "mysql-slow-%{[agent.version]}-%{+yyyy.MM}"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

> 注意：
>
> 1. filebeat的mysql模組匹配的是二進位制版的mysql
> 2. mariadb的慢日誌和mysql二進位制安裝的慢日誌格式不一樣

1. **啟用filebeat模組：mysql**

```text
filebeat modules enable mysql
```

1. **配置filebeat模組mysql：配置日誌路徑**

```text
cat > /etc/filebeat/modules.d/mysql.yml << 'EOF'
- module: mysql
  access:
    enabled: true
    var.paths: ["/data/3306/data/slow.log"]
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-6"></a>

1. **驗證：ES檢視到索引 mysql-slow**

## 第11章 收集tomcat的json日誌 <a id="&#x7B2C;11&#x7AE0;-&#x6536;&#x96C6;tomcat&#x7684;json&#x65E5;&#x8A8C;"></a>

### Tomcat日誌介紹 <a id="tomcat&#x65E5;&#x8A8C;&#x4ECB;&#x7D39;"></a>

| 型別 | 檔名 |
| :--- | :--- |
| 控制檯輸出的日誌 | catalina.out |
| Cataline引擎的日誌檔案 | catalina.日期.log |
| 應用初始化的日誌 | localhost.日期.log |
| 訪問tomcat的日誌 | localhost\_access\_log.日期 |
| Tomcat預設manager應用日誌 | manager.日期.log |

[參考文件](https://www.cnblogs.com/operationhome/p/9680040.html)

### Tomcat部署 <a id="tomcat&#x90E8;&#x7F72;"></a>

1. **tomcat和JDK安裝**

```text
cd /data/soft/
rpm -ivh /data/soft/jdk-*.rpm
wget https://mirror.bit.edu.cn/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz
tar xf apache-tomcat-9.0.41.tar.gz -C /opt/
ln -s /opt/apache-tomcat-9.0.41 /opt/tomcat/
```

1. **tomcat修改配置檔案：日誌JSON格式**

```text
vim /opt/tomcat/conf/server.xml
... ...
               pattern="{&quot;clientip&quot;:&quot;%h&quot;,&quot;ClientUser&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&quot;,&quot;AccessTime&quot;:&quot;%t&quot;,&quot;method&quot;:&quot;%r&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;SendBytes&quot;:&quot;%b&quot;,&quot;Query?string&quot;:&quot;%q&quot;,&quot;partner&quot;:&quot;%{Referer}i&quot;,&quot;AgentVersion&quot;:&quot;%{User-Agent}i&quot;}"/>
      </Host>
    </Engine>
  </Service>
</Server>
```

1. **tomcat啟動並檢視日誌**

```text
/opt/tomcat/bin/startup.sh
tail -f /usr/local/tomcat/logs/localhost_access_log.*.txt
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-3"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/tomcat/logs/localhost_access_log.*.txt
  json.keys_under_root: true
  json.overwrite_keys: true

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

> 支援使用萬用字元\*

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-7"></a>

1. **驗證：ES檢視到索引 tomcat**

## 第12章 收集Java多行日誌 <a id="&#x7B2C;12&#x7AE0;-&#x6536;&#x96C6;java&#x591A;&#x884C;&#x65E5;&#x8A8C;"></a>

### 介紹 <a id="&#x4ECB;&#x7D39;"></a>

java日誌特點：一個報錯，多行日誌

filebeat多行匹配模式：

* multline 是模組名，filebeat爬取的日誌滿足pattern的條件則開始多行匹配，
* negate 設定為false 匹配pattern的多行語句都需要連著上一行，
* match 合併到末尾。

[參考官方文件](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)

```text
multiline.pattern: '^\<|^[[:space:]]|^[[:space:]]+(at|\.{3})\b|^Caused by:'  #正則，自己定義，一個表示可以匹配多種模式使用or 命令也就是“|”

multiline.negate: false #預設是false，匹配pattern的行合併到上一行；true，不匹配pattern的行合併到上一行

multiline.match: after #合併到上一行的末尾或開頭

#優化引數

multiline.max_lines: 500 #最多合併500行

multiline.timeout: 5s #5s無響應則取消合併
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-4"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/elasticsearch/elasticsearch.log

  multiline.pattern: ^\[
  multiline.negate: true
  multiline.match: after

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  index: "es-%{[agent.version]}-%{+yyyy.MM}"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-8"></a>

1. **創造Java日誌：故意改錯elasticsearch配置檔案，重啟**
2. **ES檢視索引**
3. **建立新的索引模式（Index patterns），流程同上**

## 第13章 使用Redis快取 <a id="&#x7B2C;13&#x7AE0;-&#x4F7F;&#x7528;redis&#x5FEB;&#x53D6;"></a>

[參考文件](https://www.jianshu.com/p/07f2bfc45962)

### 使用Redis快取的日誌收集流程 <a id="&#x4F7F;&#x7528;redis&#x5FEB;&#x53D6;&#x7684;&#x65E5;&#x8A8C;&#x6536;&#x96C6;&#x6D41;&#x7A0B;"></a>

> 注意：
>
> Redis不支援Filebeat模組。
>
> Filebeat只支援Redis單點，不支援傳輸給Redis哨兵，叢集，主從複製。
>
> Filebeat支援Redis列表：多Redis主備，沒有負載均衡，恢復自動作為備加入。

### Nginx日誌改為json格式 <a id="nginx&#x65E5;&#x8A8C;&#x6539;&#x70BA;json&#x683C;&#x5F0F;"></a>

1. **修改配置檔案**

```text
vi /etc/nginx/nginx.conf
```

```text
    access_log  /var/log/nginx/access.log  json;
```

1. **重啟nginx，清空舊日誌，建立新日誌，檢查是否為json格式**

```text
systemctl restart nginx
> /var/log/nginx/access.log
> /var/log/nginx/error.log
for i in `seq 10`;do curl -I 127.0.0.1/1 &>/dev/null ;done
cat /var/log/nginx/access.log
cat /var/log/nginx/error.log
```

### Redis部署 <a id="redis&#x90E8;&#x7F72;"></a>

```text
yum install redis -y
systemctl start redis
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-5"></a>

1. **修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

processors:
  - drop_fields:
      fields: ["ecs","log"]

output.redis:
  hosts: ["localhost"]
  keys:
    - key: "nginx_access"
      when.contains:
        tags: "access"
    - key: "nginx_error"
      when.contains:
        tags: "error"

setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

> 優化：將兩個日誌放入redis的一個key中
>
> ```text
> cat > /etc/filebeat/filebeat.yml <<EOF
> filebeat.inputs:
> - type: log
>   enabled: true
>   paths:
>     - /var/log/nginx/access.log
>   json.keys_under_root: true
>   json.overwrite_keys: true
>   tags: ["access"]
>
> - type: log
>   enabled: true
>   paths:
>     - /var/log/nginx/error.log
>   tags: ["error"]
>
> processors:
>   - drop_fields:
>       fields: ["ecs","log"]
>
> output.redis:
>   hosts: ["localhost"]
>   key: "nginx"
>
> setup.ilm.enabled: false
> setup.template.enabled: false
>
> # logout
> logging.level: info
> logging.to_files: true
> logging.files:
>   path: /var/log/filebeat
>   name: filebeat
>   keepfiles: 7
>   permissions: 0644
> EOF
> ```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

1. **檢視redis驗證**

```text
[root@node-1 ~]# redis-cli 
127.0.0.1:6379> keys *
1) "nginx_error"
2) "nginx_access"
127.0.0.1:6379> type nginx_access
list
127.0.0.1:6379> LLEN nginx_access
(integer) 10
127.0.0.1:6379> LRANGE nginx_access 0 0
1) "{\"@timestamp\":\"2020-12-30T01:21:57.664Z\",\"@metadata\":{\"beat\":\"filebeat\",\"type\":\"_doc\",\"version\":\"7.9.1\"},\"bytes\":0,\"upstream_time\":\"-\",\"request_time\":\"0.000\",\"referer\":\"-\",\"up_host\":\"-\",\"time_local\":\"30/Dec/2020:09:19:08 +0800\",\"tags\":[\"access\"],\"input\":{\"type\":\"log\"},\"up_addr\":\"-\",\"status\":404,\"remote_addr\":\"127.0.0.1\",\"http_user_agent\":\"curl/7.29.0\",\"x_forwarded\":\"-\",\"host\":{\"name\":\"node-1\"},\"agent\":{\"hostname\":\"node-1\",\"ephemeral_id\":\"ee22f8d8-db49-4a93-a2a2-76e971707a39\",\"id\":\"999917a2-5dc1-4495-80f0-bd700385e0ec\",\"name\":\"node-1\",\"type\":\"filebeat\",\"version\":\"7.9.1\"},\"request\":\"HEAD /1 HTTP/1.1\"}"
127.0.0.1:6379> LRANGE nginx_access 0 -1
```

### Logstash部署 <a id="logstash&#x90E8;&#x7F72;"></a>

1. **Logstash安裝，依賴JDK**

```text
cd /data/soft/
rpm -ivh logstash-7.9.1.rpm
```

1. **建立子配置檔案**

```text
cat <<EOF >/etc/logstash/conf.d/redis.conf
input {
  redis {
    host => "localhost"
    port => "6379"
    db => "0"
    key => "nginx_access"
    data_type => "list"
  }
  redis {
    host => "localhost"
    port => "6379"
    db => "0"
    key => "nginx_error"
    data_type => "list"
  }
}

output {
   stdout {}
   if "access" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM}"
      }
    }
}
EOF
```

> 優化：redis根據tags輸出日誌
>
> ```text
> cat <<EOF >/etc/logstash/conf.d/redis.conf
> input {
> redis {
>  host => "localhost"
>  port => "6379"
>  db => "0"
>  key => "nginx"
>  data_type => "list"
> }
> }
>
> filter {
> mutate {
>  convert => ["upstream_time", "float"]
>  convert => ["request_time", "float"]
> }
> }
>
> output {
> stdout {}
> if "access" in [tags] {
>    elasticsearch {
>      hosts => "http://10.0.0.51:9200"
>      manage_template => false
>      index => "nginx_access-%{+yyyy.MM}"
>    }
>  }
>  if "error" in [tags] {
>    elasticsearch {
>      hosts => "http://10.0.0.51:9200"
>      manage_template => false
>      index => "nginx_error-%{+yyyy.MM}"
>    }
>  }
> }
> EOF
> ```

1. **前臺啟動測試**

```text
# 很慢，就像夯住一樣
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/redis.conf
```

1. **驗證redis資料流程：filebeat將日誌存到redis（增加），logstash從redis取走日誌（減少）**

```text
> /var/log/nginx/access.log
> /var/log/nginx/error.log
for i in `seq 10000`;do curl -I 127.0.0.1/1 &>/dev/null ;done
```

```text
redis-cli LLEN nginx_access
```

### 多Redis主備配置 <a id="&#x591A;redis&#x4E3B;&#x5099;&#x914D;&#x7F6E;"></a>

1. **Filebeat修改配置檔案**

```text
cat > /etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

processors:
  - drop_fields:
      fields: ["ecs","log"]

output.redis:
  hosts: ["localhost","10.0.0.52"]
  key: "nginx"
 
setup.ilm.enabled: false
setup.template.enabled: false

# logout
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
EOF
```

1. **Logstash修改配置檔案**

```text
cat <<EOF >/etc/logstash/conf.d/redis.conf
input {
  redis {
    host => "localhost"
    port => "6379"
    db => "0"
    key => "nginx"
    data_type => "list"
  }
  
  redis {
    host => "10.0.0.52"
    port => "6379"
    db => "0"
    key => "nginx"
    data_type => "list"
  }
}

filter {
  mutate {
    convert => ["upstream_time", "float"]
    convert => ["request_time", "float"]
  }
}

output {
   stdout {}
   if "access" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM}"
      }
    }
}
EOF
```

## 第14章 使用kafka快取 <a id="&#x7B2C;14&#x7AE0;-&#x4F7F;&#x7528;kafka&#x5FEB;&#x53D6;"></a>

[參考文件](https://www.jianshu.com/p/07f2bfc45962)

### 使用kafka為快取的日誌收集流程 <a id="&#x4F7F;&#x7528;kafka&#x70BA;&#x5FEB;&#x53D6;&#x7684;&#x65E5;&#x8A8C;&#x6536;&#x96C6;&#x6D41;&#x7A0B;"></a>

### 所有節點配置hosts和金鑰 <a id="&#x6240;&#x6709;&#x7BC0;&#x9EDE;&#x914D;&#x7F6E;hosts&#x548C;&#x91D1;&#x9470;"></a>

```text
cat <<EOF >>/etc/hosts
10.0.0.51 node-51
10.0.0.52 node-52
10.0.0.53 node-53
EOF
ssh-keygen
ssh-copy-id 10.0.0.51
ssh-copy-id 10.0.0.52
ssh-copy-id 10.0.0.53
```

### zookeeper叢集部署 <a id="zookeeper&#x53E2;&#x96C6;&#x90E8;&#x7F72;"></a>

1. **node-1安裝zookeeper並推送給其他節點**

```text
cd /data/soft
tar zxf zookeeper-3.4.11.tar.gz -C /opt/
ln -s /opt/zookeeper-3.4.11/ /opt/zookeeper
mkdir -p /data/zookeeper
cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
cat >/opt/zookeeper/conf/zoo.cfg<<EOF
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
server.1=10.0.0.51:2888:3888
server.2=10.0.0.52:2888:3888
server.3=10.0.0.53:2888:3888
EOF
echo "1" > /data/zookeeper/myid
cat /data/zookeeper/myid
rsync -avz /opt/zookeeper* 10.0.0.52:/opt/
rsync -avz /opt/zookeeper* 10.0.0.53:/opt/
rsync -avz jdk-*.rpm 10.0.0.52:/data/soft/
rsync -avz jdk-*.rpm 10.0.0.53:/data/soft/
```

1. **node-2安裝JDK修改myid**

```text
rpm -ivh /data/soft/jdk-*.rpm
mkdir -p /data/zookeeper
echo "2" > /data/zookeeper/myid
cat /data/zookeeper/myid
```

1. **node-3安裝JDK修改myid**

```text
rpm -ivh /data/soft/jdk-*.rpm
mkdir -p /data/zookeeper
echo "3" > /data/zookeeper/myid
cat /data/zookeeper/myid
```

1. **所有節點啟動zookeeper，並檢查狀態（1個leader、2個follower）**

```text
/opt/zookeeper/bin/zkServer.sh start
/opt/zookeeper/bin/zkServer.sh status
```

1. **測試zookeeper**

```text
# 一個節點上建立一個頻道
/opt/zookeeper/bin/zkCli.sh -server 10.0.0.51:2181 create /test "hello"

# 其他節點上看能否接收到
/opt/zookeeper/bin/zkCli.sh -server 10.0.0.52:2181 get /test
```

### kafka叢集部署 <a id="kafka&#x53E2;&#x96C6;&#x90E8;&#x7F72;"></a>

1. **node-1安裝kafka並推送給其他節點**

```text
cd /data/soft/
tar zxf kafka_2.11-1.0.0.tgz -C /opt/
ln -s /opt/kafka_2.11-1.0.0/ /opt/kafka
mkdir /opt/kafka/logs
cat >/opt/kafka/config/server.properties<<EOF
broker.id=1
listeners=PLAINTEXT://10.0.0.51:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/opt/kafka/logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=24
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
EOF
rsync -avz /opt/kafka* 10.0.0.52:/opt/
rsync -avz /opt/kafka* 10.0.0.53:/opt/
```

1. **node-2修改id及IP**

```text
sed -i "s#10.0.0.51:9092#10.0.0.52:9092#g" /opt/kafka/config/server.properties
sed -i "s#broker.id=1#broker.id=2#g" /opt/kafka/config/server.properties
```

1. **node-3修改id及IP**

```text
sed -i "s#10.0.0.51:9092#10.0.0.53:9092#g" /opt/kafka/config/server.properties
sed -i "s#broker.id=1#broker.id=3#g" /opt/kafka/config/server.properties
```

1. **所有節點前臺啟動並檢查程序**

```text
/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
jps
```

1. **測試kafka**

```text
# 建立topic
/opt/kafka/bin/kafka-topics.sh --create  --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181 --partitions 3 --replication-factor 3 --topic kafkatest
```

```text
# 獲取toppid
/opt/kafka/bin/kafka-topics.sh --describe --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181 --topic kafkatest
```

```text
# 刪除topic
/opt/kafka/bin/kafka-topics.sh --delete --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181 --topic kafkatest
```

1. **測試kafka通訊**

```text
# 1.建立topic
/opt/kafka/bin/kafka-topics.sh --create --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181 --partitions 3 --replication-factor 3 --topic messagetest

# 2.傳送訊息
/opt/kafka/bin/kafka-console-producer.sh --broker-list  10.0.0.51:9092,10.0.0.52:9092,10.0.0.53:9092 --topic messagetest

# 3.其他節點測試接收
/opt/kafka/bin/kafka-console-consumer.sh --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181 --topic messagetest --from-beginning

# 4.測試獲取所有的頻道
/opt/kafka/bin/kafka-topics.sh  --list --zookeeper 10.0.0.51:2181,10.0.0.52:2181,10.0.0.53:2181
```

1. **測試成功之後，後臺啟動**

```text
/opt/kafka/bin/kafka-server-start.sh  -daemon /opt/kafka/config/server.properties
```

### Filebeat配置 <a id="filebeat&#x914D;&#x7F6E;-6"></a>

1. **修改配置檔案**

```text
cat >/etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: true 
  paths:
    - /var/log/nginx/access.log
  tags: ["access"]

- type: log
  enabled: true 
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

output.kafka:
  hosts: ["10.0.0.51:9092", "10.0.0.52:9092", "10.0.0.53:9092"]
  topic: 'filebeat'

setup.ilm.enabled: false
setup.template.enabled: false
EOF
```

1. **重啟filebeat生效**

```text
systemctl restart filebeat
```

### Logstash配置 <a id="logstash&#x914D;&#x7F6E;"></a>

1. **修改配置檔案**

```text
cat >/etc/logstash/conf.d/kafka.conf <<EOF
input {
  kafka{
    bootstrap_servers=>["10.0.0.51:9092,10.0.0.52:9092,10.0.0.53:9092"]
    topics=>["filebeat"]
    #group_id=>"logstash"
    codec => "json"
  }
}

output {
   stdout {}
   if "access" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://10.0.0.51:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM}"
      }
    }
}
EOF
```

1. **前臺啟動logstash測試**

```text
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/kafka.conf
```

### 檢查收集結果 <a id="&#x6AA2;&#x67E5;&#x6536;&#x96C6;&#x7D50;&#x679C;-9"></a>

1. **清除ES索引**
2. **清除日誌，建立日誌**

```text
> /var/log/nginx/access.log
> /var/log/nginx/error.log
for i in `seq 100`;do curl -I 127.0.0.1/1 &>/dev/null ;done
```

1. **索引nginx\_access-2020.12和nginx\_error-2020.12自動建立**
2. **關閉node-2節點（follower），建立日誌，ES索引size增加，仍然可以收集日誌**

