# 原理總結

 Kafka是最初由Linkedin公司開發，是一個分散式、支援分割槽的（partition）、多副本的（replica），基於zookeeper協調的分散式訊息系統，它的最大的特性就是可以實時的處理大量資料以滿足各種需求場景：比如基於hadoop的批處理系統、低延遲的實時系統、storm/Spark流式處理引擎，web/nginx日誌、訪問日誌，訊息服務等等，用scala語言編寫，Linkedin於2010年貢獻給了Apache基金會併成為頂級開源專案。

## 1.前言

 訊息佇列的效能好壞，其檔案儲存機制設計是衡量一個訊息佇列服務技術水平和最關鍵指標之一。下面將從Kafka檔案儲存機制和物理結構角度，分析Kafka是如何實現高效檔案儲存，及實際應用效果。

  **1.1  Kafka的特性:**

 - 高吞吐量、低延遲：kafka每秒可以處理幾十萬條訊息，它的延遲最低只有幾毫秒，每個topic可以分多個partition, consumer group 對partition進行consume操作。

 - 可擴充套件性：kafka叢集支援熱擴充套件

 - 永續性、可靠性：訊息被持久化到本地磁碟，並且支援資料備份防止資料丟失

 - 容錯性：允許叢集中節點失敗（若副本數量為n,則允許n-1個節點失敗）

 - 高併發：支援數千個客戶端同時讀寫

 **1.2   Kafka的使用場景：**

 - 日誌收集：一個公司可以用Kafka可以收集各種服務的log，通過kafka以統一介面服務的方式開放給各種consumer，例如hadoop、Hbase、Solr等。

 - 訊息系統：解耦和生產者和消費者、快取訊息等。

 - 使用者活動跟蹤：Kafka經常被用來記錄web使用者或者app使用者的各種活動，如瀏覽網頁、搜尋、點選等活動，這些活動資訊被各個伺服器釋出到kafka的topic中，然後訂閱者通過訂閱這些topic來做實時的監控分析，或者裝載到hadoop、資料倉庫中做離線分析和挖掘。

 - 運營指標：Kafka也經常用來記錄運營監控資料。包括收集各種分散式應用的資料，生產各種操作的集中反饋，比如報警和報告。

 - 流式處理：比如spark streaming和storm

 - 事件源

 **1.3  Kakfa的設計思想**

 -  **Kakfa Broker Leader的選舉：** Kakfa Broker叢集受Zookeeper管理。所有的Kafka Broker節點一起去Zookeeper上註冊一個臨時節點，因為只有一個Kafka Broker會註冊成功，其他的都會失敗，所以這個成功在Zookeeper上註冊臨時節點的這個Kafka Broker會成為Kafka Broker Controller，其他的Kafka broker叫Kafka Broker follower。（這個過程叫Controller在ZooKeeper註冊Watch）。這個Controller會監聽其他的Kafka Broker的所有資訊，如果這個kafka broker controller宕機了，在zookeeper上面的那個臨時節點就會消失，此時所有的kafka broker又會一起去Zookeeper上註冊一個臨時節點，因為只有一個Kafka Broker會註冊成功，其他的都會失敗，所以這個成功在Zookeeper上註冊臨時節點的這個Kafka Broker會成為Kafka Broker Controller，其他的Kafka broker叫Kafka Broker follower。例如：一旦有一個broker宕機了，這個kafka broker controller會讀取該宕機broker上所有的partition在zookeeper上的狀態，並選取ISR列表中的一個replica作為partition leader（如果ISR列表中的replica全掛，選一個倖存的replica作為leader; 如果該partition的所有的replica都宕機了，則將新的leader設定為-1，等待恢復，等待ISR中的任一個Replica“活”過來，並且選它作為Leader；或選擇第一個“活”過來的Replica（不一定是ISR中的）作為Leader），這個broker宕機的事情，kafka controller也會通知zookeeper，zookeeper就會通知其他的kafka broker。

 這裡曾經發生過一個bug，TalkingData使用Kafka0.8.1的時候，kafka controller在Zookeeper上註冊成功後，它和Zookeeper通訊的timeout時間是6s，也就是如果kafka controller如果有6s中沒有和Zookeeper做心跳，那麼Zookeeper就認為這個kafka controller已經死了，就會在Zookeeper上把這個臨時節點刪掉，那麼其他Kafka就會認為controller已經沒了，就會再次搶著註冊臨時節點，註冊成功的那個kafka broker成為controller，然後，之前的那個kafka controller就需要各種shut down去關閉各種節點和事件的監聽。 但是當kafka的讀寫流量都非常巨大的時候，TalkingData的一個bug是，由於網路等原因，kafka controller和Zookeeper有6s中沒有通訊，於是重新選舉出了一個新的kafka controller，但是原來的controller在shut down的時候總是不成功，這個時候producer進來的message由於Kafka叢集中存在兩個kafka controller而無法落地。導致資料淤積。

 **這裡曾經還有一個bug，** TalkingData使用Kafka0.8.1的時候，當ack=0的時候，表示producer傳送出去message，只要對應的kafka broker topic partition leader接收到的這條message，producer就返回成功，不管partition leader 是否真的成功把message真正存到kafka。 **當ack=1的時候，** 表示producer傳送出去message，同步的把message存到對應topic的partition的leader上，然後producer就返回成功，partition leader非同步的把message同步到其他partition replica上。 **當ack=all或-1，** 表示producer傳送出去message，同步的把message存到對應topic的partition的leader和對應的replica上之後，才返回成功。 **但是如果某個** **kafka controller**   **切換的時候，會導致partition leader的切換（老的**   kafka controller上面的partition leader會選舉到其他的kafka broker上 **）,但是這樣就會導致丟資料。**

 -   **Consumergroup：** 各個consumer（consumer 執行緒）可以組成一個組（ Consumer group  ），partition中的每個message只能被組（ Consumer group  ）中的一個consumer（consumer 執行緒）消費，如果一個message可以被多個consumer（consumer 執行緒）消費的話，那麼這些consumer必須在不同的組。Kafka不支援一個partition中的message由兩個或兩個以上的同一個consumer group下的consumer thread來處理，除非再啟動一個新的consumer group。所以如果想同時對一個topic做消費的話，啟動多個consumer group就可以了，但是要注意的是，這裡的多個consumer的消費都必須是順序讀取partition裡面的message，新啟動的consumer預設從partition佇列最頭端最新的地方開始阻塞的讀message。它不能像AMQ那樣可以多個BET作為consumer去互斥的（for update悲觀鎖）併發處理message，這是因為多個BET去消費一個Queue中的資料的時候，由於要保證不能多個執行緒拿同一條message，所以就需要行級別悲觀所（for update）,這就導致了consume的效能下降，吞吐量不夠。而kafka為了保證吞吐量，只允許同一個consumer group下的一個consumer執行緒去訪問一個partition。如果覺得效率不高的時候，可以加partition的數量來橫向擴充套件，那麼再加新的consumer thread去消費。如果想多個不同的業務都需要這個topic的資料，起多個consumer group就好了，大家都是順序的讀取message，offsite的值互不影響。這樣沒有鎖競爭，充分發揮了橫向的擴充套件性，吞吐量極高。這也就形成了分散式消費的概念。

 當啟動一個consumer group去消費一個topic的時候，無論topic裡面有多個少個partition，無論我們consumer group裡面配置了多少個consumer thread，這個consumer group下面的所有consumer thread一定會消費全部的partition；即便這個consumer group下只有一個consumer thread，那麼這個consumer thread也會去消費所有的partition。因此，最優的設計就是，consumer group下的consumer thread的數量等於partition數量，這樣效率是最高的。

 同一partition的一條message只能被同一個Consumer Group內的一個Consumer消費。不能夠一個consumer group的多個consumer同時消費一個partition。

 一個consumer group下，無論有多少個consumer，這個consumer group一定 會 去把這個topic下所有的partition都消費了。 當consumer group裡面的consumer數量小於這個topic下的partition數量的時候，如下圖groupA,groupB，就會出現一個conusmer thread消費多個partition的情況，總之是這個topic下的partition都會被消費。 如果consumer group裡面的consumer數量等於這個topic下的partition數量的時候，如下圖groupC，此時效率是最高的，每個partition都有一個consumer thread去消費。 當consumer group裡面的consumer數量大於這個topic下的partition數量的時候，如下圖GroupD，就會有一個consumer thread空閒。 因此，我們在設定consumer group的時候，只需要指明裡面有幾個consumer數量即可，無需指定對應的消費partition序號，consumer會自動進行rebalance。

 多個Consumer Group下的consumer可以消費同一條message，但是這種消費也是以o（1）的方式順序的讀取message去消費,，所以一定會重複消費這批message的，不能向AMQ那樣多個BET作為consumer消費（對message加鎖，消費的時候不能重複消費message）

 -  **Consumer Rebalance的觸發條件：** （1）Consumer增加或刪除會觸發 Consumer Group的Rebalance（2）Broker的增加或者減少都會觸發 Consumer Rebalance

 -  **Consumer：** Consumer處理partition裡面的message的時候是o（1）順序讀取的。所以必須維護著上一次讀到哪裡的offsite資訊。high level API,offset存於Zookeeper中，low level API的offset由自己維護。一般來說都是使用high level api的。Consumer的delivery gurarantee，預設是讀完message先commmit再處理message，autocommit預設是true，這時候先commit就會更新offsite+1，一旦處理失敗，offsite已經+1，這個時候就會丟message；也可以配置成讀完訊息處理再commit，這種情況下consumer端的響應就會比較慢的，需要等處理完才行。

 一般情況下，一定是一個consumer group處理一個topic的message。Best Practice是這個consumer group裡面consumer的數量等於topic裡面partition的數量，這樣效率是最高的，一個consumer thread處理一個partition。如果這個consumer group裡面consumer的數量小於topic裡面partition的數量，就會有consumer thread同時處理多個partition（這個是kafka自動的機制，我們不用指定），但是總之這個topic裡面的所有partition都會被處理到的。。如果這個consumer group裡面consumer的數量大於topic裡面partition的數量，多出的consumer thread就會閒著啥也不幹，剩下的是一個consumer thread處理一個partition，這就造成了資源的浪費，因為一個partition不可能被兩個consumer thread去處理。 所以我們線上的分散式多個service服務，每個service裡面的kafka consumer數量都小於對應的topic的partition數量，但是所有服務的consumer數量只和等於partition的數量，這是因為分散式service服務的所有consumer都來自一個consumer group，如果來自不同的consumer group就會處理重複的message了（同一個consumer group下的consumer不能處理同一個partition，不同的consumer group可以處理同一個topic，那麼都是順序處理message，一定會處理重複的。一般這種情況都是兩個不同的業務邏輯，才會啟動兩個consumer group來處理一個topic）。

 如果producer的流量增大，當前的topic的parition數量=consumer數量，這時候的應對方式就是很想擴充套件： 增加topic下的partition，同時增加這個consumer group下的consumer。

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b6e55079b91f13db062ce66173729be50045d822b63d027c54981a6e9aa069458.jpg)

 -  **Delivery Mode :** Kafka producer 傳送message不用維護message的offsite資訊，因為這個時候，offsite就相當於一個自增id，producer就儘管傳送message就好了。而且Kafka與AMQ不同，AMQ大都用在處理業務邏輯上，而Kafka大都是日誌，所以Kafka的producer一般都是大批量的batch傳送message，向這個topic一次性發送一大批message，load balance到一個partition上，一起插進去，offsite作為自增id自己增加就好。但是Consumer端是需要維護這個partition當前消費到哪個message的offsite資訊的，這個offsite資訊，high level api是維護在Zookeeper上，low level api是自己的程式維護。（Kafka管理介面上只能顯示high level api的consumer部分，因為low level api的partition offsite資訊是程式自己維護，kafka是不知道的，無法在管理介面上展示 ）當使用high level api的時候，先拿message處理，再定時自動commit offsite+1（也可以改成手動）, 並且kakfa處理message是沒有鎖操作的。因此如果處理message失敗，此時還沒有commit offsite+1，當consumer thread重啟後會重複消費這個message。但是作為高吞吐量高併發的實時處理系統，at least once的情況下，至少一次會被處理到，是可以容忍的。如果無法容忍，就得使用low level api來自己程式維護這個offsite資訊，那麼想什麼時候commit offsite+1就自己搞定了。

 -  **Topic & Partition：** Topic相當於傳統訊息系統MQ中的一個佇列queue，producer端傳送的message必須指定是傳送到哪個topic，但是不需要指定topic下的哪個partition，因為kafka會把收到的message進行load balance，均勻的分佈在這個topic下的不同的partition上（ hash\(message\) % \[broker數量\]  ）。物理上儲存上，這個topic會分成一個或多個partition，每個partiton相當於是一個子queue。在物理結構上，每個partition對應一個物理的目錄（資料夾），資料夾命名是\[topicname\]\_\[partition\]\_\[序號\]，一個topic可以有無數多的partition，根據業務需求和資料量來設定。在kafka配置檔案中可隨時更高num.partitions引數來配置更改topic的partition數量，在建立Topic時通過引數指定parittion數量。Topic建立之後通過Kafka提供的工具也可以修改partiton數量。

 一般來說，（1）一個Topic的Partition數量大於等於Broker的數量，可以提高吞吐率。（2）同一個Partition的Replica儘量分散到不同的機器，高可用。

 **當add a new partition的時候** ，partition裡面的message不會重新進行分配，原來的partition裡面的message資料不會變，新加的這個partition剛開始是空的，隨後進入這個topic的message就會重新參與所有partition的load balance

 -  **Partition Replica：** 每個partition可以在其他的kafka broker節點上存副本，以便某個kafka broker節點宕機不會影響這個kafka叢集。存replica副本的方式是按照kafka broker的順序存。例如有5個kafka broker節點，某個topic有3個partition，每個partition存2個副本，那麼partition1存broker1,broker2，partition2存broker2,broker3。。。以此類推 （replica副本數目不能大於kafka broker節點的數目，否則報錯。這裡的replica數其實就是partition的副本總數，其中包括一個leader，其他的就是copy副本） 。這樣如果某個broker宕機，其實整個kafka內資料依然是完整的。但是，replica副本數越高，系統雖然越穩定，但是回來帶資源和效能上的下降；replica副本少的話，也會造成系統丟資料的風險。

 （1）怎樣傳送訊息：producer先把message傳送到partition leader，再由leader傳送給其他partition follower。（如果讓producer傳送給每個replica那就太慢了）

 （2）在向Producer傳送ACK前需要保證有多少個Replica已經收到該訊息：根據ack配的個數而定

 （3）怎樣處理某個Replica不工作的情況：如果這個部工作的partition replica不在ack列表中，就是producer在傳送訊息到partition leader上，partition leader向partition follower傳送message沒有響應而已，這個不會影響整個系統，也不會有什麼問題。如果這個不工作的partition replica在ack列表中的話，producer傳送的message的時候會等待這個不工作的partition replca寫message成功，但是會等到time out，然後返回失敗因為某個ack列表中的partition replica沒有響應，此時kafka會自動的把這個部工作的partition replica從ack列表中移除，以後的producer傳送message的時候就不會有這個ack列表下的這個部工作的partition replica了。 

 （4）怎樣處理Failed Replica恢復回來的情況：如果這個partition replica之前不在ack列表中，那麼啟動後重新受Zookeeper管理即可，之後producer傳送message的時候，partition leader會繼續傳送message到這個partition follower上。如果這個partition replica之前在ack列表中，此時重啟後，需要把這個partition replica再手動加到ack列表中。（ack列表是手動新增的，出現某個部工作的partition replica的時候自動從ack列表中移除的）

 -  **Partition leader與follower：** partition也有leader和follower之分。leader是主partition，producer寫kafka的時候先寫partition leader，再由partition leader push給其他的partition follower。partition leader與follower的資訊受Zookeeper控制，一旦partition leader所在的broker節點宕機，zookeeper會衝其他的broker的partition follower上選擇follower變為parition leader。

 -  **Topic分配partition和partition replica的演算法：** （1）將Broker（size=n）和待分配的Partition排序。（2）將第i個Partition分配到第（i%n）個Broker上。（3）將第i個Partition的第j個Replica分配到第（\(i + j\) % n）個Broker上

 **- 訊息投遞可靠性**

 一個訊息如何算投遞成功，Kafka提供了三種模式：

 - 第一種是啥都不管，傳送出去就當作成功，這種情況當然不能保證訊息成功投遞到broker；

 - 第二種是Master-Slave模型，只有當Master和所有Slave都接收到訊息時，才算投遞成功，這種模型提供了最高的投遞可靠性，但是損傷了效能；

 - 第三種模型，即只要Master確認收到訊息就算投遞成功；實際使用時，根據應用特性選擇，絕大多數情況下都會中和可靠性和效能選擇第三種模型

 訊息在broker上的可靠性，因為訊息會持久化到磁碟上，所以如果正常stop一個broker，其上的資料不會丟失；但是如果不正常stop，可能會使存在頁面快取來不及寫入磁碟的訊息丟失，這可以通過配置flush頁面快取的週期、閾值緩解，但是同樣會頻繁的寫磁碟會影響效能，又是一個選擇題，根據實際情況配置。

 訊息消費的可靠性，Kafka提供的是“At least once”模型，因為訊息的讀取進度由offset提供，offset可以由消費者自己維護也可以維護在zookeeper裡，但是當訊息消費後consumer掛掉，offset沒有即時寫回，就有可能發生重複讀的情況，這種情況同樣可以通過調整commit offset週期、閾值緩解，甚至消費者自己把消費和commit offset做成一個事務解決，但是如果你的應用不在乎重複消費，那就乾脆不要解決，以換取最大的效能。

 -  **Partition ack：** 當ack=1，表示producer寫partition leader成功後，broker就返回成功，無論其他的partition follower是否寫成功。當ack=2，表示producer寫partition leader和其他一個follower成功的時候，broker就返回成功，無論其他的partition follower是否寫成功。當ack=-1\[parition的數量\]的時候，表示只有producer全部寫成功的時候，才算成功，kafka broker才返回成功資訊。 這裡需要注意的是，如果ack=1的時候，一旦有個broker宕機導致partition的follower和leader切換，會導致丟資料。

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af983935153f5945457186c2677a83b13546f60c37d8438b74d22734b3707c56a863ff7.jpg)

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4bc7735e8d70369cdafd977091281f604b6542c106aeb55cbb237cafeb3d1fde42.jpg)

 -  **message狀態** ：在Kafka中，訊息的狀態被儲存在consumer中，broker不會關心哪個訊息被消費了被誰消費了，只記錄一個offset值（指向partition中下一個要被消費的訊息位置），這就意味著如果consumer處理不好的話，broker上的一個訊息可能會被消費多次。

 -  **message持久化** ：Kafka中會把訊息持久化到本地檔案系統中，並且保持o\(1\)極高的效率。我們眾所周知IO讀取是非常耗資源的效能也是最慢的，這就是為了資料庫的瓶頸經常在IO上，需要換SSD硬碟的原因。但是Kafka作為吞吐量極高的MQ，卻可以非常高效的message持久化到檔案。這是因為Kafka是順序寫入o（1）的時間複雜度，速度非常快。也是高吞吐量的原因。由於message的寫入持久化是順序寫入的，因此message在被消費的時候也是按順序被消費的，保證partition的message是順序消費的。一般的機器,單機每秒100k條資料。

 -  **message有效期** ：Kafka會長久保留其中的訊息，以便consumer可以多次消費，當然其中很多細節是可配置的。

 -  **Produer :** Producer向Topic傳送message，不需要指定partition，直接傳送就好了。kafka通過partition ack來控制是否傳送成功並把資訊返回給producer，producer可以有任意多的thread，這些kafka伺服器端是不care的。Producer端的delivery guarantee預設是At least once的。也可以設定Producer非同步傳送實現At most once。Producer可以用主鍵冪等性實現Exactly once

 -  **Kafka高吞吐量** ：Kafka的高吞吐量體現在讀寫上，分散式併發的讀和寫都非常快，寫的效能體現在以o\(1\)的時間複雜度進行順序寫入。讀的效能體現在以o\(1\)的時間複雜度進行順序讀取，   對topic進行partition分割槽，consume group中的consume執行緒可以以很高能效能進行順序讀。

 - Kafka delivery guarantee\(message傳送保證\)：（1）At most once訊息可能會丟，絕對不會重複傳輸；（2）At least once 訊息絕對不會丟，但是可能會重複傳輸；（3）Exactly once每條資訊肯定會被傳輸一次且僅傳輸一次，這是使用者想要的。

 -  **批量傳送** ：Kafka支援以訊息集合為單位進行批量傳送，以提高push效率。

 -  **push-and-pull** : Kafka中的Producer和consumer採用的是push-and-pull模式，即Producer只管向broker push訊息，consumer只管從broker pull訊息，兩者對訊息的生產和消費是非同步的。

 -  **Kafka叢集中broker之間的關係** ：不是主從關係，各個broker在叢集中地位一樣，我們可以隨意的增加或刪除任何一個broker節點。

 -  **負載均衡方面** ：Kafka提供了一個 metadata API來管理broker之間的負載（對Kafka0.8.x而言，對於0.7.x主要靠zookeeper來實現負載均衡）。

 -  **同步非同步** ：Producer採用非同步push方式，極大提高Kafka系統的吞吐率（可以通過引數控制是採用同步還是非同步方式）。

 -  **分割槽機制partition** ：Kafka的broker端支援訊息分割槽partition，Producer可以決定把訊息發到哪個partition，在一個partition   中message的順序就是Producer傳送訊息的順序，一個topic中可以有多個partition，具體partition的數量是可配置的。partition的概念使得kafka作為MQ可以橫向擴充套件，吞吐量巨大。partition可以設定replica副本，replica副本存在不同的kafka broker節點上，第一個partition是leader,其他的是follower，message先寫到partition leader上，再由partition leader push到parition follower上。所以說kafka可以水平擴充套件，也就是擴充套件partition。

 -  **離線資料裝載** ：Kafka由於對可拓展的資料持久化的支援，它也非常適合向Hadoop或者資料倉庫中進行資料裝載。

 -  **實時資料與離線資料：** kafka既支援離線資料也支援實時資料，因為kafka的message持久化到檔案，並可以設定有效期，因此可以把kafka作為一個高效的儲存來使用，可以作為離線資料供後面的分析。當然作為分散式實時訊息系統，大多數情況下還是用於實時的資料處理的，但是當cosumer消費能力下降的時候可以通過message的持久化在淤積資料在kafka。

 -  **外掛支援** ：現在不少活躍的社群已經開發出不少外掛來拓展Kafka的功能，如用來配合Storm、Hadoop、flume相關的外掛。

 -  **解耦** :  相當於一個MQ，使得Producer和Consumer之間非同步的操作，系統之間解耦

 -  **冗餘** :  replica有多個副本，保證一個broker node宕機後不會影響整個服務

 -  **擴充套件性** :  broker節點可以水平擴充套件，partition也可以水平增加，partition replica也可以水平增加

 -  **峰值** :  在訪問量劇增的情況下，kafka水平擴充套件, 應用仍然需要繼續發揮作用

 -  **可恢復性** :  系統的一部分元件失效時，由於有partition的replica副本，不會影響到整個系統。

 -  **順序保證性** ：由於kafka的producer的寫message與consumer去讀message都是順序的讀寫，保證了高效的效能。

 -  **緩衝** ：由於producer那面可能業務很簡單，而後端consumer業務會很複雜並有資料庫的操作，因此肯定是producer會比consumer處理速度快，如果沒有kafka，producer直接呼叫consumer，那麼就會造成整個系統的處理速度慢，加一層kafka作為MQ，可以起到緩衝的作用。

 -  **非同步通訊** ：作為MQ，Producer與Consumer非同步通訊

