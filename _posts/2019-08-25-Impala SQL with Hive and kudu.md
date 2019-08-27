---
layout: post
date: 2019-08-25 16:48
status: public
title: Impala SQL with Hive and kudu
---

# 背景
全来自Apache大家族
Hive是蜂巢 基于hadoop的数据仓库工具
建立了类SQL的查询语句HQL，因此学习成本低。同时也可自定义用MapReduce

Kudu是条纹羚 是Hadoop一个新的解决方案，目标是时间序列数据与快速插入与更新新数据，根据历史数据刷新预测模型。支持Impala

Impala是黑斑羚 适用于Hadoop的现代开源SQL引擎

所以仅从使用上考虑的话，学习Impala就行了。

# Impala OverView
impala支持大多数和HiveQL相同的语句和子句，包括但不限于JOIN，AGGREGATE， DISTINCT，UNION ALL，ORDER BY，LIMIT和（不相关）子查询中FROM的条款。Impala也支持INSERT INTO和INSERT OVERWRITE。

impala支持的数据类型具有相同的名字和含义等同Hive数据类型： STRING，TINYINT，SMALLINT，INT， BIGINT，FLOAT，DOUBLE，BOOLEAN， STRING，TIMESTAMP。

参考：
https://impala.apache.org/docs/build/html/topics/impala_langref.html
Impala和Hive之间的SQL差异：https://impala.apache.org/docs/build/html/topics/impala_langref_unsupported.html#langref_hiveql_delta
内置函数：https://impala.apache.org/docs/build/html/topics/impala_functions.html#builtins

# Impala SQL

## DDL
CREATE， DROP或ALTER 与SQL标准一致

注意SYNC_DDL是否开启就行，SYNC_DDL表示同步所有主机，开启的话所有DDL和DML中的INSERT会有延迟等待所有主机结点都已更新。

COMPUTE STATS语句收集有关表中数据的卷和分布以及所有关联列和分区的信息。

创建用户定义函数（UDF），它可以使用期间实现定制逻辑SELECT或INSERT 操作。
使用以下命令创建持久性Java UDF CREATE FUNCTION：
```sql
CREATE FUNCTION [IF NOT EXISTS] [db_name.]function_name
  LOCATION 'hdfs_path_to_jar'
  SYMBOL='class_name'
```
  
## DML
DML指的是“数据操作语言”，它是修改表中存储的数据的SQL语句的子集。由于Impala专注于查询性能并利用HDFS存储的仅附加性质，因此目前Impala仅支持一小组DML语句：

+ DELETE语句（仅限Impala 2.8或更高版本）。仅适用于Kudu表。
+ INSERT声明。
+ LOAD DATA声明。不适用于HBase或Kudu表。
+ Refresh声明（仅限Impala 2.8或更高版本）。仅适用于Kudu表。
+ UPSERT声明（仅限Impala 2.8或更高版本）。仅适用于Kudu表。

要模拟其他数据库系统中的语句UPDATE或DELETE语句的效果，通常使用INSERT或CREATE TABLE AS SELECT将数据从一个表复制到另一个表，在复制操作期间过滤掉或更改相应的行。

### INSERT

+ INSERT INTO语法将数据追加到表。现有数据文件保持原样，插入的数据放入一个或多个新数据文件中。
+ INSERT OVERWRITE语法表中的替换数据。目前，覆盖的数据文件立即被删除; 他们没有通过HDFS垃圾机制。

+ 用UPSERT 或 INSERT IGNORE 处理Kudu表
> 目前，INSERT OVERWRITE语法不能与Kudu表一起使用。
> Kudu表需要每行唯一的主键。如果INSERT 语句尝试插入主键列的值与现有行相同的行，则会丢弃该行并继续插入操作。当由于重复的主键而丢弃行时，语句将以警告结束，而不是错误。（这是对Kudu早期版本的更改，其中默认情况下在这种情况下返回错误，并且INSERT IGNORE需要语法 才能使语句成功。该IGNORE子句不再是INSERT 语法的一部分。）
> 对于您希望替换具有重复主键值的行而不是丢弃新数据的情况，您可以使用该UPSERT 语句而不是INSERT。UPSERT插入全新的行，对于与表中现有主键匹配的行，更新非主键列以反映“upserted”数据中的值 。
> Impala 和 kudu不支持事务。Impala and Kudu do not support transactions

### REFRESH
该REFRESH语句从Metastore数据库重新加载表的元数据，并从HDFS NameNode执行文件和块元数据的增量重新加载。REFRESH用于避免Impala与外部元数据源（即Hive Metastore（HMS）和NameNodes）之间的不一致。

Kudu考虑因素：
> 默认情况下，Kudu表的大部分元数据都由底层存储层处理。Kudu表对Metastore数据库的依赖性较小，并且在Impala端需要较少的元数据缓存。例如，有关Kudu表中分区的信息由Kudu管理，Impala不会缓存Kudu表的任何块位置元数据。如果Kudu服务未与Hive Metastore集成，Impala将在Hive Metastore中管理Kudu表元数据。
> Kudu表所需 的REFRESH和INVALIDATE METADATA语句比HDFS支持的表更少。在Kudu表中添加，删除或更新数据时，即使通过使用Kudu API的客户端程序直接对Kudu进行更改，也不需要任何语句。**仅在更改Kudu表模式（例如添加或删除列）后运行或运行 REFRESH table_name或INVALIDATE METADATA table_name**