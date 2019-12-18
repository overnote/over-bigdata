## 一 MapReduce的combiner

每一个 map 都可能会产生大量的本地输出，Combiner 的作用就是对 map 端的输出先做一次合并，以减少在 map 和 reduce 节点之间的数据传输量，以提高网络IO 性能，是 MapReduce 的一种优化手段之一。  

- combiner 是 MR 程序中 Mapper 和 Reducer 之外的一种组件
- combiner 组件的父类就是 Reducer

combiner 和 reducer 的区别在于运行的位置：combiner 是在每一个 maptask 所在的节点运行 Reducer 是接收全局所有Mapper的输出结果。  

combiner 的意义就是对每一个 maptask 的输出进行局部汇总，以减小网络传输量。  

具体实现步骤：
- 1、自定义一个 combiner 继承 Reducer，重写 reduce 方法 
- 2、在 job 中设置：  job.setCombinerClass(CustomCombiner.class)

combiner 能够应用的前提是不能影响最终的业务逻辑，而且，combiner 的输出 kv 应该跟 reducer 的输入 kv 类型要对应起来。  

