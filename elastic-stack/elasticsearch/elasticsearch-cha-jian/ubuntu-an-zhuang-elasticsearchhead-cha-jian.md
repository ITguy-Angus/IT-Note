# ubuntu安装ElasticSearch-head插件



1.elasticsearch-head插件介绍：  
elasticsearch-head是一个界面化的集群操作和管理工具，可以对集群进行傻瓜式操作。你可以通过插件把它集成到elasticsearch（5.0版本后不支持此方式）,由于我ES使用的是6.X版本，所以我把es-head安装成一个独立webapp,它依赖Node.js库，使用Grunt工具构建，所以需要先安装Node.js和Grunt。

2.安装node.js

\(0\)先检查是否已经安装了node.js 和npm,如果有跳过这一步

```text
$ node -v #验证node.js是否已经安装，如果能查看到版本号说明已经安装
$ npm -v #验证npm工具是否已经安装，如果能查看到版本号说明已经安装
```

\(1\)下载node.js

```text
$ sudo su
$ cd /usr/elasticsearch/
$ wget https://nodejs.org/dist/v6.9.2/node-v6.9.2-linux-x64.tar.xz
```

\(2\)解压文件：

```text
$ xz -d node-v6.9.2-linux-x64.tar.xz #解压出来是tar，需要继续解压
$ tar –xvf node-v6.9.2-linux-x64.tar 
$ mv node-v6.9.2-linux-x64 /usr/local/node //移到node目录
```

\(3\)配置环境变量

```text
$ vi /etc/profile #打开配置文件
```

在配置文件中添加：

```text
NODE_HOME=/usr/local/node #添加node_home变量
PATH= $PATH:$JAVA_HOME/bin:$NODE_HOME/bin #把node_home变量添加到环境变量中
export PATH
```

\(4\)重新加载配置文件

```text
$ source /etc/profile #重新加载环境变量配置文件
```

  
3.安装elasticsearch-head插件  
下载地址：https://github.com/mobz/elasticsearch-head 把源码elasticsearch-head-master.zip下载下来，然后上传到/usr/elasticsearch/下。解压。\(of course 直接git clone也OK \)

```text
$ wget https://codeload.github.com/mobz/elasticsearch-head/zip/master 
$ unzip elasticsearch-head-master.zip
$ mv elasticsearch-head-master /usr/elasticsearch/
```

安装grunt工具  
进入es-head,执行npm,使用taobao镜像

```text
$ npm install -g grunt --registry=https://registry.npm.taobao.org #安装grunt工具
```

编译elasticsearch-head源码

```text
$ npm install -g cnpm --registry=https://registry.npm.taobao.org #安装cnpm
$ cnpm install #使用cnpm代替npm编译es-head源码
```

编译好后es-head根目录下会出现一个叫node\_modules的目录，该目录就是存放源码编译后的可执行文件。

插件配置修改  
（1）设置插件管理界面跨主机访问  
插件默认是只有127.0.0.1才能访问，这样我们就无法跨主机访问管理界面，所以需要把它改成0.0.0.0，才能跨机访问。该配置在head插件安装目录根目录下，文件名为Gruntfile.js。

```text
$ vim Gruntfile.js
```

在port: 9100上面加一行hostname: '0.0.0.0',这样其他机器就可以访问了。如果有开防火墙，记得开放9100端口。

![](../../../.gitbook/assets/tu-pian-%20%2816%29.png)

（2）设置连接elasticsearch的地址：插件默认是连接本机es的，即127.0.0.1:9200,如果ES不在本机上或者端口不是默认端口，需要在app.js进行配置，配置文件在head插件安装目录的\_site目录下。把this.base\_uri = this.config.base\_uri \|\| this.prefs.get\(“app-base\_uri”\) \|\| “http://localhost:9200”;这行配置中的http://localhost:9200改成你elasticsearch服务所在IP地址。

```text
$ vim _site/app.js
```

![](../../../.gitbook/assets/tu-pian-%20%2813%29.png)

（3）elasticsearch配置允许跨域访问：修改elasticsearch配置文件，

```text
$ locate elasticsearch.yml #查看配置文件所在位置
```

![](../../../.gitbook/assets/tu-pian-%20%2818%29.png)

```text
$ vim /etc/elasticsearch/elasticsearch.yml
```

在最后加上两行：

```text
http.cors.enabled: true
http.cors.allow-origin: “*”
```

保存后重启es。

```text
$ sudo /etc/init.d/elasticsearch restart
```

进入_elasticsearch-head-master_下，运行grunt server

```text
$ cd elasticsearch-head-master
$ grunt server
```

如果能打印以下的信息，那就说明head插件安装并运行成功，我们可以通过http://ip:9100访问管理页面。

![](../../../.gitbook/assets/tu-pian-%20%2814%29.png)

![](../../../.gitbook/assets/tu-pian-%20%2815%29.png)

这里并不是后台启动，我们可以创建一个后台启动文件，使其能够后台启动。

```text
$ vim es-head-start.sh
```

文件输入以下信息，保存后执行 sh es-head-start.sh实现后台启动。

```text
#!/bin/bash
echo "START elasticsearch-head "
nohup grunt server &exit
```

