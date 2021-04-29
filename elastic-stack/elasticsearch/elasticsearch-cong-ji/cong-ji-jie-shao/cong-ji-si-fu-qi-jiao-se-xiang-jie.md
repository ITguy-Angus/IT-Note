# 叢集伺服器腳色詳解

### 角色規劃

一個節點在預設情況會下同時扮演:master eligible，data node 和 ingest node。

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210224155022.png)

在生產環境中建議每個節點只承擔一個角色：

* Dedicated **master** eligible nodes:負責分片管理，索引建立，叢集管理等操作，使用低配置的 CPU，RAM 和磁碟。
* Dedicated **data** nodes:負責資料儲存及處理客戶端請求，使用高配置的 CPU, RAM 和磁碟。
* Dedicated **ingest** nodes:負責資料處理，使用高配置 CPU，中等配置的RAM，低配置的磁碟。
* Dedicate **coordinating** only node \(client node\)：扮演Load Balancers，降低master和data nodes的負載，負責搜尋結果的Gather/Reduce。使用高/中等配置 CPU，高/中等配置的RAM，低配置的磁碟。

  ![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210224155748.png)

\#\#節點間資料加密

!\[image-20210302170617563\]\(/Users/chengzhiwei/Library/Application Support/typora-user-images/image-20210302170617563.png\)

* 加密資料 ，避免資料抓包，敏感資訊洩漏。
* 驗證身份 ，避免 Impostor Node。

```text
#生成CA
bin/elasticsearch-certutil ca
#簽名證書
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

#elasticsearch.yml配置
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```

### 主節點選舉

從高可用 & 避免腦裂的角度出發，一般在生產環境中配置3臺master節點。

在新版7.0的es中，對es的叢集發現系統做了調整，不再有discovery.zen.minimum\_master\_nodes這個控制叢集腦裂的配置，轉而由叢集自主控制，並且新版在啟動一個新的叢集的時候需要有cluster.initial\_master\_nodes初始化叢集列表。在叢集初始化第一次完成選舉後，應當刪除cluster.initial\_master\_nodes配置。

```text
discovery.seed_hosts: ["master1","master2","master3"] #master-eligible節點
cluster.initial_master_nodes: ["master1"]  #初始化選舉的master節點
```

### 架構設計

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210224160329.png)

* 當系統中有大量的複雜查詢及聚合時候，增加 Coordinating 節點，增加查詢的效能。
* 當磁碟容量無法滿足需求時或者磁碟讀寫壓力大時，可以增加資料節點。
* 讀寫分離：配置LB負載均衡寫請求到Ingest節點，讀請求到Coordinating節點。

* Kibana部署在每臺Coordinating上，同樣使用LB做流量分發。

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210224160617.png)

### Hot & Warm 架構

Hot 節點\(通常使用 SSD\):索引有不斷有新文件寫入，通常使用 SSD。

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210225112140.png)

Warm 節點\(通常使用 HDD\):索引不存在新資料的寫入，同時也不存在大量的資料查詢。

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210225112151.png)

實現原理，藉助Elasticsearch的分片分配策略：

* 第一：叢集節點層面支援規劃節點型別，這是劃分熱暖節點的前提。
* 第二：索引層面支援將資料路由到給定節點，這為資料寫入冷、熱節點做了保障。

在 Elastic Stack 6.6 版本後推出了新功能 Index Lifecycle Management\(索引生命週期管理\)，支援針對索引的全生命週期託管管理，並且在 Kibana 上也提供了一套UI介面來配置策略。

例如這裡配置了一條IML策略：

* 當docs數超過5000時進行rollover操作。
* 20天后進入warm階段，索引只讀。
* 40天后進入cold階段，副本分片縮減為0。
* 60天后進入delete階段，刪除索引。

