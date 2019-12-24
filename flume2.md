#flume进阶
<hr style="height:1px;border:none;border-top:1px solid #555555;" />

<img src = 'img/flume3.png'/>
:herb: put事务流程
doPut将数据先写入临时缓冲区,临时缓冲区putlist把source采集的数据，提交到channel，等待chennel答复，如果不足，回滚

:herb: take事务
临时缓冲区takelist：toTake把事务从channel拉过来，doCommit发送成功，清除缓存区，否则drollback

#flume agent内部原理
<hr style="height:1px;border:none;border-top:1px solid #555555;" />
<img src = 'img/flume4.png'/>
:herb: ChannelSelector<br>
作用就是选出Event将要被发往哪个Channel。其共有两种类型，分别是Replicating（复制）和Multiplexing（多路复用）。ReplicatingSelector会将同一个Event发往所有的Channel，Multiplexing会根据相应的原则，将不同的Event发往不同的Channel。

:herb: interception链<br>
interception独立于 channel_processor是因为筛选逻辑经常会变（可插拔功能）

:herb: sink processor<br>
SinkProcessor共有三种类型，分别是DefaultSinkProcessor、LoadBalancingSinkProcessor和FailoverSinkProcessor
DefaultSinkProcessor对应的是单个的Sink，LoadBalancingSinkProcessor和FailoverSinkProcessor对应的是Sink Group，LoadBalancingSinkProcessor可以实现负载均衡的功能，FailoverSinkProcessor可以实现故障转移的功能。


#flume拓扑结构
<hr style="height:1px;border:none;border-top:1px solid #555555;" />

:herb: 简单串联<br>
<img src = 'img/flume5.png'/>
前面是客户端 后面是服务端（先启动服务端，再启动服务端），这样风险会变大，相连的部分需要用avro类型的sink和source。这种模式是将多个flume顺序连接起来了，从最初的source开始到最终sink传送的目的存储系统。此模式不建议桥接过多的flume数量，flume数量过多不仅会影响传输速率，而且一旦传输过程中某个节点flume宕机，会影响整个传输系统。

:herb: 复制和多路复用<br>
<img src = 'img/flume6.png'/>
Flume支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个channel中，或者将不同数据分发到不同的channel中，sink可以选择传送到不同的目的地。

:herb: 负载均衡和故障转移
<img src = 'img/flume7.png'/>  
Flume支持使用将多个sink逻辑上分到一个sink组，sink组配合不同的SinkProcessor可以实现负载均衡和错误恢复的功能。注意这里说的故障是针对后面三个，如果前面的挂了就全完了。

:herb: 聚合
<img src = 'img/flume8.png'/>
最常见实用，web应用通常分布在上百个服务器。产生的日志，处理起来也非常麻烦。用flume的这种组合方式能很好的解决这一问题，每台服务器部署一个flume采集日志，传送到一个集中收集日志的flume，再由此flume上传到hdfs、hive、hbase等，进行日志分析。
