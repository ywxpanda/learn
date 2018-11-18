# Kafka

    kafka.apache.org

## 概述

    Kafka用于构建实时数据管道和流应用程序。它具有水平可扩展性，容错性，快速性，并在数千家公司的生产中运行。

## 架构和核心

    producer:生产者
    consumer:消费者
    broker:容器
    topic:主题（不同的消费者消费不同的topic）

## 安装部署

### zookeeper安装

    1.配置环境变量
    2.bin目录下
        cp zoo_sample.cfg zoo.cfg
        编辑zoo.cfg，更改dataDir更改kafka文件存放的目录
    3.启动
        ./zkServer.sh
        jps:QuorumPeerMain
    4.连接
    ./zkCli.sh

### kafka安装

    下载地址：http://kafka.apache.org/downloads ,选择对应scala的版本
    1.配置环境变量
    2.server.properties
        broker.id:不能重复，对应每一个kafka节点
        listeners:监听端口
        hostname:启动的服务器的地址
        log.dirs:配置日志生成的位置（默认tmp下，在服务器重启后会被清除）
        num.partitions:配置分区的数量
        zookeeper.connection:配置zookeeper的连接地址
    3.启动
        kafka-server-start.sh /**/servers.properties
    4.创建topic(zookeeper)
        指定zookeeper的地址，指定副本系数，指定分区数，指定topic的名称
        kafka-topic.sh --create --zookeeper ***:2181 --replication-factor 1 --partitions 1 --topic ***
    5.查看所有的topic(zookeeper)
        kafka-topic.sh --list --zookeeper ***:2181
    6.生产消息(broker)
        指定broker的监听位置
        kafka-console-consumer.sh --broker-list ***:9092 --topic ***
    7.消费消息
        kafka-console-consumer.sh --zookeeper localhost:2181 --topic *** --from-beginning
        from-beginning:从头开始消费
    8.查看topic的信息
        kafka-topics.sh --describe --zookeeper ***:2181 --topic ***
    9.多broker配置
        修改server.properties：broker.id,listeners,logs.dir
        注：使用同一个zookeeper
        a) 启动kafka
         kafka-server-start.sh /**/servers-1.properties
         kafka-server-start.sh /**/servers-2.properties
         kafka-server-start.sh /**/servers-3.properties
        b) 创建topic
            一个分区三个副本
            kafka-topic.sh --create --zookeeper ***:2181 --replication-factor 3 --partitions 1 --topic ***
        c) 查看信息
            kafka-topics.sh --describe --zookeeper hadoop000:2181 --topic ***
        d) 生产消费
            kafka-console-producer.sh --broker-list hadoop000:9092,....(多个) --topic helloopic
            kafka-console-consumer.sh --zookeeper hadoop000:2181 --topic ***
             --from-beginning
