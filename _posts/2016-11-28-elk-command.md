---
title: ELK命令
categories:
 - 架构设计
tags:
 - elk
---

### 简介
ELK由Elasticsearch、Logstash和Kibana三部分组件组成；

Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

Logstash是一个完全开源的工具，它可以对你的日志进行收集、分析，并将其存储供以后使用

kibana 是一个开源和免费的工具，它可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

[ELK 日志分析系统](http://467754239.blog.51cto.com/4878013/1700828)


### 开启logstash命令：
```bash
$ sudo /usr/local/nginx/sbin/nginx
```

#### -d 表示以daemon的方式启动
```bash
$ /usr/local/elasticsearch-5.0.1/bin/elasticsearch -d
```

#### 或者(下面那条可输出错误日志)
```bash
$ nohup /usr/local/elasticsearch-5.0.1/bin/elasticsearch > /home/johnzhang/logs/elasticsearch.log 2>&1 &
```

#### 检查连接是否成功
```bash
$ curl -X GET http://localhost:9200
$ netstat -tnlp |grep java
```

#### 启动logstash
- -f 配置文件路径
- nohup 将任务放到后台，但是依然可以使用标准输入，前台能够接收任何输入，重定向标准输出和标准错误到当前目
- 录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。
```bash
$ nohup sudo /usr/local/logstash-5.0.1/bin/logstash -f /usr/local/logstash-5.0.1/etc/logstash_agent.conf > /home/johnzhang/logs/logstash.log 2>&1 &

$ curl 'http://localhost:9200/_search?pretty'

$ nohup /usr/local/kibana-5.0.1-linux-x86_64/bin/kibana > /home/johnzhang/logs/kibana.log 2>&1 &
```

### 服务器crontab命令
```bash
22 * * * * /home/johnzhang/php-projects/test/cleanlog/del-all-old-logs.sh >> /home/johnzhang/logs/root-del-old-logs-cron.log 2>&1
```

### 服务器logstash开启命令
```bash
/elk/logstash-5.0.0/bin/logstash -f /elk/logstash-5.0.0/conf/index.conf -l /elk/logstash-5.0.0/logs/stdout.log --verbose &
```

### 停止logstash
- 在运行 logstash 的服务器上，输入以下命令：
```bash
ps -ef | grep logstash
```
这将显示 logstash 进程标识。

- 输入以下命令：
```bash
Kill <process_id>
```
