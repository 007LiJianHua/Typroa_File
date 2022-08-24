[toc]

# SQL语句---数据管理操作

## 一、数据管理操作

### 1、添加数据

> * insert into 表名（字段名，字段名）values (数据，数据)

```bash
mysql> insert into account(name, password, level) values("martin", "redhat", 10);
Query OK, 1 row affected (0.00 sec)

mysql> insert into account(name, password) values("robin", "123");
Query OK, 1 row affected (0.00 sec)

mysql> select * from account;
+----+--------+----------+-------+
| id | name   | password | level |
+----+--------+----------+-------+
|  1 | martin | redhat   |    10 |
|  2 | robin  | 123      |     1 |
+----+--------+----------+-------+
2 rows in set (0.00 sec)

mysql> insert into account(name,password,level) values("lz", "aaa", 30),("tome", "bbb", 8);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> insert into account values(5, "king", "ccc", 16);
Query OK, 1 row affected (0.00 sec)
```

### 2、删除数据

> delete from 表明 where 条件

```bash
mysql> delete from account where level < 15;
```

### 3、更新数据

> update 表名 set 字段名称=新值 where 条件

```bash
mysql> update account set password="123" where name="lz";
```



## 二、数据查询

### 1、查询类型

* 单表查询
* 多表查询/连接查询
* 嵌套查询/子查询

### 2、单表查询

#### 1、导入jiaowu数据库

```bash
[root@localhost ~]# mysql -uroot -pWWW.1.com < jiaowu.sql 
```

#### 2、查询语法

> select 字段名称，字段名称  from 表名 [查询字句]

```bash
mysql> select * from tutors;
mysql> select Tname, Age from tutors;
mysql> select Tname as 教师姓名, Age as 年龄 from tutors;
```

#### 3、按条件查询

> 条件的写法
>
> * 数学运算符
>
>   * =，!=,>,<,<=,>=
>
>   ```bash
>   mysql> select Tname, Age from tutors where Age > 80;
>   mysql> select Tname, Age from tutors where Tname = "HuYiDao";
>   ```
>
> * 逻辑运算符
>
>   * and,or,not
>
>   ```bash
>   mysql> select Tname, Age from tutors where Age >= 70 and Age <= 80;
>   
>   mysql> select Tname, Age from tutors where Age between 70 and 80;
>   
>   mysql> select * from tutors where Tname = "Huyidao" or Tname = "YiDeng" ;
>   
>   IN("数据","数据", "数据")
>   mysql> select * from tutors where Tname in("Huyidao","YiDeng");
>   
>   mysql> select * from tutors where not Age > 80;
>   ```
>
> * 模糊查询
>
>   * LIKE “通配符”
>     * %  任意字符
>     * _   任意单个字符
>
> ```bash
> mysql> select Tname, Age from tutors where Tname like "%ang%";
> mysql> select Tname, Age from tutors where Tname like "%ai";
> ```
>
> * RLIKE “正则表达式”
>
> ```bash
> mysql> select Tname from tutors where Tname RLIKE "^[HN]";
> mysql> select Tname from tutors where Tname RLIKE "ai$";
> mysql> select Tname from tutors where Tname RLIKE "ang";
> ```
>
> 

#### 4、IS NULL / IS NOT NULL

```bash
mysql> select * from students where CID2 is NULL;
mysql> select * from students where CID2 is not NULL;
```

### 3、排序

> order by 字段名称 [ASC|DESC]
>
> * ASC  升序
> * DESC  降序

```bash
mysql> select * from tutors order by Age ;
mysql> select * from tutors order by Age DESC;
```

### 4、限制结果的行数

> limit n[m]
>
> ​	limit 2
>
> ​	limit 2,4
>
> 忽略前n行数，包括n，显示后续的连续m行

```bash
mysql> select * from tutors limit 2;
mysql> select * from tutors limit 2,4;

mysql> select Tname, Age from tutors order by Age Desc limit 1;

mysql> select Tname, Gender, Age from tutors where Gender = "M" order by Age limit 1;
```

### 5、聚合函数

> - sum( )	求和
> - avg( )	平均值 
> - max( )	最大值
> - min( )	最小值 
> - count( )	计数

```bash
mysql> select avg(Age) as 平均年龄 from tutors;
+--------------+
| 平均年龄     |
+--------------+
|      67.5556 |
+--------------+
mysql> select count(*) from tutors;
```

### 6、数据分组

> group by 字段名称 [having 条件]
>
> 执行顺序：
>
> ​	**先分组--->对每组数据先进行聚合运算--->having 条件对结果进行过滤**

```bash
mysql> select count(*) as 人数, gender as 性别 from tutors group by Gender;
mysql> select avg(Age) as 平均年龄, gender as 性别 from tutors group by Gender;
```

```bash
mysql> select avg(age) as 平均年龄 from tutors group by gender having 平均年龄 > 65;
```

### 7、去重

distinct 字段名称

```bash
mysql> select distinct Tname from tutors;
```



## 三、嵌套查询/子查询

* 将一个查询的结果作为另一个查询的条件使用

```bash
    mysql> select Tname, Age from tutors where Age > (select avg(Age) from tutors);
mysql> select Tname, Age from tutors where Age in(select Age from students);
```

## 四、日期时间处理函数

> - year()
> - month()
> - day()
> - date()
> - time()
> - hour()
> - minute()

