## 一 CDH版zookeeper环境

下载地址： http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.0.tar.gz

node01修改配置：
```
# 创建zk数据存放目录
mkdir -p /data/zookeeper/datas
cd /usr/local/zookeeper-3.4.5-cdh5.14.0/conf 
cp zoo_sample.cfg zoo.cfg

# 修改zk配置文件
vim  zoo.cfg
dataDir=/data/zookeeper/datas
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888

# 下发任务
cd /usr/local
scp -r zookeeper-3.4.5-cdh5.14.0/ node02:$PWD
scp -r zookeeper-3.4.5-cdh5.14.0/ node03:$PWD

```

启动：
```
# 三台机器都创建myid文件并写入
mkdir -p /data/zookeeper/datas
cd /data/zookeeper/datas
touch myid 
echo 1 > myid               # node02 为 echo 2, node03 为 echo 3

# 三台服务器启动zookeeper，三台机器都执行以下命令启动zookeeper
cd  /usr/local/zookeeper-3.4.5-cdh5.14.0
bin/zkServer.sh start
```