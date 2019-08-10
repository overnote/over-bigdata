## 一 三台机器的Linux编译准备

#### 1.0 说明

需要编译原因：官方推荐使用CDH图形化界面安装，移除了源码中的依赖环境，但是该图形化对硬件要求较高，学习仍然需要使用虚拟机编译安装。  

但是即使是编译，也需要大量的硬件，软件，网络支持（很容易失败），这里直接推荐使用已经编译完成的hadoop，或者使用一台配置较好的阿里云服务器进行编译。

#### 1.2 环境准备

这里使用一台全新的阿里云CentOS6.9系统进行编译。  

安装系统依赖：
```
yum install -y autoconf automake libtool cmake
yum install -y lzo-devel zlib-devel gcc gcc-c++
yum install -y ncurses-devel openssl-devel bzip2-devel

service iptables stop		# 关闭防火墙
chkconfig iptables off		# 关闭防火墙开机自启动

vim /etc/selinux/config		# 注释 SELINUX=enforcing，添加新的：SELINUX=disabled
```

安装jdk：
```
# 卸载原有JAVA
rpm -e java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64    tzdata-java-2016j-1.el6.noarch java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64 --nodeps

# 安装jdk7
mkdir -p /usr/local/java
tar -zxvf jdk-7u75-linux-x64.tar.gz -C /usr/local/

# 配置环境
vim /etc/profile
export JAVA_HOME=/usr/local/jdk1.7.0_75
export PATH=:$JAVA_HOME/bin:$PATH

# 环境生效
source /etc/profile
```

配置maven环境
```
tar -zxvf apache-maven-3.0.5 -C /usr/local
vim /etc/profile
export MAVEN_HOME=/usr/local/apache-maven-3.6.1
export MAVEN_OPTS="-Xms4096m -Xmx4096m"
export PATH=:$MAVEN_HOME/bin:$PATH

source /etc/profile
```

安装findbugs  https://sourceforge.net/projects/findbugs/files/findbugs/1.3.9/findbugs-1.3.9.tar.gz/download
```
tar -zxvf findbugs-1.3.9.tar.gz -C /usr/local
vim /etc/profile
export FINDBUGS_HOME=/usr/local/findbugs-1.3.9
export PATH=:$FINDBUGS_HOME/bin:$PATH
source  /etc/profile
```

安装protobuf  https://github.com/protocolbuffers/protobuf/releases/tag/v2.6.1 
```
tar -zxvf protobuf-2.5.0.tar.gz -C /usr/local
cd  /usr/local/protobuf-2.5.0
./configure
make && make install
```

安装snappy https://src.fedoraproject.org/repo/pkgs/snappy/
```
tar -zxf snappy-1.1.1.tar.gz -C /usr/local
cd /usr/local/snappy-1.1.1/
./configure
make && make install
```

## 二 编译

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