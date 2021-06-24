# Untitled

## [ES--es7.6.1开通安全防护认证流程](https://segmentfault.com/a/1190000039033241)

## 1.啥是X-Pack？ <a id="item-1"></a>

X-Pack是Elastic Stack扩展功能，提供安全性，警报，监视，报告，机器学习和许多其他功能。 ES7.0+之后，默认情况下，当安装Elasticsearch时，会安装X-Pack，无需单独再安装。

* 1、5.X版本之前：没有x-pack，是独立的：security安全,watch查看，alert警告等独立单元。
* 2、5.X版本：对原本的安全，警告，监视，图形和报告做了一个封装，形成了x-pack。
* 3、6.3 版本之前：需要额外安装。
* 4、6.3版本及之后：已经集成在一起发布，无需额外安装，基础安全属于付费黄金版内容。 7 .1版本：基础安全免费。

自6.8以及7.1+版本之后，基础级安全永久免费。

基础版本安全功能列表如下：  
![&#x57FA;&#x7840;&#x7248;&#x529F;&#x80FD;&#x56FE;](https://segmentfault.com/img/bVcNUny)

## 2.ES配置 <a id="item-2"></a>

#### 2.1集群化配置（docker环境）

**1.进入ES docker容器，生成节点证书**

借助elasticsearch-certutil命令生成证书  
 （在elasticsearch目录下执行/usr/share/elasticsearch）

```text
/usr/share/elasticsearch/bin/elasticsearch-certutil.bat ca -out config/elastic-certificates.p12 -pass ""
```

或

```text
/usr/share/elasticsearch/bin/elasticsearch-certutil ca -out config/elastic-certificates.p12 -pass ""
```

或

```text
/usr/share/elasticsearch/bin/elasticsearch-certutil ca
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

//-out 指定证书生成路径，-pass "" 默认无密码  
**注：只需要使用由同一CA签名的证书，即可自动允许该节点加入集群**  
![image](https://segmentfault.com/img/bVcNWjX)

**2.配置加密通信**

启用安全功能后，必须使用TLS来确保节点之间的通信已加密。  
在elasticsearch.yml中心新增配置如下：

```text
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12 
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12 
```

具体配置如下：

```text
cluster.name: es-docker
node.name: es-nodename
network.bind_host: 0.0.0.0
network.publish_host: ***.***.***.*** ##本机ip地址
http.port: 9200 ## 开放端口
transport.tcp.port: 9300 ##开放tcp端口
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
node.master: true 
node.data: true
discovery.seed_hosts: ["***.***.***.***:9300","elasticsearch2:10293","elasticsearch1:10193"]
cluster.initial_master_nodes: ["es-nodename1","es-nodename2","es-nodename3"]
```

**注意：**  
**1.最好将证书与配置文件放在同一目录下**  
**2.生成的这书最好手动赋一下权 chmod 777 \*.ca**  
**3.其他节点参照第4步配置，配置完成后统一重启ES**

**3.设置集群密码**

ES中内置了几个管理其他集成组件的账号即：`apm_system`, `beats_system`, `elastic`, `kibana`, `logstash_system`, `remote_monitoring_user`，使用之前，首先需要添加一下密码：

```text
/bin/elasticsearch-setup-passwords interactive
```

或者

```text
bin/elasticsearch-setup-passwords interactive -u "http://elasticsearch22:10492"
```

* interactive：给用户一一设置密码。
* auto：自动生成密码。

![image](https://segmentfault.com/img/bVcNWkR)  
**注意：必须配置好xpack之后，才能设置密码。否则会报错。**  
如果这个地方报如下错误：

```text
Failed to determine the health of the cluster running at http://192.168.3.42:9200
Unexpected response code [503] from calling GET http://192.168.3.42:9200/_cluster/health?pretty
Cause: master_not_discovered_exception

It is recommended that you resolve the issues with your cluster before running elasticsearch-setup-passwords.
It is very likely that the password changes will fail when run against an unhealthy cluster.
Do you want to continue with the password setup process [y/N]y
```

可能是有脏数据导致，此时可以停掉es，删除 data 数据目录，然后重新启动在进行操作。

**4.配置其他节点**

* 将配置好的带证书的文件copy到另外的其他节点下，放在同样目录下
* elasticsearch.yml 复制新增配置和配置好的节点一样
* 除了node.name使用各自主机名之外，其他配置都一样。

**5.配置完成**

配置完毕之后，可以通过如下方式访问es服务：

```text
curl -XGET -u elastic 'localhost:9200/_xpack/security/user?pretty'
curl 127.0.0.1:9200 -u elastic
```

**6.错误记录**

1.重启kibana时报错

```text
 Error: Index .kibana belongs to a version of Kibana that cannot be automatically migrated. Reset it or use the X-Pack upgrade assistant.
    at assertIsSupportedIndex (/usr/share/kibana/src/core/server/saved_objects/migrations/core/elastic_index.js:318:11)
    at fetchInfo (/usr/share/kibana/src/core/server/saved_objects/migrations/core/elastic_index.js:70:10)
```

原因：这是由于，在es没配置xpack之前,索引中已存在关于.kibana的索引，配置xpack以后重启kibana导致与原本已存在的.kibana的索引不兼容  
解决方法：  
 1.在配置xpack之前，先关闭正在运行的kibana,  
 2.删除es集群中关于.kibana的索引数据  
 3.重新配置一遍再重启就可以了

```text
Unable to connect to Elasticsearch. Error: [resource_already_exists_exception] index [.kibana_2/0eg0V-EgQHS1egmO169B1Q] already exists, with { index_uuid="0eg0V-EgQHS1egmO169B1Q" & index=".kibana_2" }
  log   [04:02:50.038] [warning][savedobjects-service] Another Kibana instance appears to be migrating the index. Waiting for that migration to complete. If no other Kibana instance is attempting migrations, you can get past this message by deleting index .kibana_2 and restarting Kibana.
  log   [04:02:50.041] [warning][savedobjects-service] Unable to connect to Elasticsearch. Error: [resource_already_exists_exception] index [.kibana_task_manager_1/HueSg7AlRxuUiFutevaNWQ] already exists, with { index_uuid="HueSg7AlRxuUiFutevaNWQ" & index=".kibana_task_manager_1" }
  log   [04:02:50.041] [warning][savedobjects-service] Another Kibana instance appears to be migrating the index. Waiting for that migration to complete. If no other Kibana instance is attempting migrations, you can get past this message by deleting index .kibana_task_manager_1 and restarting Kibana.
```

