# ElasticSearch 叢集的規劃部署與運維

#### 1，常見的叢集部署方式 <a id="1&#xFF0C;&#x5E38;&#x898B;&#x7684;&#x53E2;&#x96C6;&#x90E8;&#x7F72;&#x65B9;&#x5F0F;"></a>

ES 有以下不同型別的節點：

* **Master**\(eligible\)節點：只有 Master eligible 節點可以成為 Master 節點。
  * Master 節點用於維護索引資訊和叢集狀態。
* **Data** 節點：負責資料儲存。
* **Ingest** 節點：資料預處理。
* **Coordinating** 節點：處理使用者請求。
* **ML** 節點：機器學習相關功能。

在開發環境中，一個節點可以承擔多種角色。

但是在生產環境，建議一個節點只負責單一角色，以達到高可用性及高效能。同時根據業務需求和硬體資源來合理分配節點。

**1.1，節點配置引數**

在預設情況下，一個節點會同時扮演 Master eligible Node，Data Node 和 Ingest Node。

各型別的節點配置引數如下：

| 節點型別 | 配置引數 | 預設值 |
| :--- | :--- | :--- |
| Master eligible | node.master | true |
| Data Node | node.data | true |
| Ingest Node | node.ingest | true |
| Coordinating | 無 | - |
| ML | node.ml | true（需要 enable x-pack） |

預設情況下，每個節點都是一個 Coordinating 節點，可以將 `node.master`，`node.data` 和 `node.ingest` 同時設定為 `false`，讓一個節點**只負責** Coordinating 節點的角色。

**1.2，配置單一角色**

預設情況下，一個節點會承擔多個角色，可以通過配置讓一個節點只負責單一角色。

單一職責節點配置：

* **Master** 節點：從高可用和避免腦裂的角度考慮，生產環境可配置 3 個 Master節點。
  * node.master：`true`
  * node.ingest：`false`
  * node.data：`false`
* **Data** 節點
  * node.master：`false`
  * node.ingest：`false`
  * node.data：`true`
* **Ingest** 節點
  * node.master：`false`
  * node.ingest：`true`
  * node.data：`false`
* **Coordinating** 節點
  * node.master：`false`
  * node.ingest：`false`
  * node.data：`false`

**1.3，水平擴充套件架構**

叢集的水平擴充套件：

* 當需要更多的磁碟容量和讀寫能力時，可以增加 Data Node；
* 當系統有大量的複雜查詢和聚合分析時，可以增加 Coordinating Node。

