## 一 多job串联

一个稍复杂点的处理逻辑往往需要多个mapreduce程序串联处理，多job的串联可以借助mapreduce框架的JobControl实现  

```java
        ControlledJob cJob1 = new ControlledJob(job1.getConfiguration());
        ControlledJob cJob2 = new ControlledJob(job2.getConfiguration());
        ControlledJob cJob3 = new ControlledJob(job3.getConfiguration());
        cJob1.setJob(job1);
        cJob2.setJob(job2);
        cJob3.setJob(job3);
        // 设置作业依赖关系
        cJob2.addDependingJob(cJob1);
        cJob3.addDependingJob(cJob2);
        JobControl jobControl = new JobControl("RecommendationJob");
        jobControl.addJob(cJob1);
        jobControl.addJob(cJob2);
        jobControl.addJob(cJob3);
 
 
        // 新建一个线程来运行已加入JobControl中的作业，开始进程并等待结束
        Thread jobControlThread = new Thread(jobControl);
        jobControlThread.start();
        while (!jobControl.allFinished()) {
            Thread.sleep(500);
        }
        jobControl.stop();
 
        return 0;
```

