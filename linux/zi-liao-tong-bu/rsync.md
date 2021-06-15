# Rsync

## 安裝

一、 Rsync介紹  
 rsync是Unix下的一款應用軟體，它能同步更新兩處電腦的檔案與目錄，並適當利用差分編碼以減少資料傳輸。rsync中一項與其他大部份類似程式或協定中所未見的重要特性是映像對每個目標只需要一次傳送。rsync可複製／顯示目錄內容，以及複製檔案，並可選擇性的壓縮以及遞歸複製。  
 在常駐模式（daemon mode）下，rsync預設監聽TCP埠873，以原生rsync傳輸協定或者透過遠端shell如RSH或者SSH伺服檔案。SSH情況下，rsync用戶端執行程式必須同時在本機和遠端機器上安裝。  
 rsync是自由軟體，以GNU通用公共許可證發行。

Rsync在備份的部份  
 主要是傳送資料差異的部份，因此大多都被使用在差異備份上。除了資料第一次傳輸是整份檔案外，之後都只會傳送資料間異動的部份。  
 備份相關名詞 :  
 鏡像備份\(mirror backup\) : 如同鏡子一般，把資料完整的做一份備份。  
 增量備份\(incremental backup\) : 只備份從上次備份後，資料有異動的部份。  
 二、 Rsync設定  
 這次在rsync設定部份，直接使用範例的方式會比較快了解。

範例 :  
 把A主機的網頁資料\( /var/www/html \)，定期備份到B主機的\( /home/backup/\)  
 A主機\( Server端 \) IP :192.168.1.222  
 B主機\( Clinet端 \) IP :192.168.1.122

在開始設定之前，有兩個檔案要說明一下:  
 /etc/rsyncd.conf 為rsync設定檔  
 /etc/rsyncd.secrets 為rsync 密碼檔

1. 先設定B主機  
    1.1 在B主機上查詢是否有安裝rsync

   \[root@localhost ~\]\# rpm -qa \| grep 'rsync'  
    rsync-2.6.8-3.1  
    \#如果沒有安裝的話 請自行補上  
    \[root@localhost ~\]\# yum -y install rsync

1.1 修改 /etc/xinetd.d/rsync 設定  
 在這邊所設定的rsync是用xinetd來做啟動的。所以如果在rsync並無特別設定的話，  
 則會依照xinetd.conf所設定的執行。

```text
[root@localhost ~]# vim /etc/xinetd.d/rsync 
service rsync
{
        disable = yes  改為 no 
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon
        log_on_failure  += USERID
```

1.2修改rsync.conf 設定檔 \(如果沒有此檔的話請自行建立\)  
 以下所使用到的設定參數請依照需求選取設定。

```text
[root@localhost ~]# vim /etc/rsyncd.conf
###backup config###                      //註解
 [web]                                   //module name 
           comment = backup dmz web      //說明
           hosts allow = 192.168.1.222   //允許使用rsync連入的ip
           hosts deny = *                //不允許連入的ip，*表示全檔
           path = /home/backup/          //存放備份資料的目錄
           auth users = root    //認證帳號 (要設定於rsyncd.secrets 內的帳號)
           uid = root                    //用來啟動rsync server 的uid
           gid = root                    //用來啟動rsync server 的gid 
           secrets file = /etc/rsyncd.secrets      //密碼檔存放路徑
           read only = no                          //是否設定為唯讀
     dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz
                                          //不要對這些附檔名的檔案做壓縮
```

1.3設定連線的帳號密碼

```text
[root@localhost ~]# vim /etc/rsyncd.secrets
root:12345  (設定格式:  帳號:密碼)
修改rsyncd.secrets使用權限 (很重要)
[root@localhost ~]# chmod 600 /etc/rsyncd.secrets
[root@localhost ~]# chown root.root  /etc/rsyncd.secrets
```

1.4重新啟動 xinetd，不是啟動 rsync 。  
 這邊應該會有人有所疑問，為什麼是重啟xinetd而不是rsync。  
 在2.1步驟時，是使用xinetd來啟動rsync。因此在這邊是重啟xinetd而不是rsync。

\[root@localhost ~\]\# /etc/init.d/xinetd restart

1.5 設定開機自動啟動xinetd  
 基本上xinetd，已經有預設開機會啟動，如果沒有話請自行手動設定

```text
[root@localhost ~]# chkconfig xinetd on              #開機自動啟動
[root@localhost ~]# /etc/init.d/xinetd  restart      #手動啟動
```

1.6 測試看看是否都正常啟用 \(rsync的port為873\)

