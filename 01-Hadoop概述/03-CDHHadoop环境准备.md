## 一 三台机器的Linux环境准备

这里使用CentOS6.9系统：
```
# 安装依赖
yum install -y autoconf automake libtool cmake
yum install -y lzo-devel zlib-devel gcc gcc-c++
yum install -y ncurses-devel openssl-devel bzip2-devel

# 关闭防火墙与selinux--见03节-Linux基础环境配置

# 安装jdk--见03节-Linux基础环境配置

# 安装maven

# 配置jdk，maven环境
tar -zxvf apache-maven-3.6.1 -C /usr/local
vim /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_11
export PATH=:$JAVA_HOME/bin:$PATH
export MAVEN_HOME=/usr/local/apache-maven-3.6.1
export MAVEN_OPTS="-Xms4096m -Xmx4096m"
export PATH=:$MAVEN_HOME/bin:$PATH

source /etc/profile


# 安装findbugs  https://sourceforge.net/projects/findbugs/files/findbugs/1.3.9/findbugs-1.3.9.tar.gz/download
tar -zxvf findbugs-1.3.9.tar.gz -C /usr/local
vim /etc/profile
export FINDBUGS_HOME=/usr/local/findbugs-1.3.9
export PATH=:$FINDBUGS_HOME/bin:$PATH
source  /etc/profile

# 安装protobuf  https://github.com/protocolbuffers/protobuf/releases/tag/v2.6.1 
tar -zxvf protobuf-2.6.1.tar.gz -C /usr/local
cd  /usr/local/protobuf-2.6.1
./configure
make && make install

# 安装snappy https://src.fedoraproject.org/repo/pkgs/snappy/
tar -zxf snappy-1.1.1.tar.gz -C /usr/local
cd /usr/local/snappy-1.1.1/
./configure
make && make install
```

## 二 CDH版zookeeper环境

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

# 创建myid文件并写入内容
mkdir -p /data/zookeeper/datas
cd /data/zookeeper/datas
touch myid 
echo 1 > myid

# 下发任务
cd /usr/local
scp -r zookeeper-3.4.5-cdh5.14.0/ node02:$PWD
scp -r zookeeper-3.4.5-cdh5.14.0/ node03:$PWD

```

node2与node03修改配置:
```
mkdir -p /data/zookeeper/datas
cd /data/zookeeper/datas
touch myid 
echo 2 > myid    # node03 为 echo 3
```

启动：三台服务器启动zookeeper，三台机器都执行以下命令启动zookeeper
```
cd  /usr/local/zookeeper-3.4.5-cdh5.14.0
bin/zkServer.sh start
```