## 一 Zookeeper的shell操作

#### 1.1 客户端连接

```
zkCli.sh        # 可选项有 -server ip
```

连接上以后，可以通过一些命令操作Zookeeper：
```
ls /            # 从根节点查看当前存在哪些节点
help            # 输出zookepper帮助提示
```

#### 1.2 节点的增删改查

创建节点：
```
语法：
    create [-s] [-e] path data acl
示例：创建一个永久节点 abc，其数据为 hello
    create /abc hello  
参数：
    -s：按顺序创建节点，如上述示例添加`-s`手，生成的节点名将会是abc0000000001，依次递增
    -e：生成的节点是临时的，当客户端断开后，节点消失，往往配个wathc机制使用
```

查询节点：
```
语法：
    ls path [watch]
    ls2 path [watch]         # 将会查出节点的统计信息
示例：查看根节点下有哪些节点
    ls /
```

查询节点数据：
```
语法：
    get path [watch]           
示例：查询创建的abc节点的数据，得到的结果将是 hello
     get /abc 
```

设置节点数据：
```
语法：data是要更新的新内容，version表示数据版本。
    set path data [version]
示例：
    set /abc 123
```

删除节点：
```
语法：
    delete path [version]       # 若删除节点存在子节点，则无法删除，必须先删除子节点，再删除
    rmr path                    # 递归删除节点
```

#### 1.3 其他常用命令

`setquota -n|-b val path`： 对节点增加限制。 
- n:表示子节点的最大个数 
- b:表示数据值的最大长度 
- val:子节点最大个数或数据值的最大长度 
- path:节点路径

`listquota path`：列出指定节点的 quota，子节点个数为 2,数据长度-1 表示没限制

`delquota [-n|-b] path`： 删除 quota

`history`: 列出命令历史  

`redo`：该命令可以重新执行指定命令编号的历史命令,命令编号可以通过history 查看