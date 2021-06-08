# Logstash 設定

## Kafka 相關設定

本文介绍如何使用Logstash将Kafka中的数据写入到ElasticSearch。使用的各个组件版本如下：

* kafka\_2.11-2.2.0
* elasticsearch 6.8.3
* logstash 6.8.3

组件安装这里就不赘述了，主要说一下操作流程和需要注意的一些细节。

Logstash工作的流程由三部分组成：

* **input**：输入（即source），必须要有，指明从那里读取数据。
* **filter**：过滤，logstash对数据的ETL就是在这个里面进行的，这一步可选。
* **output**：输出（即sink），必须要有，指明数据写入到哪里去。

所以在我们的场景里面，input就是kafka，output就是es。至于是否需要filter，看你的场景需不需要对input的数据做transform了，本文没有使用filter。input需要使用`logstash-input-kafka`插件，该插件logstash默认就带了，可以使用`bin/logstash-plugin list | grep kafka`命令确认。

### 1. 基本功能 <a id="d0"></a>

创建一个的Logstash配置文件 _kafka.conf_，内容如下：

```text
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["nyc-test"]
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-kafka-%{+YYYY.MM.dd}"
  }
}
```



这个配置文件非常简单，

* 在input中配置了kafka的broker和topic信息，有了这两项配置，Logstash就知道从哪个kafka的哪个topic读取数据了；
* 在output中配置了es的hosts和index信息，有了这两项配置，Logstash就知道数据写到哪个es的哪个index里面去了。

启动Logstash：`bin/logstash -f kafka.conf`，确保没有错误。然后我们在kafka中创建上述配置文件中的topic，并写入一些数据：

```text
-> % bin/kafka-console-producer.sh --broker-list localhost:9092 --topic nyc-test
>{"key1": "value1"}
>{"key2": "value2"}
>
```



如果没有出错的话，此时数据已经写入到es了，我们查看一下：

```text
// 查看索引
-> % curl "http://localhost:9200/_cat/indices?v"
health status index                     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-kafka-2019.09.19 TgpGY4xgT4OMBgEKfbnDbQ   5   1          2            0      8.2kb          8.2kb

// 查看索引里面的数据
-> % curl "http://localhost:9200/logstash-kafka-2019.09.19/_search?pretty"
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logstash-kafka-2019.09.19",
        "_type" : "doc",
        "_id" : "t7O8SG0BpqLLPSZL6Tw4",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "@timestamp" : "2019-09-19T14:56:42.617Z",
          "message" : "{\"key1\": \"value1\"}"
        }
      },
      {
        "_index" : "logstash-kafka-2019.09.19",
        "_type" : "doc",
        "_id" : "uLO9SG0BpqLLPSZLojxO",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "@timestamp" : "2019-09-19T14:57:30.328Z",
          "message" : "{\"key2\": \"value2\"}"
        }
      }
    ]
  }
}
```



没问题，索引创建出来了，并且数据也写进去了。这样基本功能就算完成了。kafka插件还提供了许多自定义的配置项，我结合一些实际场景来介绍一下。

### 2. 一些有用的配置项 <a id="d1"></a>

#### 2.1 反序列化JSON <a id="d10"></a>

es是按照json格式存储数据的，上面的例子中，我们输入到kafka的数据是json格式的，但是经Logstash写入到es之后，整条数据变成一个字符串存储到`message`字段里面了。如果我们想要保持原来的json格式写入到es，只需要在input里面再加一条配置项：`codec => "json"`.

#### 2.2 并行传输 <a id="d11"></a>

Logstash的input读取数的时候可以多线程并行读取，`logstash-input-kafka`插件中对应的配置项是`consumer_threads`，默认值为1。一般这个默认值不是最佳选择，那这个值该配置多少呢？这个需要对kafka的模型有一定了解：

* kafka的topic是分区的，数据存储在每个分区内；
* kafka的consumer是分组的，任何一个consumer属于某一个组，一个组可以包含多个consumer，同一个组内的consumer不会重复消费的同一份数据。

所以，对于kafka的consumer，一般最佳配置是**同一个组内consumer个数（或线程数）等于topic的分区数**，这样consumer就会均分topic的分区，达到比较好的均衡效果。举个例子，比如一个topic有n个分区，consumer有m个线程。那最佳场景就是n=m，此时一个线程消费一个分区。如果n小于m，即线程数多于分区数，那多出来的线程就会空闲。如果n大于m，那就会存在一些线程同时消费多个分区的数据，造成线程间负载不均衡。所以，一般`consumer_threads`配置为你消费的topic的所包含的partition个数即可。如果有多个Logstash实例，那就让`实例个数 * consumer_threads`等于分区数即可。

消费者组名可以通过`group_id`配置，默认值为`logstash`。

#### 2.3 如何避免重复数据 <a id="d12"></a>

有些业务场景可能不能忍受重复数据，有一些配置项可以帮我们在一定程度上解决问题。这里需要先梳理一下可能造成重复数据的场景：

