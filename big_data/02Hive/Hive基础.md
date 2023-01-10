### Hive 基础
（2023.01.06 ~ 2023.01.10）

### 1. Hive 概述
- Hive 是 Facebook 开源，基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供**类 SQL**查询功能；
- Hive 本质是一个 Hadoop 客户端，用于**将 HQL（Hive SQL）转化为 MapReduce 程序**。
	- Hive 中每张表的数据存储在 **HDFS**；
	- Hive 分析数据底层的实现是 MapReduce（也可配置为 Spark 或者 Tez）；
	- 执行程序运行在 Yarn 上；

#### 1.1 Hive 架构
![Hive 架构](../images/02hive/hive_architecture.png)

![Hive 架构 2](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_architecture2.png)

#### 各模块说明
1）元数据：Metastore
- 元数据包括：数据库（默认是 default）、表名、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；
- 默认存储在**derby 数据库中**，但是，derby 只支持单客户端访问；推荐使用 Mysql 存储 Metastore；

2）驱动器：Driver
- 解析器（SQLParser）：将 SQL 字符串转换成抽象语法树（AST）；
- 语义分析（Semantic Analyzer）：将 AST 进一步划分为 QueryBlock；
- 逻辑计划生成器（Logical Plan Gen）：将语法树生成逻辑计划；
- 逻辑优化器（Logical Optimizer）：对逻辑计划进行优化；
- 物理计划生成器（Physical Plan Gen）：根据优化后的逻辑计划生成物理计划；
- 物理优化器（Physical Optimizer）：对物理计划进行优化；
- 执行器（Execution）：执行该计划，得到查询结果并返回给客户端；

#### 1.2 Hive 安装
```sh
# 解压文件
tar -zxvf apache-hive-3.1.3-bin.tar.gz

# 重命名文件为 hive-3.1.3

# ~/.bashrc 添加环境变量
export HIVE_HOME=/home/Documents/hive-3.1.3
export PATH=$PATH:$HIVE_HOME/bin

# source 配置文件
source ~/.bashrc

# 初始化 derby 数据库
bin/schematool -dbType derby -initSchema
```

#### 1.3  配置 Hive 元数据存储到 MySQL
1）新建 Hive 元数据库`hive_meta`；
2）将 MySQL 的 JDBC 驱动拷贝到 Hive 的 lib 目录下
  - `cp mysql-connector-java-5.7.39.jar $HIVE_HOME/lib`
3）在`$HIVE_HOME/conf`目录下，新建`hive-site.xml`文件
<details>
<summary>hive-site.xml 详细内容</summary>

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- jdbc 连接的 URL -->
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://localhost:3306/hive_meta?useSSL=false</value>
	</property>
	<!-- jdbc 连接的 Driver -->
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<!-- jdbc 连接的 username -->
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>
	<!-- jdbc 连接的 password -->
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>root</value>
	</property>
	<!-- hive 默认在 HDFS 的工作目录 -->
	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/user/hive/warehouse</value>
	</property>
</configuration>
```
</details>

4）初始化 Hive 源数据库
  - `bin/schematool -dbType mysql -initSchema -verbose`

#### 1.4 Hive 服务部署
#### 1.4.1 Hiveserver2 服务
- Hiveserver2 服务的作用是提供 jdbc/odbc 接口，为用户提供**远程访问** hive 数据的功能；
- `hive.server2.enable.doAs`：是否启用用户模拟功能；
  - 若启用，则 Hiveserver2 会模拟成客户端 A、客户端 B、客户端 C 的登录用户去访问 Hadoop 集群数据；
  - 若未启用，则会直接使用启动 Hadoop 集群的用户访问集群内数据；

![hiveserver2 作用](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hiveserver2作用.png)

#### 1.4.2 Hiveserver2 部署
1）Hadoop 端配置
- hiveserver2 的模拟用户功能，依赖于 Hadoop 提供的 proxy user(代理用户功能)，只有 Hadoop 中的代理用户才能模拟其他用户的身份访问 Hadoop 集群；因此，需要将**hiveserver2 的启动用户**设置为 Hadoop 的代理用户；
- `vim $HADOOP_HOME/etc/hadoop/core-site.xml`

<details>
<summary>core-site.xml 修改</summary>

```xml
<!-- 配置所有节点的 XXX(修改为自己的用户名称) 用户都可作为代理用户 -->
<property>
	<name>hadoop.proxyuser.XXX.hosts</name>
	<value>*</value>
</property>

<!-- 配置 XXX(修改为自己的用户名称) 用户能够代理的用户组为任意组 -->
<property>
	<name>hadoop.proxyuser.XXX.groups</name>
	<value>*</value>
</property>

<!-- 配置 XXX(修改为自己的用户名称) 用户能够代理的用户为任意用户 -->
<property>
	<name>hadoop.proxyuser.XXX.users</name>
	<value>*</value>
</property>
```
</details>

2）Hive 端配置
- `hive-site.xml`
<details>
<summary>hive-site.xml 配置</summary>

```xml
<!-- 指定 hiveserver2 连接的 host -->
<property>
	<name>hive.server2.thrift.bind.host</name>
	<value>127.0.0.1</value>
</property>

<!-- 指定 hiveserver2 连接的端口号 -->
<property>
	<name>hive.server2.thrift.port</name>
	<value>10000</vaue>
</property>
```
</details>

3）测试
<details>
<summary> 测试过程 </summary>

```sh
# 后台启动 hiveserver2
 nohup bin/hive --service hiveserver2 1>/dev/null 2>&1 &

# 使用 beeline 客户端，进行远程访问
# 启动 beeline 客户端
bin/beeline

# 连接客户端
!connect jdbc:hive2://127.0.0.1:10000

# 输入用户名
abc
# 输入密码，此处，可直接回车

# 退出客户端
!quit
```
</details>

#### 1.4.3 metastore 服务
- Hive 的 metastore 服务的作用是为 Hive CLI 或者 Hiveserver2 提供元数据访问接口；
- metastore 有两种运行模式，分别为嵌入式模式和独立服务模式：

![metastore 嵌入式模式](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_metastore嵌入.png)

![metastore 独立模式](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_metastore独立.png)

#### 嵌入式模式缺点：
- 嵌入式模式下，每个 Hive CLI 都需要直接连接源数据库，当 Hive CLI 较多时，数据库压力会比较大；
- 每个客户端都需要用户元数据库的读写权限，元数据库的安全得不到很好的保证；

#### 独立服务模式配置
1）保证 metastore 服务的配置文件 `hive-site.xml`，包含连接元数据库所需的以下参数：
<details>
<summary>hive-site.xml 配置</summary>

```xml
<!-- jdbc 连接的 URL -->
<property>
	<name>javax.jdo.option.ConnectionURL</name>
	<value>jdbc:mysql://localhost:3306/hive_meta?useSSL=false</value>
