[TOC]



# 《hive 编程指南》阅读笔记



## 第1章 基础知识

### 1.1 Hadoop 和 MapReduce综述



hadoop？ 

hdfs：一种分布式文件系统

hive：基于hdfs的一种数据管理工具

#### MapReduce

**MapReduce**是一种计算模型， 该模型可以将大型数据处理任务拆分成很多个单个的可以在服务器集群中并行执行的任务。这些任务的计算结果可以合并在一起来计算最终结果。

**map**操作将集合中的元素从一种形式转换成另一种形式

**reduce**的过程会将值的集合转换成一个值，或者转换成另一个集合。



### 1.2 Hadoop 生态系统中的Hive



## 第 2 章 基础操作





## 第 3 章 数据类型和文件格式



### 3.1 基本数据类型

| 数据类型  | 描述             |
| --------- | ---------------- |
| TINYING   | 1byte 有符号整数 |
| SMALINT   | 2byte 有符号整数 |
| INT       | 4byte 有符号整数 |
| BIGINT    | 8byte 有符号整数 |
| BOOLEAN   | 布尔类型         |
| FLOAT     | 单精度浮点数     |
| DOUBLE    | 双精度浮点数     |
| STRING    | 字符串           |
| TIMESTAMP | 时间类型         |
| BINARY    | 字节数组         |





### 3.2 集合数据类型

| 数据类型 | 描述                                               |
| -------- | -------------------------------------------------- |
| STRUCT   | 和C语言中的struct类似，都可以通过“.”访问元素内容。 |
| MAP      | 键值对                                             |
| ARRAY    | 数组                                               |



### 3.3 文本文件数据编码

Hive中默认的记录和字段分割符

| 分隔符                    | 描述                               |
| ------------------------- | ---------------------------------- |
| \n                        | 换行符                             |
| ^A(Ctrl +  A)八进制`\001` | 用于分割字段（列）。               |
| ^B(Ctrl +  B)八进制`\002` | 用于分割array\struct\map中的元素。 |
| ^C(Ctrl +  C)八进制`\003` | 用于表示map中键值对之间的分隔符。  |



### 3.4 读时模式

当用户想传统数据库中写入数据时， 数据库在写入的时候就会进行检查。

但是hive只会在查询时进行检查。 缺少列会返回null， 如果类型不一致且无法转化也会返回null。



## 第 4 章 HiveQL：数据定义

Hive不支持行级别的插入，更新和删除操作。 

Hive也不支持事务。



###  4.1 Hive中的数据库

Hive中数据库的概念本质上仅仅是表的一个目录或者命名空间。







## 第 5 章 HiveQL：数据操作

### 5.1 向管理表中装载数据

如果分区不存在，这个命令会先创建分区目录，然后再复制数据到该目录下。

```sql
-- 从本地路径加载数据目录到表的指定分区
load data local inpath '${env:HOME/data_file_path}'
overwrite into table table_name
partition (country = 'US', state = 'CA');
```

注：

1. `local`关键字是从本地文件系统copy文件到hdfs中，

​		如果没有`local`关键字，那就是从hdfs中移动文件。（分布式文件中不需要多份数据）

2. `overwrite` 关键字保证目标文件夹中之前存在的数据会被删除

   不用该关键字，会保留之前的文件，并且重命名之前的文件“文件名_序列号”

3. 如果目标表时分区表，那么需要使用`partition`子句，而且还必须为分区指定value
4. 对于`inpath`子句中使用的文件路径还有一个限制，即该路径下不可以包含任何文件夹
5. Hive不会严重用户load的数据是否和表的模型是否匹配。但是，Hive会验证文件格式是否和表结构定义一致。

### 5.2 通过查询语句向表中插入数据

```sql
insert overwrite table table_name partition (dt = '2022-04-16')
select * from source_table where dt = '2022-04-16';
```

ps：`overwrite` 关键字，覆盖之前分区中的内容（`into` 关键字，会追加数据）

#### 动态分区插入

```sql
insert overwrite table table_name partition (year, month, day)
select ...., year, month, day from source_table;
```

Hive会根据select语句中的最后几列来确定分区字段 dt的值。

混合使用动态和静态分区(**静态分区键必须出现在动态分区键之前**)：

```sql
insert overwrite table table_name partition (year = '2022', month, day)
select ...., year, month, day from source_table;
```

> ps: 动态分区默认情况下是没开启的
>
> 猜测容易出错，如果查询结果错误，会导致一次创建过多分区，导致分区文件数量暴增