1. 数据产生的时候就有重复，业务想对重复数据去重（注意是去重，不是merge）。
2. 数据写入到Kafka时没有重复，但后续流程可能因为网络抖动、传输失败等导致重试造成数据重复。

对于第1种场景，只要原始数据中有唯一字段就可以去重；对于第2种场景，不需要依赖业务数据就可以去重。去重的原理也很简单，利用es document id即可。对于es，如果写入数据时没有指定document id，就会随机生成一个uuid，如果指定了，就使用指定的值。对于需要去重的场景，我们指定document id即可。在output elasticsearch中可以通过`document_id`字段指定document id。对于场景1非常简单，指定业务中的惟一字段为document id即可。主要看下场景2。

对于场景2，我们需要构造出一个“uuid”能惟一标识kafka中的一条数据，这个也非常简单：`<topic>+<partition>+<offset>`，这三个值的组合就可以惟一标识kafka集群中的一条数据。input kafka插件也已经帮我们把消息对应的元数据信息记录到了`@metadata`（Logstash的元数据字段，不会输出到output里面去）字段里面：

* `[@metadata][kafka][topic]`：索引信息
* `[@metadata][kafka][consumer_group]`：消费者组信息
* `[@metadata][kafka][partition]`：分区信息
* `[@metadata][kafka][offset]`：offset信息
* `[@metadata][kafka][key]`：消息的key（如果有的话）
* `[@metadata][kafka][timestamp]`：时间戳信息（消息创建的时间或者broker收到的时间）

所以，就可以这样配置document id了：

```text
document_id => "%{[@metadata][kafka][topic]}-%{[@metadata][kafka][partition]}-%{[@metadata][kafka][offset]}"
```



当然，如果每条kafka消息都有一个唯一的uuid的话，也可以在写入kafka的时候，将其写为key，然后这里就可以使用`[@metadata][kafka][key]`作为document id了。

最后一定要注意，**只有当`decorate_events`选项配置为true的时候，上面的@metadata才会记录那些元数据，否则不会记录。而该配置项的默认值是false，即不记录。**

现在我们把上面提到的那些配置都加入到 _kafka.conf_ 里面去，再运行一遍看看效果。新的配置文件：

```text
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["nyc-test"]
    consumer_threads => 2
    group_id => "logstash"
    codec => "json"
    decorate_events => true
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-kafka-%{+YYYY.MM.dd}"
    document_id => "%{[@metadata][kafka][topic]}-%{[@metadata][kafka][partition]}-%{[@metadata][kafka][offset]}"
  }
}
```



之前创建"nyc-test"这个topic时，我建了两个partition。之前的测试中没有配置`consumer_threads`，所以使用了默认值1，可以在Logstash中看到如下日志：

```text
[2019-09-19T22:54:48,207][INFO ][org.apache.kafka.clients.consumer.internals.ConsumerCoordinator] [Consumer clientId=logstash-0, groupId=logstash] Setting newly assigned partitions [nyc-test-1, nyc-test-0]
```



因为只有一个consumer，所以两个分区都分给了它。这次我们将`consumer_threads`设置成了2，看下效果：

```text
[2019-09-19T23:23:52,981][INFO ][org.apache.kafka.clients.consumer.internals.ConsumerCoordinator] [Consumer clientId=logstash-0, groupId=logstash] Setting newly assigned partitions [nyc-test-0]
[2019-09-19T23:23:52,982][INFO ][org.apache.kafka.clients.consumer.internals.ConsumerCoordinator] [Consumer clientId=logstash-1, groupId=logstash] Setting newly assigned partitions [nyc-test-1]
```



有两个线程，即两个consumer，所以各分到一个partition。

然后我们再写入两条数据：

```text
-> % bin/kafka-console-producer.sh --broker-list localhost:9092 --topic nyc-test
>{"key1": "value1"}
>{"key2": "value2"}
// 新写两条数据
>{"key3": "value3"}
>{"key4":{"key5": "value5"}}
```



然后查ES：

```text
{
    "_index" : "logstash-kafka-2019.09.19",
    "_type" : "doc",
    "_id" : "nyc-test-1-1",
    "_score" : 1.0,
    "_source" : {
      "key3" : "value3",
      "@version" : "1",
      "@timestamp" : "2019-09-19T15:18:05.971Z"
    }
},
{
    "_index" : "logstash-kafka-2019.09.19",
    "_type" : "doc",
    "_id" : "nyc-test-0-1",
    "_score" : 1.0,
    "_source" : {
        "@timestamp" : "2019-09-19T15:19:00.183Z",
        "key4" : {
        "key5" : "value5"
        },
        "@version" : "1"
    }
}
```



可以看到新写的两条数据的document id已经变了，而且消息内容也成json了，而不是之前作为一个字符串放在message字段中。

另外，大家有兴趣可以试一下使用`[@metadata][kafka][key]`做document id的情况。参考下面的方式指定kafka消息key（默认没有指定的话，key就是null）：

```text
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic nyc-test --property "parse.key=true" --property "key.separator=:"
```



当然，还有很多其它配置项，但大多不需要我们更改。有兴趣的可以查看文末链接。

