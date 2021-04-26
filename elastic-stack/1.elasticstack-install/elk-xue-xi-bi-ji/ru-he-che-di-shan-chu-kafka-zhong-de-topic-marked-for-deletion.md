---
description: 如何彻底删除Kafka中的topic (marked for deletion)
---

# 如何彻底删除Kafka中的topic \(marked for deletion\)

工作中因为各种原因， 例如topic中消息堆积的太多，或者kafka所在磁盘空间满了等等，可能需要彻底清理一下kafka topic，那么如何彻底删除topic？ 方法一（配置delete.topic.enable=true） 修改kafaka配置文件server.properties， 添加delete.topic.enable=true，重启kafka，之后通过kafka命令行就可以直接删除topic 通过命令行删除topic：

./bin/kafka-topics.sh --delete --zookeeper {zookeeper server} --topic {topic name}

方法二（没有配置delete.topic.enable=true）

1、通过命令行删除topic：

./bin/kafka-topics.sh --delete --zookeeper {zookeeper server} --topic {topic name}

因为kafaka配置文件中server.properties没有配置delete.topic.enable=true，此时的删除并不是真正的删除，只是把topic标记为：marked for deletion 你可以通过命令：./bin/kafka-topics --zookeeper {zookeeper server} --list 来查看所有topic 2、删除kafka存储目录（server.properties文件log.dirs配置，默认为"/tmp/kafka-logs"）相关topic目录 在这里插入图片描述

3， 若想真正删除它，需要登录zookeeper客户端：

命令：./bin/zkCli.sh

找到topic所在的目录：ls /brokers/topics

执行命令：rmr /brokers/topics/{topic name}即可，此时topic被彻底删除。

另外被标记为marked for deletion的topic你可以在zookeeper客户端中通过命令获得：ls /admin/delete\_topics/{topic name}，如果你删除了此处的topic，那么marked for deletion 标记消失

zookeeper 的config中也有有关topic的信息： ls /config/topics/{topic name}暂时不知道有什么用

总结

彻底删除topic： 1、确保kafka的配置文件server.proeprties中设置delete.topic.enable=true，如果没有， 确保cluster的所有kafka配置文件设置该参数并重启，然后直接通过命令删除，如果命令删除不了，直接通过zookeeper命令行删除掉broker下的topic。

2、删除kafka存储目录（server.properties文件log.dirs配置，默认为"/tmp/kafka-logs"）相关topic目录 ———————————————— 版权声明：本文为CSDN博主「russle」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。 原文链接：[https://blog.csdn.net/russle/article/details/82881297](https://blog.csdn.net/russle/article/details/82881297)

