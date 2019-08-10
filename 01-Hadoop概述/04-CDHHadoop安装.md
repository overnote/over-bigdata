## 环境简介

```
服务器IP	    192.168.120.111	    192.168.120.112	    192.168.120.113
主机名	        node01.hadoop.com	node02.hadoop.com	node03.hadoop.com
HDFS            
                NameNode            
                SecondaryNameNode
                DataNode            DataNode            DataNode
YARN            ResourceManger                          
                NodeManager         NodeManager         NodeManager
MapReduce       JobHistoryServer                                         
```

## 一 第一步：安装hadoop

编译 CDH版hadoop
```
# 下载地址 http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.0-src.tar.gz
tar -zxvf hadoop-2.6.0-cdh5.14.0-src.tar.gz -C /usr/local
cd  /usr/local/hadoop-2.6.0-cdh5.14.0

# 编译不支持snappy压缩：
mvn package -Pdist,native -DskipTests –Dtar   

# 编译支持snappy压缩：
mvn package -DskipTests -Pdist,native -Dtar -Drequire.snappy -e -X

# 查看hadoop支持的压缩方式以及本地库
bin/hadoop checknative  
```

注意，如果出现错误`An Ant BuildException has occured: exec returned: 2`,是因为tomcat的压缩包没有下载完成，需要自己下载一个对应版本的apache-tomcat-6.0.53.tar.gz的压缩包放到指定路径下面去即可,这两个路径下面需要放上这个tomcat的压缩包:
```
/usr/local/hadoop-2.6.0-cdh5.14.0/hadoop-hdfs-project/hadoop-hdfs-httpfs/downloads
/usr/local/hadoop-2.6.0-cdh5.14.0/hadoop-common-project/hadoop-kms/downloads
```


## 二 第二步：修改配置文件

#### 2.1 第一台机器修改/etc/core-site.xml

```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node01:8020</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/tempDatas</value>
	</property>
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>			<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	</property>
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>		<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	</property>
</configuration>
```

#### 2.2 修改第一台机器 etc/hadoop/hdfs-site.xml

```xml
<configuration>
	<!-- NameNode存储元数据信息的路径，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割   --> 
	<!--   集群动态上下线 
	<property>
		<name>dfs.hosts</name>
		<value>/export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/accept_host</value>
	</property>
	
	<property>
		<name>dfs.hosts.exclude</name>
		<value>/export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/deny_host</value>
	</property>
	 -->
	 
	 <property>
			<name>dfs.namenode.secondary.http-address</name>
			<value>node01:50090</value>
	</property>

	<property>
		<name>dfs.namenode.http-address</name>
		<value>node01:50070</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/namenodeDatas</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/datanodeDatas</value>
	</property>
	
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/edits</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/snn/edits</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
</configuration>
```

#### 2.3 修改第一台机器 etc/hadoophadoop-env.sh
```
vim hadoop-env.sh
export JAVA_HOME=/usr/local/jdk1.8.0_141
```

#### 2.4 修改第一台机器 etc/hadoop/mapred-site.xml
如果没有该文件，可以修改 mapred-site.xml
```xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>

	<property>
		<name>mapreduce.job.ubertask.enable</name>
		<value>true</value>
	</property>
	
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>node01:10020</value>
	</property>

	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>node01:19888</value>
	</property>
</configuration>
```

#### 2.5 第一台机器修改 etc/hadoop/yarn-site.xml

```xml
<configuration>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>node01</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
</configuration>
```

#### 2.6 修改第一台机器 etc/hadoop/slaves

```
node01
node02
node03
```

## 三 第三步：创建文件目录

node01创建如下目录：
```
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/tempDatas
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/namenodeDatas
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/datanodeDatas 
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/edits
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/snn/name
mkdir -p /usr/local/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/snn/edits
```

## 四 第四步：分发安装包

node01执行：
```
cd /usr/local/
scp -r hadoop-2.6.0-cdh5.14.0/ node02:$PWD
scp -r hadoop-2.6.0-cdh5.14.0/ node03:$PWD
```

## 五 启动

#### 5.1 配置hadoop环境变量

所有机器都执行：
```
vim  /etc/profile
export HADOOP_HOME=/export/servers/hadoop-2.6.0-cdh5.14.0
export PATH=:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
source /etc/profile
```

#### 5.2 启动准备

要启动 Hadoop 集群，需要启动 HDFS 和 YARN 两个集群。  

注意：首次启动HDFS时，必须对其进行格式化操作。本质上是一些清理和准备工作，因为此时的 HDFS 在物理上还是不存在的。  

```
bin/hdfs namenode  -format或者bin/hadoop namenode –format
```

#### 5.3 启动方式一：单节点逐一启动

在主节点上使用以下命令启动 HDFS NameNode：
```
hadoop-daemon.sh start namenode 
```

在每个从节点上使用以下命令启动 HDFS DataNode： 
```
hadoop-daemon.sh start datanode 
```

在主节点上使用以下命令启动 YARN ResourceManager：
``` 
yarn-daemon.sh  start resourcemanager 
```

在每个从节点上使用以下命令启动 YARN nodemanager： 
```
yarn-daemon.sh start nodemanager 
```

以上脚本位于$HADOOP_PREFIX/sbin/目录下。如果想要停止某个节点上某个角色，只需要把命令中的start 改为stop 即可。

#### 5.4 启动方式二：脚本一键启动

如果配置了 etc/hadoop/slaves 和 ssh 免密登录，则可以使用程序脚本启动所有Hadoop 两个集群的相关进程，在主节点所设定的机器上执行。  

启动集群:node01节点上执行以下命令
```
cd /usr/local/hadoop-2.6.0-cdh5.14.0/
sbin/start-dfs.sh
sbin/start-yarn.sh
sbin/mr-jobhistory-daemon.sh start historyserver
```

停止集群:不建议随意停止集群
```
sbin/stop-dfs.sh
sbin/stop-yarn.sh
sbin/mr-jobhistory-daemon.sh start historyserver
```

#### 5.5 浏览器查看启动页面

hdfs初体验：从Linux 本地上传一个文本文件到 hdfs 的/test/input 目录下
```
hadoop fs -mkdir -p /test/input 
hadoop fs -put /root/install.log  /test/input 
```

hdfs集群访问地址：http://192.168.120.111:50070/dfshealth.html#tab-overview  

yarn集群访问地址：http://192.168.120.111:8088/cluster  

jobhistory访问地址：http://192.168.120.111:19888/jobhistory