```text
[root@localhost ~]#  netstat -tnlp | grep 873
tcp    0    0 0.0.0.0:873        0.0.0.0:*         LISTEN      4235/xinetd

[root@localhost ~]# telnet localhost 873  
#若出現下列訊息表示正常
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
@RSYNCD: 29
```

1. 設定A主機  
    2.1 在A主機上查詢是否有安裝rsync

   \[root@localhost ~\]\# rpm -qa \| grep 'rsync'  
    rsync-2.6.8-3.1  
    \#如果沒有安裝的話 請自行補上  
    \[root@localhost ~\]\# yum -y install rsync

2.2 設定登入B主機的密碼  
 在A主機上只需要設定密碼就可以。

```text
[root@localhost ~]# vim /etc/rsyncd.secrets
12345  (設定格式:  密碼)
#修改rsyncd.secrets使用權限 (很重要)
[root@localhost ~]# chmod 600 /etc/rsyncd.secrets
[root@localhost ~]# chown root.root  /etc/rsyncd.secrets
```

2.3 測試rsync是否可傳輸資料

```text
[root@localhost ~]# /usr/bin/rsync -avrHS --delete --password-file=/etc/rsyncd.secrets  /opt root@192.168.1.122::web 
#或是
[root@localhost ~]#/usr/bin/rsync -rvlHpogDtS --delete --password-file=/etc/rsyncd.secrets  /opt/ root@192.168.1.122::web

#rsync結束後會看到下列訊息
sent 19380160 bytes  received 1062 bytes  1685323.65 bytes/sec
total size is 19374133  speedup is 1.00
```

3設定排程  
 如果剛才的同步指令沒有問題的話。就可以開始設定排程了  
 3.1先自行寫一個rsync 的script 用來設定排程

```text
[root@localhost ~]# vim /root/bin/rsync.sh   #存放在/root/bin/
#!/bin/sh
/usr/bin/rsync -avrHS --delete --password-file=/etc/rsyncd.secrets  /opt root@192.168.1.122::web
```

3.2設定排程  


## 參數

