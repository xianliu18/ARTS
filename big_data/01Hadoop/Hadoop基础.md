### Hadoop 基础

### 1. Hadoop 概念
- Hadoop 主要用来解决海量数据的**存储**和海量数据的**分析计算**问题；
- Hadoop 组成：
  - **MapReduce**：负责计算；
  - **Yarn**：负责资源调度；
  - **HDFS**：负责数据存储；
- 运行 Hadoop 原生的 `WordCount`
	- 准备文件：`hadoop-3.2.3/wcinput/myWordCount.txt`
	- 运行程序：`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.3.jar wordcount wcinput/ ./wcoutput`
- MapReduce 分而治之的策略，来处理不具有计算依赖关系的任务
  - Map 阶段，把大数据拆分成若干份小数据，多个程序同时并行计算产生中间结果；
  - Reduce 聚合阶段，通过程序对并行的结果进行最终的汇总计算，得出最终的结果；

#### 1.1 Linux 常用命令
- `scp`：实现服务器与服务器之间的数据拷贝(from server1 to server2)；
  - `scp -r $pdir/$fname $user@host:$pdir/$fname`
  - `-r`：递归拷贝
- `rsync`：主要用于备份和镜像，具有速度快、避免复制相同内容和支持符号链接的优点(from server1 to server2)；
  - `rsync`：只对差异文件做更新；
  - `rsync -av $pdir/$fname $user@host:$pdir/$fname`
  - `-a`：归档拷贝，即保留文件的属性，权限，所有人不变；
  - `-v`：显示复制过程；
- `ssh-copy-id`：将本机的公钥复制到远程机器的`authorized_keys`文件中；
  - `ssh-copy-id -i ~/.ssh/id_rsa.pub $user@host`
  - `-i`：指定公钥文件；

#### 1.2 Hadoop 配置文件说明
- 默认配置文件
  - `hadoop-commons-3.2.3.jar/core-default.xml`
  - `hadoop-hdfs-3.2.3.jar/hdfs-default.xml`
  - `hadoop-yarn-common-3.2.3.jar/yarn-default.xml`
  - `hadoop-mapreduce-client-core-3.2.3.jar/mapred-default.xml`
- 自定义配置文件
  - `$HADOOP_HOME/etc/hadoop/core-site.xml`
  - `$HADOOP_HOME/etc/hadoop/hdfs-site.xml`
  - `$HADOOP_HOME/etc/hadoop/yarn-site.xml`
  - `$HADOOP_HOME/etc/hadoop/mapred-site.xml`

#### 1.3 自定义配置文件内容
1，HDFS 配置，目录位置：`$HADOOP_HOME/etc/hadoop/`
- `hadoop-env.sh`
```sh
export JAVA_HOME=/opt/jdk1.8.0_181
```

- `core-site.xml`
```xml
<configuration>
	<property>
		<!-- 指定HDFS中NameNode的地址 -->
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:8020</value>
	</property>
		<!-- 指定Hadoop运行时产生文件的存储目录，hadoop启动时，会自动创建 -->
		<property>
			<name>hadoop.tmp.dir</name>
			<value>/(自定义路径）/hadoop-3.2.3/data/tmp</value>
		</property>
</configuration>
```

- `hdfs-site.xml`
  - 指定副本数量
```xml
 <configuration>
    <!-- nameNode 文件的副本数量 -->
     <property>
         <name>dfs.replication</name>
         <value>1</value>
     </property>
     <!-- nameNode web 端访问地址-->
     <property>
         <name>dfs.namenode.http-address</name>
         <value>localhost:9870</value>
     </property>
	 <!-- 2nn web 端访问地址-->
     <property>
         <name>dfs.namenode.secondary.http-address</name>
         <value>localhost:9868</value>
     </property>
 </configuration>
```

- `yarn-site.xml`
```xml
<configuration>
	<!-- Reducer 获取数据的方式-->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>0.0.0.0</value>
	</property>
	<!-- 指定YARN 的 ResourceManager 的地址-->
	<property>
		<name>yarn.resourcemanager.webapp.address</name>
		<value>${yarn.resourcemanager.hostname}:8090</value>
	</property>
	<!-- 开启日志聚集功能 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<!-- 设置日志聚集服务器地址 -->
	<property>
		<name>yarn.log.server.url</name>
		<value>http://hadoop102:19888/jobhistory/logs</value>
	</property>
	<!-- 日志保留7天 -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
</configuration>
```

- `mapred-site.xml`
```xml
<configuration>
	<!-- 指定MR运行在YARN上 -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<!-- 历史记录服务器端地址 -->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>localhost:10020</value>
	</property>
	<!-- 历史服务器web端地址 -->
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>0.0.0.0:19888</value>
	</property>
</configuration>
```

- 配置 `workers`
  - `vim $HADOOP_HOME/etc/hadoop/workers`
```
hadoop101
hadoop102
hadoop103
```

#### 1.4 启动服务
1. `bin/hdfs namenode -format`：格式化命令；第一次启动 Hadoop 时，需要将单节点（或集群）中的`/data`,`/logs` 目录删除；
2. `bin/hdfs dfs -put 本地文件 hdfs目录`：将本地文件上传至 HDFS 中；
3. `sbin/hadoop-daemon.sh start(stop) namenode`：启动（或停止）单节点 NameNode；
4. `sbin/hadoop-daemon.sh start(stop) datanode`：启动（或停止）单节点 DataNode；
5. `sbin/yarn-daemon.sh start(stop) resourcemanager`：启动（或停止）单节点 ResourceManager；
6. `sbin/yarn-daemon.sh start(stop) nodemanager`：启动（或停止）单节点 NodeManager；
7. `sbin/mr-jobhistory-daemon.sh start(stop) historyserver`：启动（或停止）单节点 History；
8. `sbin/start-dfs.sh`：启动 HDFS 集群（注意：所有节点均已配置 `$HADOOP_HOME/etc/hadoop/slaves`）；
9. `sbin/start-yarn.sh`：启动 YARN 集群（注意：所有节点均已配置 `$HADOOP_HOME/etc/hadoop/slaves`），即启动集群的 `ResourceManager` 和 `NodeManagers`；
10. `sbin/stop-dfs.sh`：停止 HDFS 集群；
11. `sbin/stop-yarn.sh`：停止 YARN 集群；
12. `bin/mapred --daemon start historyserver`：启动历史服务器；

#### 1.5 基本测试
- 上传小文件
```sh
# 创建文件夹
hadoop fs -mkdir /input

# 上传文件
hadoop fs -put demo.txt /input

# 跳过回收站，删除文件
hdfs fs -rm -r -skipTrash /input/demo.txt
```