## 2.Kafka檔案儲存機制

 **2.1 Kafka部分名詞解釋如下：**

 Kafka 的釋出訂閱的物件是topic。我們可以為每類資料建立一個topic，把向topic釋出訊息的客戶端稱作producer，從topic訂閱訊息的客戶端稱作consumer。Producers和consumers可以同時從多個topic讀寫資料。一個kafka叢集由一個或多個broker伺服器組成，它負責持久化和備份具體的kafka訊息。

*  **Broker** ：Kafka節點，一個Kafka節點就是一個broker，多個broker可以組成一個Kafka叢集。
*  **Topic** ：一類訊息，訊息存放的目錄即主題，例如page view日誌、click日誌等都可以以topic的形式存在，Kafka叢集能夠同時負責多個topic的分發。
*  **Partition** ：topic物理上的分組，一個topic可以分為多個partition，每個partition是一個有序的佇列
*  **Segment** ：partition物理上由多個segment組成，每個Segment存著message資訊
*  **Producer** : 生產message傳送到topic
*  **Consumer** : 訂閱topic消費message, consumer作為一個執行緒來消費
*  **Consumer Group** ：一個 Consumer Group包含多個consumer, 這個是預先在配置檔案中配置好的。各個consumer（consumer 執行緒）可以組成一個組（Consumer group ），partition中的每個message只能被組（Consumer group ） 中的一個consumer（consumer 執行緒 ）消費，如果一個message可以被多個consumer（consumer 執行緒 ） 消費的話，那麼這些consumer必須在不同的組。Kafka不支援一個partition中的message由兩個或兩個以上的consumer thread來處理，即便是來自不同的consumer group的也不行。它不能像AMQ那樣可以多個BET作為consumer去處理message，這是因為多個BET去消費一個Queue中的資料的時候，由於要保證不能多個執行緒拿同一條message，所以就需要行級別悲觀所（for update）,這就導致了consume的效能下降，吞吐量不夠。而kafka為了保證吞吐量，只允許一個consumer執行緒去訪問一個partition。如果覺得效率不高的時候，可以加partition的數量來橫向擴充套件，那麼再加新的consumer thread去消費。這樣沒有鎖競爭，充分發揮了橫向的擴充套件性，吞吐量極高。這也就形成了分散式消費的概念。

 **2.2 kafka一些原理概念**

 1.持久化

 kafka使用檔案儲存訊息\(append only log\),這就直接決定kafka在效能上嚴重依賴檔案系統的本身特性.且無論任何OS下,對檔案系統本身的優化是非常艱難的.檔案快取/直接記憶體對映等是常用的手段.因為kafka是對日誌檔案進行append操作,因此磁碟檢索的開支是較小的;同時為了減少磁碟寫入的次數,broker會將訊息暫時buffer起來,當訊息的個數\(或尺寸\)達到一定閥值時,再flush到磁碟,這樣減少了磁碟IO呼叫的次數.對於kafka而言,較高效能的磁碟,將會帶來更加直接的效能提升.

 2.效能

 除磁碟IO之外,我們還需要考慮網路IO,這直接關係到kafka的吞吐量問題.kafka並沒有提供太多高超的技巧;對於producer端,可以將訊息buffer起來,當訊息的條數達到一定閥值時,批量傳送給broker;對於consumer端也是一樣,批量fetch多條訊息.不過訊息量的大小可以通過配置檔案來指定.對於kafka broker端,似乎有個sendfile系統呼叫可以潛在的提升網路IO的效能:將檔案的資料對映到系統記憶體中,socket直接讀取相應的記憶體區域即可,而無需程序再次copy和交換\(這裡涉及到"磁碟IO資料"/"核心記憶體"/"程序記憶體"/"網路緩衝區",多者之間的資料copy\).

 其實對於producer/consumer/broker三者而言,CPU的開支應該都不大,因此啟用訊息壓縮機制是一個良好的策略;壓縮需要消耗少量的CPU資源,不過對於kafka而言,網路IO更應該需要考慮.可以將任何在網路上傳輸的訊息都經過壓縮.kafka支援gzip/snappy等多種壓縮方式.

 3.負載均衡

 kafka叢集中的任何一個broker,都可以向producer提供metadata資訊,這些metadata中包含"叢集中存活的servers列表"/"partitions leader列表"等資訊\(請參看zookeeper中的節點資訊\). 當producer獲取到metadata資訊之後, producer將會和Topic下所有partition leader保持socket連線;訊息由producer直接通過socket傳送到broker,中間不會經過任何"路由層".

 非同步傳送，將多條訊息暫且在客戶端buffer起來,並將他們批量傳送到broker;小資料IO太多,會拖慢整體的網路延遲,批量延遲傳送事實上提升了網路效率;不過這也有一定的隱患,比如當producer失效時,那些尚未傳送的訊息將會丟失。

 4.Topic模型

 其他JMS實現,訊息消費的位置是有prodiver保留,以便避免重複傳送訊息或者將沒有消費成功的訊息重發等,同時還要控制訊息的狀態.這就要求JMS broker需要太多額外的工作.在kafka中,partition中的訊息只有一個consumer在消費,且不存在訊息狀態的控制,也沒有複雜的訊息確認機制,可見kafka broker端是相當輕量級的.當訊息被consumer接收之後,consumer可以在本地儲存最後訊息的offset,並間歇性的向zookeeper註冊offset.由此可見,consumer客戶端也很輕量級。

 kafka中consumer負責維護訊息的消費記錄,而broker則不關心這些,這種設計不僅提高了consumer端的靈活性,也適度的減輕了broker端設計的複雜度;這是和眾多JMS prodiver的區別.此外,kafka中訊息ACK的設計也和JMS有很大不同,kafka中的訊息是批量\(通常以訊息的條數或者chunk的尺寸為單位\)傳送給consumer,當訊息消費成功後,向zookeeper提交訊息的offset,而不會向broker交付ACK.或許你已經意識到,這種"寬鬆"的設計,將會有"丟失"訊息/"訊息重發"的危險.

 5.訊息傳輸一致

 Kafka提供3種訊息傳輸一致性語義：最多1次，最少1次，恰好1次。

 最少1次：可能會重傳資料，有可能出現數據被重複處理的情況;

 最多1次：可能會出現資料丟失情況;

 恰好1次：並不是指真正只傳輸1次，只不過有一個機制。確保不會出現“資料被重複處理”和“資料丟失”的情況。

 at most once: 消費者fetch訊息,然後儲存offset,然後處理訊息;當client儲存offset之後,但是在訊息處理過程中consumer程序失效\(crash\),導致部分訊息未能繼續處理.那麼此後可能其他consumer會接管,但是因為offset已經提前儲存,那麼新的consumer將不能fetch到offset之前的訊息\(儘管它們尚沒有被處理\),這就是"at most once".

 at least once: 消費者fetch訊息,然後處理訊息,然後儲存offset.如果訊息處理成功之後,但是在儲存offset階段zookeeper異常或者consumer失效,導致儲存offset操作未能執行成功,這就導致接下來再次fetch時可能獲得上次已經處理過的訊息,這就是"at least once".

 "Kafka Cluster"到消費者的場景中可以採取以下方案來得到“恰好1次”的一致性語義：

 最少1次＋消費者的輸出中額外增加已處理訊息最大編號：由於已處理訊息最大編號的存在，不會出現重複處理訊息的情況。

 6.副本

 kafka中,replication策略是基於partition,而不是topic;kafka將每個partition資料複製到多個server上,任何一個partition有一個leader和多個follower\(可以沒有\);備份的個數可以通過broker配置檔案來設定。leader處理所有的read-write請求,follower需要和leader保持同步.Follower就像一個"consumer",消費訊息並儲存在本地日誌中;leader負責跟蹤所有的follower狀態,如果follower"落後"太多或者失效,leader將會把它從replicas同步列表中刪除.當所有的follower都將一條訊息儲存成功,此訊息才被認為是"committed",那麼此時consumer才能消費它,這種同步策略,就要求follower和leader之間必須具有良好的網路環境.即使只有一個replicas例項存活,仍然可以保證訊息的正常傳送和接收,只要zookeeper叢集存活即可.

 選擇follower時需要兼顧一個問題,就是新leader server上所已經承載的partition leader的個數,如果一個server上有過多的partition leader,意味著此server將承受著更多的IO壓力.在選舉新leader,需要考慮到"負載均衡",partition leader較少的broker將會更有可能成為新的leader.

 7.log

 每個log entry格式為"4個位元組的數字N表示訊息的長度" + "N個位元組的訊息內容";每個日誌都有一個offset來唯一的標記一條訊息,offset的值為8個位元組的數字,表示此訊息在此partition中所處的起始位置..每個partition在物理儲存層面,有多個log file組成\(稱為segment\).segment file的命名為"最小offset".kafka.例如"00000000000.kafka";其中"最小offset"表示此segment中起始訊息的offset.

 獲取訊息時,需要指定offset和最大chunk尺寸,offset用來表示訊息的起始位置,chunk size用來表示最大獲取訊息的總長度\(間接的表示訊息的條數\).根據offset,可以找到此訊息所在segment檔案,然後根據segment的最小offset取差值,得到它在file中的相對位置,直接讀取輸出即可。

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af98393df13a39b63693d13bcd7ba20544cce217c849a24c1fa39ea919c9bf30f75a31e.jpg)

 8.分散式

 kafka使用zookeeper來儲存一些meta資訊,並使用了zookeeper watch機制來發現meta資訊的變更並作出相應的動作\(比如consumer失效,觸發負載均衡等\)

 Broker node registry: 當一個kafka broker啟動後,首先會向zookeeper註冊自己的節點資訊\(臨時znode\),同時當broker和zookeeper斷開連線時,此znode也會被刪除.

 Broker Topic Registry: 當一個broker啟動時,會向zookeeper註冊自己持有的topic和partitions資訊,仍然是一個臨時znode.

 Consumer and Consumer group: 每個consumer客戶端被建立時,會向zookeeper註冊自己的資訊;此作用主要是為了"負載均衡".一個group中的多個consumer可以交錯的消費一個topic的所有partitions;簡而言之,保證此topic的所有partitions都能被此group所消費,且消費時為了效能考慮,讓partition相對均衡的分散到每個consumer上.

 Consumer id Registry: 每個consumer都有一個唯一的ID\(host:uuid,可以通過配置檔案指定,也可以由系統生成\),此id用來標記消費者資訊.

 Consumer offset Tracking: 用來跟蹤每個consumer目前所消費的partition中最大的offset.此znode為持久節點,可以看出offset跟group\_id有關,以表明當group中一個消費者失效,其他consumer可以繼續消費.

 Partition Owner registry: 用來標記partition正在被哪個consumer消費.臨時znode。此節點表達了"一個partition"只能被group下一個consumer消費,同時當group下某個consumer失效,那麼將會觸發負載均衡\(即:讓partitions在多個consumer間均衡消費,接管那些"遊離"的partitions\)

 當consumer啟動時,所觸發的操作:

 A\) 首先進行"Consumer id Registry";

 B\) 然後在"Consumer id Registry"節點下注冊一個watch用來監聽當前group中其他consumer的"leave"和"join";只要此znode path下節點列表變更,都會觸發此group下consumer的負載均衡.\(比如一個consumer失效,那麼其他consumer接管partitions\).

 C\) 在"Broker id registry"節點下,註冊一個watch用來監聽broker的存活情況;如果broker列表變更,將會觸發所有的groups下的consumer重新balance.

 總結:

 1\) Producer端使用zookeeper用來"發現"broker列表,以及和Topic下每個partition leader建立socket連線併發送訊息.

 2\) Broker端使用zookeeper用來註冊broker資訊,已經監測partition leader存活性.

 3\) Consumer端使用zookeeper用來註冊consumer資訊,其中包括consumer消費的partition列表等,同時也用來發現broker列表,並和partition leader建立socket連線，並獲取訊息。

 9.Leader的選擇

 Kafka的核心是日誌檔案，日誌檔案在叢集中的同步是分散式資料系統最基礎的要素。

 如果leaders永遠不會down的話我們就不需要followers了！一旦leader down掉了，需要在followers中選擇一個新的leader.但是followers本身有可能延時太久或者crash，所以必須選擇高質量的follower作為leader.必須保證，一旦一個訊息被提交了，但是leader down掉了，新選出的leader必須可以提供這條訊息。大部分的分散式系統採用了多數投票法則選擇新的leader,對於多數投票法則，就是根據所有副本節點的狀況動態的選擇最適合的作為leader.Kafka並不是使用這種方法。

 Kafka動態維護了一個同步狀態的副本的集合（a set of in-sync replicas），簡稱ISR，在這個集合中的節點都是和leader保持高度一致的，任何一條訊息必須被這個集合中的每個節點讀取並追加到日誌中了，才回通知外部這個訊息已經被提交了。因此這個集合中的任何一個節點隨時都可以被選為leader.ISR在ZooKeeper中維護。ISR中有f+1個節點，就可以允許在f個節點down掉的情況下不會丟失訊息並正常提供服。ISR的成員是動態的，如果一個節點被淘汰了，當它重新達到“同步中”的狀態時，他可以重新加入ISR.這種leader的選擇方式是非常快速的，適合kafka的應用場景。

 一個邪惡的想法：如果所有節點都down掉了怎麼辦？Kafka對於資料不會丟失的保證，是基於至少一個節點是存活的，一旦所有節點都down了，這個就不能保證了。

 實際應用中，當所有的副本都down掉時，必須及時作出反應。可以有以下兩種選擇:

 1. 等待ISR中的任何一個節點恢復並擔任leader。

 2. 選擇所有節點中（不只是ISR）第一個恢復的節點作為leader.

 這是一個在可用性和連續性之間的權衡。如果等待ISR中的節點恢復，一旦ISR中的節點起不起來或者資料都是了，那叢集就永遠恢復不了了。如果等待ISR意外的節點恢復，這個節點的資料就會被作為線上資料，有可能和真實的資料有所出入，因為有些資料它可能還沒同步到。Kafka目前選擇了第二種策略，在未來的版本中將使這個策略的選擇可配置，可以根據場景靈活的選擇。

 這種窘境不只Kafka會遇到，幾乎所有的分散式資料系統都會遇到。

 10.副本管理

 以上僅僅以一個topic一個分割槽為例子進行了討論，但實際上一個Kafka將會管理成千上萬的topic分割槽.Kafka儘量的使所有分割槽均勻的分佈到叢集所有的節點上而不是集中在某些節點上，另外主從關係也儘量均衡這樣每個幾點都會擔任一定比例的分割槽的leader.

 優化leader的選擇過程也是很重要的，它決定了系統發生故障時的空窗期有多久。Kafka選擇一個節點作為“controller”,當發現有節點down掉的時候它負責在游泳分割槽的所有節點中選擇新的leader,這使得Kafka可以批量的高效的管理所有分割槽節點的主從關係。如果controller down掉了，活著的節點中的一個會備切換為新的controller.

 11.Leader與副本同步

 對於某個分割槽來說，儲存正分割槽的"broker"為該分割槽的"leader"，儲存備份分割槽的"broker"為該分割槽的"follower"。備份分割槽會完全複製正分割槽的訊息，包括訊息的編號等附加屬性值。為了保持正分割槽和備份分割槽的內容一致，Kafka採取的方案是在儲存備份分割槽的"broker"上開啟一個消費者程序進行消費，從而使得正分割槽的內容與備份分割槽的內容保持一致。一般情況下，一個分割槽有一個“正分割槽”和零到多個“備份分割槽”。可以配置“正分割槽+備份分割槽”的總數量，關於這個配置，不同主題可以有不同的配置值。注意，生產者，消費者只與儲存正分割槽的"leader"進行通訊。

 Kafka允許topic的分割槽擁有若干副本，這個數量是可以配置的，你可以為每個topic配置副本的數量。Kafka會自動在每個副本上備份資料，所以當一個節點down掉時資料依然是可用的。

 Kafka的副本功能不是必須的，你可以配置只有一個副本，這樣其實就相當於只有一份資料。

 建立副本的單位是topic的分割槽，每個分割槽都有一個leader和零或多個followers.所有的讀寫操作都由leader處理，一般分割槽的數量都比broker的數量多的多，各分割槽的leader均勻的分佈在brokers中。所有的followers都複製leader的日誌，日誌中的訊息和順序都和leader中的一致。followers向普通的consumer那樣從leader那裡拉取訊息並儲存在自己的日誌檔案中。

 許多分散式的訊息系統自動的處理失敗的請求，它們對一個節點是否著（alive）”有著清晰的定義。Kafka判斷一個節點是否活著有兩個條件：

 1. 節點必須可以維護和ZooKeeper的連線，Zookeeper通過心跳機制檢查每個節點的連線。

 2. 如果節點是個follower,他必須能及時的同步leader的寫操作，延時不能太久。

 符合以上條件的節點準確的說應該是“同步中的（in sync）”，而不是模糊的說是“活著的”或是“失敗的”。Leader會追蹤所有“同步中”的節點，一旦一個down掉了，或是卡住了，或是延時太久，leader就會把它移除。至於延時多久算是“太久”，是由引數replica.lag.max.messages決定的，怎樣算是卡住了，怎是由引數replica.lag.time.max.ms決定的。

 只有當訊息被所有的副本加入到日誌中時，才算是“committed”，只有committed的訊息才會傳送給consumer，這樣就不用擔心一旦leader down掉了訊息會丟失。Producer也可以選擇是否等待訊息被提交的通知，這個是由引數acks決定的。

 Kafka保證只要有一個“同步中”的節點，“committed”的訊息就不會丟失。

 **2.3  kafka拓撲結構**

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229b3b6fe1660611a84fad94c0184c3c8ffd833162454b763f1b21e6af063921a6ac.jpg)

 一個典型的Kafka叢集中包含若干Producer（可以是web前端FET，或者是伺服器日誌等），若干broker（Kafka支援水平擴充套件，一般broker數量越多，叢集吞吐率越高），若干ConsumerGroup，以及一個Zookeeper叢集。Kafka通過Zookeeper管理Kafka叢集配置：選舉Kafka broker的leader，以及在Consumer Group發生變化時進行rebalance，因為consumer消費kafka topic的partition的offsite資訊是存在Zookeeper的。Producer使用push模式將訊息釋出到broker，Consumer使用pull模式從broker訂閱並消費訊息。

 分析過程分為以下4個步驟：

