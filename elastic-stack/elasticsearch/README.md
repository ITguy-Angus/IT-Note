---
description: 分散式架構原理
---

# Elasticsearch

## es分散式架構原理 <a id="outline__1"></a>

elasticsearch設計的理念就是分散式搜尋引擎，底層實現還是基於Lucene的，核心思想是在多型機器上啟動多個es程序例項，組成一個es叢集。一下是es的幾個概念：

接近實時  
es是一個接近實時的搜尋平臺，這就意味著，從索引一個文件直到文件能夠被搜尋到有一個輕微的延遲  
 叢集（cluster）  
一個叢集有多個節點（伺服器）組成，通過所有的節點一起儲存你的全部資料並且通過聯合索引和搜尋功能的節點的集合，每一個叢集有一個唯一的名稱標識  
 節點（node）  
一個節點就是一個單一的伺服器，是你的叢集的一部分，儲存資料，並且參與叢集和搜尋功能，一個節點可以通過配置特定的名稱來加入特定的叢集，在一個叢集中，你想啟動多少個節點就可以啟動多少個節點。  
 索引（index）  
一個索引就是還有某些共有特性的文件的集合，一個索引被一個名稱唯一標識，並且這個名稱被用於索引通過文件去執行搜尋，更新和刪除操作。  
 型別（type）  
type 在6.0.0已經不贊成使用  
 文件（document）  
一個文件是一個基本的搜尋單元

總結：  
es中，儲存資料的基本單位就是索引，比如說es中儲存了一些訂單系統的銷售資料，就因該在es中建立一個索引，order—index，所有的銷售資料就會都寫到這個索引裡面去，一個索引就像資料庫。而type就相當於每一張表，  
一個index裡面可以有多個type，而mapping就相當於表的結構定義，定義了什麼欄位型別等，你往index的一個type裡新增一行資料就叫做一個document，每一個document有多個filed，每一個filed就代表這個document的一個欄位的值。

分片（shards）

在一個搜尋裡儲存的資料，潛在的情況下可能會超過單個節點的硬體的儲存限制，為了解決這個問題，elasticsearch便提供了分片的功能，它可以將索引劃分為多個分片，當你建立一個索引的時候，你就可以簡單的定義你想要的分片的數量，每一個分片本身是一個全功能的完全獨立的索引，可以部署到叢集中的任何一個節點。分片的兩個總要原因：  
（1）它允許你水平切分你的內容卷  
（2）它允許通過分片來分佈和並執行操作來應對日益增長的執行量

複製（replica）  
在一個網路情況下，故障可能會隨時發生，有一個故障恢復機制是必須的，為了達到這個目的，ES允許你製作一個或多個拷貝放入一個叫做複製分片或短暫的複製品中。複製對於以下兩個主要原因很重要  
（1）高可用。它提供了高可用的以來防止分片或者節點宕機，為此，一個非常重要的注意點就是絕對不要講一個分片的拷貝放在跟這個分片相同的機器上。  
（2）高併發。它允許你的分片可以提供超出自身吞吐量的搜尋服務，搜尋行為可以在分片所有的拷貝中並行執行。  
總之，一個完整的流程就是，ES客戶端將一份資料寫入primary shard,它會將分成成對的shard分片，並將資料進行復制，ES客戶端取資料的時候就會在replica或primary 的shard中去讀。ES叢集有多個節點，會自動選舉一個節點為master節點，這個master節點其實就是幹一些管理類的操作，比如維護後設資料，負責切換primary shard 和replica shard的身份之類的，要是master節點宕機了，那麼就會重新選舉下一個節點為master為節點。如果時非master宕機了，那麼就會有master節點，讓那個宕機的節點上的primary shard的身份轉移到replica shard上，如果修復了宕機的那臺機器，重啟之後，master節點就會控制將缺失的replica shard 分配過去，同步後續的修改工作，讓叢集恢復正常。

## es寫入資料的過程 <a id="outline__2"></a>

客戶端選擇一個node傳送請求過去，這個node就是coordinating node \(協調節點\)  
 coordinating node，對document進行路由，將請求轉發給對應的node  
 實際上的node上的primary shard處理請求，然後將資料同步到replica node  
 coordinating node，如果發現primary node和所有的replica node都搞定之後，就會返回請求到客戶端

## es讀資料過程 <a id="outline__3"></a>

查詢，GET某一條的資料，寫入某個document，這個document會自動給你分配一個全域性的唯一ID，同時跟住這個ID進行hash路由到對應的primary shard上面去，當然也可以手動的設定ID

客戶端傳送任何一個請求到任意一個node，成為coordinate node  
 coordinate node 對document進行路由，將請求轉發到對應的node，此時會使用round-robin隨機輪訓演算法，在primary shard 以及所有的replica中隨機選擇一個，讓讀請求負載均衡，  
 接受請求的node，返回document給coordinate note  
 coordinate node返回給客戶端

## es搜尋資料過程 <a id="outline__4"></a>

客戶端傳送一個請求給coordinate node  
 協調節點將搜尋的請求轉發給所有的shard對應的primary shard 或replica shard  
 query phase：每一個shard 將自己搜尋的結果（其實也就是一些唯一標識），返回給協調節點，有協調節點進行資料的合併，排序，分頁等操作，產出最後的結果  
 fetch phase ，接著由協調節點，根據唯一標識去各個節點進行拉去資料，最總返回給客戶端

## 寫入資料的底層原理 <a id="outline__5"></a>