- 使用 HDFS 的文件，运行 `wordcount` 程序
```sh
# wcoutput：指定结果输出目录
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.3.jar wordcount wcinput/demo.txt wcoutput
```

- 常用端口号：
```markdown
1，Hadoop 3.x
	- HDFS NameNode 内部常用端口：8020/9000/9820
	- HDFS NameNode 对用户的查询端口：9870
	- Yarn 查看任务运行情况：8088
	- 历史服务器：19888

2，Hadoop 2.x
	- HDFS NameNode 内部常用端口：8020/9000
	- HDFS NameNode 对用户的查询端口：50070
	- Yarn 查看任务运行情况：8088
	- 历史服务器：19888
```

- 常用配置文件：
```markdown
1，Hadoop 3.x
	- core-site.xml  hdfs-site.xml  yarn-site.xml  mapred-site.xml  workers

2，Hadoop 2.x
	- core-site.xml  hdfs-site.xml  yarn-site.xml  mapred-site.xml  slaves
```

### 2. HDFS
- HDFS 是分布式文件管理系统，适合**一次写入，多次读出**的场景；

#### 2.1 HDFS 组成

![hdfs架构](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/hdfs架构.png)

- **NameNode(nn)**：存储文件的**元数据**，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的 DataNode 等；处理客户端读写请求；
- **DataNode(dn)**：在本地文件系统存储文件块数据，以及块数据的校验和；执行数据块的读写操作；
- **Secondary NameNode(2nn)**：并非 NameNode 的热备，当 NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务；
- **Client**：
  - 文件切分，文件上传 HDFS 时，Client 将文件切分成一个个小文件；
  - 与 NameNode 交互，获取文件的位置信息；
  - 与 DataNode 交互，读取或写入数据；
  - Client 提供一些命令来管理 HDFS，比如 NameNode 格式化；
  - Client 通过一些命令来访问 HDFS，比如对 HDFS 增删改查操作；

#### 2.2 HDFS 的 Shell 操作
- 基本语法
  - `hadoop fs 具体命令` 或者 `hdfs dfs 具体命令`
```sh
# 查看帮助命令
hadoop fs -help rm

# 创建目录
hadoop fs -mkdir /input

# 上传文件，将本地 demo.txt 复制到 hdfs 的 input 目录
hadoop fs -put ./demo.txt /input

# 下载文件
hadoop fs -get /bigdata/test.log ./test0103.log
```

#### 2.3 HDFS 读写流程

1，写数据流程
![hdfs 写入流程](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/hdfs写入流程.png)

**流程说明：**
- Client 发送文件上传请求，通过 RPC 与 NameNode 建立通讯，NameNode 检查目标文件是否存在，目录是否存在，返回是否可以上传；
- 客户端上传之前对文件进行切分，切片规则：按 DataNode 的 block 块大小进行切片；Client 请求第一个 block 该传输到哪些 DataNode 服务器上；
- NameNode 根据配置文件中指定的备份数量及副本放置策略进行文件分配，返回可用的 DataNode 地址，如 A，B，C；
- Client 请求 3 台 DataNode 中的一台 A 上传数据（本质上是一个 RPC 调用，建立 pipeline），A 收到请求会继续调用 B，然后 B 调用 C，将整个 pipeline 建立完成后，逐级返回 Client；
- Client 开始往 A 上传第一个 block（先从磁盘读取数据放到一个本地内存缓存），以 packet 为单位（默认 64K），A 收到一个 packet，就会传给 B，B 传给 C；A 每传一个 packet 会放入一个应答队列等待应答；
- 数据被分割成一个个 packet 数据包在 pipeline 上依次传输，在 pipeline 反方向上，逐个发送 ack（ack 应答机制），最终由 pipeline 中第一个 DataNode 节点 A 将 pipeline ack 发送给 Client；
- 当一个 block 传输完成之后，Client 再次请求 NameNode 上传第二个 block 到服务器。

2，读数据流程
![读数据流程](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/hdfs读取流程.png)

**流程说明：**
- Client 向 NameNode 发送 RPC 请求，来确定请求文件 block 所在位置；
- NameNode 会视情况返回文件的部分或者全部 block 列表，对于每个 block，NameNode 都会返回含有该 block 副本的 DataNode 地址；
- 这些返回的 DataNode 地址，会按照集群拓扑结构得出 DataNode 与客户端的距离，然后进行排序，排序两个规则：网络拓扑结构中距离 Client 近的靠前；心跳机制中超时汇报的 DataNode 状态为 STALE，排在后面；
- Client 选取排序靠前的 DataNode 来读取 block，如果客户端本身就是 DataNode，那么将从本地直接获取数据；底层上本质是建立 Socket Stream（FSDataStream），重复的调用父类 DataInputStream 的 read 方法，直到这个块上的数据读取完毕；
- 当读完列表的 block 后，若文件读取还没有结束，客户端会继续向 NameNode 获取下一批的 block 列表；
- 读取完一个 block 都会进行 checksum 验证，如果读取 DataNode 时出现错误，客户端会通知 NameNode，然后再从下一个拥有该 block 副本的 DataNode 继续读；
- read 方法是并行的读取 block 信息，不是一块一块的读取；NameNode 只是返回 Client 请求包含块的 DataNode 地址，并不是返回请求块的数据；
- 最终读取来所有的 block 会合并成一个完成的最终文件；

3，Edits 和 Fsimage
- FsImage：是 NameNode 中关于元数据的镜像，一般称为**检查点（checkpoint）**，包含了整个 HDFS 文件系统的所有目录和文件信息
  - 对于文件来说，包括数据块描述信息，修改时间，访问时间等；
  - 对于目录来说，包括修改时间，访问权限控制信息（目录所属用户，所在组）等；
- Edits：记录客户端对 HDFS 的添加、删除、重命名、追加等操作；
- FsImage 和 Edits 主要用于在集群启动时，将集群的状态恢复到关闭前的状态；换句话说，集群启动时，会将 FsImage、Edits 加载到内存中，进行合并，合并后恢复完成。

![FsImage 和 Edits 关系](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/FsImages和Edits关系.png)

### 3. MapReduce
#### 3.1 概述
- MapReduce 是一个分布式运算程序的编程框架，是用户开发“基于 Hadoop 的数据分析应用”的核心框架；
- 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上；

1，MapReduce 进程
一个完整的 MapReduce 程序在分布式运行时，有三类实例进程：
	- **MrAppMaster**：负责整个程序的过程调度及状态协调；
	- **MapTask**：负责 Map 阶段的整个数据处理流程；
	- **ReduceTask**：负责 Reduce 阶段的整个数据处理流程；

