---
layout: article
title:	Hadoop集群搭建
date:	2018-06-10 16:41:26
categories:
    - article
tags:
    - Hadoop
    - BigData
---


**hadoop版本** hadoop-2.7.5

[hadoop实验配置文件](./conf)

**hadoop配置文件位置** /opt/app/hadoop-2.7.5/etc/hadoop

** 主要修改hadoop文件 **

* /opt/app/hadoop-2.7.5/etc/hadoop/core-site.xml
* /opt/app/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
* /opt/app/hadoop-2.7.5/etc/hadoop/yarn-site.xml
* /opt/app/hadoop-2.7.5/etc/hadoop/mapred-site.xml
* /opt/app/hadoop-2.7.5/etc/hadoop/slaves

### hadoop配置

安装hadoop
~~~shell
cd /opt/app
wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.7.5/hadoop-2.7.5.tar.gz 
tar -zxvf hadoop-2.7.5.tar.gz && rm -rf hadoop-2.7.5.tar.gz
~~~

配置hadoop环境变量

~~~shell
HADOOP_HOME="/opt/app/hadoop-2.7.5"
echo "export HADOOP_HOME=$HADOOP_HOME" >> /etc/profile
source /etc/profile
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
hadoop version
hdfs version
~~~

配置core-site.xml

~~~xml
<configuration>
 <!--hadoop主机端口配置 -->
 <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop_master:9000</value>
 </property>
 <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hadoop.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hadoop.groups</name>
        <value>*</value>
    </property>>
</configuration>
~~~

> [core-site.xml配置官方文档说明](http://hadoop.apache.org/docs/r2.7.5/hadoop-project-dist/hadoop-common/core-default.xml)

配置hdfs-site.xml

~~~xml
<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop-master:9001</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/opt/app/hadoop-2.7.5/data/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/opt/app/hadoop-2.7.5/data/hdfs/datanode</value>
    </property>
    <!-- 节点数量 -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.datanode.max.transfer.threads</name>
        <value>409600</value>
    </property>
    <property>
        <name>dfs.client.use.datanode.hostname</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.datanode.use.datanode.hostname</name>
        <value>true</value>
    </property>
</configuration>
~~~

> [hdfs-site.xml配置官方文档说明](http://hadoop.apache.org/docs/r2.7.5/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)


配置yarn-site.xml

~~~shell
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>hadoop-master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>hadoop-master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>hadoop-master:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>hadoop-master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>hadoop-master:8088</value>
    </property>
	<property>
		<name>yarn.scheduler.minimum-allocation-mb</name>
		<value>2048</value>
	</property>
	<property>
		<name>yarn.nodemanager.vmem-pmem-ratio</name>
		<value>2.1</value>
	</property>
	<property>
		<name>yarn.nodemanager.resource.memory-mb</name>
		<value>20480</value>
	</property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>2</value>
    </property>
</configuration>
~~~

> [yarn-site.xml配置官方文档说明](http://hadoop.apache.org/docs/r2.7.5/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)

配置mapred-site.xml

~~~shell
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop_master:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop_master:19888</value>
    </property>
</configuration>
~~~

> [mapred-site.xml配置官方文档说明](http://hadoop.apache.org/docs/r2.7.5/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)


将配置好的hadoop发送到子节点上（hadoop_node_1）

~~~shell
scp -r /opt/app/hadoop-2.7.5 root@hadoop_node_1:/opt/app/hadoop-2.7.5
scp -r /opt/app/hadoop-2.7.5 root@hadoop_node_2:/opt/app/hadoop-2.7.5
~~~

将子节点加入集群中

~~~shell
cat << EOF > /opt/app/hadoop-2.7.5/etc/hadoop/slaves
hadoop-node-1
hadoop-node-2
EOF
~~~


### 启动hadoop

初始化HDFS（主节点）

~~~shell
mkdir -p /opt/app/hadoop-2.7.5/hdfsdata/hdfs/namenode /opt/app/hadoop-2.7.5/hdfsdata/hdfs/datanode
/opt/app/hadoop-2.7.5/bin/hdfs namenode -format
~~~

![](./img/1.png)

启动hadoop

~~~shell
/opt/app/hadoop-2.7.5/sbin/start-all.sh
~~~

查看下是否启动成功

~~~xml
[root@hadoop_master logs]# jps
4695 SecondaryNameNode
5097 Jps
4508 NameNode
4846 ResourceManager
~~~

> 主节点已经启动了 **SecondaryNameNode、NameNode、ResourceManager**

~~~xml
[root@hadoop_node_1 logs]# jps
1939 Jps
1756 DataNode
~~~

> 主节点已经启动了 **DataNode**

到这说明hadoop集群已经搭建完成了

页面访问namenode
[http://192.168.56.110:50070](http://192.168.56.110:50070)

![](./img/2.jpg)

页面访问datanode
[http://192.168.56.111:50075](http://192.168.56.111:50075)
[http://192.168.56.112:50075](http://192.168.56.112:50075)

页面访问yarn
[http://192.168.56.110:8088](http://192.168.56.110:8088)

停止hadoop

~~~shell
/opt/app/hadoop-2.7.5/sbin/stop-all.sh
~~~