```bash
mysql> insert into time(date_time) values ('2022-02-23 17:11:24');

Query OK, 1 row affected (0.00 sec)

mysql> select * from time;
+---------------------+
| date_time           |
+---------------------+
| 2022-02-23 17:11:24 |
+---------------------+
1 row in set (0.00 sec)


mysql> select year(date_time) from time;
+-----------------+
| year(date_time) |
+-----------------+
|            2022 |
+-----------------+
1 row in set (0.01 sec)

mysql> select month(date_time) from time;
+------------------+
| month(date_time) |
+------------------+
```

## 五、多表查询/连接查询

### 1、连接查询类型

**前提：多张表之间要存在相关联的字段**

* 内连接
* 外连接
  * 左外连接
  * 右外连接

### 2、内连接

> * 特征：相关联字段存在相同的值时，才会显示结果
> * 语法：
>   * select 表名.字段名称 表名.字段名 from 表r join 表名 on 相关联字段

```bash
mysql> select students.Name ,students.Age ,tutors.Tname  from students inner join tutors on students.TID = tutors.TID;
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

* 使用select  from  where

```bash
mysql> select students.Name ,students.Age ,tutors.Tname  from students,tutors where students.TID = tutors.TID;
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

### 3、左外连接

> * 特征： 以左表为主，显示左表中所有数据；相关联存在相同值时，显示对应数据，没有相同的值时显示为NULL
>   * <font color="red">**注意：多个表中联结查询时，先去找到能连接表的表,利用column**</font>
> * 语法:
>   * select 表名.字段名称,表名.字段名称,表名.字段名称 from 表名 left join 表名 on 相关联字段  

* 三个表使用 select from where

```bash
mysql> select students.SID,students.Name,courses.CID,courses.Cname ,scores.Score 
	   from · left join scores on students.SID = scores.SID left join courses on scores.CID = courses.CID;
+-----+--------------+------+-----------------+-------+
| SID | Name         | CID  | Cname           | Score |
+-----+--------------+------+-----------------+-------+
|   1 | GuoJing      |    7 | Qiankundanuoyi  |    45 |
|   1 | GuoJing      |    2 | TaiJiquan       |    90 |
|   2 | YangGuo      |    3 | Yiyangzhi       |    71 |
|   2 | YangGuo      |    2 | TaiJiquan       |    67 |
|   3 | DingDian     |    1 | Hamagong        |    99 |
|   3 | DingDian     |    6 | Qishangquan     |    32 |
|   4 | HuFei        |   10 | Jiuyinbaiguzhua |    36 |
|   4 | HuFei        |    8 | Wanliduxing     |    95 |
|   5 | HuangRong    | NULL | NULL            |  NULL |
|   6 | YueLingshang | NULL | NULL            |  NULL |
|   7 | ZhangWuji    | NULL | NULL            |  NULL |
|   8 | Xuzhu        | NULL | NULL            |  NULL |
|   9 | LingHuchong  | NULL | NULL            |  NULL |
|  10 | YiLin        | NULL | NULL            |  NULL |
+-----+--------------+------+-----------------+-------+
14 rows in set (0.00 sec)

```

### 4、右外连接

> 特征: 以右表为主，显示右表中所有数据；相关联存在相同值时，显示对应数据，没有相同的值时显示为NULL
>
> 语法: 
>
> select 表名.字段名称,表名.字段名称,表名.字段名称 from 表名 right join 表名 on 相关联字段 

```bash
mysql> select students.Name, students.Age, courses.Cname 
    -> from students left join courses
    -> on students.CID1 = courses.CID;
```

## 六、测试题

数据库为jiaowu

1、查询数据库中学生所选课的平均成绩大于67分的学生姓名，学号，及平均分数

```mysql
mysql> select students.Name, students.SID, avg(scores.Score) as 平均成绩 
	   from students, scores 
	   where students.SID = scores.SID 
	   group by students.SID 
	   having avg(scores.Score)>67;
+---------+-----+--------------+
| Name    | SID | 平均成绩     |
+---------+-----+--------------+
| YangGuo |   2 |           69 |
| GuoJing |   1 |         67.5 |
+---------+-----+--------------+
2 rows in set (0.00 sec)

```

2、查询所有学生各科目得分（显示学生姓名，课程名称，成绩，对所有学生按每课程的成绩倒叙排序）

```mysql
mysql> select students.Name, courses.Cname, scores.Score 
	   from students left join scores 
	   on students.SID = scores.SID 
	   left join courses 
	   on scores.CID = courses.CID 
	   order by scores.Score asc;
+--------------+-----------------+-------+
| Name         | Cname           | Score |
+--------------+-----------------+-------+
| HuangRong    | NULL            |  NULL |
| YueLingshang | NULL            |  NULL |
| ZhangWuji    | NULL            |  NULL |
| Xuzhu        | NULL            |  NULL |
| LingHuchong  | NULL            |  NULL |
| YiLin        | NULL            |  NULL |
| DingDian     | Qishangquan     |    32 |
| HuFei        | Jiuyinbaiguzhua |    36 |
| GuoJing      | Qiankundanuoyi  |    45 |
| YangGuo      | TaiJiquan       |    67 |
| YangGuo      | Yiyangzhi       |    71 |
| GuoJing      | TaiJiquan       |    90 |
| HuFei        | Wanliduxing     |    95 |
| DingDian     | Hamagong        |    99 |
+--------------+-----------------+-------+
14 rows in set (0.00 sec)

```

