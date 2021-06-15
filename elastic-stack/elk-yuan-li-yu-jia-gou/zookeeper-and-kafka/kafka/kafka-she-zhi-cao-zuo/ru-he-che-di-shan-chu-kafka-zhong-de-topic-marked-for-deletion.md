---
description: 如何彻底删除Kafka中的topic (marked for deletion)
---

# 如何彻底删除Kafka中的topic \(marked for deletion\)

kafka删除topic时发现提示只是被标记删除了，并没有真正的删除，如下：

| 1 2 3 4 5  | \[root@lee000 ~\]\# kafka-topics.sh --delete --zookeeper lee000:2181 --topic my-replicated-topic  Topic my-replicated-topic is already marked for deletion.  \[root@lee000 ~\]\# kafka-topics.sh --list --zookeeper lee000:2181  \_\_consumer\_offsets  my-replicated-topic - marked for deletion |
| :--- | :--- |


### 方案：

**1）配置server.properties**

启动Kafaka时如果加载的配置文件中"server.properties"没有配置"delete.topic.enable=true"，那么此时的删除并不是真正的删除，而是把该topic标记为"marked for deletion"。追加参数后记得重启Kafka。

| 1  | delete.topic.enable=true |
| :--- | :--- |


**2）通过zookeeper客户端zkCli.sh删除**

> "server.properties"中配置"delete.topic.enable=true"并重启Kafka后执行前面的删除命令如果仍然提示"marked for deletion"，则可以继续以下操作。

启动zookeeper客户端：

| 1  | zkCli.sh |
| :--- | :--- |


查看topics目录下所有topic：

| 1  | ls /brokers/topics |
| :--- | :--- |


删除指定topic：

| 1  | deleteall /brokers/topics/{topic name} |
| :--- | :--- |


如下：

| 1 2 3 4 5 6 7 8  | \[root@lee000 tmp\]\# zkCli.sh  Connecting to localhost:2181  ...  \[zk: localhost:2181\(CONNECTED\) 4\] ls /brokers/topics  \[\_\_consumer\_offsets, my-replicated-topic\]  \[zk: localhost:2181\(CONNECTED\) 7\] deleteall /brokers/topics/my-replicated-topic  \[zk: localhost:2181\(CONNECTED\) 8\] ls /brokers/topics  \[\_\_consumer\_offsets\] |
| :--- | :--- |


自此，再次查看可以发现topic被真正删除：

| 1 2  | \[root@lee000 tmp\]\# kafka-topics.sh --list --zookeeper lee000:2181  \_\_consumer\_offsets |
| :--- | :--- |


**3）删除kafka存储目录（按需）**

Kafka的存储目录由"server.properties"文件中的"log.dirs"参数指定，默认为"/tmp/kafka-logs"。  
 删除该目录下topic相关目录。

