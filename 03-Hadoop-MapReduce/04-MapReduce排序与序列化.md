## 一 理解MapReduce排序与序列化

- 序列化（Serialization）是指把结构化对象转化为字节流。 
- 反序列化（Deserialization）是序列化的逆过程。把字节流转为结构化对象。当要在进程间传递对象或持久化对象的时候，就需要序列化对象成字节流，反之当要将接收到或从磁盘读取的字节流转换为对象，就要进行反序列化。   

Java 的序列化（Serializable）是一个重量级序列化框架，一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系…），不便于在网络中高效传输；所以hadoop 自己开发了一套序列化机制（Writable），精简，高效。不用像 java 对象类一样传输多层的父子关系，需要哪个属性就传输哪个属性值，大大的减少网络传输的开销。  

Writable是Hadoop的序列化格式，hadoop定义了这样一个Writable接口。 一个类要支持可序列化只需实现这个接口即可。  

另外Writable有一个子接口是WritableComparable，writableComparable是既可实现序列化，也可以对key进行比较，我们这里可以通过自定义key实现WritableComparable来实现我们的排序功能。  

数据格式如下：
```
a	1
a	9
b	3
a	7
b	8
b	10
a	5
```

## 二 排序示例

需求：要求第一列按照字典顺序进行排列，第一列相同的时候，第二列按照升序  

思路：将map端输出的`<key,value>`中的key和value组合成一个新的key（称为newKey），value值不变。这里就变成<(key,value),value>，在针对newKey排序的时候，如果key相同，就再对value进行排序。  

第一步：自定义我们的数据类型以及比较器:
```java
public class PairWritable implements WritableComparable<PairWritable> {
    // 组合key,第一部分是我们第一列，第二部分是我们第二列
    private String first;
    private int second;
    public PairWritable() {
    }
    public PairWritable(String first, int second) {
        this.set(first, second);
    }
    /**
     * 方便设置字段
     */
    public void set(String first, int second) {
        this.first = first;
        this.second = second;
    }
    /**
     * 反序列化
     */
    @Override
    public void readFields(DataInput input) throws IOException {
        this.first = input.readUTF();
        this.second = input.readInt();
    }
    /**
     * 序列化
     */
    @Override
    public void write(DataOutput output) throws IOException {
        output.writeUTF(first);
        output.writeInt(second);
    }
    /*
     * 重写比较器
     */
    public int compareTo(PairWritable o) {
        //每次比较都是调用该方法的对象与传递的参数进行比较，说白了就是第一行与第二行比较完了之后的结果与第三行比较，
        //得出来的结果再去与第四行比较，依次类推
        System.out.println(o.toString());
        System.out.println(this.toString());
        int comp = this.first.compareTo(o.first);
        if (comp != 0) {
            return comp;
        } else { // 若第一个字段相等，则比较第二个字段
            return Integer.valueOf(this.second).compareTo(
                    Integer.valueOf(o.getSecond()));
        }
    }

    public int getSecond() {
        return second;
    }

    public void setSecond(int second) {
        this.second = second;
    }
    public String getFirst() {
        return first;
    }
    public void setFirst(String first) {
        this.first = first;
    }
    @Override
    public String toString() {
        return "PairWritable{" +
                "first='" + first + '\'' +
                ", second=" + second +
                '}';
    }
}
```

第二步：自定义我们的map逻辑:
```java
public class SortMapper extends Mapper<LongWritable,Text,PairWritable,IntWritable> {

    private PairWritable mapOutKey = new PairWritable();
    private IntWritable mapOutValue = new IntWritable();

    @Override
    public  void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String lineValue = value.toString();
        String[] strs = lineValue.split("\t");
        //设置组合key和value ==> <(key,value),value>
        mapOutKey.set(strs[0], Integer.valueOf(strs[1]));
        mapOutValue.set(Integer.valueOf(strs[1]));
        context.write(mapOutKey, mapOutValue);
    }
}
```

第三步：自定义我们的reduce逻辑
```java
public class SortReducer extends Reducer<PairWritable,IntWritable,Text,IntWritable> {

    private Text outPutKey = new Text();
    @Override
    public void reduce(PairWritable key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
//迭代输出
        for(IntWritable value : values) {
            outPutKey.set(key.getFirst());
            context.write(outPutKey, value);
        }
    }
}
```

第四步：main方法
```java
public class SecondarySort  extends Configured implements Tool {
    @Override
    public int run(String[] args) throws Exception {
        Configuration conf = super.getConf();
        conf.set("mapreduce.framework.name","local");
        Job job = Job.getInstance(conf, SecondarySort.class.getSimpleName());
        job.setJarByClass(SecondarySort.class);

        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("file:///E:\\testhadoop\\demo2\\input"));
        TextOutputFormat.setOutputPath(job,new Path("file:///E:\\testhadoop\\demo2\\output"));
        job.setMapperClass(SortMapper.class);
        job.setMapOutputKeyClass(PairWritable.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setReducerClass(SortReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        boolean b = job.waitForCompletion(true);
        return b?0:1;
    }


    public static void main(String[] args) throws Exception {
        Configuration entries = new Configuration();
        ToolRunner.run(entries,new SecondarySort(),args);
    }
}
```