#集群配置
<hr style="height:1px;border:none;border-top:1px solid #555555;" />

在搭建分布式i群之前有几个需要做的
<ul>
<li>IP地址修改</li>
<li>主机名修改</li>
<li>关闭防火墙</li>
<li>配置ssh免密登录</li>
<li>分发脚本</li>
<li>安装JDK</li>
</ul>

如果是用虚拟机，可以直接新建虚拟机或者使用虚拟机克隆

#### 配置ssh免密登录

进入到家目录下面的.ssh

在namenode机器（hadoop102）生成密钥
`ssh-keygen -t rsa`

传给各个主机
```
ssh-copy-key hadoop102
ssh-copy-key hadoop103
ssh-copy-key hadoop104
```
在yarn机器(hadoop103)也得生成密钥，然后传给其他机器，因为yarn要连接其他机器


#### 分发脚本
在 /home/user/bin 下面新建xsync文件

```shell
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for((host=103; host<105; host++)); do
        echo ------------------- hadoop$host --------------
        rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
done
```

`chmod 777 xsync`

#### 安装JDK
在 /opt目录下新建 software和module文件夹，前者放安装包，后者放解压后的软件，记得修改这两个文件夹的用户权限

`sudo chown 用户组：用户名  文件`

将JDK压缩包传入虚拟机

解压到 /opt/module中
`tar -zxvf jar包 -C /opt/module`

添加环境变量

`sudo vim /etc/profile`

编辑文档

```
JAVA_HOME=/opt/module/jdk1.8.0_144
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME
```
记得`source /etc/profile`
