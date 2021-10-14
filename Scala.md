# Zookeeper

##### 集群规划

在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。

##### 分布式安装部署

###### 解压——安装

```
[ahua@hadoop102 zookeeper]$ tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz -C /opt/module/
```

###### 配置服务器编号

- 在/opt/module/apache-zookeeper-3.6.2-bin/目录下创建zkData文件夹

```
[ahua@hadoop102 apache-zookeeper-3.6.2-bin]$ mkdir zkData
```

- 在zkData文件夹下创建一个myid文件，并添加内容：2

```
[ahua@hadoop102 apache-zookeeper-3.6.2-bin]$ cd zkData/
[ahua@hadoop102 zkData]$ vim myid
添加内容：2
```

- 分发Zookeeper安装包

```
[ahua@hadoop102 module]$ xsync apache-zookeeper-3.6.2-bin/
```

> 分发完毕后，分别在hadoop103、hadoop104上修改myid文件中内容为3、 4

###### 配置zoo.cfg文件

- 重命名/opt/module/apache-zookeeper-3.6.2-bin/conf/这个目录下的zoo_sample.cfg为zoo.cfg

```shell
[ahua@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg
```

- 修改zoo.cfg配置文件

```
[ahua@hadoop103 conf]$ vim zoo.cfg

#修改数据存储路径配置
dataDir=/opt/module/apache-zookeeper-3.6.2-bin/zkData

#末尾添加如下配置
###########cluster###########
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

- 同步zoo.cfg配置文件

```shell
[ahua@hadoop102 conf]$ xsync zoo.cfg
```

###### 集群操作

- 分别启动Zookeeper

```shell
[ahua@hadoop102 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start
[ahua@hadoop103 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start
[ahua@hadoop104 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start
```

- 查看状态

```shell
[ahua@hadoop102 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

[ahua@hadoop103 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader

[ahua@hadoop104 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```



------

# Flume

##### 安装部署

- 将apache-flume-1.9.0-bin.tar.gz上传到/opt/software/目录下
- 解压apache-flume-1.9.0-bin.tar.gz到/opt/module/目录下

```
[ahua@hadoop102 software]$ tar -zxvf apache-flume-1.9.0-bin.tar.gz -C /opt/module/
```

- 将/opt/module/apache-flume-1.9.0-bin/lib/文件夹下的guava-11.0.2.jar删除以兼容hadoop-3.2.2

```
[ahua@hadoop102 lib]$ rm /opt/module/apache-flume-1.9.0-bin/lib/guava-11.0.2.jar
```

- 将/opt/module/apache-flume-1.9.0-bin/conf/目录下的配置文件flume-env.sh.template修改为flume-env.sh

```
[ahua@hadoop102 conf]$ mv flume-env.sh.template flume-env.sh
```

- 在flume-env.sh中配置$JAVA_HOME环境变量位置，可通过echo $JAVA_HOME/获取

```
[ahua@hadoop102 conf]$ vim flume-env.sh
```

- flume-env.sh文件中添加：

```
export JAVA_HOME=/opt/module/jdk1.8.0_271/
```

- 分发flume

```
xsync /opt/module/apache-flume-1.9.0-bin/
```

------

##### **入门案例**

###### 1.监控端口数据官方案例

`案例需求:使用Flume监听一个端口，收集该端口数据，并打印到控制台。`

1.安装netcat工具

```
[ahua@hadoop102 ~]$ sudo yum install -y nc
```

2.连接示例：

- hadoop102上执行：

```
[ahua@hadoop102 ~]$ nc -lk 44444
开启服务端
```

- hadoop103上执行：

```
[ahua@hadoop103 ~]$ nc hadoop102 44444
开启客户端，连接服务端
```

> 然后可以尝试在任一窗口发送数据，另一窗口会接收到数据。

3.判断44444端口是否被占用，被占用时显示如下：

```bash
[ahua@hadoop102 ~]$ sudo netstat -tunlp | grep 44444
tcp        0      0 0.0.0.0:44444           0.0.0.0:*               LISTEN      1562/nc             
tcp6       0      0 :::44444                :::*                    LISTEN      1562/nc
```

4.创建Flume Agent配置文件flume-netcat-logger.conf

​	在apache-flume-1.9.0-bin目录下创建job文件夹并进入job文件夹

```
[ahua@hadoop102 apache-flume-1.9.0-bin]$ mkdir job
[ahua@hadoop102 apache-flume-1.9.0-bin]$ cd job/
```

​	在job文件夹下创建Flume Agent配置文件flume-netcat-logger.conf

```
[ahua@hadoop102 job]$ vim flume-netcat-logger.conf
```

​	在flume-netcat-logger.conf配置文件中添加如下内容：

```xml
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

> 此配置文件内容来源于Flume官方手册
>
> [Flume 1.9.0 User Guide]( https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html)

5.先开启Flume监听端口（在apache-flume-1.9.0-bin目录下）

​	第一种写法：

```
[ahua@hadoop102 apache-flume-1.9.0-bin]$ bin/flume-ng agent --conf conf/ --conf-file job/flume-netcat-logger.conf --name a1 -Dflume.root.logger=INFO,console
```

​	第二种写法（推荐，-c-n-f顺序可变）：

```
[ahua@hadoop102 apache-flume-1.9.0-bin]$ bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

```
参数说明：
--conf/-c：表示配置文件存储在conf/目录
--name/-n：表示给agent起名为a1
--conf-file/-f：flume本次启动读取的配置文件是在job文件夹下的flume-netcat-logger.conf文件
-Dflume.root.logger=INFO,console：-D 表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括：log、info、warn、error。
```

6.新开一个hadoop102远程连接窗口，使用netcat工具与本机localhost的44444端口建立连接（作为客户端）并发送数据

```
[ahua@hadoop102 ~]$ nc localhost 44444
hello
ahua
```

7.在Flume端监听页面观察接收数据情况

```
2021-03-19 15:01:33,910 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F                                  hello }
2021-03-19 15:01:34,679 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 61 68 75 61                                     ahua }
```

###### 2.实时监控单个追加文件

`案例需求：实时监控Hive日志，并上传到HDFS中`

file-flume-logger.conf

```
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -f /opt/module/hive/logs/hive.log-------------

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

file-flume-hdfs.conf

```
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log----------

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9000/flume/%Y%m%d/%H-----------
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 1000
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 30
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k2.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

###### 3.实时监控目录下多个新文件



###### 4.实时监控目录下的多个追加文件







------

**总结**

需要修改配置的文件

1.将/opt/module/apache-flume-1.9.0-bin/conf/目录下的配置文件flume-env.sh.template修改为flume-env.sh

阿里云离线数仓传输日志

测试：输出到控制台

flume/jobs/test_ali_taildir_memory_logger.conf

```
# 定义组件名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

###source 部分
a1.sources.r1.type = TAILDIR
#记录偏移量实现断点续传
a1.sources.r1.positionFile =/opt/module/apache-flume-1.9.0-bin/jobs/taildir_position/test_ali_taildir_position.json
a1.sources.r1.channels = c1
a1.sources.r1.filegroups= f1
a1.sources.r1.filegroups.f1 = /opt/module/logs/app.+
a1.sources.r1.fileHeader = true

###channel 部分
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 1000

###sink 部分 先输出到控制台中
a1.sinks.k1.type = logger

# 把 source 和 sink 绑定到 channel 中
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

> /opt/module/apache-flume-1.9.0-bin/bin/flume-ng agent -n a1 -c /opt/module/apache-flume-1.9.0-bin/conf/ -f /opt/module/apache-flume-1.9.0-bin/jobs/test_ali_taildir_memory_logger.conf -Dflume.root.logger=info,console

输出到DataHub

flume/jobs/ali_taildir_memory_datahub.conf

```
# 定义组件名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

### source 部分
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile =/opt/module/apache-flume-1.9.0-bin/jobs/taildir_position/ali_taildir_position.json
a1.sources.r1.channels = c1
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/logs/app.+
a1.sources.r1.fileHeader = true

### channel 部分
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 1000

### sink 部分
a1.sinks.k1.type = com.aliyun.datahub.flume.sink.DatahubSink a1.sinks.k1.datahub.accessID = 
LTAI4FiU71dZAL17SdLBa6Nt a1.sinks.k1.datahub.accessKey = 63YzSmqMOSjDR5A2ZXEzFLM2tREY6m 
a1.sinks.k1.datahub.endPoint  =  http://dh-cn-hangzhou.aliyun-inc.com
a1.sinks.k1.datahub.project = gmall_datahub
a1.sinks.k1.datahub.topic = base_log 
a1.sinks.k1.batchSize = 100
a1.sinks.k1.serializer = DELIMITED
a1.sinks.k1.serializer.delimiter = "\\u007C"
a1.sinks.k1.serializer.fieldnames = event_time,log_string a1.sinks.k1.serializer.charset = UTF-8 
a1.sinks.k1.shard.number = 1
a1.sinks.k1.shard.maxTimeOut = 60

# 把 source 和 sink 绑定到 channel 中
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```





------

# Kafka

##### 安装部署

- 将kafka_2.12-2.7.0.tgz上传到/opt/software/目录下
- 解压kafka_2.12-2.7.0.tgz到/opt/module/目录下

```
[ahua@hadoop102 software]$ tar -zxvf kafka_2.12-2.7.0.tgz -C /opt/module/
```

- 在/opt/module/kafka_2.12-2.7.0/目录下创建logs文件夹

```
[ahua@hadoop102 kafka_2.12-2.7.0]$ mkdir logs
```

- 修改配置文件server.properties

```
[ahua@hadoop102 kafka_2.12-2.7.0]$ cd config/
[ahua@hadoop102 config]$ vim server.properties
```

- 修改以下内容:


```
# broker 的全局唯一编号，不能重复
broker.id=0

# 删除 topic 功能使能
delete.topic.enable=true

# kafka 数据存放的路径
log.dirs=/opt/module/kafka_2.12-2.7.0/data

# 配置连接 Zookeeper 集群地址 默认为localhost:2181
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

> #删除 topic 功能使能
> delete.topic.enable=true
>
> 此项有待考证

- 分发Kafka安装包

```
[ahua@hadoop102 module]$ xsync kafka_2.12-2.7.0/
```

> 分发完毕后，分别在hadoop103、hadoop104上修改server.properties文件中的broker.id，分别为1、2，broker.id不能重复

- 配置环境变量

```
[ahua@hadoop102 module]$ sudo vim /etc/profile.d/my_env.sh

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka_2.12-2.7.0
export PATH=$PATH:$KAFKA_HOME/bin

[ahua@hadoop102 module]$ source /etc/profile
```

- 在hadoop102上分发my_env.sh，在hadoop103、hadoop104上source，使环境变量生效

```
[ahua@hadoop102 module]$ sudo xsync /etc/profile.d/my_env.sh

[ahua@hadoop103 kafka_2.13-2.7.0]$ source /etc/profile
[ahua@hadoop104 kafka_2.13-2.7.0]$ source /etc/profile
```

- 启动/关闭集群——依次在hadoop102、 hadoop103、 hadoop104节点上启动Kafka

> 启动Kafka之前，应先启动Zookeeper

```
#先启动Zookeeper
[ahua@hadoop102 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start
[ahua@hadoop103 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start
[ahua@hadoop104 apache-zookeeper-3.6.2-bin]$ bin/zkServer.sh start

#再启动Kafka
[ahua@hadoop102 kafka_2.13-2.7.0]$ bin/kafka-server-start.sh -daemon config/server.properties
[ahua@hadoop103 kafka_2.13-2.7.0]$ bin/kafka-server-start.sh -daemon config/server.properties
[ahua@hadoop104 kafka_2.13-2.7.0]$ bin/kafka-server-start.sh -daemon config/server.properties

#关闭Kafka
[ahua@hadoop102 kafka_2.13-2.7.0]$ bin/kafka-server-stop.sh stop
[ahua@hadoop103 kafka_2.13-2.7.0]$ bin/kafka-server-stop.sh stop
[ahua@hadoop104 kafka_2.13-2.7.0]$ bin/kafka-server-stop.sh stop
```

> 客户端启动查看bin/zkCli.sh
>
> 关闭的时候要先关kafka，再关zookeeper

##### Eagle安装配置

------



# HBase

##### 安装部署

- 将hbase-2.3.5-bin.tar.gz上传到/opt/software/目录下
- 解压hbase-2.3.5-bin.tar.gz到/opt/module/目录下

```
[ahua@hadoop102 module]$ tar -zxvf hbase-2.3.5-bin.tar.gz -C /opt/module/
```

- 修改/opt/module/hbase-2.3.5/conf/目录下HBase的配置文件

```bash
#修改hbase-env.sh内容
[ahua@hadoop102 conf]$ vim hbase-env.sh
#修改内容如下
export JAVA_HOME=/opt/module/jdk1.8.0_271
export HBASE_MANAGES_ZK=false
```

```xml
#修改hbase-site.xml内容
[ahua@hadoop102 conf]$ vim hbase-site.xml
#添加内容如下
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop102:8020/hbase</value>
  </property>
<!--在2.3.5版本中这个配置已经存在,但是默认是false,这里要改成true-->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop102,hadoop103,hadoop104</value>
  </property>
</configuration>
```

```bash
#修改hbase-env.sh内容
[ahua@hadoop102 conf]$ vim regionservers
#修改内容如下
hadoop102
hadoop103
hadoop104
```

- 分发hbase

```shell
xsync hbase-2.3.5
```

- 启动

```shell
[ahua@hadoop102 hbase-2.3.5]$ bin/start-hbase.sh
[ahua@hadoop102 hbase-2.3.5]$ bin/stop-hbase.sh
```

> bin/hbase-daemon.sh start master
>
> bin/hbase-daemon.sh start regionserver

------

# Shell脚本

##### 环境变量my_env.sh

> /etc/profile.d/my_env.sh

```shell
sudo vim /etc/profile.d/my_env.sh
```

```shell
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_271
export PATH=$PATH:$JAVA_HOME/bin

#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.2.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka_2.12-2.7.0
export PATH=$PATH:$KAFKA_HOME/bin

#HIVE_HOME
export HIVE_HOME=/opt/module/apache-hive-3.1.2-bin
export PATH=$PATH:$HIVE_HOME/bin

#SPARK_HOME
export SPARK_HOME=/opt/module/spark-3.0.0-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin

#HBASE_HOME
export HBASE_HOME=/opt/module/hbase-2.3.5
export PATH=$PATH:$HBASE_HOME/bin
```

> 让环境变量生效，貌似都可以
>
> source /etc/profile.d/my_env.sh
>
> source /etc/profile



##### 集群分发脚本xsync

> /home/ahua/bin/xsync

```shell
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
	then
		echo Not Enough Arguement!
        exit;
fi

#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
	echo ==================== $host ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
		#4. 判断文件是否存在
        if [ -e $file ]
			then
				#5. 获取父目录
				pdir=$(cd -P $(dirname $file); pwd)
				#6. 获取当前文件的名称
				fname=$(basename $file)
				ssh $host "mkdir -p $pdir"
				rsync -av $pdir/$fname $host:$pdir
			else
				echo $file does not exists!
        fi
    done
done
```

> 脚本创建后，要给执行权限
>
> chmod 777 xsync

##### 进程查看脚本jpsall

> /home/ahua/bin/jpsall

```bash
#!/bin/bash

for host in hadoop102 hadoop103 hadoop104

do

        echo =============== $host ===============

        ssh $host jps

done
```

> 脚本创建后，要给执行权限
>
> chmod 777 jpsall

##### Hadoop群起群关脚本myhadoop.sh

> /home/ahua/bin/myhadoop.sh

```bash
#!/bin/bash

if [ $# -lt 1 ]

then
        echo "No Args Input..."
        exit ;
fi

case $1 in
"start")

        echo " =================== 启动 hadoop 集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.2.2/sbin/start-dfs.sh"

        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.2.2/sbin/start-yarn.sh"

        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.2.2/bin/mapred --daemon start historyserver"
;;
"stop")

        echo " =================== 关闭 hadoop 集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102  "/opt/module/hadoop-3.2.2/bin/mapred --daemon stop historyserver"

        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.2.2/sbin/stop-yarn.sh"

        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.2.2/sbin/stop-dfs.sh"
;;
*)
        echo "Input Args Error..."
;;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 myhadoop.sh

##### Zookeeper群起群关脚本zk.sh

> /home/ahua/bin/zk.sh

```shell
#!/bin/bash

case $1 in 
"start"){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== zookeeper $i 启动 ==============="
		ssh $i "/opt/module/apache-zookeeper-3.6.2-bin/bin/zkServer.sh start"
	done
};;

"stop"){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== zookeeper $i 停止 ==============="
		ssh $i "/opt/module/apache-zookeeper-3.6.2-bin/bin/zkServer.sh stop"
	done
};;

"status"){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== zookeeper $i 状态 ==============="
		ssh $i "/opt/module/apache-zookeeper-3.6.2-bin/bin/zkServer.sh status"
	done
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 zk.sh

##### Flume f1.sh

> 保存在hadoop102/103 /home/ahua/bin/f1.sh
>
> 保存到hadoop104也可，不过在hadoop104上并不会用到此文件

```bash
#!/bin/bash

case $1 in 
"start"){

	for i in hadoop102 hadoop103
	do
		echo "=============== $i flume 开启 ==============="
		ssh $i "nohup /opt/module/apache-flume-1.9.0-bin/bin/flume-ng agent -n a1 -c /opt/module/apache-flume-1.9.0-bin/conf/ -f /opt/module/apache-flume-1.9.0-bin/jobs/taildir-kafka.conf -Dflume.root.logger=INFO,LOGFILE >/opt/module/apache-flume-1.9.0-bin/log1.txt 2>&1 &"
	done
};;

"stop"){

	for i in hadoop102 hadoop103
	do
		echo "=============== $i flume 关闭 ==============="
#		ssh $i "ps -ef | grep taildir-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
		ssh $i "ps -ef | awk '/taildir-kafka.conf/ && !/awk/{print \$2}' | xargs -n1 kill -9 "
	done
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 f1.sh
>
> 单发给hadoop103
>
> [ahua@hadoop102 bin]$ scp -r /home/ahua/bin/f1.sh hadoop103:/home/ahua/bin/
>
> ```shell
> #!/bin/bash
> 
> case $1 in 
> "start"){
> 
> 	for i in hadoop102 hadoop103
> 	do
> 		echo "=============== $i flume 开启 ==============="
> 		ssh $i "nohup /opt/module/apache-flume-1.9.0-bin/bin/flume-ng agent -n a1 -c /opt/module/apache-flume-1.9.0-bin/conf/ -f /opt/module/apache-flume-1.9.0-bin/jobs/taildir-kafka.conf -Dflume.root.logger=INFO,LOGFILE >/opt/module/apache-flume-1.9.0-bin/log1.txt 2>&1 &"
> 	done
> };;
> 
> "stop"){
> 
> 	for i in hadoop102 hadoop103
> 	do
> 		echo "=============== $i flume 关闭 ==============="
> #		ssh $i "ps -ef | grep taildir-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
> 		ssh $i "ps -ef | awk '/taildir-kafka.conf/ && !/awk/{print \$2}' | xargs -n1 kill -9 "
> 	done
> };;
> esac
> ```
>
> 保留了日志信息的脚本
>
> ssh $i "nohup /opt/module/apache-flume-1.9.0-bin/bin/flume-ng agent -n a1 -c /opt/module/apache-flume-1.9.0-bin/conf/ -f /opt/module/apache-flume-1.9.0-bin/jobs/taildir-kafka.conf >/dev/null 2>&1 &"

##### Flume f2.sh

> 保存在hadoop104 /home/ahua/bin/f2.sh
>
> 保存在hadoop102/103上也可，不过并不会在102/103上用到此文件

```bash
#!/bin/bash

case $1 in 
"start"){

	for i in hadoop104
	do
		echo "=============== $i flume 开启 ==============="
		ssh $i "nohup /opt/module/apache-flume-1.9.0-bin/bin/flume-ng agent -n a1 -c /opt/module/apache-flume-1.9.0-bin/conf/ -f /opt/module/apache-flume-1.9.0-bin/jobs/kafka-file-hdfs.conf -Dflume.root.logger=INFO,LOGFILE >/opt/module/apache-flume-1.9.0-bin/log2.txt 2>&1 &"
	done
};;

"stop"){

	for i in hadoop104
	do
		echo "=============== $i flume 关闭 ==============="
#		ssh $i "ps -ef | grep kafka-file-hdfs | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
		ssh $i "ps -ef | awk '/kafka-file-hdfs.conf/ && !/awk/{print \$2}' | xargs -n1 kill -9 "
	done
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 f2.sh

##### Kafka群起群关脚本kk.sh

> /home/ahua/bin/kk.sh

```bash
#!/bin/bash

case $1 in 
"start"){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== 启动 $i Kafka ==============="
		ssh $i "/opt/module/kafka_2.12-2.7.0/bin/kafka-server-start.sh -daemon /opt/module/kafka_2.12-2.7.0/config/server.properties"
	done
};;

"stop"){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== 停止 $i Kafka ==============="
		ssh $i "/opt/module/kafka_2.12-2.7.0/bin/kafka-server-stop.sh stop"
	done
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 kk.sh

##### 采集通道启动/停止脚本cluster.sh

```shell
#!/bin/bash

case $1 in
"start"){
        echo ================== 启动 集群 ==================

        #启动 Zookeeper 集群
        zk.sh start

        #启动 Hadoop 集群
        myhadoop.sh start

        #启动 Kafka 采集集群
        kk.sh start

        #启动 Flume 采集集群
        f1.sh start

        #启动 Flume 消费集群
        f2.sh start

        };;
"stop"){
        echo ================== 停止 集群 ==================

        #停止 Flume 消费集群
        f2.sh stop

        #停止 Flume 采集集群
        f1.sh stop

        #停止 Kafka 采集集群
        kk.sh stop
        
        #停止 Hadoop 集群
        myhadoop.sh stop
        
        #停止 Zookeeper 集群
        zk.sh stop

};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 kk.sh

##### 模拟日志生成脚本log.sh 

/home/ahua/bin/log.sh 

```bash
#！/bin/bash
for i in hadoop102 hadoop103; do 
	echo "=============== $i ==============="
	ssh $i "cd /opt/module/gmall_log/app_log;java -jar gmall2020-mock-log-2021-01-22.jar >/dev/null 2>&1 &"
done
```

> 脚本创建后，要给执行权限
>
> chmod 777 log.sh
>
> 注：
>
> ① /opt/module/applog/为 jar 包及配置文件所在路径
> ② /dev/null代表Linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。
> 标准输入0：从键盘获得输入 /proc/self/fd/0
> 标准输出1：输出到屏幕（即控制台） /proc/self/fd/1
> 错误输出2：输出到屏幕（即控制台） /proc/self/fd/2
>
> `>/dev/null 2>&1`是`1>/dev/null 2>/dev/null`的简写
>
> 最后一个`&`表示命令在后台执行

## 业务数据采集同步

##### Sqoop导入命令和脚本3.0

###### sqoop导入命令mysql->hdfs

```bash
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/gmall \
--username root \
--password ahuamysql \
--table user_info \
--columns id,login_name \
--where "id >= 10 and id <= 30" \
--target-dir /test \
--delete-target-dir \
--num-mappers 2 \
--fields-terminated-by '\t' \
--split-by id
```

```bash
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/gmall \
--username root \
--password ahuamysql \
--query "select id,login_name from user_info where id >= 10 and id <= 30 and \$CONDITIONS" \
--target-dir /test1 \
--delete-target-dir \
--num-mappers 2 \
--fields-terminated-by '\t' \
--split-by id
```

###### sqoop导入脚本mysql_to_hdfs.sh

> /home/ahua/bin/mysql_to_hdfs.sh

```bash
#!/bin/bash

APP=gmall
sqoop=/opt/module/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop

if [ -n "$2" ] ;then
    do_date=$2
else
    do_date=`date -d '-1 day' +%F`
fi

import_data(){
$sqoop import \
-Dmapreduce.job.queuename=hive \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password ahuamysql \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 and \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

#建索引
hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer -Dmapreduce.job.queuename=hive /origin_data/$APP/db/$1/$do_date
}

import_order_info(){
  import_data order_info "select
                            id, 
                            final_total_amount, 
                            order_status, 
                            user_id, 
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            province_id,
                            benefit_reduce_amount,
                            original_total_amount,
                            feight_fee      
                        from order_info
                        where (date_format(create_time,'%Y-%m-%d')='$do_date' 
                        or date_format(operate_time,'%Y-%m-%d')='$do_date')"
}

import_coupon_use(){
  import_data coupon_use "select
                          id,
                          coupon_id,
                          user_id,
                          order_id,
                          coupon_status,
                          get_time,
                          using_time,
                          used_time
                        from coupon_use
                        where (date_format(get_time,'%Y-%m-%d')='$do_date'
                        or date_format(using_time,'%Y-%m-%d')='$do_date'
                        or date_format(used_time,'%Y-%m-%d')='$do_date')"
}

import_order_status_log(){
  import_data order_status_log "select
                                  id,
                                  order_id,
                                  order_status,
                                  operate_time
                                from order_status_log
                                where date_format(operate_time,'%Y-%m-%d')='$do_date'"
}

import_activity_order(){
  import_data activity_order "select
                                id,
                                activity_id,
                                order_id,
                                create_time
                              from activity_order
                              where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_user_info(){
  import_data "user_info" "select 
                            id,
                            name,
                            birthday,
                            gender,
                            email,
                            user_level, 
                            create_time,
                            operate_time
                          from user_info 
                          where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                          or DATE_FORMAT(operate_time,'%Y-%m-%d')='$do_date')"
}

import_order_detail(){
  import_data order_detail "select 
                              od.id,
                              order_id, 
                              user_id, 
                              sku_id,
                              sku_name,
                              order_price,
                              sku_num, 
                              od.create_time,
                              source_type,
                              source_id  
                            from order_detail od
                            join order_info oi
                            on od.order_id=oi.id
                            where DATE_FORMAT(od.create_time,'%Y-%m-%d')='$do_date'"
}

import_payment_info(){
  import_data "payment_info"  "select 
                                id,  
                                out_trade_no, 
                                order_id, 
                                user_id, 
                                alipay_trade_no, 
                                total_amount,  
                                subject, 
                                payment_type, 
                                payment_time 
                              from payment_info 
                              where DATE_FORMAT(payment_time,'%Y-%m-%d')='$do_date'"
}

import_comment_info(){
  import_data comment_info "select
                              id,
                              user_id,
                              sku_id,
                              spu_id,
                              order_id,
                              appraise,
                              comment_txt,
                              create_time
                            from comment_info
                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_order_refund_info(){
  import_data order_refund_info "select
                                id,
                                user_id,
                                order_id,
                                sku_id,
                                refund_type,
                                refund_num,
                                refund_amount,
                                refund_reason_type,
                                create_time
                              from order_refund_info
                              where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          create_time
                        from sku_info where 1=1"
}

import_base_category1(){
  import_data "base_category1" "select 
                                  id,
                                  name 
                                from base_category1 where 1=1"
}

import_base_category2(){
  import_data "base_category2" "select
                                  id,
                                  name,
                                  category1_id 
                                from base_category2 where 1=1"
}

import_base_category3(){
  import_data "base_category3" "select
                                  id,
                                  name,
                                  category2_id
                                from base_category3 where 1=1"
}

import_base_province(){
  import_data base_province "select
                              id,
                              name,
                              region_id,
                              area_code,
                              iso_code
                            from base_province
                            where 1=1"
}

import_base_region(){
  import_data base_region "select
                              id,
                              region_name
                            from base_region
                            where 1=1"
}

import_base_trademark(){
  import_data base_trademark "select
                                tm_id,
                                tm_name
                              from base_trademark
                              where 1=1"
}

import_spu_info(){
  import_data spu_info "select
                            id,
                            spu_name,
                            category3_id,
                            tm_id
                          from spu_info
                          where 1=1"
}

import_favor_info(){
  import_data favor_info "select
                          id,
                          user_id,
                          sku_id,
                          spu_id,
                          is_cancel,
                          create_time,
                          cancel_time
                        from favor_info
                        where 1=1"
}

import_cart_info(){
  import_data cart_info "select
                        id,
                        user_id,
                        sku_id,
                        cart_price,
                        sku_num,
                        sku_name,
                        create_time,
                        operate_time,
                        is_ordered,
                        order_time,
                        source_type,
                        source_id
                      from cart_info
                      where 1=1"
}

import_coupon_info(){
  import_data coupon_info "select
                          id,
                          coupon_name,
                          coupon_type,
                          condition_amount,
                          condition_num,
                          activity_id,
                          benefit_amount,
                          benefit_discount,
                          create_time,
                          range_type,
                          spu_id,
                          tm_id,
                          category3_id,
                          limit_num,
                          operate_time,
                          expire_time
                        from coupon_info
                        where 1=1"
}

import_activity_info(){
  import_data activity_info "select
                              id,
                              activity_name,
                              activity_type,
                              start_time,
                              end_time,
                              create_time
                            from activity_info
                            where 1=1"
}

import_activity_rule(){
    import_data activity_rule "select
                                    id,
                                    activity_id,
                                    condition_amount,
                                    condition_num,
                                    benefit_amount,
                                    benefit_discount,
                                    benefit_level
                                from activity_rule
                                where 1=1"
}

import_base_dic(){
    import_data base_dic "select
                            dic_code,
                            dic_name,
                            parent_code,
                            create_time,
                            operate_time
                          from base_dic
                          where 1=1"
}

case $1 in
  "order_info")
     import_order_info
;;
  "base_category1")
     import_base_category1
;;
  "base_category2")
     import_base_category2
;;
  "base_category3")
     import_base_category3
;;
  "order_detail")
     import_order_detail
;;
  "sku_info")
     import_sku_info
;;
  "user_info")
     import_user_info
;;
  "payment_info")
     import_payment_info
;;
  "base_province")
     import_base_province
;;
  "base_region")
     import_base_region
;;
  "base_trademark")
     import_base_trademark
;;
  "activity_info")
      import_activity_info
;;
  "activity_order")
      import_activity_order
;;
  "cart_info")
      import_cart_info
;;
  "comment_info")
      import_comment_info
;;
  "coupon_info")
      import_coupon_info
;;
  "coupon_use")
      import_coupon_use
;;
  "favor_info")
      import_favor_info
;;
  "order_refund_info")
      import_order_refund_info
;;
  "order_status_log")
      import_order_status_log
;;
  "spu_info")
      import_spu_info
;;
  "activity_rule")
      import_activity_rule
;;
  "base_dic")
      import_base_dic
;;

"first")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_province
   import_base_region
   import_base_trademark
   import_activity_info
   import_activity_order
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
;;
"all")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_trademark
   import_activity_info
   import_activity_order
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
;;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 mysql_to_hdfs.sh
>
> date -d '-1 day' +%F 昨天的日期
>
> do_date=$(date -d '-1 day' +%F)
>
> Hive中的 Null在底层是以“ “\N”来存储，而 MySQL中的 Null在底层就是 Null，为了
>保证数据两端的一致性。在导出数据时采用 --input-null-string和 --input-null-non-string两个参数。导入数据时采用 --null-string和 --null-non-string。
> 
> 如果要指定队列，需要在两处添加
> 
>-Dmapreduce.job.queuename=hive

###### sqoop导出脚本hdfs_to_mysql.sh

> /home/ahua/bin/hdfs_to_mysql.sh

```bash
#!/bin/bash

hive_db_name=gmall
mysql_db_name=gmall_report

export_data() {
/opt/module/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop export \
-Dmapreduce.job.queuename=hive \
--connect "jdbc:mysql://hadoop102:3306/${mysql_db_name}?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password ahuamysql \
--table $1 \
--num-mappers 1 \
--export-dir /warehouse/$hive_db_name/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $2 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}

case $1 in
  "ads_uv_count")
     export_data "ads_uv_count" "dt"
;;
  "ads_user_action_convert_day") 
     export_data "ads_user_action_convert_day" "dt"
;;
  "ads_user_topic")
     export_data "ads_user_topic" "dt"
;;
  "ads_area_topic")
     export_data "ads_area_topic" "dt,iso_code"
;;
   "all")
     export_data "ads_user_topic" "dt"
     export_data "ads_area_topic" "dt,iso_code"
     #其余表省略未写
;;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 hdfs_to_mysql.sh

##### Sqoop导入命令和脚本4.0

###### 业务数据首日同步导入脚本mysql_to_hdfs_init.sh

> /home/ahua/bin/mysql_to_hdfs_init.sh
>
> 对比3.0，就是将3.0的导入脚本拆分成了首日同步和每日同步两个
>
> 因为有历史数据，所以区分首日每日

```shell
#! /bin/bash

APP=gmall
sqoop=/opt/module/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi

import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password ahuamysql \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 where \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

# 建索引
hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/db/$1/$do_date
}

import_order_info(){
  import_data order_info "select
                            id, 
                            total_amount, 
                            order_status, 
                            user_id, 
                            payment_way,
                            delivery_address,
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            expire_time,
                            tracking_no,
                            province_id,
                            activity_reduce_amount,
                            coupon_reduce_amount,                            
                            original_total_amount,
                            feight_fee,
                            feight_fee_reduce      
                        from order_info"
}

import_coupon_use(){
  import_data coupon_use "select
                          id,
                          coupon_id,
                          user_id,
                          order_id,
                          coupon_status,
                          get_time,
                          using_time,
                          used_time,
                          expire_time
                        from coupon_use"
}

import_order_status_log(){
  import_data order_status_log "select
                                  id,
                                  order_id,
                                  order_status,
                                  operate_time
                                from order_status_log"
}

import_user_info(){
  import_data "user_info" "select 
                            id,
                            login_name,
                            nick_name,
                            name,
                            phone_num,
                            email,
                            user_level, 
                            birthday,
                            gender,
                            create_time,
                            operate_time
                          from user_info"
}

import_order_detail(){
  import_data order_detail "select 
                              id,
                              order_id, 
                              sku_id,
                              sku_name,
                              order_price,
                              sku_num, 
                              create_time,
                              source_type,
                              source_id,
                              split_total_amount,
                              split_activity_amount,
                              split_coupon_amount
                            from order_detail"
}

import_payment_info(){
  import_data "payment_info"  "select 
                                id,  
                                out_trade_no, 
                                order_id, 
                                user_id, 
                                payment_type, 
                                trade_no, 
                                total_amount,  
                                subject, 
                                payment_status,
                                create_time,
                                callback_time 
                              from payment_info"
}

import_comment_info(){
  import_data comment_info "select
                              id,
                              user_id,
                              sku_id,
                              spu_id,
                              order_id,
                              appraise,
                              create_time
                            from comment_info"
}

import_order_refund_info(){
  import_data order_refund_info "select
                                id,
                                user_id,
                                order_id,
                                sku_id,
                                refund_type,
                                refund_num,
                                refund_amount,
                                refund_reason_type,
                                refund_status,
                                create_time
                              from order_refund_info"
}

import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          is_sale,
                          create_time
                        from sku_info"
}

import_base_category1(){
  import_data "base_category1" "select 
                                  id,
                                  name 
                                from base_category1"
}

import_base_category2(){
  import_data "base_category2" "select
                                  id,
                                  name,
                                  category1_id 
                                from base_category2"
}

import_base_category3(){
  import_data "base_category3" "select
                                  id,
                                  name,
                                  category2_id
                                from base_category3"
}

import_base_province(){
  import_data base_province "select
                              id,
                              name,
                              region_id,
                              area_code,
                              iso_code,
                              iso_3166_2
                            from base_province"
}

import_base_region(){
  import_data base_region "select
                              id,
                              region_name
                            from base_region"
}

import_base_trademark(){
  import_data base_trademark "select
                                id,
                                tm_name
                              from base_trademark"
}

import_spu_info(){
  import_data spu_info "select
                            id,
                            spu_name,
                            category3_id,
                            tm_id
                          from spu_info"
}

import_favor_info(){
  import_data favor_info "select
                          id,
                          user_id,
                          sku_id,
                          spu_id,
                          is_cancel,
                          create_time,
                          cancel_time
                        from favor_info"
}

import_cart_info(){
  import_data cart_info "select
                        id,
                        user_id,
                        sku_id,
                        cart_price,
                        sku_num,
                        sku_name,
                        create_time,
                        operate_time,
                        is_ordered,
                        order_time,
                        source_type,
                        source_id
                      from cart_info"
}

import_coupon_info(){
  import_data coupon_info "select
                          id,
                          coupon_name,
                          coupon_type,
                          condition_amount,
                          condition_num,
                          activity_id,
                          benefit_amount,
                          benefit_discount,
                          create_time,
                          range_type,
                          limit_num,
                          taken_count,
                          start_time,
                          end_time,
                          operate_time,
                          expire_time
                        from coupon_info"
}

import_activity_info(){
  import_data activity_info "select
                              id,
                              activity_name,
                              activity_type,
                              start_time,
                              end_time,
                              create_time
                            from activity_info"
}

import_activity_rule(){
    import_data activity_rule "select
                                    id,
                                    activity_id,
                                    activity_type,
                                    condition_amount,
                                    condition_num,
                                    benefit_amount,
                                    benefit_discount,
                                    benefit_level
                                from activity_rule"
}

import_base_dic(){
    import_data base_dic "select
                            dic_code,
                            dic_name,
                            parent_code,
                            create_time,
                            operate_time
                          from base_dic"
}


import_order_detail_activity(){
    import_data order_detail_activity "select
                                                                id,
                                                                order_id,
                                                                order_detail_id,
                                                                activity_id,
                                                                activity_rule_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_activity"
}


import_order_detail_coupon(){
    import_data order_detail_coupon "select
                                                                id,
								                                order_id,
                                                                order_detail_id,
                                                                coupon_id,
                                                                coupon_use_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_coupon"
}


import_refund_payment(){
    import_data refund_payment "select
                                                        id,
                                                        out_trade_no,
                                                        order_id,
                                                        sku_id,
                                                        payment_type,
                                                        trade_no,
                                                        total_amount,
                                                        subject,
                                                        refund_status,
                                                        create_time,
                                                        callback_time
                                                    from refund_payment"                                                    

}

import_sku_attr_value(){
    import_data sku_attr_value "select
                                                    id,
                                                    attr_id,
                                                    value_id,
                                                    sku_id,
                                                    attr_name,
                                                    value_name
                                                from sku_attr_value"
}


import_sku_sale_attr_value(){
    import_data sku_sale_attr_value "select
                                                            id,
                                                            sku_id,
                                                            spu_id,
                                                            sale_attr_value_id,
                                                            sale_attr_id,
                                                            sale_attr_name,
                                                            sale_attr_value_name
                                                        from sku_sale_attr_value"
}

case $1 in
  "order_info")
     import_order_info
;;
  "base_category1")
     import_base_category1
;;
  "base_category2")
     import_base_category2
;;
  "base_category3")
     import_base_category3
;;
  "order_detail")
     import_order_detail
;;
  "sku_info")
     import_sku_info
;;
  "user_info")
     import_user_info
;;
  "payment_info")
     import_payment_info
;;
  "base_province")
     import_base_province
;;
  "base_region")
     import_base_region
;;
  "base_trademark")
     import_base_trademark
;;
  "activity_info")
      import_activity_info
;;
  "cart_info")
      import_cart_info
;;
  "comment_info")
      import_comment_info
;;
  "coupon_info")
      import_coupon_info
;;
  "coupon_use")
      import_coupon_use
;;
  "favor_info")
      import_favor_info
;;
  "order_refund_info")
      import_order_refund_info
;;
  "order_status_log")
      import_order_status_log
;;
  "spu_info")
      import_spu_info
;;
  "activity_rule")
      import_activity_rule
;;
  "base_dic")
      import_base_dic
;;
  "order_detail_activity")
      import_order_detail_activity
;;
  "order_detail_coupon")
      import_order_detail_coupon
;;
  "refund_payment")
      import_refund_payment
;;
  "sku_attr_value")
      import_sku_attr_value
;;
  "sku_sale_attr_value")
      import_sku_sale_attr_value
;;
  "all")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_region
   import_base_province
   import_base_trademark
   import_activity_info
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
   import_order_detail_activity
   import_order_detail_coupon
   import_refund_payment
   import_sku_attr_value
   import_sku_sale_attr_value
;;
esac
```

> 说明1：
>
> [ -n 变量值 ] 判断变量的值，是否为空
>
> -- 变量的值，非空，返回true
>
> -- 变量的值，为空，返回false
>
> 注意：[ -n 变量值 ]不会解析数据，使用[ -n 变量值 ]时，需要对变量加上双引号(" ")
>
> 说明2：
>
> 查看date命令的使用，[ahua@hadoop102 ~]$ date --help
>
> Hive 中的 Null 在底层是以“\N”来存储，而 MySQL 中的 Null 在底层就是 Null。为了
> 保证数据两端的一致性，在导出数据时采用--input-null-string 和--input-null-non-string 两个参
> 数，导入数据时采用--null-string 和--null-non-string。

###### 业务数据每日同步导入脚本mysql_to_hdfs_daily.sh

> /home/ahua/bin/mysql_to_hdfs_daily.sh
>
> 每日比首日少导入两个表（地区和省份表）

```shell
#! /bin/bash

APP=gmall
sqoop=/opt/module/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop

if [ -n "$2" ] ;then
    do_date=$2
else
    do_date=`date -d '-1 day' +%F`
fi

import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password ahuamysql \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 and  \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

# 建索引
hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/db/$1/$do_date
}

import_order_info(){
  import_data order_info "select
                            id, 
                            total_amount, 
                            order_status, 
                            user_id, 
                            payment_way,
                            delivery_address,
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            expire_time,
                            tracking_no,
                            province_id,
                            activity_reduce_amount,
                            coupon_reduce_amount,                            
                            original_total_amount,
                            feight_fee,
                            feight_fee_reduce      
                        from order_info
                        where (date_format(create_time,'%Y-%m-%d')='$do_date' 
                        or date_format(operate_time,'%Y-%m-%d')='$do_date')"
}

import_coupon_use(){
  import_data coupon_use "select
                          id,
                          coupon_id,
                          user_id,
                          order_id,
                          coupon_status,
                          get_time,
                          using_time,
                          used_time,
                          expire_time
                        from coupon_use
                        where (date_format(get_time,'%Y-%m-%d')='$do_date'
                        or date_format(using_time,'%Y-%m-%d')='$do_date'
                        or date_format(used_time,'%Y-%m-%d')='$do_date'
                        or date_format(expire_time,'%Y-%m-%d')='$do_date')"
}

import_order_status_log(){
  import_data order_status_log "select
                                  id,
                                  order_id,
                                  order_status,
                                  operate_time
                                from order_status_log
                                where date_format(operate_time,'%Y-%m-%d')='$do_date'"
}

import_user_info(){
  import_data "user_info" "select 
                            id,
                            login_name,
                            nick_name,
                            name,
                            phone_num,
                            email,
                            user_level, 
                            birthday,
                            gender,
                            create_time,
                            operate_time
                          from user_info 
                          where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                          or DATE_FORMAT(operate_time,'%Y-%m-%d')='$do_date')"
}

import_order_detail(){
  import_data order_detail "select 
                              id,
                              order_id, 
                              sku_id,
                              sku_name,
                              order_price,
                              sku_num, 
                              create_time,
                              source_type,
                              source_id,
                              split_total_amount,
                              split_activity_amount,
                              split_coupon_amount
                            from order_detail 
                            where DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date'"
}

import_payment_info(){
  import_data "payment_info"  "select 
                                id,  
                                out_trade_no, 
                                order_id, 
                                user_id, 
                                payment_type, 
                                trade_no, 
                                total_amount,  
                                subject, 
                                payment_status,
                                create_time,
                                callback_time 
                              from payment_info 
                              where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                              or DATE_FORMAT(callback_time,'%Y-%m-%d')='$do_date')"
}

import_comment_info(){
  import_data comment_info "select
                              id,
                              user_id,
                              sku_id,
                              spu_id,
                              order_id,
                              appraise,
                              create_time
                            from comment_info
                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_order_refund_info(){
  import_data order_refund_info "select
                                id,
                                user_id,
                                order_id,
                                sku_id,
                                refund_type,
                                refund_num,
                                refund_amount,
                                refund_reason_type,
                                refund_status,
                                create_time
                              from order_refund_info
                              where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          is_sale,
                          create_time
                        from sku_info where 1=1"
}

import_base_category1(){
  import_data "base_category1" "select 
                                  id,
                                  name 
                                from base_category1 where 1=1"
}

import_base_category2(){
  import_data "base_category2" "select
                                  id,
                                  name,
                                  category1_id 
                                from base_category2 where 1=1"
}

import_base_category3(){
  import_data "base_category3" "select
                                  id,
                                  name,
                                  category2_id
                                from base_category3 where 1=1"
}

import_base_province(){
  import_data base_province "select
                              id,
                              name,
                              region_id,
                              area_code,
                              iso_code,
                              iso_3166_2
                            from base_province
                            where 1=1"
}

import_base_region(){
  import_data base_region "select
                              id,
                              region_name
                            from base_region
                            where 1=1"
}

import_base_trademark(){
  import_data base_trademark "select
                                id,
                                tm_name
                              from base_trademark
                              where 1=1"
}

import_spu_info(){
  import_data spu_info "select
                            id,
                            spu_name,
                            category3_id,
                            tm_id
                          from spu_info
                          where 1=1"
}

import_favor_info(){
  import_data favor_info "select
                          id,
                          user_id,
                          sku_id,
                          spu_id,
                          is_cancel,
                          create_time,
                          cancel_time
                        from favor_info
                        where 1=1"
}

import_cart_info(){
  import_data cart_info "select
                        id,
                        user_id,
                        sku_id,
                        cart_price,
                        sku_num,
                        sku_name,
                        create_time,
                        operate_time,
                        is_ordered,
                        order_time,
                        source_type,
                        source_id
                      from cart_info
                      where 1=1"
}

import_coupon_info(){
  import_data coupon_info "select
                          id,
                          coupon_name,
                          coupon_type,
                          condition_amount,
                          condition_num,
                          activity_id,
                          benefit_amount,
                          benefit_discount,
                          create_time,
                          range_type,
                          limit_num,
                          taken_count,
                          start_time,
                          end_time,
                          operate_time,
                          expire_time
                        from coupon_info
                        where 1=1"
}

import_activity_info(){
  import_data activity_info "select
                              id,
                              activity_name,
                              activity_type,
                              start_time,
                              end_time,
                              create_time
                            from activity_info
                            where 1=1"
}

import_activity_rule(){
    import_data activity_rule "select
                                    id,
                                    activity_id,
                                    activity_type,
                                    condition_amount,
                                    condition_num,
                                    benefit_amount,
                                    benefit_discount,
                                    benefit_level
                                from activity_rule
                                where 1=1"
}

import_base_dic(){
    import_data base_dic "select
                            dic_code,
                            dic_name,
                            parent_code,
                            create_time,
                            operate_time
                          from base_dic
                          where 1=1"
}


import_order_detail_activity(){
    import_data order_detail_activity "select
                                                                id,
                                                                order_id,
                                                                order_detail_id,
                                                                activity_id,
                                                                activity_rule_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_activity
                                                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}


import_order_detail_coupon(){
    import_data order_detail_coupon "select
                                                                id,
								                                                order_id,
                                                                order_detail_id,
                                                                coupon_id,
                                                                coupon_use_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_coupon
                                                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}


import_refund_payment(){
    import_data refund_payment "select
                                                        id,
                                                        out_trade_no,
                                                        order_id,
                                                        sku_id,
                                                        payment_type,
                                                        trade_no,
                                                        total_amount,
                                                        subject,
                                                        refund_status,
                                                        create_time,
                                                        callback_time
                                                    from refund_payment
                                                    where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                                                    or DATE_FORMAT(callback_time,'%Y-%m-%d')='$do_date')"                                                    

}

import_sku_attr_value(){
    import_data sku_attr_value "select
                                                    id,
                                                    attr_id,
                                                    value_id,
                                                    sku_id,
                                                    attr_name,
                                                    value_name
                                                from sku_attr_value
                                                where 1=1"
}


import_sku_sale_attr_value(){
    import_data sku_sale_attr_value "select
                                                            id,
                                                            sku_id,
                                                            spu_id,
                                                            sale_attr_value_id,
                                                            sale_attr_id,
                                                            sale_attr_name,
                                                            sale_attr_value_name
                                                        from sku_sale_attr_value
                                                        where 1=1"
}

case $1 in
  "order_info")
     import_order_info
;;
  "base_category1")
     import_base_category1
;;
  "base_category2")
     import_base_category2
;;
  "base_category3")
     import_base_category3
;;
  "order_detail")
     import_order_detail
;;
  "sku_info")
     import_sku_info
;;
  "user_info")
     import_user_info
;;
  "payment_info")
     import_payment_info
;;
  "base_province")
     import_base_province
;;
  "activity_info")
      import_activity_info
;;
  "cart_info")
      import_cart_info
;;
  "comment_info")
      import_comment_info
;;
  "coupon_info")
      import_coupon_info
;;
  "coupon_use")
      import_coupon_use
;;
  "favor_info")
      import_favor_info
;;
  "order_refund_info")
      import_order_refund_info
;;
  "order_status_log")
      import_order_status_log
;;
  "spu_info")
      import_spu_info
;;
  "activity_rule")
      import_activity_rule
;;
  "base_dic")
      import_base_dic
;;
  "order_detail_activity")
      import_order_detail_activity
;;
  "order_detail_coupon")
      import_order_detail_coupon
;;
  "refund_payment")
      import_refund_payment
;;
  "sku_attr_value")
      import_sku_attr_value
;;
  "sku_sale_attr_value")
      import_sku_sale_attr_value
;;
"all")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_trademark
   import_activity_info
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
   import_order_detail_activity
   import_order_detail_coupon
   import_refund_payment
   import_sku_attr_value
   import_sku_sale_attr_value
;;
esac
```

## 数仓搭建3.0

#### 数仓搭建-ODS层

##### ODS层用户行为数据导入脚本hdfs_to_ods_log.sh

> /home/ahua/bin/hdfs_to_ods_log.sh

先创建支持lzo压缩的分区表，再通过下面的脚本实现ODS用户行为数据的加载和建索引

###### 建表语句——1张表

> ```sql
> --ODS层日志数据建表语句--
> --创建支持lzo压缩的分区表
> drop table if exists ods_log;
> CREATE EXTERNAL TABLE ods_log (`line` string)
> PARTITIONED BY (`dt` string) -- 按照时间创建分区
> STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；
>   INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
>   OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
> LOCATION '/warehouse/gmall/ods/ods_log'  -- 指定数据在hdfs上的存储位置
> ;
> --
> -- 加载数据
> -- load data inpath '/origin_data/gmall/log/topic_log/2021-04-03' into table ods_log partition(dt='2021-04-03');
> -- 为lzo压缩文件创建索引
> -- hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer -Dmapreduce.job.queuename=hive /warehouse/gmall/ods/ods_log/dt=2021-04-03
> --
> -- select * from ods_log where dt='2021-04-03' limit 2;
> ```

###### 数据导入脚本

```bash
#!/bin/bash

# 定义变量方便修改
APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive
hadoop=/opt/module/hadoop-3.2.2/bin/hadoop

# 如果输入日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
   do_date=$1
else
   do_date=`date -d "-1 day" +%F`
fi

echo ================== 日志日期为 $do_date ==================
sql="
load data inpath '/origin_data/$APP/log/topic_log/$do_date' into table ${APP}.ods_log partition(dt='$do_date');
"

$hive -e "$sql"

$hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer -Dmapreduce.job.queuename=hive /warehouse/$APP/ods/ods_log/dt=$do_date
```

> 脚本创建后，要给执行权限
>
> chmod 777 hdfs_to_ods_log.sh
>
> 导数据 建索引
>
> 测试加载数据：hdfs_to_ods_log.sh 2021-04-03

##### ODS层业务数据导入脚本hdfs_to_ods_db.sh

> /home/ahua/bin/hdfs_to_ods_db.sh

###### 建表语句——23张表

```sql
--3.3.1 订单表（增量及更新）
--hive (gmall)>
drop table if exists ods_order_info;
create external table ods_order_info (
    `id` string COMMENT '订单号',
    `final_total_amount` decimal(16,2) COMMENT '订单金额',
    `order_status` string COMMENT '订单状态',
    `user_id` string COMMENT '用户id',
    `out_trade_no` string COMMENT '支付流水号',
    `create_time` string COMMENT '创建时间',
    `operate_time` string COMMENT '操作时间',
    `province_id` string COMMENT '省份ID',
    `benefit_reduce_amount` decimal(16,2) COMMENT '优惠金额',
    `original_total_amount` decimal(16,2)  COMMENT '原价金额',
    `feight_fee` decimal(16,2)  COMMENT '运费'
) COMMENT '订单表'
PARTITIONED BY (`dt` string) -- 按照时间创建分区
row format delimited fields terminated by '\t' -- 指定分割符为\t
STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；输出数据采用TextOutputFormat
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_order_info/' -- 指定数据在hdfs上的存储位置
;
--3.3.2 订单详情表（增量）
--hive (gmall)>
drop table if exists ods_order_detail;
create external table ods_order_detail(
    `id` string COMMENT '编号',
    `order_id` string  COMMENT '订单号',
    `user_id` string COMMENT '用户id',
    `sku_id` string COMMENT '商品id',
    `sku_name` string COMMENT '商品名称',
    `order_price` decimal(16,2) COMMENT '商品价格',
    `sku_num` bigint COMMENT '商品数量',
    `create_time` string COMMENT '创建时间',
    `source_type` string COMMENT '来源类型',
    `source_id` string COMMENT '来源编号'
) COMMENT '订单详情表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_order_detail/';
--3.3.3 SKU商品表（全量）
--hive (gmall)>
drop table if exists ods_sku_info;
create external table ods_sku_info(
    `id` string COMMENT 'skuId',
    `spu_id` string   COMMENT 'spuid',
    `price` decimal(16,2) COMMENT '价格',
    `sku_name` string COMMENT '商品名称',
    `sku_desc` string COMMENT '商品描述',
    `weight` string COMMENT '重量',
    `tm_id` string COMMENT '品牌id',
    `category3_id` string COMMENT '品类id',
    `create_time` string COMMENT '创建时间'
) COMMENT 'SKU商品表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_sku_info/';
--3.3.4 用户表（增量及更新）
--hive (gmall)>
drop table if exists ods_user_info;
create external table ods_user_info(
    `id` string COMMENT '用户id',
    `name`  string COMMENT '姓名',
    `birthday` string COMMENT '生日',
    `gender` string COMMENT '性别',
    `email` string COMMENT '邮箱',
    `user_level` string COMMENT '用户等级',
    `create_time` string COMMENT '创建时间',
    `operate_time` string COMMENT '操作时间'
) COMMENT '用户表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_user_info/';
--3.3.5 商品一级分类表（全量）
--hive (gmall)>
drop table if exists ods_base_category1;
create external table ods_base_category1(
    `id` string COMMENT 'id',
    `name`  string COMMENT '名称'
) COMMENT '商品一级分类表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_category1/';
--3.3.6 商品二级分类表（全量）
--hive (gmall)>
drop table if exists ods_base_category2;
create external table ods_base_category2(
    `id` string COMMENT ' id',
    `name` string COMMENT '名称',
    category1_id string COMMENT '一级品类id'
) COMMENT '商品二级分类表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_category2/';
--3.3.7 商品三级分类表（全量）
--hive (gmall)>
drop table if exists ods_base_category3;
create external table ods_base_category3(
    `id` string COMMENT ' id',
    `name`  string COMMENT '名称',
    category2_id string COMMENT '二级品类id'
) COMMENT '商品三级分类表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_category3/';
--3.3.8 支付流水表（增量）
--hive (gmall)>
drop table if exists ods_payment_info;
create external table ods_payment_info(
    `id`   bigint COMMENT '编号',
    `out_trade_no`    string COMMENT '对外业务编号',
    `order_id`        string COMMENT '订单编号',
    `user_id`         string COMMENT '用户编号',
    `alipay_trade_no` string COMMENT '支付宝交易流水编号',
    `total_amount`    decimal(16,2) COMMENT '支付金额',
    `subject`         string COMMENT '交易内容',
    `payment_type`    string COMMENT '支付类型',
    `payment_time`    string COMMENT '支付时间'
)  COMMENT '支付流水表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_payment_info/';
--3.3.9 省份表（特殊）
--hive (gmall)>
drop table if exists ods_base_province;
create external table ods_base_province (
    `id`   bigint COMMENT '编号',
    `name`        string COMMENT '省份名称',
    `region_id`    string COMMENT '地区ID',
    `area_code`    string COMMENT '地区编码',
    `iso_code` string COMMENT 'iso编码,superset可视化使用'
)  COMMENT '省份表'
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_province/';
--3.3.10 地区表（特殊）
--hive (gmall)>
drop table if exists ods_base_region;
create external table ods_base_region (
    `id` string COMMENT '编号',
    `region_name` string COMMENT '地区名称'
)  COMMENT '地区表'
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_region/';
--3.3.11 品牌表（全量）
--hive (gmall)>
drop table if exists ods_base_trademark;
create external table ods_base_trademark (
    `tm_id`   string COMMENT '编号',
    `tm_name` string COMMENT '品牌名称'
)  COMMENT '品牌表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_trademark/';
--3.3.12 订单状态表（增量）
--hive (gmall)>
drop table if exists ods_order_status_log;
create external table ods_order_status_log (
    `id`   string COMMENT '编号',
    `order_id` string COMMENT '订单ID',
    `order_status` string COMMENT '订单状态',
    `operate_time` string COMMENT '修改时间'
)  COMMENT '订单状态表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_order_status_log/';
--3.3.13 SPU商品表（全量）
--hive (gmall)>
drop table if exists ods_spu_info;
create external table ods_spu_info(
    `id` string COMMENT 'spuid',
    `spu_name` string COMMENT 'spu名称',
    `category3_id` string COMMENT '品类id',
    `tm_id` string COMMENT '品牌id'
) COMMENT 'SPU商品表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_spu_info/';
--3.3.14 商品评论表（增量）
--hive (gmall)>
drop table if exists ods_comment_info;
create external table ods_comment_info(
    `id` string COMMENT '编号',
    `user_id` string COMMENT '用户ID',
    `sku_id` string COMMENT '商品sku',
    `spu_id` string COMMENT '商品spu',
    `order_id` string COMMENT '订单ID',
    `appraise` string COMMENT '评价',
    `create_time` string COMMENT '评价时间'
) COMMENT '商品评论表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_comment_info/';
--3.3.15 退单表（增量）
--hive (gmall)>
drop table if exists ods_order_refund_info;
create external table ods_order_refund_info(
    `id` string COMMENT '编号',
    `user_id` string COMMENT '用户ID',
    `order_id` string COMMENT '订单ID',
    `sku_id` string COMMENT '商品ID',
    `refund_type` string COMMENT '退款类型',
    `refund_num` bigint COMMENT '退款件数',
    `refund_amount` decimal(16,2) COMMENT '退款金额',
    `refund_reason_type` string COMMENT '退款原因类型',
    `create_time` string COMMENT '退款时间'
) COMMENT '退单表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_order_refund_info/';
--3.3.16 加购表（全量）
--hive (gmall)>
drop table if exists ods_cart_info;
create external table ods_cart_info(
    `id` string COMMENT '编号',
    `user_id` string  COMMENT '用户id',
    `sku_id` string  COMMENT 'skuid',
    `cart_price` decimal(16,2)  COMMENT '放入购物车时价格',
    `sku_num` bigint  COMMENT '数量',
    `sku_name` string  COMMENT 'sku名称 (冗余)',
    `create_time` string  COMMENT '创建时间',
    `operate_time` string COMMENT '修改时间',
    `is_ordered` string COMMENT '是否已经下单',
    `order_time` string  COMMENT '下单时间',
    `source_type` string COMMENT '来源类型',
    `source_id` string COMMENT '来源编号'
) COMMENT '加购表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_cart_info/';
--3.3.17 商品收藏表（全量）
--hive (gmall)>
drop table if exists ods_favor_info;
create external table ods_favor_info(
    `id` string COMMENT '编号',
    `user_id` string  COMMENT '用户id',
    `sku_id` string  COMMENT 'skuid',
    `spu_id` string  COMMENT 'spuid',
    `is_cancel` string  COMMENT '是否取消',
    `create_time` string  COMMENT '收藏时间',
    `cancel_time` string  COMMENT '取消时间'
) COMMENT '商品收藏表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_favor_info/';
--3.3.18 优惠券领用表（新增及变化）
--hive (gmall)>
drop table if exists ods_coupon_use;
create external table ods_coupon_use(
    `id` string COMMENT '编号',
    `coupon_id` string  COMMENT '优惠券ID',
    `user_id` string  COMMENT 'skuid',
    `order_id` string  COMMENT 'spuid',
    `coupon_status` string  COMMENT '优惠券状态',
    `get_time` string  COMMENT '领取时间',
    `using_time` string  COMMENT '使用时间(下单)',
    `used_time` string  COMMENT '使用时间(支付)'
) COMMENT '优惠券领用表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_coupon_use/';
--3.3.19 优惠券表（全量）
--hive (gmall)>
drop table if exists ods_coupon_info;
create external table ods_coupon_info(
  `id` string COMMENT '购物券编号',
  `coupon_name` string COMMENT '购物券名称',
  `coupon_type` string COMMENT '购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券',
  `condition_amount` decimal(16,2) COMMENT '满额数',
  `condition_num` bigint COMMENT '满件数',
  `activity_id` string COMMENT '活动编号',
  `benefit_amount` decimal(16,2) COMMENT '减金额',
  `benefit_discount` decimal(16,2) COMMENT '折扣',
  `create_time` string COMMENT '创建时间',
  `range_type` string COMMENT '范围类型 1、商品 2、品类 3、品牌',
  `spu_id` string COMMENT '商品id',
  `tm_id` string COMMENT '品牌id',
  `category3_id` string COMMENT '品类id',
  `limit_num` bigint COMMENT '最多领用次数',
  `operate_time`  string COMMENT '修改时间',
  `expire_time`  string COMMENT '过期时间'
) COMMENT '优惠券表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_coupon_info/';
--3.3.20 活动表（全量）
--hive (gmall)>
drop table if exists ods_activity_info;
create external table ods_activity_info(
    `id` string COMMENT '编号',
    `activity_name` string  COMMENT '活动名称',
    `activity_type` string  COMMENT '活动类型',
    `start_time` string  COMMENT '开始时间',
    `end_time` string  COMMENT '结束时间',
    `create_time` string  COMMENT '创建时间'
) COMMENT '活动表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_activity_info/';
--3.3.21 活动订单关联表（增量）
--hive (gmall)>
drop table if exists ods_activity_order;
create external table ods_activity_order(
    `id` string COMMENT '编号',
    `activity_id` string  COMMENT '优惠券ID',
    `order_id` string  COMMENT 'skuid',
    `create_time` string  COMMENT '领取时间'
) COMMENT '活动订单关联表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_activity_order/';
--3.3.22 优惠规则表（全量）
--hive (gmall)>
drop table if exists ods_activity_rule;
create external table ods_activity_rule(
    `id` string COMMENT '编号',
    `activity_id` string  COMMENT '活动ID',
    `condition_amount` decimal(16,2) COMMENT '满减金额',
    `condition_num` bigint COMMENT '满减件数',
    `benefit_amount` decimal(16,2) COMMENT '优惠金额',
    `benefit_discount` decimal(16,2) COMMENT '优惠折扣',
    `benefit_level` string  COMMENT '优惠级别'
) COMMENT '优惠规则表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_activity_rule/';
--3.3.23 编码字典表（全量）
--hive (gmall)>
drop table if exists ods_base_dic;
create external table ods_base_dic(
    `dic_code` string COMMENT '编号',
    `dic_name` string  COMMENT '编码名称',
    `parent_code` string  COMMENT '父编码',
    `create_time` string  COMMENT '创建日期',
    `operate_time` string  COMMENT '操作日期'
) COMMENT '编码字典表'
PARTITIONED BY (`dt` string)
row format delimited fields terminated by '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
location '/warehouse/gmall/ods/ods_base_dic/';
```

###### 数据导入脚本

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else
    do_date=`date -d "-1 day" +%F`
fi

sql1="
load data inpath '/origin_data/$APP/db/order_info/$do_date' OVERWRITE into table ${APP}.ods_order_info partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/order_detail/$do_date' OVERWRITE into table ${APP}.ods_order_detail partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/sku_info/$do_date' OVERWRITE into table ${APP}.ods_sku_info partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/user_info/$do_date' OVERWRITE into table ${APP}.ods_user_info partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/payment_info/$do_date' OVERWRITE into table ${APP}.ods_payment_info partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/base_category1/$do_date' OVERWRITE into table ${APP}.ods_base_category1 partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/base_category2/$do_date' OVERWRITE into table ${APP}.ods_base_category2 partition(dt='$do_date');

load data inpath '/origin_data/$APP/db/base_category3/$do_date' OVERWRITE into table ${APP}.ods_base_category3 partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/base_trademark/$do_date' OVERWRITE into table ${APP}.ods_base_trademark partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/activity_info/$do_date' OVERWRITE into table ${APP}.ods_activity_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/activity_order/$do_date' OVERWRITE into table ${APP}.ods_activity_order partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/cart_info/$do_date' OVERWRITE into table ${APP}.ods_cart_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/comment_info/$do_date' OVERWRITE into table ${APP}.ods_comment_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/coupon_info/$do_date' OVERWRITE into table ${APP}.ods_coupon_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/coupon_use/$do_date' OVERWRITE into table ${APP}.ods_coupon_use partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/favor_info/$do_date' OVERWRITE into table ${APP}.ods_favor_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/order_refund_info/$do_date' OVERWRITE into table ${APP}.ods_order_refund_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/order_status_log/$do_date' OVERWRITE into table ${APP}.ods_order_status_log partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/spu_info/$do_date' OVERWRITE into table ${APP}.ods_spu_info partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/activity_rule/$do_date' OVERWRITE into table ${APP}.ods_activity_rule partition(dt='$do_date'); 

load data inpath '/origin_data/$APP/db/base_dic/$do_date' OVERWRITE into table ${APP}.ods_base_dic partition(dt='$do_date'); 
"

sql2=" 
load data inpath '/origin_data/$APP/db/base_province/$do_date' OVERWRITE into table ${APP}.ods_base_province;

load data inpath '/origin_data/$APP/db/base_region/$do_date' OVERWRITE into table ${APP}.ods_base_region;
"
case $1 in
"first"){
    $hive -e "$sql1$sql2"
};;
"all"){
    $hive -e "$sql1"
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 hdfs_to_ods_db.sh
>
> 导数据  不需建索引
>
> 测试加载数据:
>
> hdfs_to_ods_db.sh first 2021-04-03
>
> hdfs_to_ods_db.sh all 2021-04-04

#### 数仓搭建-DWD层

###### DWD层用户行为数据脚本ods_to_dwd_log.sh

> 同样，在执行脚本前需要创建DWD层的相应的日志表，
>
> 启动日志表、页面日志表、动作日志表、曝光日志表、错误日志表

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
    do_date=$1
else 
    do_date=`date -d "-1 day" +%F`
fi

sql="
SET mapreduce.job.queuename=hive;
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_start_log partition(dt='$do_date')
select 
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.start.entry'),
    get_json_object(line,'$.start.loading_time'),
    get_json_object(line,'$.start.open_ad_id'),
    get_json_object(line,'$.start.open_ad_ms'),
    get_json_object(line,'$.start.open_ad_skip_ms'),
    get_json_object(line,'$.ts')
from ${APP}.ods_log
where dt='$do_date'
and get_json_object(line,'$.start') is not null;

insert overwrite table ${APP}.dwd_page_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.sourceType'),
    get_json_object(line,'$.ts')
from ${APP}.ods_log
where dt='$do_date'
and get_json_object(line,'$.page') is not null;

insert overwrite table ${APP}.dwd_action_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.sourceType'),
    get_json_object(action,'$.action_id'),
    get_json_object(action,'$.item'),
    get_json_object(action,'$.item_type'),
    get_json_object(action,'$.ts')
from ${APP}.ods_log lateral view ${APP}.explode_json_array(get_json_object(line,'$.actions')) tmp as action
where dt='$do_date'
and get_json_object(line,'$.actions') is not null;

insert overwrite table ${APP}.dwd_display_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.sourceType'),
    get_json_object(line,'$.ts'),
    get_json_object(display,'$.displayType'),
    get_json_object(display,'$.item'),
    get_json_object(display,'$.item_type'),
    get_json_object(display,'$.order')
from ${APP}.ods_log lateral view ${APP}.explode_json_array(get_json_object(line,'$.displays')) tmp as display
where dt='$do_date'
and get_json_object(line,'$.displays') is not null;

insert overwrite table ${APP}.dwd_error_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.sourceType'),
    get_json_object(line,'$.start.entry'),
    get_json_object(line,'$.start.loading_time'),
    get_json_object(line,'$.start.open_ad_id'),
    get_json_object(line,'$.start.open_ad_ms'),
    get_json_object(line,'$.start.open_ad_skip_ms'),
    get_json_object(line,'$.actions'),
    get_json_object(line,'$.displays'),
    get_json_object(line,'$.ts'),
    get_json_object(line,'$.err.error_code'),
    get_json_object(line,'$.err.msg')
from ${APP}.ods_log 
where dt='$do_date'
and get_json_object(line,'$.err') is not null;
"

$hive -e "$sql"
```

> 脚本创建后，要给执行权限
>
> chmod 777 ods_to_dwd_log.sh
>
> 导入数据：
>
> ods_to_dwd_log.sh 2021-04-03
>
> 后面使用Azkaban调度时就可以不加日期了

###### DWD层业务数据导入脚本ods_to_dwd_db.sh

| 8个事实表                                                   | 6个维度表                                 |
| :---------------------------------------------------------- | ----------------------------------------- |
| 支付事实表（事务型事实表）dwd_fact_payment_info             | 商品维度表（全量）dwd_dim_sku_info        |
| 退款事实表（事务型事实表）dwd_fact_order_refund_info        | 优惠券维度表（全量）dwd_dim_coupon_info   |
| 评价事实表（事务型事实表）dwd_fact_comment_info             | 活动维度表（全量）dwd_dim_activity_info   |
| 订单明细事实表（事务型事实表）dwd_fact_order_detail         | 地区维度表（特殊）dwd_dim_base_province   |
| 加购事实表（周期型快照事实表，每日快照）dwd_fact_cart_info  | 时间维度表（特殊）dwd_dim_date_info       |
| 收藏事实表（周期型快照事实表，每日快照）dwd_fact_favor_info | 用户维度表（拉链表）dwd_dim_user_info_his |
| 优惠券领用事实表（累积型快照事实表）dwd_fact_coupon_use     | 订单拉链临时表dwd_dim_user_info_his_tmp   |
| 订单事实表（累积型快照事实表）dwd_fact_order_info           | 时间临时表dwd_dim_date_info_tmp           |

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

sql1="
set mapreduce.job.queuename=hive;
set hive.exec.dynamic.partition.mode=nonstrict;
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;

insert overwrite table ${APP}.dwd_dim_sku_info partition(dt='$do_date')
select  
    sku.id,
    sku.spu_id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.tm_id,
    ob.tm_name,
    sku.category3_id,
    c2.id category2_id,
    c1.id category1_id,
    c3.name category3_name,
    c2.name category2_name,
    c1.name category1_name,
    spu.spu_name,
    sku.create_time
from
(
    select * from ${APP}.ods_sku_info where dt='$do_date'
)sku
join
(
    select * from ${APP}.ods_base_trademark where dt='$do_date'
)ob on sku.tm_id=ob.tm_id
join
(
    select * from ${APP}.ods_spu_info where dt='$do_date'
)spu on spu.id = sku.spu_id
join 
(
    select * from ${APP}.ods_base_category3 where dt='$do_date'
)c3 on sku.category3_id=c3.id
join 
(
    select * from ${APP}.ods_base_category2 where dt='$do_date'
)c2 on c3.category2_id=c2.id 
join 
(
    select * from ${APP}.ods_base_category1 where dt='$do_date'
)c1 on c2.category1_id=c1.id;


insert overwrite table ${APP}.dwd_dim_coupon_info partition(dt='$do_date')
select
    id,
    coupon_name,
    coupon_type,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    create_time,
    range_type,
    spu_id,
    tm_id,
    category3_id,
    limit_num,
    operate_time,
    expire_time
from ${APP}.ods_coupon_info
where dt='$do_date';


insert overwrite table ${APP}.dwd_dim_activity_info partition(dt='$do_date')
select
    id,
    activity_name,
    activity_type,
    start_time,
    end_time,
    create_time
from ${APP}.ods_activity_info 
where dt='$do_date';

insert overwrite table ${APP}.dwd_fact_order_detail partition(dt='$do_date')
select
    id,
    order_id,
    user_id,
    sku_id,
    sku_num,
    order_price,
    sku_num,
    create_time,
    province_id,
    source_type,
    source_id,
    original_amount_d,
    if(rn=1,final_total_amount-(sum_div_final_amount-final_amount_d),final_amount_d),
    if(rn=1,feight_fee-(sum_div_feight_fee-feight_fee_d),feight_fee_d),
    if(rn=1,benefit_reduce_amount-(sum_div_benefit_reduce_amount-benefit_reduce_amount_d),benefit_reduce_amount_d)
from
(
    select
        od.id,
        od.order_id,
        od.user_id,
        od.sku_id,
        od.sku_name,
        od.order_price,
        od.sku_num,
        od.create_time,
        oi.province_id,
        od.source_type,
        od.source_id,
        round(od.order_price*od.sku_num,2) original_amount_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.final_total_amount,2) final_amount_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.feight_fee,2) feight_fee_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.benefit_reduce_amount,2) benefit_reduce_amount_d,
        row_number() over(partition by od.order_id order by od.id desc) rn,
        oi.final_total_amount,
        oi.feight_fee,
        oi.benefit_reduce_amount,
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.final_total_amount,2)) over(partition by od.order_id) sum_div_final_amount,
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.feight_fee,2)) over(partition by od.order_id) sum_div_feight_fee,
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.benefit_reduce_amount,2)) over(partition by od.order_id) sum_div_benefit_reduce_amount
    from 
    (
        select * from ${APP}.ods_order_detail where dt='$do_date'
    ) od
    join 
    (
        select * from ${APP}.ods_order_info where dt='$do_date'
    ) oi
    on od.order_id=oi.id
)t1;

insert overwrite table ${APP}.dwd_fact_payment_info partition(dt='$do_date')
select
    pi.id,
    pi.out_trade_no,
    pi.order_id,
    pi.user_id,
    pi.alipay_trade_no,
    pi.total_amount,
    pi.subject,
    pi.payment_type,
    pi.payment_time,          
    oi.province_id
from
(
    select * from ${APP}.ods_payment_info where dt='$do_date'
)pi
join
(
    select id, province_id from ${APP}.ods_order_info where dt='$do_date'
)oi
on pi.order_id = oi.id;


insert overwrite table ${APP}.dwd_fact_order_refund_info partition(dt='$do_date')
select
    id,
    user_id,
    order_id,
    sku_id,
    refund_type,
    refund_num,
    refund_amount,
    refund_reason_type,
    create_time
from ${APP}.ods_order_refund_info
where dt='$do_date';


insert overwrite table ${APP}.dwd_fact_comment_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    spu_id,
    order_id,
    appraise,
    create_time
from ${APP}.ods_comment_info
where dt='$do_date';


insert overwrite table ${APP}.dwd_fact_cart_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    cart_price,
    sku_num,
    sku_name,
    create_time,
    operate_time,
    is_ordered,
    order_time,
    source_type,
    source_id
from ${APP}.ods_cart_info
where dt='$do_date';


insert overwrite table ${APP}.dwd_fact_favor_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    spu_id,
    is_cancel,
    create_time,
    cancel_time
from ${APP}.ods_favor_info
where dt='$do_date';

insert overwrite table ${APP}.dwd_fact_coupon_use partition(dt)
select
    if(new.id is null,old.id,new.id),
    if(new.coupon_id is null,old.coupon_id,new.coupon_id),
    if(new.user_id is null,old.user_id,new.user_id),
    if(new.order_id is null,old.order_id,new.order_id),
    if(new.coupon_status is null,old.coupon_status,new.coupon_status),
    if(new.get_time is null,old.get_time,new.get_time),
    if(new.using_time is null,old.using_time,new.using_time),
    if(new.used_time is null,old.used_time,new.used_time),
    date_format(if(new.get_time is null,old.get_time,new.get_time),'yyyy-MM-dd')
from
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time
    from ${APP}.dwd_fact_coupon_use
    where dt in
    (
        select
            date_format(get_time,'yyyy-MM-dd')
        from ${APP}.ods_coupon_use
        where dt='$do_date'
    )
)old
full outer join
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time
    from ${APP}.ods_coupon_use
    where dt='$do_date'
)new
on old.id=new.id;


insert overwrite table ${APP}.dwd_fact_order_info partition(dt)
select
    if(new.id is null,old.id,new.id),
    if(new.order_status is null,old.order_status,new.order_status),
    if(new.user_id is null,old.user_id,new.user_id),
    if(new.out_trade_no is null,old.out_trade_no,new.out_trade_no),
    if(new.tms['1001'] is null,old.create_time,new.tms['1001']),--1001对应未支付状态
    if(new.tms['1002'] is null,old.payment_time,new.tms['1002']),
    if(new.tms['1003'] is null,old.cancel_time,new.tms['1003']),
    if(new.tms['1004'] is null,old.finish_time,new.tms['1004']),
    if(new.tms['1005'] is null,old.refund_time,new.tms['1005']),
    if(new.tms['1006'] is null,old.refund_finish_time,new.tms['1006']),
    if(new.province_id is null,old.province_id,new.province_id),
    if(new.activity_id is null,old.activity_id,new.activity_id),
    if(new.original_total_amount is null,old.original_total_amount,new.original_total_amount),
    if(new.benefit_reduce_amount is null,old.benefit_reduce_amount,new.benefit_reduce_amount),
    if(new.feight_fee is null,old.feight_fee,new.feight_fee),
    if(new.final_total_amount is null,old.final_total_amount,new.final_total_amount),
    date_format(if(new.tms['1001'] is null,old.create_time,new.tms['1001']),'yyyy-MM-dd')
from
(
    select
        id,
        order_status,
        user_id,
        out_trade_no,
        create_time,
        payment_time,
        cancel_time,
        finish_time,
        refund_time,
        refund_finish_time,
        province_id,
        activity_id,
        original_total_amount,
        benefit_reduce_amount,
        feight_fee,
        final_total_amount
    from ${APP}.dwd_fact_order_info
    where dt
    in
    (
        select
          date_format(create_time,'yyyy-MM-dd')
        from ${APP}.ods_order_info
        where dt='$do_date'
    )
)old
full outer join
(
    select
        info.id,
        info.order_status,
        info.user_id,
        info.out_trade_no,
        info.province_id,
        act.activity_id,
        log.tms,
        info.original_total_amount,
        info.benefit_reduce_amount,
        info.feight_fee,
        info.final_total_amount
    from
    (
        select
            order_id,
            str_to_map(concat_ws(',',collect_set(concat(order_status,'=',operate_time))),',','=') tms
        from ${APP}.ods_order_status_log
        where dt='$do_date'
        group by order_id
    )log
    join
    (
        select * from ${APP}.ods_order_info where dt='$do_date'
    )info
    on log.order_id=info.id
    left join
    (
        select * from ${APP}.ods_activity_order where dt='$do_date'
    )act
    on log.order_id=act.order_id
)new
on old.id=new.id;
"

sql2="
insert overwrite table ${APP}.dwd_dim_base_province
select 
    bp.id,
    bp.name,
    bp.area_code,
    bp.iso_code,
    bp.region_id,
    br.region_name
from ${APP}.ods_base_province bp
join ${APP}.ods_base_region br
on bp.region_id=br.id;
"

sql3="
insert overwrite table ${APP}.dwd_dim_user_info_his_tmp
select * from 
(
    select 
        id,
        name,
        birthday,
        gender,
        email,
        user_level,
        create_time,
        operate_time,
        '$do_date' start_date,
        '9999-99-99' end_date
    from ${APP}.ods_user_info where dt='$do_date'

    union all 
    select 
        uh.id,
        uh.name,
        uh.birthday,
        uh.gender,
        uh.email,
        uh.user_level,
        uh.create_time,
        uh.operate_time,
        uh.start_date,
        if(ui.id is not null  and uh.end_date='9999-99-99', date_add(ui.dt,-1), uh.end_date) end_date
    from ${APP}.dwd_dim_user_info_his uh left join 
    (
        select
            *
        from ${APP}.ods_user_info
        where dt='$do_date'
    ) ui on uh.id=ui.id
)his 
order by his.id, start_date;

insert overwrite table ${APP}.dwd_dim_user_info_his 
select * from ${APP}.dwd_dim_user_info_his_tmp;
"

case $1 in
"first"){
    $hive -e "$sql1$sql2"
};;
"all"){
    $hive -e "$sql1$sql3"
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 ods_to_dwd_db.sh
>
> 导入数据：
>
> ods_to_dwd_db.sh 2021-04-03

#### 数仓搭建-DWS层

###### DWS层数据导入脚本dwd_to_dws.sh

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
    do_date=$1
else
    do_date=`date -d "-1 day" +%F`
fi

sql="
set mapreduce.job.queuename=hive;
with
tmp_start as
(
    select  
        mid_id,
        brand,
        model,
        count(*) login_count
    from ${APP}.dwd_start_log
    where dt='$do_date'
    group by mid_id,brand,model
),
tmp_page as
(
    select
        mid_id,
        brand,
        model,        
        collect_set(named_struct('page_id',page_id,'page_count',page_count)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count
        from ${APP}.dwd_page_log
        where dt='$do_date'
        group by mid_id,brand,model,page_id
    )tmp
    group by mid_id,brand,model
)
insert overwrite table ${APP}.dws_uv_detail_daycount partition(dt='$do_date')
select
    nvl(tmp_start.mid_id,tmp_page.mid_id),
    nvl(tmp_start.brand,tmp_page.brand),
    nvl(tmp_start.model,tmp_page.model),
    tmp_start.login_count,
    tmp_page.page_stats
from tmp_start 
full outer join tmp_page
on tmp_start.mid_id=tmp_page.mid_id
and tmp_start.brand=tmp_page.brand
and tmp_start.model=tmp_page.model;


with
tmp_login as
(
    select
        user_id,
        count(*) login_count
    from ${APP}.dwd_start_log
    where dt='$do_date'
    and user_id is not null
    group by user_id
),
tmp_cart as
(
    select
        user_id,
        count(*) cart_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and user_id is not null
    and action_id='cart_add'
    group by user_id
),tmp_order as
(
    select
        user_id,
        count(*) order_count,
        sum(final_total_amount) order_amount
    from ${APP}.dwd_fact_order_info
    where dt='$do_date'
    group by user_id
) ,
tmp_payment as
(
    select
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_fact_payment_info
    where dt='$do_date'
    group by user_id
),
tmp_order_detail as
(
    select
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'order_amount',order_amount)) order_stats
    from
    (
        select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(final_amount_d) as decimal(20,2)) order_amount
        from ${APP}.dwd_fact_order_detail
        where dt='$do_date'
        group by user_id,sku_id
    )tmp
    group by user_id
)

insert overwrite table ${APP}.dws_user_action_daycount partition(dt='$do_date')
select
    tmp_login.user_id,
    login_count,
    nvl(cart_count,0),
    nvl(order_count,0),
    nvl(order_amount,0.0),
    nvl(payment_count,0),
    nvl(payment_amount,0.0),
    order_stats
from tmp_login
left outer join tmp_cart on tmp_login.user_id=tmp_cart.user_id
left outer join tmp_order on tmp_login.user_id=tmp_order.user_id
left outer join tmp_payment on tmp_login.user_id=tmp_payment.user_id
left outer join tmp_order_detail on tmp_login.user_id=tmp_order_detail.user_id;

with 
tmp_order as
(
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(final_amount_d) order_amount
    from ${APP}.dwd_fact_order_detail
    where dt='$do_date'
    group by sku_id
),
tmp_payment as
(
    select
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(final_amount_d) payment_amount
    from ${APP}.dwd_fact_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and order_id in
    (
        select
            id
        from ${APP}.dwd_fact_order_info
        where (dt='$do_date'
        or dt=date_add('$do_date',-1))
        and date_format(payment_time,'yyyy-MM-dd')='$do_date'
    )
    group by sku_id
),
tmp_refund as
(
    select
        sku_id,
        count(*) refund_count,
        sum(refund_num) refund_num,
        sum(refund_amount) refund_amount
    from ${APP}.dwd_fact_order_refund_info
    where dt='$do_date'
    group by sku_id
),
tmp_cart as
(
    select
        item sku_id,
        count(*) cart_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and user_id is not null
    and action_id='cart_add'
    group by item 
),tmp_favor as
(
    select
        item sku_id,
        count(*) favor_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and user_id is not null
    and action_id='favor_add'
    group by item 
),
tmp_appraise as
(
select
    sku_id,
    sum(if(appraise='1201',1,0)) appraise_good_count,
    sum(if(appraise='1202',1,0)) appraise_mid_count,
    sum(if(appraise='1203',1,0)) appraise_bad_count,
    sum(if(appraise='1204',1,0)) appraise_default_count
from ${APP}.dwd_fact_comment_info
where dt='$do_date'
group by sku_id
)

insert overwrite table ${APP}.dws_sku_action_daycount partition(dt='$do_date')
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_count),
    sum(refund_num),
    sum(refund_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count)
from
(
    select
        sku_id,
        order_count,
        order_num,
        order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_payment
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_count,
        refund_num,
        refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count        
    from tmp_refund
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cart
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_favor
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_appraise
)tmp
group by sku_id;

with 
tmp_login as
(
    select
        area_code,
        count(*) login_count
    from ${APP}.dwd_start_log
    where dt='$do_date'
    group by area_code
),
tmp_op as
(
    select
        province_id,
        sum(if(date_format(create_time,'yyyy-MM-dd')='$do_date',1,0)) order_count,
        sum(if(date_format(create_time,'yyyy-MM-dd')='$do_date',final_total_amount,0)) order_amount,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='$do_date',1,0)) payment_count,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='$do_date',final_total_amount,0)) payment_amount
    from ${APP}.dwd_fact_order_info
    where (dt='$do_date' or dt=date_add('$do_date',-1))
    group by province_id
)
insert overwrite table ${APP}.dws_area_stats_daycount partition(dt='$do_date')
select
    pro.id,
    pro.province_name,
    pro.area_code,
    pro.iso_code,
    pro.region_id,
    pro.region_name,
    nvl(tmp_login.login_count,0),
    nvl(tmp_op.order_count,0),
    nvl(tmp_op.order_amount,0.0),
    nvl(tmp_op.payment_count,0),
    nvl(tmp_op.payment_amount,0.0)
from ${APP}.dwd_dim_base_province pro
left join tmp_login on pro.area_code=tmp_login.area_code
left join tmp_op on pro.id=tmp_op.province_id;


with
tmp_op as
(
    select
        activity_id,
        sum(if(date_format(create_time,'yyyy-MM-dd')='$do_date',1,0)) order_count,
        sum(if(date_format(create_time,'yyyy-MM-dd')='$do_date',final_total_amount,0)) order_amount,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='$do_date',1,0)) payment_count,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='$do_date',final_total_amount,0)) payment_amount
    from ${APP}.dwd_fact_order_info
    where (dt='$do_date' or dt=date_add('$do_date',-1))
    and activity_id is not null
    group by activity_id
),
tmp_display as
(
    select
        item activity_id,
        count(*) display_count
    from ${APP}.dwd_display_log
    where dt='$do_date'
    and item_type='activity_id'
    group by item
),
tmp_activity as
(
    select
        *
    from ${APP}.dwd_dim_activity_info
    where dt='$do_date'
)
insert overwrite table ${APP}.dws_activity_info_daycount partition(dt='$do_date')
select
    nvl(tmp_op.activity_id,tmp_display.activity_id),
    tmp_activity.activity_name,
    tmp_activity.activity_type,
    tmp_activity.start_time,
    tmp_activity.end_time,
    tmp_activity.create_time,
    tmp_display.display_count,
    tmp_op.order_count,
    tmp_op.order_amount,
    tmp_op.payment_count,
    tmp_op.payment_amount
from tmp_op
full outer join tmp_display on tmp_op.activity_id=tmp_display.activity_id
left join tmp_activity on nvl(tmp_op.activity_id,tmp_display.activity_id)=tmp_activity.id;
"

$hive -e "$sql"
```

> 脚本创建后，要给执行权限
>
> chmod 777 dwd_to_dws.sh
>
> 导入数据：
>
> dwd_to_dws.sh 2021-04-03

#### 数仓搭建-DWT层

###### DWT层数据导入脚本dws_to_dwt.sh

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
    do_date=$1
else 
    do_date=`date -d "-1 day" +%F`
fi

sql="
set mapreduce.job.queuename=hive;
insert overwrite table ${APP}.dwt_uv_topic
select
    nvl(new.mid_id,old.mid_id),
    nvl(new.model,old.model),
    nvl(new.brand,old.brand),
    if(old.mid_id is null,'$do_date',old.login_date_first),
    if(new.mid_id is not null,'$do_date',old.login_date_last),
    if(new.mid_id is not null, new.login_count,0),
    nvl(old.login_count,0)+if(new.login_count>0,1,0)
from
(
    select
        *
    from ${APP}.dwt_uv_topic
)old
full outer join
(
    select
        *
    from ${APP}.dws_uv_detail_daycount
    where dt='$do_date'
)new
on old.mid_id=new.mid_id;

insert overwrite table ${APP}.dwt_user_topic
select
    nvl(new.user_id,old.user_id),
    if(old.login_date_first is null and new.login_count>0,'$do_date',old.login_date_first),
    if(new.login_count>0,'$do_date',old.login_date_last),
    nvl(old.login_count,0)+if(new.login_count>0,1,0),
    nvl(new.login_last_30d_count,0),
    if(old.order_date_first is null and new.order_count>0,'$do_date',old.order_date_first),
    if(new.order_count>0,'$do_date',old.order_date_last),
    nvl(old.order_count,0)+nvl(new.order_count,0),
    nvl(old.order_amount,0)+nvl(new.order_amount,0),
    nvl(new.order_last_30d_count,0),
    nvl(new.order_last_30d_amount,0),
    if(old.payment_date_first is null and new.payment_count>0,'$do_date',old.payment_date_first),
    if(new.payment_count>0,'$do_date',old.payment_date_last),
    nvl(old.payment_count,0)+nvl(new.payment_count,0),
    nvl(old.payment_amount,0)+nvl(new.payment_amount,0),
    nvl(new.payment_last_30d_count,0),
    nvl(new.payment_last_30d_amount,0)
from
${APP}.dwt_user_topic old
full outer join
(
    select
        user_id,
        sum(if(dt='$do_date',login_count,0)) login_count,
        sum(if(dt='$do_date',order_count,0)) order_count,
        sum(if(dt='$do_date',order_amount,0)) order_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_count,
        sum(if(dt='$do_date',payment_amount,0)) payment_amount,
        sum(if(login_count>0,1,0)) login_last_30d_count,
        sum(order_count) order_last_30d_count,
        sum(order_amount) order_last_30d_amount,
        sum(payment_count) payment_last_30d_count,
        sum(payment_amount) payment_last_30d_amount
    from ${APP}.dws_user_action_daycount
    where dt>=date_add( '$do_date',-30)
    group by user_id
)new
on old.user_id=new.user_id;

insert overwrite table ${APP}.dwt_sku_topic
select 
    nvl(new.sku_id,old.sku_id),
    sku_info.spu_id,
    nvl(new.order_count30,0),
    nvl(new.order_num30,0),
    nvl(new.order_amount30,0),
    nvl(old.order_count,0) + nvl(new.order_count,0),
    nvl(old.order_num,0) + nvl(new.order_num,0),
    nvl(old.order_amount,0) + nvl(new.order_amount,0),
    nvl(new.payment_count30,0),
    nvl(new.payment_num30,0),
    nvl(new.payment_amount30,0),
    nvl(old.payment_count,0) + nvl(new.payment_count,0),
    nvl(old.payment_num,0) + nvl(new.payment_num,0),
    nvl(old.payment_amount,0) + nvl(new.payment_amount,0),
    nvl(new.refund_count30,0),
    nvl(new.refund_num30,0),
    nvl(new.refund_amount30,0),
    nvl(old.refund_count,0) + nvl(new.refund_count,0),
    nvl(old.refund_num,0) + nvl(new.refund_num,0),
    nvl(old.refund_amount,0) + nvl(new.refund_amount,0),
    nvl(new.cart_count30,0),
    nvl(old.cart_count,0) + nvl(new.cart_count,0),
    nvl(new.favor_count30,0),
    nvl(old.favor_count,0) + nvl(new.favor_count,0),
    nvl(new.appraise_good_count30,0),
    nvl(new.appraise_mid_count30,0),
    nvl(new.appraise_bad_count30,0),
    nvl(new.appraise_default_count30,0)  ,
    nvl(old.appraise_good_count,0) + nvl(new.appraise_good_count,0),
    nvl(old.appraise_mid_count,0) + nvl(new.appraise_mid_count,0),
    nvl(old.appraise_bad_count,0) + nvl(new.appraise_bad_count,0),
    nvl(old.appraise_default_count,0) + nvl(new.appraise_default_count,0) 
from 
(
    select
        sku_id,
        spu_id,
        order_last_30d_count,
        order_last_30d_num,
        order_last_30d_amount,
        order_count,
        order_num,
        order_amount  ,
        payment_last_30d_count,
        payment_last_30d_num,
        payment_last_30d_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_last_30d_count,
        refund_last_30d_num,
        refund_last_30d_amount,
        refund_count,
        refund_num,
        refund_amount,
        cart_last_30d_count,
        cart_count,
        favor_last_30d_count,
        favor_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count 
    from ${APP}.dwt_sku_topic
)old
full outer join 
(
    select 
        sku_id,
        sum(if(dt='$do_date', order_count,0 )) order_count,
        sum(if(dt='$do_date',order_num ,0 ))  order_num, 
        sum(if(dt='$do_date',order_amount,0 )) order_amount ,
        sum(if(dt='$do_date',payment_count,0 )) payment_count,
        sum(if(dt='$do_date',payment_num,0 )) payment_num,
        sum(if(dt='$do_date',payment_amount,0 )) payment_amount,
        sum(if(dt='$do_date',refund_count,0 )) refund_count,
        sum(if(dt='$do_date',refund_num,0 )) refund_num,
        sum(if(dt='$do_date',refund_amount,0 )) refund_amount,  
        sum(if(dt='$do_date',cart_count,0 )) cart_count,
        sum(if(dt='$do_date',favor_count,0 )) favor_count,
        sum(if(dt='$do_date',appraise_good_count,0 )) appraise_good_count,  
        sum(if(dt='$do_date',appraise_mid_count,0 ) ) appraise_mid_count ,
        sum(if(dt='$do_date',appraise_bad_count,0 )) appraise_bad_count,  
        sum(if(dt='$do_date',appraise_default_count,0 )) appraise_default_count,
        sum(order_count) order_count30 ,
        sum(order_num) order_num30,
        sum(order_amount) order_amount30,
        sum(payment_count) payment_count30,
        sum(payment_num) payment_num30,
        sum(payment_amount) payment_amount30,
        sum(refund_count) refund_count30,
        sum(refund_num) refund_num30,
        sum(refund_amount) refund_amount30,
        sum(cart_count) cart_count30,
        sum(favor_count) favor_count30,
        sum(appraise_good_count) appraise_good_count30,
        sum(appraise_mid_count) appraise_mid_count30,
        sum(appraise_bad_count) appraise_bad_count30,
        sum(appraise_default_count) appraise_default_count30 
    from ${APP}.dws_sku_action_daycount
    where dt >= date_add ('$do_date', -30)
    group by sku_id    
)new 
on new.sku_id = old.sku_id
left join 
(select * from ${APP}.dwd_dim_sku_info where dt='$do_date') sku_info
on nvl(new.sku_id,old.sku_id)= sku_info.id;

insert overwrite table ${APP}.dwt_activity_topic
select
    nvl(new.id,old.id),
    nvl(new.activity_name,old.activity_name),
    nvl(new.activity_type,old.activity_type),
    nvl(new.start_time,old.start_time),
    nvl(new.end_time,old.end_time),
    nvl(new.create_time,old.create_time),
    nvl(new.display_count,0),
    nvl(new.order_count,0),
    nvl(new.order_amount,0.0),
    nvl(new.payment_count,0),
    nvl(new.payment_amount,0.0),
    nvl(new.display_count,0)+nvl(old.display_count,0),
    nvl(new.order_count,0)+nvl(old.order_count,0),
    nvl(new.order_amount,0.0)+nvl(old.order_amount,0.0),
    nvl(new.payment_count,0)+nvl(old.payment_count,0),
    nvl(new.payment_amount,0.0)+nvl(old.payment_amount,0.0)
from
(
    select
        *
    from ${APP}.dwt_activity_topic
)old
full outer join
(
    select
        *
    from ${APP}.dws_activity_info_daycount
    where dt='$do_date'
)new
on old.id=new.id;

insert overwrite table ${APP}.dwt_area_topic
select
    nvl(old.id,new.id),
    nvl(old.province_name,new.province_name),
    nvl(old.area_code,new.area_code),
    nvl(old.iso_code,new.iso_code),
    nvl(old.region_id,new.region_id),
    nvl(old.region_name,new.region_name),
    nvl(new.login_day_count,0),
    nvl(new.login_last_30d_count,0),
    nvl(new.order_day_count,0),
    nvl(new.order_day_amount,0.0),
    nvl(new.order_last_30d_count,0),
    nvl(new.order_last_30d_amount,0.0),
    nvl(new.payment_day_count,0),
    nvl(new.payment_day_amount,0.0),
    nvl(new.payment_last_30d_count,0),
    nvl(new.payment_last_30d_amount,0.0)
from 
(
    select
        *
    from ${APP}.dwt_area_topic
)old
full outer join
(
    select
        id,
        province_name,
        area_code,
        iso_code,
        region_id,
        region_name,
        sum(if(dt='$do_date',login_count,0)) login_day_count,
        sum(if(dt='$do_date',order_count,0)) order_day_count,
        sum(if(dt='$do_date',order_amount,0.0)) order_day_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_day_count,
        sum(if(dt='$do_date',payment_amount,0.0)) payment_day_amount,
        sum(login_count) login_last_30d_count,
        sum(order_count) order_last_30d_count,
        sum(order_amount) order_last_30d_amount,
        sum(payment_count) payment_last_30d_count,
        sum(payment_amount) payment_last_30d_amount
    from ${APP}.dws_area_stats_daycount
    where dt>=date_add('$do_date',-30)
    group by id,province_name,area_code,iso_code,region_id,region_name
)new
on old.id=new.id;
"

$hive -e "$sql"

```

> 脚本创建后，要给执行权限
>
> chmod 777 dws_to_dwt.sh 
>
> 导入数据：
>
> dws_to_dwt.sh  2021-04-03

#### 数仓搭建-ADS层

###### ADS层数据导入脚本dwt_to_ads.sh

|                      设备主题                       |
| :-------------------------------------------------: |
|        活跃设备数（日、周、月）ads_uv_count         |
|           每日新增设备数ads_new_mid_count           |
|          留存率ads_user_retention_day_rate          |
|             沉默用户数ads_silent_count              |
|            本周回流用户数ads_back_count             |
|             流失用户数ads_wastage_count             |
|    最近连续三周活跃用户数ads_continuity_wk_count    |
| 最近七天内连续三天活跃用户数ads_continuity_uv_count |

|              会员主题               |
| :---------------------------------: |
|       会员信息ads_user_topic        |
| 漏斗分析ads_user_action_convert_day |

|                     商品主题                      |
| :-----------------------------------------------: |
|           商品个数信息ads_product_info            |
|         商品销量排名ads_product_sale_topN         |
|        商品收藏排名ads_product_favor_topN         |
|      商品加入购物车排名ads_product_cart_topN      |
| 商品退款率排名（最近30天）ads_product_refund_topN |
|          商品差评率ads_appraise_bad_topN          |

|     营销主题（用户+商品+购买行为）      |
| :-------------------------------------: |
|     下单数目统计ads_order_daycount      |
|    支付信息统计ads_payment_daycount     |
| 品牌复购率ads_sale_tm_category1_stat_mn |

|          地区主题          |
| :------------------------: |
| 地区主题信息ads_area_topic |

```bash
#!/bin/bash

APP=gmall
hive=/opt/module/apache-hive-3.1.2-bin/bin/hive

# 如果是输入的日期,则取输入日期;如果没输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
    do_date=$1
else 
    do_date=`date -d 1 day" +%F`
fi

sql="
set mapreduce.job.queuename=hive;
insert into table ${APP}.ads_uv_count 
select  
    '$do_date' dt,
    daycount.ct,
    wkcount.ct,
    mncount.ct,
    if(date_add(next_day('$do_date','MO'),-1)='$do_date','Y','N') ,
    if(last_day('$do_date')='$do_date','Y','N') 
from 
(
    select  
        '$do_date' dt,
        count(*) ct
    from ${APP}.dwt_uv_topic
    where login_date_last='$do_date'  
)daycount join 
( 
    select  
        '$do_date' dt,
        count (*) ct
    from ${APP}.dwt_uv_topic
    where login_date_last>=date_add(next_day('$do_date','MO'),-7) 
    and login_date_last<= date_add(next_day('$do_date','MO'),-1) 
) wkcount on daycount.dt=wkcount.dt
join 
( 
    select  
        '$do_date' dt,
        count (*) ct
    from ${APP}.dwt_uv_topic
    where date_format(login_date_last,'yyyy-MM')=date_format('$do_date','yyyy-MM')  
)mncount on daycount.dt=mncount.dt;

insert into table ${APP}.ads_new_mid_count 
select
    login_date_first,
    count(*)
from ${APP}.dwt_uv_topic
where login_date_first='$do_date'
group by login_date_first;

insert into table ${APP}.ads_silent_count
select
    '$do_date',
    count(*) 
from ${APP}.dwt_uv_topic
where login_date_first=login_date_last
and login_date_last<=date_add('$do_date',-7);


insert into table ${APP}.ads_back_count
select
'$do_date',
concat(date_add(next_day('$do_date','MO'),-7),'_', date_add(next_day('$do_date','MO'),-1)),
    count(*)
from
(
    select
        mid_id
    from ${APP}.dwt_uv_topic
    where login_date_last>=date_add(next_day('$do_date','MO'),-7) 
    and login_date_last<= date_add(next_day('$do_date','MO'),-1)
    and login_date_first<date_add(next_day('$do_date','MO'),-7)
)current_wk
left join
(
    select
        mid_id
    from ${APP}.dws_uv_detail_daycount
    where dt>=date_add(next_day('$do_date','MO'),-7*2) 
    and dt<= date_add(next_day('$do_date','MO'),-7-1) 
    group by mid_id
)last_wk
on current_wk.mid_id=last_wk.mid_id
where last_wk.mid_id is null;

insert into table ${APP}.ads_wastage_count
select
     '$do_date',
     count(*)
from 
(
    select 
        mid_id
    from ${APP}.dwt_uv_topic
    where login_date_last<=date_add('$do_date',-7)
    group by mid_id
)t1;

insert into table ${APP}.ads_user_retention_day_rate
select
    '$do_date',--统计日期
    date_add('$do_date',-1),--新增日期
    1,--留存天数
    sum(if(login_date_first=date_add('$do_date',-1) and login_date_last='$do_date',1,0)),--$do_date的1日留存数
    sum(if(login_date_first=date_add('$do_date',-1),1,0)),--$do_date新增
    sum(if(login_date_first=date_add('$do_date',-1) and login_date_last='$do_date',1,0))/sum(if(login_date_first=date_add('$do_date',-1),1,0))*100
from ${APP}.dwt_uv_topic

union all

select
    '$do_date',--统计日期
    date_add('$do_date',-2),--新增日期
    2,--留存天数
    sum(if(login_date_first=date_add('$do_date',-2) and login_date_last='$do_date',1,0)),--$do_date的2日留存数
    sum(if(login_date_first=date_add('$do_date',-2),1,0)),--$do_date新增
    sum(if(login_date_first=date_add('$do_date',-2) and login_date_last='$do_date',1,0))/sum(if(login_date_first=date_add('$do_date',-2),1,0))*100
from ${APP}.dwt_uv_topic

union all

select
    '$do_date',--统计日期
    date_add('$do_date',-3),--新增日期
    3,--留存天数
    sum(if(login_date_first=date_add('$do_date',-3) and login_date_last='$do_date',1,0)),--$do_date的3日留存数
    sum(if(login_date_first=date_add('$do_date',-3),1,0)),--$do_date新增
    sum(if(login_date_first=date_add('$do_date',-3) and login_date_last='$do_date',1,0))/sum(if(login_date_first=date_add('$do_date',-3),1,0))*100
from ${APP}.dwt_uv_topic;


insert into table ${APP}.ads_continuity_wk_count
select
    '$do_date',
    concat(date_add(next_day('$do_date','MO'),-7*3),'_',date_add(next_day('$do_date','MO'),-1)),
    count(*)
from
(
    select
        mid_id
    from
    (
        select
            mid_id
        from ${APP}.dws_uv_detail_daycount
        where dt>=date_add(next_day('$do_date','monday'),-7)
        and dt<=date_add(next_day('$do_date','monday'),-1)
        group by mid_id

        union all

        select
            mid_id
        from ${APP}.dws_uv_detail_daycount
        where dt>=date_add(next_day('$do_date','monday'),-7*2)
        and dt<=date_add(next_day('$do_date','monday'),-7-1)
        group by mid_id

        union all

        select
            mid_id
        from ${APP}.dws_uv_detail_daycount
        where dt>=date_add(next_day('$do_date','monday'),-7*3)
        and dt<=date_add(next_day('$do_date','monday'),-7*2-1)
        group by mid_id
    )t1
    group by mid_id
    having count(*)=3
)t2;


insert into table ${APP}.ads_continuity_uv_count
select
    '$do_date',
    concat(date_add('$do_date',-6),'_','$do_date'),
    count(*)
from
(
    select mid_id
    from
    (
        select mid_id      
        from
        (
            select 
                mid_id,
                date_sub(dt,rank) date_dif
            from
            (
                select 
                    mid_id,
                    dt,
                    rank() over(partition by mid_id order by dt) rank
                from ${APP}.dws_uv_detail_daycount
                where dt>=date_add('$do_date',-6) and dt<='$do_date'
            )t1
        )t2 
        group by mid_id,date_dif
        having count(*)>=3
    )t3 
    group by mid_id
)t4;


insert into table ${APP}.ads_user_topic
select
    '$do_date',
    sum(if(login_date_last='$do_date',1,0)),
    sum(if(login_date_first='$do_date',1,0)),
    sum(if(payment_date_first='$do_date',1,0)),
    sum(if(payment_count>0,1,0)),
    count(*),
    sum(if(login_date_last='$do_date',1,0))/count(*),
    sum(if(payment_count>0,1,0))/count(*),
    sum(if(login_date_first='$do_date',1,0))/sum(if(login_date_last='$do_date',1,0))
from ${APP}.dwt_user_topic;

with
tmp_uv as
(
    select
        '$do_date' dt,
        sum(if(array_contains(pages,'home'),1,0)) home_count,
        sum(if(array_contains(pages,'good_detail'),1,0)) good_detail_count
    from
    (
        select
            mid_id,
            collect_set(page_id) pages
        from ${APP}.dwd_page_log
        where dt='$do_date'
        and page_id in ('home','good_detail')
        group by mid_id
    )tmp
),
tmp_cop as
(
    select 
        '$do_date' dt,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from ${APP}.dws_user_action_daycount
    where dt='$do_date'
)
insert into table ${APP}.ads_user_action_convert_day
select
    tmp_uv.dt,
    tmp_uv.home_count,
    tmp_uv.good_detail_count,
    tmp_uv.good_detail_count/tmp_uv.home_count*100,
    tmp_cop.cart_count,
    tmp_cop.cart_count/tmp_uv.good_detail_count*100,
    tmp_cop.order_count,
    tmp_cop.order_count/tmp_cop.cart_count*100,
    tmp_cop.payment_count,
    tmp_cop.payment_count/tmp_cop.order_count*100
from tmp_uv
join tmp_cop
on tmp_uv.dt=tmp_cop.dt;

insert into table ${APP}.ads_product_info
select
    '$do_date' dt,
    sku_num,
    spu_num
from
(
    select
        '$do_date' dt,
        count(*) sku_num
    from
        ${APP}.dwt_sku_topic
) tmp_sku_num
join
(
    select
        '$do_date' dt,
        count(*) spu_num
    from
    (
        select
            spu_id
        from
            ${APP}.dwt_sku_topic
        group by
            spu_id
    ) tmp_spu_id
) tmp_spu_num
on
    tmp_sku_num.dt=tmp_spu_num.dt;


insert into table ${APP}.ads_product_sale_topN
select
    '$do_date' dt,
    sku_id,
    payment_amount
from
    ${APP}.dws_sku_action_daycount
where
    dt='$do_date'
order by payment_amount desc
limit 10;

insert into table ${APP}.ads_product_favor_topN
select
    '$do_date' dt,
    sku_id,
    favor_count
from
    ${APP}.dws_sku_action_daycount
where
    dt='$do_date'
order by favor_count desc
limit 10;

insert into table ${APP}.ads_product_cart_topN
select
    '$do_date' dt,
    sku_id,
    cart_count
from
    ${APP}.dws_sku_action_daycount
where
    dt='$do_date'
order by cart_count desc
limit 10;


insert into table ${APP}.ads_product_refund_topN
select
    '$do_date',
    sku_id,
    refund_last_30d_count/payment_last_30d_count*100 refund_ratio
from ${APP}.dwt_sku_topic
order by refund_ratio desc
limit 10;


insert into table ${APP}.ads_appraise_bad_topN
select
    '$do_date' dt,
    sku_id,
appraise_bad_count/(appraise_good_count+appraise_mid_count+appraise_bad_count+appraise_default_count) appraise_bad_ratio
from
    ${APP}.dws_sku_action_daycount
where
    dt='$do_date'
order by appraise_bad_ratio desc
limit 10;


insert into table ${APP}.ads_order_daycount
select
    '$do_date',
    sum(order_count),
    sum(order_amount),
    sum(if(order_count>0,1,0))
from ${APP}.dws_user_action_daycount
where dt='$do_date';


insert into table ${APP}.ads_payment_daycount
select
    tmp_payment.dt,
    tmp_payment.payment_count,
    tmp_payment.payment_amount,
    tmp_payment.payment_user_count,
    tmp_skucount.payment_sku_count,
    tmp_time.payment_avg_time
from
(
    select
        '$do_date' dt,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(payment_count>0,1,0)) payment_user_count
    from ${APP}.dws_user_action_daycount
    where dt='$do_date'
)tmp_payment
join
(
    select
        '$do_date' dt,
        sum(if(payment_count>0,1,0)) payment_sku_count 
    from ${APP}.dws_sku_action_daycount
    where dt='$do_date'
)tmp_skucount on tmp_payment.dt=tmp_skucount.dt
join
(
    select
        '$do_date' dt,
        sum(unix_timestamp(payment_time)-unix_timestamp(create_time))/count(*)/60 payment_avg_time
    from ${APP}.dwd_fact_order_info
    where dt='$do_date'
    and payment_time is not null
)tmp_time on tmp_payment.dt=tmp_time.dt;


with 
tmp_order as
(
    select
        user_id,
        order_stats_struct.sku_id sku_id,
        order_stats_struct.order_count order_count
    from ${APP}.dws_user_action_daycount lateral view explode(order_detail_stats) tmp as order_stats_struct
    where date_format(dt,'yyyy-MM')=date_format('$do_date','yyyy-MM')
),
tmp_sku as
(
    select
        id,
        tm_id,
        category1_id,
        category1_name
    from ${APP}.dwd_dim_sku_info
    where dt='$do_date'
)
insert into table ${APP}.ads_sale_tm_category1_stat_mn
select
    tm_id,
    category1_id,
    category1_name,
    sum(if(order_count>=1,1,0)) buycount,
    sum(if(order_count>=2,1,0)) buyTwiceLast,
    sum(if(order_count>=2,1,0))/sum( if(order_count>=1,1,0)) buyTwiceLastRatio,
    sum(if(order_count>=3,1,0))  buy3timeLast  ,
    sum(if(order_count>=3,1,0))/sum( if(order_count>=1,1,0)) buy3timeLastRatio ,
    date_format('$do_date' ,'yyyy-MM') stat_mn,
    '$do_date' stat_date
from
(
    select 
        tmp_order.user_id,
        tmp_sku.category1_id,
        tmp_sku.category1_name,
        tmp_sku.tm_id,
        sum(order_count) order_count
    from tmp_order
    join tmp_sku
    on tmp_order.sku_id=tmp_sku.id
    group by tmp_order.user_id,tmp_sku.category1_id,tmp_sku.category1_name,tmp_sku.tm_id
)tmp
group by tm_id, category1_id, category1_name;

insert into table ${APP}.ads_area_topic
select
    '$do_date',
    id,
    province_name,
    area_code,
    iso_code,
    region_id,
    region_name,
    login_day_count,
    order_day_count,
    order_day_amount,
    payment_day_count,
    payment_day_amount
from ${APP}.dwt_area_topic;
"

$hive -e "$sql"
```

> 脚本创建后，要给执行权限
>
> chmod 777 dwt_to_ads.sh
>
> 导入数据：
>
> dwt_to_ads.sh 2021-04-03

## 数仓搭建4.0

##### ——数仓搭建-ODS层——

> 先创建支持lzo压缩的分区表，再通过下面的脚本实现ODS层用户行为数据的加载和建索引

###### 建表语句(1个)

> 没有指定队列
>
> -Dmapreduce.job.queuename=hive

```sql
-- ODS层日志数据建表语句（1个） --
-- 创建支持lzo压缩的分区表
-- OUTPUTFORMAT 对于load方式没用
drop table if exists ods_log;
CREATE EXTERNAL TABLE ods_log (`line` string)
PARTITIONED BY (`dt` string) -- 按照时间创建分区
STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_log' -- 指定数据在hdfs上的存储位置
;

-- 使用脚本进行数据装载和创建索引
-- 数据装载
-- 2021-01-01
-- load data inpath '/origin_data/gmall/log/topic_log/2021-01-01' into table ods_log partition(dt='2021-01-01');
-- 为支持lzo压缩压缩的分区表创建索引（不在DataGrip中执行）
-- hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_log/dt=2021-01-01
```

###### ODS层日志表加载数据脚本hdfs_to_ods_log.sh

> /home/ahua/bin/hdfs_to_ods_log.sh

> 没有定义
>
> hive=/opt/module/apache-hive-3.1.2-bin/bin/hive
>
> hadoop=/opt/module/hadoop-3.2.2/bin/hadoop
>
> 没有指定队列
>
>  -Dmapreduce.job.queuename=hive

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$1" ] ;then
   do_date=$1
else
   do_date=`date -d "-1 day" +%F`
fi

echo ================== 日志日期为 $do_date ==================
sql="
load data inpath '/origin_data/$APP/log/topic_log/$do_date' into table ${APP}.ods_log partition(dt='$do_date');
"

hive -e "$sql"

# 创建索引
hadoop jar /opt/module/hadoop-3.2.2/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer  /warehouse/$APP/ods/ods_log/dt=$do_date
```

> 脚本创建后，要给执行权限
>
> chmod 777 hdfs_to_ods_log.sh
>
> 导数据 建索引 （按文档进行）
>
> 测试加载数据：hdfs_to_ods_log.sh 2021-01-01

###### ODS层业务表首日数据装载脚本hdfs_to_ods_db_init.sh

> /home/ahua/bin/hdfs_to_ods_db_init.sh
> 因为有历史数据，所以区分首日每日

###### 建表语句（27个）

```sql
-- 业务数据建表完成后不需要创建索引（日志数据需要），因为在使用sqoop从MySQL导数据到HDFS时就已经创建索引了
-- ODS层业务数据建表语句（27个）--
-- 01.活动信息表
DROP TABLE IF EXISTS ods_activity_info;
CREATE EXTERNAL TABLE ods_activity_info(
    `id` STRING COMMENT '编号',
    `activity_name` STRING  COMMENT '活动名称',
    `activity_type` STRING  COMMENT '活动类型',
    `start_time` STRING  COMMENT '开始时间',
    `end_time` STRING  COMMENT '结束时间',
    `create_time` STRING  COMMENT '创建时间'
) COMMENT '活动信息表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_activity_info/';
-- 02.活动规则表
DROP TABLE IF EXISTS ods_activity_rule;
CREATE EXTERNAL TABLE ods_activity_rule(
    `id` STRING COMMENT '编号',
    `activity_id` STRING  COMMENT '活动ID',
    `activity_type` STRING COMMENT '活动类型',
    `condition_amount` DECIMAL(16,2) COMMENT '满减金额',
    `condition_num` BIGINT COMMENT '满减件数',
    `benefit_amount` DECIMAL(16,2) COMMENT '优惠金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '优惠折扣',
    `benefit_level` STRING COMMENT '优惠级别'
) COMMENT '活动规则表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_activity_rule/';
-- 03.一级品类表
DROP TABLE IF EXISTS ods_base_category1;
CREATE EXTERNAL TABLE ods_base_category1(
    `id` STRING COMMENT 'id',
    `name` STRING COMMENT '名称'
) COMMENT '商品一级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category1/';
-- 04.二级品类表
DROP TABLE IF EXISTS ods_base_category2;
CREATE EXTERNAL TABLE ods_base_category2(
    `id` STRING COMMENT ' id',
    `name` STRING COMMENT '名称',
    `category1_id` STRING COMMENT '一级品类id'
) COMMENT '商品二级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category2/';
-- 05.三级品类表
DROP TABLE IF EXISTS ods_base_category3;
CREATE EXTERNAL TABLE ods_base_category3(
    `id` STRING COMMENT ' id',
    `name` STRING COMMENT '名称',
    `category2_id` STRING COMMENT '二级品类id'
) COMMENT '商品三级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category3/';
-- 06.编码字典表
DROP TABLE IF EXISTS ods_base_dic;
CREATE EXTERNAL TABLE ods_base_dic(
    `dic_code` STRING COMMENT '编号',
    `dic_name` STRING COMMENT '编码名称',
    `parent_code` STRING COMMENT '父编码',
    `create_time` STRING COMMENT '创建日期',
    `operate_time` STRING COMMENT '操作日期'
) COMMENT '编码字典表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_dic/';
-- 07.省份表
DROP TABLE IF EXISTS ods_base_province;
CREATE EXTERNAL TABLE ods_base_province (
    `id` STRING COMMENT '编号',
    `name` STRING COMMENT '省份名称',
    `region_id` STRING COMMENT '地区ID',
    `area_code` STRING COMMENT '地区编码',
    `iso_code` STRING COMMENT 'ISO-3166编码，供可视化使用',
    `iso_3166_2` STRING COMMENT 'IOS-3166-2编码，供可视化使用'
)  COMMENT '省份表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_province/';
-- 08.地区表
DROP TABLE IF EXISTS ods_base_region;
CREATE EXTERNAL TABLE ods_base_region (
    `id` STRING COMMENT '编号',
    `region_name` STRING COMMENT '地区名称'
)  COMMENT '地区表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_region/';
-- 09.品牌表
DROP TABLE IF EXISTS ods_base_trademark;
CREATE EXTERNAL TABLE ods_base_trademark (
    `id` STRING COMMENT '编号',
    `tm_name` STRING COMMENT '品牌名称'
)  COMMENT '品牌表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_trademark/';
-- 10.购物车表
DROP TABLE IF EXISTS ods_cart_info;
CREATE EXTERNAL TABLE ods_cart_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户id',
    `sku_id` STRING COMMENT 'skuid',
    `cart_price` DECIMAL(16,2)  COMMENT '放入购物车时价格',
    `sku_num` BIGINT COMMENT '数量',
    `sku_name` STRING COMMENT 'sku名称 (冗余)',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '修改时间',
    `is_ordered` STRING COMMENT '是否已经下单',
    `order_time` STRING COMMENT '下单时间',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号'
) COMMENT '加购表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_cart_info/';
-- 11.评论表
DROP TABLE IF EXISTS ods_comment_info;
CREATE EXTERNAL TABLE ods_comment_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `sku_id` STRING COMMENT '商品sku',
    `spu_id` STRING COMMENT '商品spu',
    `order_id` STRING COMMENT '订单ID',
    `appraise` STRING COMMENT '评价',
    `create_time` STRING COMMENT '评价时间'
) COMMENT '商品评论表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_comment_info/';
-- 12.优惠券信息表
DROP TABLE IF EXISTS ods_coupon_info;
CREATE EXTERNAL TABLE ods_coupon_info(
    `id` STRING COMMENT '购物券编号',
    `coupon_name` STRING COMMENT '购物券名称',
    `coupon_type` STRING COMMENT '购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券',
    `condition_amount` DECIMAL(16,2) COMMENT '满额数',
    `condition_num` BIGINT COMMENT '满件数',
    `activity_id` STRING COMMENT '活动编号',
    `benefit_amount` DECIMAL(16,2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '折扣',
    `create_time` STRING COMMENT '创建时间',
    `range_type` STRING COMMENT '范围类型 1、商品 2、品类 3、品牌',
    `limit_num` BIGINT COMMENT '最多领用次数',
    `taken_count` BIGINT COMMENT '已领用次数',
    `start_time` STRING COMMENT '开始领取时间',
    `end_time` STRING COMMENT '结束领取时间',
    `operate_time` STRING COMMENT '修改时间',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_coupon_info/';
-- 13.优惠券领用表
DROP TABLE IF EXISTS ods_coupon_use;
CREATE EXTERNAL TABLE ods_coupon_use(
    `id` STRING COMMENT '编号',
    `coupon_id` STRING  COMMENT '优惠券ID',
    `user_id` STRING  COMMENT 'skuid',
    `order_id` STRING  COMMENT 'spuid',
    `coupon_status` STRING  COMMENT '优惠券状态',
    `get_time` STRING  COMMENT '领取时间',
    `using_time` STRING  COMMENT '使用时间(下单)',
    `used_time` STRING  COMMENT '使用时间(支付)',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券领用表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_coupon_use/';
-- 14.收藏表
DROP TABLE IF EXISTS ods_favor_info;
CREATE EXTERNAL TABLE ods_favor_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户id',
    `sku_id` STRING COMMENT 'skuid',
    `spu_id` STRING COMMENT 'spuid',
    `is_cancel` STRING COMMENT '是否取消',
    `create_time` STRING COMMENT '收藏时间',
    `cancel_time` STRING COMMENT '取消时间'
) COMMENT '商品收藏表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_favor_info/';
-- 15.订单明细表
DROP TABLE IF EXISTS ods_order_detail;
CREATE EXTERNAL TABLE ods_order_detail(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `sku_id` STRING COMMENT '商品id',
    `sku_name` STRING COMMENT '商品名称',
    `order_price` DECIMAL(16,2) COMMENT '商品价格',
    `sku_num` BIGINT COMMENT '商品数量',
    `create_time` STRING COMMENT '创建时间',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号',
    `split_final_amount` DECIMAL(16,2) COMMENT '分摊最终金额',
    `split_activity_amount` DECIMAL(16,2) COMMENT '分摊活动优惠',
    `split_coupon_amount` DECIMAL(16,2) COMMENT '分摊优惠券优惠'
) COMMENT '订单详情表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail/';
-- 16.订单明细活动关联表
DROP TABLE IF EXISTS ods_order_detail_activity;
CREATE EXTERNAL TABLE ods_order_detail_activity(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `order_detail_id` STRING COMMENT '订单明细id',
    `activity_id` STRING COMMENT '活动id',
    `activity_rule_id` STRING COMMENT '活动规则id',
    `sku_id` BIGINT COMMENT '商品id',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '订单详情活动关联表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail_activity/';
-- 17.订单明细优惠券关联表
DROP TABLE IF EXISTS ods_order_detail_coupon;
CREATE EXTERNAL TABLE ods_order_detail_coupon(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `order_detail_id` STRING COMMENT '订单明细id',
    `coupon_id` STRING COMMENT '优惠券id',
    `coupon_use_id` STRING COMMENT '优惠券领用记录id',
    `sku_id` STRING COMMENT '商品id',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '订单详情活动关联表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail_coupon/';
-- 18.订单表
DROP TABLE IF EXISTS ods_order_info;
CREATE EXTERNAL TABLE ods_order_info (
    `id` STRING COMMENT '订单号',
    `final_amount` DECIMAL(16,2) COMMENT '订单最终金额',
    `order_status` STRING COMMENT '订单状态',
    `user_id` STRING COMMENT '用户id',
    `payment_way` STRING COMMENT '支付方式',
    `delivery_address` STRING COMMENT '送货地址',
    `out_trade_no` STRING COMMENT '支付流水号',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间',
    `expire_time` STRING COMMENT '过期时间',
    `tracking_no` STRING COMMENT '物流单编号',
    `province_id` STRING COMMENT '省份ID',
    `activity_reduce_amount` DECIMAL(16,2) COMMENT '活动减免金额',
    `coupon_reduce_amount` DECIMAL(16,2) COMMENT '优惠券减免金额',
    `original_amount` DECIMAL(16,2)  COMMENT '订单原价金额',
    `feight_fee` DECIMAL(16,2)  COMMENT '运费',
    `feight_fee_reduce` DECIMAL(16,2)  COMMENT '运费减免'
) COMMENT '订单表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_info/';
-- 19.退单表
DROP TABLE IF EXISTS ods_order_refund_info;
CREATE EXTERNAL TABLE ods_order_refund_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `order_id` STRING COMMENT '订单ID',
    `sku_id` STRING COMMENT '商品ID',
    `refund_type` STRING COMMENT '退单类型',
    `refund_num` BIGINT COMMENT '退单件数',
    `refund_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_reason_type` STRING COMMENT '退单原因类型',
    `refund_status` STRING COMMENT '退单状态',--退单状态应包含买家申请、卖家审核、卖家收货、退款完成等状态。此处未涉及到，故该表按增量处理
    `create_time` STRING COMMENT '退单时间'
) COMMENT '退单表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_refund_info/';
-- 20.订单状态日志表
DROP TABLE IF EXISTS ods_order_status_log;
CREATE EXTERNAL TABLE ods_order_status_log (
    `id` STRING COMMENT '编号',
    `order_id` STRING COMMENT '订单ID',
    `order_status` STRING COMMENT '订单状态',
    `operate_time` STRING COMMENT '修改时间'
)  COMMENT '订单状态表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_status_log/';
-- 21.支付表
DROP TABLE IF EXISTS ods_payment_info;
CREATE EXTERNAL TABLE ods_payment_info(
    `id` STRING COMMENT '编号',
    `out_trade_no` STRING COMMENT '对外业务编号',
    `order_id` STRING COMMENT '订单编号',
    `user_id` STRING COMMENT '用户编号',
    `payment_type` STRING COMMENT '支付类型',
    `trade_no` STRING COMMENT '交易编号',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `subject` STRING COMMENT '交易内容',
    `payment_status` STRING COMMENT '支付状态',
    `create_time` STRING COMMENT '创建时间',
    `callback_time` STRING COMMENT '回调时间'
)  COMMENT '支付流水表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_payment_info/';
-- 22.退款表
DROP TABLE IF EXISTS ods_refund_payment;
CREATE EXTERNAL TABLE ods_refund_payment(
    `id` STRING COMMENT '编号',
    `out_trade_no` STRING COMMENT '对外业务编号',
    `order_id` STRING COMMENT '订单编号',
    `sku_id` STRING COMMENT 'SKU编号',
    `payment_type` STRING COMMENT '支付类型',
    `trade_no` STRING COMMENT '交易编号',
    `refund_amount` DECIMAL(16,2) COMMENT '支付金额',
    `subject` STRING COMMENT '交易内容',
    `refund_status` STRING COMMENT '支付状态',
    `create_time` STRING COMMENT '创建时间',
    `callback_time` STRING COMMENT '回调时间'
)  COMMENT '支付流水表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_refund_payment/';
-- 23.商品平台属性表
DROP TABLE IF EXISTS ods_sku_attr_value;
CREATE EXTERNAL TABLE ods_sku_attr_value(
    `id` STRING COMMENT '编号',
    `attr_id` STRING COMMENT '平台属性ID',
    `value_id` STRING COMMENT '平台属性值ID',
    `sku_id` STRING COMMENT '商品ID',
    `attr_name` STRING COMMENT '平台属性名称',
    `value_name` STRING COMMENT '平台属性值名称'
) COMMENT 'sku平台属性表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_attr_value/';
-- 24.商品（SKU）表
DROP TABLE IF EXISTS ods_sku_info;
CREATE EXTERNAL TABLE ods_sku_info(
    `id` STRING COMMENT 'skuId',
    `spu_id` STRING COMMENT 'spuid',
    `price` DECIMAL(16,2) COMMENT '价格',
    `sku_name` STRING COMMENT '商品名称',
    `sku_desc` STRING COMMENT '商品描述',
    `weight` DECIMAL(16,2) COMMENT '重量',
    `tm_id` STRING COMMENT '品牌id',
    `category3_id` STRING COMMENT '品类id',
    `is_sale` STRING COMMENT '是否在售',
    `create_time` STRING COMMENT '创建时间'
) COMMENT 'SKU商品表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_info/';
-- 25.商品销售属性表
DROP TABLE IF EXISTS ods_sku_sale_attr_value;
CREATE EXTERNAL TABLE ods_sku_sale_attr_value(
    `id` STRING COMMENT '编号',
    `sku_id` STRING COMMENT 'sku_id',
    `spu_id` STRING COMMENT 'spu_id',
    `sale_attr_value_id` STRING COMMENT '销售属性值id',
    `sale_attr_id` STRING COMMENT '销售属性id',
    `sale_attr_name` STRING COMMENT '销售属性名称',
    `sale_attr_value_name` STRING COMMENT '销售属性值名称'
) COMMENT 'sku销售属性名称'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_sale_attr_value/';
-- 26.商品（SPU）表
DROP TABLE IF EXISTS ods_spu_info;
CREATE EXTERNAL TABLE ods_spu_info(
    `id` STRING COMMENT 'spuid',
    `spu_name` STRING COMMENT 'spu名称',
    `category3_id` STRING COMMENT '品类id',
    `tm_id` STRING COMMENT '品牌id'
) COMMENT 'SPU商品表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_spu_info/';
-- 27.用户表
DROP TABLE IF EXISTS ods_user_info;
CREATE EXTERNAL TABLE ods_user_info(
    `id` STRING COMMENT '用户id',
    `login_name` STRING COMMENT '用户名称',
    `nick_name` STRING COMMENT '用户昵称',
    `name` STRING COMMENT '用户姓名',
    `phone_num` STRING COMMENT '手机号码',
    `email` STRING COMMENT '邮箱',
    `user_level` STRING COMMENT '用户等级',
    `birthday` STRING COMMENT '生日',
    `gender` STRING COMMENT '性别',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间'
) COMMENT '用户表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_user_info/';
```

###### 首日数据装载脚本hdfs_to_ods_db_init.sh

> 没有定义
>
> hive=/opt/module/apache-hive-3.1.2-bin/bin/hive
>
> 命令格式：hdfs_to_ods_db_init.sh [all] [必须输入指定第一天日期]
>
> 该脚本只执行一次
>
> hdfs_to_ods_db_init.sh all 2021-01-01

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

ods_order_info=" 
load data inpath '/origin_data/$APP/db/order_info/$do_date' OVERWRITE into table ${APP}.ods_order_info partition(dt='$do_date');"

ods_order_detail="
load data inpath '/origin_data/$APP/db/order_detail/$do_date' OVERWRITE into table ${APP}.ods_order_detail partition(dt='$do_date');"

ods_sku_info="
load data inpath '/origin_data/$APP/db/sku_info/$do_date' OVERWRITE into table ${APP}.ods_sku_info partition(dt='$do_date');"

ods_user_info="
load data inpath '/origin_data/$APP/db/user_info/$do_date' OVERWRITE into table ${APP}.ods_user_info partition(dt='$do_date');"

ods_payment_info="
load data inpath '/origin_data/$APP/db/payment_info/$do_date' OVERWRITE into table ${APP}.ods_payment_info partition(dt='$do_date');"

ods_base_category1="
load data inpath '/origin_data/$APP/db/base_category1/$do_date' OVERWRITE into table ${APP}.ods_base_category1 partition(dt='$do_date');"

ods_base_category2="
load data inpath '/origin_data/$APP/db/base_category2/$do_date' OVERWRITE into table ${APP}.ods_base_category2 partition(dt='$do_date');"

ods_base_category3="
load data inpath '/origin_data/$APP/db/base_category3/$do_date' OVERWRITE into table ${APP}.ods_base_category3 partition(dt='$do_date'); "

ods_base_trademark="
load data inpath '/origin_data/$APP/db/base_trademark/$do_date' OVERWRITE into table ${APP}.ods_base_trademark partition(dt='$do_date'); "

ods_activity_info="
load data inpath '/origin_data/$APP/db/activity_info/$do_date' OVERWRITE into table ${APP}.ods_activity_info partition(dt='$do_date'); "

ods_cart_info="
load data inpath '/origin_data/$APP/db/cart_info/$do_date' OVERWRITE into table ${APP}.ods_cart_info partition(dt='$do_date'); "

ods_comment_info="
load data inpath '/origin_data/$APP/db/comment_info/$do_date' OVERWRITE into table ${APP}.ods_comment_info partition(dt='$do_date'); "

ods_coupon_info="
load data inpath '/origin_data/$APP/db/coupon_info/$do_date' OVERWRITE into table ${APP}.ods_coupon_info partition(dt='$do_date'); "

ods_coupon_use="
load data inpath '/origin_data/$APP/db/coupon_use/$do_date' OVERWRITE into table ${APP}.ods_coupon_use partition(dt='$do_date'); "

ods_favor_info="
load data inpath '/origin_data/$APP/db/favor_info/$do_date' OVERWRITE into table ${APP}.ods_favor_info partition(dt='$do_date'); "

ods_order_refund_info="
load data inpath '/origin_data/$APP/db/order_refund_info/$do_date' OVERWRITE into table ${APP}.ods_order_refund_info partition(dt='$do_date'); "

ods_order_status_log="
load data inpath '/origin_data/$APP/db/order_status_log/$do_date' OVERWRITE into table ${APP}.ods_order_status_log partition(dt='$do_date'); "

ods_spu_info="
load data inpath '/origin_data/$APP/db/spu_info/$do_date' OVERWRITE into table ${APP}.ods_spu_info partition(dt='$do_date'); "

ods_activity_rule="
load data inpath '/origin_data/$APP/db/activity_rule/$do_date' OVERWRITE into table ${APP}.ods_activity_rule partition(dt='$do_date');" 

ods_base_dic="
load data inpath '/origin_data/$APP/db/base_dic/$do_date' OVERWRITE into table ${APP}.ods_base_dic partition(dt='$do_date'); "

ods_order_detail_activity="
load data inpath '/origin_data/$APP/db/order_detail_activity/$do_date' OVERWRITE into table ${APP}.ods_order_detail_activity partition(dt='$do_date'); "

ods_order_detail_coupon="
load data inpath '/origin_data/$APP/db/order_detail_coupon/$do_date' OVERWRITE into table ${APP}.ods_order_detail_coupon partition(dt='$do_date'); "

ods_refund_payment="
load data inpath '/origin_data/$APP/db/refund_payment/$do_date' OVERWRITE into table ${APP}.ods_refund_payment partition(dt='$do_date'); "

ods_sku_attr_value="
load data inpath '/origin_data/$APP/db/sku_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_attr_value partition(dt='$do_date'); "

ods_sku_sale_attr_value="
load data inpath '/origin_data/$APP/db/sku_sale_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_sale_attr_value partition(dt='$do_date'); "

ods_base_province=" 
load data inpath '/origin_data/$APP/db/base_province/$do_date' OVERWRITE into table ${APP}.ods_base_province;"

ods_base_region="
load data inpath '/origin_data/$APP/db/base_region/$do_date' OVERWRITE into table ${APP}.ods_base_region;"

case $1 in
    "ods_order_info"){
        hive -e "$ods_order_info"
    };;
    "ods_order_detail"){
        hive -e "$ods_order_detail"
    };;
    "ods_sku_info"){
        hive -e "$ods_sku_info"
    };;
    "ods_user_info"){
        hive -e "$ods_user_info"
    };;
    "ods_payment_info"){
        hive -e "$ods_payment_info"
    };;
    "ods_base_category1"){
        hive -e "$ods_base_category1"
    };;
    "ods_base_category2"){
        hive -e "$ods_base_category2"
    };;
    "ods_base_category3"){
        hive -e "$ods_base_category3"
    };;
    "ods_base_trademark"){
        hive -e "$ods_base_trademark"
    };;
    "ods_activity_info"){
        hive -e "$ods_activity_info"
    };;
    "ods_cart_info"){
        hive -e "$ods_cart_info"
    };;
    "ods_comment_info"){
        hive -e "$ods_comment_info"
    };;
    "ods_coupon_info"){
        hive -e "$ods_coupon_info"
    };;
    "ods_coupon_use"){
        hive -e "$ods_coupon_use"
    };;
    "ods_favor_info"){
        hive -e "$ods_favor_info"
    };;
    "ods_order_refund_info"){
        hive -e "$ods_order_refund_info"
    };;
    "ods_order_status_log"){
        hive -e "$ods_order_status_log"
    };;
    "ods_spu_info"){
        hive -e "$ods_spu_info"
    };;
    "ods_activity_rule"){
        hive -e "$ods_activity_rule"
    };;
    "ods_base_dic"){
        hive -e "$ods_base_dic"
    };;
    "ods_order_detail_activity"){
        hive -e "$ods_order_detail_activity"
    };;
    "ods_order_detail_coupon"){
        hive -e "$ods_order_detail_coupon"
    };;
    "ods_refund_payment"){
        hive -e "$ods_refund_payment"
    };;
    "ods_sku_attr_value"){
        hive -e "$ods_sku_attr_value"
    };;
    "ods_sku_sale_attr_value"){
        hive -e "$ods_sku_sale_attr_value"
    };;
    "ods_base_province"){
        hive -e "$ods_base_province"
    };;
    "ods_base_region"){
        hive -e "$ods_base_region"
    };;
    "all"){
        hive -e "$ods_order_info$ods_order_detail$ods_sku_info$ods_user_info$ods_payment_info$ods_base_category1$ods_base_category2$ods_base_category3$ods_base_trademark$ods_activity_info$ods_cart_info$ods_comment_info$ods_coupon_info$ods_coupon_use$ods_favor_info$ods_order_refund_info$ods_order_status_log$ods_spu_info$ods_activity_rule$ods_base_dic$ods_order_detail_activity$ods_order_detail_coupon$ods_refund_payment$ods_sku_attr_value$ods_sku_sale_attr_value$ods_base_province$ods_base_region"
    };;
esac
```

###### 每日数据装载脚本hdfs_to_ods_db_daily.sh

> /home/ahua/bin/hdfs_to_ods_db_daily.sh
> 因为有历史数据，所以区分首日每日
>
> 没有定义hadoop hive
>
> hive=/opt/module/apache-hive-3.1.2-bin/bin/hive
>
> 命令格式：hdfs_to_ods_db_daily.sh [all] [可输入指定日期]
>
> 该脚本每日执行

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

ods_order_info=" 
load data inpath '/origin_data/$APP/db/order_info/$do_date' OVERWRITE into table ${APP}.ods_order_info partition(dt='$do_date');"

ods_order_detail="
load data inpath '/origin_data/$APP/db/order_detail/$do_date' OVERWRITE into table ${APP}.ods_order_detail partition(dt='$do_date');"

ods_sku_info="
load data inpath '/origin_data/$APP/db/sku_info/$do_date' OVERWRITE into table ${APP}.ods_sku_info partition(dt='$do_date');"

ods_user_info="
load data inpath '/origin_data/$APP/db/user_info/$do_date' OVERWRITE into table ${APP}.ods_user_info partition(dt='$do_date');"

ods_payment_info="
load data inpath '/origin_data/$APP/db/payment_info/$do_date' OVERWRITE into table ${APP}.ods_payment_info partition(dt='$do_date');"

ods_base_category1="
load data inpath '/origin_data/$APP/db/base_category1/$do_date' OVERWRITE into table ${APP}.ods_base_category1 partition(dt='$do_date');"

ods_base_category2="
load data inpath '/origin_data/$APP/db/base_category2/$do_date' OVERWRITE into table ${APP}.ods_base_category2 partition(dt='$do_date');"

ods_base_category3="
load data inpath '/origin_data/$APP/db/base_category3/$do_date' OVERWRITE into table ${APP}.ods_base_category3 partition(dt='$do_date'); "

ods_base_trademark="
load data inpath '/origin_data/$APP/db/base_trademark/$do_date' OVERWRITE into table ${APP}.ods_base_trademark partition(dt='$do_date'); "

ods_activity_info="
load data inpath '/origin_data/$APP/db/activity_info/$do_date' OVERWRITE into table ${APP}.ods_activity_info partition(dt='$do_date'); "

ods_cart_info="
load data inpath '/origin_data/$APP/db/cart_info/$do_date' OVERWRITE into table ${APP}.ods_cart_info partition(dt='$do_date'); "

ods_comment_info="
load data inpath '/origin_data/$APP/db/comment_info/$do_date' OVERWRITE into table ${APP}.ods_comment_info partition(dt='$do_date'); "

ods_coupon_info="
load data inpath '/origin_data/$APP/db/coupon_info/$do_date' OVERWRITE into table ${APP}.ods_coupon_info partition(dt='$do_date'); "

ods_coupon_use="
load data inpath '/origin_data/$APP/db/coupon_use/$do_date' OVERWRITE into table ${APP}.ods_coupon_use partition(dt='$do_date'); "

ods_favor_info="
load data inpath '/origin_data/$APP/db/favor_info/$do_date' OVERWRITE into table ${APP}.ods_favor_info partition(dt='$do_date'); "

ods_order_refund_info="
load data inpath '/origin_data/$APP/db/order_refund_info/$do_date' OVERWRITE into table ${APP}.ods_order_refund_info partition(dt='$do_date'); "

ods_order_status_log="
load data inpath '/origin_data/$APP/db/order_status_log/$do_date' OVERWRITE into table ${APP}.ods_order_status_log partition(dt='$do_date'); "

ods_spu_info="
load data inpath '/origin_data/$APP/db/spu_info/$do_date' OVERWRITE into table ${APP}.ods_spu_info partition(dt='$do_date'); "

ods_activity_rule="
load data inpath '/origin_data/$APP/db/activity_rule/$do_date' OVERWRITE into table ${APP}.ods_activity_rule partition(dt='$do_date');" 

ods_base_dic="
load data inpath '/origin_data/$APP/db/base_dic/$do_date' OVERWRITE into table ${APP}.ods_base_dic partition(dt='$do_date'); "

ods_order_detail_activity="
load data inpath '/origin_data/$APP/db/order_detail_activity/$do_date' OVERWRITE into table ${APP}.ods_order_detail_activity partition(dt='$do_date'); "

ods_order_detail_coupon="
load data inpath '/origin_data/$APP/db/order_detail_coupon/$do_date' OVERWRITE into table ${APP}.ods_order_detail_coupon partition(dt='$do_date'); "

ods_refund_payment="
load data inpath '/origin_data/$APP/db/refund_payment/$do_date' OVERWRITE into table ${APP}.ods_refund_payment partition(dt='$do_date'); "

ods_sku_attr_value="
load data inpath '/origin_data/$APP/db/sku_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_attr_value partition(dt='$do_date'); "

ods_sku_sale_attr_value="
load data inpath '/origin_data/$APP/db/sku_sale_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_sale_attr_value partition(dt='$do_date'); "

ods_base_province=" 
load data inpath '/origin_data/$APP/db/base_province/$do_date' OVERWRITE into table ${APP}.ods_base_province;"

ods_base_region="
load data inpath '/origin_data/$APP/db/base_region/$do_date' OVERWRITE into table ${APP}.ods_base_region;"

case $1 in
    "ods_order_info"){
        hive -e "$ods_order_info"
    };;
    "ods_order_detail"){
        hive -e "$ods_order_detail"
    };;
    "ods_sku_info"){
        hive -e "$ods_sku_info"
    };;
    "ods_user_info"){
        hive -e "$ods_user_info"
    };;
    "ods_payment_info"){
        hive -e "$ods_payment_info"
    };;
    "ods_base_category1"){
        hive -e "$ods_base_category1"
    };;
    "ods_base_category2"){
        hive -e "$ods_base_category2"
    };;
    "ods_base_category3"){
        hive -e "$ods_base_category3"
    };;
    "ods_base_trademark"){
        hive -e "$ods_base_trademark"
    };;
    "ods_activity_info"){
        hive -e "$ods_activity_info"
    };;
    "ods_cart_info"){
        hive -e "$ods_cart_info"
    };;
    "ods_comment_info"){
        hive -e "$ods_comment_info"
    };;
    "ods_coupon_info"){
        hive -e "$ods_coupon_info"
    };;
    "ods_coupon_use"){
        hive -e "$ods_coupon_use"
    };;
    "ods_favor_info"){
        hive -e "$ods_favor_info"
    };;
    "ods_order_refund_info"){
        hive -e "$ods_order_refund_info"
    };;
    "ods_order_status_log"){
        hive -e "$ods_order_status_log"
    };;
    "ods_spu_info"){
        hive -e "$ods_spu_info"
    };;
    "ods_activity_rule"){
        hive -e "$ods_activity_rule"
    };;
    "ods_base_dic"){
        hive -e "$ods_base_dic"
    };;
    "ods_order_detail_activity"){
        hive -e "$ods_order_detail_activity"
    };;
    "ods_order_detail_coupon"){
        hive -e "$ods_order_detail_coupon"
    };;
    "ods_refund_payment"){
        hive -e "$ods_refund_payment"
    };;
    "ods_sku_attr_value"){
        hive -e "$ods_sku_attr_value"
    };;
    "ods_sku_sale_attr_value"){
        hive -e "$ods_sku_sale_attr_value"
    };;
    "all"){
        hive -e "$ods_order_info$ods_order_detail$ods_sku_info$ods_user_info$ods_payment_info$ods_base_category1$ods_base_category2$ods_base_category3$ods_base_trademark$ods_activity_info$ods_cart_info$ods_comment_info$ods_coupon_info$ods_coupon_use$ods_favor_info$ods_order_refund_info$ods_order_status_log$ods_spu_info$ods_activity_rule$ods_base_dic$ods_order_detail_activity$ods_order_detail_coupon$ods_refund_payment$ods_sku_attr_value$ods_sku_sale_attr_value"
    };;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 hdfs_to_ods_db_daily.sh
>
> 导数据  不需建索引

##### ——数仓搭建-DIM层——

> 该层维度表共有6个
>
> 商品维度表（全量）dim_sku_info
>
> 优惠券维度表（全量）dim_coupon_info
>
> 活动维度表（全量）dim_activity_rule_info
>
> 地区维度表（特殊）dim_base_province
>
> 时间维度表（特殊）dim_date_info	时间临时表tmp_dim_date_info
>
> 用户维度表（拉链表）dim_user_info

###### 建表语句（5/6）个

```sql
-- DIM维度层建表语句（6个）--
-- 01.商品维度表（全量）
DROP TABLE IF EXISTS dim_sku_info;
CREATE EXTERNAL TABLE dim_sku_info (
    `id` STRING COMMENT '商品id',
    `price` DECIMAL(16,2) COMMENT '商品价格',
    `sku_name` STRING COMMENT '商品名称',
    `sku_desc` STRING COMMENT '商品描述',
    `weight` DECIMAL(16,2) COMMENT '重量',
    `is_sale` BOOLEAN COMMENT '是否在售',
    `spu_id` STRING COMMENT 'spu编号',
    `spu_name` STRING COMMENT 'spu名称',
    `category3_id` STRING COMMENT '三级分类id',
    `category3_name` STRING COMMENT '三级分类名称',
    `category2_id` STRING COMMENT '二级分类id',
    `category2_name` STRING COMMENT '二级分类名称',
    `category1_id` STRING COMMENT '一级分类id',
    `category1_name` STRING COMMENT '一级分类名称',
    `tm_id` STRING COMMENT '品牌id',
    `tm_name` STRING COMMENT '品牌名称',
    `sku_attr_values` ARRAY<STRUCT<attr_id:STRING,value_id:STRING,attr_name:STRING,value_name:STRING>> COMMENT '平台属性',
    `sku_sale_attr_values` ARRAY<STRUCT<sale_attr_id:STRING,sale_attr_value_id:STRING,sale_attr_name:STRING,sale_attr_value_name:STRING>> COMMENT '销售属性',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '商品维度表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_sku_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 02.优惠券维度表（全量）
DROP TABLE IF EXISTS dim_coupon_info;
CREATE EXTERNAL TABLE dim_coupon_info(
    `id` STRING COMMENT '购物券编号',
    `coupon_name` STRING COMMENT '购物券名称',
    `coupon_type` STRING COMMENT '购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券',
    `condition_amount` DECIMAL(16,2) COMMENT '满额数',
    `condition_num` BIGINT COMMENT '满件数',
    `activity_id` STRING COMMENT '活动编号',
    `benefit_amount` DECIMAL(16,2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '折扣',
    `create_time` STRING COMMENT '创建时间',
    `range_type` STRING COMMENT '范围类型 1、商品 2、品类 3、品牌',
    `limit_num` BIGINT COMMENT '最多领取次数',
    `taken_count` BIGINT COMMENT '已领取次数',
    `start_time` STRING COMMENT '可以领取的开始日期',
    `end_time` STRING COMMENT '可以领取的结束日期',
    `operate_time` STRING COMMENT '修改时间',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券维度表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_coupon_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 03.活动维度表（全量）
DROP TABLE IF EXISTS dim_activity_rule_info;
CREATE EXTERNAL TABLE dim_activity_rule_info(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING COMMENT '活动ID',
    `activity_name` STRING  COMMENT '活动名称',
    `activity_type` STRING  COMMENT '活动类型',
    `start_time` STRING  COMMENT '开始时间',
    `end_time` STRING  COMMENT '结束时间',
    `create_time` STRING  COMMENT '创建时间',
    `condition_amount` DECIMAL(16,2) COMMENT '满减金额',
    `condition_num` BIGINT COMMENT '满减件数',
    `benefit_amount` DECIMAL(16,2) COMMENT '优惠金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '优惠折扣',
    `benefit_level` STRING COMMENT '优惠级别'
) COMMENT '活动信息表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_activity_rule_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 04.地区维度表（特殊）——不需要分区
DROP TABLE IF EXISTS dim_base_province;
CREATE EXTERNAL TABLE dim_base_province (
    `id` STRING COMMENT 'id',
    `province_name` STRING COMMENT '省市名称',
    `area_code` STRING COMMENT '地区编码',
    `iso_code` STRING COMMENT 'ISO-3166编码，供可视化使用',
    `iso_3166_2` STRING COMMENT 'IOS-3166-2编码，供可视化使用',
    `region_id` STRING COMMENT '地区id',
    `region_name` STRING COMMENT '地区名称'
) COMMENT '地区维度表'
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_base_province/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 06.用户维度表（拉链表）
DROP TABLE IF EXISTS dim_user_info;
CREATE EXTERNAL TABLE dim_user_info(
    `id` STRING COMMENT '用户id',
    `login_name` STRING COMMENT '用户名称',
    `nick_name` STRING COMMENT '用户昵称',
    `name` STRING COMMENT '用户姓名',
    `phone_num` STRING COMMENT '手机号码',
    `email` STRING COMMENT '邮箱',
    `user_level` STRING COMMENT '用户等级',
    `birthday` STRING COMMENT '生日',
    `gender` STRING COMMENT '性别',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间',
    `start_date` STRING COMMENT '开始日期',
    `end_date` STRING COMMENT '结束日期'
) COMMENT '用户表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_user_info/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

###### 时间维度表（特殊）建表语句与装载

```sql
-- 05.时间维度表（特殊）——不需要分区，每年手动导入
DROP TABLE IF EXISTS dim_date_info;
CREATE EXTERNAL TABLE dim_date_info(
    `date_id` STRING COMMENT '日',
    `week_id` STRING COMMENT '周ID',
    `week_day` STRING COMMENT '周几',
    `day` STRING COMMENT '每月的第几天',
    `month` STRING COMMENT '第几月',
    `quarter` STRING COMMENT '第几季度',
    `year` STRING COMMENT '年',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_date_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 创建时间维度表的临时表
DROP TABLE IF EXISTS tmp_dim_date_info;
CREATE EXTERNAL TABLE tmp_dim_date_info (
    `date_id` STRING COMMENT '日',
    `week_id` STRING COMMENT '周ID',
    `week_day` STRING COMMENT '周几',
    `day` STRING COMMENT '每月的第几天',
    `month` STRING COMMENT '第几月',
    `quarter` STRING COMMENT '第几季度',
    `year` STRING COMMENT '年',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/tmp/tmp_dim_date_info/';

-- 上传date_info.txt,tmp_dim_date_info表中就会存在数据了
-- 数据装载完整后，dim_date_info表中就会存在数据了
insert overwrite table dim_date_info select * from tmp_dim_date_info;
```

数据装载例子

> 商品维度表首日数据装载语句（后续用脚本执行）

> ```sql
> with
> sku as
> (
>     select
>         id,
>         price,
>         sku_name,
>         sku_desc,
>         weight,
>         is_sale,
>         spu_id,
>         category3_id,
>         tm_id,
>         create_time
>     from ods_sku_info
>     where dt='2021-01-01'
> ),
> spu as
> (
>     select
>         id,
>         spu_name
>     from ods_spu_info
>     where dt='2021-01-01'
> ),
> c3 as
> (
>     select
>         id,
>         name,
>         category2_id
>     from ods_base_category3
>     where dt='2021-01-01'
> ),
> c2 as
> (
>     select
>         id,
>         name,
>         category1_id
>     from ods_base_category2
>     where dt='2021-01-01'
> ),
> c1 as
> (
>     select
>         id,
>         name
>     from ods_base_category1
>     where dt='2021-01-01'
> ),
> tm as
> (
>     select
>         id,
>         tm_name
>     from ods_base_trademark
>     where dt='2021-01-01'
> ),
> attr as
> (
>     select
>         sku_id,
>         collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
>     from ods_sku_attr_value
>     where dt='2021-01-01'
>     group by sku_id
> ),
> sale_attr as
> (
>     select
>         sku_id,
>         collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
>     from ods_sku_sale_attr_value
>     where dt='2021-01-01'
>     group by sku_id
> )
> insert overwrite table dim_sku_info partition(dt='2021-01-01')
> select
>     sku.id,
>     sku.price,
>     sku.sku_name,
>     sku.sku_desc,
>     sku.weight,
>     sku.is_sale,
>     sku.spu_id,
>     spu.spu_name,
>     sku.category3_id,
>     c3.name,
>     c3.category2_id,
>     c2.name,
>     c2.category1_id,
>     c1.name,
>     sku.tm_id,
>     tm.tm_name,
>     attr.attrs,
>     sale_attr.sale_attrs,
>     sku.create_time
> from sku
> left join spu on sku.spu_id=spu.id
> left join c3 on sku.category3_id=c3.id
> left join c2 on c3.category2_id=c2.id
> left join c1 on c2.category1_id=c1.id
> left join tm on sku.tm_id=tm.id
> left join attr on sku.id=attr.sku_id
> left join sale_attr on sku.id=sale_attr.sku_id;
> ```

###### DIM层首日数据装载脚本ods_to_dim_db_init.sh

> /home/ahua/bin/ods_to_dim_db_init.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

# 商品维度表（全量）
dim_sku_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
with
sku as
(
    select
        id,
        price,
        sku_name,
        sku_desc,
        weight,
        is_sale,
        spu_id,
        category3_id,
        tm_id,
        create_time
    from ${APP}.ods_sku_info
    where dt='$do_date'
),
spu as
(
    select
        id,
        spu_name
    from ${APP}.ods_spu_info
    where dt='$do_date'
),
c3 as
(
    select
        id,
        name,
        category2_id
    from ${APP}.ods_base_category3
    where dt='$do_date'
),
c2 as
(
    select
        id,
        name,
        category1_id
    from ${APP}.ods_base_category2
    where dt='$do_date'
),
c1 as
(
    select
        id,
        name
    from ${APP}.ods_base_category1
    where dt='$do_date'
),
tm as
(
    select
        id,
        tm_name
    from ${APP}.ods_base_trademark
    where dt='$do_date'
),
attr as
(
    select
        sku_id,
        collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
    from ${APP}.ods_sku_attr_value
    where dt='$do_date'
    group by sku_id
),
sale_attr as
(
    select
        sku_id,
        collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
    from ${APP}.ods_sku_sale_attr_value
    where dt='$do_date'
    group by sku_id
)

insert overwrite table ${APP}.dim_sku_info partition(dt='$do_date')
select
    sku.id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.is_sale,
    sku.spu_id,
    spu.spu_name,
    sku.category3_id,
    c3.name,
    c3.category2_id,
    c2.name,
    c2.category1_id,
    c1.name,
    sku.tm_id,
    tm.tm_name,
    attr.attrs,
    sale_attr.sale_attrs,
    sku.create_time
from sku
left join spu on sku.spu_id=spu.id
left join c3 on sku.category3_id=c3.id
left join c2 on c3.category2_id=c2.id
left join c1 on c2.category1_id=c1.id
left join tm on sku.tm_id=tm.id
left join attr on sku.id=attr.sku_id
left join sale_attr on sku.id=sale_attr.sku_id;
"

# 优惠券维度表（全量）
dim_coupon_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_coupon_info partition(dt='$do_date')
select
    id,
    coupon_name,
    coupon_type,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    create_time,
    range_type,
    limit_num,
    taken_count,
    start_time,
    end_time,
    operate_time,
    expire_time
from ${APP}.ods_coupon_info
where dt='$do_date';
"

# 活动维度表（全量）
dim_activity_rule_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_activity_rule_info partition(dt='$do_date')
select
    ar.id,
    ar.activity_id,
    ai.activity_name,
    ar.activity_type,
    ai.start_time,
    ai.end_time,
    ai.create_time,
    ar.condition_amount,
    ar.condition_num,
    ar.benefit_amount,
    ar.benefit_discount,
    ar.benefit_level
from
(
    select
        id,
        activity_id,
        activity_type,
        condition_amount,
        condition_num,
        benefit_amount,
        benefit_discount,
        benefit_level
    from ${APP}.ods_activity_rule
    where dt='$do_date'
)ar
left join
(
    select
        id,
        activity_name,
        start_time,
        end_time,
        create_time
    from ${APP}.ods_activity_info
    where dt='$do_date'
)ai
on ar.activity_id=ai.id;
"

# 地区维度表（特殊）——不需要分区，不需要每日装载
dim_base_province="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_base_province
select
    bp.id,
    bp.name,
    bp.area_code,
    bp.iso_code,
    bp.iso_3166_2,
    bp.region_id,
    br.region_name
from ${APP}.ods_base_province bp
join ${APP}.ods_base_region br on bp.region_id = br.id;
"

# 用户维度表（拉链表）
dim_user_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_user_info partition(dt='9999-99-99')
select
    id,
    login_name,
    nick_name,
    md5(name),
    md5(phone_num),
    md5(email),
    user_level,
    birthday,
    gender,
    create_time,
    operate_time,
    '$do_date',
    '9999-99-99'
from ${APP}.ods_user_info
where dt='$do_date';
"

case $1 in
"dim_sku_info"){
    hive -e "$dim_sku_info"
};;
"dim_coupon_info"){
    hive -e "$dim_coupon_info"
};;
"dim_activity_rule_info"){
    hive -e "$dim_activity_rule_info"
};;
"dim_base_province"){
    hive -e "$dim_base_province"
};;
"dim_user_info"){
    hive -e "$dim_user_info"
};;
"all"){
    hive -e "$dim_sku_info$dim_coupon_info$dim_activity_rule_info$dim_base_province$dim_user_info"
};;
esac
```

###### DIM层每日数据装载脚本ods_to_dim_db_daily.sh

> /home/ahua/bin/ods_to_dim_db_daily.sh
>
> 地区维度表即便是输入的all参数，也不会装载，要想装载，必须显式的输入dim_base_province
>
> 即`ods_to_dim_db_daily.sh dim_base_province`命令，从ODS层导入DIM层

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

# 商品维度表（全量）
dim_sku_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
with
sku as
(
    select
        id,
        price,
        sku_name,
        sku_desc,
        weight,
        is_sale,
        spu_id,
        category3_id,
        tm_id,
        create_time
    from ${APP}.ods_sku_info
    where dt='$do_date'
),
spu as
(
    select
        id,
        spu_name
    from ${APP}.ods_spu_info
    where dt='$do_date'
),
c3 as
(
    select
        id,
        name,
        category2_id
    from ${APP}.ods_base_category3
    where dt='$do_date'
),
c2 as
(
    select
        id,
        name,
        category1_id
    from ${APP}.ods_base_category2
    where dt='$do_date'
),
c1 as
(
    select
        id,
        name
    from ${APP}.ods_base_category1
    where dt='$do_date'
),
tm as
(
    select
        id,
        tm_name
    from ${APP}.ods_base_trademark
    where dt='$do_date'
),
attr as
(
    select
        sku_id,
        collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
    from ${APP}.ods_sku_attr_value
    where dt='$do_date'
    group by sku_id
),
sale_attr as
(
    select
        sku_id,
        collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
    from ${APP}.ods_sku_sale_attr_value
    where dt='$do_date'
    group by sku_id
)

insert overwrite table ${APP}.dim_sku_info partition(dt='$do_date')
select
    sku.id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.is_sale,
    sku.spu_id,
    spu.spu_name,
    sku.category3_id,
    c3.name,
    c3.category2_id,
    c2.name,
    c2.category1_id,
    c1.name,
    sku.tm_id,
    tm.tm_name,
    attr.attrs,
    sale_attr.sale_attrs,
    sku.create_time
from sku
left join spu on sku.spu_id=spu.id
left join c3 on sku.category3_id=c3.id
left join c2 on c3.category2_id=c2.id
left join c1 on c2.category1_id=c1.id
left join tm on sku.tm_id=tm.id
left join attr on sku.id=attr.sku_id
left join sale_attr on sku.id=sale_attr.sku_id;
"

# 优惠券维度表（全量）
dim_coupon_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_coupon_info partition(dt='$do_date')
select
    id,
    coupon_name,
    coupon_type,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    create_time,
    range_type,
    limit_num,
    taken_count,
    start_time,
    end_time,
    operate_time,
    expire_time
from ${APP}.ods_coupon_info
where dt='$do_date';
"

# 活动维度表（全量）
dim_activity_rule_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_activity_rule_info partition(dt='$do_date')
select
    ar.id,
    ar.activity_id,
    ai.activity_name,
    ar.activity_type,
    ai.start_time,
    ai.end_time,
    ai.create_time,
    ar.condition_amount,
    ar.condition_num,
    ar.benefit_amount,
    ar.benefit_discount,
    ar.benefit_level
from
(
    select
        id,
        activity_id,
        activity_type,
        condition_amount,
        condition_num,
        benefit_amount,
        benefit_discount,
        benefit_level
    from ${APP}.ods_activity_rule
    where dt='$do_date'
)ar
left join
(
    select
        id,
        activity_name,
        start_time,
        end_time,
        create_time
    from ${APP}.ods_activity_info
    where dt='$do_date'
)ai
on ar.activity_id=ai.id;
"

# 地区维度表（特殊）——不需要分区，不需要每日装载
dim_base_province="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_base_province
select
    bp.id,
    bp.name,
    bp.area_code,
    bp.iso_code,
    bp.iso_3166_2,
    bp.region_id,
    bp.name
from ${APP}.ods_base_province bp
join ${APP}.ods_base_region br on bp.region_id = br.id;
"

# 用户维度表（拉链表）
# 使用动态分区，要设置为非严格模式
dim_user_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
with
tmp as
(
    select
        old.id old_id,
        old.login_name old_login_name,
        old.nick_name old_nick_name,
        old.name old_name,
        old.phone_num old_phone_num,
        old.email old_email,
        old.user_level old_user_level,
        old.birthday old_birthday,
        old.gender old_gender,
        old.create_time old_create_time,
        old.operate_time old_operate_time,
        old.start_date old_start_date,
        old.end_date old_end_date,
        new.id new_id,
        new.login_name new_login_name,
        new.nick_name new_nick_name,
        new.name new_name,
        new.phone_num new_phone_num,
        new.email new_email,
        new.user_level new_user_level,
        new.birthday new_birthday,
        new.gender new_gender,
        new.create_time new_create_time,
        new.operate_time new_operate_time,
        new.start_date new_start_date,
        new.end_date new_end_date
    from
    (
        select
            id,
            login_name,
            nick_name,
            name,
            phone_num,
            email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            start_date,
            end_date
        from ${APP}.dim_user_info
        where dt='9999-99-99'
        and start_date<'$do_date'
    )old
    full outer join
    (
        select
            id,
            login_name,
            nick_name,
            md5(name) name,
            md5(phone_num) phone_num,
            md5(email) email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            '$do_date' start_date,
            '9999-99-99' end_date
        from ${APP}.ods_user_info
        where dt='$do_date'
    )new
    on old.id=new.id
)
insert overwrite table ${APP}.dim_user_info partition(dt)
select
    nvl(new_id,old_id),
    nvl(new_login_name,old_login_name),
    nvl(new_nick_name,old_nick_name),
    nvl(new_name,old_name),
    nvl(new_phone_num,old_phone_num),
    nvl(new_email,old_email),
    nvl(new_user_level,old_user_level),
    nvl(new_birthday,old_birthday),
    nvl(new_gender,old_gender),
    nvl(new_create_time,old_create_time),
    nvl(new_operate_time,old_operate_time),
    nvl(new_start_date,old_start_date),
    nvl(new_end_date,old_end_date),
    nvl(new_end_date,old_end_date) dt
from tmp
union all
select
    old_id,
    old_login_name,
    old_nick_name,
    old_name,
    old_phone_num,
    old_email,
    old_user_level,
    old_birthday,
    old_gender,
    old_create_time,
    old_operate_time,
    old_start_date,
    cast(date_add('$do_date',-1) as string),
    cast(date_add('$do_date',-1) as string) dt
from tmp
where new_id is not null and old_id is not null;
"

case $1 in
"dim_sku_info"){
    hive -e "$dim_sku_info"
};;
"dim_coupon_info"){
    hive -e "$dim_coupon_info"
};;
"dim_activity_rule_info"){
    hive -e "$dim_activity_rule_info"
};;
"dim_base_province"){
    hive -e "$dim_base_province"
};;
"dim_user_info"){
    hive -e "$dim_user_info"
};;
"all"){
    hive -e "$dim_sku_info$dim_coupon_info$dim_activity_rule_info$dim_user_info"
};;
esac
```

##### ——数仓搭建-DWD层——

###### DWD层用户行为数据建表语句（5个）

```sql
-- DWD层日志数据建表语句（5个） --
-- 01.启动日志表
DROP TABLE IF EXISTS dwd_start_log;
CREATE EXTERNAL TABLE dwd_start_log(
    `area_code` STRING COMMENT '地区编码',
    `brand` STRING COMMENT '手机品牌',
    `channel` STRING COMMENT '渠道',
    `is_new` STRING COMMENT '是否首次启动',
    `model` STRING COMMENT '手机型号',
    `mid_id` STRING COMMENT '设备id',
    `os` STRING COMMENT '操作系统',
    `user_id` STRING COMMENT '会员id',
    `version_code` STRING COMMENT 'app版本号',
    `entry` STRING COMMENT 'icon手机图标 notice 通知 install 安装后启动',
    `loading_time` BIGINT COMMENT '启动加载时间',
    `open_ad_id` STRING COMMENT '广告页ID ',
    `open_ad_ms` BIGINT COMMENT '广告总共播放时间',
    `open_ad_skip_ms` BIGINT COMMENT '用户跳过广告时点',
    `ts` BIGINT COMMENT '时间'
) COMMENT '启动日志表'
PARTITIONED BY (`dt` STRING) -- 按照时间创建分区
STORED AS PARQUET -- 采用parquet列式存储
LOCATION '/warehouse/gmall/dwd/dwd_start_log' -- 指定在HDFS上存储位置
TBLPROPERTIES('parquet.compression'='lzo') -- 采用LZO压缩
;

-- 02.页面日志表
DROP TABLE IF EXISTS dwd_page_log;
CREATE EXTERNAL TABLE dwd_page_log(
    `area_code` STRING COMMENT '地区编码',
    `brand` STRING COMMENT '手机品牌',
    `channel` STRING COMMENT '渠道',
    `is_new` STRING COMMENT '是否首次启动',
    `model` STRING COMMENT '手机型号',
    `mid_id` STRING COMMENT '设备id',
    `os` STRING COMMENT '操作系统',
    `user_id` STRING COMMENT '会员id',
    `version_code` STRING COMMENT 'app版本号',
    `during_time` BIGINT COMMENT '持续时间毫秒',
    `page_item` STRING COMMENT '目标id ',
    `page_item_type` STRING COMMENT '目标类型',
    `last_page_id` STRING COMMENT '上页类型',
    `page_id` STRING COMMENT '页面ID ',
    `source_type` STRING COMMENT '来源类型',
    `ts` bigint
) COMMENT '页面日志表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_page_log'
TBLPROPERTIES('parquet.compression'='lzo');

-- 03.动作日志表（UDTF函数处理）
-- 需要创建永久函数
DROP TABLE IF EXISTS dwd_action_log;
CREATE EXTERNAL TABLE dwd_action_log(
    `area_code` STRING COMMENT '地区编码',
    `brand` STRING COMMENT '手机品牌',
    `channel` STRING COMMENT '渠道',
    `is_new` STRING COMMENT '是否首次启动',
    `model` STRING COMMENT '手机型号',
    `mid_id` STRING COMMENT '设备id',
    `os` STRING COMMENT '操作系统',
    `user_id` STRING COMMENT '会员id',
    `version_code` STRING COMMENT 'app版本号',
    `during_time` BIGINT COMMENT '持续时间毫秒',
    `page_item` STRING COMMENT '目标id ',
    `page_item_type` STRING COMMENT '目标类型',
    `last_page_id` STRING COMMENT '上页类型',
    `page_id` STRING COMMENT '页面id ',
    `source_type` STRING COMMENT '来源类型',
    `action_id` STRING COMMENT '动作id',
    `item` STRING COMMENT '目标id ',
    `item_type` STRING COMMENT '目标类型',
    `ts` BIGINT COMMENT '时间'
) COMMENT '动作日志表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_action_log'
TBLPROPERTIES('parquet.compression'='lzo');

-- 04.曝光日志表（UDTF函数处理）
DROP TABLE IF EXISTS dwd_display_log;
CREATE EXTERNAL TABLE dwd_display_log(
    `area_code` STRING COMMENT '地区编码',
    `brand` STRING COMMENT '手机品牌',
    `channel` STRING COMMENT '渠道',
    `is_new` STRING COMMENT '是否首次启动',
    `model` STRING COMMENT '手机型号',
    `mid_id` STRING COMMENT '设备id',
    `os` STRING COMMENT '操作系统',
    `user_id` STRING COMMENT '会员id',
    `version_code` STRING COMMENT 'app版本号',
    `during_time` BIGINT COMMENT 'app版本号',
    `page_item` STRING COMMENT '目标id ',
    `page_item_type` STRING COMMENT '目标类型',
    `last_page_id` STRING COMMENT '上页类型',
    `page_id` STRING COMMENT '页面ID ',
    `source_type` STRING COMMENT '来源类型',
    `ts` BIGINT COMMENT 'app版本号',
    `display_type` STRING COMMENT '曝光类型',
    `item` STRING COMMENT '曝光对象id ',
    `item_type` STRING COMMENT 'app版本号',
    `order` BIGINT COMMENT '曝光顺序',
    `pos_id` BIGINT COMMENT '曝光位置'
) COMMENT '曝光日志表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_display_log'
TBLPROPERTIES('parquet.compression'='lzo');

-- 05.错误日志表
DROP TABLE IF EXISTS dwd_error_log;
CREATE EXTERNAL TABLE dwd_error_log(
    `area_code` STRING COMMENT '地区编码',
    `brand` STRING COMMENT '手机品牌',
    `channel` STRING COMMENT '渠道',
    `is_new` STRING COMMENT '是否首次启动',
    `model` STRING COMMENT '手机型号',
    `mid_id` STRING COMMENT '设备id',
    `os` STRING COMMENT '操作系统',
    `user_id` STRING COMMENT '会员id',
    `version_code` STRING COMMENT 'app版本号',
    `page_item` STRING COMMENT '目标id ',
    `page_item_type` STRING COMMENT '目标类型',
    `last_page_id` STRING COMMENT '上页类型',
    `page_id` STRING COMMENT '页面ID ',
    `source_type` STRING COMMENT '来源类型',
    `entry` STRING COMMENT ' icon手机图标  notice 通知 install 安装后启动',
    `loading_time` STRING COMMENT '启动加载时间',
    `open_ad_id` STRING COMMENT '广告页ID ',
    `open_ad_ms` STRING COMMENT '广告总共播放时间',
    `open_ad_skip_ms` STRING COMMENT '用户跳过广告时点',
    `actions` STRING COMMENT '动作',
    `displays` STRING COMMENT '曝光',
    `ts` STRING COMMENT '时间',
    `error_code` STRING COMMENT '错误码',
    `msg` STRING COMMENT '错误信息'
) COMMENT '错误日志表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_error_log'
TBLPROPERTIES('parquet.compression'='lzo');
```

###### DWD层用户行为数据装载脚本ods_to_dwd_log.sh

> /home/ahua/bin/ods_to_dwd_log.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

# 01.启动日志表
dwd_start_log="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_start_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.start.entry'),
    get_json_object(line,'$.start.loading_time'),
    get_json_object(line,'$.start.open_ad_id'),
    get_json_object(line,'$.start.open_ad_ms'),
    get_json_object(line,'$.start.open_ad_skip_ms'),
    get_json_object(line,'$.ts')
from ${APP}.ods_log
where dt='$do_date'
and get_json_object(line,'$.start') is not null;"

# 02.页面日志表
dwd_page_log="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_page_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.source_type'),
    get_json_object(line,'$.ts')
from ${APP}.ods_log
where dt='$do_date'
and get_json_object(line,'$.page') is not null;"

# 03.动作日志表（UDTF函数处理）
dwd_action_log="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_action_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.source_type'),
    get_json_object(action,'$.action_id'),
    get_json_object(action,'$.item'),
    get_json_object(action,'$.item_type'),
    get_json_object(action,'$.ts')
from ${APP}.ods_log lateral view ${APP}.explode_json_array(get_json_object(line,'$.actions')) tmp as action
where dt='$do_date'
and get_json_object(line,'$.actions') is not null;"

# 04.曝光日志表（UDTF函数处理）
dwd_display_log="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_display_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.source_type'),
    get_json_object(line,'$.ts'),
    get_json_object(display,'$.display_type'),
    get_json_object(display,'$.item'),
    get_json_object(display,'$.item_type'),
    get_json_object(display,'$.order'),
    get_json_object(display,'$.pos_id')
from ${APP}.ods_log lateral view ${APP}.explode_json_array(get_json_object(line,'$.displays')) tmp as display
where dt='$do_date'
and get_json_object(line,'$.displays') is not null;"

# 05.错误日志表
dwd_error_log="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_error_log partition(dt='$do_date')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.source_type'),
    get_json_object(line,'$.start.entry'),
    get_json_object(line,'$.start.loading_time'),
    get_json_object(line,'$.start.open_ad_id'),
    get_json_object(line,'$.start.open_ad_ms'),
    get_json_object(line,'$.start.open_ad_skip_ms'),
    get_json_object(line,'$.actions'),
    get_json_object(line,'$.displays'),
    get_json_object(line,'$.ts'),
    get_json_object(line,'$.err.error_code'),
    get_json_object(line,'$.err.msg')
from ${APP}.ods_log
where dt='$do_date'
and get_json_object(line,'$.err') is not null;"

case $1 in
    dwd_start_log )
        hive -e "$dwd_start_log"
    ;;
    dwd_page_log )
        hive -e "$dwd_page_log"
    ;;
    dwd_action_log )
        hive -e "$dwd_action_log"
    ;;
    dwd_display_log )
        hive -e "$dwd_display_log"
    ;;
    dwd_error_log )
        hive -e "$dwd_error_log"
    ;;
    all )
        hive -e "$dwd_start_log$dwd_page_log$dwd_action_log$dwd_display_log$dwd_error_log"
    ;;
esac
```

###### DWD层业务数据建表语句（9个事实表）

> 评价事实表（事务型事实表）dwd_comment_info
>
> 订单明细事实表（事务型事实表）dwd_order_detail
>
> 退单事实表（事务型事实表）dwd_order_refund_info
>
> 加购事实表（周期型快照事实表，每日快照）dwd_cart_info
>
> 收藏事实表（周期型快照事实表，每日快照）dwd_favor_info
>
> 优惠券领用事实表（累积型快照事实表）dwd_coupon_use
>
> 支付事实表（累积型快照事实表）dwd_payment_info
>
> 退款事实表（累积型快照事实表）dwd_refund_payment
>
> 订单事实表（累积型快照事实表）dwd_order_info

```sql
-- DWD层业务数据建表语句（9个） --
-- 01.评价事实表（事务型事实表）
DROP TABLE IF EXISTS dwd_comment_info;
CREATE EXTERNAL TABLE dwd_comment_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `sku_id` STRING COMMENT '商品sku',
    `spu_id` STRING COMMENT '商品spu',
    `order_id` STRING COMMENT '订单ID',
    `appraise` STRING COMMENT '评价(好评、中评、差评、默认评价)',
    `create_time` STRING COMMENT '评价时间'
) COMMENT '评价事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_comment_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 02.订单明细事实表（事务型事实表）
DROP TABLE IF EXISTS dwd_order_detail;
CREATE EXTERNAL TABLE dwd_order_detail (
    `id` STRING COMMENT '订单编号',
    `order_id` STRING COMMENT '订单号',
    `user_id` STRING COMMENT '用户id',
    `sku_id` STRING COMMENT 'sku商品id',
    `province_id` STRING COMMENT '省份ID',
    `activity_id` STRING COMMENT '活动ID',
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `coupon_id` STRING COMMENT '优惠券ID',
    `create_time` STRING COMMENT '创建时间',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号',
    `sku_num` BIGINT COMMENT '商品数量',
    `original_amount` DECIMAL(16,2) COMMENT '原始价格',
    `split_activity_amount` DECIMAL(16,2) COMMENT '活动优惠分摊',
    `split_coupon_amount` DECIMAL(16,2) COMMENT '优惠券优惠分摊',
    `split_final_amount` DECIMAL(16,2) COMMENT '最终价格分摊'
) COMMENT '订单明细事实表表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_order_detail/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 03.退单事实表（事务型事实表）
-- 本应是累计快照事实表，这里做了简化
DROP TABLE IF EXISTS dwd_order_refund_info;
CREATE EXTERNAL TABLE dwd_order_refund_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `order_id` STRING COMMENT '订单ID',
    `sku_id` STRING COMMENT '商品ID',
    `province_id` STRING COMMENT '地区ID',
    `refund_type` STRING COMMENT '退单类型',
    `refund_num` BIGINT COMMENT '退单件数',
    `refund_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_reason_type` STRING COMMENT '退单原因类型',
    `create_time` STRING COMMENT '退单时间'
) COMMENT '退单事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_order_refund_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 04.加购事实表（周期型快照事实表，每日快照）
DROP TABLE IF EXISTS dwd_cart_info;
CREATE EXTERNAL TABLE dwd_cart_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `sku_id` STRING COMMENT '商品ID',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号',
    `cart_price` DECIMAL(16,2) COMMENT '加入购物车时的价格',
    `is_ordered` STRING COMMENT '是否已下单',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '修改时间',
    `order_time` STRING COMMENT '下单时间',
    `sku_num` BIGINT COMMENT '加购数量'
) COMMENT '加购事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_cart_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 05.收藏事实表（周期型快照事实表，每日快照）
DROP TABLE IF EXISTS dwd_favor_info;
CREATE EXTERNAL TABLE dwd_favor_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING  COMMENT '用户id',
    `sku_id` STRING  COMMENT 'skuid',
    `spu_id` STRING  COMMENT 'spuid',
    `is_cancel` STRING  COMMENT '是否取消',
    `create_time` STRING  COMMENT '收藏时间',
    `cancel_time` STRING  COMMENT '取消时间'
) COMMENT '收藏事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_favor_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 06.优惠券领用事实表（累积型快照事实表）
DROP TABLE IF EXISTS dwd_coupon_use;
CREATE EXTERNAL TABLE dwd_coupon_use(
    `id` STRING COMMENT '编号',
    `coupon_id` STRING  COMMENT '优惠券ID',
    `user_id` STRING  COMMENT 'userid',
    `order_id` STRING  COMMENT '订单id',
    `coupon_status` STRING  COMMENT '优惠券状态',
    `get_time` STRING  COMMENT '领取时间',
    `using_time` STRING  COMMENT '使用时间(下单)',
    `used_time` STRING  COMMENT '使用时间(支付)',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券领用事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_coupon_use/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 07.支付事实表（累积型快照事实表）
DROP TABLE IF EXISTS dwd_payment_info;
CREATE EXTERNAL TABLE dwd_payment_info (
    `id` STRING COMMENT '编号',
    `order_id` STRING COMMENT '订单编号',
    `user_id` STRING COMMENT '用户编号',
    `province_id` STRING COMMENT '地区ID',
    `trade_no` STRING COMMENT '交易编号',
    `out_trade_no` STRING COMMENT '对外交易编号',
    `payment_type` STRING COMMENT '支付类型',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `payment_status` STRING COMMENT '支付状态',
    `create_time` STRING COMMENT '创建时间',--调用第三方支付接口的时间
    `callback_time` STRING COMMENT '完成时间'--支付完成时间，即支付成功回调时间
) COMMENT '支付事实表表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_payment_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 08.退款事实表（累积型快照事实表）
DROP TABLE IF EXISTS dwd_refund_payment;
CREATE EXTERNAL TABLE dwd_refund_payment (
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `order_id` STRING COMMENT '订单编号',
    `sku_id` STRING COMMENT 'SKU编号',
    `province_id` STRING COMMENT '地区ID',
    `trade_no` STRING COMMENT '交易编号',
    `out_trade_no` STRING COMMENT '对外交易编号',
    `payment_type` STRING COMMENT '支付类型',
    `refund_amount` DECIMAL(16,2) COMMENT '退款金额',
    `refund_status` STRING COMMENT '退款状态',
    `create_time` STRING COMMENT '创建时间',--调用第三方支付接口的时间
    `callback_time` STRING COMMENT '回调时间'--支付接口回调时间，即支付成功时间
) COMMENT '退款事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_refund_payment/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 09.订单事实表（累积型快照事实表）
DROP TABLE IF EXISTS dwd_order_info;
CREATE EXTERNAL TABLE dwd_order_info(
    `id` STRING COMMENT '编号',
    `order_status` STRING COMMENT '订单状态',
    `user_id` STRING COMMENT '用户ID',
    `province_id` STRING COMMENT '地区ID',
    `payment_way` STRING COMMENT '支付方式',
    `delivery_address` STRING COMMENT '邮寄地址',
    `out_trade_no` STRING COMMENT '对外交易编号',
    `tracking_no` STRING COMMENT '物流单号',
    `create_time` STRING COMMENT '创建时间(未支付状态)',
    `payment_time` STRING COMMENT '支付时间(已支付状态)',
    `cancel_time` STRING COMMENT '取消时间(已取消状态)',
    `finish_time` STRING COMMENT '完成时间(已完成状态)',
    `refund_time` STRING COMMENT '退款时间(退款中状态)',
    `refund_finish_time` STRING COMMENT '退款完成时间(退款完成状态)',
    `expire_time` STRING COMMENT '过期时间',
    `feight_fee` DECIMAL(16,2) COMMENT '运费',
    `feight_fee_reduce` DECIMAL(16,2) COMMENT '运费减免',
    `activity_reduce_amount` DECIMAL(16,2) COMMENT '活动减免',
    `coupon_reduce_amount` DECIMAL(16,2) COMMENT '优惠券减免',
    `original_amount` DECIMAL(16,2) COMMENT '订单原始价格',
    `final_amount` DECIMAL(16,2) COMMENT '订单最终价格'
) COMMENT '订单事实表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwd/dwd_order_info/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

###### DWD层业务数据首日装载脚本ods_to_dwd_db_init.sh

> /home/ahua/bin/ods_to_dwd_db_init.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

# 01.评价事实表（事务型事实表）
dwd_comment_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_comment_info partition(dt)
select
    id,
    user_id,
    sku_id,
    spu_id,
    order_id,
    appraise,
    create_time,
    date_format(create_time,'yyyy-MM-dd')
from ${APP}.ods_comment_info
where dt='$do_date';
"

# 02.订单明细事实表（事务型事实表）
dwd_order_detail="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_detail partition(dt)
select
    od.id,
    od.order_id,
    oi.user_id,
    od.sku_id,
    oi.province_id,
    oda.activity_id,
    oda.activity_rule_id,
    odc.coupon_id,
    od.create_time,
    od.source_type,
    od.source_id,
    od.sku_num,
    od.order_price*od.sku_num,
    od.split_activity_amount,
    od.split_coupon_amount,
    od.split_final_amount,
    date_format(create_time,'yyyy-MM-dd')
from
(
    select
        *
    from ${APP}.ods_order_detail
    where dt='$do_date'
)od
left join
(
    select
        id,
        user_id,
        province_id
    from ${APP}.ods_order_info
    where dt='$do_date'
)oi
on od.order_id=oi.id
left join
(
    select
        order_detail_id,
        activity_id,
        activity_rule_id
    from ${APP}.ods_order_detail_activity
    where dt='$do_date'
)oda
on od.id=oda.order_detail_id
left join
(
    select
        order_detail_id,
        coupon_id
    from ${APP}.ods_order_detail_coupon
    where dt='$do_date'
)odc
on od.id=odc.order_detail_id;
"

# 03.退单事实表（事务型事实表）
# 本应是累计快照事实表，这里做了简化
dwd_order_refund_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_refund_info partition(dt)
select
    ri.id,
    ri.user_id,
    ri.order_id,
    ri.sku_id,
    oi.province_id,
    ri.refund_type,
    ri.refund_num,
    ri.refund_amount,
    ri.refund_reason_type,
    ri.create_time,
    date_format(ri.create_time,'yyyy-MM-dd')
from
(
    select * from ${APP}.ods_order_refund_info where dt='$do_date'
)ri
left join
(
    select id,province_id from ${APP}.ods_order_info where dt='$do_date'
)oi
on ri.order_id=oi.id;
"

# 04.加购事实表（周期型快照事实表，每日快照）
dwd_cart_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_cart_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    source_type,
    source_id,
    cart_price,
    is_ordered,
    create_time,
    operate_time,
    order_time,
    sku_num
from ${APP}.ods_cart_info
where dt='$do_date';
"

# 05.收藏事实表（周期型快照事实表，每日快照）
dwd_favor_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_favor_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    spu_id,
    is_cancel,
    create_time,
    cancel_time
from ${APP}.ods_favor_info
where dt='$do_date';
"

# 06.优惠券领用事实表（累积型快照事实表）
dwd_coupon_use="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_coupon_use partition(dt)
select
    id,
    coupon_id,
    user_id,
    order_id,
    coupon_status,
    get_time,
    using_time,
    used_time,
    expire_time,
    coalesce(date_format(used_time,'yyyy-MM-dd'),date_format(expire_time,'yyyy-MM-dd'),'9999-99-99')
from ${APP}.ods_coupon_use
where dt='$do_date';
"

# 07.支付事实表（累积型快照事实表）
dwd_payment_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_payment_info partition(dt)
select
    pi.id,
    pi.order_id,
    pi.user_id,
    oi.province_id,
    pi.trade_no,
    pi.out_trade_no,
    pi.payment_type,
    pi.payment_amount,
    pi.payment_status,
    pi.create_time,
    pi.callback_time,
    nvl(date_format(pi.callback_time,'yyyy-MM-dd'),'9999-99-99')
from
(
    select * from ${APP}.ods_payment_info where dt='$do_date'
)pi
left join
(
    select id,province_id from ${APP}.ods_order_info where dt='$do_date'
)oi
on pi.order_id=oi.id;
"

# 08.退款事实表（累积型快照事实表）
dwd_refund_payment="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_refund_payment partition(dt)
select
    rp.id,
    user_id,
    order_id,
    sku_id,
    province_id,
    trade_no,
    out_trade_no,
    payment_type,
    refund_amount,
    refund_status,
    create_time,
    callback_time,
    nvl(date_format(callback_time,'yyyy-MM-dd'),'9999-99-99')
from
(
    select
        id,
        out_trade_no,
        order_id,
        sku_id,
        payment_type,
        trade_no,
        refund_amount,
        refund_status,
        create_time,
        callback_time
    from ${APP}.ods_refund_payment
    where dt='$do_date'
)rp
left join
(
    select
        id,
        user_id,
        province_id
    from ${APP}.ods_order_info
    where dt='$do_date'
)oi
on rp.order_id=oi.id;
"

# 09.订单事实表（累积型快照事实表）
dwd_order_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_info partition(dt)
select
    oi.id,
    oi.order_status,
    oi.user_id,
    oi.province_id,
    oi.payment_way,
    oi.delivery_address,
    oi.out_trade_no,
    oi.tracking_no,
    oi.create_time,
    times.ts['1002'] payment_time,
    times.ts['1003'] cancel_time,
    times.ts['1004'] finish_time,
    times.ts['1005'] refund_time,
    times.ts['1006'] refund_finish_time,
    oi.expire_time,
    feight_fee,
    feight_fee_reduce,
    activity_reduce_amount,
    coupon_reduce_amount,
    original_amount,
    final_amount,
    case
        when times.ts['1003'] is not null then date_format(times.ts['1003'],'yyyy-MM-dd')
        when times.ts['1004'] is not null and date_add(date_format(times.ts['1004'],'yyyy-MM-dd'),7)<='$do_date' and times.ts['1005'] is null then date_add(date_format(times.ts['1004'],'yyyy-MM-dd'),7)
        when times.ts['1006'] is not null then date_format(times.ts['1006'],'yyyy-MM-dd')
        when oi.expire_time is not null then date_format(oi.expire_time,'yyyy-MM-dd')
        else '9999-99-99'
    end
from
(
    select
        *
    from ${APP}.ods_order_info
    where dt='$do_date'
)oi
left join
(
    select
        order_id,
        str_to_map(concat_ws(',',collect_set(concat(order_status,'=',operate_time))),',','=') ts
    from ${APP}.ods_order_status_log
    where dt='$do_date'
    group by order_id
)times
on oi.id=times.order_id;
"

case $1 in
    dwd_comment_info )
        hive -e "$dwd_comment_info"
    ;;
    dwd_order_detail )
        hive -e "$dwd_order_detail"
    ;;
    dwd_order_refund_info )
        hive -e "$dwd_order_refund_info"
    ;;
    dwd_cart_info )
        hive -e "$dwd_cart_info"
    ;;
    dwd_favor_info )
        hive -e "$dwd_favor_info"
    ;;
    dwd_coupon_use )
        hive -e "$dwd_coupon_use"
    ;;
    dwd_payment_info )
        hive -e "$dwd_payment_info"
    ;;
    dwd_refund_payment )
        hive -e "$dwd_refund_payment"
    ;;
    dwd_order_info )
        hive -e "$dwd_order_info"
    ;;
    all )
        hive -e "$dwd_comment_info$dwd_order_detail$dwd_order_refund_info$dwd_cart_info$dwd_favor_info$dwd_coupon_use$dwd_payment_info$dwd_refund_payment$dwd_order_info"
    ;;
esac
```

###### DWD层业务数据每日装载脚本ods_to_dwd_db_daily.sh

> /home/ahua/bin/ods_to_dwd_db_daily.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

# 假设某累积型快照事实表，某天所有的业务记录全部完成，则会导致9999-99-99分区的数据未被覆盖，从而导致数据重复，该函数根据9999-99-99分区的数据的末次修改时间判断其是否被覆盖了，如果未被覆盖，就手动清理
clear_data(){
    current_date=`date +%F`
    current_date_timestamp=`date -d "$current_date" +%s`

    last_modified_date=`hadoop fs -ls /warehouse/gmall/dwd/$1 | grep '9999-99-99' | awk '{print $6}'`
    last_modified_date_timestamp=`date -d "$last_modified_date" +%s`

    if [[ $last_modified_date_timestamp -lt $current_date_timestamp ]]; then
        echo "clear table $1 partition(dt=9999-99-99)"
        hadoop fs -rm -r -f /warehouse/gmall/dwd/$1/dt=9999-99-99/*
    fi
}

# 01.评价事实表（事务型事实表）
dwd_comment_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_comment_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    spu_id,
    order_id,
    appraise,
    create_time
from ${APP}.ods_comment_info where dt='$do_date';
"

# 02.订单明细事实表（事务型事实表）
dwd_order_detail="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_detail partition(dt='$do_date')
select
    od.id,
    od.order_id,
    oi.user_id,
    od.sku_id,
    oi.province_id,
    oda.activity_id,
    oda.activity_rule_id,
    odc.coupon_id,
    od.create_time,
    od.source_type,
    od.source_id,
    od.sku_num,
    od.order_price*od.sku_num,
    od.split_activity_amount,
    od.split_coupon_amount,
    od.split_final_amount
from
(
    select
        *
    from ${APP}.ods_order_detail
    where dt='$do_date'
)od
left join
(
    select
        id,
        user_id,
        province_id
    from ${APP}.ods_order_info
    where dt='$do_date'
)oi
on od.order_id=oi.id
left join
(
    select
        order_detail_id,
        activity_id,
        activity_rule_id
    from ${APP}.ods_order_detail_activity
    where dt='$do_date'
)oda
on od.id=oda.order_detail_id
left join
(
    select
        order_detail_id,
        coupon_id
    from ${APP}.ods_order_detail_coupon
    where dt='$do_date'
)odc
on od.id=odc.order_detail_id;
"

# 03.退单事实表（事务型事实表）
# 本应是累计快照事实表，这里做了简化
dwd_order_refund_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_refund_info partition(dt='$do_date')
select
    ri.id,
    ri.user_id,
    ri.order_id,
    ri.sku_id,
    oi.province_id,
    ri.refund_type,
    ri.refund_num,
    ri.refund_amount,
    ri.refund_reason_type,
    ri.create_time
from
(
    select * from ${APP}.ods_order_refund_info where dt='$do_date'
)ri
left join
(
    select id,province_id from ${APP}.ods_order_info where dt='$do_date'
)oi
on ri.order_id=oi.id;
"

# 04.加购事实表（周期型快照事实表，每日快照）
dwd_cart_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_cart_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    source_type,
    source_id,
    cart_price,
    is_ordered,
    create_time,
    operate_time,
    order_time,
    sku_num
from ${APP}.ods_cart_info
where dt='$do_date';
"

# 05.收藏事实表（周期型快照事实表，每日快照）
dwd_favor_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_favor_info partition(dt='$do_date')
select
    id,
    user_id,
    sku_id,
    spu_id,
    is_cancel,
    create_time,
    cancel_time
from ${APP}.ods_favor_info
where dt='$do_date';
"

# 06.优惠券领用事实表（累积型快照事实表）
dwd_coupon_use="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_coupon_use partition(dt)
select
    nvl(new.id,old.id),
    nvl(new.coupon_id,old.coupon_id),
    nvl(new.user_id,old.user_id),
    nvl(new.order_id,old.order_id),
    nvl(new.coupon_status,old.coupon_status),
    nvl(new.get_time,old.get_time),
    nvl(new.using_time,old.using_time),
    nvl(new.used_time,old.used_time),
    nvl(new.expire_time,old.expire_time),
    coalesce(date_format(nvl(new.used_time,old.used_time),'yyyy-MM-dd'),date_format(nvl(new.expire_time,old.expire_time),'yyyy-MM-dd'),'9999-99-99')
from
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time,
        expire_time
    from ${APP}.dwd_coupon_use
    where dt='9999-99-99'
)old
full outer join
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time,
        expire_time
    from ${APP}.ods_coupon_use
    where dt='$do_date'
)new
on old.id=new.id;
"

# 07.支付事实表（累积型快照事实表）
dwd_payment_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_payment_info partition(dt)
select
    nvl(new.id,old.id),
    nvl(new.order_id,old.order_id),
    nvl(new.user_id,old.user_id),
    nvl(new.province_id,old.province_id),
    nvl(new.trade_no,old.trade_no),
    nvl(new.out_trade_no,old.out_trade_no),
    nvl(new.payment_type,old.payment_type),
    nvl(new.payment_amount,old.payment_amount),
    nvl(new.payment_status,old.payment_status),
    nvl(new.create_time,old.create_time),
    nvl(new.callback_time,old.callback_time),
    nvl(date_format(nvl(new.callback_time,old.callback_time),'yyyy-MM-dd'),'9999-99-99')
from
(
    select id,
       order_id,
       user_id,
       province_id,
       trade_no,
       out_trade_no,
       payment_type,
       payment_amount,
       payment_status,
       create_time,
       callback_time
    from ${APP}.dwd_payment_info
    where dt = '9999-99-99'
)old
full outer join
(
    select
        pi.id,
        pi.out_trade_no,
        pi.order_id,
        pi.user_id,
        oi.province_id,
        pi.payment_type,
        pi.trade_no,
        pi.payment_amount,
        pi.payment_status,
        pi.create_time,
        pi.callback_time
    from
    (
        select * from ${APP}.ods_payment_info where dt='$do_date'
    )pi
    left join
    (
        select id,province_id from ${APP}.ods_order_info where dt='$do_date'
    )oi
    on pi.order_id=oi.id
)new
on old.id=new.id;
"

# 08.退款事实表（累积型快照事实表）
dwd_refund_payment="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_refund_payment partition(dt)
select
    nvl(new.id,old.id),
    nvl(new.user_id,old.user_id),
    nvl(new.order_id,old.order_id),
    nvl(new.sku_id,old.sku_id),
    nvl(new.province_id,old.province_id),
    nvl(new.trade_no,old.trade_no),
    nvl(new.out_trade_no,old.out_trade_no),
    nvl(new.payment_type,old.payment_type),
    nvl(new.refund_amount,old.refund_amount),
    nvl(new.refund_status,old.refund_status),
    nvl(new.create_time,old.create_time),
    nvl(new.callback_time,old.callback_time),
    nvl(date_format(nvl(new.callback_time,old.callback_time),'yyyy-MM-dd'),'9999-99-99')
from
(
    select
        id,
        user_id,
        order_id,
        sku_id,
        province_id,
        trade_no,
        out_trade_no,
        payment_type,
        refund_amount,
        refund_status,
        create_time,
        callback_time
    from ${APP}.dwd_refund_payment
    where dt='9999-99-99'
)old
full outer join
(
    select
        rp.id,
        user_id,
        order_id,
        sku_id,
        province_id,
        trade_no,
        out_trade_no,
        payment_type,
        refund_amount,
        refund_status,
        create_time,
        callback_time
    from
    (
        select
            id,
            out_trade_no,
            order_id,
            sku_id,
            payment_type,
            trade_no,
            refund_amount,
            refund_status,
            create_time,
            callback_time
        from ${APP}.ods_refund_payment
        where dt='$do_date'
    )rp
    left join
    (
        select
            id,
            user_id,
            province_id
        from ${APP}.ods_order_info
        where dt='$do_date'
    )oi
    on rp.order_id=oi.id
)new
on old.id=new.id;
"

# 09.订单事实表（累积型快照事实表）
dwd_order_info="
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dwd_order_info partition(dt)
select
    nvl(new.id,old.id),
    nvl(new.order_status,old.order_status),
    nvl(new.user_id,old.user_id),
    nvl(new.province_id,old.province_id),
    nvl(new.payment_way,old.payment_way),
    nvl(new.delivery_address,old.delivery_address),
    nvl(new.out_trade_no,old.out_trade_no),
    nvl(new.tracking_no,old.tracking_no),
    nvl(new.create_time,old.create_time),
    nvl(new.payment_time,old.payment_time),
    nvl(new.cancel_time,old.cancel_time),
    nvl(new.finish_time,old.finish_time),
    nvl(new.refund_time,old.refund_time),
    nvl(new.refund_finish_time,old.refund_finish_time),
    nvl(new.expire_time,old.expire_time),
    nvl(new.feight_fee,old.feight_fee),
    nvl(new.feight_fee_reduce,old.feight_fee_reduce),
    nvl(new.activity_reduce_amount,old.activity_reduce_amount),
    nvl(new.coupon_reduce_amount,old.coupon_reduce_amount),
    nvl(new.original_amount,old.original_amount),
    nvl(new.final_amount,old.final_amount),
    case
        when new.cancel_time is not null then date_format(new.cancel_time,'yyyy-MM-dd')
        when new.finish_time is not null and date_add(date_format(new.finish_time,'yyyy-MM-dd'),7)='$do_date' and new.refund_time is null then '$do_date'
        when new.refund_finish_time is not null then date_format(new.refund_finish_time,'yyyy-MM-dd')
        when new.expire_time is not null then date_format(new.expire_time,'yyyy-MM-dd')
        else '9999-99-99'
    end
from
(
    select
        id,
        order_status,
        user_id,
        province_id,
        payment_way,
        delivery_address,
        out_trade_no,
        tracking_no,
        create_time,
        payment_time,
        cancel_time,
        finish_time,
        refund_time,
        refund_finish_time,
        expire_time,
        feight_fee,
        feight_fee_reduce,
        activity_reduce_amount,
        coupon_reduce_amount,
        original_amount,
        final_amount
    from ${APP}.dwd_order_info
    where dt='9999-99-99'
)old
full outer join
(
    select
        oi.id,
        oi.order_status,
        oi.user_id,
        oi.province_id,
        oi.payment_way,
        oi.delivery_address,
        oi.out_trade_no,
        oi.tracking_no,
        oi.create_time,
        times.ts['1002'] payment_time,
        times.ts['1003'] cancel_time,
        times.ts['1004'] finish_time,
        times.ts['1005'] refund_time,
        times.ts['1006'] refund_finish_time,
        oi.expire_time,
        feight_fee,
        feight_fee_reduce,
        activity_reduce_amount,
        coupon_reduce_amount,
        original_amount,
        final_amount
    from
    (
        select
            *
        from ${APP}.ods_order_info
        where dt='$do_date'
    )oi
    left join
    (
        select
            order_id,
            str_to_map(concat_ws(',',collect_set(concat(order_status,'=',operate_time))),',','=') ts
        from ${APP}.ods_order_status_log
        where dt='$do_date'
        group by order_id
    )times
    on oi.id=times.order_id
)new
on old.id=new.id;
"

case $1 in
    dwd_comment_info )
        hive -e "$dwd_comment_info"
    ;;
    dwd_order_detail )
        hive -e "$dwd_order_detail"
    ;;
    dwd_order_refund_info )
        hive -e "$dwd_order_refund_info"
    ;;
    dwd_cart_info )
        hive -e "$dwd_cart_info"
    ;;
    dwd_favor_info )
        hive -e "$dwd_favor_info"
    ;;
    dwd_coupon_use )
        hive -e "$dwd_coupon_use"
        clear_data dwd_coupon_use
    ;;
    dwd_payment_info )
        hive -e "$dwd_payment_info"
        clear_data dwd_payment_info
    ;;
    dwd_refund_payment )
        hive -e "$dwd_refund_payment"
        clear_data dwd_refund_payment
    ;;
    dwd_order_info )
        hive -e "$dwd_order_info"
        clear_data dwd_order_info
    ;;
    all )
        hive -e "$dwd_comment_info$dwd_order_detail$dwd_order_refund_info$dwd_cart_info$dwd_favor_info$dwd_coupon_use$dwd_payment_info$dwd_refund_payment$dwd_order_info"
        clear_data dwd_order_info
        clear_data dwd_payment_info
        clear_data dwd_coupon_use
        clear_data dwd_refund_payment
    ;;
esac
```

##### ——数仓搭建-DWS层——

###### 建表语句（6个）

```sql
-- DWS层建表语句（6个） --
-- 01.访客主题
DROP TABLE IF EXISTS dws_visitor_action_daycount;
CREATE EXTERNAL TABLE dws_visitor_action_daycount
(
    `mid_id` STRING COMMENT '设备id',
    `brand` STRING COMMENT '设备品牌',
    `model` STRING COMMENT '设备型号',
    `is_new` STRING COMMENT '是否首次访问',
    `channel` ARRAY<STRING> COMMENT '渠道',
    `os` ARRAY<STRING> COMMENT '操作系统',
    `area_code` ARRAY<STRING> COMMENT '地区ID',
    `version_code` ARRAY<STRING> COMMENT '应用版本',
    `visit_count` BIGINT COMMENT '访问次数',
    `page_stats` ARRAY<STRUCT<page_id:STRING,page_count:BIGINT,during_time:BIGINT>> COMMENT '页面访问统计'
) COMMENT '每日设备行为表'
PARTITIONED BY(`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_visitor_action_daycount'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 02.用户主题
DROP TABLE IF EXISTS dws_user_action_daycount;
CREATE EXTERNAL TABLE dws_user_action_daycount
(
    `user_id` STRING COMMENT '用户id',
    `login_count` BIGINT COMMENT '登录次数',
    `cart_count` BIGINT COMMENT '加入购物车次数',
    `favor_count` BIGINT COMMENT '收藏次数',
    `order_count` BIGINT COMMENT '下单次数',
    `order_activity_count` BIGINT COMMENT '订单参与活动次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '订单减免金额(活动)',
    `order_coupon_count` BIGINT COMMENT '订单用券次数',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '订单减免金额(优惠券)',
    `order_original_amount` DECIMAL(16,2)  COMMENT '订单单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '订单总金额',
    `payment_count` BIGINT COMMENT '支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `refund_order_count` BIGINT COMMENT '退单次数',
    `refund_order_num` BIGINT COMMENT '退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_payment_count` BIGINT COMMENT '退款次数',
    `refund_payment_num` BIGINT COMMENT '退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '退款金额',
    `coupon_get_count` BIGINT COMMENT '优惠券领取次数',
    `coupon_using_count` BIGINT COMMENT '优惠券使用(下单)次数',
    `coupon_used_count` BIGINT COMMENT '优惠券使用(支付)次数',
    `appraise_good_count` BIGINT COMMENT '好评数',
    `appraise_mid_count` BIGINT COMMENT '中评数',
    `appraise_bad_count` BIGINT COMMENT '差评数',
    `appraise_default_count` BIGINT COMMENT '默认评价数',
    `order_detail_stats` array<struct<sku_id:string,sku_num:bigint,order_count:bigint,activity_reduce_amount:decimal(16,2),coupon_reduce_amount:decimal(16,2),original_amount:decimal(16,2),final_amount:decimal(16,2)>> COMMENT '下单明细统计'
) COMMENT '每日用户行为'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_user_action_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 03.商品主题
DROP TABLE IF EXISTS dws_sku_action_daycount;
CREATE EXTERNAL TABLE dws_sku_action_daycount
(
    `sku_id` STRING COMMENT 'sku_id',
    `order_count` BIGINT COMMENT '被下单次数',
    `order_num` BIGINT COMMENT '被下单件数',
    `order_activity_count` BIGINT COMMENT '参与活动被下单次数',
    `order_coupon_count` BIGINT COMMENT '使用优惠券被下单次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '优惠金额(活动)',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '优惠金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '被下单原价金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '被下单最终金额',
    `payment_count` BIGINT COMMENT '被支付次数',
    `payment_num` BIGINT COMMENT '被支付件数',
    `payment_amount` DECIMAL(16,2) COMMENT '被支付金额',
    `refund_order_count` BIGINT  COMMENT '被退单次数',
    `refund_order_num` BIGINT COMMENT '被退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '被退单金额',
    `refund_payment_count` BIGINT  COMMENT '被退款次数',
    `refund_payment_num` BIGINT COMMENT '被退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '被退款金额',
    `cart_count` BIGINT COMMENT '被加入购物车次数',
    `favor_count` BIGINT COMMENT '被收藏次数',
    `appraise_good_count` BIGINT COMMENT '好评数',
    `appraise_mid_count` BIGINT COMMENT '中评数',
    `appraise_bad_count` BIGINT COMMENT '差评数',
    `appraise_default_count` BIGINT COMMENT '默认评价数'
) COMMENT '每日商品行为'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_sku_action_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 04.优惠券主题
DROP TABLE IF EXISTS dws_coupon_info_daycount;
CREATE EXTERNAL TABLE dws_coupon_info_daycount(
    `coupon_id` STRING COMMENT '优惠券ID',
    `get_count` BIGINT COMMENT '被领取次数',
    `order_count` BIGINT COMMENT '被使用(下单)次数',
    `order_reduce_amount` DECIMAL(16,2) COMMENT '用券下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '用券订单原价金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '用券下单最终金额',
    `payment_count` BIGINT COMMENT '被使用(支付)次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '用券支付优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '用券支付总金额',
    `expire_count` BIGINT COMMENT '过期次数'
) COMMENT '每日优惠券统计'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_coupon_info_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 05.活动主题
DROP TABLE IF EXISTS dws_activity_info_daycount;
CREATE EXTERNAL TABLE dws_activity_info_daycount(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING COMMENT '活动ID',
    `order_count` BIGINT COMMENT '参与某活动某规则下单次数',    `order_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单减免金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单最终金额',
    `payment_count` BIGINT COMMENT '参与某活动某规则支付次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则支付减免金额',
    `payment_amount` DECIMAL(16,2) COMMENT '参与某活动某规则支付金额'
) COMMENT '每日活动统计'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_activity_info_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 06.地区主题
DROP TABLE IF EXISTS dws_area_stats_daycount;
CREATE EXTERNAL TABLE dws_area_stats_daycount(
    `province_id` STRING COMMENT '地区编号',
    `visit_count` BIGINT COMMENT '访问次数',
    `login_count` BIGINT COMMENT '登录次数',
    `visitor_count` BIGINT COMMENT '访客人数',
    `user_count` BIGINT COMMENT '用户人数',
    `order_count` BIGINT COMMENT '下单次数',
    `order_original_amount` DECIMAL(16,2) COMMENT '下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '下单最终金额',
    `payment_count` BIGINT COMMENT '支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `refund_order_count` BIGINT COMMENT '退单次数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_payment_count` BIGINT COMMENT '退款次数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '退款金额'
) COMMENT '每日地区统计'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_area_stats_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

###### DWS层首日数据装载脚本dwd_to_dws_init.sh

> /home/ahua/bin/dwd_to_dws_init.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi

# 01.访客主题
#    select
#        mid_id,
#        brand,
#        model,
#        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,--ods_page_log中，同一天内，同一设备的is_new字段，可能全部为1，可能全部为0，也可能部分为0，部分为1(卸载重装),故做该处理
dws_visitor_action_daycount="
insert overwrite table ${APP}.dws_visitor_action_daycount partition(dt='$do_date')
select
    t1.mid_id,
    t1.brand,
    t1.model,
    t1.is_new,
    t1.channel,
    t1.os,
    t1.area_code,
    t1.version_code,
    t1.visit_count,
    t3.page_stats
from
(
    select
        mid_id,
        brand,
        model,
        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,
        collect_set(channel) channel,
        collect_set(os) os,
        collect_set(area_code) area_code,
        collect_set(version_code) version_code,
        sum(if(last_page_id is null,1,0)) visit_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and last_page_id is null
    group by mid_id,model,brand
)t1
join
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count,'during_time',during_time)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count,
            sum(during_time) during_time
        from ${APP}.dwd_page_log
        where dt='$do_date'
        group by mid_id,model,brand,page_id
    )t2
    group by mid_id,model,brand
)t3
on t1.mid_id=t3.mid_id
and t1.brand=t3.brand
and t1.model=t3.model;
"

# 02.用户主题
dws_user_action_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_login as
(
    select
        dt,
        user_id,
        count(*) login_count
    from ${APP}.dwd_page_log
    where user_id is not null
    and last_page_id is null
    group by dt,user_id
),
tmp_cf as
(
    select
        dt,
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where user_id is not null
    and action_id in ('cart_add','favor_add')
    group by dt,user_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),user_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from ${APP}.dwd_refund_payment
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.user_id
),
tmp_coupon as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt) dt,
        coalesce(coupon_get.user_id,coupon_using.user_id,coupon_used.user_id) user_id,
        nvl(coupon_get_count,0) coupon_get_count,
        nvl(coupon_using_count,0) coupon_using_count,
        nvl(coupon_used_count,0) coupon_used_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_get_count
        from ${APP}.dwd_coupon_use
        where get_time is not null
        group by user_id,date_format(get_time,'yyyy-MM-dd')
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_using_count
        from ${APP}.dwd_coupon_use
        where using_time is not null
        group by user_id,date_format(using_time,'yyyy-MM-dd')
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.user_id=coupon_using.user_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_used_count
        from ${APP}.dwd_coupon_use
        where used_time is not null
        group by user_id,date_format(used_time,'yyyy-MM-dd')
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.user_id,coupon_using.user_id)=coupon_used.user_id
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_od as
(
    select
        dt,
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            date_format(create_time,'yyyy-MM-dd') dt,
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from ${APP}.dwd_order_detail
        group by date_format(create_time,'yyyy-MM-dd'),user_id,sku_id
    )t1
    group by dt,user_id
)
insert overwrite table ${APP}.dws_user_action_daycount partition(dt)
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats,
    coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt,tmp_od.dt)
from tmp_login
full outer join tmp_cf
on tmp_login.user_id=tmp_cf.user_id
and tmp_login.dt=tmp_cf.dt
full outer join tmp_order
on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
and coalesce(tmp_login.dt,tmp_cf.dt)=tmp_order.dt
full outer join tmp_pay
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt)=tmp_pay.dt
full outer join tmp_ri
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt)=tmp_ri.dt
full outer join tmp_rp
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt)=tmp_rp.dt
full outer join tmp_comment
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt)=tmp_comment.dt
full outer join tmp_coupon
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt)=tmp_coupon.dt
full outer join tmp_od
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt)=tmp_od.dt;
"

# 03.商品主题
dws_sku_action_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
        where callback_time is not null
    )pi on pi.order_id=od.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),sku_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from ${APP}.dwd_refund_payment
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.sku_id
),
tmp_cf as
(
    select
        dt,
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where action_id in ('cart_add','favor_add')
    group by dt,item
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
)
insert overwrite table ${APP}.dws_sku_action_daycount partition(dt)
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count),
    dt
from
(
    select
        dt,
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by dt,sku_id;
"

# 04.优惠券主题
dws_coupon_info_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_cu as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt,coupon_exprie.dt) dt,
        coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id,coupon_exprie.coupon_id) coupon_id,
        nvl(get_count,0) get_count,
        nvl(order_count,0) order_count,
        nvl(payment_count,0) payment_count,
        nvl(expire_count,0) expire_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) get_count
        from ${APP}.dwd_coupon_use
        group by date_format(get_time,'yyyy-MM-dd'),coupon_id
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) order_count
        from ${APP}.dwd_coupon_use
        where using_time is not null
        group by date_format(using_time,'yyyy-MM-dd'),coupon_id
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.coupon_id=coupon_using.coupon_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) payment_count
        from ${APP}.dwd_coupon_use
        where used_time is not null
        group by date_format(used_time,'yyyy-MM-dd'),coupon_id
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.coupon_id,coupon_using.coupon_id)=coupon_used.coupon_id
    full outer join
    (
        select
            date_format(expire_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) expire_count
        from ${APP}.dwd_coupon_use
        where expire_time is not null
        group by date_format(expire_time,'yyyy-MM-dd'),coupon_id
    )coupon_exprie
    on coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt)=coupon_exprie.dt
    and coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id)=coupon_exprie.coupon_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where coupon_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),coupon_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            order_id,
            coupon_id,
            split_coupon_amount,
            split_final_amount
        from ${APP}.dwd_order_detail
        where coupon_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),coupon_id
)
insert overwrite table ${APP}.dws_coupon_info_daycount partition(dt)
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count),
    dt
from
(
    select
        dt,
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by dt,coupon_id;
"

# 05.活动主题
dws_activity_info_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where activity_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),activity_rule_id,activity_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            activity_rule_id,
            activity_id,
            order_id,
            split_activity_amount,
            split_final_amount
        from ${APP}.dwd_order_detail
        where activity_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),activity_rule_id,activity_id
)
insert overwrite table ${APP}.dws_activity_info_daycount partition(dt)
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    dt
from
(
    select
        dt,
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        dt,
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by dt,activity_rule_id,activity_id;
"

# 06.地区主题
dws_area_stats_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_vu as
(
    select
        dt,
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            dt,
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from ${APP}.dwd_page_log
        where last_page_id is null
        group by dt,area_code
    )tmp
    left join ${APP}.dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
),
tmp_ro as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from ${APP}.dwd_refund_payment
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
)
insert overwrite table ${APP}.dws_area_stats_daycount partition(dt)
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount),
    dt
from
(
    select
        dt,
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by dt,province_id;
"

case $1 in
    "dws_visitor_action_daycount" )
        hive -e "$dws_visitor_action_daycount"
    ;;
    "dws_user_action_daycount" )
        hive -e "$dws_user_action_daycount"
    ;;
    "dws_sku_action_daycount" )
        hive -e "$dws_sku_action_daycount"
    ;;
    "dws_coupon_info_daycount" )
        hive -e "$dws_coupon_info_daycount"
    ;;
    "dws_activity_info_daycount" )
        hive -e "$dws_activity_info_daycount"
    ;;
    "dws_area_stats_daycount" )
        hive -e "$dws_area_stats_daycount"
    ;;
    "all" )
        hive -e "$dws_visitor_action_daycount$dws_user_action_daycount$dws_sku_action_daycount$dws_coupon_info_daycount$dws_activity_info_daycount$dws_area_stats_daycount"
    ;;
esac
```



###### DWS层每日数据装载脚本dwd_to_dws_daily.sh

> /home/ahua/bin/dwd_to_dws_daily.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

# 01.访客主题
#    select
#        mid_id,
#        brand,
#        model,
#        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,--ods_page_log中，同一天内，同一设备的is_new字段，可能全部为1，可能全部为0，也可能部分为0，部分为1(卸载重装),故做该处理
dws_visitor_action_daycount="insert overwrite table ${APP}.dws_visitor_action_daycount partition(dt='$do_date')
select
    t1.mid_id,
    t1.brand,
    t1.model,
    t1.is_new,
    t1.channel,
    t1.os,
    t1.area_code,
    t1.version_code,
    t1.visit_count,
    t3.page_stats
from
(
    select
        mid_id,
        brand,
        model,
        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,
        collect_set(channel) channel,
        collect_set(os) os,
        collect_set(area_code) area_code,
        collect_set(version_code) version_code,
        sum(if(last_page_id is null,1,0)) visit_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and last_page_id is null
    group by mid_id,model,brand
)t1
join
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count,'during_time',during_time)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count,
            sum(during_time) during_time
        from ${APP}.dwd_page_log
        where dt='$do_date'
        group by mid_id,model,brand,page_id
    )t2
    group by mid_id,model,brand
)t3
on t1.mid_id=t3.mid_id
and t1.brand=t3.brand
and t1.model=t3.model;
"

# 02.用户主题
dws_user_action_daycount="
with
tmp_login as
(
    select
        user_id,
        count(*) login_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and user_id is not null
    and last_page_id is null
    group by user_id
),
tmp_cf as
(
    select
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and user_id is not null
    and action_id in ('cart_add','favor_add')
    group by user_id
),
tmp_order as
(
    select
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    where (dt='$do_date'
    or dt='9999-99-99')
    and date_format(create_time,'yyyy-MM-dd')='$do_date'
    group by user_id
),
tmp_pay as
(
    select
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    where dt='$do_date'
    group by user_id
),
tmp_ri as
(
    select
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by user_id
),
tmp_rp as
(
    select
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount
        from ${APP}.dwd_refund_payment
        where dt='$do_date'
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
        where dt>=date_add('$do_date',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by rp.user_id
),
tmp_coupon as
(
    select
        user_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_using_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_used_count
    from ${APP}.dwd_coupon_use
    where (dt='$do_date' or dt='9999-99-99')
    and (date_format(get_time, 'yyyy-MM-dd') = '$do_date'
    or date_format(using_time,'yyyy-MM-dd')='$do_date'
    or date_format(used_time,'yyyy-MM-dd')='$do_date')
    group by user_id
),
tmp_comment as
(
    select
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    where dt='$do_date'
    group by user_id
),
tmp_od as
(
    select
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from ${APP}.dwd_order_detail
        where dt='$do_date'
        group by user_id,sku_id
    )t1
    group by user_id
)
insert overwrite table ${APP}.dws_user_action_daycount partition(dt='$do_date')
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats
from tmp_login
full outer join tmp_cf on tmp_login.user_id=tmp_cf.user_id
full outer join tmp_order on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
full outer join tmp_pay on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
full outer join tmp_ri on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
full outer join tmp_rp on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
full outer join tmp_comment on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
full outer join tmp_coupon on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
full outer join tmp_od on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id;
"

# 03.商品主题
dws_sku_action_daycount="
with
tmp_order as
(
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    group by sku_id
),
tmp_pay as
(
    select
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by sku_id
),
tmp_ri as
(
    select
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by sku_id
),
tmp_rp as
(
    select
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount
        from ${APP}.dwd_refund_payment
        where dt='$do_date'
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
        where dt>=date_add('$do_date',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by rp.sku_id
),
tmp_cf as
(
    select
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and action_id in ('cart_add','favor_add')
    group by item
),
tmp_comment as
(
    select
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    where dt='$do_date'
    group by sku_id
)
insert overwrite table ${APP}.dws_sku_action_daycount partition(dt='$do_date')
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count)
from
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by sku_id;
"

# 04.优惠券主题
dws_coupon_info_daycount="
with
tmp_cu as
(
    select
        coupon_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='$do_date',1,0)) get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='$do_date',1,0)) order_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='$do_date',1,0)) payment_count,
        sum(if(date_format(expire_time,'yyyy-MM-dd')='$do_date',1,0)) expire_count
    from ${APP}.dwd_coupon_use
    where dt='9999-99-99'
    or dt='$do_date'
    group by coupon_id
),
tmp_order as
(
    select
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    and coupon_id is not null
    group by coupon_id
),
tmp_pay as
(
    select
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and coupon_id is not null
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by coupon_id
)
insert overwrite table ${APP}.dws_coupon_info_daycount partition(dt='$do_date')
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count)
from
(
    select
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by coupon_id;
"

# 05.活动主题
dws_activity_info_daycount="
with
tmp_order as
(
    select
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    and activity_id is not null
    group by activity_rule_id,activity_id
),
tmp_pay as
(
    select
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and activity_id is not null
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by activity_rule_id,activity_id
)
insert overwrite table ${APP}.dws_activity_info_daycount partition(dt='$do_date')
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by activity_rule_id,activity_id;
"

# 06.地区主题
dws_area_stats_daycount="
with
tmp_vu as
(
    select
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from ${APP}.dwd_page_log
        where dt='$do_date'
        and last_page_id is null
        group by area_code
    )tmp
    left join ${APP}.dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    where dt='$do_date'
    or dt='9999-99-99'
    and date_format(create_time,'yyyy-MM-dd')='$do_date'
    group by province_id
),
tmp_pay as
(
    select
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    where dt='$do_date'
    group by province_id
),
tmp_ro as
(
    select
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by province_id
),
tmp_rp as
(
    select
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from ${APP}.dwd_refund_payment
    where dt='$do_date'
    group by province_id
)
insert overwrite table ${APP}.dws_area_stats_daycount partition(dt='$do_date')
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount)
from
(
    select
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by province_id;
"

case $1 in
    "dws_visitor_action_daycount" )
        hive -e "$dws_visitor_action_daycount"
    ;;
    "dws_user_action_daycount" )
        hive -e "$dws_user_action_daycount"
    ;;
    "dws_sku_action_daycount" )
        hive -e "$dws_sku_action_daycount"
    ;;
    "dws_coupon_info_daycount" )
        hive -e "$dws_coupon_info_daycount"
    ;;
    "dws_activity_info_daycount" )
        hive -e "$dws_activity_info_daycount"
    ;;
    "dws_area_stats_daycount" )
        hive -e "$dws_area_stats_daycount"
    ;;
    "all" )
        hive -e "$dws_visitor_action_daycount$dws_user_action_daycount$dws_sku_action_daycount$dws_coupon_info_daycount$dws_activity_info_daycount$dws_area_stats_daycount"
    ;;
esac
```



##### ——数仓搭建-DWT层——

###### 建表语句（6个）

```sql
-- DWT层建表语句（6个） --
-- 01.访客主题
DROP TABLE IF EXISTS dwt_visitor_topic;
CREATE EXTERNAL TABLE dwt_visitor_topic
(
    `mid_id` STRING COMMENT '设备id',
    `brand` STRING COMMENT '手机品牌',
    `model` STRING COMMENT '手机型号',
    `channel` ARRAY<STRING> COMMENT '渠道',
    `os` ARRAY<STRING> COMMENT '操作系统',
    `area_code` ARRAY<STRING> COMMENT '地区ID',
    `version_code` ARRAY<STRING> COMMENT '应用版本',
    `visit_date_first` STRING  COMMENT '首次访问时间',
    `visit_date_last` STRING  COMMENT '末次访问时间',
    `visit_last_1d_count` BIGINT COMMENT '最近1日访问次数',
    `visit_last_1d_day_count` BIGINT COMMENT '最近1日访问天数',
    `visit_last_7d_count` BIGINT COMMENT '最近7日访问次数',
    `visit_last_7d_day_count` BIGINT COMMENT '最近7日访问天数',
    `visit_last_30d_count` BIGINT COMMENT '最近30日访问次数',
    `visit_last_30d_day_count` BIGINT COMMENT '最近30日访问天数',
    `visit_count` BIGINT COMMENT '累积访问次数',
    `visit_day_count` BIGINT COMMENT '累积访问天数'
) COMMENT '设备主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_visitor_topic'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 02.用户主题
DROP TABLE IF EXISTS dwt_user_topic;
CREATE EXTERNAL TABLE dwt_user_topic
(
    `user_id` STRING  COMMENT '用户id',
    `login_date_first` STRING COMMENT '首次活跃日期',
    `login_date_last` STRING COMMENT '末次活跃日期',
    `login_date_1d_count` STRING COMMENT '最近1日登录次数',
    `login_last_1d_day_count` BIGINT COMMENT '最近1日登录天数',
    `login_last_7d_count` BIGINT COMMENT '最近7日登录次数',
    `login_last_7d_day_count` BIGINT COMMENT '最近7日登录天数',
    `login_last_30d_count` BIGINT COMMENT '最近30日登录次数',
    `login_last_30d_day_count` BIGINT COMMENT '最近30日登录天数',
    `login_count` BIGINT COMMENT '累积登录次数',
    `login_day_count` BIGINT COMMENT '累积登录天数',
    `order_date_first` STRING COMMENT '首次下单时间',
    `order_date_last` STRING COMMENT '末次下单时间',
    `order_last_1d_count` BIGINT COMMENT '最近1日下单次数',
    `order_activity_last_1d_count` BIGINT COMMENT '最近1日订单参与活动次数',
    `order_activity_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日订单减免金额(活动)',
    `order_coupon_last_1d_count` BIGINT COMMENT '最近1日下单用券次数',
    `order_coupon_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日订单减免金额(优惠券)',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日原始下单金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日最终下单金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日下单次数',
    `order_activity_last_7d_count` BIGINT COMMENT '最近7日订单参与活动次数',
    `order_activity_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日订单减免金额(活动)',
    `order_coupon_last_7d_count` BIGINT COMMENT '最近7日下单用券次数',
    `order_coupon_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日订单减免金额(优惠券)',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日原始下单金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日最终下单金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日下单次数',
    `order_activity_last_30d_count` BIGINT COMMENT '最近30日订单参与活动次数',
    `order_activity_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日订单减免金额(活动)',
    `order_coupon_last_30d_count` BIGINT COMMENT '最近30日下单用券次数',
    `order_coupon_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日订单减免金额(优惠券)',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日原始下单金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日最终下单金额',
    `order_count` BIGINT COMMENT '累积下单次数',
    `order_activity_count` BIGINT COMMENT '累积订单参与活动次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '累积订单减免金额(活动)',
    `order_coupon_count` BIGINT COMMENT '累积下单用券次数',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '累积订单减免金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积原始下单金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积最终下单金额',
    `payment_date_first` STRING COMMENT '首次支付时间',
    `payment_date_last` STRING COMMENT '末次支付时间',
    `payment_last_1d_count` BIGINT COMMENT '最近1日支付次数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日支付次数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日支付次数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日支付金额',
    `payment_count` BIGINT COMMENT '累积支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1日退单次数',
    `refund_order_last_1d_num` BIGINT COMMENT '最近1日退单件数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7日退单次数',
    `refund_order_last_7d_num` BIGINT COMMENT '最近7日退单件数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30日退单次数',
    `refund_order_last_30d_num` BIGINT COMMENT '最近30日退单件数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_num` BIGINT COMMENT '累积退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1日退款次数',
    `refund_payment_last_1d_num` BIGINT COMMENT '最近1日退款件数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7日退款次数',
    `refund_payment_last_7d_num` BIGINT COMMENT '最近7日退款件数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30日退款次数',
    `refund_payment_last_30d_num` BIGINT COMMENT '最近30日退款件数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_num` BIGINT COMMENT '累积退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额',
    `cart_last_1d_count` BIGINT COMMENT '最近1日加入购物车次数',
    `cart_last_7d_count` BIGINT COMMENT '最近7日加入购物车次数',
    `cart_last_30d_count` BIGINT COMMENT '最近30日加入购物车次数',
    `cart_count` BIGINT COMMENT '累积加入购物车次数',
    `favor_last_1d_count` BIGINT COMMENT '最近1日收藏次数',
    `favor_last_7d_count` BIGINT COMMENT '最近7日收藏次数',
    `favor_last_30d_count` BIGINT COMMENT '最近30日收藏次数',
    `favor_count` BIGINT COMMENT '累积收藏次数',
    `coupon_last_1d_get_count` BIGINT COMMENT '最近1日领券次数',
    `coupon_last_1d_using_count` BIGINT COMMENT '最近1日用券(下单)次数',
    `coupon_last_1d_used_count` BIGINT COMMENT '最近1日用券(支付)次数',
    `coupon_last_7d_get_count` BIGINT COMMENT '最近7日领券次数',
    `coupon_last_7d_using_count` BIGINT COMMENT '最近7日用券(下单)次数',
    `coupon_last_7d_used_count` BIGINT COMMENT '最近7日用券(支付)次数',
    `coupon_last_30d_get_count` BIGINT COMMENT '最近30日领券次数',
    `coupon_last_30d_using_count` BIGINT COMMENT '最近30日用券(下单)次数',
    `coupon_last_30d_used_count` BIGINT COMMENT '最近30日用券(支付)次数',
    `coupon_get_count` BIGINT COMMENT '累积领券次数',
    `coupon_using_count` BIGINT COMMENT '累积用券(下单)次数',
    `coupon_used_count` BIGINT COMMENT '累积用券(支付)次数',
    `appraise_last_1d_good_count` BIGINT COMMENT '最近1日好评次数',
    `appraise_last_1d_mid_count` BIGINT COMMENT '最近1日中评次数',
    `appraise_last_1d_bad_count` BIGINT COMMENT '最近1日差评次数',
    `appraise_last_1d_default_count` BIGINT COMMENT '最近1日默认评价次数',
    `appraise_last_7d_good_count` BIGINT COMMENT '最近7日好评次数',
    `appraise_last_7d_mid_count` BIGINT COMMENT '最近7日中评次数',
    `appraise_last_7d_bad_count` BIGINT COMMENT '最近7日差评次数',
    `appraise_last_7d_default_count` BIGINT COMMENT '最近7日默认评价次数',
    `appraise_last_30d_good_count` BIGINT COMMENT '最近30日好评次数',
    `appraise_last_30d_mid_count` BIGINT COMMENT '最近30日中评次数',
    `appraise_last_30d_bad_count` BIGINT COMMENT '最近30日差评次数',
    `appraise_last_30d_default_count` BIGINT COMMENT '最近30日默认评价次数',
    `appraise_good_count` BIGINT COMMENT '累积好评次数',
    `appraise_mid_count` BIGINT COMMENT '累积中评次数',
    `appraise_bad_count` BIGINT COMMENT '累积差评次数',
    `appraise_default_count` BIGINT COMMENT '累积默认评价次数'
)COMMENT '会员主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_user_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 03.商品主题
DROP TABLE IF EXISTS dwt_sku_topic;
CREATE EXTERNAL TABLE dwt_sku_topic
(
    `sku_id` STRING COMMENT 'sku_id',
    `order_last_1d_count` BIGINT COMMENT '最近1日被下单次数',
    `order_last_1d_num` BIGINT COMMENT '最近1日被下单件数',
    `order_activity_last_1d_count` BIGINT COMMENT '最近1日参与活动被下单次数',
    `order_coupon_last_1d_count` BIGINT COMMENT '最近1日使用优惠券被下单次数',
    `order_activity_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日优惠金额(活动)',
    `order_coupon_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日优惠金额(优惠券)',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日被下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日被下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日被下单次数',
    `order_last_7d_num` BIGINT COMMENT '最近7日被下单件数',
    `order_activity_last_7d_count` BIGINT COMMENT '最近7日参与活动被下单次数',
    `order_coupon_last_7d_count` BIGINT COMMENT '最近7日使用优惠券被下单次数',
    `order_activity_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日优惠金额(活动)',
    `order_coupon_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日优惠金额(优惠券)',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日被下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日被下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日被下单次数',
    `order_last_30d_num` BIGINT COMMENT '最近30日被下单件数',
    `order_activity_last_30d_count` BIGINT COMMENT '最近30日参与活动被下单次数',
    `order_coupon_last_30d_count` BIGINT COMMENT '最近30日使用优惠券被下单次数',
    `order_activity_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日优惠金额(活动)',
    `order_coupon_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日优惠金额(优惠券)',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日被下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日被下单最终金额',
    `order_count` BIGINT COMMENT '累积被下单次数',
    `order_num` BIGINT COMMENT '累积被下单件数',
    `order_activity_count` BIGINT COMMENT '累积参与活动被下单次数',
    `order_coupon_count` BIGINT COMMENT '累积使用优惠券被下单次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '累积优惠金额(活动)',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '累积优惠金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积被下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积被下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日被支付次数',
    `payment_last_1d_num` BIGINT COMMENT '最近1日被支付件数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日被支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日被支付次数',
    `payment_last_7d_num` BIGINT COMMENT '最近7日被支付件数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日被支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日被支付次数',
    `payment_last_30d_num` BIGINT COMMENT '最近30日被支付件数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日被支付金额',
    `payment_count` BIGINT COMMENT '累积被支付次数',
    `payment_num` BIGINT COMMENT '累积被支付件数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积被支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1日退单次数',
    `refund_order_last_1d_num` BIGINT COMMENT '最近1日退单件数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7日退单次数',
    `refund_order_last_7d_num` BIGINT COMMENT '最近7日退单件数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30日退单次数',
    `refund_order_last_30d_num` BIGINT COMMENT '最近30日退单件数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_num` BIGINT COMMENT '累积退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1日退款次数',
    `refund_payment_last_1d_num` BIGINT COMMENT '最近1日退款件数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7日退款次数',
    `refund_payment_last_7d_num` BIGINT COMMENT '最近7日退款件数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30日退款次数',
    `refund_payment_last_30d_num` BIGINT COMMENT '最近30日退款件数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_num` BIGINT COMMENT '累积退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额',
    `cart_last_1d_count` BIGINT COMMENT '最近1日被加入购物车次数',
    `cart_last_7d_count` BIGINT COMMENT '最近7日被加入购物车次数',
    `cart_last_30d_count` BIGINT COMMENT '最近30日被加入购物车次数',
    `cart_count` BIGINT COMMENT '累积被加入购物车次数',
    `favor_last_1d_count` BIGINT COMMENT '最近1日被收藏次数',
    `favor_last_7d_count` BIGINT COMMENT '最近7日被收藏次数',
    `favor_last_30d_count` BIGINT COMMENT '最近30日被收藏次数',
    `favor_count` BIGINT COMMENT '累积被收藏次数',
    `appraise_last_1d_good_count` BIGINT COMMENT '最近1日好评数',
    `appraise_last_1d_mid_count` BIGINT COMMENT '最近1日中评数',
    `appraise_last_1d_bad_count` BIGINT COMMENT '最近1日差评数',
    `appraise_last_1d_default_count` BIGINT COMMENT '最近1日默认评价数',
    `appraise_last_7d_good_count` BIGINT COMMENT '最近7日好评数',
    `appraise_last_7d_mid_count` BIGINT COMMENT '最近7日中评数',
    `appraise_last_7d_bad_count` BIGINT COMMENT '最近7日差评数',
    `appraise_last_7d_default_count` BIGINT COMMENT '最近7日默认评价数',
    `appraise_last_30d_good_count` BIGINT COMMENT '最近30日好评数',
    `appraise_last_30d_mid_count` BIGINT COMMENT '最近30日中评数',
    `appraise_last_30d_bad_count` BIGINT COMMENT '最近30日差评数',
    `appraise_last_30d_default_count` BIGINT COMMENT '最近30日默认评价数',
    `appraise_good_count` BIGINT COMMENT '累积好评数',
    `appraise_mid_count` BIGINT COMMENT '累积中评数',
    `appraise_bad_count` BIGINT COMMENT '累积差评数',
    `appraise_default_count` BIGINT COMMENT '累积默认评价数'
 )COMMENT '商品主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_sku_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 04.优惠券主题
DROP TABLE IF EXISTS dwt_coupon_topic;
CREATE EXTERNAL TABLE dwt_coupon_topic(
    `coupon_id` STRING COMMENT '优惠券ID',
    `get_last_1d_count` BIGINT COMMENT '最近1日领取次数',
    `get_last_7d_count` BIGINT COMMENT '最近7日领取次数',
    `get_last_30d_count` BIGINT COMMENT '最近30日领取次数',
    `get_count` BIGINT COMMENT '累积领取次数',
    `order_last_1d_count` BIGINT COMMENT '最近1日使用某券下单次数',
    `order_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单优惠金额',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日使用某券下单次数',
    `order_last_7d_reduce_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单优惠金额',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日使用某券下单次数',
    `order_last_30d_reduce_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单优惠金额',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单最终金额',
    `order_count` BIGINT COMMENT '累积使用(下单)次数',
    `order_reduce_amount` DECIMAL(16,2) COMMENT '使用某券累积下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '使用某券累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '使用某券累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日使用某券支付次数',
    `payment_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日使用某券优惠金额',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日使用某券支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日使用某券支付次数',
    `payment_last_7d_reduce_amount` DECIMAL(16,2) COMMENT '最近7日使用某券优惠金额',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日使用某券支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日使用某券支付次数',
    `payment_last_30d_reduce_amount` DECIMAL(16,2) COMMENT '最近30日使用某券优惠金额',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日使用某券支付金额',
    `payment_count` BIGINT COMMENT '累积使用(支付)次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '使用某券累积优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '使用某券累积支付金额',
    `expire_last_1d_count` BIGINT COMMENT '最近1日过期次数',
    `expire_last_7d_count` BIGINT COMMENT '最近7日过期次数',
    `expire_last_30d_count` BIGINT COMMENT '最近30日过期次数',
    `expire_count` BIGINT COMMENT '累积过期次数'
)comment '优惠券主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_coupon_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 05.活动主题
DROP TABLE IF EXISTS dwt_activity_topic;
CREATE EXTERNAL TABLE dwt_activity_topic(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING  COMMENT '活动ID',
    `order_last_1d_count` BIGINT COMMENT '最近1日参与某活动某规则下单次数',
    `order_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单优惠金额',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单最终金额',
    `order_count` BIGINT COMMENT '参与某活动某规则累积下单次数',
    `order_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日参与某活动某规则支付次数',
    `payment_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则支付优惠金额',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则支付金额',
    `payment_count` BIGINT COMMENT '参与某活动某规则累积支付次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积支付优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积支付金额'
) COMMENT '活动主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_activity_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

-- 06.地区主题
DROP TABLE IF EXISTS dwt_area_topic;
CREATE EXTERNAL TABLE dwt_area_topic(
    `province_id` STRING COMMENT '编号',
    `visit_last_1d_count` BIGINT COMMENT '最近1日访客访问次数',
    `login_last_1d_count` BIGINT COMMENT '最近1日用户访问次数',
    `visit_last_7d_count` BIGINT COMMENT '最近7访客访问次数',
    `login_last_7d_count` BIGINT COMMENT '最近7日用户访问次数',
    `visit_last_30d_count` BIGINT COMMENT '最近30日访客访问次数',
    `login_last_30d_count` BIGINT COMMENT '最近30日用户访问次数',
    `visit_count` BIGINT COMMENT '累积访客访问次数',
    `login_count` BIGINT COMMENT '累积用户访问次数',
    `order_last_1d_count` BIGINT COMMENT '最近1天下单次数',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1天下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1天下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7天下单次数',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7天下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7天下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30天下单次数',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30天下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30天下单最终金额',
    `order_count` BIGINT COMMENT '累积下单次数',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1天支付次数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7天支付次数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30天支付次数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天支付金额',
    `payment_count` BIGINT COMMENT '累积支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1天退单次数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7天退单次数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30天退单次数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1天退款次数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7天退款次数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30天退款次数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额'
) COMMENT '地区主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_area_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");
```



###### DWT层首日数据装载脚本dws_to_dwt_init.sh

> /home/ahua/bin/dws_to_dwt_init.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

# 01.访客主题
#    case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
#         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-09'--无法获取准确的首次登录日期，给定一个数仓搭建日之前的日期
#         else old.visit_date_first end,
dwt_visitor_topic="
insert overwrite table ${APP}.dwt_visitor_topic partition(dt='$do_date')
select
    nvl(1d_ago.mid_id,old.mid_id),
    nvl(1d_ago.brand,old.brand),
    nvl(1d_ago.model,old.model),
    nvl(1d_ago.channel,old.channel),
    nvl(1d_ago.os,old.os),
    nvl(1d_ago.area_code,old.area_code),
    nvl(1d_ago.version_code,old.version_code),
    case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-09'
         else old.visit_date_first end,
    if(1d_ago.mid_id is not null,'$do_date',old.visit_date_last),
    nvl(1d_ago.visit_count,0),
    if(1d_ago.mid_id is null,0,1),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.visit_last_7d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(7d_ago.mid_id is null,0,1),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.visit_last_30d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(30d_ago.mid_id is null,0,1),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.visit_day_count,0)+if(1d_ago.mid_id is null,0,1)
from
(
    select
        mid_id,
        brand,
        model,
        channel,
        os,
        area_code,
        version_code,
        visit_date_first,
        visit_date_last,
        visit_last_1d_count,
        visit_last_1d_day_count,
        visit_last_7d_count,
        visit_last_7d_day_count,
        visit_last_30d_count,
        visit_last_30d_day_count,
        visit_count,
        visit_day_count
    from ${APP}.dwt_visitor_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt='$do_date'
)1d_ago
on old.mid_id=1d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.mid_id=7d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.mid_id=30d_ago.mid_id;
"

# 02.用户主题
#    login_date_first,--以用户的创建日期作为首次登录日期
#    nvl(login_date_last,date_add('$do_date',-1)),--若有历史登录记录，则根据历史记录获取末次登录日期，否则统一指定一个日期
dwt_user_topic="
insert overwrite table ${APP}.dwt_user_topic partition(dt='$do_date')
select
    id,
    login_date_first,
    nvl(login_date_last,date_add('$do_date',-1)),
    nvl(login_last_1d_count,0),
    nvl(login_last_1d_day_count,0),
    nvl(login_last_7d_count,0),
    nvl(login_last_7d_day_count,0),
    nvl(login_last_30d_count,0),
    nvl(login_last_30d_day_count,0),
    nvl(login_count,0),
    nvl(login_day_count,0),
    order_date_first,
    order_date_last,
    nvl(order_last_1d_count,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    payment_date_first,
    payment_date_last,
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(coupon_last_1d_get_count,0),
    nvl(coupon_last_1d_using_count,0),
    nvl(coupon_last_1d_used_count,0),
    nvl(coupon_last_7d_get_count,0),
    nvl(coupon_last_7d_using_count,0),
    nvl(coupon_last_7d_used_count,0),
    nvl(coupon_last_30d_get_count,0),
    nvl(coupon_last_30d_using_count,0),
    nvl(coupon_last_30d_used_count,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id,
        date_format(create_time,'yyyy-MM-dd') login_date_first
    from ${APP}.dim_user_info
    where dt='9999-99-99'
)t1
left join
(
    select
        user_id user_id,
        max(dt) login_date_last,
        sum(if(dt='$do_date',login_count,0)) login_last_1d_count,
        sum(if(dt='$do_date' and login_count>0,1,0)) login_last_1d_day_count,
        sum(if(dt>=date_add('$do_date',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6) and login_count>0,1,0)) login_last_7d_day_count,
        sum(if(dt>=date_add('$do_date',-29),login_count,0)) login_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29) and login_count>0,1,0)) login_last_30d_day_count,
        sum(login_count) login_count,
        sum(if(login_count>0,1,0)) login_day_count,
        min(if(order_count>0,dt,null)) order_date_first,
        max(if(order_count>0,dt,null)) order_date_last,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='$do_date',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='$do_date',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_activity_count) order_activity_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_count) order_coupon_count,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        min(if(payment_count>0,dt,null)) payment_date_first,
        max(if(payment_count>0,dt,null)) payment_date_last,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='$do_date',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='$do_date',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='$do_date',coupon_get_count,0)) coupon_last_1d_get_count,
        sum(if(dt='$do_date',coupon_using_count,0)) coupon_last_1d_using_count,
        sum(if(dt='$do_date',coupon_used_count,0)) coupon_last_1d_used_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_get_count,0)) coupon_last_7d_get_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_using_count,0)) coupon_last_7d_using_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_used_count,0)) coupon_last_7d_used_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_get_count,0)) coupon_last_30d_get_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_using_count,0)) coupon_last_30d_using_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_used_count,0)) coupon_last_30d_used_count,
        sum(coupon_get_count) coupon_get_count,
        sum(coupon_using_count) coupon_using_count,
        sum(coupon_used_count) coupon_used_count,
        sum(if(dt='$do_date',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='$do_date',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='$do_date',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='$do_date',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from ${APP}.dws_user_action_daycount
    group by user_id
)t2
on t1.id=t2.user_id;
"

# 03.商品主题
dwt_sku_topic="
insert overwrite table ${APP}.dwt_sku_topic partition(dt='$do_date')
select
    id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_num,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_num,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_num,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_num,0),
    nvl(order_activity_count,0),
    nvl(order_coupon_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_num,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_num,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_num,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_num,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id
    from ${APP}.dim_sku_info
    where dt='$do_date'
)t1
left join
(
    select
        sku_id,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_num,0)) order_last_1d_num,
        sum(if(dt='$do_date',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='$do_date',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='$do_date',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_num,0)) order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_num,0)) order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_num) order_num,
        sum(order_activity_count) order_activity_count,
        sum(order_coupon_count) order_coupon_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_num,0)) payment_last_1d_num,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_num,0)) payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_num,0)) payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_num) payment_num,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='$do_date',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='$do_date',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='$do_date',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='$do_date',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='$do_date',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='$do_date',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from ${APP}.dws_sku_action_daycount
    group by sku_id
)t2
on t1.id=t2.sku_id;
"

# 04.优惠券主题
dwt_coupon_topic="
insert overwrite table ${APP}.dwt_coupon_topic partition(dt='$do_date')
select
    id,
    nvl(get_last_1d_count,0),
    nvl(get_last_7d_count,0),
    nvl(get_last_30d_count,0),
    nvl(get_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_reduce_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_reduce_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_reduce_amount,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_reduce_amount,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0),
    nvl(expire_last_1d_count,0),
    nvl(expire_last_7d_count,0),
    nvl(expire_last_30d_count,0),
    nvl(expire_count,0)
from
(
    select
        id
    from ${APP}.dim_coupon_info
    where dt='$do_date'
)t1
left join
(
    select
        coupon_id coupon_id,
        sum(if(dt='$do_date',get_count,0)) get_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),get_count,0)) get_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),get_count,0)) get_last_30d_count,
        sum(get_count) get_count,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_reduce_amount,0)) order_last_7d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_reduce_amount,0)) order_last_30d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_reduce_amount,0)) payment_last_7d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_reduce_amount,0)) payment_last_30d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',expire_count,0)) expire_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),expire_count,0)) expire_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),expire_count,0)) expire_last_30d_count,
        sum(expire_count) expire_count
    from ${APP}.dws_coupon_info_daycount
    group by coupon_id
)t2
on t1.id=t2.coupon_id;
"

# 05.活动主题
dwt_activity_topic="
insert overwrite table ${APP}.dwt_activity_topic partition(dt='$do_date')
select
    t1.activity_rule_id,
    t1.activity_id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0)
from
(
    select
        activity_rule_id,
        activity_id
    from ${APP}.dim_activity_rule_info
    where dt='$do_date'
)t1
left join
(
    select
        activity_rule_id,
        activity_id,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount
    from ${APP}.dws_activity_info_daycount
    group by activity_rule_id,activity_id
)t2
on t1.activity_rule_id=t2.activity_rule_id
and t1.activity_id=t2.activity_id;
"

# 06.地区主题
dwt_area_topic="
insert overwrite table ${APP}.dwt_area_topic partition(dt='$do_date')
select
    id,
    nvl(visit_last_1d_count,0),
    nvl(login_last_1d_count,0),
    nvl(visit_last_7d_count,0),
    nvl(login_last_7d_count,0),
    nvl(visit_last_30d_count,0),
    nvl(login_last_30d_count,0),
    nvl(visit_count,0),
    nvl(login_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_amount,0)
from
(
    select
        id
    from ${APP}.dim_base_province
)t1
left join
(
    select
        province_id province_id,
        sum(if(dt='$do_date',visit_count,0)) visit_last_1d_count,
        sum(if(dt='$do_date',login_count,0)) login_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),visit_count,0)) visit_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),visit_count,0)) visit_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),login_count,0)) login_last_30d_count,
        sum(visit_count) visit_count,
        sum(login_count) login_count,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_amount) refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    group by province_id
)t2
on t1.id=t2.province_id;
"

case $1 in
    "dwt_visitor_topic" )
        hive -e "$dwt_visitor_topic"
    ;;
    "dwt_user_topic" )
        hive -e "$dwt_user_topic"
    ;;
    "dwt_sku_topic" )
        hive -e "$dwt_sku_topic"
    ;;
    "dwt_coupon_topic" )
        hive -e "$dwt_coupon_topic"
    ;;
    "dwt_activity_topic" )
        hive -e "$dwt_activity_topic"
    ;;
    "dwt_area_topic" )
        hive -e "$dwt_area_topic"
    ;;
    "all" )
        hive -e "$dwt_visitor_topic$dwt_user_topic$dwt_sku_topic$dwt_coupon_topic$dwt_activity_topic$dwt_area_topic"
    ;;
esac
```



###### DWT层每日数据装载脚本dws_to_dwt_daily.sh

> /home/ahua/bin/dws_to_dwt_daily.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

clear_date=`date -d "$do_date -2 day" +%F`

# 01.访客主题
#     case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
#         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-09'--无法获取准确的首次登录日期，给定一个数仓搭建日之前的日期
#         else old.visit_date_first end,
dwt_visitor_topic="
insert overwrite table ${APP}.dwt_visitor_topic partition(dt='$do_date')
select
    nvl(1d_ago.mid_id,old.mid_id),
    nvl(1d_ago.brand,old.brand),
    nvl(1d_ago.model,old.model),
    nvl(1d_ago.channel,old.channel),
    nvl(1d_ago.os,old.os),
    nvl(1d_ago.area_code,old.area_code),
    nvl(1d_ago.version_code,old.version_code),
    case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-09'
         else old.visit_date_first end,
    if(1d_ago.mid_id is not null,'$do_date',old.visit_date_last),
    nvl(1d_ago.visit_count,0),
    if(1d_ago.mid_id is null,0,1),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.visit_last_7d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(7d_ago.mid_id is null,0,1),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.visit_last_30d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(30d_ago.mid_id is null,0,1),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.visit_day_count,0)+if(1d_ago.mid_id is null,0,1)
from
(
    select
        mid_id,
        brand,
        model,
        channel,
        os,
        area_code,
        version_code,
        visit_date_first,
        visit_date_last,
        visit_last_1d_count,
        visit_last_1d_day_count,
        visit_last_7d_count,
        visit_last_7d_day_count,
        visit_last_30d_count,
        visit_last_30d_day_count,
        visit_count,
        visit_day_count
    from ${APP}.dwt_visitor_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt='$do_date'
)1d_ago
on old.mid_id=1d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.mid_id=7d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.mid_id=30d_ago.mid_id;
alter table ${APP}.dwt_visitor_topic drop partition(dt='$clear_date');
"

# 02.用户主题
dwt_user_topic="
insert overwrite table ${APP}.dwt_user_topic partition(dt='$do_date')
select
    nvl(1d_ago.user_id,old.user_id),
    nvl(old.login_date_first,'$do_date'),
    if(1d_ago.user_id is not null,'$do_date',old.login_date_last),
    nvl(1d_ago.login_count,0),
    if(1d_ago.user_id is not null,1,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.login_last_7d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(7d_ago.user_id is null,0,1),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.login_last_30d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(30d_ago.user_id is null,0,1),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(old.login_day_count,0)+if(1d_ago.user_id is not null,1,0),
    if(old.order_date_first is null and 1d_ago.order_count>0, '$do_date', old.order_date_first),
    if(1d_ago.order_count>0,'$do_date',old.order_date_last),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    if(old.payment_date_first is null and 1d_ago.payment_count>0, '$do_date', old.payment_date_first),
    if(1d_ago.payment_count>0,'$do_date',old.payment_date_last),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)-nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)-nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)-nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_num,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)-nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)-nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.coupon_get_count,0),
    nvl(1d_ago.coupon_using_count,0),
    nvl(1d_ago.coupon_used_count,0),
    nvl(old.coupon_last_7d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(7d_ago.coupon_get_count,0),
    nvl(old.coupon_last_7d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(7d_ago.coupon_using_count,0),
    nvl(old.coupon_last_7d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(7d_ago.coupon_used_count,0),
    nvl(old.coupon_last_30d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(30d_ago.coupon_get_count,0),
    nvl(old.coupon_last_30d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(30d_ago.coupon_using_count,0),
    nvl(old.coupon_last_30d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(30d_ago.coupon_used_count,0),
    nvl(old.coupon_get_count,0)+nvl(1d_ago.coupon_get_count,0),
    nvl(old.coupon_using_count,0)+nvl(1d_ago.coupon_using_count,0),
    nvl(old.coupon_used_count,0)+nvl(1d_ago.coupon_used_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count, 0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        user_id,
        login_date_first,
        login_date_last,
        login_date_1d_count,
        login_last_1d_day_count,
        login_last_7d_count,
        login_last_7d_day_count,
        login_last_30d_count,
        login_last_30d_day_count,
        login_count,
        login_day_count,
        order_date_first,
        order_date_last,
        order_last_1d_count,
        order_activity_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_last_1d_count,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_activity_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_last_7d_count,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_activity_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_last_30d_count,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_date_first,
        payment_date_last,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        coupon_last_1d_get_count,
        coupon_last_1d_using_count,
        coupon_last_1d_used_count,
        coupon_last_7d_get_count,
        coupon_last_7d_using_count,
        coupon_last_7d_used_count,
        coupon_last_30d_get_count,
        coupon_last_30d_using_count,
        coupon_last_30d_used_count,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dwt_user_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt='$do_date'
)1d_ago
on old.user_id=1d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.user_id=7d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.user_id=30d_ago.user_id;
alter table ${APP}.dwt_user_topic drop partition(dt='$clear_date');
"

# 03.商品主题
dwt_sku_topic="
insert overwrite table ${APP}.dwt_sku_topic partition(dt='$do_date')
select
    nvl(1d_ago.sku_id,old.sku_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_num,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_num,0)+nvl(1d_ago.order_num,0)- nvl(7d_ago.order_num,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_num,0)+nvl(1d_ago.order_num,0)- nvl(30d_ago.order_num,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_num,0)+nvl(1d_ago.order_num,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_num,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_num,0)+nvl(1d_ago.payment_num,0)- nvl(7d_ago.payment_num,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_num,0)+nvl(1d_ago.payment_num,0)- nvl(30d_ago.payment_num,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_num,0)+nvl(1d_ago.payment_num,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(old.refund_order_last_1d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_last_1d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_last_1d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)- nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)- nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(1d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count,0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        sku_id,
        order_last_1d_count,
        order_last_1d_num,
        order_activity_last_1d_count,
        order_coupon_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_num,
        order_activity_last_7d_count,
        order_coupon_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_num,
        order_activity_last_30d_count,
        order_coupon_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_num,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_num,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_num,
        payment_last_30d_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dwt_sku_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt='$do_date'
)1d_ago
on old.sku_id=1d_ago.sku_id
left join
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.sku_id=7d_ago.sku_id
left join
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.sku_id=30d_ago.sku_id;
alter table ${APP}.dwt_sku_topic drop partition(dt='$clear_date');
"

# 04.优惠券主题
dwt_coupon_topic="
insert overwrite table ${APP}.dwt_coupon_topic partition(dt='$do_date')
select
    nvl(1d_ago.coupon_id,old.coupon_id),
    nvl(1d_ago.get_count,0),
    nvl(old.get_last_7d_count,0)+nvl(1d_ago.get_count,0)- nvl(7d_ago.get_count,0),
    nvl(old.get_last_30d_count,0)+nvl(1d_ago.get_count,0)- nvl(30d_ago.get_count,0),
    nvl(old.get_count,0)+nvl(1d_ago.get_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(7d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(30d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(old.payment_last_1d_count,0)+nvl(1d_ago.payment_count,0)- nvl(1d_ago.payment_count,0),
    nvl(old.payment_last_1d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_1d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(7d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(30d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.expire_count,0),
    nvl(old.expire_last_7d_count,0)+nvl(1d_ago.expire_count,0)- nvl(7d_ago.expire_count,0),
    nvl(old.expire_last_30d_count,0)+nvl(1d_ago.expire_count,0)- nvl(30d_ago.expire_count,0),
    nvl(old.expire_count,0)+nvl(1d_ago.expire_count,0)
from
(
    select
        coupon_id,
        get_last_1d_count,
        get_last_7d_count,
        get_last_30d_count,
        get_count,
        order_last_1d_count,
        order_last_1d_reduce_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_reduce_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_reduce_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_reduce_amount,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_reduce_amount,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_reduce_amount,
        payment_last_30d_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_last_1d_count,
        expire_last_7d_count,
        expire_last_30d_count,
        expire_count
    from ${APP}.dwt_coupon_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt='$do_date'
)1d_ago
on old.coupon_id=1d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.coupon_id=7d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.coupon_id=30d_ago.coupon_id;
alter table ${APP}.dwt_coupon_topic drop partition(dt='$clear_date');
"

# 05.活动主题
dwt_activity_topic="
insert overwrite table ${APP}.dwt_activity_topic partition(dt='$do_date')
select
    nvl(1d_ago.activity_rule_id,old.activity_rule_id),
    nvl(1d_ago.activity_id,old.activity_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from ${APP}.dwt_activity_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from ${APP}.dws_activity_info_daycount
    where dt='$do_date'
)1d_ago
on old.activity_rule_id=1d_ago.activity_rule_id;
alter table ${APP}.dwt_activity_topic drop partition(dt='$clear_date');
"

# 06.地区主题
dwt_area_topic="
insert overwrite table ${APP}.dwt_area_topic partition(dt='$do_date')
select
    nvl(old.province_id, 1d_ago.province_id),
    nvl(1d_ago.visit_count,0),
    nvl(1d_ago.login_count,0),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)

from
(
    select
        province_id,
        visit_last_1d_count,
        login_last_1d_count,
        visit_last_7d_count,
        login_last_7d_count,
        visit_last_30d_count,
        login_last_30d_count,
        visit_count,
        login_count,
        order_last_1d_count,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dwt_area_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt='$do_date'
)1d_ago
on old.province_id=1d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.province_id= 7d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.province_id= 30d_ago.province_id;
alter table ${APP}.dwt_area_topic drop partition(dt='$clear_date');
"

case $1 in
    "dwt_visitor_topic" )
        hive -e "$dwt_visitor_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_visitor_topic/dt=$clear_date
    ;;
    "dwt_user_topic" )
        hive -e "$dwt_user_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_user_topic/dt=$clear_date
    ;;
    "dwt_sku_topic" )
        hive -e "$dwt_sku_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_sku_topic/dt=$clear_date
    ;;
    "dwt_coupon_topic" )
        hive -e "$dwt_coupon_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_coupon_topic/dt=$clear_date
    ;;
    "dwt_activity_topic" )
        hive -e "$dwt_activity_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_activity_topic/dt=$clear_date
    ;;
    "dwt_area_topic" )
        hive -e "$dwt_area_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_area_topic/dt=$clear_date
    ;;
    "all" )
        hive -e "$dwt_visitor_topic$dwt_user_topic$dwt_sku_topic$dwt_coupon_topic$dwt_activity_topic$dwt_area_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_visitor_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_user_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_sku_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_activity_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_coupon_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_area_topic/dt=$clear_date
    ;;
esac
```



##### ——数仓搭建-ADS层——

###### 建表语句（6个主题，共12个表）

```sql
-- ADS层建表语句（6个主题，12个表） --
-- 1.访客主题
---- 1.1 访客统计
DROP TABLE IF EXISTS ads_visit_stats;
CREATE EXTERNAL TABLE ads_visit_stats (
  `dt` STRING COMMENT '统计日期',
  `is_new` STRING COMMENT '新老标识,1:新,0:老',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `channel` STRING COMMENT '渠道',
  `uv_count` BIGINT COMMENT '日活(访问人数)',
  `duration_sec` BIGINT COMMENT '页面停留总时长',
  `avg_duration_sec` BIGINT COMMENT '一次会话，页面停留平均时长,单位为描述',
  `page_count` BIGINT COMMENT '页面总浏览数',
  `avg_page_count` BIGINT COMMENT '一次会话，页面平均浏览数',
  `sv_count` BIGINT COMMENT '会话次数',
  `bounce_count` BIGINT COMMENT '跳出数',
  `bounce_rate` DECIMAL(16,2) COMMENT '跳出率'
) COMMENT '访客统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_visit_stats/';
---- 1.2 路径分析
DROP TABLE IF EXISTS ads_page_path;
CREATE EXTERNAL TABLE ads_page_path
(
    `dt` STRING COMMENT '统计日期',
    `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `source` STRING COMMENT '跳转起始页面ID',
    `target` STRING COMMENT '跳转终到页面ID',
    `path_count` BIGINT COMMENT '跳转次数'
)  COMMENT '页面浏览路径'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_page_path/';

-- 2.用户主题
---- 2.1 用户统计
DROP TABLE IF EXISTS ads_user_total;
CREATE EXTERNAL TABLE `ads_user_total` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,0:累积值,1:最近1天,7:最近7天,30:最近30天',
  `new_user_count` BIGINT COMMENT '新注册用户数',
  `new_order_user_count` BIGINT COMMENT '新增下单用户数',
  `order_final_amount` DECIMAL(16,2) COMMENT '下单总金额',
  `order_user_count` BIGINT COMMENT '下单用户数',
  `no_order_user_count` BIGINT COMMENT '未下单用户数(具体指活跃用户中未下单用户)'
) COMMENT '用户统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_total/';
---- 2.2 用户变动统计
DROP TABLE IF EXISTS ads_user_change;
CREATE EXTERNAL TABLE `ads_user_change` (
  `dt` STRING COMMENT '统计日期',
  `user_churn_count` BIGINT COMMENT '流失用户数',
  `user_back_count` BIGINT COMMENT '回流用户数'
) COMMENT '用户变动统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_change/';
---- 2.3 用户行为漏斗分析
DROP TABLE IF EXISTS ads_user_action;
CREATE EXTERNAL TABLE `ads_user_action` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `home_count` BIGINT COMMENT '浏览首页人数',
  `good_detail_count` BIGINT COMMENT '浏览商品详情页人数',
  `cart_count` BIGINT COMMENT '加入购物车人数',
  `order_count` BIGINT COMMENT '下单人数',
  `payment_count` BIGINT COMMENT '支付人数'
) COMMENT '漏斗分析'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_action/';
---- 2.4 用户留存率
DROP TABLE IF EXISTS ads_user_retention;
CREATE EXTERNAL TABLE ads_user_retention (
  `dt` STRING COMMENT '统计日期',
  `create_date` STRING COMMENT '用户新增日期',
  `retention_day` BIGINT COMMENT '截至当前日期留存天数',
  `retention_count` BIGINT COMMENT '留存用户数量',
  `new_user_count` BIGINT COMMENT '新增用户数量',
  `retention_rate` DECIMAL(16,2) COMMENT '留存率'
) COMMENT '用户留存率'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_retention/';

-- 3.商品主题
---- 3.1 商品统计
DROP TABLE IF EXISTS ads_order_spu_stats;
CREATE EXTERNAL TABLE `ads_order_spu_stats` (
    `dt` STRING COMMENT '统计日期',
    `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `spu_id` STRING COMMENT '商品ID',
    `spu_name` STRING COMMENT '商品名称',
    `tm_id` STRING COMMENT '品牌ID',
    `tm_name` STRING COMMENT '品牌名称',
    `category3_id` STRING COMMENT '三级品类ID',
    `category3_name` STRING COMMENT '三级品类名称',
    `category2_id` STRING COMMENT '二级品类ID',
    `category2_name` STRING COMMENT '二级品类名称',
    `category1_id` STRING COMMENT '一级品类ID',
    `category1_name` STRING COMMENT '一级品类名称',
    `order_count` BIGINT COMMENT '订单数',
    `order_amount` DECIMAL(16,2) COMMENT '订单金额'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_spu_stats/';
---- 3.2 品牌复购率
DROP TABLE IF EXISTS ads_repeat_purchase;
CREATE EXTERNAL TABLE `ads_repeat_purchase` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `tm_id` STRING COMMENT '品牌ID',
  `tm_name` STRING COMMENT '品牌名称',
  `order_repeat_rate` DECIMAL(16,2) COMMENT '复购率'
) COMMENT '品牌复购率'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_repeat_purchase/';

-- 4.订单主题
---- 4.1 订单统计
DROP TABLE IF EXISTS ads_order_total;
CREATE EXTERNAL TABLE `ads_order_total` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `order_count` BIGINT COMMENT '订单数',
  `order_amount` DECIMAL(16,2) COMMENT '订单金额',
  `order_user_count` BIGINT COMMENT '下单人数'
) COMMENT '订单统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_total/';
---- 4.2 各地区订单统计
DROP TABLE IF EXISTS ads_order_by_province;
CREATE EXTERNAL TABLE `ads_order_by_province` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `province_id` STRING COMMENT '省份ID',
  `province_name` STRING COMMENT '省份名称',
  `area_code` STRING COMMENT '地区编码',
  `iso_code` STRING COMMENT '国际标准地区编码',
  `iso_code_3166_2` STRING COMMENT '国际标准地区编码',
  `order_count` BIGINT COMMENT '订单数',
  `order_amount` DECIMAL(16,2) COMMENT '订单金额'
) COMMENT '各地区订单统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_by_province/';

-- 5.优惠券主题
---- 5.1 优惠券统计
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE EXTERNAL TABLE ads_coupon_stats (
  `dt` STRING COMMENT '统计日期',
  `coupon_id` STRING COMMENT '优惠券ID',
  `coupon_name` STRING COMMENT '优惠券名称',
  `start_date` STRING COMMENT '发布日期',
  `rule_name` STRING COMMENT '优惠规则，例如满100元减10元',
  `get_count`  BIGINT COMMENT '领取次数',
  `order_count` BIGINT COMMENT '使用(下单)次数',
  `expire_count`  BIGINT COMMENT '过期次数',
  `order_original_amount` DECIMAL(16,2) COMMENT '使用优惠券订单原始金额',
  `order_final_amount` DECIMAL(16,2) COMMENT '使用优惠券订单最终金额',
  `reduce_amount` DECIMAL(16,2) COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) COMMENT '补贴率'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_coupon_stats/';

-- 6.活动主题
---- 6.1 活动统计
DROP TABLE IF EXISTS ads_activity_stats;
CREATE EXTERNAL TABLE `ads_activity_stats` (
  `dt` STRING COMMENT '统计日期',
  `activity_id` STRING COMMENT '活动ID',
  `activity_name` STRING COMMENT '活动名称',
  `start_date` STRING COMMENT '活动开始日期',
  `order_count` BIGINT COMMENT '参与活动订单数',
  `order_original_amount` DECIMAL(16,2) COMMENT '参与活动订单原始金额',
  `order_final_amount` DECIMAL(16,2) COMMENT '参与活动订单最终金额',
  `reduce_amount` DECIMAL(16,2) COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) COMMENT '补贴率'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_activity_stats/';
```

###### ADS层数据装载脚本dwt_to_ads.sh

> /home/ahua/bin/dwt_to_ads.sh

```shell
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果输入了日期,则取输入日期;如果未输入日期,则取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

# -- 6.活动主题
# ---- 6.1 活动统计
ads_activity_stats="
insert overwrite table ${APP}.ads_activity_stats
select * from ${APP}.ads_activity_stats
union
select
    '$do_date' dt,
    t4.activity_id,
    activity_name,
    start_date,
    order_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        activity_id,
        activity_name,
        date_format(start_time,'yyyy-MM-dd') start_date
    from ${APP}.dim_activity_rule_info
    where dt='$do_date'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('$do_date',-29)
    group by activity_id,activity_name,start_time
)t4
left join
(
    select
        activity_id,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(order_reduce_amount) reduce_amount,
        cast(sum(order_reduce_amount)/sum(order_original_amount)*100 as decimal(16,2)) reduce_rate
    from ${APP}.dwt_activity_topic
    where dt='$do_date'
    group by activity_id
)t5
on t4.activity_id=t5.activity_id;
"

# -- 5.优惠券主题
# ---- 5.1 优惠券统计
ads_coupon_stats="
insert overwrite table ${APP}.ads_coupon_stats
select * from ${APP}.ads_coupon_stats
union
select
    '$do_date' dt,
    t1.id,
    coupon_name,
    start_date,
    rule_name,
    get_count,
    order_count,
    expire_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        id,
        coupon_name,
        date_format(start_time,'yyyy-MM-dd') start_date,
        case
            when coupon_type='3201' then concat('满',condition_amount,'元减',benefit_amount,'元')
            when coupon_type='3202' then concat('满',condition_num,'件打', (1-benefit_discount)*10,'折')
            when coupon_type='3203' then concat('减',benefit_amount,'元')
        end rule_name
    from ${APP}.dim_coupon_info
    where dt='$do_date'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('$do_date',-29)
)t1
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        expire_count,
        order_original_amount,
        order_final_amount,
        order_reduce_amount reduce_amount,
        cast(order_reduce_amount/order_original_amount as decimal(16,2)) reduce_rate
    from ${APP}.dwt_coupon_topic
    where dt='$do_date'
)t2
on t1.id=t2.coupon_id;
"

# ---- 4.2 各地区订单统计
ads_order_by_province="
insert overwrite table ${APP}.ads_order_by_province
select * from ${APP}.ads_order_by_province
union
select
    dt,
    recent_days,
    province_id,
    province_name,
    area_code,
    iso_code,
    iso_3166_2,
    order_count,
    order_amount
from
(
    select
        '$do_date' dt,
        recent_days,
        province_id,
        sum(order_count) order_count,
        sum(order_amount) order_amount
    from
    (
        select
            recent_days,
            province_id,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then order_last_1d_final_amount
                when recent_days=7 then order_last_7d_final_amount
                when recent_days=30 then order_last_30d_final_amount
            end order_amount
        from ${APP}.dwt_area_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='$do_date'
    )t1
    group by recent_days,province_id
)t2
join ${APP}.dim_base_province t3
on t2.province_id=t3.id;
"

# -- 3.商品主题
# ---- 3.1 商品统计
ads_order_spu_stats="
insert overwrite table ${APP}.ads_order_spu_stats
select * from ${APP}.ads_order_spu_stats
union
select
    '$do_date' dt,
    recent_days,
    spu_id,
    spu_name,
    tm_id,
    tm_name,
    category3_id,
    category3_name,
    category2_id,
    category2_name,
    category1_id,
    category1_name,
    sum(order_count),
    sum(order_amount)
from
(
    select
        recent_days,
        sku_id,
        case
            when recent_days=1 then order_last_1d_count
            when recent_days=7 then order_last_7d_count
            when recent_days=30 then order_last_30d_count
        end order_count,
        case
            when recent_days=1 then order_last_1d_final_amount
            when recent_days=7 then order_last_7d_final_amount
            when recent_days=30 then order_last_30d_final_amount
        end order_amount
    from ${APP}.dwt_sku_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
left join
(
    select
        id,
        spu_id,
        spu_name,
        tm_id,
        tm_name,
        category3_id,
        category3_name,
        category2_id,
        category2_name,
        category1_id,
        category1_name
    from ${APP}.dim_sku_info
    where dt='$do_date'
)t2
on t1.sku_id=t2.id
group by recent_days,spu_id,spu_name,tm_id,tm_name,category3_id,category3_name,category2_id,category2_name,category1_id,category1_name;
"

# -- 4.订单主题
# ---- 4.1 订单统计
ads_order_total="
insert overwrite table ${APP}.ads_order_total
select * from ${APP}.ads_order_total
union
select
    '$do_date',
    recent_days,
    sum(order_count),
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count
from
(
    select
        recent_days,
        user_id,
        case when recent_days=0 then order_count
             when recent_days=1 then order_last_1d_count
             when recent_days=7 then order_last_7d_count
             when recent_days=30 then order_last_30d_count
        end order_count,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount
    from ${APP}.dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
group by recent_days;
"

# ---- 1.2 路径分析
ads_page_path="
insert overwrite table ${APP}.ads_page_path
select * from ${APP}.ads_page_path
union
select
    '$do_date',
    recent_days,
    source,
    target,
    count(*)
from
(
    select
        recent_days,
        concat('step-',step,':',source) source,
        concat('step-',step+1,':',target) target
    from
    (
        select
            recent_days,
            page_id source,
            lead(page_id,1,null) over (partition by recent_days,session_id order by ts) target,
            row_number() over (partition by recent_days,session_id order by ts) step
        from
        (
            select
                recent_days,
                last_page_id,
                page_id,
                ts,
                concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by mid_id,recent_days order by ts)) session_id
            from ${APP}.dwd_page_log lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('$do_date',-30)
            and dt>=date_add('$do_date',-recent_days+1)
        )t2
    )t3
)t4
group by recent_days,source,target;
"

# ---- 3.2 品牌复购率
ads_repeat_purchase="
insert overwrite table ${APP}.ads_repeat_purchase
select * from ${APP}.ads_repeat_purchase
union
select
    '$do_date' dt,
    recent_days,
    tm_id,
    tm_name,
    cast(sum(if(order_count>=2,1,0))/sum(if(order_count>=1,1,0))*100 as decimal(16,2))
from
(
    select
        recent_days,
        user_id,
        tm_id,
        tm_name,
        sum(order_count) order_count
    from
    (
        select
            recent_days,
            user_id,
            sku_id,
            count(*) order_count
        from ${APP}.dwd_order_detail lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('$do_date',-29)
        and dt>=date_add('$do_date',-recent_days+1)
        group by recent_days, user_id,sku_id
    )t1
    left join
    (
        select
            id,
            tm_id,
            tm_name
        from ${APP}.dim_sku_info
        where dt='$do_date'
    )t2
    on t1.sku_id=t2.id
    group by recent_days,user_id,tm_id,tm_name
)t3
group by recent_days,tm_id,tm_name;
"

# ---- 2.3 用户行为漏斗分析
ads_user_action="
with
tmp_page as
(
    select
        '$do_date' dt,
        recent_days,
        sum(if(array_contains(pages,'home'),1,0)) home_count,
        sum(if(array_contains(pages,'good_detail'),1,0)) good_detail_count
    from
    (
        select
            recent_days,
            mid_id,
            collect_set(page_id) pages
        from
        (
            select
                dt,
                mid_id,
                page.page_id
            from ${APP}.dws_visitor_action_daycount lateral view explode(page_stats) tmp as page
            where dt>=date_add('$do_date',-29)
            and page.page_id in('home','good_detail')
        )t1 lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('$do_date',-recent_days+1)
        group by recent_days,mid_id
    )t2
    group by recent_days
),
tmp_cop as
(
    select
        '$do_date' dt,
        recent_days,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from
    (
        select
            recent_days,
            user_id,
            case
                when recent_days=1 then cart_last_1d_count
                when recent_days=7 then cart_last_7d_count
                when recent_days=30 then cart_last_30d_count
            end cart_count,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then payment_last_1d_count
                when recent_days=7 then payment_last_7d_count
                when recent_days=30 then payment_last_30d_count
            end payment_count
        from ${APP}.dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='$do_date'
    )t1
    group by recent_days
)
insert overwrite table ${APP}.ads_user_action
select * from ${APP}.ads_user_action
union
select
    tmp_page.dt,
    tmp_page.recent_days,
    home_count,
    good_detail_count,
    cart_count,
    order_count,
    payment_count
from tmp_page
join tmp_cop
on tmp_page.recent_days=tmp_cop.recent_days;
"

# ---- 2.2 用户变动统计
ads_user_change="
insert overwrite table ${APP}.ads_user_change
select * from ${APP}.ads_user_change
union
select
    churn.dt,
    user_churn_count,
    user_back_count
from
(
    select
        '$do_date' dt,
        count(*) user_churn_count
    from ${APP}.dwt_user_topic
    where dt='$do_date'
    and login_date_last=date_add('$do_date',-7)
)churn
join
(
    select
        '$do_date' dt,
        count(*) user_back_count
    from
    (
        select
            user_id,
            login_date_last
        from ${APP}.dwt_user_topic
        where dt='$do_date'
        and login_date_last='$do_date'
    )t1
    join
    (
        select
            user_id,
            login_date_last login_date_previous
        from ${APP}.dwt_user_topic
        where dt=date_add('$do_date',-1)
    )t2
    on t1.user_id=t2.user_id
    where datediff(login_date_last,login_date_previous)>=8
)back
on churn.dt=back.dt;
"

# ---- 2.4 用户留存率
ads_user_retention="
insert overwrite table ${APP}.ads_user_retention
select * from ${APP}.ads_user_retention
union
select
    '$do_date',
    login_date_first create_date,
    datediff('$do_date',login_date_first) retention_day,
    sum(if(login_date_last='$do_date',1,0)) retention_count,
    count(*) new_user_count,
    cast(sum(if(login_date_last='$do_date',1,0))/count(*)*100 as decimal(16,2)) retention_rate
from ${APP}.dwt_user_topic
where dt='$do_date'
and login_date_first>=date_add('$do_date',-7)
and login_date_first<'$do_date'
group by login_date_first;
"

# -- 2.用户主题
# ---- 2.1 用户统计
ads_user_total="
insert overwrite table ${APP}.ads_user_total
select * from ${APP}.ads_user_total
union
select
    '$do_date',
    recent_days,
    sum(if(login_date_first>=recent_days_ago,1,0)) new_user_count,
    sum(if(order_date_first>=recent_days_ago,1,0)) new_order_user_count,
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count,
    sum(if(login_date_last>=recent_days_ago and order_final_amount=0,1,0)) no_order_user_count
from
(
    select
        recent_days,
        user_id,
        login_date_first,
        login_date_last,
        order_date_first,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount,
        if(recent_days=0,'1970-01-01',date_add('$do_date',-recent_days+1)) recent_days_ago
    from ${APP}.dwt_user_topic lateral view explode(Array(0,1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
group by recent_days;
"

# -- 1.访客主题
# ---- 1.1 访客统计
ads_visit_stats="
insert overwrite table ${APP}.ads_visit_stats
select * from ${APP}.ads_visit_stats
union
select
    '$do_date' dt,
    is_new,
    recent_days,
    channel,
    count(distinct(mid_id)) uv_count,
    cast(sum(duration)/1000 as bigint) duration_sec,
    cast(avg(duration)/1000 as bigint) avg_duration_sec,
    sum(page_count) page_count,
    cast(avg(page_count) as bigint) avg_page_count,
    count(*) sv_count,
    sum(if(page_count=1,1,0)) bounce_count,
    cast(sum(if(page_count=1,1,0))/count(*)*100 as decimal(16,2)) bounce_rate
from
(
    select
        session_id,
        mid_id,
        is_new,
        recent_days,
        channel,
        count(*) page_count,
        sum(during_time) duration
    from
    (
        select
            mid_id,
            channel,
            recent_days,
            is_new,
            last_page_id,
            page_id,
            during_time,
            concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by recent_days,mid_id order by ts)) session_id
        from
        (
            select
                mid_id,
                channel,
                last_page_id,
                page_id,
                during_time,
                ts,
                recent_days,
                if(visit_date_first>=date_add('$do_date',-recent_days+1),'1','0') is_new
            from
            (
                select
                    t1.mid_id,
                    t1.channel,
                    t1.last_page_id,
                    t1.page_id,
                    t1.during_time,
                    t1.dt,
                    t1.ts,
                    t2.visit_date_first
                from
                (
                    select
                        mid_id,
                        channel,
                        last_page_id,
                        page_id,
                        during_time,
                        dt,
                        ts
                    from ${APP}.dwd_page_log
                    where dt>=date_add('$do_date',-30)
                )t1
                left join
                (
                    select
                        mid_id,
                        visit_date_first
                    from ${APP}.dwt_visitor_topic
                    where dt='$do_date'
                )t2
                on t1.mid_id=t2.mid_id
            )t3 lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('$do_date',-recent_days+1)
        )t4
    )t5
    group by session_id,mid_id,is_new,recent_days,channel
)t6
group by is_new,recent_days,channel;
"

case $1 in
    "ads_activity_stats" )
        hive -e "$ads_activity_stats" 
    ;;
    "ads_coupon_stats" )
        hive -e "$ads_coupon_stats"
    ;;
    "ads_order_by_province" )
        hive -e "$ads_order_by_province" 
    ;;
    "ads_order_spu_stats" )
        hive -e "$ads_order_spu_stats" 
    ;;
    "ads_order_total" )
        hive -e "$ads_order_total" 
    ;;
    "ads_page_path" )
        hive -e "$ads_page_path" 
    ;;
    "ads_repeat_purchase" )
        hive -e "$ads_repeat_purchase" 
    ;;
    "ads_user_action" )
        hive -e "$ads_user_action" 
    ;;
    "ads_user_change" )
        hive -e "$ads_user_change" 
    ;;
    "ads_user_retention" )
        hive -e "$ads_user_retention" 
    ;;
    "ads_user_total" )
        hive -e "$ads_user_total" 
    ;;
    "ads_visit_stats" )
        hive -e "$ads_visit_stats" 
    ;;
    "all" )
        hive -e "$ads_activity_stats$ads_coupon_stats$ads_order_by_province$ads_order_spu_stats$ads_order_total$ads_page_path$ads_repeat_purchase$ads_user_action$ads_user_change$ads_user_retention$ads_user_total$ads_visit_stats"
    ;;
esac
```

##### 创建接收ADS层数据的MySQL数据库gmall_report和12个表以及Sqoop数据导出脚本

###### 建表语句（与ADS层的表对应的12个表）

```sql
#（1）访客统计
DROP TABLE IF EXISTS ads_visit_stats;
CREATE TABLE `ads_visit_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `is_new` VARCHAR(255) NOT NULL COMMENT '新老标识,1:新,0:老',
  `recent_days` INT NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `channel` VARCHAR(255) NOT NULL COMMENT '渠道',
  `uv_count` BIGINT(20) DEFAULT NULL COMMENT '日活(访问人数)',
  `duration_sec` BIGINT(20) DEFAULT NULL COMMENT '页面停留总时长',
  `avg_duration_sec` BIGINT(20)  DEFAULT NULL COMMENT '一次会话，页面停留平均时长',
  `page_count` BIGINT(20) DEFAULT NULL COMMENT '页面总浏览数',
  `avg_page_count` BIGINT(20) DEFAULT NULL COMMENT '一次会话，页面平均浏览数',
  `sv_count` BIGINT(20) DEFAULT NULL COMMENT '会话次数',
  `bounce_count` BIGINT(20) DEFAULT NULL COMMENT '跳出数',
  `bounce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '跳出率',
  PRIMARY KEY (`dt`,`recent_days`,`is_new`,`channel`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
#（2）页面路径分析
DROP TABLE IF EXISTS ads_page_path;
CREATE TABLE `ads_page_path` (      
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `source` VARCHAR(255) DEFAULT NULL COMMENT '跳转起始页面',
  `target` VARCHAR(255) DEFAULT NULL COMMENT '跳转终到页面',
  `path_count` BIGINT(255) DEFAULT NULL COMMENT '跳转次数',
  UNIQUE KEY (`dt`,`recent_days`,`source`,`target`) USING BTREE     
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（3）用户统计
DROP TABLE IF EXISTS ads_user_total;
CREATE TABLE `ads_user_total` (          
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,0:累积值,1:最近1天,7:最近7天,30:最近30天',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新注册用户数',
  `new_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增下单用户数',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '下单总金额',
  `order_user_count` BIGINT(20) DEFAULT NULL COMMENT '下单用户数',
  `no_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '未下单用户数(具体指活跃用户中未下单用户)',
  PRIMARY KEY (`dt`,`recent_days`)           
) ENGINE=INNODB DEFAULT CHARSET=utf8;
#（4）用户变动统计
DROP TABLE IF EXISTS ads_user_change;
CREATE TABLE `ads_user_change` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `user_churn_count` BIGINT(20) DEFAULT NULL  COMMENT '流失用户数',
  `user_back_count` BIGINT(20) DEFAULT NULL  COMMENT '回流用户数',
  PRIMARY KEY (`dt`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
#（5）用户行为漏斗分析
DROP TABLE IF EXISTS ads_user_action;
CREATE TABLE `ads_user_action` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `home_count` BIGINT(20) DEFAULT NULL COMMENT '浏览首页人数',
  `good_detail_count` BIGINT(20) DEFAULT NULL COMMENT '浏览商品详情页人数',
  `cart_count` BIGINT(20) DEFAULT NULL COMMENT '加入购物车人数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '下单人数',
  `payment_count` BIGINT(20) DEFAULT NULL COMMENT '支付人数',
  PRIMARY KEY (`dt`,`recent_days`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（6）用户留存率分析
DROP TABLE IF EXISTS ads_user_retention;
CREATE TABLE `ads_user_retention` (      
  `dt` DATE DEFAULT NULL COMMENT '统计日期',
  `create_date` VARCHAR(255) NOT NULL COMMENT '用户新增日期',
  `retention_day` BIGINT(20) NOT NULL COMMENT '截至当前日期留存天数',
  `retention_count` BIGINT(20) DEFAULT NULL COMMENT '留存用户数量',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增用户数量',
  `retention_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '留存率',
  PRIMARY KEY (`create_date`,`retention_day`) USING BTREE        
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（7）订单统计
DROP TABLE IF EXISTS ads_order_total;
 CREATE TABLE `ads_order_total` (   
  `dt` DATE NOT NULL COMMENT '统计日期', 
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `order_count` BIGINT(255) DEFAULT NULL COMMENT '订单数', 
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  `order_user_count` BIGINT(255) DEFAULT NULL COMMENT '下单人数',
  PRIMARY KEY (`dt`,`recent_days`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（8）各省份订单统计
DROP TABLE IF EXISTS ads_order_by_province;
CREATE TABLE `ads_order_by_province` (
  `dt` DATE NOT NULL,
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `province_id` VARCHAR(255) NOT NULL COMMENT '统计日期',
  `province_name` VARCHAR(255) DEFAULT NULL COMMENT '省份名称',
  `area_code` VARCHAR(255) DEFAULT NULL COMMENT '地区编码',
  `iso_code` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `iso_code_3166_2` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额',
  PRIMARY KEY (`dt`, `recent_days` ,`province_id`) USING BTREE       
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（9）品牌复购率
DROP TABLE IF EXISTS ads_repeat_purchase;
CREATE TABLE `ads_repeat_purchase` (         
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `order_repeat_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '复购率',
  PRIMARY KEY (`dt` ,`recent_days`,`tm_id`)          
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（10）商品统计
DROP TABLE IF EXISTS ads_order_spu_stats;
CREATE TABLE `ads_order_spu_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `spu_id` VARCHAR(255) NOT NULL COMMENT '商品ID',
  `spu_name` VARCHAR(255) DEFAULT NULL COMMENT '商品名称',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `category3_id` VARCHAR(255) NOT NULL COMMENT '三级品类ID',
  `category3_name` VARCHAR(255) DEFAULT NULL COMMENT '三级品类名称',
  `category2_id` VARCHAR(255) NOT NULL COMMENT '二级品类ID',
  `category2_name` VARCHAR(255) DEFAULT NULL COMMENT '二级品类名称',
  `category1_id` VARCHAR(255) NOT NULL COMMENT '一级品类ID',
  `category1_name` VARCHAR(255) NOT NULL COMMENT '一级品类名称',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  PRIMARY KEY (`dt`,`recent_days`,`spu_id`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8;
#（11）活动统计
DROP TABLE IF EXISTS ads_activity_stats;
CREATE TABLE `ads_activity_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `activity_id` VARCHAR(255) NOT NULL COMMENT '活动ID',
  `activity_name` VARCHAR(255) DEFAULT NULL COMMENT '活动名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',
  `order_count` BIGINT(11) DEFAULT NULL COMMENT '参与活动订单数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`activity_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
#（12）优惠券统计
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE TABLE `ads_coupon_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `coupon_id` VARCHAR(255) NOT NULL COMMENT '优惠券ID',
  `coupon_name` VARCHAR(255) DEFAULT NULL COMMENT '优惠券名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',  
  `rule_name`  VARCHAR(200) DEFAULT NULL COMMENT '优惠规则',
  `get_count`  BIGINT(20) DEFAULT NULL COMMENT '领取次数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '使用(下单)次数',
  `expire_count`  BIGINT(20) DEFAULT NULL COMMENT '过期次数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`coupon_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

###### Sqoop 导出脚本hdfs_to_mysql.sh

> /home/ahua/bin/hdfs_to_mysql.sh

> hdfs_to_mysql.sh all

```shell
#!/bin/bash

hive_db_name=gmall
mysql_db_name=gmall_report

export_data() {
/opt/module/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop export \
--connect "jdbc:mysql://hadoop102:3306/${mysql_db_name}?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password ahuamysql \
--table $1 \
--num-mappers 1 \
--export-dir /warehouse/$hive_db_name/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $2 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}

case $1 in
  "ads_activity_stats" )
    export_data "ads_activity_stats" "dt,activity_id"
  ;;

  "ads_coupon_stats" )
    export_data "ads_coupon_stats" "dt,coupon_id"
  ;;

  "ads_order_by_province" )
    export_data "ads_order_by_province" "dt,recent_days,province_id"
  ;;

  "ads_order_spu_stats" )
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
  ;;

  "ads_order_total" )
    export_data "ads_order_total" "dt,recent_days"
  ;;

  "ads_page_path" )
    export_data "ads_page_path" "dt,recent_days,source,target"
  ;;

  "ads_repeat_purchase" )
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
  ;;

  "ads_user_action" )
    export_data "ads_user_action" "dt,recent_days"
  ;;

  "ads_user_change" )
    export_data "ads_user_change" "dt"
  ;;

  "ads_user_retention" )
    export_data "ads_user_retention" "create_date,retention_day"
  ;;

  "ads_user_total" )
    export_data "ads_user_total" "dt,recent_days"
  ;;

  "ads_visit_stats" )
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
  "all" )
    export_data "ads_activity_stats" "dt,activity_id"
    export_data "ads_coupon_stats" "dt,coupon_id"
    export_data "ads_order_by_province" "dt,recent_days,province_id"
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
    export_data "ads_order_total" "dt,recent_days"
    export_data "ads_page_path" "dt,recent_days,source,target"
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
    export_data "ads_user_action" "dt,recent_days"
    export_data "ads_user_change" "dt"
    export_data "ads_user_retention" "create_date,retention_day"
    export_data "ads_user_total" "dt,recent_days"
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
esac
```

## 流程调度

###### Azkaban群起群关脚本azkaban.sh

> /home/ahua/bin/azkaban.sh

```bash
#!/bin/bash

start-web(){

	for i in hadoop102
	do
		echo "=============== 启动 $i azkaban-web-server ==============="
		ssh $i "cd /opt/module/azkaban/azkaban-web-server-3.84.4/ ; bin/start-web.sh"
	done
}

stop-web(){

	for i in hadoop102
	do
		echo "=============== 停止 $i azkaban-web-server ==============="
		ssh $i "cd /opt/module/azkaban/azkaban-web-server-3.84.4/ ; bin/shutdown-web.sh"
	done
}

start-exec(){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== 启动 $i azkaban-exec-server ==============="
		ssh $i "cd /opt/module/azkaban/azkaban-exec-server-3.84.4/ ; bin/start-exec.sh"
	done
}

activate-exec(){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== 激活 $i azkaban-exec-server ==============="
		ssh $i "curl -G "$i:12321/executor?action=activate" && echo"
	done
}

stop-exec(){

	for i in hadoop102 hadoop103 hadoop104
	do
		echo "=============== 停止 $i azkaban-exec-server ==============="
		ssh $i "cd /opt/module/azkaban/azkaban-exec-server-3.84.4/ ; bin/shutdown-exec.sh"
	done
}

case $1 in
"start-web" ){
	start-web
};;
"stop-web" ){
	stop-web
};;
"start-exec" ){
	start-exec
};;
"activate-exec" ){
	activate-exec
};;
"stop-exec" ){
	stop-exec
};;
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 azkaban.sh

###### Azkaban工作流程配置文件

azkaban.project

```yaml
azkaban-flow-version: 2.0
```

gmall.flow

```yaml
nodes:
  - name: mysql_to_hdfs_daily
    type: command
    config:
     command: /home/ahua/bin/mysql_to_hdfs_daily.sh all ${dt}
    
  - name: hdfs_to_ods_log
    type: command
    config:
     command: /home/ahua/bin/hdfs_to_ods_log.sh ${dt}
     
  - name: hdfs_to_ods_db_daily
    type: command
    dependsOn: 
     - mysql_to_hdfs_daily
    config: 
     command: /home/ahua/bin/hdfs_to_ods_db_daily.sh all ${dt}
  
  - name: ods_to_dim_db_daily
    type: command
    dependsOn: 
     - hdfs_to_ods_db_daily
    config: 
     command: /home/ahua/bin/ods_to_dim_db_daily.sh all ${dt}

  - name: ods_to_dwd_log
    type: command
    dependsOn: 
     - hdfs_to_ods_log
    config: 
     command: /home/ahua/bin/ods_to_dwd_log.sh all ${dt}
    
  - name: ods_to_dwd_db_daily
    type: command
    dependsOn: 
     - hdfs_to_ods_db_daily
    config: 
     command: /home/ahua/bin/ods_to_dwd_db_daily.sh all ${dt}
    
  - name: dwd_to_dws_daily
    type: command
    dependsOn:
     - ods_to_dim_db_daily
     - ods_to_dwd_log
     - ods_to_dwd_db_daily
    config:
     command: /home/ahua/bin/dwd_to_dws_daily.sh all ${dt}
    
  - name: dws_to_dwt_daily
    type: command
    dependsOn:
     - dwd_to_dws_daily
    config:
     command: /home/ahua/bin/dws_to_dwt_daily.sh all ${dt}
    
  - name: dwt_to_ads
    type: command
    dependsOn: 
     - dws_to_dwt_daily
    config:
     command: /home/ahua/bin/dwt_to_ads.sh all ${dt}
     
  - name: hdfs_to_mysql
    type: command
    dependsOn:
     - dwt_to_ads
    config:
      command: /home/ahua/bin/hdfs_to_mysql.sh all

```

##### Superset启停脚本superset.sh 3.0

> /home/ahua/bin/superset.sh

```bash
#!/bin/bash

superset_status(){
    result=`ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | wc -l`
    if [[ $result -eq 0 ]]; then
        return 0
    else
        return 1
    fi
}
superset_start(){
        # 该段内容取自~/.bashrc，作用是进行conda初始化
        # >>> conda initialize >>>
        # !! Contents within this block are managed by 'conda init' !!
        __conda_setup="$('/opt/module/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
        if [ $? -eq 0 ]; then
            eval "$__conda_setup"
        else
            if [ -f "/opt/module/miniconda3/etc/profile.d/conda.sh" ]; then
                . "/opt/module/miniconda3/etc/profile.d/conda.sh"
            else
                export PATH="/opt/module/miniconda3/bin:$PATH"
            fi
        fi
        unset __conda_setup
        # <<< conda initialize <<<
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            conda activate superset ; gunicorn --workers 5 --timeout 120 --bind hadoop102:8787 --daemon 'superset.app:create_app()'
        else
            echo "superset 正在运行"
        fi
}

superset_stop(){
    superset_status >/dev/null 2>&1
    if [[ $? -eq 0 ]]; then
        echo "superset 未在运行"
    else
        ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9
    fi
}

case $1 in
    start )
        echo "启动 Superset"
        superset_start
    ;;
    stop )
        echo "停止 Superset"
        superset_stop
    ;;
    restart )
        echo "重启 Superset"
        superset_stop
        superset_start
    ;;
    status )
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            echo "superset 未在运行"
        else
            echo "superset 正在运行"
        fi
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 superset.sh
>
> 启动superset：superset.sh start
>
> 停止superset：superset.sh stop
>
> 重启superset：superset.sh restart
>
> 查看superset状态：superset.sh status

##### Superset启停脚本superset.sh 4.0

> /home/ahua/bin/superset.sh

```shell
#!/bin/bash

superset_status(){
    result=`ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | wc -l`
    if [[ $result -eq 0 ]]; then
        return 0
    else
        return 1
    fi
}
superset_start(){
        source ~/.bashrc
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            conda activate superset ; gunicorn --workers 5 --timeout 120 --bind hadoop102:8787 --daemon 'superset.app:create_app()'
        else
            echo "superset 正在运行"
        fi
}

superset_stop(){
    superset_status >/dev/null 2>&1
    if [[ $? -eq 0 ]]; then
        echo "superset 未在运行"
    else
        ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9
    fi
}

case $1 in
    start )
        echo "启动 Superset"
        superset_start
    ;;
    stop )
        echo "停止 Superset"
        superset_stop
    ;;
    restart )
        echo "重启 Superset"
        superset_stop
        superset_start
    ;;
    status )
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            echo "superset 未在运行"
        else
            echo "superset 正在运行"
        fi
esac
```

> 脚本创建后，要给执行权限
>
> chmod 777 superset.sh
>
> 启动superset：superset.sh start
>
> 停止superset：superset.sh stop
>
> 重启superset：superset.sh restart
>
> 查看superset状态：superset.sh status

##### Kylin cube 自动构建脚本

> /home/ahua/bin/

```bash
#!/bin/bash
cube_name=order_detail_cube
do_date=`date -d '-1 day' +%F`

#获取00:00时间戳
start_date_unix=`date -d "$do_date 08:00:00" +%s`
start_date=$(($start_date_unix*1000))

#获取24:00的时间戳
stop_date=$(($start_date+86400000))

curl -X PUT -H "Authorization: Basic QURNSU46S1lMSU4=" -H 'Content-Type: application/json' -d '{"startTime":'$start_date', "endTime":'$stop_date', "buildType":"BUILD"}' http://hadoop102:7070/kylin/api/cubes/$cube_name/build
```

```
query

curl -X POST -H "Authorization: Basic QURNSU46S1lMSU4=" -H "Content-Type: application/json" -d '{ "sql":"select pro.region_name,sum(od.final_amount_d) from dwd_fact_order_detail od join dwd_dim_base_province pro on od.province_id=pro.id group by pro.region_name;", "project":"gmall" }' http://hadoop102:7070/kylin/api/query


select pro.region_name,sum(od.final_amount_d) from dwd_fact_order_detail od join dwd_dim_base_province pro on od.province_id=pro.id group by pro.region_name;
```



## 配置文件

##### Hadoop配置文件

> /opt/module/hadoop-3.2.2/etc/hadoop/

###### 【core-site.xml】

```xml
<configuration>

	<!-- 指定 NameNode 的地址 -->
	<property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
	</property>

	<!-- 指定 hadoop 数据的存储目录 -->
	<property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.2.2/data</value>
	</property>

	<!-- 配置 HDFS 网页登录使用的静态用户为 ahua -->
	<property>
        <name>hadoop.http.staticuser.user</name>
        <value>ahua</value>
	</property>

	<!-- 配置该 ahua(superUser) 允许通过代理访问的主机节点 -->
	<property>
		<name>hadoop.proxyuser.ahua.hosts</name>
		<value>*</value>
	</property>

	<!-- 配置该 ahua(superUser) 允许通过代理用户所属组 -->
	<property>
		<name>hadoop.proxyuser.ahua.groups</name>
		<value>*</value>
	</property>

	<!-- 配置该 ahua(superUser) 允许通过代理的用户 -->
	<property>
		<name>hadoop.proxyuser.ahua.users</name>
		<value>*</value>
	</property>
    
	<!-- LZO 压缩配置 -->
    <property>
		<name>io.compression.codecs</name>
		<value>
            org.apache.hadoop.io.compress.GzipCodec,
            org.apache.hadoop.io.compress.DefaultCodec,
            org.apache.hadoop.io.compress.BZip2Codec,
            org.apache.hadoop.io.compress.SnappyCodec,
            com.hadoop.compression.lzo.LzoCodec,
            com.hadoop.compression.lzo.LzopCodec
		</value>
	</property>
    
	<property>
		<name>io.compression.codec.lzo.class</name>
		<value>com.hadoop.compression.lzo.LzoCodec</value>
	</property>

</configuration>
```

###### 【hdfs-site.xml】

```xml
<configuration>

	<!-- nn web 端访问地址-->
	<property>
		<name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
	</property>

	<!-- 2nn web 端访问地址-->
    <property>
		<name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
	</property>
	
	<!-- 测试环境指定 HDFS 副本的数量 1 --> 
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>

</configuration>
```

> HDFS副本的数量默认是3，这里为了节省空间，设置为1

###### 【yarn-site.xml】

```xml
<configuration>

	<!--specific YARN configuration properties -->

	<!-- 指定 MR 走 shuffle -->
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
	</property>

	<!-- 指定 ResourceManager 的地址 -->
	<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
	</property>

	<!-- 环境变量的继承 3.2 以上版本不需要配置此项 -->
	<property>
        <name>yarn.nodemanager.env-whitelist</name>
                   <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
	</property>
    
	<!-- yarn 容器允许分配的最大最小内存 默认为1G -->
	<property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
	</property>

	<!-- yarn 容器允许分配的最大最小内存 默认为8G -->
	<property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
	</property>
    
	<!-- yarn 容器允许管理的物理内存大小 默认为8G -->
	<property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
	</property>
    
	<!-- 关闭 yarn 对物理内存和虚拟内存的限制检查 -->
	<property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
	</property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
	</property>
    
	<!-- 开启日志聚集功能 -->
	<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
	</property>

	<!-- 设置日志聚集服务器地址 -->
	<property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop102:19888/jobhistory/logs</value>
	</property>

	<!-- 设置日志保留时间为 7 天 -->
	<property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
	</property>

</configuration>
```

> <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
>
> <property>
>
>   <name>yarn.nodemanager.pmem-check-enabled</name>
>
>   <value>false</value>
>
> </property>
>
> <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
>
> <property>
>
>   <name>yarn.nodemanager.vmem-check-enabled</name>
>
>   <value>false</value>
>
> </property>

###### 【mapred-site.xml】

```xml
<configuration>

	<!-- 指定 MapReduce 程序运行在 Yarn 上 -->
	<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
	</property>

	<!-- 历史服务器端地址 -->
	<property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop102:10020</value>
	</property>

	<!-- 历史服务器 web 端地址 -->
	<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop102:19888</value>
	</property>

</configuration>
```

###### 【workers】

```
hadoop102
hadoop103
hadoop104
```

> 注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行

###### 【capacity-scheduler.xml】

针对容量调度器并发度低的问题，考虑调整yarn.scheduler.capacity.maximum-am-resource-percent该参数。默认值是0.1，表示集群上AM最多可使用的资源比例，目的为限制过多的app数量。

```xml
<property>
    <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
    <value>0.5</value>
    <description>
      集群中用于运行应用程序ApplicationMaster的资源比例上限，该参数通常用于限制处于活动状态的应用程序数目。该参数类型为浮点型，默认是0.1，表示10%。所有队列的ApplicationMaster资源比例上限可通过参数yarn.scheduler.capacity.maximum-am-resource-percent设置，而单个队列可通过参数yarn.scheduler.capacity.<queue-path>.maximum-am-resource-percent设置适合自己的值。
    </description>
</property
```

默认Yarn的配置下，容量调度器只有一条default队列。在capacity-scheduler.xml中可以配置多条队列，**修改**以下属性，增加hive队列。

```xml
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
     再增加一个hive队列
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
	<value>50</value>
    <description>
      default队列的容量为50%
    </description>
</property>
```

同时为新加队列**添加**必要属性：

```xml
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
	<value>50</value>
    <description>
      hive队列的容量为50%
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
	<value>1</value>
    <description>
      一个用户最多能够获取该队列资源容量的比例，取值0-1
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
	<value>80</value>
    <description>
      hive队列的最大容量（自己队列资源不够，可以使用其他队列资源上限）
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
    <description>
      开启hive队列运行，不设置队列不能使用
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
	<value>*</value>
    <description>
      访问控制，控制谁可以将任务提交到该队列,*表示任何人
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
	<value>*</value>
    <description>
      访问控制，控制谁可以管理(包括提交和取消)该队列的任务，*表示任何人
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
	<value>*</value>
	<description>
      指定哪个用户可以提交配置任务优先级
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
	<value>-1</value>
    <description>
      hive队列中任务的最大生命时长，以秒为单位。任何小于或等于零的值将被视为禁用。
	</description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
	<value>-1</value>
    <description>
      hive队列中任务的默认生命时长，以秒为单位。任何小于或等于零的值将被视为禁用。
	</description>
</property>
```

> 设置hive的默认队列为创建的hive队列
>
> hive (default)> set mapreduce.job.queuename;
> mapreduce.job.queuename=default
> hive (default)> set mapreduce.job.queuename=hive;
> hive (default)> set mapreduce.job.queuename;
> mapreduce.job.queuename=hive

##### taildir-kafka.conf

> 保存在hadoop102/103 /opt/module/flume**/jobs/

```shell
# Name the components on this agent
# 每个组件命名
a1.sources = r1
a1.channels = c1

# Describe / Configure the source
# 配置 Source
# Taildir Source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/gmall_log/app_log/log/app.*
a1.sources.r1.positionFile = /opt/module/apache-flume-1.9.0-bin/jobs/taildir_position/taildir_position.json

# Describe / Cconfigure the interceptor
# 配置 Interceptor
# ETL 数据清洗 过滤非 json 数据
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.ahua.gmall.flume.interceptor.LogInterceptor$Builder

# Describe / configure the channel
# 配置 Channel
# Kafka Channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092
a1.channels.c1.kafka.topic = topic_log
a1.channels.c1.parseAsFlumeEvent = false

# Describe / Configure the sink
# 配置 Sink
# 没有

# Bind the source and sink to the channel
# 绑定 source 与 channel 和 sink 与 channel 的关系
a1.sources.r1.channels = c1
```

> 单发给hadoop103，发送到hadoop104也可，不过在hadoop104上并不会用到此文件

##### kafka-file-hdfs.conf

> 保存在hadoop104，保存在hadoop102/103上也可，不过并不会在102/103上用到此文件

```shell
# Name the components on this agent
# 定义组件
a1.sources = r1
a1.channels = c1
a1.sinks = k1

# Describe / Configure the source
# 配置 Source
# Kafka Source
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics = topic_log
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000

# Describe / Cconfigure the interceptor
# 配置 Interceptor
# 时间戳拦截器 解决零点漂移问题
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.ahua.gmall.flume.interceptor.TimeStampInterceptor$Builder

# Describe / configure the channel
# 配置 Channel
# File Channel
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/apache-flume-1.9.0-bin/gmall/checkpoint/behavior/
a1.channels.c1.dataDirs = /opt/module/apache-flume-1.9.0-bin/gmall/data/behavior/
a1.channels.c1.maxFileSize = 2146435071
a1.channels.c1.capacity = 1000000
a1.channels.c1.keep-alive = 6

# Describe / Configure the sink
# 配置 Sink
# HDFS Sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log-
a1.sinks.k1.hdfs.round = false

# 控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

# 控制输出文件格式 使用 lzop 压缩
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = lzop

# 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

> #每5小时sink一次
>
> a1.sinks.k1.type = hdfs
>
> a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d**-%H**
>
> a1.sinks.k1.hdfs.filePrefix = log-
>
> a1.sinks.k1.hdfs.round = true     #false 变为 true
>
> a1.sinks.k1.hdfs.roundValue = 5
>
> a1.sinks.k1.hdfs.roundUnit =  hour



##### Hive配置文件

###### hive-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

	<!-- jdbc 连接的 URL -->
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
	</property>

	<!-- jdbc 连接的 Driver -->
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>

	<!-- jdbc 连接的 username -->
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>

	<!-- jdbc 连接的 password -->
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>ahuamysql</value>
	</property>

	<!-- Hive 元数据存储版本的验证 -->
	<property>
		<name>hive.metastore.schema.verification</name>
		<value>false</value>
	</property>

	<!-- Hive 元数据存储授权 -->
	<property>
		<name>hive.metastore.event.db.notification.api.auth</name>
		<value>false</value>
	</property>

	<!-- Hive 默认在 HDFS 的工作目录 -->
	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/user/hive/warehouse</value>
	</property>

	<!-- 指定存储元数据要连接的地址 -->
<!--    <property>
                <name>hive.metastore.uris</name>
                <value>thrift://hadoop102:9083</value>
        </property>
-->

	<!-- 指定 hiveserver2 连接的 host -->

	<property>
		<name>hive.server2.thrift.bind.host</name>
		<value>hadoop102</value>
	</property>

	<!-- 指定 hiveserver2 连接的端口号 -->

	<property>
		<name>hive.server2.thrift.port</name>
		<value>10000</value>
	</property>
    
    <!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
	<property>
		<name>hive.server2.active.passive.ha.enable</name>
		<value>true</value>
		<description>Whether HiveServer2 Active/Passive High Availability be enabled when Hive Interactive sessions are enabled.This will also require hive.server2.support.dynamic.service.discovery to be enabled.</description>
	</property>
    
    <!-- 打印当前库和表头 -->
	<property>
		<name>hive.cli.print.current.db</name>
		<value>true</value>
	</property>
    
	<property>
		<name>hive.cli.print.header</name>
		<value>true</value>
	</property>
    
    <!-- Spark Spark Spark Spark -->
    
    <!-- Spark 依赖位置（注意：端口号8020必须和 namenode 的端口号一致）-->
	<property>
    	<name>spark.yarn.jars</name>
   		<value>hdfs://hadoop102:8020/spark/spark-jars/*</value>
	</property>
  
	<!-- Hive 执行引擎 -->
	<property>
   		<name>hive.execution.engine</name>
        <value>spark</value>
	</property>

	<!-- Hive 和 Spark 连接超时时间 -->
	<property>
    	<name>hive.spark.client.connect.timeout</name>
    	<value>10000ms</value>
	</property>

</configuration>
```

> 注意：hive.spark.client.connect.timeout的默认值是1000ms，如果执行hive的insert语句时，抛如下异常，可以调大该参数到10000ms
>
> FAILED: SemanticException Failed to get a spark session: org.apache.hadoop.hive.ql.metadata.HiveException: Failed to create Spark client for Spark session d9e0224c-3d14-4bf4-95bc-ee3ec56df48e

###### spark-defaults.conf

> /opt/module/apache-hive-3.1.2-bin/conf/spark-defaults.conf

```shell
spark.master                        yarn
spark.eventLog.enabled              true
spark.eventLog.dir                  hdfs://hadoop102:8020/spark/spark-history
spark.executor.memory               1g
spark.driver.memory					1g
```

##### HBase配置文件

###### conf/hbase-env.sh

```
export JAVA_HOME=/opt/module/jdk1.8.0_271
export HBASE_MANAGES_ZK=false
```

###### conf/hbase-site.xml

```xml
<configuration>
    
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://hadoop102:8020/hbase</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>hadoop102,hadoop103,hadoop104</value>
	</property>
<!-- 2.3版本自带
	<property>
		<name>hbase.tmp.dir</name>
		<value>./tmp</value>
	</property>
	<property>
		<name>hbase.unsafe.stream.capability.enforce</name>
		<value>false</value>
	</property>
-->
</configuration>
```

###### conf/regionservers

```
hadoop102
hadoop103
hadoop104
```

##### Kylin

```sql
hive (default)> use gmall;

hive (gmall)>
create view dim_sku_info_view
as
select
    id,
    price,
    sku_name,
    sku_desc,
    weight,
    is_sale,
    spu_id,
    spu_name,
    category3_id,
    category3_name,
    category2_id,
    category2_name,
    category1_id,
    category1_name,
    tm_id,
    tm_name,
    create_time
from dim_sku_info;
where

hive (gmall)> select * from dim_sku_info_view;
```

```shell
#!/bin/bash
cube_name=order_cube
do_date=`date -d '-1 day' +%F`

#获取00:00时间戳
start_date_unix=`date -d "$do_date 08:00:00" +%s`
start_date=$(($start_date_unix*1000))

#获取24:00的时间戳
stop_date=$(($start_date+86400000))

curl -X PUT -H "Authorization: Basic QURNSU46S1lMSU4=" -H 'Content-Type: application/json' -d '{"startTime":'$start_date', "endTime":'$stop_date', "buildType":"BUILD"}' http://hadoop102:7070/kylin/api/cubes/$cube_name/build
```



# Scala

##### HelloWorld

###### HelloWorld.scala

```scala
/*
   object: 关键字，声明一个单例对象（伴生对象）
 */
object HelloWorld {
  /*
    main 方法：从外部可以直接调用执行的方法
    def 方法名称(参数名称: 参数类型): 返回值类型 = { 方法体 }
   */
  def main(args: Array[String]): Unit = {
    println("hello world")
    System.out.println("hello scala from java")
  }
```

> 产生两个.class字节码文件，伴生类（HelloWorld.class），伴生对象的所属类（HelloWorld$.class）

###### **Student.java**

```java
public class Student {
    private String name;
    private Integer age;
    private static String school = "CQUPT";

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public void printInfo() {
        System.out.println(this.name + " " + this.age + " " + Student.school);
    }

    public static void main(String[] args) {
        Student alice = new Student("alice", 20);
        Student bob = new Student("bob", 23);
        alice.printInfo();
        bob.printInfo();
    }
}
```

###### **Student.scala**

```scala
class Student(name: String, var age: Int) {
  def printInfo(): Unit = {
    println(name + " " + age + " " + Student.school)
  }
}

// 引入伴生对象
object Student{
  val school: String = "CQUPT"

  def main(args: Array[String]): Unit = {
    val alice = new Student("alice", 20)
    val bob = new Student("bob", 23)

    alice.printInfo()
    bob.printInfo()
  }
}
```

> Java中通过类名Student调用了静态的属性school，也即通过类来调用了其属性，而这似乎看起来并不是真正的面向对象，若是真正的面向对象，就应该也是通过对象来调用所有属性。
>
> 于是，为了解决这个问题，在Scala中就出现了伴生对象，即Scala中的`object Student`是`class Student`的伴生对象。
>
> 如果再将全局只有一份的属性，定义为伴生对象里的一个属性，那么就可以通过伴生对象的实例来调用这个属性，从而避免了Java中的类名.属性的方式来调用属性。
>
> 目前有种不知是否正确的看法——其实就很像是把原Java类`XXX`中的静态属性、静态方法单独拧出来将其放入伴生对象`object XXX`中，而自身就成为了伴生类？

##### 变量和数据类型

###### Test01_Comment.scala

> 【注释】Scala 注释使用和  Java 完全一样

```scala
/*
   这是一个简单的测试程序
   测试注释
 */
object Test01_Comment {
  /**
   * 程序的入口方法
   * @param args 外部传入的参数
   */
  def main(args: Array[String]): Unit = {
    // 打印输出
    println("hello")
  }
}
```

###### Test02_Variable.scala

> 【变量和常量】

```scala
object Test02_Variable {
  def main(args: Array[String]): Unit = {
    // 声明一个变量的通用语法
    var a: Int = 10

    //（1）声明变量时，类型可以省略，编译器自动推导，即类型推导
    var a1 = 10
    val b1 = 23

    //（2）类型确定后，就不能修改，说明Scala是强数据类型语言
    var a2 = 15    // a2类型为Int
//    a2 = "hello" // 不允许

    //（3）变量声明时，必须要有初始值
//    var a3: Int  // 不允许

    //（4）在声明/定义一个变量时，可以使用var或者val来修饰，var修饰的变量可改变，val修饰的变量不可改
    a1 = 12
//    b1 = 25 // 在（1）中已经定义了b1为常量23，于是不能再赋值，即便再赋值23也不允许

    var alice = new Student("alice", 20)
    alice = new Student("Alice", 20)
    alice = null
    val bob = new Student("bob", 23)
    bob.age = 24 // class Student(name: String, var age: Int) {}
    bob.printInfo()
//    bob = new Student("bob", 24) // 可对属性进行更改，但是自身不能改变
  }
}
```

> 常量：在程序执行的过程中，其值不会被改变的变量
> 0）回顾 Java 变量 和常量语法
> 变量类型 变量名称 = 初始值           int a = 10
> final 常量类型 常量名称 = 初始值  final int b = 20
> 1）基本语法
> **var** 变量名 [: 变量类型] = 初始值     var i:Int = 10
> **val** 常量名 [: 常量类型]  = 初始值     val j:Int = 20
>
> 注意：能用常量的地方不用变量
>
> var 修饰的对象引用可以改变，val 修饰的对象则不可改变（内存地址不能变），但对象的状态（值）却是可以改变的。（比如：自定义对象、数组、集合等等）

###### Test03_Identifier

> 【标识符】

```scala
object Test03_Identifier {
  def main(args: Array[String]): Unit = {
    //（1）以字母或者下划线开头，后接字母、数字、下划线
    val hello: String = ""
    var Hello123 = ""
    val _abc = 123

//    val h-b = "" // 不允许
//    val 123abc = 234 // 不允许

    //（2）以操作符开头，且只包含操作符（+ - * / # !等）
    val -+/% = "hello"
    println(-+/%)

    //（3）用反引号`....`包括的任意字符串，即使是Scala关键字（39个）也可以
    val `if` = "if"
    println(`if`)
  }
}
```

> Scala（ 39 个）关键字
>
> • package, import, class, **object** , **trait** , extends, **with** , type, for
> • private, protected, abstract, **sealed** , final, **implicit** , lazy, override
> • try, catch, finally, throw
> • if, else, **match** , case, do, while, for, return, **yield**
> • **def**, **val**, **var**
> • this, super
> • new
> • true, false, null

###### Test04_String.scala

> 【字符串输出】

```scala
object Test04_String {
  def main(args: Array[String]): Unit = {
    //（1）字符串，通过+号连接
    val name: String = "alice"
    val age: Int = 18
    println(age + "岁的" + name + "学习Scala")

    // *用于将一个字符串复制多次并拼接
    println(name * 3)

    //（2）printf用法：字符串，通过%传值
    printf("%d岁的%s学习Scala", age, name)
    println()

    //（3）字符串模板（插值字符串）：通过$获取变量值
    println(s"${age}岁的${name}学习Scala")

    val num: Double = 2.3456
    println(f"The num is ${num}%2.2f")    // 格式化模板字符串//**is 2.35
    println(raw"The num is ${num}%2.2f")	// The num is 2.3456%2.2f

    // 三引号表示字符串，保持多行字符串的原格式输出
    val sql = s"""
       |select *
       |from
       |  student
       |where
       |  name = ${name}
       |and
       |  age > ${age}
       |""".stripMargin
    println(sql)
  }
}
```

###### Test05_StdIn.scala

> 【键盘输入】

```scala
import scala.io.StdIn

object Test05_StdIn {
  def main(args: Array[String]): Unit = {
    // 输入信息
    println("请输入您的大名：")
    val name: String = StdIn.readLine()
    println("请输入您的芳龄：")
    val age: Int = StdIn.readInt()

    // 控制台打印输出
    println(s"${age}岁的${name}学习Scala")
  }
}
```

###### Test06_FileIO.scala

> 【读取写入文件】

```scala
import java.io.{File, PrintWriter}
import scala.io.Source

object Test06_FileIO {
  def main(args: Array[String]): Unit = {
    // 1. 从文件中读取数据
    Source.fromFile("src/main/resources/test.txt").foreach(print)

    // 2. 将数据写入文件
    val writer = new PrintWriter(new File("src/main/resources/output.txt"))
    writer.write("hello scala from java writer")
    writer.close()
  }
}
```

###### Test07_DataType.scala

> 【数据类型】整数类型、浮点类型、字符类型、布尔类型、Unit类型、Null类型和Noting类型

| 数据类型     | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| Byte [1]     | 8位有符号补码整数。数值区间为 -128 到 127                    |
| Short [2]    | 16位有符号补码整数。数值区间为 -32768 到 32767               |
| Int [4]      | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
| Long [8]     | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 = 2的(64-1)次方-1 |
| Float [4]    | 32位，IEEE 754标准的单精度浮点数                             |
| Double [8]   | 64位，IEEE 754标准的双精度浮点数                             |
| Unit[空值]   | 表示无值，和其他语言中void等同，用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成() |
| Null[空引用] | null , Null类型只有一个实例值null                            |
| Noting       | Nothing类型在Scala的类层级最低端，它是任何其他类型的子类型。 当一个函数，我们确定没有正常的返回值，可以用Nothing来指定返回类型，这样有一个好处，就是我们可以把返回的值（异常）赋给其它的函数或者变量（兼容性） |

```scala
import chapter01.Student

object Test07_DataType {
  def main(args: Array[String]): Unit = {
    // 1. 整数类型
    val a1: Byte = 127
    val a2: Byte = -128

//    val a2: Byte = 128    // error

    val a3 = 12    // 整数默认类型为Int
    val a4: Long = 1324135436436L	// 长整型数值定义
//    val a4 = 1324135436436	// error
//    val a4: Long = 1324135436436	// error

    val b1: Byte = 10
    val b2: Byte = 10 + 20
    println(b2)

//    val b3: Byte = b1 + 20
    val b3: Byte = (b1 + 20).toByte
    println(b3)

    // 2. 浮点类型
    val f1: Float = 1.2345f
    val d1 = 34.2245	//默认为double类型

    // 3. 字符类型
    val c1: Char = 'a'
    println(c1)

    val c2: Char = '9'
    println(c2)

    // 控制字符
    val c3: Char = '\t'    // 制表符
    val c4: Char = '\n'    // 换行符
    println("abc" + c3 + "def")
    println("abc" + c4 + "def")

    // 转义字符
    val c5 = '\\'    // 表示\自身
    val c6 = '\"'    // 表示"
    println("abc" + c5 + "def")
    println("abc" + c6 + "def")

    // 字符变量底层保存ASCII码
    val i1: Int = c1
    println("i1: " + i1)	// 97
    val i2: Int = c2
    println("i2: " + i2)	// 57

    val c7: Char = (i1 + 1).toChar
    println(c7)	// b
    val c8: Char = (i2 - 1).toChar
    println(c8)	// 8

    // 4. 布尔类型 占1个字节
    val isTrue: Boolean = true
    println(isTrue)

    // 5. 空类型
    // 5.1 空值Unit
    def m1(): Unit = {
      println("m1被调用执行")
    }

    val a: Unit = m1()
    println("a: " + a)
    // 输出
    // m1被调用执行
    // a: ()

    // 5.2 空引用Null
//    val n: Int = null    // error
    var student: Student = new Student("alice", 20)
    student = null
    println(student)	// null

    // 5.3 Nothing  //Nothing是Int类（任何类）的子类
    def m2(n: Int): Int = {
      if (n == 0)
        throw new NullPointerException
      else
        return n
    }

    val b: Int = m2(2)
    println("b: " + b)  // b: 2
  }
}
```

###### Test08_DataTypeConversion.scala

> 【类型转换】自动类型转换、强制类型转换、数值类型和String类型的转换

```scala
object Test08_DataTypeConversion {
  def main(args: Array[String]): Unit = {

    // 1. 自动类型转换
    //（1）自动提升原则：有多种类型的数据混合运算时，系统首先自动将所有数据转换成精度大的那种数据类型，然后再进行计算。
    val a1: Byte = 10
    val b1: Long = 2353
    val result1: Long = a1 + b1
    val result11: Int = (a1 + b1.toInt) // 强转

    //（2）把精度大的数值类型赋值给精度小的数值类型时，就会报错，反之就会进行自动类型转换。
    val a2: Byte = 10
    val b2: Int = a2
    //    val c2: Byte = b2    // error

    //（3）（byte，short）和char之间不会相互自动转换。
    val a3: Byte = 10
    val b3: Char = 'b'
    //    val c3: Byte = b3    // error
    val c3: Int = b3
    println(c3)

    //（4）byte，short，char他们三者可以计算，在计算时首先转换为int类型。
    val a4: Byte = 12
    val b4: Short = 25
    val c4: Char = 'c'
    val result4: Int = a4 + b4
    val result44: Int = a4 + b4 + c4
    println(result44)

    // 2. 强制类型转换
    //（1）将数据由高精度转换为低精度，就需要使用到强制转换
    val n1: Int = -2.9.toInt
    println("n1: " + n1)	// 2.9.toInt -> n1: 2	// -2.9.toInt -> n1: -2

    //（2）强转符号只针对于最近的操作数有效，往往会使用小括号提升优先级
    val n2: Int = 2.6.toInt + 3.7.toInt
    val n3: Int = (2.6 + 3.7).toInt
    println("n2: " + n2)	// n2: 5
    println("n3: " + n3)	// n3: 6

    // 3. 数值类型和String类型的转换
    //（1）数值转String
    val n: Int = 27
    val s: String = n + ""
    println(s)	// 27

    //（2）String转数值
    val m: Int = "12".toInt
    val f: Float = "12.3".toFloat
    val f2: Int = "12.3".toDouble.toInt
    // val f2: Int = "12.3".toInt会出现NumberFormatException异常
    println(f2)	// 12
  }
}
```

###### Test09_Problem_DataTypeConversion.scala

> 【一个题目】

```scala
/*
128: Int类型，占据4个字节，32位
原码 0000 0000 0000 0000 0000 0000 1000 0000
补码 0000 0000 0000 0000 0000 0000 1000 0000

截取最后一个字节，Byte
得到补码 1000 0000
表示最大负数 -128

130: Int类型，占据4个字节，32位
原码 0000 0000 0000 0000 0000 0000 1000 0010
补码 0000 0000 0000 0000 0000 0000 1000 0010

截取最后一个字节，Byte
得到补码 1000 0010
对应原码 1111 1110
-126
 */

object Test09_Problem_DataTypeConversion {
  def main(args: Array[String]): Unit = {
    var n: Int = 130
    val b: Byte = n.toByte
    println(b)
  }
}
```

##### 运算符

###### TestOperator.java

> 【Java中的运算符】

```java
public class TestOperator {
    public static void main(String[] args) {
        // 比较运算符
        String s1 = "hello";
        String s2 = new String("hello");

        Boolean isEqual = s1 == s2;
        System.out.println(isEqual);	// false
        System.out.println(s1.equals(s2));	// true

        System.out.println("========================");

        // 赋值运算符
        byte b = 10;
        b = (byte)(b + 1);
        b += 1;    // 默认会做强转
        System.out.println(b);	// 12

        // 自增自减
        int x = 15;
        int y = x++;
        System.out.println("x = " + x + ", y = " + y);	// 16 15

        x = 15;
        y = ++x;
        System.out.println("x = " + x + ", y = " + y);	// 16 16

        x = 23;
        x = x++;
        System.out.println(x);	// 23	// temp = x++; x = temp;
    }
}
```

###### Test01_TestOperator.scala

【运算符】算术运算符、比较运算符、逻辑运算符、赋值运算符、位运算符、运算符的本质

```scala
object Test01_TestOperator {
  def main(args: Array[String]): Unit = {
    // 1. 算术运算符
    val result1: Int = 10 / 3
    println(result1)	// 3

    val result2: Double = 10 / 3
    println(result2)	// 3.0

    val result3: Double = 10.0 / 3
    println(result3.formatted("%5.2f"))	// _3.33	// 此处至少要有5位，少于5位要在前添加空格

    val result4: Int = 10 % 3
    println(result4)	// 1

    // 2. 比较运算符
    val s1: String = "hello"
    val s2: String = new String("hello")

    println(s1 == s2)	// true
    println(s1.equals(s2))	// true
    println(s1.eq(s2))	// false	// 判断s1,s2的引用是否相等

    println("===================")

    // 3. 逻辑运算符
    def m(n: Int): Int = {
      println("m被调用")
      return n
    }

    val n = 1
    println((4 > 5) && m(n) > 0)	// false	// 不会输出"m被调用"

    // 判断一个字符串是否为空
    def isNotEmpty(str: String): Boolean = {
      return str != null && !("".equals(str.trim))
    }

    println(isNotEmpty(null))	// false

    // 4. 赋值运算符
//    var b: Byte = 10
//    b += 1	// error
    var i: Int = 12
    i += 1
    println(i)

//    i++

    // 5. 位运算符
    val a: Byte = 60
    println(a << 3)	// 480
    println(a >> 2)	// 15

    val b: Short = -13
    println(b << 2)	// -52
    println(b >> 2)	// -4
    println(b >>> 2)	// 1073741820

    // 6. 运算符的本质
    val n1: Int = 12
    val n2: Int = 37

    println(n1.+(n2))	// 49
    println(n1 + n2)	// 49

    println(1.34.*(25))	// 33.5
    println(1.34 * 25)	// 33.5

//    println(7.5 toInt toString)	// 这个要报错
    println(7.5 toInt)	// 7
    println(7.5.toInt.toFloat.toString)	// 7.0
  }
}
```

> Scala删掉了Java中的自增自减，即Scala中没有++、--操作符，可以通过+=、-=来实现同样的效果
>
> val b: Short = -13
>
> -13原码：1000 0000 0000 1101
>
> -13补码：1111 1111 1111 0011
>
> -13 << 2后补码：1111 1111 1100 1100 （左移右边补0）
>
> 再还原为原码：1000 0000 0011 0100 表示 -52
>
> -13 >> 2后补码：1111 1111 1111 1100 （右移左边补1）
>
> 再还原为原码：1000 0000 0000 0100 表示 -4
>
> -13 >>> 2后补码：00111111 11111111 11111111 11111100 不知道这里为什么是32位
>
> 再还原为原码：00111111 11111111 11111111 11111100 表示2 ^ 31 - 4 = 1073741820

##### 流程控制

###### Test01_IfElse.scala

【分支控制if-else】单分支、双分支、多分支、分支语句的返回值、嵌套分支

```scala
import scala.io.StdIn

object Test01_IfElse {
  def main(args: Array[String]): Unit = {
    println("请输入您的年龄：")
    val age: Int = StdIn.readInt()

    // 1. 单分支
    if (age >= 18) {
      println("成年")
    }
    println("===================")

    // 2. 双分支
    if (age >= 18) {
      println("成年")
    } else {
      println("未成年")
    }
    println("===================")

    // 3. 多分支
    if (age <= 6) {
      println("童年")
    } else if (age < 18) {
      println("青少年")
    } else if (age < 35) {
      println("青年")
    } else if (age < 60) {
      println("中年")
    } else {
      println("老年")
    }
    println("===================")

    // 4. 分支语句的返回值
    val result: Any = if (age <= 6) {
      println("童年")
      "童年"
    } else if (age < 18) {
      println("青少年")
      "青少年"
    } else if (age < 35) {
      println("青年")
      age
    } else if (age < 60) {
      println("中年")
      age
    } else {
      println("老年")
      age
    }
    println("result: " + result)

    // java中三元运算符 String res = (age >= 18)?"成年":"未成年"
    val res: String = if (age >= 18) {
      "成年"
    } else {
      "未成年"
    }

    val res2 = if (age >= 18) "成年" else "未成年"
    println("===================")

    // 5. 嵌套分支
    if (age >= 18) {
      println("成年")
      if (age >= 35) {
        if (age >= 60) {
          println("老年")
        } else {
          println("中年")
        }
      } else {
        println("青年")
      }
    } else {
      println("未成年")
      if (age <= 6) {
        println("童年")
      } else {
        println("青少年")
      }
    }
  }
}
```

###### Test02_ForLoop.scala

> 【循环控制for】范围遍历（to、until）、循环守卫、循环步长（by）、嵌套循环、循环引入变量、循环返回值

```scala
import scala.collection.immutable

object Test02_ForLoop {
  def main(args: Array[String]): Unit = {
    // java for语法： for(int i = 0; i < 10; i++){ System.out.println(i + ". hello world") }

    // 1. 范围遍历
    for (i <- 1 to 10) {
      println(i + ". hello world")
    }
    for (i: Int <- 1.to(10)) {
      println(i + ". hello world")
    }

    println("======================")

    //    for (i <- Range(1, 10)){
    //      println(i + ". hello world")	// 输出9次
    //    }
    for (i <- 1 until 10) {
      println(i + ". hello world")	// 输出9次
    }

    println("======================")

    // 2. 集合遍历
    for (i <- Array(12, 34, 53)) {
      println(i)
    }
    for (i <- List(12, 34, 53)) {
      println(i)
    }
    for (i <- Set(12, 34, 53)) {
      println(i)
    }

    println("======================")

    // 3. 循环守卫
    for (i <- 1 to 10) {
      if (i != 5) {
        println(i)
      }
    }

    for (i <- 1 to 10 if i != 5) {
      println(i)
    }

    println("======================")

    // 4. 循环步长
    for (i <- 1 to 10 by 2) {
      println(i)	// 1 3 5 7 9
    }
    println("-------------------")
    for (i <- 13 to 30 by 3) {
      println(i)
    }

    println("-------------------")
    for (i <- 30 to 13 by -2) {
      println(i)
    }
    for (i <- 10 to 1 by -1) {
      println(i)
    }
    println("-------------------")
    for (i <- 1 to 10 reverse) {
      println(i)
    }
    println("-------------------")
    //    for (i <- 30 to 13 by 0){
    //      println(i)
    //    }    // error，step不能为0

    for (data <- 1.0 to 10.0 by 0.3) {	// to 过期了，有可能精度缺失
      println(data)
    }

    println("======================")

    // 5. 循环嵌套
    for (i <- 1 to 3) {
      for (j <- 1 to 3) {
        println("i = " + i + ", j = " + j)
      }
    }
    println("-------------------")
    for (i <- 1 to 4; j <- 1 to 5) {
      println("i = " + i + ", j = " + j)
    }

    println("======================")

    // 6. 循环引入变量
    for (i <- 1 to 10) {
      val j = 10 - i
      println("i = " + i + ", j = " + j)
    }

    for (i <- 1 to 10; j = 10 - i) {
      println("i = " + i + ", j = " + j)
    }

    for {
      i <- 1 to 10
      j = 10 - i
    } {
      println("i = " + i + ", j = " + j)
    }

    println("======================")

    // 7. 循环返回值 默认返回空() 开发中很少使用
    val a = for (i <- 1 to 10) {
      i
    }
    println("a = " + a)	// a = ()

    val b: immutable.IndexedSeq[Int] = for (i <- 1 to 10) yield i * i
    println("b = " + b)	// b = Vector(1, 4, 9, 16, 25, 36, 49, 64, 81, 100)
  }
}
```

###### Test03_Practice_MulTable.scala

> 【练习】九九乘法表

```scala
// 输出九九乘法表
object Test03_Practice_MulTable {
  def main(args: Array[String]): Unit = {
    for (i <- 1 to 9) {
      for (j <- 1 to i) {
        print(s"$j * $i = ${i * j} \t")
      }
      println()
    }

    // 简写
    for (i <- 1 to 9; j <- 1 to i) {
      print(s"$j * $i = ${i * j} \t")
      if (j == i) println()
    }
  }
}
```

###### Test04_Practice_Pyramid.scala

> 【练习】九层妖塔

```scala
// 打印输出一个九层妖塔
object Test04_Practice_Pyramid {
  def main(args: Array[String]): Unit = {
    for (i <- 1 to 9) {
      val stars = 2 * i - 1
      val spaces = 9 - i
      println(" " * spaces + "*" * stars)
    }

    for (i <- 1 to 9; stars = 2 * i - 1; spaces = 9 - i) {
      println(" " * spaces + "*" * stars)
    }

    for (stars <- 1 to 17 by 2; spaces = (17 - stars) / 2) {
      println(" " * spaces + "*" * stars)
    }
  }
}
```

###### Test05_WhileLoop.scala

> 【循环控制while，do...while】不推荐使用

```scala
object Test05_WhileLoop {
  def main(args: Array[String]): Unit = {
    // while
    var a: Int = 10
    while (a >= 1) {
      println("this is a while loop: " + a)
      a -= 1
    }

    var b: Int = 0
    do {
      println("this is a do-while loop: " + b)
      b -= 1
    } while (b > 0)
  }
}
```

> 说明：
> （1）循环条件是返回一个布尔值的表达式
> （2）while 循环是先判断再执行语句
> （3）与 for 语句不同，while 语句没有返回值 ，即整个 while 语句的结果是 Unit 类型()
> （4）因为 while 中没有返回值，所以当要用该语句来计算并返回结果时，就不可避免的使用变量，而变量需要声明在 while 循环的外部，那么就等同于循环的内部对外部的变量造成了影响，所以不推荐使用，而是推荐使用 for 循环 。

###### Test06_Break.scala

> 【循环中断】

TestBreak.java

```java
public class TestBreak {
    public static void main(String[] args) {
        try {
            for (int i = 0; i < 5; i++) {
                if (i == 3)
//                    break;
                    throw new RuntimeException();
                System.out.println(i);
            }
        } catch (Exception e) {
            // 什么都不做，只是退出循环
        }
        System.out.println("这是循环外的代码");
    }
}
```

```scala
import scala.util.control.Breaks
import scala.util.control.Breaks._

object Test06_Break {
  def main(args: Array[String]): Unit = {
    // 1. 采用抛出异常的方式，退出循环
    try {
      for (i <- 0 until 5) {
        if (i == 3)
          throw new RuntimeException
        println(i)
      }
    } catch {
      case e: Exception => // 什么都不做，只是退出循环
    }

    // 2. 使用Scala中的Breaks类的break方法，实现异常的抛出和捕捉
    Breaks.breakable(
      for (i <- 0 until 5) {
        if (i == 3)
          Breaks.break()
        println(i)
      }
    )

    // import scala.util.control.Breaks._
    breakable(
      for (i <- 0 until 5) {
        if (i == 3)
          break()
        println(i)
      }
    )

    println("这是循环外的代码")
  }
}
```

> Scala 内置控制结构特地去掉了 Java 中的 break 和 continue 关键字，是为了更好的适应函数式编程，推荐使用函数式的风格实现 break 和 continue 的功能，而不是一个关键字。Scala 中使用 breakable
> 控制结构来实现 break 和 continue 功能。

##### 函数式编程

> 1）面向对象编程
> 解决问题，分解对象，行为，属性，然后通过对象的关系以及行为的调用来解决问题。
> 对象：用户
> 行为：登录 、连接 JDBC 、读取数据库
> 属性：用户名、密码
>
> Scala 语言是一个**完全面向对象编程**语言，万物皆对象。
> 对象的本质：对数据和行为的一个封装
>
> 2）函数式编程
> 解决问题时，将问题分解成一个一个的步骤，将每个步骤进行封装（函数），通过调用
> 这些封装好的步骤，解决问题。
> 例如：请求 -> 用户名、密码 -> 连接 JDBC -> 读取数据库
>
> Scala 语言是一个**完全函数式编程**语言，万物皆函数。
> 函数的本质：函数可以当做一个值进行传递
>
> 3）在 Scala 中函数式编程和面向对象编程完美融合在一起了。

###### Test01_FunctionAndMethod.scala

> 【函数和方法的区别】
>
> （1）为完成某一功能的程序语句的集合，称为函数
> （2）类中的函数称之方法

```scala
object Test01_FunctionAndMethod {
  def main(args: Array[String]): Unit = {
    // 定义函数
    def sayHi(name: String): Unit = {
      println("hi, " + name)
    }

    // 调用函数
    sayHi("alice")	// hi, alice

    // 调用对象方法
    Test01_FunctionAndMethod.sayHi("bob")	// Hi, bob

    // 获取方法返回值
    val result = Test01_FunctionAndMethod.sayHello("cary")
    println(result)
  }

  // 定义对象的方法
  def sayHi(name: String): Unit = {
    println("Hi, " + name)
  }

  def sayHello(name: String): String = {
    println("Hello, " + name)
    return "Hello"
  }
}
```

> ```scala
> hi, alice
> Hi, bob
> Hello, cary
> Hello
> ```
>
> （1）Scala 语言可以在任何的语法结构中声明任何的语法
> （2）函数没有重载和重写的概念；方法可以进行重载和重写
> （3）Scala 中函数可以嵌套定义

###### Test02_FunctionDefine.scala

> 【函数定义】
>
> （1）函数 1 ：无参，无返回值
>
> （2）函数 2 ：无参，有返回值
>
> （3）函数 3 ：有参，无返回值
>
> （4）函数 4 ：有参，有返回值
>
> （5）函数 5 ：多参，无返回值
>
> （6）函数 6 ：多参，有返回值

```scala
object Test02_FunctionDefine {
  def main(args: Array[String]): Unit = {
    //    （1）函数1：无参，无返回值
    def f1(): Unit = {
      println("1. 无参，无返回值")
    }

    f1()
    println(f1())

    println("=========================")

    //    （2）函数2：无参，有返回值
    def f2(): Int = {
      println("2. 无参，有返回值")
      return 12
    }

    println(f2())

    println("=========================")

    //    （3）函数3：有参，无返回值
    def f3(name: String): Unit = {
      println("3：有参，无返回值 " + name)
    }

    println(f3("alice"))

    println("=========================")

    //    （4）函数4：有参，有返回值
    def f4(name: String): String = {
      println("4：有参，有返回值 " + name)
      return "hi, " + name
    }

    println(f4("alice"))

    println("=========================")

    //    （5）函数5：多参，无返回值
    def f5(name1: String, name2: String): Unit = {
      println("5：多参，无返回值")
      println(s"${name1}和${name2}都是我的好朋友")
    }

    f5("alice", "bob")

    println("=========================")

    //    （6）函数6：多参，有返回值
    def f6(a: Int, b: Int): Int = {
      println("6：多参，有返回值")
      return a + b
    }

    println(f6(12, 37))
  }
}
```

> ```scala
> 1. 无参，无返回值
> 1. 无参，无返回值
> ()
> =========================
> 2. 无参，有返回值
> 12
> =========================
> 3：有参，无返回值 alice
> ()
> =========================
> 4：有参，有返回值 alice
> hi, alice
> =========================
> 5：多参，无返回值
> alice和bob都是我的好朋友
> =========================
> 6：多参，有返回值
> 49
> ```

###### Test03_FunctionParameter.scala

> 【函数参数】
>
> （1）可变参数
> （2）如果参数列表中存在多个参数，那么可变参数一般放置在最后
> （3）参数默认值，一般将有默认值的参数放置在参数列表的后面
> （4）带名参数

```scala
object Test03_FunctionParameter {
  def main(args: Array[String]): Unit = {
    //    （1）可变参数
    def f1(str: String*): Unit = {
      println(str)
    }

    f1("alice")
    f1("aaa", "bbb", "ccc")

    //    （2）如果参数列表中存在多个参数，那么可变参数一般放置在最后
    def f2(str1: String, str2: String*): Unit = {
      println("str1: " + str1 + " str2: " + str2)
    }

    f2("alice")
    f2("aaa", "bbb", "ccc")

    //    （3）参数默认值，一般将有默认值的参数放置在参数列表的后面
    def f3(name: String = "CQUPT"): Unit = {
      println("My school is " + name)
    }

    f3("school")
    f3()

    //    （4）带名参数
    def f4(name: String = "ahua", age: Int): Unit = {
      println(s"${age}岁的${name}学习Scala")
    }

    f4("alice", 20)
    f4(age = 21, name = "bob")
    f4(age = 23)
  }
}
```

> ```scala
> WrappedArray(alice)
> WrappedArray(aaa, bbb, ccc)
> str1: alice str2: WrappedArray()
> str1: aaa str2: WrappedArray(bbb, ccc)
> My school is school
> My school is CQUPT
> 20岁的alice学习Scala
> 21岁的bob学习Scala
> 23岁的ahua学习Scala
> ```

###### Test04_Simplify.scala

> 【函数至简原则（重点）】
>
> 函数至简原则：能省则省
> （1）return 可以省略， Scala 会使用函数体的最后一行代码作为返回值
> （2）如果函数体只有一行代码，可以省略花括号
> （3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略）
> （4）如果有 return ，则不能省略返回值类型，必须指定
> （5）如果函数明确声明 unit ，那么即使函数体中使用 return 关键字也不起作用
> （6）Scala 如果期望是无返回值类型，可以省略等号
> （7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加
> （8）如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略
> （9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略

```scala
// 函数至简原则
object Test04_Simplify {
  def main(args: Array[String]): Unit = {

    def f0(name: String): String = {
      return name
    }

    println(f0("ahua"))

    println("==========================")

    //    （1）return可以省略，Scala会使用函数体的最后一行代码作为返回值
    def f1(name: String): String = {
      name
    }

    println(f1("ahua"))

    println("==========================")

    //    （2）如果函数体只有一行代码，可以省略花括号
    def f2(name: String): String = name

    println(f2("ahua"))

    println("==========================")

    //    （3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略）
    def f3(name: String) = name

    println(f3("ahua"))

    println("==========================")

    //    （4）如果有return，则不能省略返回值类型，必须指定
    //    def f4(name: String) = {
    //      return name
    //    }
    //
    //    println(f4("ahua"))

    println("==========================")

    //    （5）如果函数明确声明unit，那么即使函数体中使用return关键字也不起作用
    def f5(name: String): Unit = {
      return name
    }

    println(f5("ahua"))	// ()

    println("==========================")

    //    （6）Scala如果期望是无返回值类型，可以省略等号
    def f6(name: String) {
      println(name)
    }

    println(f6("ahua"))
    // 输出
    // ahua
    // ()

    println("==========================")

    //    （7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加
    def f7(): Unit = {
      println("ahua")
    }

    f7()
    f7
    // 输出
    // ahua
    // ahua

    println("==========================")

    //    （8）如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略
    def f8: Unit = {
      println("atguigu")
    }

    //    f8()	// error
    f8

    println("==========================")

    //    （9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略
    def f9(name: String): Unit = {
      println(name)
    }

    // 匿名函数，lambda表达式
    (name: String) => {
      println(name)
    }

      println("==========================")

  }
}
```

###### Test05_Lambda.scala

> 【匿名函数】

```scala
object Test05_Lambda {
  def main(args: Array[String]): Unit = {
    val fun = (name: String) => {
      println(name)
    }
    fun("ahua")

    println("========================")

    // 定义一个函数，以函数作为参数输入
    def f(func: String => Unit): Unit = {
      func("ahua")
    }

    f(fun)
    f((name: String) => {
      println(name)
    })

    println("========================")

    // 匿名函数的简化原则
    //    （1）参数的类型可以省略，会根据形参进行自动的推导
    f((name) => {
      println(name)
    })

    //    （2）类型省略之后，发现只有一个参数，则圆括号可以省略；其他情况：没有参数和参数超过1的永远不能省略圆括号
    f(name => {
      println(name)
    })

    //    （3）匿名函数如果只有一行，则大括号也可以省略
    f(name => println(name))

    //    （4）如果参数只出现一次，则参数省略且后面参数可以用_代替
    f(println(_))

    //     (5) 如果可以推断出，当前传入的println是一个函数体，而不是调用语句，可以直接省略下划线
    f(println)

    println("=========================")

    // 实际示例，定义一个“二元运算”函数，只操作1和2两个数，但是具体运算通过参数传入
    def dualFunctionOneAndTwo(fun: (Int, Int) => Int): Int = {
      fun(1, 2)
    }

    val add = (a: Int, b: Int) => a + b
    val minus = (a: Int, b: Int) => a - b

    println(dualFunctionOneAndTwo(add))
    println(dualFunctionOneAndTwo(minus))

    // 匿名函数简化
    println(dualFunctionOneAndTwo((a: Int, b: Int) => a + b))
    println(dualFunctionOneAndTwo((a: Int, b: Int) => a - b))

    println(dualFunctionOneAndTwo((a, b) => a + b))
    println(dualFunctionOneAndTwo(_ + _))
    println(dualFunctionOneAndTwo(_ - _))

    println(dualFunctionOneAndTwo((a, b) => b - a))
    println(dualFunctionOneAndTwo(-_ + _))
  }
}
```

###### Test06_HighOrderFunction.scala

> 【高阶函数】
>
> （1）函数可以作为值进行传递
>
> （1）函数可以作为参数进行传递
>
> （3）函数可以作为函数返回值返回

```scala
object Test06_HighOrderFunction {
  def main(args: Array[String]): Unit = {
    def f(n: Int): Int = {
      println("f调用")
      n + 1
    }

    def fun(): Int = {
      println("fun调用")
      1
    }

    val result: Int = f(123)
    println(result)

    println("=========================")

    // 1. 函数作为值进行传递
    val f1: Int => Int = f
    val f2 = f _

    println(f1)
    println(f1(12))
    println(f2)
    println(f2(35))
    println("-------------------------")

    val f3: () => Int = fun	// val f3 = fun	println(f3)	// 输出 1
    val f4 = fun _
    println(f3)
    println(f4)

    println("=========================")

    // 2. 函数作为参数进行传递
    // 定义二元计算函数
    def dualEval(op: (Int, Int) => Int, a: Int, b: Int): Int = {
      op(a, b)
    }

    def add(a: Int, b: Int): Int = {
      a + b
    }

    println(dualEval(add, 12, 35))
    println(dualEval((a, b) => a + b, 12, 35))
    println(dualEval(_ + _, 12, 35))

	println("=========================")

    // 3. 函数作为函数的返回值返回
    def f5(): Int => Unit = {
      def f6(a: Int): Unit = {
        println("f6调用 " + a)
      }

      f6 // 将函数直接返回
    }

    //    val f6 = f5()
    //    println(f6)
    //    println(f6(25))

    println(f5()(25))
  }
}
```

> ```scala
> f调用
> 124
> =========================
> chapter05.Test06_HighOrderFunction$$$Lambda$5/2096171631@6debcae2
> f调用
> 13
> chapter05.Test06_HighOrderFunction$$$Lambda$6/2114694065@5ba23b66
> f调用
> 36
> -------------------------
> chapter05.Test06_HighOrderFunction$$$Lambda$7/2101440631@35bbe5e8
> chapter05.Test06_HighOrderFunction$$$Lambda$8/2109957412@2c8d66b2
> =========================
> 47
> 47
> 47
> =========================
> f6调用 25
> ()
> ```

###### Test07_Practice_CollectionOperation.scala

> 【练习】

```scala
object Test07_Practice_CollectionOperation {
  def main(args: Array[String]): Unit = {
    val arr: Array[Int] = Array(12, 45, 75, 98)

    // 对数组进行处理，将操作抽象出来，处理完毕之后的结果返回一个新的数组
    def arrayOperation(array: Array[Int], op: Int => Int): Array[Int] = {
      for (elem <- array) yield op(elem)
    }

    // 定义一个加一操作
    def addOne(elem: Int): Int = {
      elem + 1
    }

    // 调用函数
    val newArray: Array[Int] = arrayOperation(arr, addOne)

    println(newArray.mkString(", "))

    // 传入匿名函数，实现元素翻倍
    val newArray2 = arrayOperation(arr, _ * 2)
    println(newArray2.mkString(", "))
  }
}
```

> ```
> 13,46,76,99
> 24,90,150,196
> ```

###### Test08_Practice.scala

> 【练习】
>
> 练习1：
>
> 定义一个匿名函数，并将它作为值赋给变量 fun 。
>
> 函数有三个参数，类型分别为Int，String，Char；返回值类型为Boolean。
>
> 要求调用函数fun(0, “”, ‘0’)得到返回值为false，其它情况均返回true。
>
> 练习2：
>
> 定义一个函数func，它接收一个Int类型的参数，返回一个函数（记作f1）。
>
> 它返回的函数f1，接收一个String类型的参数，同样返回一个函数（记作f2）。
>
> 函数f2接收一个Char类型的参数，返回一个Boolean的值。
>
> 要求调用函数func(0) (“”) (‘0’)得到返回值为false，其它情况均返回true。

```scala
object Test08_Practice {
  def main(args: Array[String]): Unit = {
    // 1. 练习1
    val fun = (i: Int, s: String, c: Char) => {
      if (i == 0 && s == "" && c == '0') false else true
    }

    println(fun(0, "", '0'))
    println(fun(0, "", '1'))
    println(fun(23, "", '0'))
    println(fun(0, "hello", '0'))

    println("===========================")

    // 2. 练习2
    def func(i: Int): String => (Char => Boolean) = {
      def f1(s: String): Char => Boolean = {
        def f2(c: Char): Boolean = {
          if (i == 0 && s == "" && c == '0') false else true
        }

        f2
      }

      f1
    }

    println(func(0)("")('0'))
    println(func(0)("")('1'))
    println(func(23)("")('0'))
    println(func(0)("hello")('0'))

    // 匿名函数简写
    def func1(i: Int): String => (Char => Boolean) = {
      s => c => if (i == 0 && s == "" && c == '0') false else true
    }

    println(func1(0)("")('0'))
    println(func1(0)("")('1'))
    println(func1(23)("")('0'))
    println(func1(0)("hello")('0'))

    // 柯里化
    def func2(i: Int)(s: String)(c: Char): Boolean = {
      if (i == 0 && s == "" && c == '0') false else true
    }

    println(func2(0)("")('0'))
    println(func2(0)("")('1'))
    println(func2(23)("")('0'))
    println(func2(0)("hello")('0'))
  }
}
```

> ```scala
> false
> true
> true
> true
> ===========================
> false
> true
> true
> true
> false
> true
> true
> true
> false
> true
> true
> true
> ```

###### Test09_ClosureAndCurrying.scala

> 【函数柯里化&闭包】
>
> 闭包：函数式编程的标配
>
> 闭包：如果一个函数，访问到了它的外部（局部）变量的值，那么这个函数和他所处的环境，称为闭包
>
> 函数柯里化：把一个参数列表的多个参数，变成多个参数列表

```scala
object Test09_ClosureAndCurrying {
  def main(args: Array[String]): Unit = {
    def add(a: Int, b: Int): Int = {
      a + b
    }

    // 1. 考虑固定一个加数的场景
    def addByFour(b: Int): Int = {
      4 + b
    }

    // 2. 扩展固定加数改变的情况
    def addByFive(b: Int): Int = {
      5 + b
    }

    // 3. 将固定加数作为另一个参数传入，但是是作为“第一层参数”传入
    def addByFour1(): Int => Int = {
      val a = 4

      def addB(b: Int): Int = {
        a + b
      }

      addB
    }

    def addByA(a: Int): Int => Int = {
      def addB(b: Int): Int = {
        a + b
      }

      addB
    }

    println(addByA(35)(24))

    val addByFour2 = addByA(4)
    val addByFive2 = addByA(5)

    println(addByFour2(13))
    println(addByFive2(25))

    // 4. lambda表达式简写
    def addByA1(a: Int): Int => Int = {
      (b: Int) => {
        a + b
      }
    }

    def addByA2(a: Int): Int => Int = {
      b => a + b
    }

    def addByA3(a: Int): Int => Int = a + _

    val addByFour3 = addByA3(4)
    val addByFive3 = addByA3(5)

    println(addByFour3(13))
    println(addByFive3(25))

    // 5. 柯里化
    def addCurrying(a: Int)(b: Int): Int = {
      a + b
    }

    println(addCurrying(35)(24))
  }
}
```

###### Test10_Recursion.scala

> 【递归】阶乘、尾递归
>
> 一个函数/方法在函数/方法体内又调用了本身，我们称之为递归调用
>
> 递归算法
>
> （1）方法调用自身
>
> （2）方法必须要有跳出的逻辑
>
> （3）方法调用自身时，传递的参数应该有规律
>
> （4）scala 中的递归必须声明函数返回值类型

TestRecursion.java

```java
public class TestRecursion {
    public static void main(String[] args) {
        // 计算阶乘
        System.out.println(factorial(5));
        System.out.println(fact(5));
    }

    // 1. 循环实现
    public static int factorial(int n) {
        int result = 1;
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    // 2. 递归实现
    public static int fact(int n) {
        // 基准情形 0! = 1
        if (n == 0) return 1;
        return fact(n - 1) * n;
    }
}
```

```scala
import scala.annotation.tailrec

object Test10_Recursion {
  def main(args: Array[String]): Unit = {
    println(fact(5))
    println(tailFact(5))
  }

  // 递归实现计算阶乘
  def fact(n: Int): Int = {
    if (n == 0) return 1
    fact(n - 1) * n
  }

  // 尾递归实现
  def tailFact(n: Int): Int = {
    @tailrec
    def loop(n: Int, currRes: Int): Int = {
      if (n == 0) return currRes
      loop(n - 1, currRes * n)
    }

    loop(n, 1)
  }
}
```

###### Test11_ControlAbstraction.scala

> 【控制抽象】传值参数、传名参数
>
> 值调用： 把计算后的值传递过去
>
> 名调用： 把代码传递过去

```scala
object Test11_ControlAbstraction {
  def main(args: Array[String]): Unit = {
    // 1. 传值参数
    def f0(a: Int): Unit = {
      println("a: " + a)
      println("a: " + a)
    }

    f0(23)

    def f1(): Int = {
      println("f1调用")
      12
    }

    f0(f1())

    println("========================")

    // 2. 传名参数，传递的不再是具体的值，而是代码块
    def f2(a: => Int): Unit = {
      println("a: " + a)
      println("a: " + a)
    }

    f2(23)
    f2(f1())

    f2({
      println("这是一个代码块")
      29
    })

  }
}
```

> ```scala
> a: 23
> a: 23
> f1调用
> a: 12
> a: 12
> ========================
> a: 23
> a: 23
> f1调用
> a: 12
> f1调用
> a: 12
> 这是一个代码块
> a: 29
> 这是一个代码块
> a: 29
> ```
>
> 是不是将 a: => Int 认为是返回值为Int类型的一段代码，在函数体中a出现一次，这段代码执行一次。
>
> 而函数也是相应的一段代码，可以直接传入，如：f2(f1())。
>
> 其实，再从数学的角度考虑，将代码块视为一个多项式。这个名传递，不就是将一个多项式（可以计算出结果）整体带入到另一个式子里计算吗？并且没出现一次，都要重新计算一次这个多项式；而对于值传递，就是先计算出这个多项式的准确数值，然后再把此结果带入另一个式子里计算。这样的话，这个多项式其实只计算一次哎。
>
> 那么名传递岂不是要多进行一些计算吗？
>
> 比如：
>
> 值传递的结果
>
> ```
> f1调用
> a: 12
> a: 12
> ```
>
> 名传递的结果
>
> ```
> f1调用
> a: 12
> f1调用
> a: 12
> ```
>
> 很明显名传递多执行了一部分代码啊。

###### Test12_MyWhile.scala

> 【自定义一个 While 循环】

```scala
object Test12_MyWhile {
  def main(args: Array[String]): Unit = {
    var n = 10

    // 1. 常规的while循环
    while (n >= 1) {
      println(n)
      n -= 1
    }

    // 2. 用闭包实现一个函数，将代码块作为参数传入，递归调用
    def myWhile(condition: => Boolean): (=> Unit) => Unit = {
      // 内层函数需要递归调用，参数就是循环体
      def doLoop(op: => Unit): Unit = {
        if (condition) {
          op
          myWhile(condition)(op)
        }
      }

      doLoop _
    }

    println("=================")
    n = 10
    myWhile(n >= 1) {
      println(n)
      n -= 1
    }

    // 3. 用匿名函数实现
    def myWhile2(condition: => Boolean): (=> Unit) => Unit = {
      // 内层函数需要递归调用，参数就是循环体
      op => {
        if (condition) {
          op
          myWhile2(condition)(op)
        }
      }
    }

    println("=================")
    n = 10
    myWhile2(n >= 1) {
      println(n)
      n -= 1
    }

    // 3. 用柯里化实现
    def myWhile3(condition: => Boolean)(op: => Unit): Unit = {
      if (condition) {
        op
        myWhile3(condition)(op)
      }
    }

    println("=================")
    n = 10
    myWhile3(n >= 1) {
      println(n)
      n -= 1
    }
  }
}
```

###### Test13_Lazy.scala

> 【惰性加载】
>
> 当函数返回值被声明为 lazy 时 ，函数的执行将被推迟 ，直到我们首次对此取值，该函数才会执行。这种函数我们称之为惰性函数。
>
> 注意：lazy 不能修饰 var 类型的变量。

```scala
object Test13_Lazy {
  def main(args: Array[String]): Unit = {
    lazy val result: Int = sum(13, 47)

    println("1. 函数调用")
    println("2. result = " + result)
    println("4. result = " + result)
  }

  def sum(a: Int, b: Int): Int = {
    println("3. sum调用")
    a + b
  }
}
```

> ```scala
> 1. 函数调用
> 3. sum调用
> 2. result = 60
> 4. result = 60
> ```

##### 面向对象

> Scala 的面向对象思想和 Java 的面向对象思想和概念是一致的。
> Scala 中语法和 Java 不同 ，补充了更多的功能。

###### Test01_Package.scala

> 【包语句】
>
> （1）一个源文件中可以声明多个 package
> （2）子包中的类可以直接访问父包中的内容，而无需导包

```scala
// 用嵌套风格定义包
package com {

  import com.ahua.scala.Inner

  // 在外层包中定义单例对象
  object Outer {
    var out: String = "out"

    // 外层访问内层要导内层包
    def main(args: Array[String]): Unit = {
      println(Inner.in)	// in
    }
  }
  package ahua {
    package scala {

      // 内层包中定义单例对象
      object Inner {
        var in: String = "in"

        // 内层访问外层不需要导包
        def main(args: Array[String]): Unit = {
          println(Outer.out)	// out
          Outer.out = "outer"
          println(Outer.out)	// outer
        }
      }

    }

  }

}

// 在同一文件中定义不同的包
package aaa {
  package bbb {

    object Test01_Package {
      def main(args: Array[String]): Unit = {
        import com.atguigu.scala.Inner
        println(Inner.in)	// in
      }
    }

  }

}
```

###### package

> 【包对象】
>
> 在 Scala 中可以为每个包定义一个同名的包对象，定义在包对象中的成员，作为其对应包下所有 class 和 object 的共享变量，可以被直接访问。

```scala
package object chapter06 {
  // 定义当前包共享的属性和方法
  val commonValue = "Scala"

  def commonMethod() = {
    println(s"我在学习${commonValue}")
  }
}
```

###### Test02_PackageObject.scala

> 【访问包对象】
>
> 如采用嵌套方式管理包，则包对象可与包定义在同一文件中，但是要保证包对象与包声明在同一作用域中（同一层级下）。

```scala
//package chapter06
//
//object Test02_PackageObject {
//  def main(args: Array[String]): Unit = {
//    commonMethod()
//    println(commonValue)
//  }
//}

package chapter06 {

  object Test02_PackageObject {
    def main(args: Array[String]): Unit = {
      commonMethod()
      println(commonValue)
    }
  }

}

package ccc {
  package ddd {

    object Test02_PackageObject {
      def main(args: Array[String]): Unit = {
        println(school)
      }
    }

  }

}

// 定义一个包对象
package object ccc {
  val school: String = "CQUPT"
}
```

> 包 ccc 与包对象 ccc 要在同一层级下，才能访问 school
>
> 若把
>
> ```scala
> package object ccc {
>   val school: String = "CQUPT"
> }
> ```
>
> 改为
>
> ```scala
> package object ddd {
>   val school: String = "CQUPT"
> }
> ```
>
> 由于package ddd 与 package object ddd 不在同一层级下，那么在 main 中就不能访问到 school 了。
>
> 如果改成下面这种就又可以了：
>
> ```scala
> package ccc {
>   package ddd {
> 
>     object Test02_PackageObject {
>       def main(args: Array[String]): Unit = {
>         println(school)
>       }
>     }
> 
>   }
> 
>   // 定义一个包对象
>   package object ddd {
>     val school: String = "CQUPT"
>   }
> 
> }
> ```

###### Test03_Class.scala

> 【类和对象】
>
> 类：可以看成一个模板
> 对象：表示具体的事物
>
> 回顾：Java 中的类
> 如果类是 public 的，则必须和文件名一致
> 一般，一个 .java 有一个 public 类
> 注意：Scala 中没有 public，一个 .scala 中可以写多个类 
> 基本语法
> [修饰符 ] class 类名{
> 		类体
> }
> 说明
> （1）Scala 语法中，类并不声明为 public ，所有这些类都具有公有可见性（即默认就是 public）
> （2）一个 Scala 源文件可以包含多个类

```scala
import scala.beans.BeanProperty

object Test03_Class {
  def main(args: Array[String]): Unit = {
    // 创建一个对象
    val student = new Student()
    //    student.name   // error, 不能访问private属性
    println(student.age)
    println(student.sex)
    student.sex = "female"
    println(student.sex)
  }
}

// 定义一个类
class Student {
  // 定义属性
  private var name: String = "alice"
  @BeanProperty
  var age: Int = _
  var sex: String = _
}
```

> ```scala
> 0
> null
> female
> ```

###### Test04_Access.scala

> 【访问权限】
>
> 在 Java 中，访问权限分为：public、private、protected 和默认。在 Scala 中，则可以通过类似的修饰符达到同样的效果。但是使用上有区别。
> （1） Scala 中属性和方法的默认访问权限为 public，但 Scala 中无 public 关键字。
> （2） private 为私有权限，只在类的内部和伴生对象中可用。
> （3） protected 为受保护权限， Scala 中受保护权限比 Java 中更严格，同类、子类可以访问，同包无法访问。
> （4） private[包名] 增加包访问权限，包名下的其他类也可以使用

```scala
object Test04_Access {
  def main(args: Array[String]): Unit = {
    // 创建对象
    val person: Person = new Person()
    //    person.idCard    // error	(2)
    //    person.name    // error	(3)
    println(person.age)	//	(4)
    println(person.sex)

    person.printInfo()

    val worker: Worker = new Worker()
    //    worker = new Worker()
    //    worker.age = 23
    worker.printInfo()
  }
}

// 定义一个子类
class Worker extends Person {
  override def printInfo(): Unit = {
    //    println(idCard)    // error
    name = "bob"
    age = 25
    sex = "male"

    println(s"Worker: $name $sex $age")
  }
}
```

Test04_ClassForAccess.scala

```scala
object Test04_ClassForAccess {

}

// 定义一个父类
class Person {
  private var idCard: String = "3523566"
  protected var name: String = "alice"
  var sex: String = "female"
  private[chapter06] var age: Int = 18

  def printInfo(): Unit = {
    println(s"Person: $idCard $name $sex $age")
  }
}
```

> ```
> 18
> female
> Person: 3523566 alice female 18
> Worker: bob male 25
> ```

###### Test05_Constructor.scala

> 【构造器】主构造器、辅助构造器
>
> （1）辅助构造器函数的名称 this，可以有多个 ，编译器通过参数的个数及类型来区分。
> （2）辅助构造方法不能直接构建对象 ，必须直接或者间接调用主构造方法。
> （3）构造器调用其他另外的构造器， 要求被调用构造器必须提前声明。

```scala
object Test05_Constructor {
  def main(args: Array[String]): Unit = {
    val student1 = new Student1
    student1.Student1()
    
    println("========================")

    val student2 = new Student1("alice")
    
    println("========================")

    val student3 = new Student1("bob", 25)
  }
}

// 定义一个类
class Student1() {// 主构造器
  // 定义属性
  var name: String = _
  var age: Int = _

  println("1. 主构造方法被调用")

  // 声明辅助构造方法
  def this(name: String) {
    this() // 直接调用主构造器
    println("2. 辅助构造方法一被调用")
    this.name = name
    println(s"name: $name age: $age")
  }

  def this(name: String, age: Int) {
    this(name)
    println("3. 辅助构造方法二被调用")
    this.age = age
    println(s"name: $name age: $age")
  }

  def Student1(): Unit = {
    println("一般方法被调用")
  }
}
```

> ```scala
> 1. 主构造方法被调用
> 一般方法被调用
> ========================
> 1. 主构造方法被调用
> 2. 辅助构造方法一被调用
> name: alice age: 0
> ========================
> 1. 主构造方法被调用
> 2. 辅助构造方法一被调用
> name: bob age: 0
> 3. 辅助构造方法二被调用
> name: bob age: 25
> ```

###### Test06_ConstructorParams.scala

> 【构造器参数】
>
> Scala 类的主构造器函数的形参包括三种类型：未用任何修饰、var 修饰、val 修饰
> （1）未用任何修饰符修饰，这个参数就是一个局部变量
> （2）var 修饰参数，作为类的成员属性使用，可以修改
> （3）val 修饰参数，作为类只读属性使用，不能修改

```scala
object Test06_ConstructorParams {
  def main(args: Array[String]): Unit = {
    val student2 = new Student2
    student2.name = "alice"
    student2.age = 18
    println(s"student2: name = ${student2.name}, age = ${student2.age}")

    val student3 = new Student3("bob", 20)
    println(s"student3: name = ${student3.name}, age = ${student3.age}")

    val student4 = new Student4("cary", 25)
    //    println(s"student4: name = ${student4.name}, age = ${student4.age}")
    student4.printInfo()

    val student5 = new Student5("bob", 20)
    println(s"student5: name = ${student5.name}, age = ${student5.age}")

    //    student5.age = 21	// student5的name age赋值后就不能再更改了

    val student6 = new Student6("cary", 25, "CQUPT")
    println(s"student6: name = ${student6.name}, age = ${student6.age}")
    student6.printInfo()
  }
}

// 定义类
// 无参构造器
class Student2 {
  // 单独定义属性
  var name: String = _
  var age: Int = _
}

// 上面定义等价于
class Student3(var name: String, var age: Int)

// 主构造器参数无修饰
class Student4(name: String, age: Int) {
  def printInfo() {
    println(s"student4: name = ${name}, age = $age")
  }
}

//class Student4(_name: String, _age: Int){
//  var name: String = _name
//  var age: Int = _age
//}

class Student5(val name: String, val age: Int)

class Student6(var name: String, var age: Int) {
  var school: String = _

  def this(name: String, age: Int, school: String) {
    this(name, age)
    this.school = school
  }

  def printInfo() {
    println(s"student6: name = ${name}, age = $age, school = $school")
  }
}
```

> ```scala
> student2: name = alice, age = 18
> student3: name = bob, age = 20
> student4: name = cary, age = 25
> student3: name = bob, age = 20
> student6: name = cary, age = 25
> student6: name = cary, age = 25, school = CQUPT
> ```

###### Test07_Inherit.scala

> 【继承和多态】
>
> 基本语法
>
> class 子类名 extends 父类名 { 类体 }
>
> （1）子类继承父类的 属性 和 方法
> （2） scala 是单继承
> （3）继承的调用顺序：父类构造器->子类构造器

```scala
object Test07_Inherit {
  def main(args: Array[String]): Unit = {
      
    val student1: Student7 = new Student7("alice", 18)

    println("=========================")

    val student2 = new Student7("bob", 20, "std001")

    student1.printInfo()
    student2.printInfo()

    val teacher = new Teacher
    teacher.printInfo()

    def personInfo(person: Person7): Unit = {
      person.printInfo()
    }

    println("=========================")

    val person = new Person7
    personInfo(student1)
    personInfo(teacher)
    personInfo(person)
  }
}

// 定义一个父类
class Person7() {
  var name: String = _
  var age: Int = _

  println("1. 父类的主构造器调用")

  def this(name: String, age: Int) {
    this()
    println("2. 父类的辅助构造器调用")
    this.name = name
    this.age = age
  }

  def printInfo(): Unit = {
    println(s"Person: $name $age")
  }
}

// 定义子类
class Student7(name: String, age: Int) extends Person7(name, age) {
  var stdNo: String = _

  println("3. 子类的主构造器调用")

  def this(name: String, age: Int, stdNo: String) {
    this(name, age)
    println("4. 子类的辅助构造器调用")
    this.stdNo = stdNo
  }

  override def printInfo(): Unit = {
    println(s"Student: $name $age $stdNo")
  }
}

class Teacher extends Person7 {
  override def printInfo(): Unit = {
    println(s"Teacher")
  }
}
```

###### Test08_DynamicBind.scala

> 【多态】动态绑定、静态绑定
>
> 在Java中，属性是静态绑定的，方法是动态绑定的
>
> 在Scala中，属性和方法都是动态绑定的

TestDynamicBind.java

```java
public class TestDynamicBind {
    public static void main(String[] args) {
        Worker worker = new Worker();
        System.out.println(worker.name);
        worker.hello();
        worker.hi();

        System.out.println("=========================");

        // 多态
        Person person = new Worker();
        System.out.println(person.name);    // 静态绑定属性
        person.hello();    // 动态绑定方法
//        person.hi();     // error
    }
}

class Person {
    String name = "person";

    public void hello() {
        System.out.println("hello person");
    }
}

class Worker extends Person {
    String name = "worker";

    public void hello() {
        System.out.println("hello worker");
    }

    public void hi() {
        System.out.println("hi worker");
    }
}
```

> ```java
> worker
> hello worker
> hi worker
> =========================
> person
> hello worker
> ```
>
> person.name仍然输出是person，而person.hello(); 调用的则是子类的hello方法

```scala
object Test08_DynamicBind {
  def main(args: Array[String]): Unit = {
    val student: Person8 = new Student8
    println(student.name)
    student.hello()
  }
}

class Person8 {
  val name: String = "person"

  def hello(): Unit = {
    println("hello person")
  }
}

class Student8 extends Person8 {
  override val name: String = "student"

  override def hello(): Unit = {
    println("hello student")
  }
}
```

> ```scala
> student
> hello student
> ```

###### Test09_AbstractClass.scala

> 【抽象类】抽象属性、抽象方法
>
> 1）基本语法
>
> （1）定义抽象类：abstract class Person{}	//通过 abstract 关键字标记抽象类
> （2）定义抽象属性：val/var name:String	//一个属性没有初始化，就是抽象属性
> （3）定义抽象方法：def hello():String	//只声明而没有实现的方法，就是抽象方法
>
> 2）继承&重写
> （1）如果父类为抽象类，那么子类需要将抽象的属性和方法实现，否则子类也需声明为抽象类
> （2）重写非抽象方法需要用 override 修饰，重写抽象方法则可以不加 override
> （3）子类中调用父类的方法使用 super 关键字
> （4）子类对**抽象属性**进行实现，父类抽象属性可以用 var 修饰；
> 子类对**非抽象属性**重写 ，父类非抽象属性只支持 val 类型，而不支持 var。
> 因为 var 修饰的为可变变量，子类继承之后就可以直接使用，没有必要重写。

```scala
object Test09_AbstractClass {
  def main(args: Array[String]): Unit = {
    val student = new Student9
    student.eat()
    student.sleep()
  }
}

// 定义抽象类
abstract class Person9 {
  // 非抽象属性
  var name: String = "person"

  // 抽象属性
  var age: Int

  // 非抽象方法
  def eat(): Unit = {
    println("person eat")
  }

  // 抽象方法
  def sleep(): Unit
}

// 定义具体的实现子类
class Student9 extends Person9 {
  // 实现抽象属性和方法
  var age: Int = 18

  def sleep(): Unit = {
    println("student sleep")
  }

  // 重写非抽象属性和方法
  //  override val name: String = "student"
  name = "student"

  override def eat(): Unit = {
    super.eat()
    println("student eat")
  }
}
```

> ```scala
> person eat
> student eat
> student sleep
> ```
>
> 由于在抽象父类中`var name: String = "person"`，name被定义为var，则在子类中只需要重新赋值就行了，而不需要再去重写name；而如果抽象父类中name是定义为val，则在子类中就可重写name，`override val name: String = "student"`，就是正确的。

###### Test10_AnonymousClass.scala

> 【匿名子类】

```scala
object Test10_AnonymousClass {
  def main(args: Array[String]): Unit = {
    val person: Person10 = new Person10 {
      override var name: String = "alice"

      override def eat(): Unit = println("person eat")
    }
    println(person.name)
    person.eat()
  }
}

// 定义抽象类
abstract class Person10 {
  var name: String

  def eat(): Unit
}
```

> ```scala
> alice
> person eat
> ```
>
> 没看懂，和匿名子类有什么关系。
>
> 就是说，没有特意去实现一个子类，再创建一个实例，调用这个方法，也能够得到相同结果吗？

###### Test11_Object.scala

> 【单例对象（伴生对象）】单例对象、apply方法
>
> Scala语言是完全面向对象的语言，所以并没有静态的操作（即在Scala中没有静态的概念）。但是为了能够和Java语言交互（因为Java中有静态概念），就产生了一种特殊的对象来模拟类对象，该对象为单例对象。若单例对象名与类名一致，则称该单例对象为这个类的伴生对象，这个类的所有“静态”内容都可以放置在它的伴生对象中声明。
>
> （1）单例对象采用 object 关键字声明
> （2）单例对象对应的类称之为 伴生类 ，伴生对象的名称和伴生类名一致
> （3）单例对象中的属性和方法都可以通过伴生对象名（类名）直接调用访问

```scala
object Test11_Object {
  def main(args: Array[String]): Unit = {
    //    val student = new Student11("alice", 18)
    //    student.printInfo()

    val student1 = Student11.newStudent("alice", 18)
    student1.printInfo()

    val student2 = Student11.apply("bob", 19)
    student2.printInfo()

    val student3 = Student11("bob", 19)
    student3.printInfo()
  }
}

// 定义类
class Student11 private(val name: String, val age: Int) {
  def printInfo() {
    println(s"student: name = ${name}, age = $age, school = ${Student11.school}")
  }
}

// 伴生对象
object Student11 {
  val school: String = "CQUPT"

  // 定义一个类的对象实例的创建方法
  def newStudent(name: String, age: Int): Student11 = new Student11(name, age)

  def apply(name: String, age: Int): Student11 = new Student11(name, age)
}
```

> ```scala
> student: name = alice, age = 18, school = CQUPT
> student: name = bob, age = 19, school = CQUPT
> student: name = bob, age = 19, school = CQUPT
> ```
>
> （1）通过伴生对象的 apply 方法， 实现不使用 new 方法创建对象 。
> （2）如果想让主构造器变成私有的，可以在之前加上 private。
> （3）apply 方法可以重载。
> （4）Scala 中 obj (arg) 的语句实际是在调用该对象的 apply 方法，即 obj.apply(arg) 。用以统一面向对象编程和函数式编程的风格。
> （5）当使用 new 关键字构建对象时，调用的其实是类的构造方法；当直接使用类名构建对象时，调用的其实时伴生对象的 apply 方法。

###### Test12_Singleton.scala

> 【单例模式】

```scala
object Test12_Singleton {
  def main(args: Array[String]): Unit = {
    val student1 = Student12.getInstance()
    student1.printInfo()

    val student2 = Student12.getInstance()
    student2.printInfo()

    // 打印输出引用地址
    println(student1)
    println(student2)
  }
}

class Student12 private(val name: String, val age: Int) {
  def printInfo() {
    println(s"student: name = ${name}, age = $age, school = ${Student11.school}")
  }
}

// 饿汉式
//object Student12 {
//  private val student: Student12 = new Student12("alice", 18)
//  def getInstance(): Student12 = student
//}

// 懒汉式
object Student12 {
  private var student: Student12 = _

  def getInstance(): Student12 = {
    if (student == null) {
      // 如果没有对象实例的话，就创建一个
      student = new Student12("alice", 18)
    }
    student
  }
}
```

> ```scala
> student: name = alice, age = 18, school = CQUPT
> student: name = alice, age = 18, school = CQUPT
> chapter06.Student12@4563e9ab
> chapter06.Student12@4563e9ab
> ```

###### Test13_Trait.scala

> 【特质（Trait）】类似于Java中的接口（Interface）
>
> Scala 语言中，采用特质 trait（特征）来代替接口的概念，也就是说，多个类具有相同的特质（特征）时，就可以将这个特质（特征）独立出来，采用关键字 trait 声明。
> Scala 中的 trait 中即可以有抽象属性和方法，也可以有具体的属性和方法。一个类可以**混入（ mixin ）**多个特质。这种感觉类似于 Java 中的抽象类。
> Scala 引入 trait 特征，第一是可以替代 Java 的接口，第二也是对单继承机制的一种补充。

```scala
object Test13_Trait {
  def main(args: Array[String]): Unit = {
    val student: Student13 = new Student13
    student.sayHello()
    student.study()
    student.dating()
    student.play()
  }
}

// 定义一个父类
class Person13 {
  val name: String = "person"
  var age: Int = 18

  def sayHello(): Unit = {
    println("hello from: " + name)
  }

  def increase(): Unit = {
    println("person increase")
  }
}

// 定义一个特质
trait Young {
  // 声明抽象和非抽象属性
  var age: Int
  val name: String = "young"

  // 声明抽象和非抽象的方法
  def play(): Unit = {
    println(s"young people $name is playing")
  }

  def dating(): Unit
}

class Student13 extends Person13 with Young {
  // 重写冲突的属性
  override val name: String = "student"

  // 实现抽象方法
  def dating(): Unit = println(s"student $name is dating")

  def study(): Unit = println(s"student $name is studying")

  // 重写父类方法
  override def sayHello(): Unit = {
    super.sayHello()
    println(s"hello from: student $name")
  }
}
```

> ```scala
> hello from: student
> hello from: student student
> student student is studying
> student student is dating
> young people student is playing
> ```

###### Test14_TraitMixin.scala

> 【Trait 混入（ mixin ）】

```scala
object Test14_TraitMixin {
  def main(args: Array[String]): Unit = {
    val student = new Student14
    student.study()
    student.increase()

    student.play()
    student.increase()

    student.dating()
    student.increase()

    println("===========================")

    // 动态混入
    val studentWithTalent = new Student14 with Talent {
      override def dancing(): Unit = println("student is good at dancing")

      override def singing(): Unit = println("student is good at singing")
    }

    studentWithTalent.sayHello()
    studentWithTalent.play()
    studentWithTalent.study()
    studentWithTalent.dating()
    studentWithTalent.dancing()
    studentWithTalent.singing()
  }
}

// 再定义一个特质
trait Knowledge {
  var amount: Int = 0

  def increase(): Unit
}

trait Talent {
  def singing(): Unit

  def dancing(): Unit
}

class Student14 extends Person13 with Young with Knowledge {
  // 重写冲突的属性
  override val name: String = "student"

  // 实现抽象方法
  def dating(): Unit = println(s"student $name is dating")

  def study(): Unit = println(s"student $name is studying")

  // 重写父类方法
  override def sayHello(): Unit = {
    super.sayHello()
    println(s"hello from: student $name")
  }

  // 实现特质中的抽象方法
  override def increase(): Unit = {
    amount += 1
    println(s"student $name knowledge increased: $amount")
  }
}
```

> ```scala
> student student is studying
> student student knowledge increased: 1
> young people student is playing
> student student knowledge increased: 2
> student student is dating
> student student knowledge increased: 3
> ===========================
> hello from: student
> hello from: student student
> young people student is playing
> student student is studying
> student student is dating
> student is good at dancing
> student is good at singing
> ```

###### Test15_TraitOverlying.scala

> 【特质叠加/执行顺序】
>
> 由于一个类可以混入（mixin ）多个 trait ，且 trait 中可以有具体的属性和方法，若混入
> 的特质中具有相同的方法（方法名，参数列表，返回值均相同），必然会出现继承冲突问题。
> 冲突分为以下两种：
>
> 第一种，一个类（ Sub ）混入的两个 trait（ TraitA TraitB ）中具有相同的具体方法，且
> 两个 trait 之间没有任何关系，解决这类冲突问题，直接在类（ Sub ）中重写冲突方法。
>
> 就像Text13那样
>
> 第二种，一个类（ Sub ）混入的两个 trait（TraitA TraitB ）中具有相同的具体方法，且
> 两个 trait 继承自相同的 trait（ TraitC ），及所谓的“钻石问题”，解决这类冲突问题 Scala
> 采用了 **特质叠加** 的策略。
>
> 当一个类混入多个特质的时候，scala 会对所有的特质及其父特质按照一定的顺序进行排序。

```scala
object Test15_TraitOverlying {
  def main(args: Array[String]): Unit = {
    val student = new Student15
    student.increase()

    // 钻石问题特征叠加
    val myFootBall = new MyFootBall
    println(myFootBall.describe())
  }
}

// 定义球类特征
trait Ball {
  def describe(): String = "ball"
}

// 定义颜色特征
trait ColorBall extends Ball {
  var color: String = "red"

  override def describe(): String = color + "-" + super.describe()
}

// 定义种类特征
trait CategoryBall extends Ball {
  var category: String = "foot"

  override def describe(): String = category + "-" + super.describe()
}

// 定义一个自定义球的类
class MyFootBall extends CategoryBall with ColorBall {
  override def describe(): String = "my ball is a " + super[CategoryBall].describe()
}

trait Knowledge15 {
  var amount: Int = 0

  def increase(): Unit = {
    println("knowledge increased")
  }
}

trait Talent15 {
  def singing(): Unit

  def dancing(): Unit

  def increase(): Unit = {
    println("talent increased")
  }
}

class Student15 extends Person13 with Talent15 with Knowledge15 {
  override def dancing(): Unit = println("dancing")

  override def singing(): Unit = println("singing")

  override def increase(): Unit = {
    super[Person13].increase()
  }
}
```

> ```scala
> person increase
> my ball is a foot-ball
> ```
>
> 自定义球类中`super[CategoryBall].describe()`改为`super.describe()`的话，输出为`red-foot-ball`
>
> MyClass 中的 super 指代 Color，Color 中的 super 指代 Category，Category 中的 super 指代 Ball
>
> 如果想要调用某个指定的混入特质中的方法，可以增加约束：super[]，例如：super[CategoryBall].describe()

###### Test16_TraitSelfType.scala

> 【特质自身类型】

```scala
object Test16_TraitSelfType {
  def main(args: Array[String]): Unit = {
    val user = new RegisterUser("alice", "123456")
    user.insert()
  }
}

// 用户类
class User(val name: String, val password: String)

trait UserDao {
  _: User =>

  // 向数据库插入数据
  def insert(): Unit = {
    println(s"insert into db: ${this.name}")
  }
}

// 定义注册用户类
class RegisterUser(name: String, password: String) extends User(name, password) with UserDao
```

> _: User => 中，
>
> _可以是其它的字符比如：abc
>
> ```scala
> insert into db: alice
> ```
>
> UserDao要使用User中的一些属性，但是又不想和User有什么继承上的关系，则就可以指定一个自身类型，然后就相当于在UserDao中有了一个User对象一样，就可以调用User中的一些属性了。
>
> 当前要用到的这个User类的实例，相当于是从外部注入的，就实现依赖注入的功能。

> **特质和抽象类的区别**
>
> 优先使用特质。一个类扩展多个特质是很方便的，但却只能扩展一个抽象类。
>
> 如果你需要构造函数参数，使用抽象类。因为抽象类可以定义 带参数 的构造函数，而特质不行（有无参构造器） 。

###### Test17_Extends.scala

> 【拓展】类型检查和转换、枚举类、应用类
>
> obj.isInstanceOf[T] T]：判断 obj 是不是 T 类型
>
> obj.asInstanceOf[T] T]：将 obj 强转成 T 类型
>
> classOf 获取对象的类名
>
> 使用 type 关键字可以定义新的数据数据类型名称，本质上就是类型的一个别名

```scala
object Test17_Extends {
  def main(args: Array[String]): Unit = {
    // 1. 类型的检测和转换
    val student: Student17 = new Student17("alice", 18)
    student.study()
    student.sayHi()
    val person: Person17 = new Student17("bob", 20)
    person.sayHi()

    // 类型判断
    println("student is Student17: " + student.isInstanceOf[Student17])
    println("student is Person17: " + student.isInstanceOf[Person17])
    println("person is Person17: " + person.isInstanceOf[Person17])
    println("person is Student: " + person.isInstanceOf[Student17])

    val person2: Person17 = new Person17("cary", 35)
    println("person2 is Student17: " + person2.isInstanceOf[Student17])

    // 类型转换
    if (person.isInstanceOf[Student17]) {
      val newStudent = person.asInstanceOf[Student17]
      newStudent.study()
    }

    println(classOf[Student17])
	println("========================")

    // 2. 测试枚举类
    println(WorkDay.MONDAY)
  }
}

class Person17(val name: String, val age: Int) {
  def sayHi(): Unit = {
    println("hi from person " + name)
  }
}

class Student17(name: String, age: Int) extends Person17(name, age) {
  override def sayHi(): Unit = {
    println("hi from student " + name)
  }

  def study(): Unit = {
    println("student study")
  }
}

// 定义枚举类对象
object WorkDay extends Enumeration {
  val MONDAY = Value(1, "Monday")
  val TUESDAY = Value(2, "TuesDay")
}

// 定义应用类对象
object TestApp extends App {
  println("app start")

  type MyString = String
  val a: MyString = "abc"
  println(a)
}
```

> ```scala
> student study
> hi from student alice
> hi from student bob
> student is Student17: true
> student is Person17: true
> person is Person17: true
> person is Student: true
> person2 is Student17: false
> student study
> class chapter06.Student17
> ========================
> Monday
> ```

##### 集合

###### Test01_ImmutableArray,scala

> 【不可变数组】能用不可变就用不可变

```scala
object Test01_ImmutableArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建数组
    val arr: Array[Int] = new Array[Int](5)
    // 另一种创建方式
    val arr2 = Array(12, 37, 42, 58, 97)
    println(arr)

    // 2. 访问元素
    println(arr(0))
    println(arr(1))
    println(arr(4))
    //    println(arr(5))

    arr(0) = 12
    arr(4) = 57
    println(arr(0))
    println(arr(1))
    println(arr(4))

    println("========================")

    // 3. 数组的遍历
    // 1) 普通for循环
    for (i <- 0 until arr.length) {
      println(arr(i))
    }

    for (i <- arr.indices) println(arr(i))

    println("---------------------")

    // 2) 直接遍历所有元素，增强for循环
    for (elem <- arr2) println(elem)

    println("---------------------")

    // 3) 迭代器
    val iter = arr2.iterator

    while (iter.hasNext)
      println(iter.next())

    println("---------------------")

    // 4) 调用foreach方法
    arr2.foreach((elem: Int) => println(elem))

    arr.foreach(println)

    println(arr2.mkString("--"))

    println("========================")
    // 4. 添加元素
    val newArr = arr2.:+(73)
    println(arr2.mkString("--"))
    println(newArr.mkString("--"))

    val newArr2 = newArr.+:(30)
    println(newArr2.mkString("--"))

    val newArr3 = newArr2 :+ 15
    val newArr4 = 19 +: 29 +: newArr3 :+ 26 :+ 73
    println(newArr4.mkString(", "))
  }
}
```

> ```scala
> [I@4563e9ab
> 0
> 0
> 0
> 12
> 0
> 57
> ========================
> 12
> 0
> 0
> 0
> 57
> 12
> 0
> 0
> 0
> 57
> ---------------------
> 12
> 37
> 42
> 58
> 97
> ---------------------
> 12
> 37
> 42
> 58
> 97
> ---------------------
> 12
> 37
> 42
> 58
> 97
> 12
> 0
> 0
> 0
> 57
> 12--37--42--58--97
> ========================
> 12--37--42--58--97
> 12--37--42--58--97--73
> 30--12--37--42--58--97--73
> 19, 29, 30, 12, 37, 42, 58, 97, 73, 15, 26, 73
> ```
>

###### Test02_ArrayBuffer.scala

> 【可变数组】

```scala
import scala.collection.mutable
import scala.collection.mutable.ArrayBuffer

object Test02_ArrayBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变数组
    val arr1: ArrayBuffer[Int] = new ArrayBuffer[Int]()
    val arr2 = ArrayBuffer(23, 57, 92)

    println(arr1)
    println(arr2)

    // 2. 访问元素
    //    println(arr1(0))     // error
    println(arr2(1))
    arr2(1) = 39
    println(arr2(1))

    println("======================")
    // 3. 添加元素
    val newArr1 = arr1 :+ 15
    println(arr1)
    println(newArr1)
    println(arr1 == newArr1)

    val newArr2 = arr1 += 19
    println(arr1)
    println(newArr2)
    println(arr1 == newArr2)
    newArr2 += 13
    println(arr1)

    77 +=: arr1
    println(arr1)
    println(newArr2)

    println("----------------------")

    arr1.append(36)
    arr1.prepend(11, 76)
    arr1.insert(1, 13, 59)
    println(arr1)

    arr1.insertAll(2, newArr1)
    arr1.prependAll(newArr2)

    println(arr1)

    println("======================")
    // 4. 删除元素
    arr1.remove(3)
    println(arr1)

    arr1.remove(0, 10)
    println(arr1)

    arr1 -= 13
    println(arr1)

    println("======================")
    // 5. 可变数组转换为不可变数组
    val arr: ArrayBuffer[Int] = ArrayBuffer(23, 56, 98)
    val newArr: Array[Int] = arr.toArray
    println(newArr.mkString(", "))
    println(arr)

    // 6. 不可变数组转换为可变数组
    val buffer: mutable.Buffer[Int] = newArr.toBuffer
    println(buffer)
    println(newArr)
  }
}
```

> ```scala
> ArrayBuffer()
> ArrayBuffer(23, 57, 92)
> 57
> 39
> ======================
> ArrayBuffer()
> ArrayBuffer(15)
> false
> ArrayBuffer(19)
> ArrayBuffer(19)
> true
> ArrayBuffer(19, 13)
> ArrayBuffer(77, 19, 13)
> ArrayBuffer(77, 19, 13)
> ----------------------
> ArrayBuffer(11, 13, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(11, 13, 15, 59, 76, 77, 19, 13, 36, 11, 13, 15, 59, 76, 77, 19, 13, 36)
> ======================
> ArrayBuffer(11, 13, 15, 76, 77, 19, 13, 36, 11, 13, 15, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(15, 59, 76, 77, 19, 13, 36)
> ArrayBuffer(15, 59, 76, 77, 19, 36)
> ======================
> 23, 56, 98
> ArrayBuffer(23, 56, 98)
> ArrayBuffer(23, 56, 98)
> [I@6cd8737
> ```

###### Test03_MulArray.scala

> 【二维数组】

```scala
object Test03_MulArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建二维数组
    val array: Array[Array[Int]] = Array.ofDim[Int](2, 3)

    // 2. 访问元素
    array(0)(2) = 19
    array(1)(0) = 25

    println(array.mkString(", "))
    for (i <- 0 until array.length; j <- 0 until array(i).length) {
      println(array(i)(j))
    }
    for (i <- array.indices; j <- array(i).indices) {
      print(array(i)(j) + "\t")
      if (j == array(i).length - 1) println()
    }

    array.foreach(line => line.foreach(println))

    array.foreach(_.foreach(println))
  }
}
```

> ```scala
> [I@b684286, [I@880ec60
> 0
> 0
> 19
> 25
> 0
> 0
> 0	0	19	
> 25	0	0	
> 0
> 0
> 19
> 25
> 0
> 0
> 0
> 0
> 19
> 25
> 0
> 0
> ```

###### Test04_List.scala

> 【不可变列表】

```scala
object Test04_List {
  def main(args: Array[String]): Unit = {
    // 1. 创建一个List
    val list1 = List(23, 65, 87)
    println(list1)	// List(23, 65, 87)

    // 2. 访问和遍历元素
    println(list1(1))
    //    list1(1) = 12
    list1.foreach(println)

    // 3. 添加元素
    val list2 = 10 +: list1
    val list3 = list1 :+ 23
    println(list1)	// List(23, 65, 87)
    println(list2)	// List(10, 23, 65, 87)
    println(list3)	// List(23, 65, 87, 23)

    println("==================")

    val list4 = list2.::(51)
    println(list4)	// List(51, 10, 23, 65, 87)

    val list5 = Nil.::(13)
    println(list5)	// List(13)

    val list6 = 73 :: 32 :: Nil
    val list7 = 17 :: 28 :: 59 :: 16 :: Nil
    println(list7)	// List(17, 28, 59, 16)

    // 4. 合并列表
    val list8 = list6 :: list7
    println(list8)	// List(List(73, 32), 17, 28, 59, 16)

    val list9 = list6 ::: list7
    println(list9)	// List(73, 32, 17, 28, 59, 16)

    val list10 = list6 ++ list7
    println(list10)	// List(73, 32, 17, 28, 59, 16)

  }
}
```

> ```scala
> List(23, 65, 87)
> 65
> 23
> 65
> 87
> List(23, 65, 87)
> List(10, 23, 65, 87)
> List(23, 65, 87, 23)
> ==================
> List(51, 10, 23, 65, 87)
> List(13)
> List(17, 28, 59, 16)
> List(List(73, 32), 17, 28, 59, 16)
> List(73, 32, 17, 28, 59, 16)
> List(73, 32, 17, 28, 59, 16)
> 
> Process finished with exit code 0
> 
> ```

###### Test05_ListBuffer.scala

> 【可变列表】

```scala
import scala.collection.mutable.ListBuffer

object Test05_ListBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变列表
    val list1: ListBuffer[Int] = new ListBuffer[Int]()
    val list2 = ListBuffer(12, 53, 75)

    println(list1)	// ListBuffer()
    println(list2)	// ListBuffer(12, 53, 75)

    println("==============")

    // 2. 添加元素
    list1.append(15, 62)
    list2.prepend(20)

    list1.insert(1, 19, 22)

    println(list1)	// ListBuffer(15, 19, 22, 62)
    println(list2)	// ListBuffer(20, 12, 53, 75)

    println("==============")

    31 +=: 96 +=: list1 += 25 += 11
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)

    println("==============")
    // 3. 合并list
    val list3 = list1 ++ list2	// list1 和 list2 都不变
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)
    println(list2)	// ListBuffer(20, 12, 53, 75)

    println("==============")

    list1 ++=: list2	// list1 不变 list2 前边加上 list1
    println(list1)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11)
    println(list2)	// ListBuffer(31, 96, 15, 19, 22, 62, 25, 11, 20, 12, 53, 75)

    println("==============")

    // 4. 修改元素
    list2(3) = 30	// 第 4 个元素被更改
    list2.update(0, 89)
    println(list2)	// ListBuffer(89, 96, 15, 30, 22, 62, 25, 11, 20, 12, 53, 75)

    // 5. 删除元素
    list2.remove(2)	// 删掉第 3 个元素 15
    list2 -= 25	// 删掉值为 25 的元素
    println(list2)	// ListBuffer(89, 96, 30, 22, 62, 11, 20, 12, 53, 75)
  }
}
```

###### Test06_ImmutableSet.scala

> 【不可变集合set】

```scala
object Test06_ImmutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1 = Set(13, 23, 53, 12, 13, 23, 78)
    println(set1)	// Set(78, 53, 13, 12, 23)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 129
    println(set1)	// Set(78, 53, 13, 12, 23)
    println(set2)	// Set(78, 53, 13, 129, 12, 23)
    println("==================")

    // 3. 合并set
    val set3 = Set(19, 13, 23, 53, 67, 99)
    val set4 = set2 ++ set3	// 将 set2 和 set3 合并
    println(set2)	// Set(78, 53, 13, 129, 12, 23)
    println(set3)	// Set(53, 13, 67, 99, 23, 19)
    println(set4)	// Set(78, 53, 13, 129, 12, 67, 99, 23, 19)

    // 4. 删除元素
    val set5 = set3 - 13	// set3 不变 set5 删除元素 13
    println(set3)	// Set(53, 13, 67, 99, 23, 19)
    println(set5)	// Set(53, 67, 99, 23, 19)
  }
}
```

###### Test07_MutableSet.scala

> 【可变集合set】

```scala
import scala.collection.mutable

object Test07_MutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1: mutable.Set[Int] = mutable.Set(13, 23, 53, 12, 13, 23, 78) // 元素被去重
    println(set1)	// Set(12, 78, 13, 53, 23)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 11
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set2)	// Set(12, 78, 13, 53, 11, 23)

    set1 += 11	// set1 本身添加元素 11
    println(set1)	// Set(12, 78, 13, 53, 11, 23)

    val flag1 = set1.add(10)
    println(flag1)	// true
    println(set1)	// Set(12, 78, 13, 53, 10, 11, 23)
    val flag2 = set1.add(10)	// 已存在元素 10 添加失败
    println(flag2)	// false
    println(set1)	// Set(12, 78, 13, 53, 10, 11, 23)

    println("==================")

    // 3. 删除元素
    set1 -= 11
    println(set1)	// Set(12, 78, 13, 53, 10, 23)

    val flag3 = set1.remove(10)
    println(flag3)	// true
    println(set1)	// Set(12, 78, 13, 53, 23)
    val flag4 = set1.remove(10)
    println(flag4)	// false
    println(set1)	// Set(12, 78, 13, 53, 23)

    println("==================")

    // 4. 合并两个Set
    val set3 = mutable.Set(13, 12, 13, 27, 98, 29)
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 13, 29, 98)

    val set4 = set1 ++ set3
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 13, 29, 98)
    println(set4)	// Set(12, 27, 78, 13, 53, 29, 98, 23)
    
    set3 ++= set1
    println(set1)	// Set(12, 78, 13, 53, 23)
    println(set3)	// Set(12, 27, 78, 13, 53, 29, 98, 23)
  }
}
```

###### Test08_ImmutableMap.scala

> 【集合MAP】

```scala
object Test08_ImmutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: Map[String, Int] = Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)	// Map(a -> 13, b -> 25, hello -> 3)
    println(map1.getClass)	// class scala.collection.immutable.Map$Map3

    println("==========================")
    // 2. 遍历元素
    map1.foreach(println)
    map1.foreach((kv: (String, Int)) => println(kv))

    println("============================")

    // 3. 取map中所有的key 或者 value
    for (key <- map1.keys) {
      println(s"$key ---> ${map1.get(key)}")
    }

    // 4. 访问某一个key的value
    println("a: " + map1.get("a").get)	// a: 13
    println("c: " + map1.get("c"))	// c: None 由于没有 "c" 对应的 value, 如果 map1.get("c").get,会抛异常
    println("c: " + map1.getOrElse("c", 0))	// c: 0

    println(map1("a"))	// 13
  }
}
```

> ```scala
> Map(a -> 13, b -> 25, hello -> 3)
> class scala.collection.immutable.Map$Map3
> ==========================
> (a,13)
> (b,25)
> (hello,3)
> (a,13)
> (b,25)
> (hello,3)
> ============================
> a ---> Some(13)
> b ---> Some(25)
> hello ---> Some(3)
> a: 13
> c: None
> c: 0
> 13
> ```

###### Test09_MutableMap.scala

> 【可变MAP】

```scala
import scala.collection.mutable

object Test09_MutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: mutable.Map[String, Int] = mutable.Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)	// Map(b -> 25, a -> 13, hello -> 3)
    println(map1.getClass)	// class scala.collection.mutable.HashMap

    println("==========================")

    // 2. 添加元素
    map1.put("c", 5)
    map1.put("d", 9)
    println(map1)	// Map(b -> 25, d -> 9, a -> 13, c -> 5, hello -> 3)

    map1 += (("e", 7))
    println(map1)	// Map(e -> 7, b -> 25, d -> 9, a -> 13, c -> 5, hello -> 3)

    println("====================")

    // 3. 删除元素
    println(map1("c"))	// 5
    map1.remove("c")
    println(map1.getOrElse("c", 0))	// 0

    map1 -= "d"
    println(map1)	// Map(e -> 7, b -> 25, a -> 13, hello -> 3)

    println("====================")

    // 4. 修改元素
    map1.update("c", 5)
    map1.update("e", 10)
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)

    println("====================")

    // 5. 合并两个Map
    val map2: Map[String, Int] = Map("aaa" -> 11, "b" -> 29, "hello" -> 5)
    //    map1 ++= map2
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)
    println(map2)	// Map(aaa -> 11, b -> 29, hello -> 5)

    println("---------------------------")
    val map3: Map[String, Int] = map2 ++ map1	// map1 中的值 添加到 map2 中并更新修改成为 map3
    println(map1)	// Map(e -> 10, b -> 25, a -> 13, c -> 5, hello -> 3)
    println(map2)	// Map(aaa -> 11, b -> 29, hello -> 5)
    println(map3)	// Map(e -> 10, a -> 13, b -> 25, c -> 5, hello -> 3, aaa -> 11)
  }
}
```

Test10_Tuple.scala

> 【元组】

```scala
object Test10_Tuple {
  def main(args: Array[String]): Unit = {
    // 1. 创建元组
    val tuple: (String, Int, Char, Boolean) = ("hello", 100, 'a', true)
    println(tuple)	// (hello,100,a,true)

    // 2. 访问数据
    println(tuple._1)	// hello
    println(tuple._2)	// 100
    println(tuple._3)	// a
    println(tuple._4)	// true

    println(tuple.productElement(1))	// 100

    println("====================")
    // 3. 遍历元组数据
    for (elem <- tuple.productIterator)
      println(elem)

    println("====================")
    // 4. 嵌套元组
    val mulTuple = (12, 0.3, "hello", (23, "scala"), 29)
    println(mulTuple._4._2)	// scala
  }
}
```

> ```scala
> (hello,100,a,true)
> hello
> 100
> a
> true
> 100
> ====================
> hello
> 100
> a
> true
> ====================
> scala
> ```

###### Test11_CommonOp.scala

> 【集合常用函数】基本属性和常用操作

```scala
object Test11_CommonOp {
  def main(args: Array[String]): Unit = {
    val list = List(1, 3, 5, 7, 2, 89)
    val set = Set(23, 34, 423, 75)

    //    （1）获取集合长度
    println(list.length)	// 6

    //    （2）获取集合大小
    println(set.size)	// 4
    println("====================")

    //    （3）循环遍历
    for (elem <- list)
      println(elem)

    set.foreach(println)

    //    （4）迭代器
    for (elem <- list.iterator) println(elem)

    println("====================")
    //    （5）生成字符串
    println(list)	// List(1, 3, 5, 7, 2, 89)
    println(set)	// Set(23, 34, 423, 75)
    println(list.mkString("--"))	// 1--3--5--7--2--89

    //    （6）是否包含
    println(list.contains(23))	// false
    println(set.contains(23))	// true
  }
}
```

> ```scala
> 6
> 4
> ====================
> 1
> 3
> 5
> 7
> 2
> 89
> 23
> 34
> 423
> 75
> 1
> 3
> 5
> 7
> 2
> 89
> ====================
> List(1, 3, 5, 7, 2, 89)
> Set(23, 34, 423, 75)
> 1--3--5--7--2--89
> false
> true
> ```

###### Test12_DerivedCollection.scala

> 【集合常用函数】衍生集合

```scala
object Test12_DerivedCollection {
  def main(args: Array[String]): Unit = {
    val list1 = List(1, 3, 5, 7, 2, 89)
    val list2 = List(3, 7, 2, 45, 4, 8, 19)

    //    （1）获取集合的头
    println(list1.head)	// 1

    //    （2）获取集合的尾（不是头的就是尾）
    println(list1.tail)	// List(3, 5, 7, 2, 89)

    //    （3）集合最后一个数据
    println(list2.last)	// 19

    //    （4）集合初始数据（不包含最后一个）
    println(list2.init)	// List(3, 7, 2, 45, 4, 8)

    //    （5）反转
    println(list1.reverse)	// List(89, 2, 7, 5, 3, 1)

    //    （6）取前（后）n个元素
    println(list1.take(3))	// List(1, 3, 5)
    println(list1.takeRight(4))	// List(5, 7, 2, 89)

    //    （7）去掉前（后）n个元素
    println(list1.drop(3))	// List(7, 2, 89)
    println(list1.dropRight(4))	// List(1, 3)

    println("=========================")
    //    （8）并集
    val union = list1.union(list2)
    println("union: " + union)	// union: List(1, 3, 5, 7, 2, 89, 3, 7, 2, 45, 4, 8, 19)
    println(list1 ::: list2)	// List(1, 3, 5, 7, 2, 89, 3, 7, 2, 45, 4, 8, 19)

    // 如果是set做并集，会去重
    val set1 = Set(1, 3, 5, 7, 2, 89)
    val set2 = Set(3, 7, 2, 45, 4, 8, 19)

    val union2 = set1.union(set2)
    println("union2: " + union2)	// union2: Set(5, 89, 1, 2, 45, 7, 3, 8, 19, 4)
    println(set1 ++ set2)	// Set(5, 89, 1, 2, 45, 7, 3, 8, 19, 4)

    println("-----------------------")
    //    （9）交集
    val intersection = list1.intersect(list2)
    println("intersection: " + intersection)	// intersection: List(3, 7, 2)

    println("-----------------------")
    //    （10）差集
    val diff1 = list1.diff(list2)	// 属于 list1, 不属于 list2
    val diff2 = list2.diff(list1)	// 属于 list2, 不属于 list1
    println("diff1: " + diff1)	// diff1: List(1, 5, 89)
    println("diff2: " + diff2)	// diff2: List(45, 4, 8, 19)

    println("-----------------------")
    //    （11）拉链
    println("zip: " + list1.zip(list2))	// zip: List((1,3), (3,7), (5,2), (7,45), (2,4), (89,8))
    println("zip: " + list2.zip(list1))	// zip: List((3,1), (7,3), (2,5), (45,7), (4,2), (8,89))

    println("-----------------------")
    //    （12）滑窗
    for (elem <- list1.sliding(3))
      println(elem)
    println("-----------------------")

    for (elem <- list2.sliding(4, 2))
      println(elem)

    println("-----------------------")
    for (elem <- list2.sliding(3, 3))	// 滚动窗口
      println(elem)
  }
}
```

> ```scala
> // 滑窗
> List(1, 3, 5)
> List(3, 5, 7)
> List(5, 7, 2)
> List(7, 2, 89)
> -----------------------
> List(3, 7, 2, 45)
> List(2, 45, 4, 8)
> List(4, 8, 19)
> -----------------------
> List(3, 7, 2)
> List(45, 4, 8)
> List(19)
> ```

###### Test13_SimpleFunction.scala

> 【集合常用函数】集合计算简单函数

```scala
object Test13_SimpleFunction {
  def main(args: Array[String]): Unit = {
    val list = List(5, 1, 8, 2, -3, 4)
    val list2 = List(("a", 5), ("b", 1), ("c", 8), ("d", 2), ("e", -3), ("f", 4))

    //    （1）求和
    var sum = 0
    for (elem <- list) {
      sum += elem
    }
    println(sum)	// 17

    println(list.sum)	// 17

    //    （2）求乘积
    println(list.product)	// -960

    //    （3）最大值
    println(list.max)	// 8
    println(list2.maxBy((tuple: (String, Int)) => tuple._2))	// (c,8)
    println(list2.maxBy(_._2))	// (c,8)

    //    （4）最小值
    println(list.min)	// -3
    println(list2.minBy(_._2))	// (e,-3)

    println("========================")

    //    （5）排序
    // 5.1 sorted
    val sortedList = list.sorted
    println(sortedList)	// List(-3, 1, 2, 4, 5, 8)

    // 从大到小逆序排序
    println(list.sorted.reverse)	// List(8, 5, 4, 2, 1, -3) 不推荐
    // 传入隐式参数
    println(list.sorted(Ordering[Int].reverse))	// List(8, 5, 4, 2, 1, -3)

    println(list2.sorted)	// List((a,5), (b,1), (c,8), (d,2), (e,-3), (f,4))

    // 5.2 sortBy
    println(list2.sortBy(_._2))	// List((e,-3), (b,1), (d,2), (f,4), (a,5), (c,8)) 默认从小到大
    println(list2.sortBy(_._2)(Ordering[Int].reverse))	// List((c,8), (a,5), (f,4), (d,2), (b,1), (e,-3)) 从大到小

    // 5.3 sortWith
    println(list.sortWith((a: Int, b: Int) => {
      a < b
    }))	// List(-3, 1, 2, 4, 5, 8)
    println(list.sortWith(_ < _))	// List(-3, 1, 2, 4, 5, 8)
    println(list.sortWith(_ > _))	// List(8, 5, 4, 2, 1, -3)
  }
}
```

###### Test12_DerivedCollection.scala

> 【集合常用函数】集合计算高级函数
>
> （1）过滤：遍历一个集合并从中获取满足指定条件的元素组成一个新的集合
>
> （2）转化/映射（map）：将集合中的每一个元素映射到某一个函数
>
> （3）扁平化
>
> （4）扁平化+映射：集合中的每个元素的子元素映射到某个函数并返回新集合，flatMap 相当于先进行 map 操作，再进行 flatten 操作
>
> （5）分组（group）：按照指定的规则对集合的元素进行分组
>
> （6）简化（归约）
>
> （7）折叠

```scala
object Test14_HighLevelFunction_Map {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    // 1. 过滤
    // 选取偶数
    val evenList = list.filter((elem: Int) => {
      elem % 2 == 0
    })
    println(evenList)	// List(2, 4, 6, 8)

    // 选取奇数
    println(list.filter(_ % 2 == 1))	// List(1, 3, 5, 7, 9)

    println("=======================")

    // 2. 映射map
    // 把集合中每个数乘2
    println(list.map(_ * 2))	// List(2, 4, 6, 8, 10, 12, 14, 16, 18)
    println(list.map(x => x * x))	// List(1, 4, 9, 16, 25, 36, 49, 64, 81) 平方

    println("=======================")

    // 3. 扁平化
    val nestedList: List[List[Int]] = List(List(1, 2, 3), List(4, 5), List(6, 7, 8, 9))

    val flatList = nestedList(0) ::: nestedList(1) ::: nestedList(2)
    println(flatList)	// List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    val flatList2 = nestedList.flatten
    println(flatList2)	// List(1, 2, 3, 4, 5, 6, 7, 8, 9)

    println("=======================")

    // 4. 扁平映射
    // 将一组字符串进行分词，并保存成单词的列表
    val strings: List[String] = List("hello world", "hello scala", "hello java", "we study")
    val splitList: List[Array[String]] = strings.map(_.split(" ")) // 分词
    val flattenList = splitList.flatten // 打散扁平化

    println(flattenList)	// List(hello, world, hello, scala, hello, java, we, study)

    val flatmapList = strings.flatMap(_.split(" "))
    println(flatmapList)	// List(hello, world, hello, scala, hello, java, we, study)

    println("========================")

    // 5. 分组groupBy
    // 分成奇偶两组
    val groupMap: Map[Int, List[Int]] = list.groupBy(_ % 2)
    val groupMap2: Map[String, List[Int]] = list.groupBy(data => if (data % 2 == 0) "偶数" else "奇数")

    println(groupMap)	// Map(1 -> List(1, 3, 5, 7, 9), 0 -> List(2, 4, 6, 8))
    println(groupMap2)	// Map(奇数 -> List(1, 3, 5, 7, 9), 偶数 -> List(2, 4, 6, 8))

    // 给定一组词汇，按照单词的首字母进行分组
    val wordList = List("china", "america", "alice", "canada", "cary", "bob", "japan")
    println(wordList.groupBy(_.charAt(0)))	// Map(c -> List(china, canada, cary), a -> List(america, alice), b -> List(bob), j -> List(japan))
  }
}
```

###### Test15_HighLevelFunction_Reduce.scala

> 【集合常用函数】集合计算高级函数
>
> （6）简化（归约）
>
> （7）折叠

```scala
object Test15_HighLevelFunction_Reduce {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4)

    // 1. reduce
    println(list.reduce(_ + _))	// 10
    println(list.reduceLeft(_ + _))	// 10
    println(list.reduceRight(_ + _))	// 10

    println("===========================")

    val list2 = List(3, 4, 5, 8, 10)
    println(list2.reduce(_ - _))	// -24
    println(list2.reduceLeft(_ - _))	// -24
    println(list2.reduceRight(_ - _)) // 3 - (4 - (5 - (8 - 10))), 6

    println("===========================")
    // 2. fold
    println(list.fold(10)(_ + _)) // 10 + 1 + 2 + 3 + 4, 20
    println(list.foldLeft(10)(_ - _)) // 10 - 1 - 2 - 3 - 4, 0
    println(list2.foldRight(11)(_ - _)) // 3 - (4 - (5 - (8 - (10 - 11)))), -5
  }
}
```

###### Test16_MergeMap.scala

> 【集合常用函数】MergeMap

```scala
import scala.collection.mutable

object Test16_MergeMap {
  def main(args: Array[String]): Unit = {
    val map1 = Map("a" -> 1, "b" -> 3, "c" -> 6)
    val map2 = mutable.Map("a" -> 6, "b" -> 2, "c" -> 9, "d" -> 3)

    //    println(map1 ++ map2)

    val map3 = map1.foldLeft(map2)(
      (mergedMap, kv) => {
        val key = kv._1
        val value = kv._2
        mergedMap(key) = mergedMap.getOrElse(key, 0) + value
        mergedMap
      }
    )

    println(map3)	// Map(b -> 5, d -> 3, a -> 7, c -> 15)
  }
}
```

> mergedMap就是map2，kv是map1的元素，把map1的每个元素融合到mergeMap中

###### Test17_CommonWordCount.scala

> 【集合常用函数】WordCount

```scala
object Test17_CommonWordCount {
  def main(args: Array[String]): Unit = {
    val stringList: List[String] = List(
      "hello",
      "hello world",
      "hello scala",
      "hello spark from scala",
      "hello flink from scala"
    )

    // 1. 对字符串进行切分，得到一个打散所有单词的列表
    //    val wordList1: List[Array[String]] = stringList.map(_.split(" "))
    //    val wordList2: List[String] = wordList1.flatten
    //    println(wordList2)
    val wordList = stringList.flatMap(_.split(" "))
    println(wordList)

    // 2. 相同的单词进行分组
    val groupMap: Map[String, List[String]] = wordList.groupBy(word => word)
    println(groupMap)

    // 3. 对分组之后的list取长度，得到每个单词的个数
    val countMap: Map[String, Int] = groupMap.map(kv => (kv._1, kv._2.length))

    // 4. 将map转换为list，并排序取前3
    val sortList: List[(String, Int)] = countMap.toList
      .sortWith(_._2 > _._2)
      .take(3)

    println(sortList)
  }
}
```

> ```scala
> List(hello, hello, world, hello, scala, hello, spark, from, scala, hello, flink, from, scala)
> Map(world -> List(world), flink -> List(flink), spark -> List(spark), scala -> List(scala, scala, scala), from -> List(from, from), hello -> List(hello, hello, hello, hello, hello))
> List((hello,5), (scala,3), (from,2))
> ```

###### Test18_ComplexWordCount.scala

> 【集合常用函数】WordCount复杂版

```scala
object Test18_ComplexWordCount {
  def main(args: Array[String]): Unit = {
    val tupleList: List[(String, Int)] = List(
      ("hello", 1),
      ("hello world", 2),
      ("hello scala", 3),
      ("hello spark from scala", 1),
      ("hello flink from scala", 2)
    )

    // 思路一：直接展开为普通版本
    val newStringList: List[String] = tupleList.map(
      kv => {
        (kv._1.trim + " ") * kv._2
      }
    )
    println(newStringList)

    // 接下来操作与普通版本完全一致
    val wordCountList: List[(String, Int)] = newStringList
      .flatMap(_.split(" ")) // 空格分词
      .groupBy(word => word) // 按照单词分组
      .map(kv => (kv._1, kv._2.size)) // 统计出每个单词的个数
      .toList
      .sortBy(_._2)(Ordering[Int].reverse)
      .take(3)

    println(wordCountList)

    println("================================")

    // 思路二：直接基于预统计的结果进行转换
    // 1. 将字符串打散为单词，并结合对应的个数包装成二元组
    val preCountList: List[(String, Int)] = tupleList.flatMap(
      tuple => {
        val strings: Array[String] = tuple._1.split(" ")
        strings.map(word => (word, tuple._2))
      }
    )
    println(preCountList)

    // 2. 对二元组按照单词进行分组
    val preCountMap: Map[String, List[(String, Int)]] = preCountList.groupBy(_._1)
    println(preCountMap)

    // 3. 叠加每个单词预统计的个数值
    val countMap: Map[String, Int] = preCountMap.mapValues(
      tupleList => tupleList.map(_._2).sum
    )
    println(countMap)

    // 4. 转换成list，排序取前3
    val countList = countMap.toList
      .sortWith(_._2 > _._2)
      .take(3)
    println(countList)
  }
}
```

> ```scala
> List(hello , hello world hello world , hello scala hello scala hello scala , hello spark from scala , hello flink from scala hello flink from scala )
> List((hello,9), (scala,6), (from,3))
> ================================
> List((hello,1), (hello,2), (world,2), (hello,3), (scala,3), (hello,1), (spark,1), (from,1), (scala,1), (hello,2), (flink,2), (from,2), (scala,2))
> Map(world -> List((world,2)), flink -> List((flink,2)), spark -> List((spark,1)), scala -> List((scala,3), (scala,1), (scala,2)), from -> List((from,1), (from,2)), hello -> List((hello,1), (hello,2), (hello,3), (hello,1), (hello,2)))
> Map(world -> 2, flink -> 2, spark -> 1, scala -> 6, from -> 3, hello -> 9)
> List((hello,9), (scala,6), (from,3))
> ```

###### Test19_Queue.scala

> 【队列Queue】

```scala
import scala.collection.immutable.Queue
import scala.collection.mutable

object Test19_Queue {
  def main(args: Array[String]): Unit = {
    // 创建一个可变队列
    val queue: mutable.Queue[String] = new mutable.Queue[String]()

    queue.enqueue("a", "b", "c")

    println(queue)	// Queue(a, b, c)
    println(queue.dequeue())	// a
    println(queue)	// Queue(b, c)
    println(queue.dequeue())	// b
    println(queue)	// Queue(c)

    queue.enqueue("d", "e")

    println(queue)	// Queue(c, d, e)
    println(queue.dequeue())	// c
    println(queue)	// Queue(d, e)

    println("==========================")

    // 不可变队列
    val queue2: Queue[String] = Queue("a", "b", "c")
    val queue3 = queue2.enqueue("d")	// queue2 未变
    println(queue2)	// Queue(a, b, c)
    println(queue3)	// Queue(a, b, c, d)

  }
}
```

###### Test20_Parallel.scala

> 【并行集合】

```scala
import scala.collection.immutable
import scala.collection.parallel.immutable.ParSeq

object Test20_Parallel {
  def main(args: Array[String]): Unit = {
    // 串行
    val result: immutable.IndexedSeq[Long] = (1 to 100).map(
      x => Thread.currentThread.getId
    )
    println(result)

    val result2: ParSeq[Long] = (1 to 100).par.map(
      x => Thread.currentThread.getId
    )
    println(result2)
  }
}
```

> ```scala
> Vector(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1)
> ParVector(11, 11, 11, 11, 11, 11, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 13, 13, 13, 13, 13, 13, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 14, 14, 14, 14, 14, 14, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12, 12)
> ```

##### 模式匹配

> Scala 中的模式匹配类似于 Java 中的 Switch 语法

###### Test01_PatternMatchBase.scala

> 【基本语法/模式守卫】
>
> （1）如果所有 case 都不匹配，那么会执行 case _ 分支，类似于 Java 中 default 语句，若此时没有 case _ 分支，那么会抛出 MatchError
> （2）每个 case 中，不需要使用 break 语句，自动中断 case 
> （3）match case 语句可以匹配任何类型，而不只是字面量
> （4）=> 后面的代码块，直到下一个 case 语句之前的代码是作为一个整体执行，可以使用{}括起来，也可以不括
>
> 模式守卫：如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫。

```scala
object Test01_PatternMatchBase {
  def main(args: Array[String]): Unit = {
    // 1. 基本定义语法
    val x: Int = 5
    val y: String = x match {
      case 1 => "one"
      case 2 => "two"
      case 3 => "three"
      case _ => "other"
    }
    println(y)	// other

    // 2. 示例：用模式匹配实现简单二元运算
    val a = 25
    val b = 13

    def matchDualOp(op: Char): Int = op match {
      case '+' => a + b
      case '-' => a - b
      case '*' => a * b
      case '/' => a / b
      case '%' => a % b
      case _ => -1
    }

    println(matchDualOp('+'))	// 38
    println(matchDualOp('/'))	// 1
    println(matchDualOp('\\'))	// -1

    println("=========================")

    // 3. 模式守卫
    // 求一个整数的绝对值
    def abs(num: Int): Int = {
      num match {
        case i if i >= 0 => i
        case i if i < 0 => -i
      }
    }

    println(abs(67))	// 67
    println(abs(0))	// 0
    println(abs(-24))	// 24
  }
}
```

###### Test02_MatchTypes.scala

> 【模式匹配类型】
>
> Scala 中，模式匹配可以匹配所有的字面量，包括字符串，字符，数字，布尔值等等
>
> 匹配常量、匹配类型、匹配数组、匹配列表、匹配元组

```scala
object Test02_MatchTypes {
  def main(args: Array[String]): Unit = {
    // 1. 匹配常量
    def describeConst(x: Any): String = x match {
      case 1 => "Int one"
      case "hello" => "String hello"
      case true => "Boolean true"
      case '+' => "Char +"
      case _ => ""
    }

    println(describeConst("hello"))	// String hello
    println(describeConst('+'))	// Char +
    println(describeConst(0.3))	// 输出为空

    println("==================================")

    // 2. 匹配类型
    def describeType(x: Any): String = x match {
      case i: Int => "Int " + i
      case s: String => "String " + s
      case list: List[String] => "List " + list
      case array: Array[Int] => "Array[Int] " + array.mkString(",")
      case a => "Something else: " + a
    }

    println(describeType(35))	// Int 35
    println(describeType("hello"))	// String hello
    println(describeType(List("hi", "hello")))	// List List(hi, hello)
    println(describeType(List(2, 23)))	// List List(2, 23) 泛型擦除?
    println(describeType(Array("hi", "hello")))	// Something else: [Ljava.lang.String;@7dc36524
    println(describeType(Array(2, 23)))	// Array[Int] 2,23

    // 3. 匹配数组
    for (arr <- List(
      Array(0),
      Array(1, 0),
      Array(0, 1, 0),
      Array(1, 1, 0),
      Array(2, 3, 7, 15),
      Array("hello", 1, 30),
    )) {
      val result = arr match {
        case Array(0) => "0"
        case Array(1, 0) => "Array(1, 0)"
        case Array(x, y) => "Array: " + x + ", " + y // 匹配两元素数组
        case Array(0, _*) => "以0开头的数组"
        case Array(x, 1, z) => "中间为1的三元素数组"
        case _ => "something else"
      }

      println(result)
    }

    println("=========================")

    // 4. 匹配列表
    // 方式一
    for (list <- List(
      List(0),
      List(1, 0),
      List(0, 0, 0),
      List(1, 1, 0),
      List(88),
      List("hello")
    )) {
      val result = list match {
        case List(0) => "0"
        case List(x, y) => "List(x, y): " + x + ", " + y
        case List(0, _*) => "List(0, ...)"
        case List(a) => "List(a): " + a
        case _ => "something else"
      }
      println(result)
    }

    // 方式二
    val list = List(1, 2, 5, 7, 24)
    // val list1 = List(24)	// something else

    list match {
      case first :: second :: rest => println(s"first: $first, second: $second, rest: $rest")
      case _ => println("something else")
    }

    println("===========================")
    // 5. 匹配元组
    for (tuple <- List(
      (0, 1),
      (0, 0),
      (0, 1, 0),
      (0, 1, 1),
      (1, 23, 56),
      ("hello", true, 0.5)
    )) {
      val result = tuple match {
        case (a, b) => "" + a + ", " + b
        case (0, _) => "(0, _)"
        case (a, 1, _) => "(a, 1, _) " + a
        case (x, y, z) => "(x, y, z) " + x + " " + y + " " + z
        case _ => "something else"
      }
      println(result)
    }
  }
}
```

> ```scala
> // 匹配数组
> 0
> Array(1, 0)
> 以0开头的数组
> 中间为1的三元素数组
> something else
> 中间为1的三元素数组
> // 匹配列表
> // 方式一
> 0
> List(x, y): 1, 0
> List(0, ...)
> something else
> List(a): 88
> List(a): hello
> // 方式二
> first: 1, second: 2, rest: List(5, 7, 24)
> // 匹配元组
> 0, 1
> 0, 0
> (a, 1, _) 0
> (a, 1, _) 0
> (x, y, z) 1 23 56
> (x, y, z) hello true 0.5
> ```

###### Test03_MatchTupleExtend.scala

> 【增强元组匹配】

```scala
object Test03_MatchTupleExtend {
  def main(args: Array[String]): Unit = {
    // 1. 在变量声明时匹配
    val (x, y) = (10, "hello")
    println(s"x: $x, y: $y")	// x: 10, y: hello

    val List(first, second, _*) = List(23, 15, 9, 78)
    println(s"first: $first, second: $second")	// first: 23, second: 15

    val fir :: sec :: rest = List(23, 15, 9, 78)
    println(s"first: $fir, second: $sec, rest: $rest")	// first: 23, second: 15, rest: List(9, 78)

    println("=====================")

    // 2. for推导式中进行模式匹配
    val list: List[(String, Int)] = List(("a", 12), ("b", 35), ("c", 27), ("a", 13))

    // 2.1 原本的遍历方式
    for (elem <- list) {
      println(elem._1 + " " + elem._2)
    }

    // 2.2 将List的元素直接定义为元组，对变量赋值
    for ((word, count) <- list) {
      println(word + ": " + count)
    }

    println("-----------------------")

    // 2.3 可以不考虑某个位置的变量，只遍历key或者value
    for ((word, _) <- list)
      println(word)

    println("-----------------------")

    // 2.4 可以指定某个位置的值必须是多少
    for (("a", count) <- list) {
      println(count)
    }
  }
}
```

> ```scala
> x: 10, y: hello
> first: 23, second: 15
> first: 23, second: 15, rest: List(9, 78)
> =====================
> a 12
> b 35
> c 27
> a 13
> a: 12
> b: 35
> c: 27
> a: 13
> -----------------------
> a
> b
> c
> a
> -----------------------
> 12
> 13
> ```

###### Test04_MatchObject.scala

> 匹配对象

```scala
object Test04_MatchObject {
  def main(args: Array[String]): Unit = {
    val student = new Student("alice", 19)

    // 针对对象实例的内容进行匹配
    val result = student match {
      case Student("alice", 18) => "Alice, 18"
      case _ => "Else"
    }

    println(result)	// Else
  }
}

// 定义类
class Student(val name: String, val age: Int)

// 定义伴生对象
object Student {
  def apply(name: String, age: Int): Student = new Student(name, age)

  // 必须实现一个unapply方法，用来对对象属性进行拆解
  def unapply(student: Student): Option[(String, Int)] = {
    if (student == null) {
      None
    } else {
      Some((student.name, student.age))
    }
  }
}
```

###### Test05_MatchCaseClass.scala

> 匹配样例类

```scala
object Test05_MatchCaseClass {
  def main(args: Array[String]): Unit = {
    val student = Student1("alice", 18)

    // 针对对象实例的内容进行匹配
    val result = student match {
      case Student1("alice", 18) => "Alice, 18"
      case _ => "Else"
    }

    println(result)	// Alice, 18
  }
}

// 定义样例类
case class Student1(name: String, age: Int)
```

###### Test06_PartialFunction.scala

> 【偏函数中的模式匹配】

```scala
object Test06_PartialFunction {
  def main(args: Array[String]): Unit = {
    val list: List[(String, Int)] = List(("a", 12), ("b", 35), ("c", 27), ("a", 13))

    // 1. map转换，实现key不变，value2倍
    val newList = list.map(tuple => (tuple._1, tuple._2 * 2))

    // 2. 用模式匹配对元组元素赋值，实现功能
    val newList2 = list.map(
      tuple => {
        tuple match {
          case (word, count) => (word, count * 2)
        }
      }
    )

    // 3. 省略lambda表达式的写法，进行简化
    val newList3 = list.map {
      case (word, count) => (word, count * 2)
    }

    println(newList)	// List((a,24), (b,70), (c,54), (a,26))
    println(newList2)	// List((a,24), (b,70), (c,54), (a,26))
    println(newList3)	// List((a,24), (b,70), (c,54), (a,26))

    // 偏函数的应用，求绝对值
    // 对输入数据分为不同的情形：正、负、0
    val positiveAbs: PartialFunction[Int, Int] = {
      case x if x > 0 => x
    }
    val negativeAbs: PartialFunction[Int, Int] = {
      case x if x < 0 => -x
    }
    val zeroAbs: PartialFunction[Int, Int] = {
      case 0 => 0
    }

    def abs(x: Int): Int = (positiveAbs orElse negativeAbs orElse zeroAbs) (x)

    println(abs(-67))	// 67
    println(abs(35))	// 35
    println(abs(0))	// 0
  }
}
```

##### 异常-隐式转换-泛型

> Scala 的异常的工作机制和 Java 一样，但是 Scala 没有“ checked （编译期）”异常，即 Scala 没有编译异常这个概念，异常都是在运行的时候捕获处理。
>
> 用 throw 关键字，抛出一个异常对象。所有异常都是 Throwable 的子类型。throw 表达式是有类型的，就是 Nothing ，因为 Nothing 是所有类型的子类型，所以 throw 表达式可以用在需要类型的地方

###### Test01_Exception.scala

> 【异常处理】

```scala
object Test01_Exception {
  def main(args: Array[String]): Unit = {
    try{
      val n = 10 / 0
    } catch {
      case e: ArithmeticException => {
        println("发生算术异常")
      }
      case e: Exception => {
        println("发生一般异常")
      }
    } finally {
      println("处理结束")
    }
  }
}
```

> ```scala
> 发生算术异常
> 处理结束
> ```

###### Test02_Implicit.scala

> 【隐式转换】隐式函数、隐式类、隐式参数
>
> 当编译器第一次编译失败的时候，会在当前的环境中查找能让代码编译通过的方法，用于将类型进行转换，实现二次编译
>
> 隐式解析机制：
>
> （1）首先会在当前代码作用域下查找隐式实体（隐式方法、隐式类、隐式对象）。 （一般是这种情况）
> （2）如果第一条规则查找隐式实体失败，会继续在隐式参数的类型的作用域里查找。类型的作用域是指与 该类型相关联的全部伴生对象以及该类型所在包的包对象。
>
> 隐式函数：隐式转换可以在不需改任何代码的情况下，扩展某个类的功能
>
> 隐式参数：普通方法或者函数中的参数可以通过 implicit 关键字声明为隐式参数，调用该方法时，就可以传入该参数，编译器会在相应的作用域寻找符合条件的隐式值。
>
> （1）同一个作用域中，相同类型的隐式值只能有一个
> （2）编译器按照隐式参数的类型去寻找对应类型的隐式值，与隐式值的名称无关
> （3）隐式参数优先于默认参数
>
> 隐式类：在Scala2.10 后提供了隐式类，可以使用 implicit 声明类，隐式类的非常强大，同样可以扩展类的功能，在集合中隐式类会发挥重要的作用
>
> （1）其所带的构造参数有且只能有一个
> （2）隐式类必须被定义在“类”或“伴生对象”或“包对象”里，即隐式类不能是顶级的

```scala
object Test02_Implicit {
  def main(args: Array[String]): Unit = {

    val new12 = new MyRichInt(12)
    println(new12.myMax(15))	// 15

    // 1. 隐式函数
    implicit def convert(num: Int): MyRichInt = new MyRichInt(num)

    println(12.myMax(15))	// 15

    println("============================")

    // 2. 隐式类
    implicit class MyRichInt2(val self: Int) {
      // 自定义比较大小的方法
      def myMax2(n: Int): Int = if (n < self) self else n

      def myMin2(n: Int): Int = if (n < self) n else self
    }

    println(12.myMin2(15))	// 12

    println("============================")

    // 3. 隐式参数

    implicit val str: String = "alice"
    //    implicit val str2: String = "alice2"	// 同一个作用域中，相同类型的隐式值只能有一个
    implicit val num: Int = 18

    def sayHello()(implicit name: String): Unit = {
      println("hello, " + name)
    }

    def sayHi(implicit name: String = "ahua"): Unit = {	// 隐式参数优先于默认参数
      println("hi, " + name)
    }

    sayHello
    sayHi

    // 简便写法
    def hiAge(): Unit = {
      println("hi, " + implicitly[Int])
    }

    hiAge()
  }
}

// 自定义类
class MyRichInt(val self: Int) {
  // 自定义比较大小的方法
  def myMax(n: Int): Int = if (n < self) self else n

  def myMin(n: Int): Int = if (n < self) n else self
}
```

> ```scala
> // 隐式参数
> hello, alice
> hi, alice
> hi, 18
> ```

###### Test03_Generics.scala

> 【泛型】

```scala
object Test03_Generics {
  def main(args: Array[String]): Unit = {
    // 1. 协变和逆变
    val child: Parent = new Child
    //    val childList: MyCollection[Parent] = new MyCollection[Child]
    val childList: MyCollection[SubChild] = new MyCollection[Child]

    // 2. 上下限
    def test[A <: Child](a: A): Unit = {
      println(a.getClass.getName)
    }

    test[SubChild](new SubChild)
  }
}

// 定义继承关系
class Parent {}

class Child extends Parent {}

class SubChild extends Child {}

// 定义带泛型的集合类型
class MyCollection[-E] {}	// 逆变
```

