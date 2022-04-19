[TOC]



# 《hive 编程指南》阅读笔记



## 第1章 基础知识

### 1.1 Hadoop 和 MapReduce综述



## 第5章 HiveQL：数据操作

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





## 第6章 HiveQL：查询



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



