資料先寫入到buffer裡面，在buffer裡面的資料時搜尋不到的，同時將資料寫入到translog日誌檔案之中  
 如果buffer快滿了，或是一段時間之後，就會將buffer資料refresh到一個新的OS cache之中，然後每隔1秒，就會將OS cache的資料寫入到segment file之中，但是如果每一秒鐘沒有新的資料到buffer之中，就會建立一個新的空的segment file，只要buffer中的資料被refresh到OS cache之中，就代表這個資料可以被搜尋到了。當然可以通過restful api 和Java api，手動的執行一次refresh操作，就是手動的將buffer中的資料刷入到OS cache之中，讓資料立馬搜尋到，只要資料被輸入到OS cache之中，buffer的內容就會被清空了。同時進行的是，資料到shard之後，就會將資料寫入到translog之中，每隔5秒將translog之中的資料持久化到磁碟之中  
 重複以上的操作，每次一條資料寫入buffer，同時會寫入一條日誌到translog日誌檔案之中去，這個translog檔案會不斷的變大，當達到一定的程度之後，就會觸發commit操作。  
 將一個commit point寫入到磁碟檔案，裡面標識著這個commit point 對應的所有segment file  
 強行將OS cache 之中的資料都fsync到磁碟檔案中去。  
 解釋：translog的作用：在執行commit之前，所有的而資料都是停留在buffer或OS cache之中，無論buffer或OS cache都是記憶體，一旦這臺機器死了，記憶體的資料就會丟失，所以需要將資料對應的操作寫入一個專門的日誌問價之中，一旦機器出現宕機，再次重啟的時候，es會主動的讀取translog之中的日誌檔案的資料，恢復到記憶體buffer和OS cache之中。  
 將現有的translog檔案進行清空，然後在重新啟動一個translog，此時commit就算是成功了，預設的是每隔30分鐘進行一次commit，但是如果translog的檔案過大，也會觸發commit，整個commit過程就叫做一個flush操作，我們也可以通過ES API,手動執行flush操作，手動將OS cache 的資料fsync到磁碟上面去，記錄一個commit point，清空translog檔案  
 補充：其實translog的資料也是先寫入到OS cache之中的，預設每隔5秒之中將資料重新整理到硬碟中去，也就是說，可能有5秒的資料僅僅停留在buffer或者translog檔案的OS cache中，如果此時機器掛了，會丟失5秒的資料，但是這樣的效能比較好，我們也可以將每次的操作都必須是直接fsync到磁碟，但是效能會比較差。  
 如果時刪除操作，commit的時候會產生一個.del檔案，裡面講某個doc標記為delete狀態，那麼搜尋的時候，會根據.del檔案的狀態，就知道那個檔案被刪除了。  
 如果時更新操作，就是講原來的doc標識為delete狀態，然後重新寫入一條資料即可。  
 buffer每次更新一次，就會產生一個segment file 檔案，所以在預設情況之下，就會產生很多的segment file 檔案，將會定期執行merge操作  
 每次merge的時候，就會將多個segment file 檔案進行合併為一個，同時將標記為delete的檔案進行刪除，然後將新的segment file 檔案寫入到磁碟，這裡會寫一個commit point，標識所有的新的segment file，然後開啟新的segment file供搜尋使用。

總之，segment的四個核心概念，refresh，flush，translog、merge

## 搜尋的底層原理 <a id="outline__6"></a>

查詢過程大體上分為查詢和取回這兩個階段，廣播查詢請求到所有相關分片，並將它們的響應整合成全域性排序後的結果集合，這個結果集合會返回給客戶端。

查詢階段

當一個節點接收到一個搜尋請求，這這個節點就會變成協調節點，第一步就是將廣播請求到搜尋的每一個節點的分片拷貝，查詢請求可以被某一個主分片或某一個副分片處理，協調節點將在之後的請求中輪訓所有的分片拷貝來分攤負載。  
 每一個分片將會在本地構建一個優先順序佇列，如果客戶端要求返回結果排序中從from 名開始的數量為size的結果集，每一個節點都會產生一個from size大小的結果集，因此優先順序佇列的大小也就是from size，分片僅僅是返回一個輕量級的結果給協調節點，包括結果級中的每一個文件的ID和進行排序所需要的資訊。  
 協調節點將會將所有的結果進行彙總，並進行全域性排序，最總得到排序結果。

取值階段

查詢過程得到的排序結果，標記處哪些文件是符合要求的，此時仍然需要獲取這些文件返回給客戶端  
 協調節點會確定實際需要的返回的文件，並向含有該文件的分片傳送get請求，分片獲取的文件返回給協調節點，協調節點將結果返回給客戶端。

## 倒排索引 <a id="outline__7"></a>

倒排索引就建立分詞與文件之間的對映關係，在倒排索引之中，資料時面向分詞的而不是面向文件的。

## 在海量資料中怎樣提高效率 <a id="outline__8"></a>

filesystem cache  
 ES的搜尋引擎是嚴重的依賴底層的filesystem cache，如果給filesystem cache更多的記憶體，儘量讓記憶體可以容納所有的index segment file 索引資料檔案  
 資料預熱  
 對於那些你覺得比較熱的資料，經常會有人訪問的資料，最好做一個專門的快取預熱子系統，就是對熱資料，每隔一段時間，你就提前訪問以下，讓資料進入filesystem cache裡面去，這樣期待下次訪問的時候，效能會更好一些。  
 冷熱分離

關於ES的效能優化，資料拆分，將大量的搜尋不到的欄位，拆分到別的儲存中去，這個類似於MySQL的分庫分表的垂直才分。

document的模型設計

不要在搜尋的時候去執行各種複雜的操作，儘量在document模型設計的時候，寫入的時候就完成了，另外對於一些複雜的操作，儘量要避免

分頁效能優化

翻頁的時候，翻得越深，每個shard返回的資料越多，而且協調節點處理的時間越長，當然是用scroll，scroll會一次性的生成所有資料的一個快照，然後每次翻頁都是通過移動遊標完成的。 api 只是在一頁一頁的往後翻