2，MapReduce 编程步骤
- 用户编写的程序分为三部分：Mapper、Reducer 和 Driver；
- Mapper 阶段
  - 用户自定义的 Mapper 类要继承 Hadoop 的`Mapper<inputKey, inputValue, outputKey, outputValue>`；
  - Mapper 的输入数据是 KV 键值对形式（KV 的类型可自定义）；
  - Mapper 的业务逻辑写在`map()`方法中；
  - Mapper 的输出数据是 KV 键值对形式（KV 的类型可自定义）；
  - `map()`方法（MapTask 进程）对每一个`<K, V>`调用一次，即会对文件的每一行调用一次`map()`方法；

- Reducer 阶段
  - 用户自定义的 Reducer 类要继承 Hadoop 的`Reducer<inputKey, inputValue, outputKey, outputValue>`；
  - Reducer 的输入数据类型对应 Mapper 的输出数据类型；
  - Reducer 的业务逻辑写在`reduce()`方法中；
  - ReduceTask 进程对每一组相同 k 的`<K, V>`组调用一次`reduce()`方法；

- Driver 阶段
  - 相当于 YARN 集群的客户端，用于提交我们整个程序到 YARN 集群，提交的是封装了 MapReduce 程序相关运行参数的 job 对象；

#### 3.2 WordCount 实战
1，创建 maven 工程，`MapReduceDemo`；
2，在 `pom.xml` 文件中添加依赖：
<details>
<summary>pom.xml</summary>

```xml
<dependencies>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>3.2.3</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.30</version>
    </dependency>
  </dependencies>
```
</details>

3，创建 log4j 配置文件`src/main/resources/log4j.properties`
<details>

<summary>配置文件：src/main/resources/log4j.properties</summary>

```properties
## 输出到控制台
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d    %p    [%c] - %m%n

## 输出到文件
#log4j.appender.logfile=org.apache.log4j.FileAppender
#log4j.appender.logfile.File=target/spring.log
#log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
#log4j.appender.stdout.layout.ConversionPattern=%d    %p    [%c] - %m%n
```
</details>

4，编写 Mapper，Reducer，Driver
<details>

<summary>WordCountMapper, WordCountReducer, WordCountDriver</summary>

```java
/** 自定义 Mapper */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private Text outK = new Text();

    private IntWritable outV = new IntWritable(1);

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 1, 获取一行
        String line = value.toString();

        // 2, 切割
        String[] words = line.split(" ");

        // 3, 循环写出
        for (String word : words) {
            outK.set(word);
            context.write(outK, outV);
        }
    }
}

/** 自定义 Reducer */
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    private IntWritable outV = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;

        for (IntWritable value : values) {
            sum += value.get();
        }
        outV.set(sum);
        context.write(key, outV);
    }
}

/** 自定义 Driver */
public class WordCountDriver {

    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        // 1, 获取 job
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 2, 设置 jar 包路径
        job.setJarByClass(WordCountDriver.class);

        // 3, 关联 mapper 和 reducer
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        // 4, 设置 map 输出的 kv 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 5, 设置最终输出的 kv 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 6, 设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path("/Users/abc/reduce.log"));
        FileOutputFormat.setOutputPath(job, new Path("/Users/abc/hadoop_result/"));

        // 7, 提交 job
        boolean result = job.waitForCompletion(true);

        System.exit(result ? 0 : 1);
    }
}
```
</details>

#### 3.3 Hadoop 序列化
- **序列化**：是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储到磁盘（持久化）和网络传输；
- **反序列化**：就是将收到的字节序列（或其他数据传输协议）或者是磁盘的持久化数据，转换成内存中的对象。
- 实现自定义 Bean 对象的序列化步骤：

<details>

<summary>实现自定义 Bean 对象的序列化</summary>

```java
// 1，必须实现 Writable 接口；
// 2，反序列化时，需要反射调用空参构造函数，所以必须有空参构造；
// 3，重写序列化方法；
// 4，重写反序列化方法；
// 	  注意：反序列化的顺序和序列化的顺序完全一致；
// 5，要想把结果显示在文件中，需要重写 toString()，可以使用 "\t" 分开，方便后续使用；
// 6，如果需要将自定义的 Bean 放在 KEY 中传输，则还需要实现 Comparable 接口（WritableComparable），
//    因为 MapReduce 框架中的 Shuffle 过程要求 KEY 必须能排序；

public class MyBeanSerial implements Writable {
	// 空参构造
    public MyBeanSerial() {
        super();
    }

	// 重写序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlow);
        out.writeLong(sumFlow);
    }

	// 重写反序列化方法
    @Override
    public void readFields(DataInput in) throws IOException {
		// 注意：反序列化的顺序和序列化的顺序完全一致
        upFlow = in.readLong();
        downFlow = in.readLong();
        sumFlow = in.readLong();
    }
}
```
</details>

#### 3.4 MapReduce 框架原理
![MapReduce 框架原理](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/mapreduce框架原理.png)

- MapTask 的并行度决定 Map 阶段的任务处理并发度，进而影响到整个 Job 的处理速度；
- MapTask 并行度决定机制：
  - 数据块：Block 是 HDFS 物理上把数据分成一块一块的；**数据块是 HDFS 存储数据单位**；
  - 数据切片：数据切片只是逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储；**数据切片是 MapReduce 程序计算输入数据的单位**，一个切片会对应启动一个 MapTask；
- 一个 Job 在 Map 阶段并行度由客户端在提交 Job 时的**切片数**决定；
- 每一个 Split 切片分配一个 MapTask 并行实例处理；
- 默认情况下，**切片大小 = BlockSize**；
- 切片时，不考虑数据集整体，而是逐个针对每一个文件单独切片；

#### 3.4.1 FileInputFormat 实现类
- 抽象类`FileInputFormat`的常见实现类：
  - `TextInputFormat`
  - `NLineInputFormat`
  - `CombineTextInputFormat`
  - 自定义 InputFormat
- `TextInputFormat`:
  - 默认的 FileInputFormat 实现类，按行读取每条记录。
  - 键是存储该行在整个文件中的**起始字节偏移量**，`LongWritable` 类型；
  - 值是这行的内容，不包括任何行终止符（换行符和回车符），`Text` 类型
- `CombineTextInputFormat`
  - 框架默认的`TextInputFormat` 切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个 MapTask，这样如果有大量小文件，就会产生大量的 MapTask，处理效率极其低下；
  - `CombineTextInputFormat`：用于**小文件过多**的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。
  - **虚拟存储切片最大值设置**：`CombineTextInputFormat.setMaxInputSplitSize(job, 4194304); // 4M`；
  
![CombileTextInputFormat 切片机制](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/CombileTextInputFormat切片机制.png)