* -v，-verbose增強可讀性
* -q，-quiet忽略非錯誤信息
* -no-motd忽略守護進程模式的MOTD信息（參見manpage告知）
* -c，-checksum基於checksum校驗，而非mod-time和size
* -a，-archive歸檔模式，和-rlptgoD（不要使用-H，-A，-X）參數相同
* -no-OPTION關閉一些顯式的OPTION（比如-no-D）
* -r，-recursive遞歸目錄
* -R，-relative使用相對路徑
* -no-implied-dirs不發送制定目錄的屬性，避免在目標使用-relative刪除連接重新傳輸文件
* -b，-backup備份（查看-suffix＆-backup-dir）
* -backup-dir = DIR基於DIR來創建備份的目錄結構
* -suffix = SUFFIX制定備份的後最（默認~w / o -backup-dir）
* -u，-update跳過接受端較新的文件
* -inplace直接在文件上更新（SEE MAN PAGE）
* -append直接追加文件
* -append-verify和-append很像，只是用文件的較舊的那部份做校驗和
* -d，-dirs僅傳輸文件而非遞歸
* -l，-links將軟鏈接當作軟鏈接來拷貝
* -L，-copy-links拷貝鏈接對應的文件或者目錄而非鏈接本身
* -copy-unsafe-links僅不安全的鏈接才被拷貝
* -safe-links忽略那些鏈接到目錄樹外的鏈接
* -k，-copy-dirlinks將鏈接翻譯成其鏈接的目錄
* -K，-keep-dirlinks將目錄鏈接在接收端轉換成目錄
* -H，-hard-links保留硬鏈接
* -p，-perms保留權限
* -E，-executability保留文件的可執行性
* -chmod = CHMOD改變文件或者目錄的權限
* -A，-acls保留ACL（暗示-perms）
* -X，-xattrs保留擴展屬性
* -o，-owner保留所有者（僅限超級用戶）
* -g，-group保留組
* -devices保留設備文件（僅限超級用戶）
* -specials保留特殊文件
* -D和-devices -specials一樣
* -t，-times保留修改時間
* -O，-omit-dir-times在-times選項裡忽略目錄的時間
* -super允許接受方執行那個一些超級用戶的活動
* -fake-super通過xattrs來存儲和回復權限
* -S，-sparse高效處理稀疏文件
* -n，-dry-run只運行，不做改變
* -W，-whole-file拷貝貝整個文件（沒有delta-xfer算法）
* -x，-one-file-system不跨越文件系統
* -B，-block-size = SIZE強制指定checksum的塊大小
* -e，-rsh = COMMAND指定要使用的遠程shell
* -rsync-path = PROGRAM指定遠程要運行的rsync路徑
* - 存在如果接收端不存在文件就不創建
* -ignore-existing如果接收端存在就不更新文件
* -remove-source-files發送端刪除已經同步的文件（非dirs）
* -del -delete-during的別名
* -delete刪除接收端上在發送端不存在的文件
* -delete-before接收端在發送前刪除，而不是發送過程中
* -delete-during接收端在發送過程中刪除
* -delete-delay在發送過程中尋找文件，在發送完成後刪除
* -delete-after接收端在發送完成後刪除
* -delete-excluded在接收端刪除排除的文件
* -ignore-errors即使有I / O錯誤也刪除
* -force即使目錄不是空的也刪除
* -max-delete = NUM​​最多刪除文件的數目
* -max-size = SIZE最大傳輸文件的大小
* -min-size = SIZE最小傳輸文件的大小
* -partial保持部分傳輸的文件
* -partial-dir = DIR制定部分傳輸文件的存放目錄
* -delay-updates先傳輸，最後再更新，保持原子性
* -m，-prune-empty-dirs接收端刪除空目錄
* -numeric-ids接收端不要將uid / gid映射為用戶名和組名
* -timeout = SECONDS設置I / O超時時間，s為單位
* -contimeout = SECONDS設置鏈接服務端的時間
* -I，-ignore-times即使mtime和size都相同也不跳過
* -size-only只要大小相同就跳過
* -modify-window = NUM​​比對時間時制定精確範圍，範圍內都認為時間相同
* -T，-temp-dir = DIR指定創建臨時文件的目錄
* -y，-fuzzy文件在接收端不存在的情況下，在當前目錄下尋找一個基礎文件，以加快傳輸
* -compare-dest = DIR接收端除了和發送端對比還和這裡指定的目錄對比，適合備份上次備份改變的文件
* -copy-dest = DIR和-compare-dest類似，只是接收端會用本地拷貝來複製那些未改變的文件
* -link-dest = DIR和-compare-dest類似，只是接收端會建立那些未改變文件的硬鏈接
* -z，-compress傳輸過程中壓縮
* -compress-level = NUM​​指定壓縮等級
* -skip-compress = LIST不壓縮指定後綴的文件
* -C，-cvs-exclude以CSV的方式自動忽略文件
* -f，-filter = RULE新增一個文件過濾規則
* -F與-filter ='dir-merge /.rsync-filter'重複相同：-filter =' - .rsync-filter'
* -exclude = PATTERN排除規則PATTERN
* -exclude-from = FILE從文件中讀取排除規則
* -include = PATTERN不要排除指定規則的文件
* -include-from = FILE從文件中讀取包含的規則
* -files-from = FILE從文件中讀取文件列表
* -0，-from0 all
* -from / filter文件由0分隔
* -s，-protect-args參數不許要空格分割; 只有通配符特殊字符
* -address = ADDRESS綁定監聽的地址
* -port = PORT制定端口號
* -sockopts = OPTIONS制定TCP選項
* -blocking-io在遠程shell中使用阻塞I / O.
* -stats給出文件統計信息
* -8，-8-bit-output輸出時不對高位字符轉義
* -h，-human-readable以易於閱讀的方式打印數字
* -progress顯示傳輸進度
* -P與-partial-progress相同
* -i，-itemize-changes打印更新的總結信息
* -out-format = FORMAT以特定的格式打印更新信息
* -log-file = FILE日誌文件
* -log-file-format = FMT日誌文件格式
* -password-file = FILE密碼文件
* -list-only僅列出文件
* -bwlimit = KBPS限制帶寬; 每秒KBytes
* -write-batch = FILE將批量更新寫入文件
* -only-write-batch = FILE和-write-batch類似但沒有更新目的地
* -read-batch = FILE從文件中讀取批量更新任務
* -protocol = NUM​​使用舊版本的協議
* -iconv = CONVERT\_SPEC要求文件名字符轉義
* -4，-ipv4更喜歡IPv4
* -6，-ipv6更喜歡IPv6
* -version打印幫助信息
* （-h）-help打印這個幫組信息（-h僅在單獨使用時與-help同意）

## 常見處理方式 

