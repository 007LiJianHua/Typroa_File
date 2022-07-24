[toc]

# SQL语句---库表管理操作

## 一、SQL类型--结构化查询语言

- DDL		数据定义语言

- - 对库、表、列的管理
  - create, drop, alter

- DML		数据管理/操作语言

- - 对表中的数据进行操作
  - insert, delete、update、select

- DCL		数据控制语言

- - 对数据库用户、权限进行管理
  - grant, revoke

## 二、数据库管理操作

### 1、查看数据库

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

> MySQL首次启动初始化生成的数据库，作用说明
>
> * information_schema
>   * 保存数据库服务器上的元数据信息（数据库名称、数据表名、字段名称、数据类型）
> * mysql
>   * 保存用户名、密码、权限
> * performance_schema
>   * 保存数据服务器性能相关的数据、例如连接数、进程、线程
> * sys
>   * 对information_schema数据库的简化，方便数据库管理元查看

### 2、创建数据库

```bash
>create database <数据库名称>

mysql> create database ljh;

2、创建一个使用utf8字符集的mydb2数据库：

CREATE DATABASE mydb2 CHARACTER SET utf8；

3、创建一个使用utf8字符集，并带比较规则的mydb3数据库。

CREATE DATABASE mydb3 CHARACTER SET utf8 COLLATE utf8_general_ci;

```

### 3、查看数据库创建信息

```bash
#指定数据库的字符集，在数据库中可以显示中文
mysql> create database testdb charset utf8;
Query OK, 1 row affected (0.00 sec)

mysql> show create database testdb;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| testdb   | CREATE DATABASE `testdb` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 3+、不显示----用\G

```bash
mysql> show create table class\G;
*************************** 1. row ***************************
       Table: class
Create Table: CREATE TABLE `class` (
  `classid` int NOT NULL,
  `studentid` int DEFAULT NULL,
  PRIMARY KEY (`classid`),
  KEY `fk_class_studentid` (`studentid`)
) ENGINE=InnoDB DEFAULT CHARSET=gbk
1 row in set (0.00 sec)

ERROR: 
No query specified
```



### 4、删除数据库

```bash
mysql> drop database caiwu;
Query OK, 0 rows affected (0.00 sec)
```

### 5、切换数据库

```bash
#切换数据库
mysql> use game;
Database changed
#查看当前数据库
mysql> select database();
+------------+
| database() |
+------------+
| game       |
+------------+
1 row in set (0.00 sec)

```

## 三、数据表的管理操作

### 1、创建表

```bash
>create table 表明（字段名称 数据类型【属性】，字段名称 数据类型【属性】.....）
```

```bash
mysql> use game;
Database changed
mysql> 
mysql> create table account(
    -> id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
    -> name CHAR(20) NOT NULL,
    -> password CHAR(30) NOT NULL,
    -> level INT UNSIGNED default 1);
Query OK, 0 rows affected (0.00 sec)
```

##### 数据类型

> - 数字
>
> - - int、smallint、tinyint、bigint、mediumint    整型
>
>   - - int unsigned	无符号整型
>
>   - float(5,3)		单精度浮点数   4.178
>
>   - double(5,3)	双精度浮点数
>
>   - decimal(5,3)	财务
>
> * 字符
>
> * - char(10)		定长字符
>   - varchar(20)	变长字符
>     - char(n)是定长格式，格式为char(n)的字段固定占用n个字符宽度，如 据长度超过n将被截取多出部分，如果长度小于n就用空字符填充。
>     - varchar(n)是变长格式，这种格式的字段根据实际数据长度分配空间，不浪费对于的空间，但是搜索数据的速度会麻烦一点。
>     - 一般地说，只要一个表有一个字段定义为varchar(n)类型，那么其余用char(n)定义的字段实际上也是varchar(n)类型。 如果你的长度本身不长，比如就3～10个字符，那么使用char(n)格式效率比较高，搜索速度快。但是如果有的数据很长，有的数据有比较短，比如注册用户的简介这样的字段，实在没有办法，而且很在乎浪费
>   - text
>   - enum("男", "女")	枚举      ENUM("yes", "no")
>
> * 日期时间
>
> * - date			YYYY-MM-DD
>   - datetime		YYYY-MM-DD HH:MM:SS
>   - timestamp	YYYY-MM-DD HH:MM:SS
>
> 在数据变化时，时间会自动更新

##### 属性

> - primary key		主键
> - unique key		惟一键
> - not null			不允许为空
> - auto_incrememnt	自动增长, 配合primary key使用
> - default "数据"		设置字段的默认值

### 2、查看表

```bash
mysql> show tables;
+----------------+
| Tables_in_game |
+----------------+
| account        |
| tb01           |
+----------------+
2 rows in set (0.00 sec)
```

### 3、查看表结构

```bash
mysql> desc account;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id       | int(11)          | NO   | PRI | NULL    | auto_increment |
| name     | char(20)         | NO   |     | NULL    |                |
| password | char(30)         | NO   |     | NULL    |                |
| level    | int(10) unsigned | YES  |     | 1       |                |
+----------+------------------+------+-----+---------+----------------+
4 rows in set (0.03 sec)
```

### 4、删除表

```bash
mysql> drop table tb01;
```

## 四、MySQL存储引擎

### 1、查看表创建信息

```bash
mysql> show create table account\G;
*************************** 1. row ***************************
       Table: account
Create Table: CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` char(20) NOT NULL,
  `password` char(30) NOT NULL,
  `level` int(10) unsigned DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

### 2、查看支持的存储引擎

```bash
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

### 3、存储引擎介绍

> 不同的存储引擎影响数据库性能、功能
>
> * InnoDB
>   
>   * 支持**事务Transaction**、外键
>   * 支持行级锁 row level lock
>   
> * MYISAM
>
> * - 支持表级锁
>   - 不支持事务
>   - 查询性能较高
>
> * MRG_MYISAM
>
> * - 支持将多个MYISAM的表进行合并
>
> * MEMORY
>
> * - 以内存来进行存储数据
>
> * BLACKHOLE
>
> * - 黑洞