## 一 maven工程的贴士

由于cdh版本的所有的软件涉及版权的问题，所以并没有将所有的jar包托管到maven仓库当中去，而是托管在了CDH自己的服务器上面，需要自己手动的添加repository去CDH仓库进行下载。  

```xml
<repositories>
    <repository>
        <id>cloudera</id>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
</repositories>
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.6.0-mr1-cdh5.14.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.6.0-cdh5.14.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.6.0-cdh5.14.0</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>2.6.0-cdh5.14.0</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>RELEASE</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <!--    <verbal>true</verbal>-->
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.4.3</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <minimizeJar>true</minimizeJar>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

maven仓库也可能出现无法下载的问题，可以解压已下载的过maven仓库到本地，然后在IDEA上选择-settings-Build-Maven-Repositories-点击右侧的Update   
cdh版本jar包下载地址：
- https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh5_maven_repo.html
- https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh5_maven_repo_514x.html

## 二 数据的访问

数据的访问有两种方式：
- 使用url方式
- 使用文件系统方式访问数据（推荐）

#### 2.1 url方式访问数据

使用标准的流接口操作hdfs：
```java
@Test
public void demo1() throws Exception{

    //第一步：注册 hdfs 的url，让java代码能够识别hdfs的url形式
    URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());

    //定义要访问的文件地址
    String url = "hdfs://192.168.86.131:8020/test/log.log";
    
    //打开文件输入流
    InputStream inputStream = null;
    FileOutputStream outputStream =null;
    try {
        inputStream = new URL(url).openStream();
        outputStream = new FileOutputStream(new File("c:\\hello.txt"));      // 数据读取到本地的一个文件中
        IOUtils.copy(inputStream, outputStream);
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        IOUtils.closeQuietly(inputStream);
        IOUtils.closeQuietly(outputStream);
    }
}
```

如果报winutils异常，这是因为windows作为了客户端操作了linux上的hadoop，解决方案：
- 把hadoop-2.6.0-cdh5.14.0这个安装包拷贝到一个没有中文没有空格的路径下面去
- 在windows上面配置hadoop的环境变量，增加变量HADOOP_HOME，值是下载的zip包解压的目录，然后在系统变量path里增加HADOOP_HOME\bin 即可。
- 把hadoop-2.6.0-cdh5.14.0中的lib中的hadoop.dll文件放到系统盘里面去  C:\Windows\System32
- 关闭windows重启

#### 2.2 使用文件系统方式访问数据（推荐）

使用Hadoop官方提供的API，主要涉及以下类： 
- Configuration：该类的对象封转了客户端或者服务器的配置
- FileSystem：该类的对象是一个文件系统的抽象类，可以用该对象的一些方法来对文件进行操作，通过 FileSystem 的静态方法 get 获得该对象  

```java
FileSystem fs = FileSystem.get(conf)
```

get 方法从 conf 中的一个参数 fs.defaultFS 的配置值判断具体是什么类型的文件系统。如果我们的代码中没有指定 fs.defaultFS，并且工程 classpath下也没有给定相应的配置，conf中的默认值就来自于hadoop的jar包中的core-default.xml ：`file:///`，此时获取的不是 DistributedFileSystem 实例，而是本地文件系统的客户端对象。  

方式一：
```java
@Test
public void getFileSystem1() throws URISyntaxException, IOException {
    Configuration configuration = new Configuration();      // 刚new出来时还是本地文件系统
    // 使用两个参数获取分布式文件系统
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://192.168.186.131:8020"), configuration);
    System.out.println(fileSystem.toString());
}
```

方式二：
```java
@Test
public void getFileSystem2() throws URISyntaxException, IOException {
    Configuration configuration = new Configuration();
    // 覆盖原始配置，设置为分布式文件系统
    configuration.set("fs.defaultFS","hdfs://192.168.186.131:8020");       
    FileSystem fileSystem = FileSystem.get(new URI("/"), configuration);
    System.out.println(fileSystem.toString());
}
```

方式三：
```java
@Test
public void getFileSystem3() throws URISyntaxException, IOException {
    Configuration configuration = new Configuration();
    FileSystem fileSystem = FileSystem.newInstance(new URI("hdfs://192.168.186.131:8020"), configuration);
    System.out.println(fileSystem.toString());
}
```

方式四：
```java
@Test
public void getFileSystem4() throws  Exception{
    Configuration configuration = new Configuration();
    configuration.set("fs.defaultFS","hdfs://192.168.186.131:8020");
    FileSystem fileSystem = FileSystem.newInstance(configuration);
    System.out.println(fileSystem.toString());
}
```