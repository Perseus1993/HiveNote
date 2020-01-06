# Hadoop面试题目

#### 常用端口号
<ul>
<li>dfs.namenode.http-address:50070</li>
<li>dfs.datanode.http-address:50075</li>
<li>SecondaryNameNode:50090</li>
<li>dfs.datanode.address:50010</li>
<li>fs.defaultFS:8020或者9000</li>
<li>yarn.resourcemanager.webapp.address:8088</li>
<li>历史服务器web访问端口：19888</li>
</ul>

# Hadoop配置文件以及简单的集群搭建
#### 配置文件
core-site.xml, hdfs-site.xml, mapred-site.xml, yarn-site.xml

hadoop-env.sh, yarn-env.sh, mapred-env.sh, slaves

#### 简单的集群搭建

JDK安装

配置ssh免密登录

配置hadoop核心文件

格式化namenode

# Hadoop读流程以及写流程
