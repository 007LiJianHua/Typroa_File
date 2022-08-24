[toc]

# MySQL日志管理

## 一、日志类型

* 错误日志
  * /var/log/mysqld.log
* 慢查询日志
* 二进制日志

## 二、慢查询日志

### 1、作用

* 记录慢查询

### 2、慢查询定义

> * 执行时常超过long_query_time指定的值的查询
> * 默认值  10

### 3、启动慢查询日志

```bash
[mysqld]
long_query_time=10
slow_query_log=ON
slow_query_log_file=slow.log   #默认放到数据目录下
```

回到mysql服务器测试

```bash
mysql> select sleep(15);
+-----------+
| sleep(15) |
+-----------+
|         0 |
+-----------+
```

在数据目录下查看慢查询日志，可以看到超过10s的命令

```bash
[root@localhost data]# cat slow.log 
/usr/sbin/mysqld, Version: 5.7.17-log (MySQL Community Server (GPL)). started with:
Tcp port: 0  Unix socket: /data/mysql.sock
Time                 Id Command    Argument
# Time: 2021-06-02T05:50:44.216487Z
# User@Host: root[root] @ localhost []  Id:     4
# Query_time: 15.000596  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1622613044;
select sleep(15);
```

### 三、二进制日志 

### 1、作用

> * 存储MySQL服务器产生的所有写操作
> * 实际应用
>   * 备份、恢复
>   * 主从复制

### 2、启用二进制日志

修改/etc/my.cnf

```bash
[mysqld]
server_id=10
log_bin=master
```

> * server_id：为MySQL服务器分配唯一的ID，取值范围1--2^32
> * log_bin：启用二进制日志，默认存放到数据目录

* 二进制日志命名格式：

master.xxxxx

* master.index
  * 文本文件，记录当前服务器都有哪些二进制日志文件

```bash
[root@localhost ~]# cat /data/master.index 
./master.000001
./master.000002
./master.000003
[root@localhost ~]# file /data/master.000001 
/data/master.000001: MySQL replication log
```

### 3、二进制日志格式

> * 起始时间
> * 事件
> * 终止时间
> * 起始位置position
> * 终止位置position
>   * 下次一个事件的起始位置，就是上一个时间的结束位置

### 4、查看二进制日志

* 查看二进制日志文件内容   mysqlbinlog

#### (1)、查看所有内容

```bash
[root@localhost ~]# mysqlbinlog /test/master.000002 
```

#### (2)、按时间查找二进制日志

```bash
--start-datetime=
--stop-datetime=

[root@localhost ~]# mysqlbinlog --start-datetime="2021-01-20 11:01:22"  --stop-datetime="2021-01-20 11:12:18" /test/master.000002
```

#### (3)、按位置查找二进制日志

```bash
--start-position=
--stop-position=

[root@localhost ~]# mysqlbinlog --start-position=219 --stop-position=313 /test/master.000002 
```

#### (4)、在MySQL中查看当前正在使用的二进制日志、最后一个事件的结束点

```mysql
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| master.000004 |     6458 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
```

#### (5)、查看所有的二进制日志、大小

```mysql
mysql> show binary logs;
```

#### (6)、查看二进制日志文件中的事件

```mysql
mysql> show binlog events in "master.000003";
+---------------+-----+----------------+-----------+-------------+---------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
+---------------+-----+----------------+-----------+-------------+---------------------------------------+
| master.000003 |   4 | Format_desc    |        10 |         123 | Server ver: 5.7.17-log, Binlog ver: 4 |
| master.000003 | 123 | Previous_gtids |        10 |         154 |                                       |
| master.000003 | 154 | Anonymous_Gtid |        10 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
| master.000003 | 219 | Query          |        10 |         307 | create database AA                    |
| master.000003 | 307 | Anonymous_Gtid |        10 |         372 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
| master.000003 | 372 | Query          |        10 |         460 | create database BB                    |
| master.000003 | 460 | Stop           |        10 |         483 |                                       |
+---------------+-----+----------------+-----------+-------------+---------------------------------------+
```

#### (7)、二进制日志滚动

* 重启mysql服务
* 人为操作
  * mysql> flush logs;
* 单个文件的大小超过1Gb

> 以上三种操作都会触发mysql日志回滚，重新生成的一个二进制文件，并使用