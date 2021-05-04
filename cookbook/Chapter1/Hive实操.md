# 第一章

# Hive 实操

## 一、Database

### 1.1 查看数据库列表

```sql
show databases;
```

![image-20200902204156290](https://img.jinguo.tech/typora/image-20200902204156290.png?imageslim)

### 1.2 查看数据库列表

```sql
use database_name;
```

![image-20200902204350677](https://img.jinguo.tech/typora/image-20200902204350677.png?imageslim)

### 1.3 新建数据库

- 语法

  ```sql
  CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name   --DATABASE|SCHEMA 是等价的
    [COMMENT database_comment] --数据库注释
    [LOCATION hdfs_path] --存储在 HDFS 上的位置
    [WITH DBPROPERTIES (property_name=property_value, ...)]; --指定额外属性
  ```

- 示例

  ```sql
  CREATE DATABASE IF NOT EXISTS hive_test
    COMMENT 'hive database for test'
    WITH DBPROPERTIES ('create'='jinguo');
  ```

  ![image-20200902203705654](https://img.jinguo.tech/typora/image-20200902203705654.png?imageslim)

### 1.4 查看数据库信息

- 语法：

```sql
DESC DATABASE [EXTENDED] db_name; --EXTENDED 表示是否显示额外属性
```

- 示例：

```sql
DESC DATABASE  EXTENDED hive_test;
```

![image-20200903100904811](https://img.jinguo.tech/typora/image-20200903100904811.png?imageslim)

### 1.5 删除数据库

语法：

```sql
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

*默认行为是 **RESTRICT**，如果数据库中存在表则删除失败。要想删除库及其中的表，可以使用 **CASCADE** 级联删除。*

示例：

```sql
  DROP DATABASE IF EXISTS hive_test;
```

![image-20200903111952592](https://img.jinguo.tech/typora/image-20200903111952592.png?imageslim)

```verilog
Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database hive_test is not empty. One or more tables exist.)
```

*当数据库中存在表的时候，删除时需要用**CASCADE***关键字

```sql
DROP DATABASE IF EXISTS hive_test CASCADE;
```

![image-20200903112539723](https://img.jinguo.tech/typora/image-20200903112539723.png?imageslim)

## 二、创建表

### 2.1 建表语法

- 语法

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name     --表名
  [(col_name data_type [COMMENT col_comment],
    ... [constraint_specification])]  --列名 列数据类型
  [COMMENT table_comment]   --表描述
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]  --分区表分区规则
  [
    CLUSTERED BY (col_name, col_name, ...) 
   [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS
  ]  --分桶表分桶规则
  [SKEWED BY (col_name, col_name, ...) ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)  
   [STORED AS DIRECTORIES] 
  ]  --指定倾斜列和值
  [
   [ROW FORMAT row_format]    
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  
  ]  -- 指定行分隔符、存储文件格式或采用自定义存储格式
  [LOCATION hdfs_path]  -- 指定表的存储位置
  [TBLPROPERTIES (property_name=property_value, ...)]  --指定表的属性
  [AS select_statement];   --从查询结果创建表
```

### 2.2 创建内部表

- 示例

```sql
 CREATE TABLE emp(
    empno INT,
    ename STRING,
    job STRING,
    mgr INT,
    hiredate TIMESTAMP,
    sal DECIMAL(7,2),
    comm DECIMAL(7,2),
    deptno INT)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t";
```

![image-20200903180953529](https://img.jinguo.tech/typora/image-20200903180953529.png?imageslim)

### 2.3 创建外部表

- 示例

```sql
 CREATE EXTERNAL TABLE emp_external(
    empno INT,
    ename STRING,
    job STRING,
    mgr INT,
    hiredate TIMESTAMP,
    sal DECIMAL(7,2),
    comm DECIMAL(7,2),
    deptno INT)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t"
    LOCATION '/hive/emp_external';
```

![image-20200903181743473](https://img.jinguo.tech/typora/image-20200903181743473.png?imageslim)

*用**DESC TABLENAME**查看表格信息*

![image-20200903184628794](https://img.jinguo.tech/typora/image-20200903184628794.png?imageslim)

*用**DESC FORMATTED TABLENAME**命令可以查看表的详细信息*

![image-20200903190348379](https://img.jinguo.tech/typora/image-20200903190348379.png?imageslim)

*通过**hdfs dfs -ls**命令可以看到Location的外部表已经存在*

![image-20200903185141700](https://img.jinguo.tech/typora/image-20200903185141700.png?imageslim)



### 2.4 内部表和外部表区别

**创建表时**

- 创建内部表：会将数据移动到数据仓库指向的路径；
- 创建外部表：仅记录数据所在的路径， 不对数据的位置做任何改变。

**删除表时**

- 内部表的元数据和数据会被一起删除
- 外部表只删除元数据，不删除数据。外部表相对来说更加安全，数据组织更加灵活，方便共享源数据。

**总结：**

1. 未被external修饰的是内部表【*managed table*】，被external修饰的为外部表【*external table*】。

2. 内部表数据由Hive自身管理，外部表数据由HDFS管理。

3. 内部表数据存储在*hive.metastore.warehouse.dir*【默认:*/user/hive/warehouse*】，外部表数据存储位置由用户自己决定。如 *location '/hive/emp_external'*

   ![image-20200904222601364](https://img.jinguo.tech/typora/image-20200904222601364.png?imageslim)

   ![image-20200904222820733](https://img.jinguo.tech/typora/image-20200904222820733.png?imageslim)

4. 删除内部表会直接删除元数据【*metadata*】及**存储数据**，删除外部表仅仅删除元数据，*HDFS*上的文件不会被删除。

5. 对内部表的修改会直接同步到元数据，而对外部表的表结构和分区进行修改，则需要修改【*MSCK REPAIR TABLE table_name*】。

### 2.5 创建分区表

- 示例

``` sql
 CREATE EXTERNAL TABLE emp_partition(
    empno INT,
    ename STRING,
    job STRING,
    mgr INT,
    hiredate TIMESTAMP,
    sal DECIMAL(7,2),
    comm DECIMAL(7,2)
    )
    PARTITIONED BY (deptno INT)  -- 按照部门编号进行分区
    ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t"
    LOCATION '/hive/emp_partition';
```

![image-20200903191518107](https://img.jinguo.tech/typora/image-20200903191518107.png?imageslim)

*用**DESC TABLENAME**查看表格信息*

![image-20200903193011185](https://img.jinguo.tech/typora/image-20200903193011185.png?imageslim)

*用**DESC FORMATTED TABLENAME**命令可以查看表的详细信息*

![image-20200903192022966](https://img.jinguo.tech/typora/image-20200903192022966.png?imageslim)

*通过**hdfs dfs -ls**命令可以看到Location的外部表已经存在*

![image-20200903191648232](https://img.jinguo.tech/typora/image-20200903191648232.png?imageslim)

### 2.6 分桶表

- 示例

```sql
 CREATE EXTERNAL TABLE emp_bucket(
    empno INT,
    ename STRING,
    job STRING,
    mgr INT,
    hiredate TIMESTAMP,
    sal DECIMAL(7,2),
    comm DECIMAL(7,2),
    deptno INT)
    CLUSTERED BY(empno) SORTED BY(empno ASC) INTO 4 BUCKETS  --按照员工编号散列到四个 bucket 中
    ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t"
    LOCATION '/hive/emp_bucket';
```

![image-20200903192305213](https://img.jinguo.tech/typora/image-20200903192305213.png?imageslim)

*用**DESC TABLENAME**查看表格信息*

![image-20200903192644607](https://img.jinguo.tech/typora/image-20200903192644607.png?imageslim)

*用**DESC FORMATTED TABLENAME**命令可以查看表的详细信息*![image-20200903192505630](https://img.jinguo.tech/typora/image-20200903192505630.png?imageslim)

*通过**hdfs dfs -ls**命令可以看到Location的外部表已经存在*

![image-20200903192829644](https://img.jinguo.tech/typora/image-20200903192829644.png?imageslim)

### 2.7 倾斜表

*通过指定一个或者多个列经常出现的值（严重偏斜），**Hive**会自动将涉及到这些值的数据拆分为单独的文件。在查询时，如果涉及到倾斜值，它就直接从独立文件中获取数据，而不是扫描所有文件，这使得性能得到提升*

- 示例

``` sql
 CREATE EXTERNAL TABLE emp_skewed(
    empno INT,
    ename STRING,
    job STRING,
    mgr INT,
    hiredate TIMESTAMP,
    sal DECIMAL(7,2),
    comm DECIMAL(7,2)
    )
    SKEWED BY (empno) ON (66,88,100)  --指定 empno 的倾斜值 66,88,100
    ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t"
    LOCATION '/hive/emp_skewed';   
```

![image-20200903193642678](https://img.jinguo.tech/typora/image-20200903193642678.png?imageslim)