</property>
<!-- jdbc 连接的 Driver -->
<property>
	<name>javax.jdo.option.ConnectionDriverName</name>
	<value>com.mysql.jdbc.Driver</value>
</property>
<!-- jdbc 连接的 username -->
<property>
	<name>javax.jdo.option.ConnectionUserName</name>
	<value>root</value>
</property>
<!-- jdbc 连接的 password -->
<property>
	<name>javax.jdo.option.ConnectionPassword</name>
	<value>root</value>
</property>
```
</details>

2）保证 Hiveserver2 和每个 Hive CLI 的配置文件`hive-site.xml`中包含访问 metastore 服务所需的以下参数：
<details>
<summary>Hive CLI 的配置文件 hive-site.xml</summary>

```xml
<!-- 指定 metastore 服务的地址 -->
<property>
	<name>hive.metastore.uris</name>
	<value>thrift://127.0.0.1:9083</value>
</property>
```
</details>

3）启动 metastore 服务：`nohup hive --service metastore 1>/dev/null 2>&1 &`；

#### 1.5 Hive 使用技巧
- 查看帮助命令：`hive -H` 或 `hive --help`

#### 1.6 Hive 常见属性配置
1）Hive 客户端显示当前库和表头
- `hive-site.xml`
<details>
<summary>hive-site.xml 配置</summary>

```xml
<property>
	<name>hive.cli.print.header</name>
	<value>true</value>
</property>
<property>
	<name>hive.cli.print.current.db</name>
	<value>true</value>
</property>
```
</details>

2）Hive  运行日志路径配置
- 日志默认路径：`/tmp/${当前用户}/hive.log`；
- 修改后日志位置：`$HIVE_HOME/logs/hive.log`；
- `$HIVE_HOME/conf/hive-log4j2.properties.template` 重命名为：`hive-log4j2.properties`；
- 修改`property.hive.log.dir`属性的值；

3）Hive 的 JVM 堆内存设置
- `$HIVE_HOME/conf/hive-env.sh.template` 重命名为：`hive-env.sh`；
- 修改`HADOOP_HEAPSIZE`的属性值为 2048；

### 2. Hive 基本语法
#### 2.1 DML
- 创建表
```sql
-- 普通建表
create table hive_demo_01;

-- Create Table As Select
-- 允许用户利用 select 查询语句返回的结果，直接建表，表的结构和查询语句的结构保持一致；

-- Create Table Like
-- 允许用户复刻一张已经存在的表结构，不包含数据；