1，说明：虚拟存储过程和切片过程
- 虚拟存储过程：
```markdown
1，将输入目录下所有文件大小，依次和设置的`setMaxInputSplitSize`值比较，如果不大于设置的最大值，逻辑上划分一个块。
2，如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，
此时，将文件均分成 2 个虚拟内存块(防止出现太小切片)。

例如：`setMaxInputSplitSize`值为 4M，输入文件大小为 8.02M，则先逻辑上分成一个 4M，剩余的大小为 4.02M，
如果按照 4M 逻辑划分，就会出现 0.02M 小的虚拟存储文件，所以将剩余的 4.02M文件切分成(2.01M 和 2.01M)两个文件；
```
- 切片过程：
```markdown
a) 判断虚拟存储的文件大小是否大于 `setMaxInputSplitSize` 值，若大于等于，则单独形成一个切片；
b）如果不大于，则跟下一个虚拟存储文件进行合并，共同形成一个切片；
c）测试举例：有 4 个小文件，分别为 1.7M，5.1M，3.4M 和 6.8M 这四个小文件则虚拟存储之后，形成 6 个文件块，
大小分别为：（1.7M，（2.55M，2.55M），3.4M，（3.4M，3.4M））

最终，会形成 3 个切片，大小分别为：
(1.7 + 2.55)M，（2.55 + 3.4）M，（3.4 + 3.4）M
```

- 测试
<details>
<summary>CombineTextInputFormat 测试</summary>

```java
// 1, 调整之前，切片个数为 4
// 2, WordCountDriver 增加如下代码，并观察运行切片个数
// 如果不设置 InputFormat，默认使用的是 TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class);

// 虚拟存储切片最大值设置 4M
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
```
</details>

#### 3.4.1 MapReduce 详细工作流程
<!--
参考链接：https://www.bilibili.com/video/BV1Qp4y1n7EN?p=94
-->
![MapReduce 详细工作流程](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/MapReduce工作流程.png)
![MapReduce 详细工作流程 2](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/MapReduce工作流程2.png)

#### 3.4.2 Shuffle 机制
- Map 方法之后，Reduce 方法之前的数据处理过程称之为 Shuffle。

![Shuffle 机制]([$$$](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/Shuffle机制.png))

#### 3.4.3 Partition(分区)
- 默认分区方式，是根据 key 的 HashCode 对 ReduceTasks 个数取模得到的；

<details>
<summary>HashPartitioner 源码</summary>

```java
public class HashPartitioner<K, V> extends Partitioner<K, V> {

	public int getPartition(K key, V value, int numReduceTasks) {
		return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
	}
}
```
</details>

- 自定义分区类

<details>
<summary>自定义 Partitioner</summary>

```java
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
	@Override
	public int getPartition(Text text, FlowBean flowBean, int numReduceTasks) {
		String phone = text.toString();
		String prePhone = phone.substring(0, 3);
		int partition;
		if ("136".equals(prePhone)) {
			partition = 0;
		} else if ("182".equals(prePhone)) {
			partition = 1;
		} else {
			partition = 2;
		}
		return partition;
	}
}

// Driver 类，指定自定义分区器
job.setPartitionerClass(CustomPartitioner.class);
// 自定义 Partition 后，要根据自定义 Partitioner 的逻辑设置相应数量的 ReduceTask
job.setNumReduceTasks(3);
```
</details>

#### 分区大小和 reduceTask 关系
```java
// MapTask 方法
public void write(K key, V value) throws IOException, InterruptedException {
	this.collector.collect(key, value, this.partitioner.getPartition(key, value, this.partitions));
}

// Partitioner 抽象类
public abstract class Partitioner<KEY, VALUE> {
	// var3 为 reduceTask 数量
	// 返回结果为 partition 数量
    public abstract int getPartition(KEY var1, VALUE var2, int var3);
}
```

- 如果 `ReduceTask` 数量**大于**`getPartition`的结果数，则会多产生几个空的输出文件`part-r-000xxx`；
- 如果 `ReduceTask` 数量**小于**`getPartition`的结果数，则有一部分分区数据无处安放，会抛出 Exception；
- 如果 `ReduceTask` 的数量为 1，则不管 MapTask 端输出多少个分区文件，最终结果都交给这一个 ReduceTask，最终也就只会产生一个结果文件`part-r-00000`；
- 分区号必须从零开始，逐一累加；

```java
// 假设自定义分区数为 5 (getPartition)，则
/**
 * 1) job.setNumReduceTasks(1);	// 可以正常运行，只会产生一个结果文件 part-r-00000
 * 2) job.setNumReduceTasks(2); // 抛出异常;
 * 3) job.setNumReduceTasks(6);	// 大于 5，程序可以正常运行，会产生空文件；
```

#### 3.4.4 排序
- MapTask 和 ReduceTask 均会对数据**按照 key**进行排序。该操作属于 Hadoop 的默认行为；
- 默认排序是按照**字典顺序**排序，且实现该排序的方法是**快速排序**;
- 对于 MapTask, 它会将处理的结果暂时放到环形缓冲区中，**当环形缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序**,并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行**归并排序**。
- 对于 ReduceTask，它从每个 MapTask 上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写到磁盘上，否则存储在内存中。如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大的文件；当所有数据拷贝完毕后，ReduceTask 统一对内存和磁盘上的所有数据进行一次归并排序。

#### 排序分类
- 部分排序：MapReduce 根据输入记录的键，对数据集排序，保证输出的每个文件内部有序；
- 全排序：最终输出结果只有一个文件，且文件内部有序；实现方式是只设置一个 ReduceTask。但该方法在处理大型文件时，效率极低，因为一台机器处理所有文件，完全丧失了 MapReduce 所提供的并行架构；

#### 自定义排序
- 需要实现 `WritableComparable` 接口，重写 `compareTo` 方法，就可以实现排序；

#### 3.4.5 Combiner 合并
1，Combiner 是 MR 程序中 Mapper 和 Reducer 之外的一种组件；
2，Combiner 组件的父类就是 Reducer；
3，Combiner 和 Reducer 的区别在于运行的位置：
	- Combiner 是在每一个 MapTask 所在的节点运行；
	- Reducer 是接收全局所有 Mapper 的输出结果；
4，Combiner 的意义就是对每一个 MapTask 的输出进行局部汇总，以减少网络传输量；
5，Combiner 能够应用的前提是不能影响最终的业务逻辑，而且，Combiner 的输出 KV 应该跟 Reducer 的输入 KV 类型要对应起来；

#### 3.5 OutputFormat 数据输出
- `OutputFormat` 是 MapReduce 输出的基类，所有实现 MapReduce 输出都实现了 `OutputFormat` 接口；
- Hadoop 默认输出格式 `TextOutputFormat`；
- 自定义 `OutputFormat`；

<details>
<summary>自定义 OutputFormat</summary>

