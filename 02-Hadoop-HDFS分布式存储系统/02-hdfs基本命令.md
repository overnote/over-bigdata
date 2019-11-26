## 一 基础命令

- ls：查看目录下文件列表
  - hdfs dfs -ls /dirs/dir1
  - hdfs dfs -ls -R             # 旧版写作 -lsr
- mkdir：创建文件夹，支持递归参数 -p
  - hdfs dfs -mkdir -p /dirs/dir1
- moveFromLocal：从本地磁盘移动到hdfs磁盘
  - hdfs dfs -moveFromeLocal /root/test.log /
- moveToLocal：该命令未实现
- mv：移动hdfs文件到另外一个hdfs路径中
- put：复制hdfs文件到另外一个hdfs路径中，如果报异常是正常的（只是日志异常）
- appendToFile：追加若干个文件内容到hdfs指定文件中
  - hdfs dfs -appendToFile localfile /user/hadoop/hadoopfile
- cat：查看文件内容
- cp：拷贝第一个文件为第二个文件名
  - hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2
  - hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
- rm：删除，支持参数 -f（强制删除） -r（递归删除） 删除只会将文件移动到垃圾箱（7天后自动删除，-skipTrash可以直接彻底删除）
  - hdfs dfs -rm hdfs://nn.example.com/file /user/hadoop/emptydir
- chmod：修改权限
  - hdfs dfs -chmod -R 777 /user
- chown：
  - hdfs dfs -chown -R mine:ruyue /test # 修改/test的用户组和用户
- expunge：清空垃圾箱

## 二 HDFS文件限额命令

hdfs文件的限额配置允许用户以文件大小或者文件个数来限制某个目录下上传的文件数量/内容总量，以便达到类似网盘限制上传效果功能。  

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

安全模式是HDFS所处的一种特殊状态，在这种状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。  

在NameNode主节点启动时，HDFS首先进入安全模式，DataNode在启动的时候会向namenode汇报可用的block等状态，当整个系统达到安全标准时，HDFS自动离开安全模式。如果HDFS出于安全模式下，则文件block不能进行任何的副本复制操作，因此达到最小的副本数量要求是基于datanode启动时的状态来判定的，启动时不会再做任何复制（从而达到最小副本数量要求）。  

hdfs集群刚启动的时候，默认30S钟的时间是处于安全期的，只有过了30S之后，集群脱离了安全期，然后才可以对集群进行操作:
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