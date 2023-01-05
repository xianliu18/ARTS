<!--
### Hive 基础
（2023.01.06 ~ 2023.01.10）

### 1. Hive 概述
- Hive 是 Facebook 开源，基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供**类 SQL**查询功能；
- Hive 本质是一个 Hadoop 客户端，用于**将 HQL（Hive SQL）转化为 MapReduce 程序**。
	- Hive 中每张表的数据存储在 **HDFS**；
	- Hive 分析数据底层的实现是 MapReduce（也可配置为 Spark 或者 Tez）；
	- 执行程序运行在 Yarn 上；

#### 1.1 Hive 架构

- https://www.bilibili.com/video/BV1g84y147sX?p=3