1 在本地機器上對兩個目錄進行同步  
rsync -zvr /var/opt/installation/inventory/ /root/temp  
參數： -z 開啟壓縮 -v 詳情輸出 -r 表示遞歸 2 利用 rsync -a 讓同步時保留時間標記 rsync 選項 -a 稱為歸檔模式，執行以下操作 遞歸模式 保留符號鏈接 保留權限 保留時間標記 保留用戶名及組名 rsync -azv /var/opt/installation/inventory/ /root/temp/ 3 僅同步一個文件  
rsync -v /var/lib/rpm/Pubkeys /root/temp/ 4 從本地同步文件到遠程服務器 rsync -avz /root/temp/ root@192.168.200.10:/home/root/temp/  
就像你所看到的，需要在遠程目錄前加上 ssh 登錄方式，格式為 username@machinename:path _ADVERTISEMENT_ 5 同步遠程文件到本地 和上面差不多，做個相反的操作  
rsync -avz root@192.168.200.10:/var/lib/rpm /root/temp 同步遠程服務器上的文件夾到本地，如果本地有的文件，遠程沒有，則刪除之。  
rsync -rave "ssh -p 22 -l root" --delete 192.168.0.200:/www/web/ /www/web/ 6 同步時指定遠程 shell 用 -e 參數可以指定遠程 ssh ，比如用 rsync -e ssh 來指定為 ssh  
rsync -avz -e ssh root@192.168.200.10:/var/lib/rpm /root/temp 7 不要覆蓋被修改過的目的文件  
使用 rsync -u 選項可以排除被修改過的目的文件  
rsync -avzu root@192.168.200.10:/var/lib/rpm /root/temp _ADVERTISEMENT_ 8 僅僅同步目錄權（不同步文件）  
使用 -d 參數  
rsync -v -d root@192.168.200.10:/var/lib/ 9 查看每個文件的傳輸進程  
使用 – -progress 參數  
rsync -avz –-progress root@192.168.200.10:/var/lib/rpm/ /root/temp/ 10 刪除在目的文件夾中創建的文件  
用 – -delete 參數  
rsync -avz – -delete root@192.168.200.10:/var/lib/rpm/ 11 不要在目的文件夾中創建新文件  
有時能只想同步目的地中存在的文件，而排除源文件中新建的文件，可以使用 – -exiting 參數  
rsync -avz –existing root@192.168.1.2:/var/lib/rpm/ 12 查看源和目的文件之間的改變情況 用 -i 參數  
rsync -avzi root@192.168.200.10:/var/lib/rpm/ /root/temp/  
輸出結果中在每個文件最前面會多顯示 9 個字母，分別表示為  
&gt; 已經傳輸  
f 表示這是一個文件  
d 表示這是一個目錄  
s 表示尺寸被更改  
t 時間標記有變化  
o 用戶被更改  
g 用戶組被更改 13 在傳輸時啟用包含和排除模式  
rsync -avz – -include ‘P\*’ – -exclude ‘\*’ root@192.168.200.10:/var/lib/rpm/ /root/temp/  
14 不要傳輸大文件 使用 – – max-size 參數  
rsync -avz – -max-size=’100K’ root@192.168.200.10:/var/lib/rpm/ /root/temp/ 15 傳輸所有文件 不管有沒有改變，再次把所有文件都傳輸一遍，用 -W 參數  
rsync -avzW root@192.168.200.10:/var/lib/rpm/ /root/temp



## Rsync 複製FTP 檔案

讓我們僅使用FTP `src`呼叫計算機。  
  
讓我們使用FTP和SSH `dst`呼叫計算機。  


```text
ssh dst
cd destination-direction
wget --mirror --ftp-user=username --ftp-password=password\
     --no-host-directories ftp://src/pathname/
```

  
請注意，在命令列上使用`wget`執行`--ftp-password`會將密碼洩露給系統上的其他任何人。 \(以及通過明線進行傳輸，但是您知道這一點。\)  
  
如果您無權訪問`wget`，那麼他們可能已安裝`ncftp`或`lftp`或`ftp`。我只是碰巧知道`wget`最好。 :\)  
  
**編輯**要使用`ftp`，您需要做更多的事情:  


```text
ftp src
user username
pass password
bin
cd /pathname
ls
```

  
此時，請注意遠端系統上的所有目錄。用`!mkdir`建立每個。然後在本地和遠端都轉到目錄:  


```text
lcd <dirname>
cd <dirname>
ls
```

  
對所有目錄重複此操作。使用`mget *`獲取所有檔案。  
  
如果這看起來很糟糕，那是因為。 FTP並非為此目的而設計的，並且如果您的新主機沒有更好的工具\(一定要尋找`ncftp`和`lftp`以及可能類似於`ftpmirror`的東西\)，那麼您可以自己編譯更好的工具，或者擅長於編寫糟糕的指令碼工具。 :\)  
  
或者，如果您可以在`src`上獲得 shell ，那也將極大地幫助您。 FTP並非僅用於傳輸數千個檔案。  
  
無論如何，這避免了在您的本地系統上反彈，這應該可以顯著提高吞吐量。

