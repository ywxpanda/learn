# flume日志收集框架

## 一 概述

    Flume是一种分布式，可靠且可用的服务，用于有效地收集，聚合和移动大量日志数据。它具有基于流数据流的简单灵活的架构。它具有可靠的可靠性机制和许多故障转移和恢复机制，具有强大的容错能力。它使用简单的可扩展数据模型，允许在线分析应用程序。

### 1.1  设计目标

    1)  可靠性
    2)  扩展性

## 二 核心组件和架构

    Source:收集
    Channel:聚集
    Sink:输出

### 安装flume

    jdk安装
    flume安装：在flume-env.sh中配置JAVA_HOME

### 简单使用

    1) 写配置文件
    配置source
    配置Channel
    配置sink
    将三个组件串联起来

#### 1.监听某个端口并输出日志

    # 定义三个组件的名称
    a1.sources = r1
    a1.sinks = k1
    a1.channels = c1

    # 指定采集器的收集方式，并绑定hostname和port
    a1.sources.r1.type = netcat
    a1.sources.r1.bind = hadoop000
    a1.sources.r1.port = 44444

    # 配置输出方式,离线时可以sink到hdfs,实时sink到kafka,或者sink到其他的source
    a1.sinks.k1.type = logger

    # 配置聚集的方式channel:内存
    a1.channels.c1.type = memory

    # 将三者进行绑定，一个source可输出多个channel，一个channel只能输出到一个sink
    a1.sources.r1.channels = c1
    a1.sinks.k1.channel = c1

##### 启动

    $FLUME_HOME/bin/flume-ng agent --name a1 --conf  $FLUME_HOME/conf --conf-file example.conf -Dflume.root.logger=INFO,console
    测试L:telnet hadoop000:44444
    输出结果：Event: { headers:{} body: 68 65 6C 6C 6F 0D  hello. }
    一条结果就是一个event

#### 2.监控一个文件实时采集新增的数据输出到控制台

    # Name the components on this agent
    a1.sources = r1
    a1.sinks = k1
    a1.channels = c1

    # Describe/configure the source
    a1.sources.r1.type = exec
    a1.sources.r1.command = tail -F /home/hadoop/data/data.log
    a1.sources.r1.shell = /bin/sh -c

    # Describe the sink
    a1.sinks.k1.type = logger

    # Use a channel which buffers events in memory
    a1.channels.c1.type = memory

##### 执行

    $FLUME_HOME/bin/flume-ng agent --name a1 --conf  $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/exec-memory-logger.conf -Dflume.root.logger=INFO,console

#### 3.将A服务器的日志实时采集存放到B服务器上

##### 3.1 exec-memory-avro.conf 将A服务器中的数据采集（文件）

    exec-memory-avro.sources = exec-source
    exec-memory-avro.sinks = avro-sink
    exec-memory-avro.channels = memory-channel

    exec-memory-avro.sources.exec-source.type = exec
    exec-memory-avro.sources.exec-source.command = tail -F /home/hadoop/data/data.log
    exec-memory-avro.sources.exec-source.shell = /bin/sh -c

    exec-memory-avro.sinks.avro-sink.type = avro
    exec-memory-avro.sinks.avro-sink.hostname = hadoop000
    exec-memory-avro.sinks.avro-sink.port = 44444

    exec-memory-avro.channels.memory-channel.type = memory

    exec-memory-avro.sources.exec-source.channels = memory-channel
    exec-memory-avro.sinks.avro-sink.channel = memory-channel

##### 3.2 avro-memory-logger 将A服务器的数据采集并输出到控制台

    avro-memory-logger.sources = avro-source
    avro-memory-logger.sinks = logger-sink
    avro-memory-logger.channels = memory-channel

    avro-memory-logger.sources.avro-source.type = avro
    avro-memory-logger.sources.avro-source.bind = hadoop000
    avro-memory-logger.sources.avro-source.port = 44444

    avro-memory-logger.sinks.logger-sink.type = logger

    avro-memory-logger.channels.memory-channel.type = memory

    avro-memory-logger.sources.avro-source.channels = memory-channel
    avro-memory-logger.sinks.logger-sink.channel = memory-channel

##### 3.3 启动 先启动输出到控制台的数据

    $FLUME_HOME/bin/flume-ng agent --name avro-memory-logger --conf  $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/avro-memory-logger.conf -Dflume.root.logger=INFO,console

    $FLUME_HOME/bin/flume-ng agent --name exec-memory-avro --conf  $FLUME_HOME/conf --conf-file $FLUME_HOME/conf/exec-memory-avro.conf -Dflume.root.logger=INFO,console