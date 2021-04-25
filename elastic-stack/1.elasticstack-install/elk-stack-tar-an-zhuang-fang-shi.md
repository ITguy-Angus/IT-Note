# ELK Stack tar安裝方式

## ELK Stack 7.12 解壓安裝

### Elasticsearch 安裝

`cd /usr/local/`

`curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-linux-x86_64.tar.gz`

 `tar -xvf elasticsearch-7.12.0-linux-x86_64.tar.gz`

`rm elasticsearch-7.12.0-linux-x86_64.tar.gz`

`mv elasticsearch-7.12.0-linux-x86_64/elasticsearch`

`ln -s /usr/local/elasticsearch elasticsearch`

* 编辑 ./config/elasticsearch.yml

`vim /usr/local/elasticsearch-7.12.0/config/elasticsearch.yml`

```text
# 添加或修改
node.name: node-1
#network.host:0.0.0.0 代表開給外網連線
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

* 修改文件包含限制一个进程可以拥有的VMA\(虚拟内存区域\)的数量配置，编辑 /etc/sysctl.conf

`vim /etc/sysctl.conf`

```text
# 添加或修改
# sudo sysctl -p 使修改生效
vm.max_map_count = 262144
```

* 修改 ECS 安全组，放行 9200 端口

`ufw allow in 9200`

`ufw allow in 9300`

查看目前防火牆規則



```text
sudo ufw status # 查看目前防火牆規則
sudo ufw status numbered #以數字排列目前防火牆規則
```

*  創建帳戶

`adduser elasticsearch`

* 将对应的文件夹权限赋给该用户

`chown -R elasticsearch /usr/local/elasticsearch/elasticsearch-7.12.0`

* 切换至elasticsearch用户 

`su elasticsearch` 

* 进入启动目录启动 /usr/local/elasticsearch-7.12.0/bin 使用后台启动方式

`./elasticsearch -d`

* 測試是否成功啟動

 `curl -X GET "localhost:9200"`



### Kibana 安裝

`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz`

`tar -xvf kibana-7.12.0-linux-x86_64.tar.gz`

`rm kibana-7.12.0-linux-x86_64.tar.gz`

`mv kibana-7.12.0-linux-x86_64/ kibana`

`ln -s /usr/local/kibana kibana`

* 設定檔設定

`vim config/kibana.yml`

```text
server.port: 5601
server.host: "123.456.789.0"
server.name: "kibana-test"
elasticsearch.hosts: ["http://10.10.0.8:9200"]
# kibana会将部分数据写入es，这个是ex中索引的名字
kibana.index: ".kibana"
```

*  創建帳戶

`adduser kibana`

* 将对应的文件夹权限赋给该用户

`chown -R kibana /usr/local/kibana`

* 切换至kibana用户 

`su kibana`

* 进入启动目录启动 /usr/local/kibana/bin 使用后台启动方式

`nohup ./kibana &`

* 測試是否成功啟動

 `http://localhost:5601`

### Logstash 安裝

* 安裝

`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.12.0-linux-x86_64.tar.gz`

`tar -xvf logstash-7.12.0-linux-x86_64.tar.gz`

`rm logstash-7.12.0-linux-x86_64.tar.gz`

`mv logstash-7.12.0-linux-x86_64/ logstash`

~~ln -s /usr/local/logstash logstash~~



*  創建帳戶

`adduser logstash`

* 将对应的文件夹权限赋给该用户

`chown -R logstash /usr/local/logstash`

* 切换至logstash用户 

`su logstash`

* 进入启动目录启动 /usr/local/logstash/bin 使用后台启动方式

`./bin/logstash -f config/logstash.conf &`

* 確認是否成功啟動

