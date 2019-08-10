## 安装前奏说明

hadoop文档：http://hadoop.apache.org/docs/  
下载地址： http://archive.apache.org/dist/hadoop/common/hadoop-2.7.5/hadoop-2.7.5.tar.gz  

hadoop安装包结构
- bin:一些shell脚本，供我们使用
- sbin:一些shell脚本，供我们使用
- etc/hadoop:所有的配置文件的路径
- lib/native:本地的C程序库

本地压缩库支持：zlib、lz4、bzip2，不支持snappy，但是snappy是谷歌出品的高性能压缩算法，需要我们自己安装一些库来支持该算法。  

hadoop 留个核心配置文件的作用：
- core-site.xml：核心配置文件，主要定义了我们文件访问的格式  hdfs://
- hadoop-env.sh：主要配置我们的java路径
- hdfs-site.xml：主要定义配置我们的hdfs的相关配置
- mapred-site.xml  主要定义我们的mapreduce相关的一些配置，没有这个文件的话会有mapred-site.xml.template，修改名字即可
- slaves：控制我们的从节点在哪里 datanode   nodemanager在哪些机器上
- yarn-site.xml：配置我们的resourcemanager资源调度

#### 1.0 环境说明

```
服务器IP	    192.168.120.111	    192.168.120.112	    192.168.120.113
主机名	        node01.hadoop.com	node02.hadoop.com	node03.hadoop.com
NameNode	    是	                否	                否
SecondaryNameNode 是	            否	                否
dataNode	    是	                是	                是
ResourceManager	是	                否	                否
NodeManager	    是	                是	                是
```


下载安装：
```
tar -zxvf hadoop-2.7.5.tar.gz -C /usr/local/
```

#### 1.1 修改第一台机器core-site.xml
```xml
<configuration>
	<property>
		<name>fs.default.name</name>
		<value>hdfs://192.168.120.111:8020</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop-2.7.5/hadoopDatas/tempDatas</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>

	<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>
	</property>
</configuration>
```

#### 1.2 修改第一台机器hdfs-site.xml
```xml
<configuration>
	<!-- NameNode存储元数据信息的路径，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割   --> 
	<!--   集群动态上下线 
	<property>
		<name>dfs.hosts</name>
		<value>/export/servers/hadoop-2.7.4/etc/hadoop/accept_host</value>
	</property>
	
	<property>
		<name>dfs.hosts.exclude</name>
		<value>/export/servers/hadoop-2.7.4/etc/hadoop/deny_host</value>
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
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas,file:///export/servers/hadoop-2.7.5/hadoopDatas/namenodeDatas2</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas,file:///export/servers/hadoop-2.7.5/hadoopDatas/datanodeDatas2</value>
	</property>
	
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/nn/edits</value>
	</property>
	

	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file://usr/local/hadoop-2.7.5/hadoopDatas/dfs/snn/edits</value>
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

#### 1.3 修改第一台机器hadoop-env.sh

```
export JAVA_HOME=/usr/local/jdk1.8.0_141
```

#### 1.4 修改第一台机器mapred-site.xml

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

#### 1.4 修改第一台机器yarn-site.xml

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

#### 1.5 修改第一台机器slaves--与单机方式唯一区别

```
cd  /usr/local/hadoop-2.7.5/etc/hadoop
vim   slaves
node01
node02
node03
```

#### 1.6 分发配置
```
cd  /usr/local/
scp -r hadoop-2.7.5 node02:$PWD
scp -r hadoop-2.7.5 node03:$PWD
```

## 二 启动集群

要启动 Hadoop 集群，需要启动 HDFS 和 YARN 两个模块。不过首次启动 HDFS 时，必须对其进行格式化操作。 本质上是一些清理和准备工作，因为此时的 HDFS 在物理上还是不存在的：
```
hdfs namenode -format 或者 hadoop namenode –format
```

创建数据文件(注意：如果之前有其他方式安装产生了下列文件夹，需要删除文件夹再次生成)：
```
cd  /usr/local/hadoop-2.7.5
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/tempDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas2
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas2
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/nn/edits
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/snn/name
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/dfs/snn/edits
```

在第一台机器上启动集群：
```
cd  /usr/local/hadoop-2.7.5/
bin/hdfs namenode -format
sbin/start-dfs.sh
sbin/start-yarn.sh
sbin/mr-jobhistory-daemon.sh start historyserver
```

查看web控制台：
```
http://node01:50070/explorer.html#/  	查看hdfs
http://node01:8088/cluster   			查看yarn集群
http://node01:19888/jobhistory  		查看历史完成的任务
```

服务停止命令：
```
cd /usr/local/hadoop-2.7.5
sbin/stop-dfs.sh
sbin/stop-yarn.sh
sbin/mr-jobhistory-daemon.sh stop historyserver
```