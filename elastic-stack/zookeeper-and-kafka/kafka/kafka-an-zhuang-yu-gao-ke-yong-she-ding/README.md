# Kafka & Zookeeper 介紹與安裝

第三层、数据分析层 Logstash作为消费者， 会去Kafka+zookeeper集群节点实时拉取原始日志， 然后将获取到的原始日志根据规则进行分析、清洗、过滤， 最后将清洗好的日志转发至Elasticsearch集群。



