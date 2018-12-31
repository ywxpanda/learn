# flume整合spark streaming

## push方式整合

### flume Agent

    # 配置netcat source
    simple-agent.sources = netcat-source
    simple-agent.sinks = avro-sink
    simple-agent.channels = memory-channel

    # 从hadoop000的44444端口发送数据
    simple-agent.sources.netcat-source.type = netcat
    simple-agent.sources.netcat-source.bind = hadoop000
    simple-agent.sources.netcat-source.port = 44444

    # hadoop000的41414接受数据
    simple-agent.sinks.avro-sink.type = avro
    simple-agent.sinks.avro-sink.hostname = hadoop000
    simple-agent.sinks.avro-sink.port = 41414

    # 配置channel为内存
    simple-agent.channels.memory-channel.type = memory

    # 绑定三者
    simple-agent.sources.netcat-source.channels = memory-channel
    simple-agent.sinks.avro-sink.channel = memory-channel

### 启动

    $FLUME_HOME/bin/flume-ng agent --name a1 --conf  $FLUME_HOME/conf --conf-file flume_push_streaming.conf -Dflume.root.logger=INFO,console