*  topic中partition儲存分佈
*  partiton中檔案儲存方式 \(partition在linux伺服器上就是一個目錄（資料夾）\)
*  partiton中segment檔案儲存結構
*  在partition中如何通過offset查詢message

 通過上述4過程詳細分析，我們就可以清楚認識到kafka檔案儲存機制的奧祕。

 **2.3 topic中partition儲存分佈**

 假設實驗環境中Kafka叢集只有一個broker，xxx/message-folder為資料檔案儲存根目錄，在Kafka broker中server.properties檔案配置\(引數log.dirs=xxx/message-folder\)，例如建立2個topic名 稱分別為report\_push、launch\_info, partitions數量都為partitions=4

 儲存路徑和目錄規則為：

 xxx/message-folder

\|--report\_push-0

\|--report\_push-1

\|--report\_push-2

\|--report\_push-3

\|--launch\_info-0

\|--launch\_info-1

\|--launch\_info-2

\|--launch\_info-3

 在Kafka檔案儲存中，同一個topic下有多個不同partition，每個partition為一個目錄，partiton命名規則為 topic名稱+有序序號 ，第一個partiton序號從0開始，序號最大值為partitions數量減1。

 訊息傳送時都被髮送到一個topic，其本質就是一個目錄，而topic由是由一些Partition組成， Partition是一個Queue的結構，每個Partition中的訊息都是有序的，生產的訊息被不斷追加到Partition上，其中的每一個訊息都被賦予了一個唯一的offset值。

 Kafka叢集會儲存所有的訊息，不管訊息有沒有被消費； 我們可以設定訊息的過期時間，只有過期的資料才會被自動清除以釋放磁碟空間。 比如我們設定訊息過期時間為2天，那麼這2天內的所有訊息都會被儲存到叢集中，資料只有超過了兩天才會被清除。

 Kafka只維護在Partition中的offset值，因為這個offsite標識著這個partition的message消費到哪條了。Consumer每消費一個訊息，offset就會加1。其實訊息的狀態完全是由Consumer控制的，Consumer可以跟蹤和重設這個offset值，這樣的話Consumer就可以讀取任意位置的訊息。

 把訊息日誌以Partition的形式存放有多重考慮，第一，方便在叢集中擴充套件，每個Partition可以通過調整以適應它所在的機器，而一個topic又可以有多個Partition組成，因此整個叢集就可以適應任意大小的資料了；第二就是可以提高併發，因為可以以Partition為單位讀寫了。

 通過上面介紹的我們可以知道，kafka中的資料是持久化的並且能夠容錯的。Kafka允許使用者為每個topic設定副本數量，副本數量決定了有幾個broker來存放寫入的資料。如果你的副本數量設定為3，那麼一份資料就會被存放在3臺不同的機器上，那麼就允許有2個機器失敗。一般推薦副本數量至少為2，這樣就可以保證增減、重啟機器時不會影響到資料消費。如果對資料持久化有更高的要求，可以把副本數量設定為3或者更多。

 Kafka中的topic是以partition的形式存放的，每一個topic都可以設定它的partition數量，Partition的數量決定了組成topic的message的數量。Producer在生產資料時，會按照一定規則（這個規則是可以自定義的）把訊息釋出到topic的各個partition中。上面將的副本都是以partition為單位的，不過只有一個partition的副本會被選舉成leader作為讀寫用。

 關於如何設定partition值需要考慮的因素。 一個partition只能被一個消費者消費（一個消費者可以同時消費多個partition） ，因此，如果設定的partition的數量小於consumer的數量，就會有消費者消費不到資料。所以，推薦partition的數量一定要大於同時執行的consumer的數量。另外一方面，建議partition的數量大於叢集broker的數量，這樣leader partition就可以均勻的分佈在各個broker中，最終使得叢集負載均衡。在Cloudera,每個topic都有上百個partition。需要注意的是，kafka需要為每個partition分配一些記憶體來快取訊息資料，如果partition數量越大，就要為kafka分配更大的heap space。

 **2.4 partiton中檔案儲存方式**

