[TOC]









# Spark 学习笔记





 



## 第1章 Spark 概述



**Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。**



### Spark  and Hadoop 

Hadoop 是一个大的生态，主要有底层的分布式文件系统HDFS、数据引擎MapReduce、资源调度器Yarn 三个核心组件以及一些列生态组成。

而 Spark 出现的时间相对较晚，主要的作用是用于数据计算。Spark 的核心模块包括Spark Core提供核心数据计算功能，Spark SQL 是操作结构化数据的组件，Spark Streaming是针对数据进行流式计算的组件。



### Spark or Hadoop 



* Hadoop 的MR数据处理框架主要用于一次性的数据计算（在处理数据式，会从存储设备中读取数据，进行逻辑操作，然后将处理的结果重新存储到介质中）。

* Spark 和Hadoop 的根本差异是多个作业之间的数据通信问题：Spark 多个作业之间数据通信是基于内存，而Hadoop是基于磁盘。
* Spark 的核心技术是RDD 弹性分布式数据集，提供了比 MapReduce 丰富的模型，可以快速在内存中对数据集 进行多次迭代
* Spark Task的启动时间快。 Spark 采用fork线程的方式，而Hadoop 采用创建新的进程的方式。
* Spark 只有在shuffle 的时候将数据写入磁盘，而Hadoop 中多个MR 作业之间的数据交互都要依赖于磁盘交互





### Spark 核心模块

* 核心Spark Core：其他组件都是基于Core扩展的
* Spark SQL：用来操作结构化数据的组件
* Spark Streaming：针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API
* Spark MLlib：机器学习算法库
* Spark GraphX：面向图计算提供的框架和算法库



## 第2章 快速上手



1. 创建Maven 项目
2. 增加Scala 插件
3. 增加Spark 框架的依赖关系





## 第3章 Spark 运行环境

local本地模式



standalone 独立部署模式



yarn 模式





## 第4章 Spark运行架构

Spark框架的核心是一个计算引擎，整体来说，它采用了标准master-slaver 的结构。

### 核心组件

两大核心组件：

* Driver : Spark 驱动器节点，用于执行Spark 任务中的main方法，负责实际代码的执行工作。

  * 将用户程序传化为job作业

  * 在Executor之间调度task任务

  * 跟踪Executor的执行情况

  * 通过UI展示查询运行情况

* Executor: 执行器是集群中工作节点（Worker）中的一个JVM进程，负责在Spark作业中运行具体任务Task ,任务之间相互独立。Spark应用启动时， Executor 节点被同时启动，并且始终伴随着整个Spark应用的声明周期而存在。

  * 负责运行组成Spark 应用的任务，并将结果返回给驱动器进程

  * 通过自身的块管理器（Block Manager）为用户程序中要求缓存的RDD提供内存式存储。





核心组件：Master & Worker 

在Spark集群的独立部署环境中，Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于Yarn环境中的ResourceMananger, 而Worker也是一个进程，一个Worker 运行在集群中的一台服务器上，由Master 分配资源对数据进行并行处理，类似于Yarn环境中的NodeManager



核心组件：ApplicationMaster

Hadoop 用户向Yarn 集群提交应用程序时，提交程序中应该包含ApplicationMaster, 用于向资源调度器申请执行任务的资源容器Container，运行用户自己的程序任务job, 监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

Resourcemanager 和 Driver 之间的解耦就是靠  ApplicationMaster



### 核心概念



1. Executor与Core : 执行器就是运行在工作节点Worker中的一个JVM进程。在提交应用的时候，可以提供参数指定计算节点的个数，以及对应资源（内存大小，cpu 核数）

   | 名称              | 说明                            |
   | ----------------- | ------------------------------- |
   | --num-executors   | executor数量                    |
   | --executor-memory | 每个executor 的内存大小         |
   | --executor-cores  | 每个executor的虚拟cpu core 数量 |

2. 并行度：整个集群并行执行任务的数量，称之为并行度。

3. 有向无环图DAG: 是由点和线组成的拓扑图形，该图形具有方向，不会闭环。Spark程序会将整个程序计算的执行过程用图形表示出来，更便于理解，可以用于表示程序的拓扑结构。

   > 大数据计算引擎框架中，我们根据使用方式的不同一般会分为四类。其中第一类就是Hadoop 默认的MapReduce, 它将计算分为两个阶段，Map 和Reduce阶段。对于上层应用来说，就不得不想方设法去拆分算法，甚至不得不在上层应用实现多个Job 的串联，以完成一个完整的算法。由于这样的问题，催生了支持DAG框架的产生。因此支持DAG的框架被划分为第二代计算引擎。如TEZ, Oozie。 接下来就是以Spark为代表的第三代计算引擎，第三代计算引擎的特点主要是Job内部支持的DAG支持（不跨越Job）,以及实时计算。