```java
public class ChatRecordWriter extends RecordWriter<Text, NullWritable> {

    private FSDataOutputStream singleOut;
    private FSDataOutputStream groupOut;

    public ChatRecordWriter(TaskAttemptContext job) {
        // 创建两个输出流
        try {
            FileSystem fs = FileSystem.get(job.getConfiguration());
            singleOut = fs.create(new Path("/Users/Documents/single.log"));
            groupOut = fs.create(new Path("/Users/Documents/group.log"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void write(Text key, NullWritable nullWritable) throws IOException, InterruptedException {
        String result = key.toString();
        if (result.contains("single")) {
            singleOut.writeBytes(result);
        } else {
            groupOut.writeBytes(result);
        }
    }

    @Override
    public void close(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        IOUtils.closeStream(singleOut);
        IOUtils.closeStream(groupOut);
    }
}
```
</details>

#### 3.6 MapTask 工作机制
![MapTask 工作机制](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/MapTask工作机制.png)

<!--
MapTask 工作机制：https://www.bilibili.com/video/BV1Qp4y1n7EN/?p=109
-->

**说明：**
- **Read 阶段**：MapTask 通过 InputFormat 获得的 RecordReader，从输入的 inputSplit 中解析出一个个的 key/value；
- **Map 阶段**：将解析出的 key/value 交给用户编写的`map()`函数处理，并产生一系列新的`key/value`；
- **Collect 阶段**：在用户编写的`map()`函数处理完成后，通常会调用`OutputCollector.collect()`将结果输出。在该函数内部，会将生成的`key/value`进行分区(调用 Partitioner)，并写入一个环形内存缓冲区；
- **Spill 阶段**：即“溢写”阶段，当环形缓冲区达到阈值（通常 80%）时，会将内存中数据写入本地磁盘，生成一个临时文件；注意，在将数据写入本地磁盘之前，会先对数据进行一次本地排序，并在必须时，对数据进行合并、压缩等操作；
  - 利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号 partition 进行排序，然后按照 key（Hash 值） 进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序；
  - 按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件`output/spillN.out`（N 表示当前溢写次数）中。如果用户设置了 Combiner，则写入文件钱，对每个分区中的数据进行一次聚集操作；
  - 将分区数据的元信息写到内存索引数据结构`SpillRecord`中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过 1MB，则将内存索引写到文件`output/spillN.out.index`中；
- **Combine 阶段**：当所有数据处理完毕后，MapTask 会对所有临时文件进行一次合并，以确保每个 MapTask 最终只会生成一个数据文件；
	- 当所有数据处理完毕后，MapTask 会将所有临时文件合并成一个大文件，并保存在文件`output/file.out`中，同时生成相应的索引文件`output/file.out.index`；
	- 在文件合并过程中，MapTask 将以分区为单位进行合并。对于某个分区，采用多轮递归合并的方式，每轮合并`io.sort.factor`（默认 10）个文件，并将产生的文件重新加入到待合并列表中，对文件排序后，重复以上过程，直到最终合并成一个大文件。
	- 让每个 MapTask 最终只生成一个数据文件，可以避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

#### 3.7 ReduceTask 工作机制
![ReduceTask 工作机制](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/ReduceTask工作机制.png)

<!--
ReduceTask 工作机制：https://www.bilibili.com/video/BV1Qp4y1n7EN?p=110
-->

- **Copy 阶段**：ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放在内存中；
- **Merge 阶段**：在远程拷贝数据的同时，ReduceTask 启动两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘文件过多；
- **Sort 阶段**：按照 MapReduce 语义，用户编写`reduce()`函数，输入数据是按照 key 进行聚集的一组数据。为了将 key 相同的数据聚集在一起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现了对自己的处理结果进行局部排序，因此，ReduceTask 只需对所有数据进行一个**归并排序**即可；
- **Reduce 阶段**：`reduce()`函数将计算后的结果写到 HDFS 上；

#### 3.7.1 ReduceTask 并行度决定机制
- MapTask 并行度由**切片个数**决定，切片个数由**输入文件和切片规则**决定；
- ReduceTask 数量，可以直接手动设置：
```java
// 默认值为 1
job.setNumReduceTasks(4);
```
- ReduceTask 注意事项
  - `ReduceTask = 0`，表示没有 Reduce 阶段，输出文件个数和 Map 个数一致；
  - ReduceTask 默认值就是 1，所以输出文件个数为一个；
  - 如果数据分部不均匀，就可能在 Reduce 阶段产生数据倾斜；
  - ReduceTask 数量并不是任意设置，还需要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有一个 ReduceTask；

#### 3.8 Join 多种应用
#### 3.8.1 Reduce Join
- **Map 端的主要工作**：为来自不同表或文件的 key/value 对，打标签以区别不同来源的记录。然后用连接字段作为 key，其余部分和新加的标志作为 value，最后进行输出；
- **Reduce 端的主要工作**：在 Reduce 端以连接字段作为 key 的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录（在 Map 阶段已经打标签）分开，最后进行合并就可以了。
- 实际案例：
<details>
<summary>Reduce Join 实际案例</summary>