*  每個partion\(目錄\)相當於一個巨型檔案被平均分配到多個大小相等segment\(段\)資料檔案中。但每個段segment file訊息數量不一定相等，這種特性方便old segment file快速被刪除。
*  每個partiton只需要支援順序讀寫就行了，segment檔案生命週期由服務端配置引數決定。

 這樣做的好處就是能快速刪除無用檔案，有效提高磁碟利用率。

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229bb07047626534ac361da3f14177cef3f781e086dd583ed71c8eff07f62b34a086.jpg)

 **2.5 partiton中segment檔案儲存結構**

 producer發message到某個topic，message會被均勻的分佈到多個partition上（隨機或根據使用者指定的回撥函式進行分佈），kafka broker收到message往對應partition的最後一個segment上新增該訊息，當某個segment上的訊息條數達到配置值或訊息釋出時間超過閾值時，segment上的訊息會被flush到磁碟，只有flush到磁碟上的訊息consumer才能消費，segment達到一定的大小後將不會再往該segment寫資料，broker會建立新的segment。

 每個part在記憶體中對應一個index，記錄每個segment中的第一條訊息偏移。

*  segment file組成：由2大部分組成，分別為index file和data file，此2個檔案一一對應，成對出現，字尾".index"和“.log”分別表示為segment索引檔案、資料檔案.
*  segment檔案命名規則：partion全域性的第一個segment從0開始，後續每個segment檔名為上一個全域性partion的最大offset\(偏移message數\)。數值最大為64位long大小，19位數字字元長度，沒有數字用0填充。

 每個segment中儲存很多條訊息，訊息id由其邏輯位置決定，即從訊息id可直接定位到訊息的儲存位置，避免id到位置的額外對映。

 下面檔案列表是筆者在Kafka broker上做的一個實驗，建立一個topicXXX包含1 partition，設定每個segment大小為500MB,並啟動producer向Kafka broker寫入大量資料,如下圖2所示segment檔案列表形象說明了上述2個規則：

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af9839355a9889a87e88fd1c1493b549733614085e9458f0499ead135dc8b5d65a0eab7.jpg)

 以上述圖2中一對segment file檔案為例，說明segment中index&lt;—-&gt;data file對應關係物理結構如下：

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229b35b48eb252c7b0cc21cf3441a27343bdb4935f675ecf38cf2a57c8904a00de7b.jpg)

 上述圖3中索引檔案儲存大量元資料，資料檔案儲存大量訊息，索引檔案中元資料指向對應資料檔案中message的物理偏移地址。其中以索引檔案中 元資料3,497為例，依次在資料檔案中表示第3個message\(在全域性partiton表示第368772個message\)、以及該訊息的物理偏移 地址為497。

 從上述圖3瞭解到segment data file由許多message組成，下面詳細說明message物理結構如下：

