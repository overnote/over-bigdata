## 一 MapReduce编程规范

#### 1.1 第一阶段：Map阶段

- 第一步：设置inputFormat类，将我们的数据切分成key，value对，输入到第二步
- 第二步：自定义map逻辑，处理我们第一步的输入数据，然后转换成新的key，value对进行输出

#### 1.2 第二阶段：shuffle

- 第三步：对输出的key，value对进行分区
- 第四步：对不同分区的数据按照相同的key进行排序
- 第五步：对分组后的数据进行规约(combine操作)，降低数据的网络拷贝（可选步骤）
- 第六步：对排序后的额数据进行分组，分组的过程中，将相同key的value放到一个集合当中

#### 1.3 第三阶段：reduce

- 第七步：对多个map的任务进行合并，排序，写reduce函数自己的逻辑，对输入的key，value对进行处理，转换成新的key，value对进行输出
- 第八步：设置outputformat将输出的key，value对数据进行保存到文件中

## 二 入门示例：WordCount

#### 2.0 需求与数据

需求：在一堆给定的文本文件中统计输出每一个单词出现的总次数。  

数据格式：
```
vim wordcount.txt

hello,world,hadoop
hive,sqoop,flume,hello
kitty,tom,jerry,world
hadoop

hdfs dfs -mkdir /wordcount/
hdfs dfs -put wordcount.txt /wordcount/
```

#### 2.1 代码示例

定义一个mapper类：
```java
public class WordCountMapper extends Mapper<LongWritable,Text,Text,LongWritable> {
    @Override
    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        String[] split = line.split(",");
        for (String word : split) {
            context.write(new Text(word),new LongWritable(1));
        }

    }
}
```

定义一个reducer类：
```java
public class WordCountReducer extends Reducer<Text,LongWritable,Text,LongWritable> {
    /**
     * 自定义我们的reduce逻辑
     * 所有的key都是我们的单词，所有的values都是我们单词出现的次数
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long count = 0;
        for (LongWritable value : values) {
            count += value.get();
        }
        context.write(key,new LongWritable(count));
    }
}
```

定义一个主类，用来描述job并提交job：
```java
public class JobMain extends Configured implements Tool {
    @Override
    public int run(String[] args) throws Exception {
        Job job = Job.getInstance(super.getConf(), JobMain.class.getSimpleName());
        //打包到集群上面运行时候，必须要添加以下配置，指定程序的main函数
        job.setJarByClass(JobMain.class);
        //第一步：读取输入文件解析成key，value对
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("hdfs://192.168.52.100:8020/wordcount"));

        //第二步：设置我们的mapper类
        job.setMapperClass(WordCountMapper.class);
        //设置我们map阶段完成之后的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);
        //第三步，第四步，第五步，第六步，省略
        //第七步：设置我们的reduce类
        job.setReducerClass(WordCountReducer.class);
        //设置我们reduce阶段完成之后的输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        //第八步：设置输出类以及输出路径
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path("hdfs://192.168.120.111:8020/wordcount_out"));
        boolean b = job.waitForCompletion(true);
        return b?0:1;
    }

    /**
     * 程序main函数的入口类
     * @param args
     * @throws Exception
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        Tool tool  =  new JobMain();
        int run = ToolRunner.run(configuration, tool, args);
        System.exit(run);
    }
}
```

贴士1：如果出现错误 `org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.AccessControlException): Permission denied: user=admin, access=WRITE, inode="/":root:supergroup:drwxr-xr-x `，直接将hdfs-site.xml当中的权限按照如下所示关闭，再重启hdfs集群即可：
```xml
<property>
    <name>dfs.permissions</name>
    <value>false</value>
</property>
```

贴士2：本地运行完成之后，就可以打成jar包放到服务器上面去运行了，实际工作当中，都是将代码打成jar包，开发main方法作为程序的入口，然后放到集群上面去运行。  

## 三 MapReduce程序运行模式

#### 3.1 本地运行模式

- （1）mapreduce程序是被提交给LocalJobRunner在本地以单进程的形式运行
- （2）而处理的数据及输出结果可以在本地文件系统，也可以在hdfs上
- （3）怎样实现本地运行？写一个程序，不要带集群的配置文件
本质是程序的conf中是否有mapreduce.framework.name=local以及yarn.resourcemanager.hostname=local参数
- （4）本地模式非常便于进行业务逻辑的debug，只要在eclipse中打断点即可

本地模式运行代码设置：
```
configuration.set("mapreduce.framework.name","local");
configuration.set(" yarn.resourcemanager.hostname","local");
TextInputFormat.addInputPath(job,new Path("file:///E:\\testhadoop\\wordcount\\input"));
TextOutputFormat.setOutputPath(job,new Path("file:///E:\\testhadoop\\wordcount\\output"));
```

#### 3.2 集群运行模式

- （1）将mapreduce程序提交给yarn集群，分发到很多的节点上并发执行
- （2）处理的数据和输出结果应该位于hdfs文件系统
- （3）提交集群的实现步骤：将程序打成JAR包，然后在集群的任意一个节点上用hadoop命令启动

```
yarn jar hadoop_hdfs_operate-1.0-SNAPSHOT.jar com.testhadoop.hdfs.demo1.JobMain
```