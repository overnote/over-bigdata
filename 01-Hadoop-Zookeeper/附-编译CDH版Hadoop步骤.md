## 附 编译hadoop步骤

由于源码包缺少编译文件，需要准备一台很好的Linux机器，编译步骤如下所示。当然，网上也有大量已经编译完成的版本可供下载。  
```
# 前提：机器必须关闭selinux，防火墙，安装jdk7（只有jdk7才能编译）

# 步骤一：安装依赖
yum update -y
yum install -y autoconf automake libtool cmake lzo-devel zlib-devel gcc gcc-c++ ncurses-devel openssl-devel bzip2-devel

# 步骤二：配置maven环境
tar -zxvf apache-maven-3.0.5-bin.tar.gz -C /usr/local
vim /etc/profile
export MAVEN_HOME=/usr/local/apache-maven-3.0.5
export MAVEN_OPTS="-Xms4096m -Xmx4096m"
export PATH=:$MAVEN_HOME/bin:$PATH

source /etc/profile

# 步骤三：安装findbugs  https://sourceforge.net/projects/findbugs/files/findbugs/1.3.9/findbugs-1.3.9.tar.gz/download
tar -zxvf findbugs-1.3.9.tar.gz -C /usr/local
vim /etc/profile
export FINDBUGS_HOME=/usr/local/findbugs-1.3.9
export PATH=:$FINDBUGS_HOME/bin:$PATH

source  /etc/profile

# 步骤四安装protobuf  https://github.com/protocolbuffers/protobuf/releases/tag/v2.6.1 
tar -zxvf protobuf-2.5.0.tar.gz -C /usr/local
cd  /usr/local/protobuf-2.5.0
./configure
make && make install

# 步骤五 安装snappy https://src.fedoraproject.org/repo/pkgs/snappy/
tar -zxf snappy-1.1.1.tar.gz -C /usr/local
cd /usr/local/snappy-1.1.1/
./configure
make && make install
```

编译CDH版Hadoop步骤：
```
# 下载地址（5.3以上版本才会支持java8） http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.0-src.tar.gz
tar -zxvf hadoop-2.6.0-cdh5.14.0-src.tar.gz -C /usr/local
cd  /usr/local/hadoop-2.6.0-cdh5.14.0

# 编译不支持snappy压缩：
mvn package -Pdist,native -DskipTests –Dtar   

# 编译支持snappy压缩：
mvn package -DskipTests -Pdist,native -Dtar -Drequire.snappy -e -X

# 查看hadoop支持的压缩方式以及本地库
./bin/hadoop checknative  
```

注意，如果出现错误`An Ant BuildException has occured: exec returned: 2`，是因为tomcat的压缩包没有下载完成，需要自己下载一个对应版本的apache-tomcat-6.0.53.tar.gz的压缩包放到指定路径下面去即可,这两个路径下面需要放上这个tomcat的压缩包:
```
/usr/local/hadoop-2.6.0-cdh5.14.0/hadoop-hdfs-project/hadoop-hdfs-httpfs/downloads
/usr/local/hadoop-2.6.0-cdh5.14.0/hadoop-common-project/hadoop-kms/downloads
```  

编译后的文件位于 dist 目录下！！