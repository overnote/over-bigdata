## 一 三台机器的Linux安装准备

### 1.1 安装后的集群设置

- node01功能： 
  - HDFS：NameNode，SecondaryNameNode，DataNode
  - YARN: ResourceManger,NodeManager
  - MapReduce: JobHistoryServer
- node02功能：
  - HDFS: DataNode
  - YARN: NodeManager
- node03功能：
  - HDFS: DataNode
  - YARN: NodeManager

### 1.2 安装CDH版的zookeeper

下载地址： http://archive.cloudera.com/cdh5/cdh/5/  
```
# 解压并创建数据文件夹
tar -zxvf zookeeper-3.4.9.tar.gz
mv  zookeeper-3.4.9.tar.gz /usr/local/

# node01修改配置
cd /usr/local/zookeeper-3.4.5-cdh5.14.0/conf 
cp zoo_sample.cfg zoo.cfg

# node01修改zk配置文件
vim  zoo.cfg
dataDir=/mydata/zookeeper/
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888

# 下发任务
cd /usr/local
scp -r zookeeper-3.4.9/ node02:$PWD
scp -r zookeeper-3.4.9/ node03:$PWD
```

启动：
```
# 三台机器都创建myid文件并写入
mkdir -p /mydata/zookeeper
cd /mydata/zookeeper/
touch myid 
echo 1 > myid               # node02 为 echo 2, node03 为 echo 3

# 三台服务器启动zookeeper，三台机器都执行以下命令启动zookeeper
cd  /usr/local/zookeeper-3.4.9
bin/zkServer.sh start
jps                         # 查看启动进程
bin/zkServer.sh status      # 查看集群状态，未安装三台集群时，不要使用该命令
```

### 1.3 安装CDH版Hadoop

生产环境中，由于版本兼容问题，我们不推荐使用Apache版本的Hadoop，推荐使用Cloudera公司的Hadoop，该公司的Hadoop各种相应配件都有与Hadoop对应的适配版本。Cloudera官方推荐使用CDH图形化界面安装，所以其安装包移除了源码中的依赖环境！！。但是图形化对硬件要求较高，安装包中又没有提供依赖环境，所以学习仍然需要使用虚拟机进行编译后再安装（即使如此也会经常失败，推荐使用一台配置很好的阿里云服务先进行编译，然后在自己的三台服务器上安装）。    

解压编译后的Hadoop文件到 /usr/local目录即可，然后检查环境：
```
cd /usr/local
./hadoop-2.7.5/bin/hadoop checknative 
```

四个如果都为true，则环境正确，按照下一节的配置修改，修改完毕后下发给node02，node03即可。

## 二 第二步：修改配置文件

进入hadoop目录后，进行如下修改。

### 2.1 第一台机器修改etc/hadoop/core-site.xml

```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node01:8020</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/mydata/hadoop/tempDatas</value>
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

### 2.2 修改第一台机器 etc/hadoop/hdfs-site.xml

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
		<value>file:///mydata/hadoop/namenodeDatas</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///mydata/hadoop/datanodeDatas</value>
	</property>
	
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///mydata/hadoop/dfs/nn/edits</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///mydata/hadoop/dfs/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///mydata/hadoop/dfs/nn/snn/edits</value>
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

### 2.3 修改第一台机器 etc/hadoop/hadoop-env.sh

```
vim hadoop-env.sh
export JAVA_HOME=/usr/local/jdk1.8.0_141
```

### 2.4 修改第一台机器 etc/hadoop/mapred-site.xml
如果没有该文件，可以修改 mapred-site.xml.template
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

### 2.5 第一台机器修改 etc/hadoop/yarn-site.xml

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

### 2.6 修改第一台机器 etc/hadoop/slaves

```
node01
node02
node03
```

## 三 第三步：创建文件目录

node01创建如下目录：
```
mkdir -p /mydata/hadoop/tempDatas
mkdir -p /mydata/hadoop/namenodeDatas
mkdir -p /mydata/hadoop/datanodeDatas 
mkdir -p /mydata/hadoop/dfs/nn/edits
mkdir -p /mydata/hadoop/dfs/snn/name
mkdir -p /mydata/hadoop/dfs/nn/snn/edits
```

## 四 第四步：分发安装包

node01执行：
```
cd /usr/local/
scp -r hadoop-2.7.5/ node02:$PWD
scp -r hadoop-2.7.5/ node03:$PWD
```

## 五 启动

### 5.1 配置hadoop环境变量

所有机器都执行：
```
vim  /etc/profile

export HADOOP_HOME=/usr/local/hadoop-2.7.5
export PATH=:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib/native"

source /etc/profile
```

### 5.2 启动准备

首次启动 Hadoop 集群，必须对node01进行格式化操作,本质上是一些清理和准备工作，因为此时的 HDFS 在物理上还是不存在的：
```
cd hadoop-2.7.5/
bin/hdfs namenode  -format			# 或 bin/hadoop namenode –format
```

### 5.3 启动方式一：单节点逐一启动

以下脚本位于$HADOOP_PREFIX/sbin/目录下。如果想要停止某个节点上某个角色，只需要把命令中的start 改为stop 即可。  
```
# 在主节点上使用以下命令启动 HDFS NameNode
hadoop-daemon.sh start namenode 

# 在每个从节点上使用以下命令启动 HDFS DataNode
hadoop-daemon.sh start datanode 

# 在主节点上使用以下命令启动 YARN ResourceManager
yarn-daemon.sh  start resourcemanager 

# 在每个从节点上使用以下命令启动 YARN nodemanager
yarn-daemon.sh start nodemanager 
```

### 5.4 启动方式二：脚本一键启动

如果配置了 etc/hadoop/slaves 和 ssh 免密登录，则可以在主节点使用程序脚本启动：
```
cd /usr/local/hadoop-2.7.5/
sbin/start-dfs.sh
sbin/start-yarn.sh
sbin/mr-jobhistory-daemon.sh start historyserver
```

停止集群:不建议随意停止集群
```
cd /usr/local/hadoop-2.6.0-cdh5.14.0/
sbin/stop-dfs.sh
sbin/stop-yarn.sh
sbin/mr-jobhistory-daemon.sh stop historyserver
```

停止后如果要重启需要在启动前格式化：`bin/hdfs namenode  -format`

## 六 浏览器查看启动页面

`jps`命令可以查看当前接待启动的情况。  

网页访问：
- hdfs集群访问地址：http://192.168.120.131:50070/dfshealth.html#tab-overview  
- yarn集群访问地址：http://192.168.120.131:8088/cluster  
- jobhistory访问地址：http://192.168.120.131:19888/jobhistory