```text
[1] 17978
root@logstash1:/usr/local/logstash# Using bundled JDK: /usr/local/logstash/jdk
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be remove
d in a future release.
Sending Logstash logs to /usr/local/logstash/logs which is now configured via log4j2.properties
[2021-04-19T01:17:47,801][INFO ][logstash.runner          ] Log4j configuration path used is: /usr/local/logstash/c
onfig/log4j2.properties
[2021-04-19T01:17:47,834][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.12.0", "jruby
.version"=>"jruby 9.2.13.0 (2.5.7) 2020-08-03 9a89c94bcc OpenJDK 64-Bit Server VM 11.0.10+9 on 11.0.10+9 +indy +jit
 [linux-x86_64]"}
[2021-04-19T01:17:48,799][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modul
es or command line options are specified
[2021-04-19T01:17:50,368][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600
}
[2021-04-19T01:17:51,843][INFO ][org.reflections.Reflections] Reflections took 76 ms to scan 1 urls, producing 23 k
eys and 47 values 
[2021-04-19T01:17:53,283][INFO ][logstash.outputs.elasticsearch][main] Elasticsearch pool URLs updated {:changes=>{
:removed=>[], :added=>[http://10.140.0.6:9200/]}}
[2021-04-19T01:17:53,635][WARN ][logstash.outputs.elasticsearch][main] Restored connection to ES instance {:url=>"h
ttp://10.140.0.6:9200/"}
[2021-04-19T01:17:53,710][INFO ][logstash.outputs.elasticsearch][main] ES Output version determined {:es_version=>7
}
[2021-04-19T01:17:53,715][WARN ][logstash.outputs.elasticsearch][main] Detected a 6.x and above cluster: the `type`
 event field won't be used to determine the document _type {:es_version=>7}
[2021-04-19T01:17:53,784][INFO ][logstash.outputs.elasticsearch][main] New Elasticsearch output {:class=>"LogStash:
:Outputs::ElasticSearch", :hosts=>["http://10.140.0.6:9200"]}
[2021-04-19T01:17:53,948][INFO ][logstash.outputs.elasticsearch][main] Using a default mapping template {:es_versio
n=>7, :ecs_compatibility=>:disabled}
[2021-04-19T01:17:54,021][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipelin
e.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>250, "pipeline.sour
ces"=>["/usr/local/logstash/config/logstash.conf"], :thread=>"#<Thread:0xac94cdd run>"}
[2021-04-19T01:17:54,062][INFO ][logstash.outputs.elasticsearch][main] Attempting to install template {:manage_temp
late=>{"index_patterns"=>"logstash-*", "version"=>60001, "settings"=>{"index.refresh_interval"=>"5s", "number_of_sh
ards"=>1}, "mappings"=>{"dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"st
ring", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"strin
g", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}]
, "properties"=>{"@timestamp"=>{"type"=>"date"}, "@version"=>{"type"=>"keyword"}, "geoip"=>{"dynamic"=>true, "prope
rties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>
{"type"=>"half_float"}}}}}}}
[2021-04-19T01:17:55,597][INFO ][logstash.javapipeline    ][main] Pipeline Java execution initialization time {"sec
onds"=>1.57}
[2021-04-19T01:17:55,646][INFO ][logstash.inputs.beats    ][main] Starting input listener {:address=>"0.0.0.0:5044"
}
[2021-04-19T01:17:55,670][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
[2021-04-19T01:17:55,752][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:mai
n], :non_running_pipelines=>[]}
[2021-04-19T01:17:55,954][INFO ][org.logstash.beats.Server][main][d30cec8718b4eec2fe086d75154440802a7f35a6572519d10
06ee383031ddb4c] Starting server on port: 5044
```

### Filebeat 安裝

* 安裝

`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.0-linux-x86_64.tar.gz`

`tar -xvf filebeat-7.12.0-linux-x86_64.tar.gz`

`rm filebeat-7.12.0-linux-x86_64.tar.gz`

`mv filebeat-7.12.0-linux-x86_64/ filebeat`

`ln -s /usr/local/filebeat filebeat`

*  創建帳戶

`adduser filebeat`

* 将对应的文件夹权限赋给该用户

`chown -R filebeat /usr/local/filebeat`

* 切换至filebeat用户 

`su filebeat`

* Config

```text
#複製貼上下面內容 (localhost改成自己的IP)
name: localhost
output:
  logstash:
    enabled: true
    hosts:
      - localhost:5044
    index: "localhost"filebeat.inputs:
    - type: log
      paths:
        - /usr/local/nginx/logs/access.log
      tags: ["access"]
#開啟debug模式
logging.level: debug
logging.selectors: [publish]
logging.to_files: true
logging.files:
    path: /var/log/filebeat
    name: filebeat-localhost
```

```text
#複製貼上下面內容 (localhost改成自己的IP)
name: 10.140.0.8
output.logstash:
    hosts:  ["10.140.0.7:5044"]
    index: "Nginx"
filebeat.inputs:
    - type: log
      paths:
        - /var/log/nginx/access.log
      tags: ["access"]
#開啟debug模式
logging.level: debug
logging.selectors: [publish]
logging.to_files: true
logging.files:
    path: /var/log/filebeat
    name: filebeat-localhost
```

* 进入启动目录启动 /usr/local/logstash/bin 使用后台启动方式注意後面可以帶入不同設定檔檔名

`/usr/local/filebeat/filebeat -c /usr/local/filebeat/nginxlog.yml &`

* 測試是否成功啟動

`ps -aux | grep filebeat`







`cd /usr/local`

`curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz`

`tar -xvf kibana-7.12.0-linux-x86_64.tar.gz`

`rm kibana-7.12.0-linux-x86_64.tar.gz`

`mv kibana-7.12.0-linux-x86_64/ kibana`

 `cd ./kibana/`

`ln -s /usr/local/kibana kibana`

*  創建帳戶

`adduser kibana`

* 将对应的文件夹权限赋给该用户

`chown -R kibana/usr/local/kibana`

* 切换至kibana用户 

`su kibana`

* 进入启动目录启动 /usr/local/elasticsearch-7.12.0/bin 使用后台启动方式

`./kibana -d`

* 測試是否成功啟動

 `curl -X GET "localhost:5601"`