```text
PUT /_ilm/policy/log_ilm_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 5000
          }
        }
      },
      "warm": {
        "min_age": "20d",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "warm"
            }
          },
          "readonly": {}
        }
      },
      "cold": {
        "min_age": "40d",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "cold"
            },
            "number_of_replicas": 0
          }
        }
      },
      "delete": {
        "min_age": "60d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### Rack Awareness

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210225113046.png)

Elasticsearch的節點可能分佈在不同的機架，當一個機架斷電時，可能會丟失幾個節點。如果一個索引相同的主分片和副本分片同時在這個機架上，就有可能導致資料的丟失。通過Rack Awareness的機制，就可以儘可能避免將同一個索引的主副分片分配在一個機架的節點上。

```text
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "rack",
    "cluster.routing.allocation.awareness.force.rack.values": "rack1,rack2"
  }
}
```

### 系統配置

#### 檔案描述符

Elasticsearch使用大量的檔案描述符（檔案控制代碼）。用完檔案描述符可能是災難性的，並極有可能導致資料丟失。確保將執行Elasticsearch的使用者開啟檔案描述符的數量限制增加到65,536或更高。

編輯`/etc/security/limits.conf`檔案，設定開啟的檔案控制代碼數：

```text
elasticsearch  -  nofile  65535
```

通過該命令檢視Elasticsearch每個節點所能使用的最大檔案描述符數量：

```text
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```

#### 禁用swap

swap對節點的效能和穩定性非常不利，swap可能導致GC持續幾分鐘而不是幾毫秒，還可能導致節點響應緩慢，甚至斷開與叢集的連線。

```text
sudo swapoff -a  #並將/etc/fstab 檔案中包含swap的行註釋掉。
```

也可以通過在 elasticsearch.yml 中設定 `bootstrap.memory_lock: true`，以保持 JVM 鎖定記憶體，保證 Elasticsearch 的效能。

#### 虛擬記憶體

Easticsearch 對各種檔案混合使用了 NioFs（ 非阻塞檔案系統）和 MMapFs （ 記憶體對映檔案系統）。請確保配置的最大對映數量，以便有足夠的虛擬記憶體可用於 mmapped 檔案。

編輯`/etc/sysctl.conf`檔案：

```text
vm.max_map_count=262144
```

#### 執行緒數

Elasticsearch對不同型別的操作使用許多執行緒池，能夠在需要時建立新執行緒很重要。確保Elasticsearch使用者可以建立的執行緒數至少為4096。

```text
elasticsearch - nproc 4096
```

#### JVM

* 將記憶體 Xms 和 Xmx 設定成一樣，避免 heap resize 時引發停頓。
* Xmx 設定不要超過實體記憶體的 50%，單個節點上，最大記憶體建議不要超過 32 G 記憶體。

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210225145707.png)

* Lucene 中的 倒排索引 segments 儲存在檔案中，為提高訪問速度，都會把它載入到記憶體中，從而提高 Lucene 效能。所以建議至少留系統一半記憶體給Lucene。
* **Node Query Cache**：負責快取filter 查詢結果\)，每個節點有一個，被所有 shard 共享，filter query查詢結果不涉及 scores 的計算。
* **Indexing Buffe**r：索引緩衝區，用於儲存新索引的文件，當其被填滿時，緩衝區中的文件被寫入磁碟中的 segments 中。
* **Shard Request Cache**： 用於快取請求結果。
* **Field Data Cache**：Elasticsearch 載入記憶體 fielddata 的預設行為是延遲載入 。在首次對text型別欄位做聚合、排序或者在指令碼中使用時，需要設定欄位為fielddata資料結構，它將會完整載入這個欄位所有 Segment 中的倒排索引到堆記憶體中。不推薦使用，因為fielddata會佔用大量堆記憶體空間 ，聚合或者排序使用doc\_value。

### 硬體配置

搜尋等效能要求高的場景，建議 SSD，按照 1 :10 的比例配置記憶體和硬碟。

日誌類和查詢併發低的場景，可以考慮使用機械硬碟儲存，按照 1:50 的比例配置記憶體和硬碟。

單節點資料建議控制在 2 TB 以內，最大不建議超過 5 TB。

大多數的ES叢集對於CPU要求都不高，一般2C~8C的配置都可以，具體CPU數根據業務場景來。

ES JVM heap 最大建議不要超過 32 G 記憶體 ，30G heap 大概能處理的資料量 10T，其餘都給OS cache，標準的建議是分50%的機器記憶體給JVM，剩餘給OS cache，或可適當將OS cache比例上調。

### 資源預估

* 通過資料量預估磁碟容量

騰訊雲在2019年4月的 meetup 分享中建議：磁碟容量大小 = 原始資料大小 \* 3.38。假設一條文件資料1K，預計資料量有10億條，那麼就有100G的原始資料大小，磁碟佔用上可能就會有300多G的內容（儲存倒排索引，行存和列存等）。

* 通過搜尋吞吐率預估系統配置

Lucene引擎非常依賴底層的File System Cache，搜尋如果需要達到ms級響應，那就需要有足夠的記憶體去快取大部分的索引資料。 比如10億級文件數，300G資料總量，假設用到5臺數據伺服器，8C64G，記憶體總量大小為300G，其中150G分給JVM堆記憶體，剩餘150G分給OS cache，作業系統用掉了50G大小，那麼剩餘100G的OS cache可以用於儲存索引資料，大約會有30%的概率是基於OS cache做磁碟索引檔案的讀寫，平均響應差不多會在亞秒級~秒級。

### 分片與副本

#### 分片大小

為什麼要控制分片儲存大小？

* 提高 Update 的效能。
* Merge 時，減少所需的資源。
* 丟失節點後，具備更快的恢復速度 / 便於分片在叢集內 Rebalancing。

日誌類應用，單個分片不要大於 50 GB；搜尋類應用，單個分片不要超過20 GB。

#### 分片數量

為什麼要控制分片的數量？

* 每個分片是一個 Lucene 的 索引，會使用機器的資源。過多的分片會導致額外的效能開銷。
* 分片的 Meta 資訊由 Master 節點維護。過多，會增加管理的負擔。

一個很好的經驗法則是：確保每個節點的分片數量保持在低於**每1GB堆記憶體對應叢集的分片在20-25之間**。分片總數在控制在10W以內。

合理設定主分片數，確保均勻分配在所有資料節點上。 限定每個索引在每個節點上可分配的主分片數：

```text
PUT my_index
{
  "settings": {
    "routing.allocation.total_shards_per_node": 2
  }
}
#5 個節點的叢集。 索引有 5 個主分片，1 個副本，應該如何設定?
#(5+5)/5=2
```

### 索引設計

Mapping 是定義文件和欄位的欄位的儲存和索引方式的過程\(類似於Mysql裡的表結構設計\)，一般會從如下角度去設計與優化：

![](https://chengzw258.oss-cn-beijing.aliyuncs.com/Elasticsearch/20210301163424.png)

Mapping 欄位的相關設定:

* enabled – 設定成 false，僅做儲存，不支援搜尋和聚合分析 \(資料儲存在 \_source 中\)
* index – 是否構倒建排索引。設定成 false，無法被搜尋，但還是支援 aggregation，並出現在 \_source

中。

* index\_options: 控制將哪些資訊新增到倒排索引中，以進行搜尋和高亮顯示。它接受以下設定：docs \| freqs \| posistions \| offsets。
* norms – 如果欄位⽤用來過濾和聚合分析，可以關閉，節約儲存。
* doc\_values – 是否啟用 doc\_values，⽤用於排序和聚合分析。
* fielddata – 如果要對 text 型別啟用排序和聚合分析， fielddata 需要設定成true
* store – 預設不儲存，資料預設儲存在 \_source。
* coerce – 預設開啟，是否開啟資料型別的自動轉換\(例如，字串轉數字\)。
* fields 多欄位特性。
* dynamic – true / false / strict 控制 Mapping 的自動更新。

**Elasticsearch != 關係型資料庫。儘可能 Denormalize 資料，從而獲取最佳的效能，使用 Nested 型別的資料，查詢速度會慢幾倍。使用 Parent / Child 關係，查詢速度會慢幾百倍。**

### 讀寫優化

#### 寫入優化

**Translog**

預設 translog 是每 5 秒被 fsync 重新整理到硬碟。如果ES有大量的寫請求會導致頻繁IO影響效能，在允許資料丟失情況（極低概率，斷電或宕機等故障）可以調低translog的刷盤頻率、提高落盤檔案的大小和週期、並改用非同步方式來提升寫的效能:

```text
PUT my-index/_settings
{
  "translog.flush_threshold_size": "1gb",  #預設512M，當 translog 超過該值，會觸發 flush
  "translog.sync_interval": "60s", #預設5s
  "translog.durability": "async" #預設是 request，每個請求都落盤。設定成 async，非同步寫入
}
```

**Refresh\_interval**

預設情況下，ES每一秒會refresh一次，產生一個新的segment，這樣會導致產生的 segment較多，segment merge較為頻繁，系統開銷增大。如果對資料的實時查詢性要求較低，可以通過下面的命令提高refresh的時間間隔，降低系統開銷:

```text
PUT my-index/_settings
{
  "refresh_interval": "60s" #預設1s
}
```

**Index Buffer**

預設是10%，這意味著分配給一個節點的總堆疊的10％將用作所有分片共享的索引緩衝區大小，用滿會導致自動觸發refresh，可以通過編輯elasticsearch.yml檔案，調整配置引數適當調大：

```text
indices.memory.index_buffer_size: 20%  #預設10%
```

**Merge併發控制**

Lucene會不斷地把一些小的segment合併成一個大的segment，合併時併發執行緒數預設值是`Math.max(1, Math.min(4, <<node.processors, node.processors>> / 2))`，當節點配置的cpu核數較高時，merge佔用的資源可能會偏高，影響叢集的效能，可以通過下面的命令調整某個index的merge過程的併發度:

```text
PUT my-index/_settings
{
  "merge.scheduler.max_thread_count": 2
}
```

**Dynamic Mapping**

建議關閉dynamic index:

```text
PUT _cluster/settings
{
  "persistent": {
    "action.auto_create_index": false
  }
}

