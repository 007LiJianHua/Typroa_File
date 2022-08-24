[toc]

# SQL语句---用户权限管理操作

## 一、用户管理

### 1、用户名格式

* 用户名@客户端地址
* 客户端地址
  * IP地址：admin@192.168.1.1
  * 主机名：admin@node01.Linux.com
  * 网段：admin@192.168.152.%
  * 所有主机：admin@&
  * 本机：admin@localhost

### 2、存储用户的表 mysql.user

```mysql
mysql> select user,host,authentication_string from mysql.user;
+------------------+-----------+----------------------------------
| user             | host      | authentication_string            
+------------------+-----------+----------------------------------
| mysql.infoschema | localhost | $A$005$THISISACOMBINATIONOFINVALI
| mysql.session    | localhost | $A$005$THISISACOMBINATIONOFINVALI
| mysql.sys        | localhost | $A$005$THISISACOMBINATIONOFINVALI
| root             | localhost | $A$005$X2v~5?MxlhIHIe90XKQ/NyXBk5
+------------------+-----------+----------------------------------
4 rows in set (0.00 sec)

```

### 3、创建用户

* create user 用户名@客户端地址  identified by ‘密码’

#### 示例1：创建允许本机登录的用户，名称为admin

* 对用户操作结束后都要刷新
  * flush privileges;

```mysql
mysql> create user 'admin'@'localhost' identified by 'WWW.1.com';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| admin            | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
5 rows in set (0.00 sec)
```

#### 示例2：创建远程连接用户

```mysql
mysql> create user 'admin'@'192.168.152.11' identified by 'WWW.1.com';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------------------+----------------+
| user             | host           |
+------------------+----------------+
| admin            | 192.168.152.11 |
| admin            | localhost      |
| mysql.infoschema | localhost      |
| mysql.session    | localhost      |
| mysql.sys        | localhost      |
| root             | localhost      |
+------------------+----------------+
6 rows in set (0.00 sec)

mysql> alter user 'ljh'@'192.168.152.11' identified with mysql_native_password by 'WWW.1.com';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

> ​							mysql Authentication plugin ‘caching_sha2_password’ is not supported问题处理
>
> * 使用mysql8.0版本，登录失败，提示 Authentication plugin ‘caching_sha2_password’ is not supported。
>
> * 原因是在MySQL 8.0以后，默认的密码加密方式是caching_sha2_password而不是mysql_native_password。
>
> 解决方法：
>
> 1.登录mysql数据库 mysql -u root -p
>
> 2.更新身份认证方式 ALTER USER ‘你的用户名’@‘地址’ IDENTIFIED WITH mysql_native_password BY ‘你的用户密码’;



测试：开启11虚拟机，先安装mariadb客户端软件，在输入

```bash
[root@localhost ~]# yum -y install mariadb
[root@localhost ~]# mysql -uljh -pWWW.1.com -h 192.168.152.10
```

#### 示例3：创建允许windows连接的用户

```mysql
mysql> create user 'admin'@'192.168.140.1' identified with mysql_native_password by 'WWW.1.com';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### 4、删除用户

> drop user 用户名@客户端地址

```mysql
mysql> drop user 'admin'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### 5、修改用户密码

##### 1、方法一

* 因为我使用的时mysql8.0，所以在很多命令上都不太一样

```mysql
mysql> alter user 'ljh'@'192.168.152.11' identified by 'WWW.2.com';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

##### 2、方法二

```mysql
mysql> update mysql.user set authentication_string=PASSWORD("WWW.3.com") where user="admin" and host="192.168.140.11";
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### 6、重置root密码

#### 1、编辑my.cnf配置文件

```bash
[mysqld]
skip-grant-tables=1			//跳过授权表
# systemctl restart mysqld 
```

* 非常危险，修改结束后改回，并重启mysqld

#### 2、使用update 更新密码

```mysql
mysql> update mysql.user set authentication_string=PASSWORD("WWW.1.com") where user="root" and host="localhost";
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec
```

## 二、用户权限管理

### 1、查看用户权限

```mysql
mysql> show grants for 'admin'@'192.168.140.11';
+------------------------------------------------+
| Grants for admin@192.168.140.11                |
+------------------------------------------------+
| GRANT USAGE ON *.* TO 'admin'@'192.168.140.11' |
+------------------------------------------------+
1 row in set (0.00 sec)

USAGE：默认最小的权限 
```

### 2、用户授权

> grant 权限，权限 on 库名.表名 to 用户名  [identified by ‘密码’]

权限：create,drop,select,update,delete,insert

all

* 示例:为存在的用户授权

```mysql
mysql> grant select on jiaowu.tutors to 'ljh'@'192.168.152.11';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for 'ljh'@'192.168.152.11';
+-------------------------------------------------------------+
| Grants for ljh@192.168.152.11                               |
+-------------------------------------------------------------+
| GRANT USAGE ON *.* TO `ljh`@`192.168.152.11`                |
| GRANT SELECT ON `jiaowu`.`tutors` TO `ljh`@`192.168.152.11` |
+-------------------------------------------------------------+
2 rows in set (0.00 sec)

```

### 3、回收权限

```mysql
mysql> revoke select on jiaowu.tutors from 'ljh'@'192.168.152.11';

Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

```