![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af98393df6f2ce2f48154ca53c1e73b0b7a6a4c04c8f3eca53e309684692a9cf10983ce.jpg)

####  引數說明：

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229bdc5dff56405a439f8e5aadb2ee37cce47bbfd992086267b95edc2ac939521575.jpg)

 **2.6 在partition中如何通過offset查詢message**

 例如讀取offset=368776的message，需要通過下面2個步驟查詢。

*  第一步查詢segment file

   上述圖2為例，其中00000000000000000000.index表示最開始的檔案，起始偏移量\(offset\)為0.第二個檔案 00000000000000368769.index的訊息量起始偏移量為368770 = 368769 + 1.同樣，第三個檔案00000000000000737337.index的起始偏移量為737338=737337 + 1，其他後續檔案依次類推，以起始偏移量命名並排序這些檔案，只要根據offset \*\*二分查詢\*\*檔案列表，就可以快速定位到具體檔案。

   當offset=368776時定位到00000000000000368769.index\|log

*  第二步通過segment file查詢message通過第一步定位到segment file，當offset=368776時，依次定位到00000000000000368769.index的元資料物理位置和 00000000000000368769.log的物理偏移地址，然後再通過00000000000000368769.log順序查詢直到 offset=368776為止。

 segment index file採取稀疏索引儲存方式，它減少索引檔案大小，通過mmap可以直接記憶體操作，稀疏索引為資料檔案的每個對應message設定一個元資料指標,它 比稠密索引節省了更多的儲存空間，但查詢起來需要消耗更多的時間。

 kafka會記錄offset到zk中。但是，zk client api對zk的頻繁寫入是一個低效的操作。0.8.2 kafka引入了native offset storage，將offset管理從zk移出，並且可以做到水平擴充套件。其原理就是利用了kafka的compacted topic，offset以consumer group,topic與partion的組合作為key直接提交到compacted topic中。同時Kafka又在記憶體中維護了的三元組來維護最新的offset資訊，consumer來取最新offset資訊的時候直接記憶體裡拿即可。當然，kafka允許你快速的checkpoint最新的offset資訊到磁碟上。

