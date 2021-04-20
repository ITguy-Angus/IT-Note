---
description: ElasticStack 介紹與安裝
---

# 1.Elastic Stack 介紹與安裝

## ELK Stack 簡介

     ELK是由Elasitc 公司開發並再2008年掛牌上市，ELK套件組成最初是由ElasticSearch、Logstash、Kibana 三個套件組成，後來又加入了Beat套件並改名為ELK Stack是一種數據處裡的框架，最先發名的組件為ElasticSearch是一個在2004為了收索食譜而發明收尋引擎，單當時他還叫做Compass就是了但在2010年就基於APACH Lucene 成為了開源軟體，後因ElasticSearch收到廣大歡迎催生了以下三個套件Logstash、Kibana、beat，並又針對商業應用開發了X-Pack擴展套件，下面就為各位列出各個套件分別不同的作用吧。

### 各組件應用

* Elasitcsearch  : 用於數據儲存與檢索，天然支持數據分片與複製，可輕鬆實現擴容，很多情況下被當成NoSql數據庫獨立使用。
* Logstash : 用於數據清洗、數據傳輸，可以適配多種輸入和輸出數據源，通時提供了豐富的過濾器插件。
* Kibana : 用於數據可視化和數據分析，可看作是ElasticSearch的介面。
* Beat 分為各個不同的組件為各系統所應用
  1. Filebeat: Beats 組件中的一種，是一種安裝於宿主機的輕量級組件，核心作用就是將宿主機上指定的文件內容提取出來並發送到指定的目的地。





