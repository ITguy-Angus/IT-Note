# 選舉與主從同步

## Controller選舉

當添加一個分區或分區增加副本的時候，都要從所有副本中選舉一個新的Leader出來。

Leader如果選舉？投票怎麼玩？是不是所有的partition副本直接發起投票，開始競選呢？比如用ZK實現。

利用ZK如何實現選舉？ZK的什麼功能可以感知到節點的變化（增加或減少）？或者説ZK為什麼能實現加鎖和釋放鎖？

用到了3個特點：watch機制；節點不允許重複寫入；臨時節點。

這樣實現是比較簡單，但也會存在一定弊端。如果分區和副本數量過多，所有的副本都直接選舉的話，一旦某個節點增減，就會造成大量watch事件被觸發，ZK的負載就會過重。

kafka早期的版本就是這樣做的，後來換了一種實現方式。

不是所有的repalica都參與leader選舉，而是由其中的一個Broker統一來指揮，這個Broker的角色就叫做Controller（控制器）。

就像Redis Sentinel的架構，執行故障轉移的時候，必須要先從所有哨兵中選一個負責故障轉移的節點一樣。kafka 也要先從所有Broker中選出**唯一的一個**Controller。

所有Broker會嘗試在zookeeper中創建臨時節點/controller，只有一個能創建成功（先到先得）。

如果Controller掛掉了或者網絡出現了問題，ZK上的臨時節點會消失。其他的Brokder通過watch監聽到Controller下線的消息後，開始競選新的Controller。方法跟之前還是一樣的，誰先在ZK裏寫入一個/cotroller節點，誰就成為新的Controller。

成為Controller節點之後，它的責任也比其他節點重了幾分：

1. 監聽Broker變化
2. 監聽Topic變化
3. 監聽Partition變化
4. 獲取和管理Broker、Topic、Partition的信息
5. 管理Partiontion的主從信息

**分區副本Leader選舉**

Controller確定以後，就可以開始做分區選主的事了。下面就是找候選人了。顯然，每個replica都想推薦自己，但所有的replica都有競選資格嗎？並不是，這裏有幾個概念。

Assigned-Replicas（AR）：一個分區的所有副本。 In-Sync Replicas\(ISR\)：上邊所有副本中，跟leader數據保持一定程度同步的。 Out-Sync Replicas\(OSR\)：跟leader同步滯後過多的副本。

AR=ISR + OSR。正常情況下OSR是空的，大家正常同步，AR=ISR。

誰能參加選舉？肯定不是AR，也不是OR，而是ISR。而且這個ISR不是固定不變的，還是一個動態列表。

前面説過，如果同步延遲超30秒，就踢出ISR，進入OSR；如果趕上來了就加入ISR。

默認情況下，當leader副本發生故障時，只有在ISR集合中的副本才有資格被選舉為新的leader。

如果ISR為空呢？羣龍不能無首。在這種情況下，可以讓ISR之外的副本參與選舉。允許ISR之外的副本參與選舉，叫做unclean leader election。

```text
unclean.leader.election.enable=false
```

把這個參數改成true\(一般不建議開啟，會造成數據丟失\)。

Controller有了，候選人也有了ISR，那麼根據什麼規則確定leader呢？

我們首先來看分佈式系統中常見的選舉協議有哪些（或者説共識算法）？

ZAB（ZK）、Raft（Redis Sentinel）他們都是Paxos算法的變種，核心思想歸納起來都是：先到先得、少數服從多數。

但kafka沒有用這些方法，而是用了一種自己實現的算法。

為什麼呢？比如ZAB這種協議，可能會出現腦裂（節點不能互通的時候，出現多個leader）、驚羣效應（大量watch事件被觸發）。

在文檔中有説明：

[https://kafka.apachecn.org/documentation.html\#design\_replicatedlog](https://www.gushiciku.cn/jump/aHR0cHM6Ly93d3cub3NjaGluYS5uZXQvYWN0aW9uL0dvVG9MaW5rP3VybD1odHRwcyUzQSUyRiUyRmthZmthLmFwYWNoZWNuLm9yZyUyRmRvY3VtZW50YXRpb24uaHRtbCUyM2Rlc2lnbl9yZXBsaWNhdGVkbG9n)

提到kafka的選舉實現，最相近的是微軟的PacificA算法。

在這種算法中，默認是讓ISR中第一個replica變成leader。像中國皇帝傳位一樣，優先傳給皇長子

## **主從同步**

leader確定之後，客户端的讀寫只能操作leader節點。follower需要向leader同步數據。

不同的raplica的offset是不一樣的，同步到底怎麼同步呢？

在之後內容，需要先理解幾個概念。

**LEO（Log End Offset）：下一條等待寫入的消息的offset（最新的offset + 1）。**

**HW（Hign Watermark 高水位）：ISR中最小的LEO。Leader會管理所有ISR中最小的LEO為HW。**

**consumer最多隻能消費到HW之前的位置**。也就是説，其他副本沒有同步過去的消息，是不能被消費的。

![](https://oscimg.oschina.net/oscnet/up-59f31f903e6691ddc9e373c6ad6ad8c0cea.png)

kafka為什麼這麼設計？

如果在同步成功之前就被消費了，consumer group 的offset會偏大，如果leader崩潰，中間會丟失消息。

接着再看消息是如何同步的。

Replica 1與Replica2各同步了1條數據，HW推進了1，變成了7，LEO因Replica2推進了1，變成了7。

![](https://oscimg.oschina.net/oscnet/up-458275e68398246296d3526f64cff392159.png)

Replica 1與Replica2各同步了2條數據，HW和LEO重疊，都到了9。

![](https://oscimg.oschina.net/oscnet/up-723acc699c7a7d1dd8306662879be9b3a69.png)

在這需要了解一下，從節點如何與主節點保持同步？

1. follower節點會向Leader發送一個fetch請求，leader向follower發送數據後，即需要更新follower的LEO。
2. follower接收到數據響應後，依次寫入消息並且更新LEO。
3. leader更新HW（ISR最小的LEO）

kafka設計了獨特的ISR複製，可以在保障數據一致性情況下又可以提供高吞吐量。

**Replica故障處理**

**follower故障**

首先follower發生鼓掌，會被先踢出ISR。

follower恢復之後，從哪開始同步數據呢？

假設Replica1宕機。

![](https://oscimg.oschina.net/oscnet/up-535e35ca8a266c8e8356b2b7bf1c4d1f83a.png)

恢復以後，首先根據之前的記錄的HW（6），把高於HW的消息截掉（6、7）。

![](https://oscimg.oschina.net/oscnet/up-fbaba0b13a052aaa79d9829a0ace730f6fa.png)

然後向Leader同步消息。追上Leader之後（30秒），重新加入ISR。

![](https://oscimg.oschina.net/oscnet/up-72dc60a0ca1f99e00d778745981a6857fec.png)

**leader故障**

還以上圖為例，如果圖中Leader發生故障。

首先選一個Leader，因為Replica1優先，它將成為Leader。

為了保證數據一致，其他follower需要把高於HW的消息截掉（這裏沒有消息需要截取）。

然後Replica2同步數據。

此時原Leader中的數據8將丟失。

注意：這種機制只能保證副本之間的數據一致性，並不能保證數據不丟失或者不重複。

