<!--
### Spark 入门

### 1，概述
- Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎；
- Spark 和 Hadoop 的根本差异是多个作业之间的数据通信问题：
  - Spark 多个作业之间数据通信是基于内存；
  - Hadoop 是基于磁盘；

#### 入门程序

<details>
<summary>入门程序</summary>

```java
def main(args: Array[String]): Unit = {
	// 1，建立和 Spark 框架连接
	val sparkConf = new SparkConf().setMaster("local").setAppName("WordCount")
	val context = new SparkContext(sparkConf);

	// 2，执行业务操作
	// 2.1 读取文件，获取一行一行的数据
	val lines = context.textFile("src/main/resources/input/demo.txt")

    // 2.2 将一行数据进行拆分，形成一个个单词
    val words = lines.flatMap(_.split(" "))

    // 2.3 根据单词进行分组，便于统计
    val wordToOne:RDD[(String, Int)] = words.map(word => (word, 1))

    // 2.4 对分组后的数据进行转换
    val wordGroup: RDD[(String, Iterable[(String, Int)])] = wordToOne.groupBy(t => t._1)

    // 2.5 将转换结果采集到控制台打印出来
    val arrayRes = wordGroup.map {
      case(word, list) => {
        list.reduce(
          (t1, t2) => {
            (t1._1, t1._2 + t2._2)
          }
        )
      }
    }
	
	//  Spark 可以将分组和聚合使用一个方法实现
	//  reduceByKey: 相同 key 的数据，可以对 value 进行 reduce 聚合  
	// val wordToCount = wordToOne.reduceByKey((x, y) => {x + y})
    // val wordToCount = wordToOne.reduceByKey(_ + _)

	arrayRes.foreach(println)

	// 3，关闭连接
	context.stop();
}
```
</details>

// TODO:
- P011