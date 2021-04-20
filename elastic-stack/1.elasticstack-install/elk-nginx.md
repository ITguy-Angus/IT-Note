---
description: Nginx 設定相關
---

# ELK Nginx

## Filebeat 設定相關

**查看可用模塊**

```text
cd /usr/local/filebeat-7.6.2-linux-x86_64
# 查看支持哪些模块
./filebeat modules list
Enabled:
Disabled:
activemq
apache
auditd
aws
azure
cef
cisco
coredns
elasticsearch
envoyproxy
googlecloud
haproxy
ibmmq
icinga
iis
iptables
kafka
kibana
logstash
misp
mongodb
mssql
mysql
nats
netflow
nginx
osquery
panw
postgresql
rabbitmq
redis
santa
suricata
system
traefik
zeek
```

 

#### **配置 nginx 模块**

可以看到，内置了很多的 module，但是都没有启用，如果需要启用需要进行 enable 操作

```text
# 启动
./filebeat modules enable nginx
# 输出，说明 nginx 模块已可用
Enabled nginx
# 禁用
./filebeat modules disable nginx
```

模块开启之后，进入到 `modules.d` 目录下，发现只有 `nginx.yml` 后面没有 `.disabled`

```text
cd modules.d/
ls -al
-rw-r--r-- 1 root root  483 Mar 26 01:23 activemq.yml.disabled
-rw-r--r-- 1 root root  475 Mar 26 01:23 apache.yml.disabled
-rw-r--r-- 1 root root  280 Mar 26 01:23 auditd.yml.disabled
-rw-r--r-- 1 root root 3064 Mar 26 01:23 aws.yml.disabled
-rw-r--r-- 1 root root 1382 Mar 26 01:23 azure.yml.disabled
-rw-r--r-- 1 root root  200 Mar 26 01:23 cef.yml.disabled
-rw-r--r-- 1 root root 1978 Mar 26 01:23 cisco.yml.disabled
-rw-r--r-- 1 root root  318 Mar 26 01:23 coredns.yml.disabled
-rw-r--r-- 1 root root  964 Mar 26 01:23 elasticsearch.yml.disabled
-rw-r--r-- 1 root root  327 Mar 26 01:23 envoyproxy.yml.disabled
-rw-r--r-- 1 root root 2019 Mar 26 01:23 googlecloud.yml.disabled
-rw-r--r-- 1 root root  376 Mar 26 01:23 haproxy.yml.disabled
-rw-r--r-- 1 root root  295 Mar 26 01:23 ibmmq.yml.disabled
-rw-r--r-- 1 root root  651 Mar 26 01:23 icinga.yml.disabled
-rw-r--r-- 1 root root  470 Mar 26 01:23 iis.yml.disabled
-rw-r--r-- 1 root root  366 Mar 26 01:23 iptables.yml.disabled
-rw-r--r-- 1 root root  398 Mar 26 01:23 kafka.yml.disabled
-rw-r--r-- 1 root root  293 Mar 26 01:23 kibana.yml.disabled
-rw-r--r-- 1 root root  471 Mar 26 01:23 logstash.yml.disabled
-rw-r--r-- 1 root root  300 Mar 26 01:23 misp.yml.disabled
-rw-r--r-- 1 root root  296 Mar 26 01:23 mongodb.yml.disabled
-rw-r--r-- 1 root root  311 Mar 26 01:23 mssql.yml.disabled
-rw-r--r-- 1 root root  471 Mar 26 01:23 mysql.yml.disabled
-rw-r--r-- 1 root root  287 Mar 26 01:23 nats.yml.disabled
-rw-r--r-- 1 root root  214 Mar 26 01:23 netflow.yml.disabled
-rw-r--r-- 1 root root  472 Mar 26 01:23 nginx.yml
-rw-r--r-- 1 root root  495 Mar 26 01:23 osquery.yml.disabled
-rw-r--r-- 1 root root  356 Mar 26 01:23 panw.yml.disabled
-rw-r--r-- 1 root root  305 Mar 26 01:23 postgresql.yml.disabled
-rw-r--r-- 1 root root  343 Mar 26 01:23 rabbitmq.yml.disabled
-rw-r--r-- 1 root root  566 Mar 26 01:23 redis.yml.disabled
-rw-r--r-- 1 root root  266 Mar 26 01:23 santa.yml.disabled
-rw-r--r-- 1 root root  299 Mar 26 01:23 suricata.yml.disabled
-rw-r--r-- 1 root root  477 Mar 26 01:23 system.yml.disabled
-rw-r--r-- 1 root root  302 Mar 26 01:23 traefik.yml.disabled
-rw-r--r-- 1 root root 1294 Mar 26 01:23 zeek.yml.disable
```

修改 `nginx.yml` 配置文件，分别增加 `access` 和 `error` 的日志文件路径，注意路径最后增加 `*`，因为 `Nginx` 以日期归档日志文件  


![](../../.gitbook/assets/tu-pian-%20%284%29.png)

