# Find



Linux 的 Find 在限定日期方面很常使用到，在這邊把幾個參數紀錄一下~

*  find
  * -mtime 搜尋檔案的修改時間\(天\)
  * -mmin 搜尋檔案的修改時間\(分鐘\)
  * -ctime 搜尋檔案的建立時間\(天\)
  * -cmin 搜尋檔案的建立時間\(分鐘\)
  * -atime 搜尋檔案的最後開啟時間\(天\)
  * -amin 搜尋檔案的最後開啟時間\(分鐘\)

範例

* find ./ -mtime 3 \# 在當前目錄下搜尋3天時修改的檔案
* find ./ -mtime +3 \# 在當前目錄下搜尋3天前修改的檔案
* find ./ -mtime -3 \# 在當前目錄下搜尋3天內修改的檔案



檔案更新日期+執行複製到另外一個目錄指

```text
find /usr/local/a/. -mmin -15 -exec cp {} /usr/local/b \;
```





檔案副檔名+大小判斷+執行指令

```text
find . -type f -name *.mp3 -size +10M -exec rm {} \;
```

