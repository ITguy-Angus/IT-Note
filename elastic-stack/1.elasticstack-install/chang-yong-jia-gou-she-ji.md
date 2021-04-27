# 常用架構設計

### 1、最簡單架構

在這種架構中，只有一個 Logstash、Elasticsearch 和 Kibana 實例。Logstash 通過輸入插件從多種數據源（比如日誌文件、標準輸入 Stdin 等）獲取數據，再經過濾插件加工數據，然後經 Elasticsearch 輸出插件輸出到 Elasticsearch，通過 Kibana 展示。![](https://i1.kknews.cc/SIG=edn20r/ctp-vzntr/o34s2s82p48546p5n68r1nq44r6581po.jpg)

![](../../.gitbook/assets/tu-pian-%20%285%29.png)

這種架構非常簡單，使用場景也有限。初學者可以搭建這個架構，了解 ELK 如何工作

### 2、以Logstash 作為日誌搜集器

![](../../.gitbook/assets/tu-pian-%20%289%29.png)

這種結構因為需要在各個伺服器上部署 Logstash，而它比較消耗 CPU 和內存資源，所以比較適合計算資源豐富的伺服器，否則容易造成伺服器性能下降，甚至可能導致無法正常工作。

### 3、以Beats 作為日誌搜集器

這種架構引入 Beats 作為日誌搜集器。目前 Beats 包括四種：

* Packetbeat（搜集網絡流量數據）；
* Topbeat（搜集系統、進程和文件系統級別的 CPU 和內存使用情況等數據）；
* Filebeat（搜集文件數據）；
* Winlogbeat（搜集 Windows 事件日誌數據）。

Beats 將搜集到的數據發送到 Logstash，經 Logstash 解析、過濾後，將其發送到 Elasticsearch 存儲，並由 Kibana 呈現給用戶。![](https://i1.kknews.cc/SIG=jl3m9n/ctp-vzntr/805n3o48s01p4ssr851722o63348685q.jpg)

![](../../.gitbook/assets/tu-pian-%20%286%29.png)

這種架構解決了 Logstash 在各伺服器節點上占用系統資源高的問題。相比 Logstash，Beats 所占系統的 CPU 和內存幾乎可以忽略不計。另外，Beats 和 Logstash 之間支持 SSL/TLS 加密傳輸，客戶端和伺服器雙向認證，保證了通信安全。

因此這種架構適合對數據安全性要求較高，同時各伺服器性能比較敏感的場景。

### 4、**引入消息隊列模式**

**Beats 還不支持輸出到消息隊列**（**新版本除外：5.0版本及以上**），所以在消息隊列前後兩端只能是 Logstash 實例。logstash從各個數據源搜集數據，**不經過任何處理轉換**僅轉發出到消息隊列（kafka、redis、rabbitMQ等），**後logstash從消息隊列取數據進行轉換分析過濾**，輸出到elasticsearch，並在kibana進行圖形化展示![](https://i2.kknews.cc/SIG=18r9j41/ctp-vzntr/7s843228381646qn93011ps776nn40r9.jpg)

![](../../.gitbook/assets/tu-pian-%20%288%29.png)

模式特點：這種架構適合於日誌規模比較龐大的情況。但由於 Logstash 日誌解析節點和 Elasticsearch 的負荷比較重，可將他們配置為集群模式，以分擔負荷。引入消息隊列，均衡了網絡傳輸，從而降低了網絡閉塞，尤其是丟失數據的可能性，但依然存在 Logstash 占用系統資源過多的問題

工作流程：Filebeat採集—&gt; logstash轉發到kafka—&gt; logstash處理從kafka緩存的數據進行分析—&gt; 輸出到es—&gt; 顯示在kibana  
  
原文網址：[https://kknews.cc/code/8kl84gg.html](https://kknews.cc/code/8kl84gg.html)