```java
// 需求描述：将商品信息表中数据根据商品 pid 合并到订单数据表中
/**
 * 	- order.txt(订单数据表 t_order)
 *		id		pid		mount
 *		1001	01		100
 *		1002	02		200
 *		1003	03		500
 *		1004	01		20
 *		1005	02		30
 *		1006	03		60
 *
 *	- product.txt(商品信息表 t_product)
 *		pid		pname
 *		01		苹果
 *		02		oppo
 *		03		荣耀
 */

/**
 * 自定义 TableBean
 */
 public class TableBean implements Writable {
    private String id;
    private String pid;
    private int amount;
    private String pname;
    private String flag;

	// 空参构造函数
    public TableBean() {
    }

    // 省略 get 和 set 方法

    @Override
    public void write(DataOutput output) throws IOException {
        output.writeUTF(id);
        output.writeUTF(pid);
        output.writeInt(amount);
        output.writeUTF(pname);
        output.writeUTF(flag);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.id = in.readUTF();
        this.pid = in.readUTF();
        this.amount = in.readInt();
        this.pname = in.readUTF();
        this.flag = in.readUTF();
    }

    @Override
    public String toString() {
        return id + "\t" + pname + "\t" + amount;
    }
}

/**
 * 自定义 Mapper
 */
 public class TableMapper extends Mapper<LongWritable, Text, Text, TableBean> {

    private String fileName;
    private Text outK = new Text();
    private TableBean outV = new TableBean();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        // 初始化，获取 order，product 表的名称
        FileSplit fileSplit = (FileSplit) context.getInputSplit();
        fileName = fileSplit.getPath().getName();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 1，获取一行
        String line = value.toString();

        String[] split = line.split("\t");

        // 2，判断是哪个文件
        if (fileName.contains("order")) {
            outK.set(split[1]);
            outV.setId(split[0]);
            outV.setPid(split[1]);
            outV.setAmount(Integer.parseInt(split[2]));
            outV.setPname("");
            outV.setFlag("order");
        } else if (fileName.contains("product")) {
            outK.set(split[0]);
            outV.setId("");
            outV.setPid(split[0]);
            outV.setAmount(0);
            outV.setPname(split[1]);
            outV.setFlag("product");
        }

        context.write(outK, outV);
    }
}

/**
 * 自定义 Reducer
 *     输出的 value：NullWritable，表示空值
 */
 public class TableReducer extends Reducer<Text, TableBean, TableBean, NullWritable> {

    @Override
    protected void reduce(Text key, Iterable<TableBean> values, Context context) throws IOException, InterruptedException {
        // 初始化集合
        ArrayList<TableBean> orderBeans = new ArrayList<>();
        TableBean productBean = new TableBean();

        // 循环遍历
        for (TableBean value : values) {
            if ("order".equals(value.getFlag())) {
                TableBean tempTableBean = new TableBean();
                try {
                    BeanUtils.copyProperties(tempTableBean, value);
                } catch (IllegalAccessException | InvocationTargetException e) {
                    throw new RuntimeException(e);
                }
                // 需要先 new 对象，然后添加；
                // 否则，后面添加的对象会覆盖前面的对象，导致最后集合里面只剩下一个对象；
                orderBeans.add(tempTableBean);
            } else {
                try {
                    BeanUtils.copyProperties(productBean, value);
                } catch (IllegalAccessException | InvocationTargetException e) {
                    throw new RuntimeException(e);
                }
            }
        }

        // 循环遍历 orderBeans，赋值 prductName
        for (TableBean orderBean : orderBeans) {
            orderBean.setPname(productBean.getPname());
            context.write(orderBean, NullWritable.get());
        }
    }
}
```
</details>

#### 3.8.2 Map Join
- 使用场景：MapJoin 适用于一张表十分小，另一张表很大的场景。
  - Reduce Join 中，Reduce 端处理过多的表，非常容易产生数据倾斜；
  - 在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能的减少数据倾斜。
- 具体办法：采用`DistributedCache`
  - 在 Mapper 的 setup 阶段，将文件读取到缓存集合中；
  - 在 Driver 驱动类中加载缓存。
```java
// 本地模式，缓存普通文件到 Task 运行节点
job.addCacheFile(new URI("file:/Users/Documents/cache/product.txt"))

// 集群模式，需要设置为 HDFS 路径
job.addCacheFile(new URI("hdfs://hadoop101:8020/cache/product.txt));
```
- 实际案例
<details>
<summary>Map Join 实际案例</summary>

```java
/** 1，修改 Driver 类
 * 	1.1 DritributedCacheDriver 加载缓存文件
 *      加载缓存数据
 *		job.addCacheFile(new URI("file:/Users/Documents/cache/product.txt"));
 *
 *	1.2 Map 端 join 的逻辑，不再需要 Reduce 阶段，因此设置 ReduceTask 数量为 0
 *		job.setNumReduceTasks(0);
 */
public class MapDriver {
    public static void main(String[] args)
            throws IOException, InterruptedException, ClassNotFoundException, URISyntaxException {
        // 1, 获取 job
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 2, 设置 jar 包路径
        job.setJarByClass(MapDriver.class);

        // 3, 关联 mapper 和 reducer
        job.setMapperClass(TableMapper.class);

        // 4, 设置 map 输出的 kv 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        // 5, 设置最终输出 KV 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // 加载缓存数据
        job.addCacheFile(new URI("/Users/map_cache/product.txt"));
        // Map 端 Join 的逻辑，不需要 Reduce 阶段，设置 reduceTask 为 0
        job.setNumReduceTasks(0);

        // 6, 设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path("/Users/map_join/"));
        FileOutputFormat.setOutputPath(job, new Path("/Users/map_result/"));

        // 7, 提交 job
        boolean result = job.waitForCompletion(true);

        System.exit(result ? 0 : 1);
    }
}

/**
 *	2，Mapper 的 setup 方法中，读取缓存的文件数据
 */
public class TableMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
    private HashMap<String, String> productMap = new HashMap<>();
    private Text outK = new Text();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        // 获取缓存的文件，并把文件内容封装到集合中
        URI[] cacheFiles = context.getCacheFiles();

        FileSystem fs = FileSystem.get(context.getConfiguration());
        FSDataInputStream fis = fs.open(new Path(cacheFiles[0]));

        // 从流中读取数据
        BufferedReader reader = new BufferedReader(new InputStreamReader(fis, "UTF-8"));

        String line;
        while (StringUtils.isNotEmpty(line = reader.readLine())) {
            // 切割
            String[] fields = line.split("\t");
            productMap.put(fields[0], fields[1]);
        }

        // 关流
        IOUtils.closeStream(reader);
        IOUtils.closeStream(fis);
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 1，获取一行
        String line = value.toString();

        String[] split = line.split("\t");

        String pName = productMap.get(split[1]);

        // 2，封装
        outK.set(split[0] + "\t" + pName + "\t" + split[2]);
        context.write(outK, NullWritable.get());
    }
}
```
</details>

### 3.9 MapReduce 开发总结
1）输入数据接口：`InputFormat`
  - 默认实现类：`TextInputFormat`;
  - `TextInputFormat`：一次读取一行文本，然后将该行的起始偏移量作为 Key，行内容作为 Value 返回；
  - `CombineTextInputFormat`：可以把多个小文件合并成一个切片处理，提高处理效率；

2）逻辑处理接口：`Mapper`
  - 用户根据业务需求实现其中三个方法：`setup()`、`map()`、`cleanup()`；

3）Partitioner 分区
  - 默认分区`HashPartitioner`，默认按照 key 的 hash 值；

4）Comparable 排序
  - 当我们用自定义的对象作为 key 指定**输出顺序**时，就必须要实现 `WritableComparable` 接口，重写其中的 `compareTo()`方法；

5）Combiner
  - 前提：不影响最终的业务逻辑；
  - 可以提前聚合 map，是解决数据倾斜的一个方法；

6）逻辑处理接口：`Reducer`
  - 用户根据业务需求实现其中的三个方法：`setup()`、`reduce()`、`cleanup()`

7）输出数据接口：`OutputFormat`
  - 默认实现类：`TextOutputFormat`；

### 4. Hadoop 数据压缩
#### 4.1 概述
- 优缺点：
  - 优点：可以减少磁盘 IO，减少磁盘存储空间；
  - 缺点：增加 CPU 开销；
- 压缩原则：
  - 运算密集型的 Job，少用压缩；
  - IO 密集型的 Job，多用压缩