### * 5.3 单个查询语句中创建并加载数据

```sql
create table table_name
as 
select os, os_version, os_name from source_table where dt = '2022-04-16';
```

一条语句完成创建表并将查询结果载入这个表的操作。

> 这个功能常用于从一个大宽表中选取部分数据集
>
> 这个功能不能用于外部表。这个地方并没有进行数据的 '装载'，只是将元数据中指定一个指向数据的路径。？？？？？？？

###  5.4 导出数据

直接导出数据：

```shell
hadoop fs -cp source_path target_path
```

也可以查询数据到指定文件

``` sql
insert overwrite local directory '/tmp/target_table_name'
select name, age, something from table_name where dt = '2022-04-16';
```





## 第 6 章 HiveQL：查询



#### 6.1.9 什么情况下Hive可以避免进行MapReduce

1. 本地模式时
2. where语句中过滤条件只是分区字段



### 6.2 Where 语句



#### 6.2.1 谓词操作符

| 操作符                  | 支持的数据类型 |
| ----------------------- | -------------- |
| A=B                     | 基本数据类型   |
| A<=>B                   | 基本数据类型   |
| A<>B, A!=B              | 基本数据类型   |
| A<B,A<=B                | 基本数据类型   |
| A>B,A>=B                | 基本数据类型   |
| A [NOT] BETWEEN B AND C | 基本数据类型   |
| A IS [NOT] NULL         | 所有数据类型   |
| A [NOT] LIKE B          | String         |
| A RLIKE B, A REGEXP B   | String         |



#### 6.2.2 关于浮点数比较

因为hive是Java实现的， 无法精确的表示一个double/float类型, 所以在比较的时候需要注意。

解决方法一： ？ 不知道书在说啥

方法二： 比较时显示的转化。cast（ num as float）

方法三： 直接不用浮点数

#### 6.2.3 LIKE 和 RLIKE



### 6.3 GROUP BY 语句



HAVING 语句



### 6.4 JOIN 语句

Hive 支持常见的sql join语句，但是，**只支持等值连接**。

#### 6.4.1 inner join

内连接

多个表join时，每次join都会 启动一个 MapReduce 任务

#### 6.4.2 join 优化

1.  当对多个表join时，如果每个ON子句都使用相同的连接键时，只会产生一个MapReduce job
2. join时，将小的表放在前，大的表放在后。因为，join的时候，在对每行记录进行连接操作时，会尝试将其他表缓存起来，然后扫描最后那个表进行计算。

#### 6.4.3 left outer join

左外连

#### 6.4.4 outer join







## 第 7 章 HiveQL：视图

视图允许保存一个查询，并像对待表一样对这个查询进行操作。只是一个逻辑结构，并不会像表一样存储数据。

### 7.1 使用视图来降低查询复制度

当我们查询一个视图时， 这个视图所定义的查询语句会和我们我们的查询语句组合在一起，然后再提交hive查询。 从逻辑上，可以想象成，hive先执行这个视图，然后使用这个结果进行后续查询。（这么说，和superset中的dataset 不是一样？ 我们的chart是对dataset进行二次处理嘛）

### 7.2  使用视图来限制基于条件过滤的数据

视图已经筛选过一次，所以后面的查询只能基于视图查询来的结果进行筛选。



### 7.3 动态分区中的视图和map类型

视图可以映射表中的 map字段。



### 7.4 视图零零碎碎相关的事情

1. hive会先解析视图，然后使用解析结果处理整个查询语句。eg：视图中使用了order by 或者limit语句

2. 创建视图时，可以使用`if not exists ` 或 `comment` 等子句修饰，和表类似

3. 视图的名称在表下唯一

4. 删除和查询视图，都和操作表类似。eg：`drop view if extists view_name  `, ` show views `

   



## *第 8 章 HiveQL：索引

为什么我们平时不使用索引？

好的， 索引已经被移除！

