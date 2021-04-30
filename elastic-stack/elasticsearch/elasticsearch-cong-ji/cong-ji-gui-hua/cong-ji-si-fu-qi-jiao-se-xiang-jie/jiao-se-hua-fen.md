# 角色划分



## 角色划分 <a id="&#x89D2;&#x8272;&#x5212;&#x5206;"></a>

在Elasticsearch中，有很多角色，常用的角色有如下：  
Master Node：主节点  
Master eligible nodes：合格节点  
Data Node：数据节点  
Coordinating Node：协调节点  
Ingest Node：ingest节点  
machine learning：机器学习节点

![](../../../../../.gitbook/assets/tu-pian-%20%2811%29.png)

![](../../../../../.gitbook/assets/tu-pian-%20%2810%29.png)

* Master Node：主节点，该节点不和应用创建连接，每个节点都保存了集群状态，master节点不占用磁盘IO和CPU，内存使用量一般。
* Master eligible nodes：合格节点，每个节点部署后不修改配置信息，默认就是一个 eligible 节点，该节点可以参加选主流程，成为Mastere节点。该节点也保存了集群节点的状态。eligible节点比Master节点更节省资源，因为它还未成为 Master 节点，只是有资格成功Master节点。
* Data Node：数据节点，该节点和索引应用创建连接、接收索引请求，该节点真正存储数据，ES集群的性能取决于该节点的个数（每个节点最优配置的情况下），data节点会占用大量的CPU、IO和内存。
* Coordinating Node：协调节点，该节点和检索应用创建连接、接受检索请求，但其本身不负责存储数据，可当负责均衡节点，该节点不占用io、cpu和内存。
* Ingest Node：ingest 节点可以看作是数据前置处理转换的节点，支持 pipeline管道 设置，可以使用 ingest 对数据进行过滤、转换等操作，类似于 logstash 中 filter 的作用，功能相当强大。

## 各节点间的关系 <a id="&#x5404;&#x8282;&#x70B9;&#x95F4;&#x7684;&#x5173;&#x7CFB;"></a>

* Master Node：master节点控制整个集群的元数据。只有Master Node节点可以修改节点状态信息及元数据\(metadata\)的处理，比如索引的新增、删除、分片路由分配、所有索引和相关 Mapping 、Setting 配置等等。
* Master eligible nodes：有资格成为Master节点但暂时并不是Master的节点被称为 eligible 节点，该节点只是与集群保持心跳，判断Master是否存活，如果Master故障则参加新一轮的Master选举。
* Data Node：data节点的分片执行查询语句获得查询结果后将结果反馈给Coordinating节点，在查询的过程中非常消耗硬件资源，如果在分片配置及优化没做好的情况下，进行一次查询非常缓慢\(硬件配置也要跟上数据量\)。
* Coordinating Node：协调节点接受客户端搜索请求后将请求转发到与查询条件相关的多个data节点的分片上，然后多个data节点的分片执行查询语句或者查询结果再返回给协调节点，协调节点把各个data节点的返回结果进行整合、排序等一系列操作后再将最终结果返回给用户请求。
* Ingest Node： Ingest节点处理时机——在数据被索引之前，通过预定义好的处理管道对数据进行预处理。默认情况下，所有节点都启用Ingest，因此任何节点都可以处理Ingest任务。我们也可以创建专用的Ingest节点。

## 资源规划 <a id="&#x8D44;&#x6E90;&#x89C4;&#x5212;"></a>

* Master Node：Elasticsearch如果做集群的话Master节点至少三台服务器或者三个Master实例加入相同集群\(生产建议每个es实例部署在不同的设备上\)，三个Master节点最多只能故障一台Master节点，数据不会丢失，如果三个节点故障两个节点，则造成数据丢失并无法组成集群。
* Master eligible nodes：Elasticsearch如果使用三台Master做集群，其中一台被真正选为了Master，那么其它两台就是 eligible 节点。
* Data Node：在Elasticsearch集群中，此节点应该是最多的，单个索引在一个data节点实例上分片数保持在3个以内\(我觉得分片数量按照Data节点数量划分比较好，每个节点上存储一个分片\)；每1GB堆内存对应集群的分片保持在20个以内；每个分片大小不要超过30G。
* Coordinating Node: 增加协调节点可增加检索并发,但检索的速度还是取决于查询所命中的分片个数以及分片中的数据量。

## Data节点建议 <a id="data&#x8282;&#x70B9;&#x5EFA;&#x8BAE;"></a>

### 内存建议 <a id="&#x5185;&#x5B58;&#x5EFA;&#x8BAE;"></a>

假如一台机器部署了一个ES实例，则ES最大可用内存给到物理内存的50%，最多不可超过32G，如果单台机器上部署了多个ES实例，则多个ES实例内存相加等于物理内存的50%，多个ES实例内存相加不宜超过32G。

### 分片建议： <a id="&#x5206;&#x7247;&#x5EFA;&#x8BAE;&#xFF1A;"></a>

* 如果单个分片每个节点可支撑90G数据，依此可计算出所需data节点数。
* 如果多个分片按照单个data节点jvm内存最大30G来计算，一个节点的分片保持在600个以内，存储保持在18T以内。

