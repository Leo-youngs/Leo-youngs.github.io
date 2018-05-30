---
title: kafka
date: 2018-05-30 10:52:58
tags: 消息队列 greenplum 
---
## Kafka安装配置
这里只用到 kafka 的基本功能，所以演示单机安装
系统： **Ubuntu 18.04**

### 安装配置 ZooKeeper
1. 下载 配置
``` bash
# 打开 zoo.cfg 可以配置端口，目录等
$ wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
$ tar zxvf zookeeper-3.4.6.tar.gz
$ cd zookeeper-3.4.6
$ cp -rf conf/zoo_sample.cfg conf/zoo.cfg
$ vim conf/zoo.cfg
```
2. 启动
``` bash 
$ sh bin/zkServer.sh start
```

### 安装配置 Kafka
1. 下载 配置 (需要修改 `server.properties zookeeper.connect=localhost:2181`)
``` bash
# 打开 zoo.cfg 可以配置端口，目录等
$ wget http://apache.fayea.com/kafka/0.8.2.1/kafka_2.10-0.8.2.1.tgz
$ tar -zxf kafka_2.10-0.8.2.1.tgz
$ cd kafka_2.10-0.8.2.1
$ vi config/server.properties

```
2. 启动
``` bash 
sh bin/kafka-server-start.sh config/server.properties
```
3. 命令简介

```bash 
# 创建Topic

$ sh bin/kafka-topics --create --topic kafkatopic --replication-factor 1 --partitions 1 --zookeeper localhost:2181

# 查看Topic
$ sh bin/kafka-topics --list --zookeeper localhost:2181

# 启动Producer 生产消息
$ sh bin/kafka-console-producer --broker-list localhost:9092 --topic kafkatopic

# 启动Consumer 消费消息
$ sh bin/kafka-console-consumer --zookeeper localhost:2181 --topic kafkatopic --from-beginning

# 删除Topic
$ sh bin/kafka-run-class kafka.admin.TopicCommand --delete --topic kafkatopic --zookeeper localhost:2181

# 查看Topic 的offset 
$ sh bin/kafka-consumer-offset-checker  --zookeeper localhost:2181 --topic kafkatopic --group consumer

```

### KafkaOffsetMonitor 安装配置 
1. 下载   https://github.com/quantifind/KafkaOffsetMonitor/releases/tag/v0.2.0
2. 启动 
```bash
java -cp KafkaOffsetMonitor-assembly-0.2.0.jar \
     com.quantifind.kafka.offsetapp.OffsetGetterWeb \
     --zk loaclhost:2181 \
     --port 8089 \
     --refresh 10.seconds \
     --retain 2.days

# 可以用 nohup & 启动
```

### Kafka-manager

1. 下载   https://github.com/yahoo/kafka-manager

2. 安装配置

```bash 

# 默认配置java 环境

git clone https://github.com/yahoo/kafka-manager/releases

cd kafka-manager

./sbt clean dist

# 命令执行完成后，在 target/universal 目录中会生产一个zip压缩包kafka-manager-1.3.3.7.zip

cd target/universal 

unzip kafka-manager-1.3.3.7.zip


# 配置 kafka-manager.zkhosts="localhost:2181"
vi conf/application.conf
```
3. 这里如果想看见更多信息需要额外配置
```bash

# 修改 KAFKA_JMX_OPTS
vi kafka-run-class.sh

  KAFKA_JMX_OPTS="-Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false"
  JMX_PORT=9999

# 添加 JMX_PORT
vi kafka-server-start.sh

export JMX_PORT=${JMX_PORT:-9999

```

4. 启动 
```bash
bin/kafka-manager
# 可以用 nohup & 启动
```
