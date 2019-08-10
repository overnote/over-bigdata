## 一 Zookeeper的JavaAPI

#### 1.0 说明

Zookeeper有原生的javaAPI，但是不是很好用，推荐使用框架:curator-framework。  

Zookeeper 是在 Java 中客户端主类，负责建立与 zookeeper 集群的会话，并提供方法进行操作。  

`org.apache.zookeeper.Watcher Watcher` 接口表示一个标准的事件处理器，其定义了事件通知相关的逻辑，包含 KeeperState 和 EventType 两个枚举类，分别代表了通知状态和事件类型，同时定义了事件的回调方法：process（WatchedEvent event）。

process 方法是 Watcher 接口中的一个回调方法，当 ZooKeeper 向客户端发送一个 Watcher 事件通知时，客户端就会对相应的 process 方法进行回调，从而实现对事件的处理。  

创建maven工程，并导入jar包：
```xml
<!-- <repositories>
    <repository>
      <id>cloudera</id>
      <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
  </repositories> -->
   <dependencies>
    	<dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.12.0</version>
        </dependency>  
		<dependency>
		    <groupId>com.google.collections</groupId>
		    <artifactId>google-collections</artifactId>
		    <version>1.0</version>
		</dependency>
  </dependencies>
  <build>
		<plugins>
			<!-- java编译插件 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

#### 1.1 节点操作

创建永久节点：
```java
/**
	 * 创建永久节点
	 * @throws Exception
	 */
	@Test
	public void createNode() throws Exception {
		RetryPolicy retryPolicy = new  ExponentialBackoffRetry(1000, 1);
        //获取客户端对象
		CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.120.111:2181,192.168.112.110:2181,192.168.120.113:2181", 1000, 1000, retryPolicy);
        //调用start开启客户端操作
		client.start();
	    //通过create来进行创建节点，并且需要指定节点类型
	    client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/hello3/world");
        client.close();
	}
```

创建临时节点：
```java
	/**
	 * 创建临时节点
	 * @throws Exception
	 */
	@Test
	public void createNode2() throws Exception {
		RetryPolicy retryPolicy = new  ExponentialBackoffRetry(3000, 1);
		CuratorFramework client = CuratorFrameworkFactory.newClient("node01:2181,node02:2181,node03:2181", 3000, 3000, retryPolicy);
		client.start();
	    client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath("/hello5/world");
		Thread.sleep(5000);
		client.close();
	}
```

修改节点数据：
```java
	/**
	 * 节点下面添加数据与修改是类似的，一个节点下面会有一个数据，新的数据会覆盖旧的数据
	 * @throws Exception
	 */
	@Test
	public void nodeData() throws Exception {
		RetryPolicy retryPolicy = new  ExponentialBackoffRetry(3000, 1);
		CuratorFramework client = CuratorFrameworkFactory.newClient("node01:2181,node02:2181,node03:2181", 3000, 3000, retryPolicy);
		client.start();
		client.setData().forPath("/hello5", "hello7".getBytes());
		client.close();
	}
```

节点数据查询：
```java
/**
	 * 数据查询
	 */
	@Test
	public void updateNode() throws Exception {
		RetryPolicy retryPolicy = new  ExponentialBackoffRetry(3000, 1);
		CuratorFramework client = CuratorFrameworkFactory.newClient("node01:2181,node02:2181,node03:2181", 3000, 3000, retryPolicy);
		client.start();
		byte[] forPath = client.getData().forPath("/hello5");
		System.out.println(new String(forPath));
		client.close();
	}
```

节点watch机制：
```java
/**
	 * zookeeper的watch机制
	 * @throws Exception
	 */
	@Test
	public void watchNode() throws Exception {
		RetryPolicy policy = new ExponentialBackoffRetry(3000, 3);
		CuratorFramework client = CuratorFrameworkFactory.newClient("node01:2181,node02:2181,node03:2181", policy);
		client.start();
		// ExecutorService pool = Executors.newCachedThreadPool();  
	        //设置节点的cache  
	        TreeCache treeCache = new TreeCache(client, "/hello5");  
	        //设置监听器和处理过程  
	        treeCache.getListenable().addListener(new TreeCacheListener() {  
	            @Override  
	            public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {  
	                ChildData data = event.getData();  
	                if(data !=null){  
	                    switch (event.getType()) { 
	                    case NODE_ADDED:  
	                        System.out.println("NODE_ADDED : "+ data.getPath() +"  数据:"+ new String(data.getData()));  
	                        break;  
	                    case NODE_REMOVED:  
	                        System.out.println("NODE_REMOVED : "+ data.getPath() +"  数据:"+ new String(data.getData()));  
	                        break;  
	                    case NODE_UPDATED:  
	                        System.out.println("NODE_UPDATED : "+ data.getPath() +"  数据:"+ new String(data.getData()));  
	                        break;  
	                          
	                    default:  
	                        break;  
	                    }  
	                }else{  
	                    System.out.println( "data is null : "+ event.getType());  
	                }  
	            }  
	        });  
	        //开始监听  
	        treeCache.start();  
	        Thread.sleep(50000000);
	}
```