[wiki-index](https://cwiki.apache.org/confluence/display/Hive/IndexDev)

### 8.1 创建索引

```sql
CREATE INDEX index_name
ON TABLE base_table_name (col_name, ...)
AS 'index.handler.class.name'
[WITH DEFERRED REBUILD]
[IDXPROPERTIES (property_name=property_value, ...)]
[IN TABLE index_table_name]
[PARTITIONED BY (col_name, ...)]
[
   [ ROW FORMAT ...] STORED AS ...
   | STORED BY ...
]
[LOCATION hdfs_path]
[TBLPROPERTIES (...)]
[COMMENT "index comment"]
```

语句解释

* partitioned by 如果省略，那么索引表的分区和原表一样。 也就是说，此处指定的分区可以和原表不一致
* deferred（推迟），推迟重建索引。此处只是定义索引结构，不重建索引。
* as后面的是指定索引的结构，可以自己实现，此处用的是hive的。
* in table子句是在一张新表中保留索引数据

### 8.2 重建索引

```sql
alter index something_index
on table table_name(col) partition( dt = '2022-04-19')
rebuild
```



### 8.3 显示索引

```sql
show formatted index on table_name;
```



### 8.4 删除索引

```sql
 drop index if exists index_name on table table_name;
```

删表会自动删除索引

### 8.5 实现一个定制化索引处理器

实现 `public interface HiveIndexHandler {}` 接口



## 第 9 章 模式设计



### 9.1 按天划分的表

### 9.2 关于分区

为什么要使用分区？



分区适量，

### 9.3  唯一键和标准化

hive不同于关系型数据库没有唯一键，索引等，应尽量避免对非标准化数据进行连接join操作。

可以使用hive的数据类型，array，map和struct

### 9.4 同一份数据多种处理

一次从一个数据源产生多个数据聚合

```sql
from table
insert overwrite table_1 select * where action= '1'
insert overwrite table_2 select * where action= '2';
```

### 9.5 对于每个表的分区

在临时表中使用分区。

etl的过程可能比较复杂，如果出问题了了，还能保留分区数据，不会被直接覆盖。

### 9.6  分桶表数据存储

分桶：将数据集分解成更容易管理的若干部分的另一个技术

> 为什么要有？
>
> 分区可以隔离数据，优化查询。不过不是所有分区都可以形成合理的分区（实际业务场景多变，文件大小等问题）



建表时需要指定分桶字段

```sql
create table weblog (user_id int, url string, ip string)
partitioned by (dt string)
clustered by (user_id) -- 分桶字段
into 96 buckets; -- 分成几个
```



### 9.7 为表增加列

```sql
alter table table_name add columns(column_add string)
```



### 9.8 使用列存储表

hive通常使用行式存储，不过也可以使用列式SerDe来以混合列式格式存储信息。



#### 9.8.1 重复数据

#### 9.8.2 多列

如果列中有众多重复数据（状态、类型），或者有非常多的字段，是那么使用列式存储比较合适



### 9.9 几乎总是使用压缩

压缩可以使磁盘上存储的数据量变小，同时在查询的时候可以降低I/O来提高查询速度。

压缩和解压缩会消耗cpu资源所以压缩不是越小越合适，也需要一定取舍。



## 第 10 章 调优

HiveQL式一种声明式语言， 用户提交查询语句后，hive会将其转化为MapReduce Job。 



### 10.1 使用explain

explain可以帮助我们学习如果将查询转化成MapReduce任务的



### 10.2 explain extended

加上 extended 可以输出有关计划的额外信息（物理信息，例如文件名等）



### 10.3 限制调整

通过修改配置， 使limit的时候，不走全表，对源数据进行抽样。

但是这个真的合适吗？ 很多情况下，像排序sum啥的，会有问题吧。

### 10.4 join 优化

同 6.4.2

ps: 大表放右边

### 10.5 本地模式

对于小数据集， 可以在执行过程中，临时启用本地模式

``` sql
set hive.exec.model.local.auto=true
```



### 10.6 并行执行

同一个任务会被转化成多个阶段（map reduce阶段，合并阶段，limit阶段等）。默认情况下，一次只会执行一个阶段。不过有些job可能会包含多个阶段，而且这些阶段如果不是互相依赖的，那么可以并发执行，减少job的执行时间。



通过开启 配置

``` sql
set hive.exec.parallel=true
```



### 10.7 严格模式

hive.mapred.mode = strict

限制的三种情况：

1. where 条件不带分区， 直接扫描全表的
2. 使用了order by 的必须带limit
3. 限制笛卡尔积



### 10.8 调整mapper 和reducer 个数

如果设置过多会导致启动阶段以及调度运行时产生过多开销，而如果设置的过于保守，则可能无法充分利用集群内在的并行性。 



hive是按照输入数据量大小来确定reducer 个数的，可以通过 dfs -count命令来计算输入量大小，计算指定目录下所有数据的大小。

命令`hive.exec.reducers.bytes.per.reducer` 查看reducer大小（默认1G，生产是256mb）

还有需要注意的是，实际文件数量，经过map之后，可能会有出入。



`mapred.reduce.tasks` 设置reducer数量

`hive.exec.reducers.max` 设置单个任务使用的最大reducer数量

> 集群总 reduce槽位个数*1.5/ 执行中的查询的平均个数



### 10.9 JVM重用

`mapred.job.reuse.jvm.num.tasks` 设置一个jvm实例在同一个job中被重新使用多少次。



### 10.10 索引

没啥用



### 10.11 动态分区调整

一次创建多个分区，见5.2.1

这个好用，但是如果一次创建分区过多容易出问题。 所以可以通过配置限制单次创建的分区数，来使用这个功能。



### 10.12 推测执行

在分布式集群环境下，因为程序Bug(包括Hadoop本身的bug)，负载不均衡或者资源分布不均等原因，会造成同一个作业的多个任务之间运行速度不一致，有些任务的运行速度可能明显慢于其他任务（比如一个作业的某个任务进度只有50%，而其他所有任务已经运行完毕），则这些任务会拖慢作业的整体执行进度。为了避免这种情况发生，Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。



### 10.13 单个MapReduce 中多个Group By

配置项 ： `hive.multigroupby.singlemr`



### 10.14 虚拟列

hive有两种虚拟列

1. `INPUT_FILE_NAME`  输入文件名，标记着mr任务的map task的输入数据中每条记录的来源（即这些输入数据存储路径，它是属于哪个目录下的哪个文件的）。
2. `BLOCK_OFFSET_INSIDE_FILE` 记录在文件中的偏移量，一个表中有多条记录，而这个`BLOCK_OFFSET_INSIDE_FILE`就是计算表中每条记录相对于表中第一条记录的第一个字符的字节偏移量（因为一个字符的大小就是一个字节，所以也可以说是字符个数偏移量）。但对于分区表而言，是每个分区中的每条记录相对于该分区中的第一条记录的第一个字符的字节偏移量，而不是相对于整张表的第一条记录的第一个字符的了





## 第 11 章  其他文件格式和压缩方法

hive的一个独特功能就是： hive不会强制要求将数据转换成特定格式才能使用。hive可以利hadoop的InputFormat API从不同过的数据源读取数据，同样的，也可以使用 OutputFormat API 将数据写成不同格式。



### 11.1 确定安装编解码器

查看属性`io.compression.codecs`， 支持的编解码器

### 11.2 选择一种压缩编/解码器

取舍：

* 压缩减少磁盘空间，减小磁盘和网络I/O操作。

* 压缩和解压缩增加cpu开销

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 是否可切分 | 对应的编码/解码器                          |
| -------- | ----- | ------- | ---------- | ---------- | ------------------------------------------ |
| DEFAULT  | 无    | DEFAULT | .deflate   | 否         | org.apache.hadoop.io.compress.DefaultCodec |
| Gzip     | gzip  | DEFAULT | .gz        | 否         | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | bzip2 | bzip2   | .bz2       | 是         | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | lzop  | LZO     | .lzo       | 否         | com.hadoop.compression.lzo.LzopCodec       |
| LZ4      | 无    | LZ4     | .lz4       | 否         | org.apache.hadoop.io.compress.Lz4Codec     |
| Snappy   | 无    | Snappy  | .snappy    | 否         | org.apache.hadoop.io.compress.SnappyCodec  |





### 11.3 开启中间压缩

对中间数据进行压缩，可以减少job中的map和reduce task间的数据传输量。

1. 开启hive中间传输数据压缩功能

```javascript
set hive.exec.compress.intermediate=true;
```

复制

2. 开启mapreduce中map输出压缩功能

```javascript
set mapreduce.map.output.compress=true;
```

复制

3. 设置mapreduce中map输出数据的压缩方式

```javascript
set mapreduce.map.output.compress.codec= org.apache.hadoop.io.compress.Snap
```



### 11.4 最终输出结果压缩



1. 开启hive最终输出数据压缩功能

```javascript
set hive.exec.compress.output=true;
```

2. 开启mapreduce最终输出数据压缩

```javascript
set mapreduce.output.fileoutputformat.compress=true;
```

3. 设置mapreduce最终数据输出压缩方式

```javascript
set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec;
```

4. 设置mapreduce最终数据输出压缩为块压缩

```javascript
set mapreduce.output.fileoutputformat.compress.type=BLOCK;
```













































