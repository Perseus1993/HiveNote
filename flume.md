#Flume
日志采集，聚合和传输系统
拿到Java EE平台的日志数据（实时推荐）
主要目的：实时读取服务器日志，写到HDFS中

####架构
agent:JVM进程。以事件（event）的形式传送到目的地

event : header(kv结构) + body（字节数组）

source: 生产事件

sink: 消费事件

channel:缓冲区 线程安全可以对接多个source sink   可以在内存或者磁盘中

Avro source: 对接多个flume
Exec source

####flume安装


type
一个source可以有多个channel
一个sink只能一个channel

hdfs.rollInterval 滚动生成新文件 默认30秒
hdfs.rollsize
hdfs.rollcount
hdfs.round 文件夹滚动