## 3.Partition Replication原則

 Kafka高效檔案儲存設計特點

*  Kafka把topic中一個parition大檔案分成多個小檔案段，通過多個小檔案段，就容易定期清除或刪除已經消費完檔案，減少磁碟佔用。
*  通過索引資訊可以快速定位message和確定response的最大大小。
*  通過index元資料全部對映到memory，可以避免segment file的IO磁碟操作。
*  通過索引檔案稀疏儲存，可以大幅降低index檔案元資料佔用空間大小。

 1. Kafka叢集partition replication預設自動分配分析

 下面以一個Kafka叢集中4個Broker舉例，建立1個topic包含4個Partition，2 Replication；資料Producer流動如圖所示：

 \(1\) ![](https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af9839379e1fc1f4fb31c04f0071c5bd4c4d0e580f6a887cb8df70d6460248068b6f609.jpg)

 \(2\)當叢集中新增2節點，Partition增加到6個時分佈情況如下：

![](https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4b3ecdb9cc1f38b8fb3bf1ae16ce553641c4da278cd53230a63b9a9f0cbc8d292a.jpg)

 副本分配邏輯規則如下：

*  在Kafka叢集中，每個Broker都有均等分配Partition的Leader機會。
*  上述圖Broker Partition中，箭頭指向為副本，以Partition-0為例:broker1中parition-0為Leader，Broker2中Partition-0為副本。
*  上述圖種每個Broker\(按照BrokerId有序\)依次分配主Partition,下一個Broker為副本，如此迴圈迭代分配，多副本都遵循此規則。

 副本分配演算法如下：