-- 查看完整的建表语句
show create table <表名称>;
```

- EXTERNAL(外部表)
  - 外部表，与之相对应的是内部表（管理表）。管理表意味着 Hive 会完全接管该表，包括元数据和 HDFS 中的数据。而外部表则意味着 Hive 只管元数据，而不完全管 HDFS 中的数据；
- 数据类型
  - Hive 中的字段类型分为**基本数据类型和复杂数据类型**；
- 基本数据类型

Hive 数据类型 | Java 数据类型 |  长度  
------------ |------------- |-------
TINYINT | byte | 1 byte 有符号整数 |
SMALLINT | short | 2 byte 有符号整数 |
INT | int | 4 byte 有符号整数 |
BIGINT | long | 8 byte 有符号整数 |
FLOAT | float | 单精度浮点数 |
DOUBLE | double | 双精度浮点数 |
DECIMAL | DECIMAL | 十进制精准数字类型 |
VARCHAR | VARCHAR | 字符序列，需要指定最大长度 |
STRING | string | 字符系列,<br/>可以使用单引号或双引号|
TIMESTAMP |     | 时间类型 |
BINARY |        | 字节数组 |

- 集合数据类型

Hive 数据类型 | 描述 |  语法示例  
------------ |------------- |-------
STRUCT | 类似于C语言的struct |  |
MAP | map | |
ARRAY | 数组 |  |

- `ROW_FORMAT`
  - 指定 SERDE，SERDE 是 Serializer and Deserializer 的简写；
  - 读数据流程：`HDFS files --> InputFileFormat --> <key, value> --> Deserializer --> Row object`；
  - 写数据流程：`Row object --> Serializer --> <key, value> --> OutputFileFormat --> HDFS files`；
- `STORED AS`
  - 指定文件格式，常用的文件格式有，textfile(默认值)，sequence file，orc file，parquet file等等；

- 建表示例

<details>
<summary>建表案例</summary>

```sql
-- Hive 中默认创建的表是内部表（管理表），Hive 会完全管理表的元数据和数据文件
create table if not exists student (
	id int,
	name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/student';

-- 创建文件 student.txt
-- 1001	张三
-- 1002	李四

-- 上传文件到 hdfs
-- hadoop fs -put student.txt /user/hive/warehouse/student

-- 删除表，会删除 hdfs 上面的文件
drop table student;

-- ############
-- 创建外部表
create external table if not exists student (
	id int,
	name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/student';

-- 删除表，不会删除 hdfs 上面的文件
drop table student;

-- #############
-- SERDE 和复杂数据类型
-- 现有如下格式的 JSON 文件，需要由 Hive 进行分析处理
/*
{
    "name": "张三",
    "friends": [
        "李四",
        "王五"
    ],
    "students": {
        "小郭": 18,
        "老于": 20
    },
    "address": {
        "street": "湖广会馆",
        "city": "北京",
        "postal_code": 10010
    }
}
*/

-- 说明：
-- 	  如果使用 JSON serde，设计表字段时，表的字段需要与 JSON 字符串的一级字段保持一致
create table teacher
(
    name    string,
    friends array<string>,
    students    map<string, int>,
    address     struct<city:string, street:string,postal_code:int>
)
row format serde 'org.apache.hadoop.hive.serde2.JsonSerDe'
location '/user/hive/warehouse/teacher';

-- 准备文件 teacher.txt
-- 上传 hdfs

-- 查询 复杂数据类型：数组，map，struct
select friends[0],students["小郭"],address.city from teacher;

-- ############
-- create table as select
create table teacher_as as select friends[0],students["小郭"],address.city from teacher;

-- create table as like
create table teacher_like like teacher;
```
</details>

#### 2.2 查看表
<details>
<summary>查看表案例</summary>

```sql
-- 查看某些表
show tables like 'stu*';

-- 查看表信息
-- 查看基本信息
desc stu;

-- 查看更多信息
desc formatted student;
```
</details>

#### 2.3 修改表

<details>
<summary>修改表案例</summary>

```sql
-- 重命名表
alter table stu rename to stu_demo;

-- 修改列
-- 增加列
alter table stu add columns(age int);

-- 更新列
alter table stu change column age ages double;

-- 替换列
alter table stu replace columns(id int, name string);
```
</details>

#### 2.4 加载表
- Load
- Insert
- Export&Import
  - `Export`：导出语句可将表的数据和元数据信息一并导出到 HDFS 路径；
  - `Import`：可将 Export 导出的内容导入 Hive，表的数据和元数据信息都会恢复；
  - Export 和 Import 可用于两个 Hive 实例之间的数据迁移；

<details>
<summary>加载表案例</summary>

```sql
-- ### Load：将文件导入到 Hive 表中
-- 案例一：加载本地文件到 hive
load data local inpath '/opt/module/datas/student.txt' into table student;

-- 案例二：overwrite
--		load 默认是在已有数据后面追加新的数据；overwrite表示覆盖表中已有数据
load data local inpath '/opt/module/datas/student.txt' overwrite into table student;

-- 案例二：加载 HDFS 文件到 hive
load data inpath '/user/my_data/student.txt' into table student;

-- ### Insert
--	使用方式有三种：
--		1，将查询结果插入到表中；
--		2，将给定 Values 插入到表中；
--		3，将查询结果写入目标路径；

-- 1，将查询结果插入到表中
insert into table student2 select * from student;

-- 2，将给定 Values 插入到表中
insert into table student2 values(5, '杨过'),(6,'金轮法王');

-- 3，将查询结果写入目标路径
-- 	  会将目标路径内容清空，并写入查询结果
insert overwrite local directory '/home/rituals/Documents/abc/student22.txt'
    row format serde 'org.apache.hadoop.hive.serde2.JsonSerDe'
select id, name from student;

-- ### Export & Import
-- 导出
export table default.student to '/user/hive/warehouse/export/student002';

-- 导入
import table student2 from '/user/hive/warehouse/export/student002';
```
</details>

### 3. Hive 查询语句
- 准备原始数据以及创建相关表

<details>
<summary>准备工作</summary>

```txt
# dept.txt
# 部门编号	部门名称	部门位置
10	行政部	1700
20	财政部	1800
30	教学部	1900
40	销售部	1700

# emp.txt
# 员工编号	姓名	岗位	薪资	部门
7369	李白	研发	800.00	30
7499	鲁班	财务	1600.00	20
7521	孙策	行政	1250.00	10
7566	香香	销售	2975.00	40
7688	伽罗	研发	1250.00	30
7639	马克	研发	2850.00	30
7763	百里	\N	2450.00	30
7729	大乔	行政	3000.00	10
7861	韩信	销售	5000.00	40
7879	刘邦	销售	1500.00	40
7890	项羽	行政	2200.00	10
7821	吕布	讲师	950.00	30
7901	刘备	行政	3000.00	10
7942	关羽	讲师	1300.00	30
```

```sql
-- 创建部门表
create table if not exists dept (
	deptno int,
	dname string,
	loc	int
)
row format delimited fields terminated by '\t';

-- 创建员工表
create table if not exists emp(
	empno	int comment '员工编号',
	ename	string comment '员工姓名',
	job		string comment '员工岗位',
	sal 	double comment '薪资',
	deptno	int	comment	'部门编号'	
)
row format delimited fields terminated by '\t';
```

```sh
# 导入数据
load data local inpath '/home/rituals/Documents/hive_data/dept.txt' into table dept;

load data local inpath '/home/rituals/Documents/hive_data/emp.txt' into table emp;
```
</details>

#### 联合(union & union all)
- `union`：结果去重；
- `union all`：结果不去重；
- `union`和`union all` 在上下拼接 SQL 结果时，有两个要求：
  - 两个 SQL 的结果，列的个数必须相同；
  - 两个 SQL 的结果，上下所对应列的类型必须一致；
- 示例：
```sql
select * from emp where deptno = 30
union
select * from emp where deptno = 40;
```

#### Order By 和 Sort By 区别
- `order by`：全局排序，只会在一个 Reduce 中排序；
- `sort by`：为每个 reduce 产生一个排序文件，每个 Reduce 内部进行排序，对全局结果集来说不是有序；

```sql
-- 测试 sort by
-- 设置 reduce 个数为 3
set mapreduce.job.reduces=3;

-- 将查询结果导出到本地文件
insert overwrite local directory '/home/rituals/Documents/sortby_result'
select * from emp sort by deptno desc;

-- 会产生三个结果文件
```

#### 3.1 Hive  函数
- Hive 提供了大量的内置函数，按照其特点分为如下几类：单行函数、聚合函数、炸裂函数、窗口函数；

```sql
-- 查看系统内置函数
show functions;

-- 查看内置函数用法
desc function upper;

-- 查看内置函数详细信息
desc function extended upper;
```

#### 3.1.1 单行函数
- 单行函数的特点是一进一出，即输入一行，输出一行；
- 单行函数按照功能分为：日期函数、字符串函数、集合函数、数学函数、流程控制函数等；

```sql
-- 流程控制函数
-- case
select 
	stu_id,
	course_id,
	case score
		when 90 then 'A'
		when 80 then 'B'
		when 70 then 'C'
		when 60 then 'D'
	end
from score_info;

-- if
select if(10 < 5, '正确', '错误');
```

#### 3.1.2 炸裂函数
- UDTF(Table-Generating Functions)，接收一行数据，输出一行或多行数据；
- `Lateral View`通常与 UDTF  配合使用。`Lateral View`可以将 UDTF 应用到源表的每行数据，将每行数据转换为一行或多行，并将源表中每行的输出结果与该行连接起来，形成一个虚表；

```sql
-- explode 函数
select explode(array("1","2","3")) as item;

select explode(map("a", 1, "b", 2, "c", 3)) as(key, value);

select posexplode(array("1","2","3")) as (pos, item);

-- inline(Array<Struct<f1:T1,...fn:Tn>> a)
select inline(array(named_struct("id", 1, "name", "张飞", "age", 12),
					named_struct("id", 2, "name", "关羽", "age", 16),
					named_struct("id", 3, "name", "刘备", "age", 18))) as (id, name, age);

-- Lateral View 案例
-- 参考链接：https://www.bilibili.com/video/BV1g84y147sX?p=76
select id,name,hobbies,hobby
from
	person
lateral view explode(hobbies) tmp as hobby;
```

#### 3.1.3 窗口函数
- 窗口函数，能为每行数据划分一个窗口，然后对窗口范围内的数据进行计算，最后将计算结果返回给该行数据；
- 窗口函数按照功能，分为：聚合函数、跨行取值函数、排名函数；
- 语法：窗口函数的语法中主要包括“**窗口**”和“**函数**”两部分。其中“窗口”用于定义计算范围，“函数”用于定义计算逻辑；
  - 绝大多数的聚合函数都可以配合窗口使用，例如`max()`、`min()`、`sum()`、`count()`、`avg()`等；
  - 窗口范围的定义分为两种类型，一种是基于**行**的，一种是基于**值**的；
    - 基于行：要求每行数据的窗口为：上一行到当前行；
    - 基于值：要求每行数据的窗口为：值位于当前值 - 1，到当前值；

<details>
<summary>窗口函数语法</summary>

```sql
-- 参考链接：https://www.bilibili.com/video/BV1g84y147sX?p=78
-- 窗口函数 - 基于行
-- sum(amount) over (order by [column] rows between A and B)
-- 统计每个用户截止每次下单的累计下单总额
select
	order_id,
	user_id,
	user_name,
	order_date,
	order_amount,
	sum(order_amount) over(partition by user_id order by order_date rows between unbounded preceding and current row)
from
	order_info;

-- 统计每个用户截止每次下单的当月累计下单总额
select
	order_id,
	user_id,
	user_name,
	order_date,
	order_amount,
	sum(order_amount) over(partition by user_id,substring(order_date, 1, 7) order by order_date rows between unbounded preceding and current row)
from
	order_info;

-- 窗口函数 - 基于值
-- sum(amount) over (order by [column] range between A and B)

-- 窗口 -- 分区
-- 	定义窗口范围时，可以指定分区字段，每个分区单独划分窗口；
-- sum(amount) over (partition by user_id order by order_date rows between A and B)

-- 窗口 -- 缺省
-- over() 中的三部分内容 partition by、order by、(rows|range) between...and... 均可省略不写
-- partition by 省略不写，表示不分区
-- order by 省略不写，表示不排序
-- between...and... 省略不写，分为两种情况：
--	若 over 中包含 order by，则默认值为：
--		range between unbounded preceding and current row
--  若 over 中不包含 order by，则默认值为：
-- 		rows between unbounded preceding and unbounded following
```
</details>

#### 跨行取值函数
- `lag` 和 `lead`：获取当前行的上/下边某行、某个字段的值；不支持自定义窗口；
- `first_value` 和 `last_value`：获取窗口内某一列的第一个值/最后一个值；

<details>
<summary>跨行取值函数案例</summary>

```SQL
-- lead 和 lag
select
	order_id,
	user_id,
	order_date,
	amount,
	-- order_date: 字段名
	-- 1：偏移量，'1970-01-01'：默认值
	lag(order_date, 1, '1970-01-01') over (partition by user_id order by order_date) last_date,
	lead(order_date, 1, '9999-12-31') over(partition by user_id order by order_date) next_date
from
	order_info;

-- first_value 和 last_value
select
	order_id,
	user_id,
	order_date,
	amount,
	-- false：是否要跳过 NULL 值
	first_value(order_date, false) over (partition by user_id order by order_date) first_date,
	last_value(order_date, false) over(partition by user_id order by order_date) last_date
from
	order_info;
```
</details>

#### 排名函数
- `rank()、dense_rank()、row_number()`：不支持自定义窗口；
<details>
<summary>排名函数案例</summary>

```sql
select
	stu_id,
	course,
	score,
	rank() over(partition by course order by score desc) rk,
	dense_rank() over(partition by course order by score desc) dense_rk,
	row_number() over(partition by course order by score desc) rn
from
	score_info;
```
</details>

#### 3.1.4 自定义函数
- 根据用户自定义函数类别，可以分为以下三种：
  - UDF(User-Defined-Function)：一进一出；
  - UDAF(User-Defined Aggregation Function)：用户自定义聚合函数，多进一出；
  - UDTF(User-Defined Table-Generating Functions)：用户自定义表生成函数，一进多出；

### 4. 分区表和分桶表
- Hive 中的分区就是把一张大表的数据按照业务需要分散的存储到多个目录，每个目录就称为该表的一个分区；
- 创建分区表
```sql
create table dept_partition
(
	deptno int comment '部门编号',
	dname	string	comment '部门名称',
	loc		string  comment '部门位置'
)
	-- 分区语法
	partitioned by (day string)
	row format delimited fields terminated by '\t';

-- 写数据
--	load
load data local inpath '/home/rituals/Documents/hive_data/dept_20220401.log' into table dept_partition partition(day="20220401");

-- insert
insert overwrite table dept_partition partition(day="20220402")
select
    deptno,
    dname,
    loc
from dept_partition
where day="20220401";
```

#### 分区表基本操作
<details>
<summary>分区表基本操作</summary>

```sql
-- 查看所有分区
show partitions customer_info;

-- 创建单个分区
alter table t_single_info add partition(day='20220403');

-- 同时创建多个分区（分区之间不能有逗号）
alter table t_single_info add partition(day='20220404') partition(day='20220405');

-- 删除单个分区
alter table t_single_info drop partition(day='20220403');

-- 删除多个分区(分区之间必须有逗号)
alter table t_single_info drop partition(day='20220404'),partition(day='20220405');

-- 二级分区，按天分区之后，再按小时分区
create table dept_partition2(
	deptno int,
	dname	string,
	loc		string
)
-- 分区中为两个字段：day，hour
partitioned by (day string, hour string)
row format delimited fields terminated by '\t';
```
</details>

#### 4.1 分桶表
- 分区针对的是数据的存储路径，分桶针对的是数据文件；
- 分桶表的基本原理是，首先为每行数据的一个指定字段计算 hash 值，然后以分桶数取模，最后将取模运算结果相同的行，写入同一个文件中，这个文件就称为一个分桶(bucket)；

<details>
<summary>分桶相关语法</summary>

```sql
-- 建表语句
create table stu_buck (
	id  int,
	name  string
)
-- clustered 用于 分桶； sorted 用于 排序
clustered by(id) sorted by(id)
into 4 buckets
row format delimited fields terminated by '\t';

-- 加载数据
load data local inpath '/home/Documents/datas/student.txt' into table stu_buck;
```
</details>

### 5. 文件格式
- Hive 表数据的存储格式，分为：`Text File`、`orc`、`parquet`、`sequence file`；
- `Text File`：Hive 默认的文件格式，文本文件中的一行内容，对应 Hive 表中的一行记录；

#### 5.1 ORC
- `ORC(Optimized Row Columnar)`：是一种**列式存储**的文件格式，ORC 文件能够提高 Hive 读写数据和处理数据的性能；
- 行存储与列存储

![列存储](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_列存储.png)

- ORC 文件基本格式

![ORC 文件基本格式](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_orc文件格式.png)

<details>
<summary>ORC 建表语句</summary>

```sql
-- 建表语句
create table orc_table
(columnA, columnB,...)
stored as orc
tblproperties(property_name=property_value, ...);

-- orc 文件支持的参数：
-- orc.compress：压缩格式，默认 ZLIB，可选 NONE，ZLIB，SNAPPY；
-- orc.compress.size：每个压缩块的大小（ORC 文件是分块压缩的），默认为 256KB；
-- orc.stripe.size：每个 stripe 的大小，默认为 64M；
-- orc.row.index.stride：索引步长（每隔多少行数据建一条索引）
```
</details>

#### 5.2 Parquet 文件
- `Parquet`：也是一个列式存储的文件格式；

![Parquet 文件格式](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_parquet文件格式.png)

<details>
<summary>Parquet 建表语句</summary>

```sql
-- 建表语句
create table parquet_table
(columnA, columnB,...)
stored as parquet
tblproperties(property_name=property_value, ...);

-- parquet.compression：压缩格式，可选项：uncompressed，snappy，gzip，lzo，brotli，lz4；
-- parquet.block.size：行组大小，通常与 HDFS 块大小保持一致；
-- parquet.page.size：页大小；
```
</details>

### 6. 企业调优
#### 6.1 Yarn 资源配置
<details>
<summary>Yarn 配置说明</summary>

```xml
<!-- 每个 NodeManager 的内存 -->
<property>
	<name>yarn.nodemanager.resource.memory-mb</name>
	<!-- 64M -->
	<value>65536</value>
</property>

<!-- 每个 NodeManager 的 CPU 核数 -->
<property>
	<name>yarn.nodemanager.resource.cpu-vcores</name>
	<value>16</value>
</property>

<!-- 单个 Container 能够使用的最大内存 -->
<property>
	<name>yarn.scheduler.maximum-allocation-mb</name>
	<value>16384</value>
</property>

<!-- 单个 Container 能够使用的最小内存 -->
<property>
	<name>yarn.scheduler.minimum-allocation-mb</name>
	<value> 512</value>
</property>
```
</details>

#### 6.2 MapReduce 资源配置
- MapReduce 资源配置主要包括 MapTask 的内存和 CPU 核数，以及 ReduceTask 的内存和 CPU 核数；

#### 6.3 测试用表
<<details>
<summary>测试用表</summary>

```sql
-- 订单表（2000 万条数据）
create table order_detail (
	id		string	comment '订单 id',
	user_id	string	comment	'用户 id',
	product_id	string	comment '商品 id',
	province_id	string	comment '省份 id',
	create_time	string	comment '下单时间',
	product_num	int		comment	'商品件数',
	total_amount	decimal(16, 2)	comment '下单金额'
)
partitioned by (dt string)
row format delimited fields terminated by '\t';

-- 加载文件
load data local inpath '/home/Documents/order_detail.txt' 
overwrite into table order_detail partition(dt='2020-06-14');

-- 支付表（600 万条数据）
create table payment_detail (
	id		string comment '支付 id',
	order_detail_id	string comment '订单明细 id',
	user_id			string comment '用户 id',
	payment_time	string comment '支付时间',
	total_amount	decimal(16, 2) comment '支付金额'
)
partitioned by (dt string)
row format delimited fields terminated by '\t';

-- 商品信息表（100 万条数据）
create table product_info (
	id		string comment '商品 id',
	product_name	string comment '商品名称',
	price	decimal(16, 2) comment '价格',
	category_id		string	comment '分类'
)
row format delimited fields terminated by '\t';

-- 省份信息表（34 条数据）
create table province_info (
	id		string	comment '省份 id',
	province_name	string comment '省份名称'
)
row format delimited fields terminated by '\t';
```
</details>

#### 6.4 Explain 查看执行计划
- Explain 呈现的执行计划，由一系列 Stage 组成，这一系列 Stage 具有依赖关系，每个 Stage 对应一个 MapReducejob，或者一个文件系统操作等；
- 若某个 Stage 对应的一个 MapReduce Job，其 Map 端和 Reduce 端的计算逻辑分别由 Map Operator Tree 和 Reduce Operator Tree 进行描述；每个 Operator Tree 由一系列的 Operator 组成，一个 Operator 代表在 Map 或 Reduce 阶段的一个单一的逻辑操作，例如：TableScan Operator，Select Operator，Join Operator 等；
- Explain 基本语法
- `Explain [formatted | extended | dependency] query-sql`
  - `formated`：将执行计划以 JSON 字符串的形式输出；
  - `extended`：输出执行计划中的额外信息，通常是读写的文件名信息；
  - `dependency`：输出执行计划读取的表及分区；

#### 6.5 分组聚合优化
- Hive 中未经优化的分组聚合，是通过一个 MapReduce Job 实现的。Map 端负责读取数据，并按照分组字段分区，通过 Shuffle，将数据发往 Reduce 端，各组数据在 Reduce 端完成最终的聚合运算；
- Hive 对分组聚合的优化主要围绕着减少 Shuffle 数据量进行，具体的做法是 map-side 聚合。所谓 map-side 聚合，就是在 map 端维护一个 hash table，利用其完成部分的聚合，然后将部分聚合的结果，按照分组字段分区，发送至 reduce 端，完成最终的聚合。map-side 聚合能有效减少 shuffle 的数据量，提高分组聚合运算的效率。

```sh
-- 启用 map-side 聚合，默认为 true
set hive.map.aggr=true;

-- 检测源表数据是否适合进行 map-side 聚合。检测的方法是：先对若干条数据进行 map-side 聚合，
-- 若聚合后的条数和聚合前的条数比值小于该值，则认为该表适合进行 map-side 聚合；否则不适合。
set hive.map.aggr.hash.min.reduction=0.5

-- 可以通过设置为 1，来强制走 map-side 分组聚合
set hive.map.aggr.hash.min.reduction=1

-- 用于检测源表是否适合 map-side 聚合的条数，默认 10 万条
set hive.groupby.mapaggr.checkinterval=100000;

-- map-side 聚合所用的 hash table，占用 map task 堆内存的最大比例，若超出该值，则会对 hash table 进行一次 flush
set hive.map.aggr.hash.force.flush.memory.threshold=0.9
```

#### 6.6 Join 优化
- Hive 拥有多种 Join 算法，包括 Common Join，Map Join， Bucket Join，Sort Merge Bucket Map Join 等；

#### Common Join
- Common Join 是 Hive 中最稳定的 join 算法；其通过一个 MapReduce Job 完成一个 join 操作。Map 端负责读取 join 操作所需表的数据，并按照关联字段进行分区，通过 shuffle，将其发送到 Reduce 端，相同 key 的数据在 Reduce 端完成最终的 Join 操作。
- 一个 SQL 语句中的**相邻**的且**关联字段相同**的多个 join 操作可以合并为一个 Common Join 任务；

![Common Join](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_common_join.png)

#### Map Join
- Map Join 算法可以通过两个只有 map 阶段的 Job 完成一个 join 操作。其适用场景为**大表 join 小表**。若某个 join 操作满足要求，则第一个 Job 会读取小表数据，将其制作为 hash table，并上传至 Hadoop 分布式缓存(本质上是上传至 HDFS)。第二个 Job 会先从分布式缓存中读取小表数据，并缓存在 Map Task 的内存中，然后扫描大表数据，这样在 map 端即可完成关联操作。

![Map Join](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_map_join.jpeg)

#### Bucket Map Join
- Bucket Map Join 是对 Map Join 算法的改进，其打破了 Map Join 只适用于大表 Join 小表的限制，可用于**大表 join 大表**的场景。
- Bucket Map Join 的核心思想是：若能保证参与 join 的表均为分桶表，且关联字段为分桶字段，且其中一张表的分桶数量是另外一张表分桶数量的整数倍，就能保证参与 join 的两张表的分桶之间具有明确的关联关系，所以就可以在两表的分桶间进行 Map Join 操作了。这样一来，第二个 Job 的 Map 端就无需再缓存小表的全表数据了，而只需缓存其所需的分桶即可。

![bucket map join](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_bucket_map_join.png)

#### Sort Merge Bucket Map Join
- Sort Merge Bucket Map Join(简称 SMB Map Join)基于 Bucket Map Join。SMB Map Join 要求，参与 join 的表均为分桶表，且需保证分桶内的数据是有序的，且**分桶字段、排序字段和关联字段**为相同字段，且其中一张表的分桶数量是另一张表分桶数量的整数倍。
- SMB Map Join 同 Bucket Join 一样，同样是利用两表各分桶之间的关联关系，在分桶之间进行 join 操作，不同的是，分桶之间的 join 操作的实现原理。Bucket Map Join，两个分桶之间的 join 实现原理为 **Hash Join 算法**；而 SMB Map Join，两个分桶之间的 join 实现原理为 **Sort Merge Join 算法**。
- Hash Join 和 Sort Merge Join 均为关系型数据库中常见的 Join 实现算法。Hash Join 的原理相对简单，就是对参与 join 的一张表构建 hash table，然后扫描另一张表，然后进行逐行匹配。Sort Merge Join 需要在两张按照关联字段排好序的表中进行，其原理如图：

![Sort Merge Bucket Map Join](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_bucket_sort_join.png)

#### 6.6.1 Map Join 优化
- Map Join 有两种触发优化的方式，一种是用户在 SQL 语句中增加 hint 提示（**已过时**），另外一种是 Hive 优化器根据参与 join 表的数据量大小，自动触发。
- **自动触发：**
  - Hive 在编译 SQL 语句阶段，起初所有的 join 操作均采用 Common Join 算法实现。之后在物理优化阶段，Hive 会根据每个 Common Join 任务所需表的大小判断该 Common Join 任务是否能够转换为 Map Join 任务，若满足要求，便将 Common Join 任务自动转换为 Map Join 任务。
  - 若有些 Common Join 任务所需的表大小，在 SQL 的编译阶段未知（例如对子查询进行 join 操作）。Hive 会在编译阶段生成一个条件任务(Conditional Task)，其下会包含一个计划列表，计划列表中包含转换后的 Map Join 任务以及原有的 Common Join 任务。最终具体采用哪个计划，是在运行时决定的。
- 查看表/分区的大小信息：`desc formatted table_name partition(partition_col='partition');`

![Conditional Task](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_conditional_task.jpeg)

```sh
# 是否自动触发 join 转换，默认为 true
hive.auto.convert.join=true

# 是否开启无条件转换，true：不开启；false：开启
# 若某大表候选之外的表大小均已知，且其大小总和小于 hive.auto.convert.join.noconditionaltask.size
# 推荐设置为 true
hive.auto.convert.join.noconditionaltask

# 一个 Common Join operator 转为 Map Join operator 的判断条件，若该 Common Join 相关的表中，
# 存在 n-1 张表的已知大小总和 <= 该值，则生成一个 Map Join 计划，此时，可能存在多种 n-1 张表的组合均满足
# 该条件，则 hive 会为每种满足条件的组合均生成一个 Map Join 计划，同时还会保留原有的 Common Join 计划作为
# 后备(back up)计划，实际运行时，优先执行 Map Join 计划，若不能执行成功，则启动 Common Join 后备计划；
hive.mapjoin.smalltable.filesize

# 无条件转 MapJoin 时的小表之和阈值，若一个 Common Join operator 相关的表中，存在 n-1 张表的大小总和
# <= 该值，此时 hive 便不会再为每种 n-1 张表的组合均生成 Map Join 计划，同时也不会保留 Common Join 作为
# 后备计划。而是只生成一个最优的 Map Join 计划。
hive.auto.convert.join.noconditionaltask.size
```

#### Bucket Map Join 优化
- Bucket Map Join 不支持自动转换，必须通过用户在 SQL 语句中增加 Hint 提示，并配置如下参数，方可使用。

<details>
<summary>Hint 提示配置</summary>

```sql
select /*+ mapjoin(ta) */
	ta.id,
	tb.id,
from table_a ta
join table_b tb on ta.id = tb.id;
```
</details>

<details>
<summary>Hive 参数配置</summary>

```sh
# 关闭 cbo 优化，cbo 会导致 hint 信息被忽略
set hive.cbo.enable=false;

# Map Join hint 默认会被忽略（因为已过时），需将如下参数设置为 false
set hive.ignore.mapjoin.hint=false;

# 启用 bucket map join 优化功能
set hive.optimize.bucketmapjoin=true;
```
</details>

#### Sorted Merge Bucket Map Join
- Sort Merge Bucket Map Join 有两种触发方式，包括 Hint 提示（已过时）和自动转换；

<details>
<summary>自动转换的相关参数</summary>

```sh
# 启动 Sort Merge Bucket Map Join 优化
set hive.optimize.bucketmapjoin.sortedmerge=true;

# 使用自动转换 SMB Join
set hive.auto.convert.sortmerge.join=true;
```
</details>

#### 6.6.2 数据倾斜
- 数据倾斜问题，通常是指参与计算的数据分布不均，即某个 key 或者某些 key 的数据量远超其他 key，导致在 shuffle 阶段，大量相同 key 的数据被发往同一个 Reduce，进而导致该 Reduce 所需的时间远超其他 Reduce，成为整个任务的瓶颈。
  - 分组聚合导致的数据倾斜；
  - Join 导致的数据倾斜；

#### 分组聚合导致的数据倾斜
- Hive 中未经优化的分组聚合，是通过一个 MapReduce Job 实现的。Map 端负责读取数据，并按照分组字段分区，通过 Shuffle，将数据发往 Reduce 端，各组数据在 Reduce 端完成最终的聚合运算；
- 如果 group by 分组字段的值分布不均，就可能导致大量相同的 key 进入同一 Reduce，从而导致数据倾斜问题。
- 由分组聚合导致的数据倾斜问题，有以下两种解决思路：
  - Map-Side 聚合
  - Skew-GroupBy 优化

#### Map-Side 聚合
- 开启 Map-Side 聚合后，数据会先在 Map 端完成部分聚合工作。这样一来，即便原始数据是倾斜的，经过 Map 端的初步聚合后，发往 Reduce 的数据也就不再倾斜了。

<details>
<summary>相关参数</summary>

```sh
-- 启用 map-side 聚合，默认为 true
set hive.map.aggr=true;

-- 检测源表数据是否适合进行 map-side 聚合。检测的方法是：先对若干条数据进行 map-side 聚合，
-- 若聚合后的条数和聚合前的条数比值小于该值，则认为该表适合进行 map-side 聚合；否则不适合。
set hive.map.aggr.hash.min.reduction=0.5

-- 可以通过设置为 1，来强制走 map-side 分组聚合
set hive.map.aggr.hash.min.reduction=1

-- 用于检测源表是否适合 map-side 聚合的条数，默认 10 万条
set hive.groupby.mapaggr.checkinterval=100000;

-- map-side 聚合所用的 hash table，占用 map task 堆内存的最大比例，若超出该值，则会对 hash table 进行一次 flush
set hive.map.aggr.hash.force.flush.memory.threshold=0.9
```
</details>

#### Skew-GroupBy 优化
- Skew-GroupBy 的原理是启动两个 MR 任务，第一个 MR 按照随机数分区，将数据分散发送到 Reduce，完成部分聚合，第二个 MR 按照分组字段分区，完成最终聚合。
- 启用分组聚合的参数：`set hive.groupby.skewindata=true;`；

#### Join 导致的数据倾斜
- 未经优化的 join 操作，默认是使用 Common join 算法，也就是通过一个 MapReduce Job 完成计算。Map 端负责读取 join 操作所需表的数据，并按照关联字段进行分区，通过 Shuffle，将其发送到 Reduce 端，相同 key 的数据在 Reduce 端完成最终的 join 操作。
- 如果关联字段的值分布不均，就可能导致大量相同的 key 进入同一 Reduce，从而导致数据倾斜问题。
- 由 join 导致的数据倾斜问题，有三种解决方案：
  - Map Join；
  - Skew Join；
  - 手动调整 SQL 语句；

#### Map Join
- 使用 map join 算法，join 操作仅在 map 端就能完成，没有 shuffle 操作，没有 reduce 阶段，自然不会产生 reduce 端的数据倾斜。该方案适用于**大表 join 小表**时发生数据倾斜的场景；

#### Skew Join
- Skew Join 的原理是，为倾斜的大 key 单独启动一个 Map Join 任务进行计算，其余 key 进行正常的 Common Join。
- 这种方案对参与 join 的源表大小没有要求，但是对两表中倾斜的 key 的数据量有要求，要求一张表中的倾斜 key 的数据量比较小。

![Skew Join](https://github.com/xianliu18/ARTS/tree/master/big_data/images/02hive/hive_skew_join.png)

<details>
<summary>Skew Join 参数配置</summary>

```sh
# 启用 skew join 优化
set hive.optimize.skewjoin=true;
# 触发 skew join 的阈值，若某个 key 的行数超过该参数值，则触发
set hive.skewjoin.key=100000;
```
</details>

#### 6.7 任务并行度
- Map 端并行度
- Reduce 端并行度

#### Map 端并行度
- Map 端并行度，是由输入文件的切片数决定的。一般情况下，Map 端的并行度无需手动调整；
- 查询的表中存在大量小文件：
  - 按照 Hadoop 默认的切片策略，一个小文件会单独启动一个 Map Task 负责计算。若查询的表中存在大量小文件，则会启动大量 Map Task，造成计算资源的浪费。这种情况下，可以使用 Hive 提供的`CombineHiveInputFormat`，多个小文件合并为一个切片，从而控制 Map Task 个数。
  - 参数配置：`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;`
- Map 端存在复杂的查询逻辑：
  - 若 SQL 语句中有正则替换、JSON 解析等复杂耗时的查询逻辑时，Map 端的计算会相对慢些。若想加快计算速度，在计算资源充足的情况下，可考虑增大 Map 端的并行度，令 Map Task 多一些，每个 Map Task 计算的数据少一些。
  - 参数配置：`set mapreduce.input.fileinputformat.split.maxsize=256000000`;

#### Reduce 端并行度
- Reduce 端的并行度，可由用户自定义，也可由 Hive 自行根据该 MR Job 输入的文件大小进行估算；

<details>
<summary>Reduce 并行度相关参数</summary>

```sh
# 指定 Reduce 端并行度，默认值为 -1，表示用户未指定
set mapreduce.job.reduces;

# Reduce 端并行度最大值
set hive.exec.reducers.max;

# 单个 Reduce Task 计算的数据量，用于估算 Reduce 并行度
set hive.exec.reducers.bytes.per.reducer;
```
</details>

- Reduce 端并行度的确定逻辑如下：
  - 若指定参数`mapreduce.job.reduces`的值为一个非负整数，则 Reduce 并行度为指定值；
  - 否则，Hive 自行估算 Reduce 并行度，估算逻辑如下：

```sh
# 假设 Job 输入的文件大小为 totalInputBytes
# 参数 hive.exec.reducers.bytes.per.reducer 的值为 bytesPerReducer
# 参数 hive.exec.reducers.max 的值为 maxReducers
# 则 Reduce 端的并行度为：

reduceNumber = min(ceil(totalInputBytes/bytesPerReducer), maxReducers)
```

#### 6.8 小文件合并
- Map 端输入文件合并
- Reduce 端输出文件合并

#### Map 端输入文件合并
- Map 端输入文件合并，是指将多个小文件划分到一个分片中，进而由一个 Map Task 去处理。目的是防止为单个小文件启动一个 Map Task，浪费计算资源。

```sh
-- 将多个小文件切片，合并为一个切片，进而由一个 Map 任务处理
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```

#### Reduce 端输出文件合并
- Reduce 端输出文件合并，是指将多个小文件合并成大文件。目的是减少 HDFS 小文件数量。其原理是根据计算任务输出文件的平均大小进行判断，若符合条件，则单独启动一个额外的任务进行合并。

<details>
<summary>Reduce 端输出文件合并</summary>

```sh
# 开启合并 map only 任务输出的小文件
set hive.merge.mapfiles=true;

# 开启合并 map reduce 任务输出的小文件
set hive.merge.mapredfiles=true;

# 合并后的文件大小
set hive.merge.size.per.task=256000000;

# 触发小文件合并任务的阈值，若计算任务输出的文件平均大小低于该值，则触发合并
set hive.merge.smallfiles.avgsize=16000000;
```
</details>

#### 6.9 其他优化

<details>
<summary>其他优化配置</summary>

```sh
# CBO 优化
# CBO：Cost Based Optimizer，即基于计算成本的优化
set hive.cbo.enable=true;

# 谓词下推优化，是指尽量将过滤操作前移，以减少后续计算步骤的数据量
# 是否开启谓词下推优化
set hive.optimize.ppd=true;

# 矢量化查询
# Hive 的矢量化查询优化，依赖于 CPU 的矢量化计算
# Hive 的矢量化查询，可以极大的提高一些典型查询场景（例如：scans, filters, aggregates, and joins)下的 CPU 使用率
set hive.vectorized.execution.enabled=true;

# Fetch 抓取
# Fetch 抓取是指，Hive 中对某些情况的查询可以不必使用 MapReduce 计算。例如：select * from customer_info;
# 在这种情况下， Hive 可以简单地读取 customer_info 对应的存储目录下的文件，然后输出查询结果到控制台；

# 是否在特定场景转换为 fetch 任务
# 设置为 none 表示不转换
# 设置为 minimal 表示支持 select *，分区字段过滤，Limit 等；
# 设置为 more 表示支持 select 任意字段，包括函数，过滤和 Limit 等
set hive.fetch.task.conversion=more;

# 本地模式
# 大多数的 Hadoop Job 是需要 Hadoop 提供的完整的可扩展性来处理大数据集的。不过，有时 Hive 的输入数据量是非常小的；
# 在这种情况下，为查询触发执行任务消耗的时间可能会比实际 job 的执行时间要多的多。对于这种情况， Hive 可以通过本地模式
# 在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。

# 是否自动转换为本地模式
set hive.exec.mode.local.auto=true;

# 设置 Local MapReduce 的最大输入数据量，当输入数据量小于这个值时，采用 Local MapReduce
# 默认为 128M, 134217728
set hive.exec.mode.local.auto.iinputbytes.max=50000000;

# 设置 Local MapReduce 的最大输入文件个数，当输入文件个数小于这个值时，采用 Local MapReduce
# 默认为 4
set hive.exec.mode.local.auto.input.files.max=10;

# 并行执行
# Hive 会将一个 SQL 语句转化成一个或多个 Stage，每个 Stage 对应一个 MR Job。默认情况下，Hive 同时
# 只会执行一个 Stage。但是 SQL 语句可能会包含多个 Stage，但多个 Stage 可能并非完全相互依赖，也就是说
# 有些 Stage 是可以并行执行的。

# 启用并行执行优化
set hive.exec.parallel=true;

# 同一个 SQL 允许最大并行度，默认为 8
set hive.exec.parallel.thread.number=8;

# 严格模式
# 分区表不适用分区过滤
# 对于分区表，除非 where 语句中含有分区字段过滤条件来限制范围，否则不允许执行；
set hive.strict.checks.no.partition.filter=true;

# 对于使用了 order by 查询的语句，要求必须使用 limit
set hive.strict.checks.orderby.no.limit=true;

# 会限制笛卡尔积查询
set hive.strict.checks.cartesian.product=true;
```
</details>
