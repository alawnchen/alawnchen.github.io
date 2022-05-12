---
title: 在centos7上面安装Kafka
date: 2018-01-03 07:03:00
tags: 
  - kafka
  - 队列
categories:
  - 技术
url: install-kafka-on-centos-7
---

## 简介

Kafka是一种高吞吐的分布式发布订阅消息系统，能够替代传统的消息队列用于解耦合数据处理，缓存未处理消息等，同时具有更高的吞吐率，支持分区、多副本、冗余，因此被广泛用于大规模消息数据处理应用。Kafka
支持Java 及多种其它语言客户端，可与Hadoop、Storm、Spark等其它大数据工具结合使用。

<!--more-->

本教程主要介绍Kafka 在Centos 7上的安装和使用，包括功能验证和集群的简单配置。

## 安装JDK

Kafka 使用Zookeeper 来保存相关配置信息，Kafka及Zookeeper 依赖Java 运行环境，从oracle网站下载JDK 安装包，解压安装：

```
tar zxvf jdk-8u65-linux-x64.tar.gz
mv jdk1.8.0_65 java
```

设置Java 环境变量：

```
JAVA_HOME=/opt/java
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME PATH
```

也可以选择yum install安装，相应设置环境变量。

## 安装Kafka

从官网下载Kafka 安装包，解压安装：

```
tar zxvf kafka_2.11-0.8.2.2.tgz
mv kafka_2.11-0.8.2.2 kafka
cd kafka
```

## 功能验证

1.启动Zookeeper

使用安装包中的脚本启动单节点Zookeeper 实例：

```
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
[2015-10-26 04:26:59,585] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)....
```

2.启动Kafka 服务

使用kafka-server-start.sh 启动kafka 服务：

```
bin/kafka-server-start.sh config/server.properties
[2015-10-26 04:28:56,115] INFO Verifying properties (kafka.utils.VerifiableProperties) [2015-10-26 04:28:56,141] INFO Property broker.id is overridden to 0 (kafka.utils.VerifiableProperties)
```

3.创建topic

使用kafka-topics.sh 创建单分区单副本的topic test：

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```


4. 查看topic：

```
bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```


5. 产生消息

使用kafka-console-producer.sh 发送消息：

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
Hello world！ Hello Kafka
```

6. 消费消息

使用kafka-console-consumer.sh 接收消息并在终端打印：

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

## 集群配置

 ### 单机多broker 集群配置

利用单节点部署多个broker。 不同的broker 设置不同的 id，监听端口及日志目录。 例如：

```
cp config/server.properties config/server-1.properties
```

编辑配置：

```
config/server-1.properties:
    broker.id=1
    port=9093
    log.dir=/tmp/kafka-logs-1
```

启动Kafka服务：

```
bin/kafka-server-start.sh config/server-1.properties &
```

启动多个服务，按上文类似方式产生和消费消息。

### 多机多broker 集群配置

分别在多个节点按上述方式安装Kafka，配置启动多个Zookeeper 实例。 例如：
在10.4.253.22，10.4.253.23，10.4.253.24三台机器部署，Zookeeper配置如下：

```
initLimit=5
syncLimit=2
server.1=10.4.253.22:2888:3888
server.2=10.4.253.23:2888:3888
server.3=10.4.253.24:2888:3888
```

分别配置多个机器上的Kafka服务 设置不同的broke id，zookeeper.connect设置如下:

```
zookeeper.connect=10.4.253.22:2181,10.4.253.23:2181,10.4.253.24:2181
```

启动Zookeeper与Kafka服务，按上文方式产生和消费消息，验证集群功能。