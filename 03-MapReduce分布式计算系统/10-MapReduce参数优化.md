## 一 资源相关参数

以下调整参数都在mapred-site.xml这个配置文件当中有（参数是在用户自己的mr应用程序中配置就可以生效）：
- (1) mapreduce.map.memory.mb: 一个Map Task可使用的资源上限（单位:MB），默认为1024。如果Map Task实际使用的资源量超过该值，则会被强制杀死。
- (2) mapreduce.reduce.memory.mb: 一个Reduce Task可使用的资源上限（单位:MB），默认为1024。如果Reduce Task实际使用的资源量超过该值，则会被强制杀死。
- (3) mapred.child.java.opts  配置每个map或者reduce使用的内存的大小，默认是200M
- (4) mapreduce.map.cpu.vcores: 每个Map task可使用的最多cpu core数目, 默认值: 1
- (5) mapreduce.reduce.cpu.vcores: 每个Reduce task可使用的最多cpu core数目, 默认值: 1，shuffle性能优化的关键参数，应在yarn启动之前就配置好
- (6)mapreduce.task.io.sort.mb   100 ，shuffle的环形缓冲区大小，默认100m
- (7)mapreduce.map.sort.spill.percent   0.8 ，环形缓冲区溢出的阈值，默认80%，应该在yarn启动之前就配置在服务器的配置文件中才能生效

以下配置都在yarn-site.xml配置文件当中配置：
- (8) yarn.scheduler.minimum-allocation-mb	  1024   给应用程序container分配的最小内存
- (9) yarn.scheduler.maximum-allocation-mb	  8192	给应用程序container分配的最大内存
- (10) yarn.scheduler.minimum-allocation-vcores	1	
- (11)yarn.scheduler.maximum-allocation-vcores	32
- (12)yarn.nodemanager.resource.memory-mb   8192  

## 二 容错相关参数

- (1) mapreduce.map.maxattempts: 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
- (2) mapreduce.reduce.maxattempts: 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
- (3) mapreduce.job.maxtaskfailures.per.tracker: 当失败的Map Task失败比例超过该值为，整个作业则失败，默认值为0. 如果你的应用程序允许丢弃部分输入数据，则该该值设为一个大于0的值，比如5，表示如果有低于5%的Map Task失败（如果一个Map Task重试次数超过mapreduce.map.maxattempts，则认为这个Map Task失败，其对应的输入数据将不会产生任何结果），整个作业仍认为成功。
- (5) mapreduce.task.timeout: Task超时时间，默认值为600000毫秒，经常需要设置的一个参数，该参数表达的意思为：如果一个task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该task处于block状态，可能是卡住了，也许永远会卡主，为了防止因为用户程序永远block住不退出，则强制设置了一个该超时时间（单位毫秒）。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是：`AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster`    


## 三 本地运行mapreduce 作业

设置以下几个参数:
- mapreduce.framework.name=local
- mapreduce.jobtracker.address=local
- fs.defaultFS=local

## 四 效率和稳定性相关参数

- (1) mapreduce.map.speculative: 是否为Map Task打开推测执行机制，默认为true，如果为true，如果Map执行时间比较长，那么集群就会推测这个Map已经卡住了，会重新启动同样的Map进行并行的执行，哪个先执行完了，就采取哪个的结果来作为最终结果，一般直接关闭推测执行
- (2) mapreduce.reduce.speculative: 是否为Reduce Task打开推测执行机制，默认为true，如果reduce执行时间比较长，那么集群就会推测这个reduce已经卡住了，会重新启动同样的reduce进行并行的执行，哪个先执行完了，就采取哪个的结果来作为最终结果，一般直接关闭推测执行
- (3) mapreduce.input.fileinputformat.split.minsize: FileInputFormat做切片时的最小切片大小，默认为0
- (4)mapreduce.input.fileinputformat.split.maxsize:  FileInputFormat做切片时的最大切片大小（已过时的配置，2.7.5当中直接把这个配置写死了，写成了Integer.maxValue的值）（切片的默认大小就等于blocksize，即 134217728)