### 提交流程

我们所写的应用程序通过Spark 客户端提交给Spark 运行环境执行计算的流程。



Spark 应用程序提交到 Yarn 环境中执行的时候，一般会有两种部署执行的方式：Client 和 Cluster。两种模式主要区别在于：Driver 程序的运行节点位置。

#### Yarn Client 模式

Client模式将用于监控和调度的Driver模块在客户端执行，而不是在Yarn中，所以一般用于测试。

* Driver在任务提交的本地机器上运行
* Driver启动后会和ResourceManager 通讯申请启动 ApplicationMaster
* ResourceManager 分配container之前，会在合适的NodeManager上启动ApplicationMaster, 负责向ResourceManager申请Executor 内存资源
* ResourceManager 接到ApplicationMaster 的资源申请后，会分配container，然后ApplicationMaster 在资源分配指定的NodeManager 上启动Executor进程
* Executor 进程启动后会向Driver反向注册，Executor 全部注册完成后Driver开始执行main函数
* 之后执行到Action算子时， 触发一个Job,并根据宽依赖开始划分stage,每个stage生成与之对应的TaskSet,之后将task分发到各个Executor上执行 



Yarn Cluster 模式

Cluster 模式将用于监控和调度的Driver 模块启动在Yarn集群资源中执行。一般用于实际生产环境。

* 在Yarn Cluster 模式下，任务提交后会和ResourceManager 通讯申请 ApplicationMaster
* ResourceManager 分配container之前，会在合适的NodeManager上启动ApplicationMaster,此时的ApplicationMaster 就是Driver
* Driver启动后向ResourcesManager 申请Executor内存， ResourceManager接到ApplicationMaster的资源申请后会分配container,然后在合适的NodeManager上启动Executor进程
* Executor 进程启动后会向Driver反向注册，Executor 全部注册完成后Driver开始执行main函数
* 之后执行到Action算子时， 触发一个Job,并根据宽依赖开始划分stage,每个stage生成与之对应的TaskSet,之后将task分发到各个Executor上执行 



## 第5章 Spark 核心编程

Spark 计算框架为了能够进行*高并发*和*高吞吐*的数据处理，封装了三大数据结构，用于处理不同的应用场景

* RDD: 弹性分布式数据集
* 累加器：分布式共享只写变量
* 广播变量：分布式共享只读变量



### RDD

RDD—— Resillient Distributed Dataset 叫做弹性分布式数据集，是Spark 中最基本的额数据处理模型。

代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

* 弹性
  * 存储的弹性：内存与磁盘的自动切换
  * 容错的弹性：数据丢失可以自动恢复
  * 计算的弹性：计算出错重试机制
  * 分片的弹性：可以根据需要重新分片
* 分布式：数据存储在大数据集群不同的节点上
* 数据集：RDD封装了计算逻辑，并不保存数据
* 数据抽象：RDD是一个抽象类，需要子类具体实现
* 不可变：RDD封装了计算逻辑，是不可以改变的，想要改变只能产生新的RDD, 在新的RDD 里面封装计算逻辑
* 可分区、并行计算



### 核心属性

* 分区列表：RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性。一个RDD有多个分区，每个分区内部包含了部分RDD数据，一个分区对应一个task线程
* 分区计算函数：Spark 在计算时，是使用分区函数对每一个分区进行计算。每个RDD都会实现compute函数以达到这个目的。
* RDD之间的依赖关系：RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系。一个RDD会依赖于其他多个RDD
* 分区器（可选）：当数据为KV类型数据时，可以通过设定分区器，自定义数据的分区。默认Hash分区/范围分区。
* 首选位置（可选）：计算任务的位置有限为存储每个Patition的位置（可自定义）。计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算



### 执行原理





### 基础编程-RDD

#### RDD的创建方式四种：

1. 从集合中创建RDD `val rdd1 = sparkContext.parallelize(List(1,2,3,4) )`
2. 从外部存储文件创建RDD `val fileRDD = sparkContext.textFile("input")`
3. 从其他RDD创建`val wordRDD = lineRDD.flatMap(_.split(" "))`
4. 直接new 创建，RDD