*  將所有N Broker和待分配的i個Partition排序.
*  將第i個Partition分配到第\(i mod n\)個Broker上.
*  將第i個Partition的第j個副本分配到第\(\(i + j\) mod n\)個Broker上.

## 4.Kafka Broker一些特性

 4.1 無狀態的Kafka Broker :

 1. Broker沒有副本機制，一旦broker宕機，該broker的訊息將都不可用。

 2. Broker不儲存訂閱者的狀態，由訂閱者自己儲存。

 3. 無狀態導致訊息的刪除成為難題（可能刪除的訊息正在被訂閱），kafka採用基於時間的SLA\(服務水平保證\)，訊息儲存一定時間（通常為7天）後會被刪除。

 4. 訊息訂閱者可以rewind back到任意位置重新進行消費，當訂閱者故障時，可以選擇最小的offset進行重新讀取消費訊息。

 4.2 message的交付與生命週期 ：

 1. 不是嚴格的JMS， 因此kafka對訊息的重複、丟失、錯誤以及順序型沒有嚴格的要求。 **（這是與AMQ最大的區別）**

 2. kafka提供at-least-once delivery,即當consumer宕機後，有些訊息可能會被重複delivery。

 3. 因每個partition只會被consumer group內的一個consumer消費，故kafka保證每個partition內的訊息會被順序的訂閱。

 4. Kafka為每條訊息為每條訊息計算CRC校驗，用於錯誤檢測，crc校驗不通過的訊息會直接被丟棄掉。

 4.3 壓縮

 Kafka支援以集合（batch）為單位傳送訊息，在此基礎上，Kafka還支援對訊息集合進行壓縮，Producer端可以通過GZIP或Snappy格式對訊息集合進行壓縮。Producer端進行壓縮之後，在Consumer端需進行解壓。壓縮的好處就是減少傳輸的資料量，減輕對網路傳輸的壓力，在對大資料處理上，瓶頸往往體現在網路上而不是CPU。

 那麼如何區分訊息是壓縮的還是未壓縮的呢，Kafka在訊息頭部添加了一個描述壓縮屬性位元組，這個位元組的後兩位表示訊息的壓縮採用的編碼，如果後兩位為0，則表示訊息未被壓縮。

 4.4 訊息可靠性

 在訊息系統中，保證訊息在生產和消費過程中的可靠性是十分重要的，在實際訊息傳遞過程中，可能會出現如下三中情況：

 - 一個訊息傳送失敗

 - 一個訊息被髮送多次

 - 最理想的情況：exactly-once ,一個訊息傳送成功且僅傳送了一次

 有許多系統聲稱它們實現了exactly-once，但是它們其實忽略了生產者或消費者在生產和消費過程中有可能失敗的情況。比如雖然一個Producer成功傳送一個訊息，但是訊息在傳送途中丟失，或者成功傳送到broker，也被consumer成功取走，但是這個consumer在處理取過來的訊息時失敗了。

 從Producer端看：Kafka是這麼處理的，當一個訊息被髮送後，Producer會等待broker成功接收到訊息的反饋（可通過引數控制等待時間），如果訊息在途中丟失或是其中一個broker掛掉，Producer會重新發送（我們知道Kafka有備份機制，可以通過引數控制是否等待所有備份節點都收到訊息）。

 從Consumer端看：前面講到過partition，broker端記錄了partition中的一個offset值，這個值指向Consumer下一個即將消費message。當Consumer收到了訊息，但卻在處理過程中掛掉，此時Consumer可以通過這個offset值重新找到上一個訊息再進行處理。Consumer還有許可權控制這個offset值，對持久化到broker端的訊息做任意處理。

 4.5 備份機制

 備份機制是Kafka0.8版本的新特性，備份機制的出現大大提高了Kafka叢集的可靠性、穩定性。有了備份機制後，Kafka允許叢集中的節點掛掉後而不影響整個叢集工作。一個備份數量為n的叢集允許n-1個節點失敗。在所有備份節點中，有一個節點作為lead節點，這個節點儲存了其它備份節點列表，並維持各個備份間的狀體同步。下面這幅圖解釋了Kafka的備份機制：

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229bd8dfc8b370eb4018b3ea7c34839bf1068857ff8662890271af6939bc5950fc17.jpg)

 4.6 Kafka高效性相關設計

 **4.6.1 訊息的持久化**

 Kafka高度依賴檔案系統來儲存和快取訊息\(AMQ的nessage是持久化到mysql資料庫中的\)，因為一般的人認為磁碟是緩慢的，這導致人們對持久化結構具有競爭性持懷疑態度。其實，磁碟的快或者慢，這決定於我們如何使用磁碟。因為磁碟線性寫的速度遠遠大於隨機寫。線性讀寫在大多數應用場景下是可以預測的。

 **4.6.2 常數時間效能保證**

 每個Topic的Partition的是一個大資料夾，裡面有無數個小資料夾segment，但partition是一個佇列，佇列中的元素是segment,消費的時候先從第0個segment開始消費，新來message存在最後一個訊息佇列中。對於segment也是對佇列，佇列元素是message,有對應的offsite標識是哪個message。消費的時候先從這個segment的第一個message開始消費，新來的message存在segment的最後。

 訊息系統的持久化佇列可以構建在對一個檔案的讀和追加上，就像一般情況下的日誌解決方案。它有一個優點，所有的操作都是常數時間，並且讀寫之間不會相互阻塞。這種設計具有極大的效能優勢：最終系統性能和資料大小完全無關，伺服器可以充分利用廉價的硬碟來提供高效的訊息服務。

 事實上還有一點，磁碟空間的無限增大而不影響效能這點，意味著我們可以提供一般訊息系統無法提供的特性。比如說，訊息被消費後不是立馬被刪除，我們可以將這些訊息保留一段相對比較長的時間（比如一個星期）。

## 5.Kafka 生產者-消費者

 訊息系統通常都會由生產者，消費者，Broker三大部分組成，生產者會將訊息寫入到Broker，消費者會從Broker中讀取出訊息，不同的MQ實現的Broker實現會有所不同，不過Broker的本質都是要負責將訊息落地到服務端的儲存系統中。具體步驟如下：

1.  生產者客戶端應用程式產生訊息：
   1.  客戶端連線物件將訊息包裝到請求中傳送到服務端
   2.  服務端的入口也有一個連線物件負責接收請求，並將訊息以檔案的形式儲存起來
   3.  服務端返回響應結果給生產者客戶端
