[toc]

# 视图、事务、索引

## 一、视图 view

> * 临时表、虚表
> * 针对多表查询
> * 作用
>   * 通过视图可以将经常用到的多表查询结果进行保存，下次再使用数据时，直接从视图中查询数据即可
>   * **对于频繁更新的数据，不适合创建视图**

### 1、创建视图

* 语法

```bash
CREATE VIEW view_name
AS
SELECT;
```

```bash
mysql> create view stu_teacher
    -> as
    -> select students.Name, students.Age, tutors.Tname from students, tutors where students.TID = tutors.TID ;
Query OK, 0 rows affected (0.00 sec)

mysql> show tables;
+------------------+
| Tables_in_jiaowu |
+------------------+
| courses          |
| scores           |
| stu_teacher      |
| students         |
| time             |
| tutors           |
+------------------+
6 rows in set (0.01 sec)

mysql> select * from stu_teacher;
+-----------+------+--------------+
| Name      | Age  | Tname        |
+-----------+------+--------------+
| GuoJing   |   19 | Miejueshitai |
| YangGuo   |   17 | HongQigong   |
| DingDian  |   25 | Jinlunfawang |
| HuFei     |   31 | YiDeng       |
| HuangRong |   16 | NingZhongze  |
+-----------+------+--------------+
5 rows in set (0.00 sec)

```

### 2、删除视图

```bash
mysql> DROP VIEW stu_teacher;
Query OK, 0 rows affected (0.00 sec)
```

## 二、事务 Transaction

> * 针对修改操作
> * 保证修改操作要么同时成功执行、要么回滚（同时删除）

### 1、启动事务

> start transaction;

### 2、提交事务

> commit;

### 3、回滚事务

> rollback;

## 三、索引 Index

### 1、索引的作用

> * 建立合适的索引，优化、提高查询速度
> * 针对数据表
> * 不推荐使用经常变化的数据字段建立索引，否则每次更新数据都会拖慢数据库

### 2、查询索引

* 默认会使用主键字段生成索引

```bash
mysql> SHOW INDEX FROM Account;
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table   | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Account |          0 | PRIMARY  |            1 | id          | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
```

### 3、创建索引

* 语法

```mysql
create index index_name on tb_name(字段，字段)
```

```mysql
mysql> create index index_name on Phone(price);
Query OK, 0 rows affected (1.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

```

### 4、删除索引

```bash
mysql> DROP INDEX username_key ON Account;
```

### 5、使用查询分析器explain

#### 1、无索引的情况

```mysql
mysql> explain select * from Phone where Price=4000;
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+-------------+
|  1 | SIMPLE      | Phone | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 476616 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```

#### 2、有索引的情况

```mysql
mysql> explain select * from Phone where Price=4000;
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | Phone | NULL       | ref  | index_name    | index_name | 4       | const |   44 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```