#或者設定白名單
PUT _cluster/settings
{
  "persistent": {
    "action.auto_create_index": "logstash-*,kibana*"
  }
}
```

**Bulk**

* 單個 bulk 請求體的資料量不要太大，官方建議大約5-15mb。
* 寫入端的 bulk 請求超時需要足夠長，建議60s以上。
* 寫入端儘量將資料輪詢打到不同節點。
* 客戶端批量寫入資料時儘量使用下面的bulk介面批量寫入，提高寫入效率：

  ```text
  POST _bulk
  {"index":{"_index":"test"}}
  {"field1":"value1"}
  {"index":{"_index":"test"}}
  {"field2":"value2"}
  {"index":{"_index":"test"}}
  {"field3":"value3"}
  ```

**寫入資料不指定\_id，讓ES自動產生**

如果使用者顯示指定id寫入資料時，ES會先發起查詢來確定index中是否已經有相同id的doc存在，若有則先刪除原有doc再寫入新doc，這樣每次寫入時，ES都會耗費一定的資源做查詢。如果不 指定就直接跳過查詢直接隨機id入庫，效能可快一倍。

**Routing**

對於資料量較大的index，一般會配置多個shard來分攤壓力。這種場景下，一個查詢會同時搜尋所有的shard，然後再將各個shard的結果合併後，返回給使用者。對於高併發的小查詢場景，每個分片通常僅抓取極少量資料，此時查詢過程中的排程開銷遠大於實際讀取資料的開銷，且查詢速度取決於最慢的一個分片。開啟routing功能後，ES會將routing相同的資料寫入到同一個分片中（也可以是多個，由index.routing\_partition\_size引數控制）。如果查詢時指定routing，那麼ES只會查詢routing指向的那個分片，可顯著降低排程開銷，提升查詢效率。

```text
PUT my_index/_doc/1?routing=user1&refresh=true 
{
  "title": "This is a document"
}