#### 4.2 压缩参数配置
- 为了支持多种压缩/解压缩算法，Hadoop 引入了编码/解码器
  - `DefaultCodec`、`GzipCodec`、`BZip2Codec`、`LzopCodec`、`SnappyCodec`
- 在 Hadoop 中启用压缩：

<details>
<summary>Hadoop 开启压缩参数说明</summary>

```java
// 1, 输入端采用压缩，可以输入：`hadoop checknative` 查看支持的压缩方式
//  core-site.xml
//		io.compression.codecs

// 2, Mapper 输出采用压缩
//	mapred-site.xml
//	mapreduce.map.output.compress	：默认为 false，设置为 true，表示 开启输出压缩
//  mapreduce.map.output.compress.codec		：指定输出编码方式

// 3, Reducer 输出采用压缩
//	mapred-site.xml
//  mapreduce.output.fileoutputformat.compress	：默认为 false，设置为 true，表示开启压缩
// 	mapreduce.output.fileoutputformat.compress.codec  ：指定输出编码方式

// Driver 中配置压缩方式
Configuration conf = new Configuration();

// 开启 map 端输出压缩
conf.setBoolean("mapreduce.map.output.compress", true);

// 设置 map 端输出压缩方式
conf.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
```
</details>

### 5. Yarn
- Yarn 是一个**资源调度平台**，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台；而 MapReduce 等运算程序则相当于运行在操作系统之上的应用程序。
- Yarn 是 Hadoop 的资源管理器：
  - **Resource Manager**：整个集群资源（内存、CPU 等）的资源管理者；
  - **Node Manager**：单个节点服务器的资源管理者；
  - **Application Master**：负责单个任务的资源管理和任务监控；
  - **Container**：任务运行的容器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等；

![Yarn 架构](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/yarn_architecture.gif)

- **Resource Manager**：
  - 处理客户端请求；
  - 监控 NodeManager；
  - 启动或监控 Application Master；
  - 资源的分配与调度；
- **Node Manager**：
  - 管理单个节点上的资源；
  - 处理来自 ResourceManager 的命令；
  - 处理来自 ApplicationMaster 的命令；
- **Application Master**：
  - 为应用程序申请资源并分配给内部的任务；
  - 任务的监控与容错；

#### 5.1 Yarn 工作机制
![Yarn 工作机制](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/yarn工作机制.png)

**详细说明：**
<!--
https://www.bilibili.com/video/BV1Qp4y1n7EN?p=127
-->
1）MR 程序提交到客户端所在的节点；
2）YarnRunner 向 ResourceManager 申请一个 Application；
3）RM 将该应用程序的资源路径返回给 YarnRunner；
4）该程序将运行所需资源提交到 HDFS 上，包括：`Job.split`(可以控制开启多少个 MapTask)，`Job.xml`(任务的启动参数)，`wc.jar`（用户的业务逻辑）；
5）程序资源提交完毕后，申请运行 MRAppMaster；
6）RM 将用户的请求初始化成一个 Task；
7）其中一个 NodeManager 领取到 Task 任务；
8）该 NodeManager 创建容器 Container，并产生 MRAppMaster；
9）Container 从 HDFS 上拷贝资源到本地；
10）MRAppMaster 向 RM 申请运行 MapTask 资源；
11）ResourceManager 就运行 MapTask 任务分配给另外两个 NodeManager，另外两个 NodeManager 分别领取任务并创建容器；
12）MRAppMaster 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动 MapTask，MapTask 对数据分区排序；
13）MRAppMaster 等待所有 MapTask 运行完毕后，向 ResourceManager 申请容器，运行 ReduceTask；
14）ReduceTask 向 MapTask 获取相应分区的数据；
15）程序运行完毕后，MRAppMaster 会向 ResourceManager 申请注销自己；

#### 5.2  作业提交全过程
- HDFS、YARN、MapReduce 三者关系

![HDFS、YARN、MapReduce 三者关系](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/HDFS_YARN_MapReduce三者关系.png)

#### 5.3 Yarn 调度器和调度算法
- Hadoop 作业调度器主要有三种：FIFO、容量(Capacity Scheduler)和公平(Fair Scheduler)；Hadoop3.1.3 默认的资源调度器是 Capacity Scheduler；
- yarn-default.xml

```xml
<property>
	<name>yarn.resourcemanager.scheduler.class</name>
	<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

#### 1，FIFO 调度器
- 单队列，根据提交作业的先后顺序，先来先服务；

#### 2，容量调度器
- 多队列：每个队列可配置一定的资源量，每个队列采用 FIFO 调度策略；
- 容量保证：管理员可为每个队列设置资源最低保证和资源使用上限；
- 灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列；
- 多租户：
  - 支持多用户共享集群和多应用程序同时运行；
  - 为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定；

![yarn 容量调度](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/yarn_容量调度.png)

#### 2.1 容量调度器资源分配算法
1）队列资源分配：
  - 从 root 开始，使用深度优先算法，优先选择**资源占用率最低**的队列分配资源；
2）作业资源分配：
  - 默认按照提交作业的**优先级**和**提交时间**顺序分配资源；
3）容器资源分配
  - 按容器的**优先级**分配资源；
  - 如果优先级相同，按照**数据本地性原则**:
    - 任务和数据在同一节点；
    - 任务和数据在同一机架；
    - 任务和数据不在同一节点也不在同一机架；
![yarn 资源分配算法](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/yarn_容量调度器资源分配.png)

#### 3，公平调度器
- Fair Scheduler 是 Facebook 开发的多用户调度器；
  - 同队列所有任务共享资源，在时间尺度上获得公平的资源；
1）与容量调度器相同点：
  - 多队列：支持多队列作业；
  - 容量保证：管理员可为每个队列设置资源最低保证和资源使用上限；
  - 灵活性
  - 多租户
2）与容量调度器不同点：
  - 核心调度策略不同
    - 容量调度器：优先选择**资源利用率低的队列**；
    - 公平调度器：优先选择对资源的**缺额比例大**的；
    - **缺额**：公平调度器设计的目标是：在时间尺度上，所有作业获得公平的资源。某一时刻一个作业应获取资源和实际获取资源的差距叫“缺额”；
  - 每个队列可以单独设置资源分配方式
    - 容量调度器：FIFO、DRF
    - 公平调度器：FIFO、FAIR、DRF

![公平调度器特点](https://github.com/xianliu18/ARTS/tree/master/big_data/images/hadoop/yarn_公平调度器.png)

#### 5.4 Yarn 常用命令
```sh
# yarn application 查看任务
yarn application -list

# 根据 Application 状态过滤
yarn application -list -appStates FINISHED

# kill application
yarn application -kill application_1672715182274_0001

# 查看日志
yarn logs --applicationId application_1672715182274_0001

