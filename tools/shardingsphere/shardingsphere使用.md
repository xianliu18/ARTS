### ShardingSphere-JDBC 深度解读

### 1. 什么是分库分表
- 分表：将一个表中的数据按照某种规则分拆到多张表中，降低锁粒度以及索引树，提升数据查询效率；
- 分库：将一个数据库中的数据按照某种规则分拆到多个数据库中，以缓解单服务器的压力(CPU、内存、磁盘、IO)。

### 2. 分库分表带来的问题
1，事务性问题
	- 分库可能导致执行一次事务所需的数据分布在不同服务器上，数据库层面无法实现事务性操作。
2，主键(自增 ID)唯一性问题
	- 主键采用全局统一 ID 生成机制：如 UUID、雪花算法、数据库号段等方式。
3，跨库多表 join 问题
4，跨库聚合查询问题

### 3. 常见水平拆分手段
#### 3.1 range 分库分表
- 例如：交易流水表按照天级别分表，适合冷数据处理；

#### 3.2 hash 分库分表
1，独立 hash
- 通过主键计算 hash 值，然后 hash 值分别对库数和表数进行取余操作获取到库索引和表索引。
- 例如，电商订单表，按照用户 ID 分配到 10 库 100 表中；
- 存在`数据偏斜问题`；
```java
// dbCount 库数量
Integer dbCount = 10;
// tbCount 表数量
Integer tbCount = 100;

// 根据用户 ID 获取分库分表索引
void getTableIdx(String userId) {
	Integer hash = userId.hashCode();
	Integer dbIdx = hash % dbCount;
	Integer tbIdx = hash % tbCount;
	return;
}
```

2，统一 hash
```java
// dbCount 库数量
Integer dbCount = 10;
// tbCount 表数量
Integer tbCount = 100;

// 根据用户 ID 获取分库分表索引
void getTableIdx(String userId) {
	int size = dbCount * tbCount;
	// 扰动函数
	int idx = (size - 1) & (userId.hashCode() ^ (userId.hashCode() >>> 16));
	// 库索引
	Integer dbIdx =  idx / tbCount;
	// 表索引
	Integer tbIdx = idx % tbCount;
	return;
}
```

<br/>

**参考资料：**
- [一文读懂数据库优化之分库分表](https://blog.csdn.net/u013291818/article/details/124191907)