#### RDD 并行度与分区

默认情况下，Spark 可以将一个作业切分成多个任务后，发送个Executor节点并行计算。而能够并行计算的任务数量我们称之为并行度。可以在构建RDD时指定。

```scala
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val dataRDD: RDD[Int] =sparkContext.makeRDD(List(1,2,3,4),4)
val fileRDD: RDD[String] =sparkContext.textFile("input",2)
fileRDD.collect().foreach(println)
sparkContext.stop()
```





#### RDD 转换算子 transformation

| **转换**                                            | **含义**                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| **map(func)**                                       | 返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成 |
| **mapPartitions(func)**                             | 类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator[T] => Iterator[U] |
| **mapPartitionsWithIndex(func)**                    | 类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是(Int, Interator[T]) => Iterator[U] |
| **filter(func)**                                    | 返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成 |
| **flatMap(func)**                                   | 类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素） |
| **union(otherDataset)**                             | 对源RDD和参数RDD求并集后返回一个新的RDD                      |
| **intersection(otherDataset)**                      | 对源RDD和参数RDD求交集后返回一个新的RDD                      |
| **subtract(otherDataset)**                          | 对源RDD和参数RDD求差集后返回一个新的RDD                      |
| **zip(otherDataset)**                               | 将两个RDD中的元素，以键值对的形式进行合并                    |
| **distinct([numTasks]))**                           | 对源RDD进行去重后返回一个新的RDD                             |
| **groupByKey([numTasks])**                          | 在一个(K,V)的RDD上调用，返回一个(K, Iterator[V])的RDD        |
| **reduceByKey(func, [numTasks])**                   | 在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，与groupByKey类似，reduce任务的个数可以通过第二个可选的参数来设置 |
| **aggreateByKey(zeroValue)(seqOp,combOp)**          | 聚合操作，第一个参数是初始值，第二个参数中的第一个是分区内计算函数，第二个是分区间函数 |
| **foldByKey(zeroValue)(func)**                      | 当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey |
| **combineByKey(create,mergeVal,mergeCom)**          | 三个参数含义：第一个数据处理，分区内数据处理，分区间数据处理 |
| **sortByKey([ascending], [numTasks])**              | 在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD |
| **sortBy(func,[ascending], [numTasks])**            | 与sortByKey类似，但是更灵活                                  |
| **join(otherDataset, [numTasks])**                  | 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD |
| **cogroup(otherDataset, [numTasks])**               | connect group : 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable,Iterable))类型的RDD |
| **coalesce(numPartitions，shuffle=false)**          | 减少 RDD 的分区数到指定值。分区合并。tips:可能会导致数据倾斜。 |
| **repartition(numPartitions)**                      | 重新给 RDD 分区。就是coalesce(numPartitions, shuffle = true) |
| **repartitionAndSortWithinPartitions(partitioner)** | 重新给 RDD 分区，并且每个分区内以记录的 key 排序             |





#### RDD action算子

| **动作**                       | **含义**                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| **reduce(func)**               | reduce将RDD中元素前两个传给输入函数，产生一个新的return值，新产生的return值与RDD中下一个元素（第三个元素）组成两个元素，再被传给输入函数，直到最后只有一个值为止。 |
| **collect()**                  | 在驱动程序中，以数组的形式返回数据集的所有元素               |
| **count()**                    | 返回RDD的元素个数                                            |
| **first()**                    | 返回RDD的第一个元素（类似于take(1)）                         |
| **take(n)**                    | 返回一个由数据集的前n个元素组成的数组                        |
| **takeOrdered(n, [ordering])** | 返回自然顺序或者自定义顺序的前 n 个元素                      |
| **saveAsTextFile(path)**       | 将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本 |
| **saveAsSequenceFile(path)**   | 将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。 |
| **saveAsObjectFile(path)**     | 将数据集的元素，以 Java 序列化的方式保存到指定的目录下       |
| **countByKey()**               | 针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。 |
| **foreach(func)**              | 在数据集的每一个元素上，运行函数func                         |
| **foreachPartition(func)**     | 在数据集的每一个分区上，运行函数func                         |





#### RDD 序列化



1. 闭包检查：从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor 端执行。

   > 那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就 形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor 端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列 化，这个操作我们称之为闭包检测。Scala2.12 版本后闭包编译方式发生了改

