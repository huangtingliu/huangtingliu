---
layout: default
title: ELASTICSEARCH搜索引擎（上）
tags: [大数据,搜索引擎]
imgPath: 2018-09-14-ELASTICSEARCH
---
> [elasticsearch](https://www.elastic.co/)是一个分布式的 RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

本文介绍在windows下elasticsearch的使用

# 一、elasticsearch安装
### 1. 下载elasticseach(本文下载的是6.3.1)  
<https://www.elastic.co/downloads/elasticsearch>  
解压下载的压缩文件，试着启动，在解压的目录的bin下运行elasticsearch.bat文件，启动成功后、默认地址127.0.0.1:9200，会出现介绍elasticsearch的json格式串
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/1downloaded-aboult.png)

# 二、单实例插件安装
elasticsearch有许多插件可以使用，本文以web可视化插件elasticsearch-head为例

### 1. 在github上下载web可视化插件 mobz/elasticsearch-head，解压
<https://github.com/mobz/elasticsearch-head>

### 2. 这里需要安装node.js（v6.0版本以上），检查node环境，node -v命令检查nodejs版本号

### 3. 在elasticsearch-head解压目录下，npm install 命令安装依赖

### 4. 试着运行elasticsearch-head，在解压目录下运行
```
npm run start  
``` 
命令启动，打开浏览器输入默认地址127.0.0.1::9100查看  
可以看到这时连接状态是未连接，因为和elasticsearch存在跨域访问问题

![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/2head-plugin-downloaded-aboult.png)

### 5. elasticsearch和elasticsearch-head插件跨域配置 
elasticsearch和elasticsearch-head插件是两个独立的个体，互相访问是有跨域问题的。所以需要更改配置
修改elasticsearch目录下config/elasticsearch.yml文件，在最后一行加上:
```
http.cors.enabled: true
http:cors.allow-origin: "*"
```

### 6. 重新启动elasticsearch
进入elasticsearch的bin目录，运行elasticsearch.bat文件

### 7. 重新启动elasticsearch-head，在解压目录下运行
```
npm run start
```
命令启动，打开浏览器输入默认地址127.0.0.1::9200查看  
可以看到这时连接状态变为:集群健康值：green(0 of 0)  了
- *green* 代表健康状态, 
- *yellow* 代表集群健康不是很好但可正常用, 
- *red* 代表集群健康已经很差了，可以搜索数据，但是会出现丢失数据的问题

# 三、分布式安装
为了防止elasticsearch服务崩溃，我们将进行分布式部署。当一台服务器崩溃时，由其他备用服务器进行服务，保证elasticsearch的高可用。
### 1. 修改elasticsearch配置
进入elasticsearch目录下，修改config/elasticsearch.yml文件：
```
cluster.name: my-cluster 
node.name: my-master-application
node.master: true
network.host: 127.0.0.1
```
- cluster.name:   设置集群名字
- node.name:   设置节点名字
- node.master: 指定为master
- network.host:  设置绑定IP，端口不修改默认9200

### 2. 重启elasticsearch
进入elasticsearch的bin目录，运行elasticsearch.bat文件,打开head插件127.0.0.1:9100查看是否正常启动
你将看到一个master，五角星图标，代表是master节点

### 3.新建从节点
新建一个文件夹es_slave,将最初下载的elasticsearch压缩包拷贝两份放入其中，分别解压es_slave1和es_slave2

### 4.修改从节点es_slave1和es_slave2的配置文件
分别修改es_slave1和es_slave2目录下config/elasticsearch.yml文件  
es_slave1的config/elasticsearch.yml文件结尾加上：
```
cluster.name: my-cluster 
node.name: my-slave1-application
network.host: 127.0.0.1
http.port: 8200
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```
es_slave2的config/elasticsearch.yml文件结尾加上：
```
cluster.name: my-cluster 
node.name: my-slave2-application
network.host: 127.0.0.1
http.port: 8300
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```
*discovery.zen.ping.unicast.hosts属性配置了将可以找到master，不加这个是游离于集群之外的，找不到master指挥官*

### 5.启动从节点es_slave1
进入es_slave1目录执行./bin/elasticsearch -d命令,打开127.0.0.1:9200
将看到除了一个五角星图标的master节点之外，还多了一个名为my-slave1-application的随从节点

### 6.启动从节点es_slave2
进入es_slave2目录执行./bin/elasticsearch -d命令,打开127.0.0.1:9200
将看到除了一个五角星图标的master节点和随从节点slave1之外，还多了一个名为my-slave2-application的随从节点
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/5distributed-installed-page.png)

# 四、总结
总结一下，本文介绍内容分为三部分
1. elasticsearch的安装运行
2. 视化插件elasticsearch-head的安装运行
3. elasticsearch的分布式环境搭建

好了，elasticsearch环境到此就搭建完成了。下期将继续介绍elasticsearch的使用。