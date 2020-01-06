#Yarn JOB提交
<img src ='/img/hadoop13.png'>

1.客户端向ResourceManagement 提交 运行的请求 (hadoop jar xxxx.jar)

2.ResourceManager进行检查,没有问题的时候,向客户端返回一个共享资源的路径以及JobId

3.客户端向HDFS提交资源,将共享资源放入共享路径下:(/tmp/hadoop-yarn/staging-dir/xxxxxxxx)

4.客户端向ResourceManager反馈共享资源放置完毕,进行job的正式提交

5.ResourceManager为这个job分配一个节点,并在这个节点上启动MRAppMaster任务

6.ResourceManager到对应的节点上去启动container容器用于装载MRAppMaster

7.MRAppMaster对job进行初始化,生成一个job工作簿,job的工作簿记录着maptask和reduce的运行进度和状态

8.MRAppMaster向ResourceManager申请maptask和reducetask的运行的资源,先发maptask然后发reducetask

9.ResourceManager向MRAppMaster返回maptask和reduce的资源节点

####  Yarn的默认调度器、调度器分类、以及他们之间的区别

1）Hadoop调度器主要分为三类：

FIFO 、Capacity Scheduler（容量调度器）和Fair Sceduler（公平调度器）。

Hadoop2.7.2默认的资源调度器是 容量调度器

FIFO调度器：先进先出，同一时间队列中只有一个任务在执行。

容量调度器：多队列；每个队列内部先进先出，同一时间队列中只有一个任务在执行。队列的并行度为队列的个数

公平调度器：多队列；每个队列内部按照缺额大小分配资源启动任务，同一时间队列中有多个任务执行。队列的并行度大于等于队列的个数。