GET my_index/_doc/1?routing=user1 
```

**分片設定**

副本在寫入時設為 0，完成後再增加。

#### 查詢優化

**使用query-bool-filter組合取代普通query**

預設情況下，ES通過一定的演算法計算返回的每條資料與查詢語句的相關度，並通過\_score欄位來表徵。但對於非全文索引的使用場景，使用者並不care查詢結果與查詢條件的相關度，只是想精確的查詢目標資料。此時，可以通過query-bool-filter組合來讓ES不計算\_score，並且儘可能的快取filter的結果集，供後續包含相同filter的查詢使用，提高查詢效率。

```text
# 普通query查詢
GET my-index/_search
{
  "query": {
    "term": {
      "user": {
        "value": "kimchy"
      }
    }
  }
}

# query-bool-filter 加速查詢
GET my-index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "user": "kimchy"
          }
        }
      ]
    }
  }
}

```

**禁用萬用字元開始的正則表達**

萬用字元開頭的正則，效能非常糟糕，需避免使用。

```text
GET my-index/_search
{
  "query": {
    "wildcard": {
      "user": {
        "value": "*imchy"
      }
    }
  }
}
```

**避免查詢時指令碼**

避免使用script查詢，例如這裡想查詢userlist列表中元素個數等於3的doc：

```text
#避免使用script查詢
GET my-index/_search
{
  "query": {
    "script": {
      "script": "doc['userlist.keyword'].length==3"
    }
  }
}
```

可以在 Index 文件時，使用 Ingest Pipeline，計算並寫入user\_length用於記錄userlist列表中元素的個數，需要查詢的時候直接查詢user\_length。

```text
#定義pipline
PUT _ingest/pipeline/my-pipline
{
  "description": "score user length",
  "processors": [
    {
      "script": {
        "lang": "painless",
        "source": "ctx.user_length = ctx.userlist.length"  #新增user_length欄位
      }
    }
  ]
}

#在插入資料時使用pipline
POST my-index/_doc?pipeline=my-pipline
{
  "userlist": ["user1","user2","user3"]
}

```

