---
description: ELK 登入設定
---

# Kibana 使用者登入套件設定

## ELK 登入設定

ELK版本 7.1以上

> 前情提要:這篇是延續[教學文章](https://medium.com/@d101201007/centos7-elk-filebeat-%E6%8C%87%E4%BB%A4%E5%AE%89%E8%A3%9D-%E7%85%A7%E8%91%97%E8%B2%BC%E4%B8%8A%E5%B0%B1%E5%B0%8D%E4%BA%86-73f456381491)的設置，elasticsearch 用了身分elastic，安裝路徑在/usr/local/下面為範例，如果不能執行確認一下路徑是否正確，或可以留言給我。
>
> #### 諾是叢集架構需要修改所有Elasticsearch 節點設定檔再一一啟動。

### **1.產生憑證 elastict**

```text
//elasticsearch 為使用者
su elasticsearch -c "/usr/local/elasticsearch/bin/elasticsearch-certutil cert"
//輸入使用者密碼
Password: 
```

![](https://miro.medium.com/max/888/1*pwOACP-LNGZybz_7bzRLpw.png)

輸入 `config/elastic-certificates.p12` ，然後Enter跳過密碼

### **2.修改設定檔**

```text
# vim /usr/local/elasticsearch/config/elasticsearch.yml
//加入
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

重啟Elastic

```text
# kill -9 $(ps aux | grep elastic | awk '{print $2}')
# usr/local/elasticsearch/bin/elasticsearch -d
```

### **3.設定預設密碼**

```text
#/usr/local/elasticsearch/bin/elasticsearch-setup-passwords interactive
--> y
--> y
--> 然後輸入想要的密碼
```

### **4.Kibana設定**

```text
# vim /usr/local/kibana/config/kibana.yml 
//加入以下
xpack.security.enabled: true
elasticsearch.username: "kibana" # 7.9.2版:"kibana_system"
elasticsearch.password: "密碼"
```

### 5.啟動kibana，就會看到登入畫面。 Root 權限使用 Elastic:剛設定的密碼 登入

```text
# /usr/local/kibana/bin/kibana --allow-root
```

![](https://miro.medium.com/max/60/1*BzA3VJ03sq-I7luLbPX75w.png?q=20)![](https://miro.medium.com/max/2289/1*BzA3VJ03sq-I7luLbPX75w.png)

### 完成 ! <a id="b953"></a>

下面給有用Logstash的看:

* **Kibana 創建role/user**

![](https://miro.medium.com/max/454/1*KnBxAw7gBTsIgtg3KxgaTg.png)

左下角設定

創建Create User : 名稱:logstash\_internal 密碼:XXX Roles:superuser

如果superuser權限太大 想改其他的可以自建roles，logstash Index privileges權限選all、 Indices:\*就可以。

* **Logstash修改**

```text
# vim /home/tool/elk-common/logstash/logstash.conf
```

解開User and password 註解並輸入帳號密碼注意需要有" "

```text
input {
  beats {
    port => 5044
  }
}
output {
  elasticsearch {
    hosts => ["http://10.140.0.6:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    #帳號密碼需要加上""
    user => "elastic"
    password => "P@ssw0rd@Data!"
  }
}
```

_如果資料沒進來，看看Logstash有沒有報錯、重啟logstash_

大神實作影片:

\[7.1以上xpack設定登入\]\([https://www.youtube.com/watch?v=nMh1HWWe6B4&feature=youtu.be](https://www.youtube.com/watch?v=nMh1HWWe6B4&feature=youtu.be)\)

```text
如果有讓你看完這篇文，可以幫我拍手 1–10 下
如果覺得這文章對你有幫助，可以幫我拍手 10–30 下
如果覺得想看到更多關於學習筆記的文章，可以幫我拍手 30–50 下
讓我知道也記得 Follow我 DerekWu
更歡迎你在下方留言，我很樂意與你討論聊天或回答問題！
```



1. ES Security









## 動手實作 EP15：阻擋未經授權的存取

這一部分的實作，我們將可以學到：

* 啟動 ES 的安全性機制，設置密碼給使用者

1. 首先先用 ssh 連接到練習用的伺服器 server1，接著啟動 ES：

```text
ssh server1
./elasticsearch/bin/elasticsearch
```

1. 接著開啟另一個 terminal，連接到 server1，啟動 Kibana：

```text
ssh server1
./kibana/bin/kibana
```

1. 在瀏覽器開啟：**`<Public_DNS>/app/kibana#/`**，會發現可以直接存取 Kibana，並**不需要輸入**什麼帳號密碼，經過授權才能開啟，接下來我們要針對這點做改進！

![](https://i.imgur.com/bMEPOwN.png)

1. 關到一開始在 server1 上啟動的 ES 和 Kibana，接著編輯 ES 的設定檔（**`elasticsearch/config/elasticsearch.yml`**），加入下面一行讓安全性的功能啟動：

```text
xpack.security.enabled: true
```

![](https://i.imgur.com/QdAa7h2.png)

1. 再次啟動 ES，這時候如果隨意對它做存取（例：curl），你會得到下面的錯誤訊息：

```text
# curl ES server
curl 'server1:9200/_cat/nodes?pretty'
```

![](https://i.imgur.com/8xhsYRh.png)

1. 開啟安全性功能之後，為了要可以存取 ES cluster，我們就必須設置使用者帳號與密碼，在另一個連接到 server1 的 terminal，使用下列指令開始設置內建使用者的密碼：

```text
./elasticsearch/bin/elasticsearch-setup-passwords interactive
```

![](https://i.imgur.com/0EkBCBF.png)

1. 設置好之後，這時候使用已經有的帳號去 curl ES server，輸入對應密碼後，就可以得到對應的回應：

```text
# user=elastic
curl -u elastic 'server1:9200/_cat/nodes?pretty'
```

![](https://i.imgur.com/GOn8dGw.png)

1. 接著重新啟動 Kibana，你會發現這時候會跳出錯誤訊息，為什麼呢？因為這時候 Kibana 的連線，並沒有被 ES cluster 授予存取的權限，當我們一起動安全性功能之後，所有連接到 ES server 都需要經過授權。

![](https://i.imgur.com/7jZPfn2.png)

1. 那該怎麼辦呢？別緊張，只要設置一下可以存取的帳號資訊進 Kibana 就可以啦！下面是設置 Keystore 的指令：

```text
# 創建一個 Kibana keystore 來儲存帳號資訊
./kibana/bin/kibana-keystore create

# 新增使用者帳號
./kibana/bin/kibana-keystore add elasticsearch.username

# 新增使用者密碼
./kibana/bin/kibana-keystore add elasticsearch.password
```

![](https://i.imgur.com/fbHX0Tf.png)

1. 設置好之後，重新啟動 Kibana，並從瀏覽器連接，你就會看到要你輸入帳號密碼的頁面啦！

![](https://i.imgur.com/CVJdwWx.png)

1. 輸入完成功登入後，眼尖的你應該會發現跟最一開始打開 Kibana 的頁面有點不同！沒錯，就是左下角的地方多了一塊 **`Security`** 的區塊啦～

![](https://i.imgur.com/A7clzd7.png)

1. 這時候點到 **`Users`** 或 **`Roles`**，就可以看到一串內建使用者/角色的設置：

![](https://i.imgur.com/MYhTPWn.png)

![](https://i.imgur.com/UhZYrF2.png)

1. 剛才我們是透過 terminal 端下指令操作的方式，下面我們來使用 Security API 來進行安全性帳號的設置看看！先從左邊的面版點選 **`Dev Tools`**：

![](https://i.imgur.com/ESsGqcN.png)

1. 將一些 sample data 放進 **`sales_record`** 的索引：

![](https://i.imgur.com/xPis5c6.png)

1. 使用 Security API，先創建一個 read\_only\_sales 的角色，這個角色只能做讀取的動作：

```text
PUT /_security/role/read_only_sales
{
  "cluster": [],
  "indices": [
    {
      "names": [ "sales_record" ],
      "privileges": ["read", "view_index_metadata"]
    }
  ]
}
```

![](https://i.imgur.com/uKJocFL.png)

1. 接著再創建 **`sales_user`** 這個使用者，並賦予對應的角色：**`read_only_sales`** 和 **`kibana_user`**（要記得加入 **`kibana_user`** 這個角色，否則該使用者將沒辦法登入 Kibana 歐！）

```text
POST /_security/user/sales_user
{
  "password" : "xxxxxxxx",
  "roles" : [ "read_only_sales", "kibana_user" ],
  "full_name" : "Sales User",
  "email" : "training@elastic.co"
}
```

![](https://i.imgur.com/3o5TaJw.png)

1. 設置完後當然要來測試看看啦，先點一下右上方的帳號圖像，原本應該是 **`elastic`** 使用者，登出後重新登入 **`sales_user`** 使用者：

Before  
 ![](https://i.imgur.com/Qnh6d34.png)

After  
 ![](https://i.imgur.com/EKuzgZX.png)

1. 到 **`Dev Tools`** 中，先用 **`GET`** 來讀取看看，應該要可以拿到東西：

```text
GET sales_record/_search
```

![](https://i.imgur.com/XGukBDL.png)

1. 但是若是要修改的話，逼逼！母湯歐～

```text
POST sales_record/_doc
{
  "product": "4008",
  "price": 8.69,
  "payment_type": "mastercard",
  "card_number": "9378906409894724",
  "name": "jack",
  "city": "paris",
  "country": "france"
}
```

![](https://i.imgur.com/ZVnjFnO.png)

### 今日心得與短結

今天實作了 ES 上安全性的功能，可以看到設置其實還蠻容易的！更進一步的內容，有興趣的同學也可以參考官方的學習資訊：[https://www.elastic.co/training/fundamentals-of-securing-elasticsearch](https://www.elastic.co/training/fundamentals-of-securing-elasticsearch)

明天開始就要到下一個主題：Elastic SIEM 基礎，這個我也不知道是幹麼的碗糕了！

