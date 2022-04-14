[TOC]



# 《hive 编程指南》阅读笔记



## 第1章 基础知识

### 1.1 Hadoop 和 MapReduce综述



## 第6章



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

