# 查看尝试运行的任务
yarn applicationattempt -status <ApplicationAttemptId>

# 查看容器
yarn container -list <ApplicationAttemptId>
```

#### 5.5 YARN 生产环境核心配置参数
1）`ResourceManager`相关：
- `yarn.resourcemanager.scheduler.class`：默认为容量调度器；
  - 并发度要求高，选择公平调度器；CDH 默认调度器；
  - 并发度要求低，选择容量调度器；
- `yarn.resourcemanager.scheduler.client.thread-count`：ResourceManager 处理调度器请求的线程数量，默认为 50；

2）`NodeManager`相关
- `yarn.nodemanager.resource.detect-hardware-capabilities`：是否让 yarn 自己检测硬件进行配置，默认 false；
- `yarn.nodemanager.resource.count-logical-processors-as-cores`：是否将虚拟核数当作 CPU 核数，默认 false；
- `yarn.nodemanager.resource.pcores-vcores-multiplier`：虚拟核数和物理核数乘数，例如：4 核 8 线程，该参数值应设为 2，默认 1.0
- `yarn.nodemanager.resource.memory-mb`：NodeManager 使用的内存，默认 8G；
- `yarn.nodemanager.resource.system-reserved-memory-mb`：NodeManager 为系统保留多少内存，以上两个参数配置一个即可；
- `yarn.nodemanager.resource.cpu-vcores`：NodeManager 使用 CPU 核数，默认 8 个；
- `yarn.nodemanager.pmem-check-enabled`：是否开启物理内存检查限制 container，默认打开；
- `yarn.nodemanager.vmem-check-enabled`：是否开启虚拟内存检查限制 container，默认打开；
- `yarn.nodemanager.vmem-pmem-ratio`：虚拟内存物理内存比例，默认 2:1

3）Container 相关
- `yarn.scheduler.minimum-allocation-mb`：容器最小内存，默认 1 G；
- `yarn.scheduler.maximum-allocation-mb`：容器最大内存，默认 8 G；
- `yarn.scheduler.minimum-allocation-vcores`：容器最小 CPU 核数，默认 1 个；
- `yarn.scheduler.maximum-allocation-vcores`：容器最大 CPU 核数，默认 4 个；

#### 5.6 Yarn
- Yarn 的工作机制
- Yarn 的调度器

### 6. 核心参数
#### 6.1 HDFS 核心参数
1）NameNode 内存生产配置
- 每个文件块大概占用 150 byte；
- Hadoop 2.x 内存默认 2000M，如果服务器内存 4G，NameNode 内存可以配置 3G，在 hadoop.env.sh 中配置为：`HADOOP_NAMENODE_OPTS=-Xmx3072m`
- Hadoop 3.x 内存是 JVM 根据服务器内存动态分配的；
  - NameNode 最小值为 1G，每增加 1000,000 个 block，增加 1G 内存；
  - DataNode 最小值为 4G，一个 dataNode 上副本总数低于 4000,000 调为 4G，超过 4000,000，每增加 1000,000，增加 1G；
  - 具体修改：`hadoop-env.sh`
```sh
# NAMENODE
export HDFS_NAMENODE_OPTS="-Dhadoop.security.logger=INFO,RFAS -Xmx1024m"

# DATANODE
export HDFS_DATANODE_OPTS="-Dhadoop.security.logger=ERROR,RFAS -Xmx1024m"
```

2）NameNode 心跳并发配置
- hdfs-site.xml
```xml
<!-- NameNode 有一个工作线程池，用来处理不同 DataNode 的并发心跳以及客户端并发的元数据操作 
企业经验：count = 20 * log(Cluster Size)
-->
<property>
	<name>dfs.namenode.handler.count</name>
	<value>21</value>
</property>
```
#### 6.2 HDFS 集群压测
- HDFS 的读写性能主要受**网络和磁盘**的影响较大；

#### 测试写性能
```sh
hadoop jar $HADOOP_HOME/hadoop-3.2.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.2.3-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128M
```

#### 测试读性能
```sh
hadoop jar $HADOOP_HOME/hadoop-3.2.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.2.3-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128M
```

#### 6.3 HDFS 存储优化
- 解决小文件存储方法之一：
  - HDFS 存档文件或 HAR 文件，它将文件存入 HDFS 块，在减少 NameNode 内存使用的同时，允许对文件进行透明的访问。具体来说，HDFS 存档文件对内还是一个个独立文件，对 NameNode 而言，却是一个整体，减少了 NameNode 的内存；
```sh
# 把 /input 目录的所以文件归档为 input.har，并把归档后的文件存储走的 /output 路径下
hadoop archive -archiveName input.har -p /input /output

# 查看文件
hadoop fs -ls har:///output/input.har

# 解压文件
hadoop fs -cp har:///output/input.har/abc.txt /
```

#### 6.4 Hadoop 小文件优化
#### Hadoop 小文件弊端
- Hadoop 上每个小文件都要在 NameNode 上创建对应的元数据，这个元数据的大小约为 150 byte，这样当小文件比较多的时候，就会产生**很多的元数据文件**，一方面会大量占用 NameNode 的内存空间，另一方面就是元数据文件过多，使得寻址索引速度变慢；
- 小文件过多，在进行 MR 计算时，会产生过多切片，需要启动过多的 MapTask。每个 MapTask 处理的数据量小，导致 MapTask 的处理时间比启动时间还小，浪费资源；

#### 解决方案
1）在数据采集的时候，就将小文件或小批数据合成大文件再上传 HDFS；
2）**Hadoop Archive**是一个高效的将小文件放入 HDFS 块中的文件存档工具，能够将多个小文件打包成一个 HAR 文件，从而达到减少 NameNode 内存的使用；
3）`CombineTextInputFormat`用于将多个小文件在切片过程中，生成一个单独的切片或者少量切片；
4）开启 **uber 模式**，实现 JVM 重用。默认情况下，每个 Task 任务都需要启动一个 JVM 来运行，如果 Task 任务计算的数据量很小，我们可以让同一个 Job 的多个 Task 运行在一个 JVM 中，不必为每个 Task 都开启一个 JVM；

#### 6.5 MapReduce 压测
```sh
# 使用 RandomWriter 来产生随机数，每个节点运行 10 个 Map 任务，每个 Map 产生大约 1 G 大小的二进制随机数
hadoop jar $HAOOP_HOME/hadoop-3.2.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.3.jar randomwriter random-data

# 执行 sort 程序
hadoop jar $HAOOP_HOME/hadoop-3.2.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.3.jar sort random-data sorted-data

# 验证数据是否真正排好序
hadoop jar $HAOOP_HOME/hadoop-3.2.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.2.3-tests.jar testmapredsort -sortInput -random-data -sortOutput sorted-data
```