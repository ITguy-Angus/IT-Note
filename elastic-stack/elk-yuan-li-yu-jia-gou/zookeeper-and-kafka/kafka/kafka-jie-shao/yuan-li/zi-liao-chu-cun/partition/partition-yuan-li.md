# Partitions 資料存儲邏輯

\*\*\*\*

![overview of producer components](https://i.stack.imgur.com/qhGRl.png)

#### 工作流程

![](https://oscimg.oschina.net/oscnet/up-2f48d6a220cc54ef64a10f4b7a664aee3c8.png)

    Kafka 中訊息是以 topic 進行分類的， 生產者生產訊息，消費者消費訊息，都是面向 topic的。 topic 是邏輯上的概念，而 partition 是物理上的概念，每個 partition 對應於一個 log 檔案，該 log 檔案中儲存的就是 producer 生產的資料。 Producer 生產的資料會被不斷追加到該log 檔案末端，且每條資料都有自己的 offset。 消費者組中的每個消費者， 都會實時記錄自己消費到了哪個 offset，以便出錯恢復時，從上次的位置繼續消費。

#### Kafka檔案儲存機制

![](https://oscimg.oschina.net/oscnet/up-ca42fae206ea735aa4fb81f1bdb686c3d34.png)

    由於生產者生產的訊息會不斷追加到 log 檔案末尾， 為防止 log 檔案過大導致資料定位效率低下， Kafka 採取了分片和索引機制，將每個 partition 分為多個 segment。 每個 segment對應兩個檔案——“.index”檔案和“.log”檔案。 這些檔案位於一個資料夾下， 該資料夾的命名規則為： topic 名稱+分割槽序號。例如， first 這個 topic 有三個分割槽，則其對應的資料夾為 first-0,first-1,first-2

```text
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log

```

index 和 log 檔案以當前 segment 的第一條訊息的 offset 命名。下圖為 index 檔案和 log檔案的結構示意圖  
 ![](https://oscimg.oschina.net/oscnet/up-51fa3e2f9b4f73892b7faa189ae3b0c08fc.png)

 “.index”檔案儲存大量的索引資訊，“.log”檔案儲存大量的資料，索引檔案中的元資料指向對應資料檔案中 message 的物理偏移地址。

### Kafka 生產者

#### 分割槽策略

**1.    分割槽的原因**

* 方便在叢集中擴充套件，每個 Partition 可以通過調整以適應它所在的機器，而一個 topic又可以有多個 Partition 組成，因此整個叢集就可以適應任意大小的資料了；
* 可以提高併發，因為可以以 Partition 為單位讀寫了。

**2.     分割槽的原則**  
 我們需要將 producer 傳送的資料封裝成一個 ProducerRecord 物件。 ![](https://oscimg.oschina.net/oscnet/up-6ea0b8babd3a202d7e9ec8719ce2e53fed7.png)

* 指明 partition 的情況下，直接將指明的值直接作為 partiton 值；
* 沒有指明 partition 值但有 key 的情況下，將 key 的 hash 值與 topic 的 partition數進行取餘得到 partition 值；
* 既沒有 partition 值又沒有 key 值的情況下，第一次呼叫時隨機生成一個整數（後面每次呼叫在這個整數上自增），將這個值與 topic 可用的 partition 總數取餘得到 partition值，也就是常說的 round-robin 演算法。

#### 資料可靠性保證

    為保證 producer 傳送的資料，能可靠的傳送到指定的 topic， topic 的每個 partition 收到producer 傳送的資料後， 都需要向 producer 傳送 ack（acknowledgement 確認收到） ，如果producer 收到 ack， 就會進行下一輪的傳送，否則重新發送資料。   

![](https://oscimg.oschina.net/oscnet/up-7bfb46c1d9883f054ec8056d0f010d12328.png)

1. 副本資料同步策略  ![](https://oscimg.oschina.net/oscnet/up-3e78a555af36fb4c0107416c17273124fe5.png)  Kafka 選擇了第二種方案，原因如下：

* 1\).同樣為了容忍 n 臺節點的故障，第一種方案需要 2n+1 個副本，而第二種方案只需要 n+1個副本，而 Kafka 的每個分割槽都有大量的資料， 第一種方案會造成大量資料的冗餘。
* 2\).雖然第二種方案的網路延遲會比較高，但網路延遲對 Kafka 的影響較小

1. ISR  採用第二種方案之後，設想以下情景： leader 收到資料，所有 follower 都開始同步資料，但有一個 follower，因為某種故障，遲遲不能與 leader 進行同步，那 leader 就要一直等下去，直到它完成同步，才能傳送 ack。這個問題怎麼解決呢？Leader 維護了一個動態的 in-sync replica set \(ISR\)，意為和 leader 保持同步的 follower 集合。當 ISR 中的 follower 完成資料的同步之後， leader 就會給 follower 傳送 ack。如果 follower長 時 間 未 向 leader 同 步 數 據 ， 則 該 follower 將 被 踢 出 ISR ， 該 時 間 閾 值 [由replica.lag.time.max.ms](https://www.gushiciku.cn/jump/aHR0cHM6Ly93d3cub3NjaGluYS5uZXQvYWN0aW9uL0dvVG9MaW5rP3VybD1odHRwJTNBJTJGJTJGeG4tLXJlcGxpY2EtZWQ5cS5sYWcudGltZS5tYXgubXM=) 引數設定。 Leader 發生故障之後，就會從 ISR 中選舉新的 leader。
2. ack應答機制 對於某些不太重要的資料，對資料的可靠性要求不是很高，能夠容忍資料的少量丟失，  所以沒必要等 ISR 中的 follower 全部接收成功。所以 Kafka 為使用者提供了三種可靠性級別，使用者根據對可靠性和延遲的要求進行權衡，選擇以下的配置。   
3. **acks引數配置**： **acks**:  **0**： producer 不等待 broker 的 ack，這一操作提供了一個最低的延遲， broker 一接收到還沒有寫入磁碟就已經返回，當 broker 故障時有可能丟失資料； **1**： producer 等待 broker 的 ack， partition 的 leader 落盤成功後返回 ack，如果在 follower同步成功之前 leader 故障，那麼將會丟失資料；

    **-1（all）** ： producer 等待 broker 的 ack， partition 的 leader 和 follower 全部落盤成功後才返回 ack。但是如果在 follower 同步完成後， broker 傳送 ack 之前， leader 發生故障，那麼會造成資料重複。 ![](https://oscimg.oschina.net/oscnet/up-d2a9231074f9ade528fa4caf6d060888091.png)

![](https://oscimg.oschina.net/oscnet/up-3efdbb2e9f41d4efd793562947f329fbc26.png)

**故障處理細節**

![](https://oscimg.oschina.net/oscnet/up-da8b287e5492ef5d3b370a1f2a0f44bb79d.png)

**LEO：指的是每個副本最大的 offset；**

**HW：指的是消費者能見到的最大的 offset， ISR 佇列中最小的 LEO。**

**（1）follower 故障** follower 發生故障後會被臨時踢出 ISR，待該 follower 恢復後， follower 會讀取本地磁碟記錄的上次的 HW，並將 log 檔案高於 HW 的部分擷取掉，從 HW 開始向 leader 進行同步。等該 follower 的 LEO 大於等於該 Partition 的 HW，即 follower 追上 leader 之後，就可以重新加入 ISR 了。

**（2）leader故障** leader 發生故障之後，會從 ISR 中選出一個新的 leader，之後，為保證多個副本之間的資料一致性， 其餘的 follower 會先將各自的 log 檔案高於 HW 的部分截掉，然後從新的 leader 同步資料。

    **注意： 這隻能保證副本之間的資料一致性，並不能保證資料不丟失或者不重複。**

**Exactly Once 語義**

將伺服器的 ACK 級別設定為-1，可以保證 Producer 到 Server 之間不會丟失資料，即 AtLeast Once 語義。相對的，將伺服器 ACK 級別設定為 0，可以保證生產者每條訊息只會被髮送一次，即 At Most Once 語義。  
         At Least Once 可以保證資料不丟失，但是不能保證資料不重複；相對的， At Least Once可以保證資料不重複，但是不能保證資料不丟失。 但是，對於一些非常重要的資訊，比如說交易資料，下游資料消費者要求資料既不重複也不丟失，即 Exactly Once 語義。 在 0.11 版本以前的 Kafka，對此是無能為力的，只能保證資料不丟失，再在下游消費者對資料做全域性去重。對於多個下游應用的情況，每個都需要單獨做全域性去重，這就對效能造成了很大影響。  
         0.11 版本的 Kafka，引入了一項重大特性：冪等性。所謂的冪等性就是指 Producer 不論向 Server 傳送多少次重複資料， Server 端都只會持久化一條。冪等性結合 At Least Once 語義，就構成了 Kafka 的 Exactly Once 語義。即：

  
         **At Least Once + 冪等性 = Exactly Once**

        要啟用冪等性，只需要將 Producer 的引數中 enable.idompotence 設定為 true 即可。 Kafka的冪等性實現其實就是將原來下游需要做的去重放在了資料上游。開啟冪等性的 Producer 在初始化的時候會被分配一個 PID，發往同一 Partition 的訊息會附帶 Sequence Number。而Broker 端會對&lt;PID, Partition, SeqNumber&gt;做快取，當具有相同主鍵的訊息提交時， Broker 只會持久化一條。

        但是 PID 重啟就會變化，同時不同的 Partition 也具有不同主鍵，所以冪等性無法保證跨分割槽跨會話的 Exactly Once。

### Kafka 高效讀寫資料

#### 1）順序寫磁碟

     Kafka 的 producer 生產資料，要寫入到 log 檔案中，寫的過程是一直追加到檔案末端，為順序寫。 官網有資料表明，同樣的磁碟，順序寫能到 600M/s，而隨機寫只有 100K/s。這與磁碟的機械機構有關，順序寫之所以快，是因為其省去了大量磁頭定址的時間。

#### 2）零複製技術

![](https://oscimg.oschina.net/oscnet/up-83149a7ddd4d8bf0055f05294ffafe689c3.png)

### Zookeeper 在 Kafka 中的作用

    Kafka 叢集中有一個 broker 會被選舉為 Controller，負責管理叢集 broker 的上下線，所有 topic 的分割槽副本分配和 leader 選舉等工作。

    Controller 的管理工作都是依賴於 Zookeeper 的。

    以下為 partition 的 leader 選舉過程：

![](https://oscimg.oschina.net/oscnet/up-781be2df2490fe8b258c00309cd9407a166.png)

###   Kafka 事務

Kafka 從 0.11 版本開始引入了事務支援。事務可以保證 Kafka 在 Exactly Once 語義的基礎上，生產和消費可以跨分割槽和會話，要麼全部成功，要麼全部失敗。

#### Producer 事務

  
 為了實現跨分割槽跨會話的事務，需要引入一個全域性唯一的 Transaction ID，並將 Producer獲得的PID 和Transaction ID 繫結。這樣當Producer 重啟後就可以通過正在進行的 TransactionID 獲得原來的 PID。為了管理 Transaction， Kafka 引入了一個新的元件 Transaction Coordinator。 Producer 就是通過和 Transaction Coordinator 互動獲得 Transaction ID 對應的任務狀態。 Transaction Coordinator 還負責將事務所有寫入 Kafka 的一個內部 Topic，這樣即使整個服務重啟，由於事務狀態得到儲存，進行中的事務狀態可以得到恢復，從而繼續進行。

#### Consumer 事務   

上述事務機制主要是從 Producer 方面考慮，對於 Consumer 而言，事務的保證就會相對較弱，尤其時無法保證 Commit 的資訊被精確消費。這是由於 Consumer 可以通過 offset 訪問任意資訊，

> #### 1. When a producer is producing a message - It will specify the topic it wants to send the message to, is that right? Does it care about partitions?

Producer will decide target partition to place any message, depending on:

* Partition id, if it's specified within the message
* **key % num partitions**, if no partition id is mentioned
* Round robin if neither **partition id** nor **message key** are available in message, meaning only value is available