![&#x5728;&#x9019;&#x88E1;&#x63D2;&#x5165;&#x5716;&#x7247;&#x63CF;&#x8FF0;](https://i.iter01.com/images/1e6d56655003803149ab25befdf7fdbdb7da89817078723a3c52372727b20483.png)

**1.4，讀寫分離架構**

使用 **Ingest** 節點對資料預處理。

![&#x5728;&#x9019;&#x88E1;&#x63D2;&#x5165;&#x5716;&#x7247;&#x63CF;&#x8FF0;](https://i.iter01.com/images/c69aba0876d3d2030913128979d46298ce58893fb31b58b168c36d737d118c78.png)

#### 2，分片設計與管理 <a id="2&#xFF0C;&#x5206;&#x7247;&#x8A2D;&#x8A08;&#x8207;&#x7BA1;&#x7406;"></a>

ES 中的文件儲存在索引中，索引的最小儲存單位是分片，不同的索引儲存在不同的分片中。

> 當討論分片時，一般是**基於某個索引**的，不同索引之間的分片互不干擾。

分片分為**主分片**和**副本分片**兩種；副本分片是主分片的拷貝，主要用於備份資料。

關於[主副分片數的設定](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-modules.html)：

* 主分片數：主分片數在索引建立時確定，之後不能修改。
  * 在 ES 7.0 以後，一個索引**預設**有一個主分片。
  * 一個索引的主分片數不能超過 **1024**。
* 副本分片數：副本分片數在索引建立之後可以動態修改。
  * 副本分片數預設為 1。

關於每個節點上的分片數的設定，可參考[這裡](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/allocation-total-shards.html)。

**2.1，主分片的設計**

如果某個索引只有**一個主分片**：

* 優點：查詢算分和聚合不精準的問題都可避免。
* 缺點：叢集無法實現水平擴充套件。
  * 因為索引（不管該索引的資料量達到了多大）只能儲存在一個主分片上（一個分片不能跨節點儲存/處理）；
  * 對於單個主分片的索引來說，即使有再多的資料節點，它也無法利用。

如果某個索引有**多個主分片**：

* 優點：叢集可以實現水平擴充套件。
  * 對於擁有多個主分片的索引，該索引的資料可以分佈在多個主分片上，不同的主分片可以分佈在不同的資料節點中；這樣，該索引就可以利用多個節點的讀寫能力，從而處理更多的資料。
  * 如果當前的**資料節點數**小於**主分片數**，當有新的資料節點加入叢集后，這些主分片就會自動被分配到新的資料節點上，從而實現水平擴容。
* 缺點：但是主分片數也不能過多，因為對於分片的管理也需要額外的資源開銷。主要會帶來以下問題：
  * 每次搜尋/聚合資料時需要從多個分片上獲取資料，並彙總；除了會帶來精準度問題，還會有效能問題。
  * 分片的 Meta 資訊由 Master 節點維護管理，過多的分片，會增加 Master 節點的負擔。

對於**分片的設計建議**：

* 從分片的儲存量考慮：
  * 對於日誌類應用，單個分片不要大於 50G；
  * 對於搜尋類應用，單個分片不要大於 20G；
* 從分片數量考慮：
  * **一個 ES 叢集的分片**（包括主分片和副本分片）**總數不超過 10 W**。

**2.2，副本分片的設計**

副本分片是主分片的備份：

* 優點：
  * 可防止資料丟失，提高系統的可用性；
  * 可以分擔主分片的查詢壓力，提高系統的**查詢**效能。
* 缺點：
  * 與主分片一樣，需要佔用系統資源，有多少個副本，就會增加多少倍的儲存消耗。
  * 會降低系統的寫入速度。

#### 3，叢集容量規劃 <a id="3&#xFF0C;&#x53E2;&#x96C6;&#x5BB9;&#x91CF;&#x898F;&#x5283;"></a>

容量規劃指的是，在一個實際專案中：

* 一個叢集需要多少節點，以及節點型別分配。
* 一個索引需要幾個主分片，幾個副本分片。

**3.1，要考慮的因素**

做容量規劃時要考慮的因素：

* 機器的軟硬體配置
* 資料量：
  * 單條文件的尺寸
  * 文件的總數量
  * 索引的總數量
* 業務需求：
  * 文件的複雜度、資料格式
  * 寫入需求
  * 查詢需求
  * 聚合需求
  * 等

**3.2，硬體配置**

對系統整體效能要求高的，建議使用 SSD，記憶體與硬碟的比例可為 1：10。

對系統整體效能要求一般的，可使用機械硬碟，記憶體與硬碟的比例可為 1：50。

JVM 配置為機器記憶體的一半，建議 JVM 記憶體配置不超過 32 G。

單個節點的資料建議控制在 2TB 以內，最大不超過 5 TB。

**3.3，常見應用場景**

有如下常見應用場景：

* 搜尋類應用：
  * 總體資料集大小基本固定，資料量增長較慢。
* 日誌類應用：
  * 每日新增資料量比較穩定，資料量持續增長，可預期。

**1，處理時間序列資料**

ES 中提供了 [Date Math 索引名](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html)用於寫入時間序列的資料。

示例：

![&#x5728;&#x9019;&#x88E1;&#x63D2;&#x5165;&#x5716;&#x7247;&#x63CF;&#x8FF0;](https://i.iter01.com/images/17b4adadaa4331434cc970375db9b52161e95935166f80b61ee9eb588f109373.png)

請求 URI 要經過 URL 編碼：

```text
# PUT /<my-index-{now/d}>
# 經過 URL 編碼後
PUT /%3Cmy-index-%7Bnow%2Fd%7D%3E
```

查詢示例：

```text
# POST /<logs-{now/d}/_search
POST /%3Clogs-%7Bnow%2Fd%7D%3E/_search

# POST /<logs-{now/w}/_search
POST /%3Clogs-%7Bnow%2Fw%7D%3E/_search
```

#### 4，ES 開發模式與生產模式 <a id="4&#xFF0C;es-&#x958B;&#x767C;&#x6A21;&#x5F0F;&#x8207;&#x751F;&#x7522;&#x6A21;&#x5F0F;"></a>

從 ES 5 開始，ES 支援開發模式與生產模式，ES 可通過配置自動選擇不同的模式去執行：

* 開發模式配置：
  * http.host：localhost
  * transport.bind\_host：localhost
* 生產模式配置：
  * http.host：真實 IP 地址
  * transport.bind\_host：真實 IP 地址

**4.1，Booststrap 檢測**

在生產模式啟動 ES 叢集時，會進行 [Booststrap 檢測](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html)（只有檢測通過才能啟動成功），它包括：

* JVM 檢測
* Linux 檢測：只在 Linux 環境進行

**4.2，JVM 配置**

JVM 通過 `config` 目錄下的 [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/jvm-options.html) 檔案進行配置，需要注意以下幾點：

* 將 Xms 和 Xmx 設定成一樣；
* Xmx 不要超過實體記憶體的 50%，最大記憶體建議不超過 32G；
* JVM 有 Server 和 Client 兩種模式，在 ES 的生產模式必須使用 Server 模式；
* 需要關閉 JVM Swapping

**4.3，更多的 ES 配置**

更多的關於 **ES 的配置**可參考其官方文件，包括：

* [Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)
* [Important Elasticsearch configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)
* [Important System Configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html)

#### 5，監控叢集狀態 <a id="5&#xFF0C;&#x76E3;&#x63A7;&#x53E2;&#x96C6;&#x72C0;&#x614B;"></a>

叢集狀態為 **Green** 只能代表分片正常分配，不能代表沒有其它問題。

ES 提供了很多監控相關的 API：

* [\_cluster/health](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html)：叢集健康狀態。
* [\_cluster/state](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html)：叢集狀態。
* [\_cluster/stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html)：叢集指標統計。
* [\_cluster/pending\_tasks](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-pending.html)：叢集中正在執行的任務。
* [\_tasks](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)：叢集任務。
* [\_cluster/allocation/explain](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-allocation-explain.html)：檢視叢集分片的分配情況，用於查詢原因。
* [\_nodes/stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html)：節點指標統計。
* [\_nodes/info](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html)：節點資訊。
* [\_index/stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html)：索引指標統計。
* 一些 [cat](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html) API。

**5.1，Slow log**

ES 的 [Slow log](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-slowlog.html) 可以設定一些閾值，當寫入時間或者查詢時間超過這些閾值後，會將相關操作記錄日誌。

**5.2，叢集診斷**

需要監控的指標：

![&#x5728;&#x9019;&#x88E1;&#x63D2;&#x5165;&#x5716;&#x7247;&#x63CF;&#x8FF0;](https://i.iter01.com/images/12d1686a75712d24602bbd5b55a4807f25344a176b10b0b8176ffe296947c9fa.png)

一個叢集診斷工具 [Support Diagnostics](https://github.com/elastic/support-diagnostics)。

（本節完。）

