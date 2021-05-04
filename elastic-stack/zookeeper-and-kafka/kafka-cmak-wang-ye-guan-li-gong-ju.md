# Kafka CMAK \(網頁管理工具\)

## 安裝條件

1. 支持 Kafka 0.8以上版本
2. Java 11+
3. zookeeper必须是3.5+版本。

## 安裝 JACA

```text
#安裝JAVA JDK
apt install openjdk-11-jdk
#驗證JAVA 是否安裝成功
java -version
```

## 安裝CMAK

### 下載安裝包

```text
$ wget https://github.com/yahoo/CMAK/releases/download/3.0.0.5/cmak-3.0.0.5.zip 
```

解压安装包，并进入解压后的目录。

```text
$ unzip cmak-3.0.0.5.zip
$ mv cmak-3.0.0.5 cmak
$ cd cmak
```

修改配置文件application.conf

```text
$ vim conf/application.conf 

#用上面的命令编辑打开文件，将下面的两个配置项配置成你实际的kafka集群对应的zookeeper地址。

kafka-manager.zkhosts="10.140.0.10:2181,10.140.0.11:2181,10.140.0.12:2181" 
cmak.zkhosts="10.140.0.10:2181,10.140.0.11:2181,10.140.0.12:2181"  
```

### 啟動服務

```text
$ bin/cmak -Dconfig.file=conf/application.conf -Dhttp.port=9001 
```

## WEB 設定



web首页

* 点击上图的“Add Cluster"进入添加集群向导。
* 配置要管理的Kafka集群信息

注意：如果需要管理监控的Kafka集群已经开启JMX\_PORT，那么就可以勾选额外蓝色的选项，否则不要勾选，按照默认的不勾选即可。

![](../../.gitbook/assets/tu-pian-%20%2819%29.png)

[![](https://s4.51cto.com/oss/202007/27/41e0dc5f12c24dd0b53b35e9c5ffd67d.jpg)](https://s4.51cto.com/oss/202007/27/41e0dc5f12c24dd0b53b35e9c5ffd67d.jpg)添加集群管理

如果有报错内容是这个：

```text
Yikes! KeeperErrorCode = Unimplemented for /kafka-manager/mutex Try again. 
```

那么你需要升级zookeeper到3.5+版本。

![](../../.gitbook/assets/tu-pian-%20%2821%29.png)

[![](https://s2.51cto.com/oss/202007/27/30065c1e13dd86a5d7dba341e9eabc23.jpg)](https://s2.51cto.com/oss/202007/27/30065c1e13dd86a5d7dba341e9eabc23.jpg)创建集群管理成功

**3.创建成功后，你就可以看到你的Kafka信息。**

![&#x96C6;&#x7FA4;&#x4FE1;&#x606F;](../../.gitbook/assets/tu-pian-%20%2822%29.png)

![&#x5177;&#x4F53;Topic&#x5217;&#x8868;](../../.gitbook/assets/tu-pian-%20%2820%29.png)

**结束语**

通过这个管理工具，我们可以进行Topic\(主题\)、分区等操作，不再需要通过命令行去调用Kafka集群获取信息，提高我们的效率。

补充一句：我之前一直用的是kafka-manager/archive/1.3.3.23.tar.gz的压缩包，新版本部署后效果是一样的。如果你所部署的kafka集群不支持最新CMAK的要求，可以下载1.3.3.23.tar.gz版本试试。