2. 序列化方法

   1. 使用Java的序列化方法，实现Serializable

   2.  Kryo 序列化框架

      ```scala
      val conf: SparkConf = new SparkConf()
           .setAppName("SerDemo")
           .setMaster("local[*]")
           // 替换默认的序列化机制
           .set("spark.serializer", 
          "org.apache.spark.serializer.KryoSerializer")
           // 注册需要使用 kryo 序列化的自定义类
           .registerKryoClasses(Array(classOf[Searcher]))
      ```

      

#### RDD依赖关系



RDD血缘关系：

RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage （血统）记录下来，以便恢复丢失的分区。RDD 的 Lineage 会记录 RDD 的元数据信息和转 换行为，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的 数据分区。

RDD依赖关系：

这里所谓的依赖关系，其实就是两个相邻 RDD 之间的关系

RDD 窄依赖：

窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游）RDD 的一个 Partition 使用， 窄依赖我们形象的比喻为独生子女。

RDD 宽依赖

宽依赖表示同一个父（上游）RDD 的 Partition 被多个子（下游）RDD 的 Partition 依赖，会 引起 Shuffle，总结：宽依赖我们形象的比喻为多生。

RDD阶段划分

DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向， 不会闭环。例如，DAG 记录了 RDD 的转换过程和任务的阶段。

RDD 任务划分

RDD 任务切分中间分为：Application、Job、Stage 和 Task 

* Application：初始化一个 SparkContext 即生成一个 Application； 
* Job：一个 Action 算子就会生成一个 Job； 
* Stage：Stage 等于宽依赖(ShuffleDependency)的个数加 1； 
* Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。



#### RDD 持久化 



**RDD Cache 缓存**

将中间结果临时缓存。如果作业执行完毕，临时文件会被清除。

RDD通过Cache或者Persist方法将前面计算结果缓存，默认情况下会把数据缓存再JVM的堆内存中。但是并不是这两个方法被调用时立即缓存的，而是触发后面的action算子时，该RDD将会被缓存再计算节点的内存中，并供后面重用。

* cache 底层还是调用的persist方法，只是存储级别不同

* cache: 将数据临时存储在内存中进行数据重用
* persist: 将数据临时存储在磁盘文件中，涉及磁盘IO,性能较低，数据安全

```scala
/**
* Persist this RDD with the default storage level (`MEMORY_ONLY`).
*/
def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)

/**
* Persist this RDD with the default storage level (`MEMORY_ONLY`).
*/
def cache(): this.type = persist()
```

| 级别                | 使用的空间 | CPU时间 | 是否再内存中 | 是否再磁盘上 | 备注                                 |
| ------------------- | ---------- | ------- | ------------ | ------------ | ------------------------------------ |
| MEMORY_ONLY         | 高         | 低      | 是           | 否           |                                      |
| MEMORY_ONLY_SER     | 低         | 高      | 是           | 否           |                                      |
| MEMORY_AND_DISK     | 高         | 中等    | 部分         | 部分         | 如果数据在内存中放不下，则溢写到磁盘 |
| MEMORY_AND_DISK_SER | 低         | 高      | 部分         | 部分         | 同上，存放的是序列化后的数据         |
| DISK_ONLY           | 低         | 高      | 否           | 是           |                                      |



**RDD CheckPoint 检查点**

将RDD中间结果长久的写入磁盘。涉及到磁盘IO，性能较低，数据安全。

为保证数据安全，会执行独立任务。

为了提高效率，会和cache联合使用

由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。

```scala
// 设置检查点路径
sc.setCheckpointDir("./checkpoint1")
// 创建一个 RDD，读取指定位置文件:hello atguigu atguigu
val lineRdd: RDD[String] = sc.textFile("input/1.txt")
// 业务逻辑
val wordRdd: RDD[String] = lineRdd.flatMap(line => line.split(" "))
val wordToOneRdd: RDD[(String, Long)] = wordRdd.map {
 word => {
 (word, System.currentTimeMillis())
 }
}
// 增加缓存,避免再重新跑一个 job 做 checkpoint
wordToOneRdd.cache()
// 数据检查点：针对 wordToOneRdd 做检查点计算
wordToOneRdd.checkpoint()
// 触发执行逻辑
wordToOneRdd.collect().foreach(println)
```



**Cache和Checkpoint 的区别**

1. Cache 缓存只是将数据临时保存，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。
2. Cache缓存的数据通常存储在磁盘，内存等地方，可靠性低。Checkpoint 的数据通常存储在HDFS等容错、高可用的文件系统，可靠性高。
3. 可以将checkpoint() 的RDD和使用cache()缓存，这样checkpoint 的job只需要从Cache缓存中读取数据即可，否则需要从头计算一次RDD。





