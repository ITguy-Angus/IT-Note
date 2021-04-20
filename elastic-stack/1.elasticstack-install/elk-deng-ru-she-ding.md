---
description: ELK 登入設定
---

# ELK 登入設定

## ELK 登入設定

ELK版本 7.1以上

> 前情提要:這篇是延續[教學文章](https://medium.com/@d101201007/centos7-elk-filebeat-%E6%8C%87%E4%BB%A4%E5%AE%89%E8%A3%9D-%E7%85%A7%E8%91%97%E8%B2%BC%E4%B8%8A%E5%B0%B1%E5%B0%8D%E4%BA%86-73f456381491)的設置，elasticsearch 用了身分elastic，安裝路徑在/usr/local/下面為範例，如果不能執行確認一下路徑是否正確，或可以留言給我。

### **1.產生憑證 elastict**

```text
//elasticsearch 為使用者
su elasticsearch -c "/usr/local/elasticsearch/bin/elasticsearch-certutil cert"
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
# nohup su elastic -c "/usr/local/elasticsearch/bin/elasticsearch &" > /dev/null
```

### **3.設定預設密碼**

```text
# /usr/local/elasticsearch/bin/elasticsearch-setup-passwords interactive
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

加入

```text
output {
  elasticsearch {
    ...
    user => elastic
    password => 密碼
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