2.  消費者客戶端應用程式消費訊息：
   1.  客戶端連線物件將消費資訊也包裝到請求中傳送給服務端
   2.  服務端從檔案儲存系統中取出訊息
   3.  服務端返回響應結果給消費者客戶端
   4.  客戶端將響應結果還原成訊息並開始處理訊息

 **5.1  Producers**

 Producers直接傳送訊息到broker上的leader partition，不需要經過任何中介或其他路由轉發。為了實現這個特性，kafka叢集中的每個broker都可以響應producer的請求，並返回topic的一些元資訊，這些元資訊包括哪些機器是存活的，topic的leader partition都在哪，現階段哪些leader partition是可以直接被訪問的。

 **Producer客戶端自己控制著訊息被推送到哪些partition。** 實現的方式可以是隨機分配、實現一類隨機負載均衡演算法，或者指定一些分割槽演算法。Kafka提供了介面供使用者實現自定義的partition，使用者可以為每個訊息指定一個partitionKey，通過這個key來實現一些hash分割槽演算法。比如，把userid作為partitionkey的話，相同userid的訊息將會被推送到同一個partition。

 以Batch的方式推送資料可以極大的提高處理效率，kafka Producer 可以將訊息在記憶體中累計到一定數量後作為一個batch傳送請求。Batch的數量大小可以通過Producer的引數控制，引數值可以設定為累計的訊息的數量（如500條）、累計的時間間隔（如100ms）或者累計的資料大小\(64KB\)。通過增加batch的大小，可以減少網路請求和磁碟IO的次數，當然具體引數設定需要在效率和時效性方面做一個權衡。

 Producers可以非同步的並行的向kafka傳送訊息，但是通常producer在傳送完訊息之後會得到一個future響應，返回的是offset值或者傳送過程中遇到的錯誤。這其中有個非常重要的引數“acks”,這個引數決定了producer要求leader partition 收到確認的副本個數，如果acks設定數量為0，表示producer不會等待broker的響應，所以，producer無法知道訊息是否傳送成功，這樣有可能會導致資料丟失，但同時，acks值為0會得到最大的系統吞吐量。

 若acks設定為1，表示producer會在leader partition收到訊息時得到broker的一個確認，這樣會有更好的可靠性，因為客戶端會等待直到broker確認收到訊息。若設定為-1，producer會在所有備份的partition收到訊息時得到broker的確認，這個設定可以得到最高的可靠性保證。

 Kafka 訊息有一個定長的header和變長的位元組陣列組成。因為kafka訊息支援位元組陣列，也就使得kafka可以支援任何使用者自定義的序列號格式或者其它已有的格式如Apache Avro、protobuf等。Kafka沒有限定單個訊息的大小，但我們推薦訊息大小不要超過1MB,通常一般訊息大小都在1~10kB之前。

 釋出訊息時，kafka client先構造一條訊息，將訊息加入到訊息集set中（kafka支援批量釋出，可以往訊息集合中新增多條訊息，一次行釋出），send訊息時，producer client需指定訊息所屬的topic。

 **5.2  Consumers**

 Kafka提供了兩套consumer api，分為high-level api和sample-api。Sample-api 是一個底層的API，它維持了一個和單一broker的連線，並且這個API是完全無狀態的，每次請求都需要指定offset值，因此，這套API也是最靈活的。

 在kafka中，當前讀到哪條訊息的offset值是由consumer來維護的 ，因此，consumer可以自己決定如何讀取kafka中的資料。 比如，consumer可以通過重設offset值來重新消費已消費過的資料。不管有沒有被消費，kafka會儲存資料一段時間，這個時間週期是可配置的，只有到了過期時間，kafka才會刪除這些資料。（這一點與AMQ不一樣，AMQ的message一般來說都是持久化到mysql中的，消費完的message會被delete掉）

 High-level API封裝了對叢集中一系列broker的訪問，可以透明的消費一個topic。它自己維持了已消費訊息的狀態，即每次消費的都是下一個訊息。

 High-level API還支援以組的形式消費topic，如果consumers有同一個組名，那麼kafka就相當於一個佇列訊息服務，而各個consumer均衡的消費相應partition中的資料。若consumers有不同的組名，那麼此時kafka就相當與一個廣播服務，會把topic中的所有訊息廣播到每個consumer。

 High level api和Low level api是針對consumer而言的，和producer無關。

 High level api是consumer讀的partition的offsite是存在zookeeper上。High level api 會啟動另外一個執行緒去每隔一段時間，offsite自動同步到zookeeper上。換句話說，如果使用了High level api， 每個message只能被讀一次，一旦讀了這條message之後，無論我consumer的處理是否ok。High level api的另外一個執行緒會自動的把offiste+1同步到zookeeper上。如果consumer讀取資料出了問題，offsite也會在zookeeper上同步。因此，如果consumer處理失敗了，會繼續執行下一條。這往往是不對的行為。因此，Best Practice是一旦consumer處理失敗，直接讓整個conusmer group拋Exception終止，但是最後讀的這一條資料是丟失了，因為在zookeeper裡面的offsite已經+1了。等再次啟動conusmer group的時候，已經從下一條開始讀取處理了。

 Low level api 是consumer讀的partition的offsite在consumer自己的程式中維護。不會同步到zookeeper上。但是為了kafka manager能夠方便的監控，一般也會手動的同步到zookeeper上。這樣的好處是一旦讀取某個message的consumer失敗了，這條message的offsite我們自己維護，我們不會+1。下次再啟動的時候，還會從這個offsite開始讀。這樣可以做到exactly once對於資料的準確性有保證。

 對於Consumer group：

 1. 允許consumer group（包含多個consumer，如一個叢集同時消費）對一個topic進行消費，不同的consumer group之間獨立消費。

 2. 為了對減小一個consumer group中不同consumer之間的分散式協調開銷，指定partition為最小的並行消費單位，即一個group內的consumer只能消費不同的partition。

![](https://mdimg.wxwenku.com/getimg/356ed03bdc643f9448b3f6485edc229b42eb0ea624e3f3d94f5de821f6319e3a4543f58e04320a9099ef17264ab97637.jpg)

 Consumer與Partition的關係：

 - 如果consumer比partition多，是浪費，因為kafka的設計是在一個partition上是不允許併發的，所以consumer數不要大於partition數

 - 如果consumer比partition少，一個consumer會對應於多個partitions，這裡主要合理分配consumer數和partition數，否則會導致partition裡面的資料被取的不均勻

 - 如果consumer從多個partition讀到資料，不保證資料間的順序性，kafka只保證在一個partition上資料是有序的，但多個partition，根據你讀的順序會有不同

 - 增減consumer，broker，partition會導致rebalance，所以rebalance後consumer對應的partition會發生變化

 - High-level介面中獲取不到資料的時候是會block的

 負載低的情況下可以每個執行緒消費多個partition。但負載高的情況下，Consumer 執行緒數最好和Partition數量保持一致。如果還是消費不過來，應該再開 Consumer 程序，程序內執行緒數同樣和分割槽數一致。

 消費訊息時，kafka client需指定topic以及partition number（每個partition對應一個邏輯日誌流，如topic代表某個產品線，partition代表產品線的日誌按天切分的結果），consumer client訂閱後，就可迭代讀取訊息，如果沒有訊息，consumer client會阻塞直到有新的訊息釋出。consumer可以累積確認接收到的訊息，當其確認了某個offset的訊息，意味著之前的訊息也都已成功接收到，此時broker會更新zookeeper上地offset registry。

 **5.3  高效的資料傳輸**

 1.  釋出者每次可釋出多條訊息（將訊息加到一個訊息集合中釋出）， consumer每次迭代消費一條訊息。

 2.  不建立單獨的cache，使用系統的page cache。釋出者順序釋出，訂閱者通常比釋出者滯後一點點，直接使用linux的page cache效果也比較後，同時減少了cache管理及垃圾收集的開銷。

 3.  使用sendfile優化網路傳輸，減少一次記憶體拷貝。

## 6.Kafka 與 Zookeeper

 **6.1 Zookeeper 協調控制**

 1. 管理broker與consumer的動態加入與離開。\(Producer不需要管理，隨便一臺計算機都可以作為Producer向Kakfa Broker發訊息\)

 2. 觸發負載均衡，當broker或consumer加入或離開時會觸發負載均衡演算法，使得一 個consumer group內的多個consumer的消費負載平衡。（因為一個comsumer消費一個或多個partition，一個partition只能被一個consumer消費）

 3.  維護消費關係及每個partition的消費資訊。

 **6.2 Zookeeper上的細節：**

 1. 每個broker啟動後會在zookeeper上註冊一個臨時的broker registry，包含broker的ip地址和埠號，所儲存的topics和partitions資訊。

 2. 每個consumer啟動後會在zookeeper上註冊一個臨時的consumer registry：包含consumer所屬的consumer group以及訂閱的topics。

 3. 每個consumer group關聯一個臨時的owner registry和一個持久的offset registry。對於被訂閱的每個partition包含一個owner registry，內容為訂閱這個partition的consumer id；同時包含一個offset registry，內容為上一次訂閱的offset。

