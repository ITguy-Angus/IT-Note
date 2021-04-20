---
description: apt安裝
---

# 1.1 ELK Stack apt 安裝方式

## ELK Stack 7.12 apt安裝

安裝的順須分別是E&gt;K&gt;L&gt;B，這次介紹的內容為Ubuntu20.04本機最基礎的研究環境。

### 前置環境安裝

1.安裝前先將套件更新一下吧

`apt update`

2.更新完畢後可以查看目前套件的的版本

`apt list | grep -E "jdk|wget|nginx|apt-transport"`

3.安裝ELK Stack 7.12所需用到的環境與安裝過程使用到的套件

`apt-get install openjdk-11-jdk wget apt-transport-https nginx`

4.檢查前置套件是否安裝完成

`apt list --installed | grep -E "jdk|wget|nginx|apt-transport"`

```text
apt-transport-https/focal-updates,now 2.0.5 all [installed]
libnginx-mod-http-image-filter/focal-updates,now 1.18.0-0ubuntu1 amd64 [installed,automatic]
libnginx-mod-http-xslt-filter/focal-updates,now 1.18.0-0ubuntu1 amd64 [installed,automatic]
libnginx-mod-mail/focal-updates,now 1.18.0-0ubuntu1 amd64 [installed,automatic]
libnginx-mod-stream/focal-updates,now 1.18.0-0ubuntu1 amd64 [installed,automatic]
nginx-common/focal-updates,now 1.18.0-0ubuntu1 all [installed,automatic]
nginx-core/focal-updates,now 1.18.0-0ubuntu1 amd64 [installed,automatic]
nginx/focal-updates,now 1.18.0-0ubuntu1 all [installed]
openjdk-11-jre-headless/focal-updates,focal-security,now 11.0.10+9-0ubuntu1~20.04 amd64 [installed,automatic]
openjdk-11-jre/focal-updates,focal-security,now 11.0.10+9-0ubuntu1~20.04 amd64 [installed]
wget/focal,now 1.20.3-1ubuntu1 amd64 [installed]
```



5.添加elastic彈性伸縮儲存庫

   1.首先導入其GPG密鑰

`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add`

   2.在以下位置創建文件/etc/apt/sources.list.d/elastic.list並在內文中添下列內容

`vim /etc/apt/sources.list.d/elastic.list`

```text
deb https://artifacts.elastic.co/packages/7.x/apt stable main
```

   3.更新儲存庫

`apt update`

### 安裝 Filebeat

`apt-get install filebeat`

`sudo chkconfig --add filebeat`

`systemctl start filebeat` 

`systemctl status filebeat`



