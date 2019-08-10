## 一 基础命令

- ls：hdfs dfs -ls /user/hadoop/file1
- lsr：
- Node：
- mkdir：
- moveFromLocal：
- moveToLocal：
- mv：
- put：
- appendToFile：追加一个或者多个文件到hdfs指定文件中.也可以从命令行读取输入。
  - hdfs dfs -appendToFile localfile /user/hadoop/hadoopfile
  - hdfs dfs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
  - hdfs dfs -appendToFile localfile hdfs://nn.example.com/hadoop/hadoopfile
  - hdfs dfs -appendToFile - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin
- cat：
  - hdfs dfs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
  - hdfs dfs -cat file:///file3 /user/hadoop/file4
- cp：
  - hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2
  - hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
- rm：
  - hdfs dfs -rm hdfs://nn.example.com/file /user/hadoop/emptydir
- rmr：
- chmod：
- chown：
- expunge：清空回收站

## 二 HDFS文件限额配置

hdfs文件的限额配置允许我们以文件大小或者文件个数来限制我们在某个目录下上传的文件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大的文件的量。  

数量限额：
```
hdfs dfs -mkdir -p /user/root/lisi          # 创建hdfs文件夹
hdfs dfsadmin -setQuota 2 lisi              # 给该文件夹下面设置最多上传两个文件，上传文件，发现只能上传一个文件
hdfs dfsadmin -clrQuota /user/root/lisi     # 清除文件数量限制
```

空间限额:
```
hdfs dfsadmin -setSpaceQuota 4k /user/root/lisi   # 限制空间大小4KB
hdfs dfs -put  /export/softwares/zookeeper-3.4.5-cdh5.14.0.tar.gz /user/root/lisi

# 上传超过4Kb的文件大小上去提示文件超过限额
hdfs dfsadmin -clrSpaceQuota /user/root/lisi   # 清除空间限额
# 重新上传
hdfs dfs -put  /export/softwares/zookeeper-3.4.5-cdh5.14.0.tar.gz /user/root/lisi
```

查看hdfs文件限额数量：
```
hdfs dfs -count -q -h /user/root/lisi
```

## 三 hdfs的安全模式

安全模式是HDFS所处的一种特殊状态，在这种状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。在NameNode主节点启动时，HDFS首先进入安全模式，DataNode在启动的时候会向namenode汇报可用的block等状态，当整个系统达到安全标准时，HDFS自动离开安全模式。如果HDFS出于安全模式下，则文件block不能进行任何的副本复制操作，因此达到最小的副本数量要求是基于datanode启动时的状态来判定的，启动时不会再做任何复制（从而达到最小副本数量要求），hdfs集群刚启动的时候，默认30S钟的时间是出于安全期的，只有过了30S之后，集群脱离了安全期，然后才可以对集群进行操作:
```
hdfs  dfsadmin  -safemode
```

## 四 hadoop基准测试

实际生产环境当中，hadoop的环境搭建完成之后，第一件事情就是进行压力测试，测试我们的集群的读取和写入速度，测试我们的网络带宽是否足够等一些基准测试。  

测试写入速度：
```
# 向HDFS文件系统中写入数据,10个文件,每个文件10MB,文件存放到
/benchmarks/TestDFSIO中
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO  -write -nrFiles 10 -fileSize 10MB

#完成之后查看写入速度结果
hdfs dfs -text /benchmarks/TestDFSIO/io_write/part-00000
```

测试读取速度，测试hdfs的读取文件性能：
```
# 在HDFS文件系统中读入10个文件,每个文件10M
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO -read -nrFiles 10 -fileSize 10MB

# 查看读取结果
hdfs dfs -text /benchmarks/TestDFSIO/io_read/part-00000
```

清除测试数据
```
hadoop jar /export/servers/hadoop-2.6.0-cdh5.14.0/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.14.0.jar TestDFSIO -clean
```