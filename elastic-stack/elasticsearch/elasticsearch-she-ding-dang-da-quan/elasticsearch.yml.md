# Elasticsearch.yml



```text
##################### Elasticsearch Configuration Example ##################### 
#
# 只是挑些重要的配置選項進行註釋,其實自帶的已經有非常細緻的英文註釋了!
# https://www.elastic.co/guide/en/elasticsearch/reference/current/modules.html
#
################################### Cluster ################################### 
# 代表一個叢集,叢集中有多個節點,其中有一個為主節點,這個主節點是可以通過選舉產生的,主從節點是對於叢集內部來說的. 
# es的一個概念就是去中心化,字面上理解就是無中心節點,這是對於叢集外部來說的,因為從外部來看es叢集,在邏輯上是個整體,你與任何一個節點的通訊和與整個es叢集通訊是等價的。 
# cluster.name可以確定你的叢集名稱,當你的elasticsearch叢集在同一個網段中elasticsearch會自動的找到具有相同cluster.name的elasticsearch服務. 
# 所以當同一個網段具有多個elasticsearch叢集時cluster.name就成為同一個叢集的標識. 
# cluster.name: elasticsearch 
#################################### Node ##################################### 
# https://www.elastic.co/guide/en/elasticsearch/reference/5.1/modules-node.html#master-node
# 節點名稱同理,可自動生成也可手動配置. 
# node.name: node-1
# 允許一個節點是否可以成為一個master節點,es是預設叢集中的第一臺機器為master,如果這臺機器停止就會重新選舉master. 
# node.master: true 
# 允許該節點儲存資料(預設開啟) 
# node.data: true 
# 配置檔案中給出了三種配置高效能叢集拓撲結構的模式,如下： 
# 1. 如果你想讓節點從不選舉為主節點,只用來儲存資料,可作為負載器 
# node.master: false 
# node.data: true 
# node.ingest: false
# 2. 如果想讓節點成為主節點,且不儲存任何資料,並保有空閒資源,可作為協調器 
# node.master: true 
# node.data: false
# node.ingest: false 
# 3. 如果想讓節點既不稱為主節點,又不成為資料節點,那麼可將他作為搜尋器,從節點中獲取資料,生成搜尋結果等 
# node.master: false 
# node.data: false 
# node.ingest: true (可不指定預設開啟)
# 4. 僅作為協調器 
# node.master: false 
# node.data: false
# node.ingest: false
# 監控叢集狀態有一下外掛和API可以使用: 
# Use the Cluster Health API [http://localhost:9200/_cluster/health], the 
# Node Info API [http://localhost:9200/_nodes] or GUI tools # such as <http://www.elasticsearch.org/overview/marvel/>, 
# <http://github.com/karmi/elasticsearch-paramedic>, 
# <http://github.com/lukas-vlcek/bigdesk> and 
# <http://mobz.github.com/elasticsearch-head> to inspect the cluster state. 
# A node can have generic attributes associated with it, which can later be used 
# for customized shard allocation filtering, or allocation awareness. An attribute 
# is a simple key value pair, similar to node.key: value, here is an example: 
# 每個節點都可以定義一些與之關聯的通用屬性，用於後期叢集進行碎片分配時的過濾
# node.rack: rack314 
# 預設情況下，多個節點可以在同一個安裝路徑啟動，如果你想讓你的es只啟動一個節點，可以進行如下設定
# node.max_local_storage_nodes: 1 
#################################### Index #################################### 
# 設定索引的分片數,預設為5 
#index.number_of_shards: 5 
# 設定索引的副本數,預設為1: 
#index.number_of_replicas: 1 
# 配置檔案中提到的最佳實踐是,如果伺服器夠多,可以將分片提高,儘量將資料平均分佈到大叢集中去
# 同時,如果增加副本數量可以有效的提高搜尋效能 
# 需要注意的是,"number_of_shards" 是索引建立後一次生成的,後續不可更改設定 
# "number_of_replicas" 是可以通過API去實時修改設定的 
#################################### Paths #################################### 
# 配置檔案儲存位置 
# path.conf: /path/to/conf 
# 資料儲存位置(單個目錄設定) 
# path.data: /path/to/data 
# 多個資料儲存位置,有利於效能提升 
# path.data: /path/to/data1,/path/to/data2 
# 臨時檔案的路徑 
# path.work: /path/to/work 
# 日誌檔案的路徑 
# path.logs: /path/to/logs 
# 外掛安裝路徑 
# path.plugins: /path/to/plugins 
#################################### Plugin ################################### 
# 設定外掛作為啟動條件,如果一下外掛沒有安裝,則該節點服務不會啟動 
# plugin.mandatory: mapper-attachments,lang-groovy 
################################### Memory #################################### 
# 當JVM開始寫入交換空間時（swapping）ElasticSearch效能會低下,你應該保證它不會寫入交換空間 
# 設定這個屬性為true來鎖定記憶體,同時也要允許elasticsearch的程序可以鎖住記憶體,linux下可以通過 `ulimit -l unlimited` 命令 
# bootstrap.mlockall: true 
# 確保 ES_MIN_MEM 和 ES_MAX_MEM 環境變數設定為相同的值,以及機器有足夠的記憶體分配給Elasticsearch 
# 注意:記憶體也不是越大越好,一般64位機器,最大分配記憶體別才超過32G 
############################## Network And HTTP ############################### 
# 設定繫結的ip地址,可以是ipv4或ipv6的,預設為0.0.0.0 
# network.bind_host: 192.168.0.1 
# 設定其它節點和該節點互動的ip地址,如果不設定它會自動設定,值必須是個真實的ip地址 
# network.publish_host: 192.168.0.1 
# 同時設定bind_host和publish_host上面兩個引數 
# network.host: 192.168.0.1 
# 設定節點間互動的tcp埠,預設是9300 
# transport.tcp.port: 9300 
# 設定是否壓縮tcp傳輸時的資料，預設為false,不壓縮
# transport.tcp.compress: true 
# 設定對外服務的http埠,預設為9200 
# http.port: 9200 
# 設定請求內容的最大容量,預設100mb 
# http.max_content_length: 100mb 
# 使用http協議對外提供服務,預設為true,開啟 
# http.enabled: false 
###################### 使用head等外掛監控叢集資訊，需要開啟以下配置項 ###########
# http.cors.enabled: true
# http.cors.allow-origin: "*"
# http.cors.allow-credentials: true
################################### Gateway ################################### 
# gateway的型別,預設為local即為本地檔案系統,可以設定為本地檔案系統 
# gateway.type: local 
# 下面的配置控制怎樣以及何時啟動一整個叢集重啟的初始化恢復過程 
# (當使用shard gateway時,是為了儘可能的重用local data(本地資料)) 
# 一個叢集中的N個節點啟動後,才允許進行恢復處理 
# gateway.recover_after_nodes: 1 
# 設定初始化恢復過程的超時時間,超時時間從上一個配置中配置的N個節點啟動後算起 
# gateway.recover_after_time: 5m 
# 設定這個叢集中期望有多少個節點.一旦這N個節點啟動(並且recover_after_nodes也符合), 
# 立即開始恢復過程(不等待recover_after_time超時) 
# gateway.expected_nodes: 2
############################# Recovery Throttling ############################# 
# 下面這些配置允許在初始化恢復,副本分配,再平衡,或者新增和刪除節點時控制節點間的分片分配 
# 設定一個節點的並行恢復數 
# 1.初始化資料恢復時,併發恢復執行緒的個數,預設為4 
# cluster.routing.allocation.node_initial_primaries_recoveries: 4 
# 2.新增刪除節點或負載均衡時併發恢復執行緒的個數,預設為2 
# cluster.routing.allocation.node_concurrent_recoveries: 2 
# 設定恢復時的吞吐量(例如:100mb,預設為0無限制.如果機器還有其他業務在跑的話還是限制一下的好) 
# indices.recovery.max_bytes_per_sec: 20mb 
# 設定來限制從其它分片恢復資料時最大同時開啟併發流的個數,預設為5 
# indices.recovery.concurrent_streams: 5 
# 注意: 合理的設定以上引數能有效的提高叢集節點的資料恢復以及初始化速度 
################################## Discovery ################################## 
# 設定這個引數來保證叢集中的節點可以知道其它N個有master資格的節點.預設為1,對於大的叢集來說,可以設定大一點的值(2-4) 
# discovery.zen.minimum_master_nodes: 1 
# 探查的超時時間,預設3秒,提高一點以應對網路不好的時候,防止腦裂 
# discovery.zen.ping.timeout: 3s 
# For more information, see 
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-discovery-zen.html> 
# 設定是否開啟多播發現節點.預設是true. 
# 當多播不可用或者叢集跨網段的時候叢集通訊還是用單播吧 
# discovery.zen.ping.multicast.enabled: false 
# 這是一個叢集中的主節點的初始列表,當節點(主節點或者資料節點)啟動時使用這個列表進行探測 
# discovery.zen.ping.unicast.hosts: ["host1", "host2:port"] 
# Slow Log部分與GC log部分略,不過可以通過相關日誌優化搜尋查詢速度 
################  X-Pack ###########################################
# 官方外掛 相關設定請檢視此處
# https://www.elastic.co/guide/en/x-pack/current/xpack-settings.html
# 
############## Memory(重點需要調優的部分) ################ 
# Cache部分: 
# es有很多種方式來快取其內部與索引有關的資料.其中包括filter cache 
# filter cache部分: 
# filter cache是用來快取filters的結果的.預設的cache type是node type.node type的機制是所有的索引內部的分片共享filter cache.node type採用的方式是LRU方式.即:當快取達到了某個臨界值之後，es會將最近沒有使用的資料清除出filter cache.使讓新的資料進入es. 
# 這個臨界值的設定方法如下：indices.cache.filter.size 值型別：eg.:512mb 20%。預設的值是10%。 
# out of memory錯誤避免過於頻繁的查詢時叢集假死 
# 1.設定es的快取型別為Soft Reference,它的主要特點是據有較強的引用功能.只有當記憶體不夠的時候,才進行回收這類記憶體,因此在記憶體足夠的時候,它們通常不被回收.另外,這些引用物件還能保證在Java丟擲OutOfMemory異常之前,被設定為null.它可以用於實現一些常用圖片的快取,實現Cache的功能,保證最大限度的使用記憶體而不引起OutOfMemory.在es的配置檔案加上index.cache.field.type: soft即可. 
# 2.設定es最大快取資料條數和快取失效時間,通過設定index.cache.field.max_size: 50000來把快取field的最大值設定為50000,設定index.cache.field.expire: 10m把過期時間設定成10分鐘. 
# index.cache.field.max_size: 50000 
# index.cache.field.expire: 10m 
# index.cache.field.type: soft 
# field data部分&&circuit breaker部分： 
# 用於fielddata快取的記憶體數量,主要用於當使用排序,faceting操作時,elasticsearch會將一些熱點資料載入到記憶體中來提供給客戶端訪問,但是這種快取是比較珍貴的,所以對它進行合理的設定. 
# 可以使用值：eg:50mb 或者 30％(節點 node heap記憶體量),預設是：unbounded #indices.fielddata.cache.size： unbounded 
# field的超時時間.預設是-1,可以設定的值型別: 5m #indices.fielddata.cache.expire: -1 
# circuit breaker部分: 
# 斷路器是elasticsearch為了防止記憶體溢位的一種操作,每一種circuit breaker都可以指定一個記憶體界限觸發此操作,這種circuit breaker的設定有一個最高階別的設定:indices.breaker.total.limit 預設值是JVM heap的70%.當記憶體達到這個數量的時候會觸發記憶體回收
# 另外還有兩組子設定： 
#indices.breaker.fielddata.limit:當系統發現fielddata的數量達到一定數量時會觸發記憶體回收.預設值是JVM heap的70% 
#indices.breaker.fielddata.overhead:在系統要載入fielddata時會進行預先估計,當系統發現要載入進記憶體的值超過limit * overhead時會進行進行記憶體回收.預設是1.03 
#indices.breaker.request.limit:這種斷路器是elasticsearch為了防止OOM(記憶體溢位),在每次請求資料時設定了一個固定的記憶體數量.預設值是40% 
#indices.breaker.request.overhead:同上,也是elasticsearch在傳送請求時設定的一個預估係數,用來防止記憶體溢位.預設值是1 
# Translog部分: 
# 每一個分片(shard)都有一個transaction log或者是與它有關的預寫日誌,(write log),在es進行索引(index)或者刪除(delete)操作時會將沒有提交的資料記錄在translog之中,當進行flush 操作的時候會將tranlog中的資料傳送給Lucene進行相關的操作.一次flush操作的發生基於如下的幾個配置 
#index.translog.flush_threshold_ops:當發生多少次操作時進行一次flush.預設是 unlimited #index.translog.flush_threshold_size:當translog的大小達到此值時會進行一次flush操作.預設是512mb 
#index.translog.flush_threshold_period:在指定的時間間隔內如果沒有進行flush操作,會進行一次強制flush操作.預設是30m #index.translog.interval:多少時間間隔內會檢查一次translog,來進行一次flush操作.es會隨機的在這個值到這個值的2倍大小之間進行一次操作,預設是5s 
#index.gateway.local.sync:多少時間進行一次的寫磁碟操作,預設是5s 
```

