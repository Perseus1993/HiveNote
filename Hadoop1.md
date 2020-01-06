# Hadoop安装

#### Hadoop安装包传到虚拟机

`tar -zxvf hadoop-2.7.2.tar.gz -C /opt/module/`

添加环境变量
`PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

HADOOP_HOME=/opt/module/hadoop-2.7.2

export PATH HADOOP_HOME `

#### 完全分布式搭建

<img src = 'img/hadoop1.png'/>

编辑core-site.xml文件

```
<!-- 指定HDFS中NameNode的地址 -->
<property>
		<name>fs.defaultFS</name>
      <value>hdfs://hadoop102:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>
```

编辑hadoop-env.sh
`export JAVA_HOME=/opt/module/jdk1.8.0_144`

编辑hdfs-site.xml,指定副本数和2nn
```
<property>
		<name>dfs.replication</name>
		<value>1</value>
</property>

<!-- 指定Hadoop辅助名称节点主机配置 -->
<property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop104:50090</value>
</property>
```

编辑yarn-env.sh

`export JAVA_HOME=/opt/module/jdk1.8.0_144`

编辑yarn-site.xml文件

```
<!-- Reducer获取数据的方式 -->
<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop103</value>
</property>
```

编辑 mapred-env.sh

`export JAVA_HOME=/opt/module/jdk1.8.0_144`

编辑 mapred-site.xml文件

```
<!-- 指定MR运行在Yarn上 -->
<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
</property>
```

编辑 slaves
把集群的host名都添加进去

最后hadoop文件分发到集群

分发 /etc/profile

最后格式化一下namenode
`bin/hdfs namenode -format`
