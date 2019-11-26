## 一 递归遍历文件系统当中的所有文件

递归遍历hdfs文件系统:
```java
@Test
public void listFile() throws Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), new Configuration());
    FileStatus[] fileStatuses = fileSystem.listStatus(new Path("/"));
    for (FileStatus fileStatus : fileStatuses) {
        if(fileStatus.isDirectory()){
            Path path = fileStatus.getPath();
            listAllFiles(fileSystem,path);
        }else{
            System.out.println("文件路径为"+fileStatus.getPath().toString());

        }
    }
}
public void listAllFiles(FileSystem fileSystem,Path path) throws  Exception{
    FileStatus[] fileStatuses = fileSystem.listStatus(path);
    for (FileStatus fileStatus : fileStatuses) {
        if(fileStatus.isDirectory()){
            listAllFiles(fileSystem,fileStatus.getPath());
        }else{
            Path path1 = fileStatus.getPath();
            System.out.println("文件路径为"+path1);
        }
    }
}
```

官方提供的直接遍历API：
```java
@Test
public void listMyFiles()throws Exception{
    //获取fileSystem类
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), new Configuration());
    //获取RemoteIterator 得到所有的文件或者文件夹，第一个参数指定遍历的路径，第二个参数表示是否要递归遍历
    RemoteIterator<LocatedFileStatus> locatedFileStatusRemoteIterator = fileSystem.listFiles(new Path("/"), true);
    while (locatedFileStatusRemoteIterator.hasNext()){
        LocatedFileStatus next = locatedFileStatusRemoteIterator.next();
        System.out.println(next.getPath().toString());
    }
    fileSystem.close();
}
```

## 二 下载HDFS文件到本地

```java
@Test
public void getFileToLocal()throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.1310:8020"), new Configuration());
    FSDataInputStream open = fileSystem.open(new Path("/test/input/install.log"));
    FileOutputStream fileOutputStream = new FileOutputStream(new File("c:\\install.log"));
    IOUtils.copy(open,fileOutputStream );
    IOUtils.closeQuietly(open);
    IOUtils.closeQuietly(fileOutputStream);
    fileSystem.close();
}
```

## 三 HDFS上创建文件夹

```java
@Test
public void mkdirs() throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), new Configuration());
    boolean mkdirs = fileSystem.mkdirs(new Path("/hello/mydir/test"));
    fileSystem.close();
}
```

## 四 HDFS文件上传

```java
@Test
public void putData() throws  Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), new Configuration());
    fileSystem.copyFromLocalFile(new Path("file:///c:\\install.log"),new Path("/hello/mydir/test"));
    fileSystem.close();
}
```

## 五 HDFS权限问题以及伪造用户

HDFS开启权限的操作：
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
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), new Configuration(),"root");
    fileSystem.copyToLocalFile(new Path("/config/core-site.xml"),new Path("file:///c:/core-site.xml"));
    fileSystem.close();
}
```