#### RDD 分区器

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认 分区。分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle 后进入哪个分 区，进而决定了 Reduce 的个数。

* 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None 
* 每个 RDD 的分区 ID 范围：0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。



#### RDD 文件读取与保存

Spark 的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。

*  文件格式分为：text 文件、csv 文件、sequence 文件以及 Object 文件；
*  文件系统分为：本地文件系统、HDFS、HBASE 以及数据库。



```scala
// text读取输入文件
val inputRDD: RDD[String] = sc.textFile("input/1.txt")
// text保存数据
inputRDD.saveAsTextFile("output")

// 保存数据为 SequenceFile
dataRDD.saveAsSequenceFile("output")
// 读取 SequenceFile 文件
sc.sequenceFile[Int,Int]("output").collect().foreach(println)

// object 保存数据
dataRDD.saveAsObjectFile("output")
//object 读取数据
sc.objectFile[Int]("output").collect().foreach(println)
```





### 累加器

#### 实现原理 

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在 Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后， 传回 Driver 端进行 merge。





### 广播变量

#### 实现原理 

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个 或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表， 广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务 分别发送。





## Spark SQL

### 是什么？

Spark SQL 是 Spark 用于结构化数据(structured data)处理的 Spark 模块。

### Hive and SparkSQL





### SparkSQL 特点

1. 易整合：无缝整合了SQL查询和Spark 编程。
2. 统一的数据访问：使用相同的方式连接不同的数据源。
3. 兼容Hive：在已有仓库上直接运行SQL或者HiveQL
4. 标准数据连接:基于JDBC/ODBC连接



### DataFrame 是什么

在 Spark 中，DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库中 的二维表格。DataFrame 与 RDD 的主要区别在于，前者带有 schema 元信息，即 DataFrame 所表示的二维表数据集的每一列都带有名称和类型。这使得 Spark SQL 得以洞察更多的结构 信息，从而对藏于 DataFrame 背后的数据源以及作用于 DataFrame 之上的变换进行了针对性 的优化，最终达到大幅提升运行时效率的目标。反观 RDD，由于无从得知所存数据元素的 具体内部结构，Spark Core 只能在 stage 层面进行简单、通用的流水线优化。

DataFrame 也是懒执行的，但性能上比 RDD 要高，主要原因：优化的执行计划，即查询计 划通过 Spark catalyst optimiser 进行优化。



### DataSet 是什么

DataSet 分布式数据集合。它提供了 RDD 的优势（强类型，使用强大的 lambda 函数的能力）以及 Spark  SQL 优化执行引擎的优点。DataSet 也可以使用功能性的转换（操作 map，flatMap，filter等等)



* DataSet 是 DataFrame API 的一个扩展，是 SparkSQL 最新的数据抽象
* 用户友好的 API 风格，既具有类型安全检查也具有 DataFrame 的查询优化特性；
* 用样例类来对 DataSet 中定义数据的结构信息，样例类中每个属性的名称直接映射到 DataSet 中的字段名称；
* DataSet 是强类型的。比如可以有 DataSet[Car]，DataSet[Person]。
* DataFrame 是 DataSet 的特列，DataFrame=DataSet[Row] ，所以可以通过 as 方法将 DataFrame 转换为 DataSet。Row 是一个类型，跟 Car、Person 这些的类型一样，所有的 表结构信息都用 Row 来表示。获取数据时需要指定顺序



SparkSession 是 Spark 最新的 SQL 查询起始点，实质上是 SQLContext 和 HiveContext 的组合，所以在 SQLContex 和 HiveContext 上可用的 API 在 SparkSession 上同样是可以使用 的。SparkSession 内部封装了 SparkContext，所以计算实际上是由 sparkContext 完成的。当 我们使用 spark-shell 的时候, spark 框架会自动的创建一个名称叫做 spark 的 SparkSession 对 象, 就像我们以前可以自动获取到一个 sc 来表示 SparkContext 对象









## Spark Streaming

### 是什么？

Spark 流使得构建可扩展的容错流应用程序变得更加容易。 

Spark Streaming 用于流式数据的处理。Spark Streaming 支持的数据输入源很多，例如：Kafka、 Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语 如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。



### Spark Streaming 的特点

* 易用
* 容错
* 易整合到Spark 体系





SparkStreaming 准实时（秒/分），微批次的数据处理框架