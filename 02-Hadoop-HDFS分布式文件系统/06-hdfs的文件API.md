## 一 下载文件到本地

```java
@Test
public void getFileToLocal()throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.52.100:8020"), new Configuration());
    FSDataInputStream open = fileSystem.open(new Path("/test/input/install.log"));
    FileOutputStream fileOutputStream = new FileOutputStream(new File("c:\\install.log"));
    IOUtils.copy(open,fileOutputStream );
    IOUtils.closeQuietly(open);
    IOUtils.closeQuietly(fileOutputStream);
    fileSystem.close();
}
```

## 二 hdfs上创建文件夹

```java
@Test
public void mkdirs() throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.52.100:8020"), new Configuration());
    boolean mkdirs = fileSystem.mkdirs(new Path("/hello/mydir/test"));
    fileSystem.close();
}
```

## 三 hdfs文件上传

```java
@Test
public void putData() throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.52.100:8020"), new Configuration());
    fileSystem.copyFromLocalFile(new Path("file:///c:\\install.log"),new Path("/hello/mydir/test"));
    fileSystem.close();
}
```

## 四 HDFS权限问题以及伪造用户

操作：
```
# 首先停止hdfs集群，在node01机器上执行以下命令
cd /usr/local/hadoop-2.6.0-cdh5.14.0
sbin/stop-dfs.sh

# 修改node01机器上的hdfs-site.xml当中的配置文件
cd /usr/local/hadoop-2.6.0-cdh5.14.0/etc/hadoop
vim hdfs-site.xml

<property>
    <name>dfs.permissions</name>
    <value>true</value>
</property>

# 修改完成之后配置文件发送到其他机器上面去
scp hdfs-site.xml node02:$PWD
scp hdfs-site.xml node03:$PWD

# 重启hdfs集群
cd /usr/local/hadoop-2.6.0-cdh5.14.0
sbin/start-dfs.sh

# 随意上传一些文件到我们hadoop集群当中准备测试使用
cd /usr/local/hadoop-2.6.0-cdh5.14.0/etc/hadoop
hdfs dfs -mkdir /config
hdfs dfs -put *.xml /config
hdfs dfs -chmod 600 /config/core-site.xml 
```

使用代码准备下载文件
```java
// 创建FileSystem对象，加上第三个参数root，表示我们使用哪个用户去获取文件系统上面的文件
@Test
public void getConfig()throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.52.100:8020"), new Configuration(),"root");
    fileSystem.copyToLocalFile(new Path("/config/core-site.xml"),new Path("file:///c:/core-site.xml"));
    fileSystem.close();
}
```

## 五 HDFS的小文件合并

由于hadoop擅长存储大文件，因为大文件的元数据信息比较少，如果hadoop集群当中有大量的小文件，那么每个小文件都需要维护一份元数据信息，会大大的增加集群管理元数据的内存压力，所以在实际工作当中，如果有必要一定要将小文件合并成大文件进行一起处理  

在我们的hdfs 的shell命令模式下，可以通过命令行将很多的hdfs文件合并成一个大文件下载到本地，命令如下
```
cd /usr/local
hdfs dfs -getmerge /config/*.xml  ./hello.xml
```

既然可以在下载的时候将这些小文件合并成一个大文件一起下载，那么肯定就可以在上传的时候将小文件合并到一个大文件里面去,代码如下：

```java
@Test
public void mergeFile() throws  Exception{
    //获取分布式文件系统
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.52.100:8020"), new Configuration(),"root");
    FSDataOutputStream outputStream = fileSystem.create(new Path("/bigfile.xml"));
    //获取本地文件系统
    LocalFileSystem local = FileSystem.getLocal(new Configuration());
    //通过本地文件系统获取文件列表，为一个集合
    FileStatus[] fileStatuses = local.listStatus(new Path("file:///F:\\传智播客大数据离线阶段课程资料\\3、大数据离线第三天\\上传小文件合并"));
    for (FileStatus fileStatus : fileStatuses) {
        FSDataInputStream inputStream = local.open(fileStatus.getPath());
       IOUtils.copy(inputStream,outputStream);
        IOUtils.closeQuietly(inputStream);
    }
    IOUtils.closeQuietly(outputStream);
    local.close();
    fileSystem.close();
}
```