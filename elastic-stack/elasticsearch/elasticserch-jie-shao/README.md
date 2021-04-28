# Elasticserch 介紹

![](https://static001.geekbang.org/infoq/bb/bbbc78e45f794b4e51d68fae680c6c97.webp)

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

