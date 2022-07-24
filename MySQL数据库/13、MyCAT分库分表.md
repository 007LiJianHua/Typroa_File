[toc]

# MyCAT分库分表

## 一、分库分表简介

> * 在业务数据量过多的时候，而且数据不断持续增长的情况下，为了缓解单台服务器的压力，可以考虑使用分库或者分表方案

## 二、垂直切分---分库

### 1、思想

* 一个数据库由很多的表构成，每个表对应这不同的业务，垂直切分是按照业务将表进行手动分类，利用MyCAT中间件进行管理，将同一个数据库中的多个表分开放在多个数据库中

![https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/36AAAFDFE64E49ED8682B3AC8A135EBB/380B36E62D124BC98D5362B60C250EA3/26069](https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/36AAAFDFE64E49ED8682B3AC8A135EBB/380B36E62D124BC98D5362B60C250EA3/26069)

### 2、配置实现

* 环境描述

> 192.168.152.10	MySQL服务器
>
> 192.168.152.11	MySQL服务器
>
> 192.168.152.12	MyCAT中间键
>
> 192.168.152.13	测试客户端

* 分别在两台MySQL上创建测试的库、表
  * 相同的库，不同的表

* 在数据库db01上安装mariadb-server，创建数据库，account表,为mycat创建连接用户

```mysql
#第一个数据库
[root@db01 ~]# yum -y install mariadb-server
[root@db01 ~]# systemctl start mariadb
[root@db01 ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@db01 ~]# mysql -uroot

MariaDB [(none)]> create database game charset utf8;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use game;
Database changed
MariaDB [game]> create table Account(
    -> id int primary key not null auto_increment,
    -> name char(30));
Query OK, 0 rows affected (0.01 sec)

MariaDB [game]> insert into Account(name) values("ljh"),("yxj");
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [game]> select * from Account;
+----+------+
| id | name |
+----+------+
|  1 | ljh  |
|  2 | yxj  |

MariaDB [game]> grant all on game.*  to 'mycatuser'@'192.168.152.12' identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [game]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

* 在数据库db02上安装mariadb-server，创建数据库，area表,为mycat创建连接用户

```mysql
#第二个数据库
[root@db02 ~]# yum -y install mariadb-server
[root@db02 ~]# systemctl start mariadb
[root@db02 ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@db02 ~]# mysql -uroot 
MariaDB [(none)]> create database game charset utf8
    -> ;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use game;
Database changed
MariaDB [game]> create table area(
    -> aid int not null primary key auto_increment,
    -> aname char(40));
Query OK, 0 rows affected (0.00 sec)

MariaDB [game]> insert into area(aname)values("北方大区"),("南方大区");
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [game]> select * from area;
+-----+--------------+
| aid | aname        |
+-----+--------------+
|   1 | 北方大区     |
|   2 | 南方大区     |
+-----+--------------+

MariaDB [game]> grant all on game.*  to 'mycatuser'@'192.168.152.12' identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [game]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

* 安装MyCAT,配置java环境变量，修改配置文件

```bash
[root@mycat ~]# tar -xf jdk-8u91-linux-x64.tar.gz -C /usr/local/^C
[root@mycat ~]# tar -xf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz -C /usr/local/
[root@mycat ~]# tail -2 /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin

[root@mycat ~]# vim /usr/local/mycat/conf/schema.xml
  <schema name="game" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn01">
                <table name="Account" primaryKey="id" type="global" dataNode="dn01" />
                <table name="area" primaryKey="aid" type="global" dataNode="dn02" />
        </schema>

        <dataNode name="dn01" dataHost="localhost1" database="game" />
        <dataNode name="dn02" dataHost="localhost2" database="game" />

        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.152.10:3306" user="mycatuser"
                                   password="redhat">
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>


        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.152.11:3306" user="mycatuser"
                                   password="redhat">
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>

[root@mycat ~]# tail -6 /usr/local/mycat/conf/server.xml
	<user name="ljh">
		<property name="password">redhat</property>
		<property name="schemas">game</property>
	</user>

</mycat:server>
[root@mycat ~]# /usr/local/mycat/bin/mycat start
Starting Mycat-server...
Mycat-server is already running.
[root@mycat ~]# netstat -tunlp | grep java
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      12735/java          
tcp6       0      0 :::1984                 :::*                    LISTEN      12735/java          
tcp6       0      0 :::8066                 :::*                    LISTEN      12735/java          
tcp6       0      0 :::9066                 :::*                    LISTEN      12735/java          
tcp6       0      0 :::45934                :::*                    LISTEN      12735/java          
tcp6       0      0 :::45428                :::*                    LISTEN      12735/java          
[root@mycat ~]# 

```

* 客户端连接MyCAT进行测试

```bash
MySQL [(none)]> show databases;
+----------+
| DATABASE |
+----------+
| game     |
+----------+
1 row in set (0.01 sec)

MySQL [(none)]> use game;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [game]> show tables;
+----------------+
| Tables_in_game |
+----------------+
| account        |
| area           |
+----------------+
2 rows in set (0.00 sec)

MySQL [game]> insert into account(name)values("jjj"),("ddd");
ERROR 1146 (42S02): Table 'game.account' doesn't exist
MySQL [game]> insert into Account(name)values("jjj"),("ddd");
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MySQL [game]> select * from Account;
+----+------+
| id | name |
+----+------+
|  1 | ljh  |
|  2 | yxj  |
|  3 | jjj  |
|  4 | ddd  |
+----+------+
4 rows in set (0.00 sec)

MySQL [game]> select * from area;
+-----+--------------+
| aid | aname        |
+-----+--------------+
|   1 | 北方大区     |
|   2 | 南方大区     |
|   3 | jjj          |
|   4 | ddd          |
+-----+--------------+
4 rows in set (0.01 sec)

MySQL [game]> 

```

## 三、水平切分

### 1、思想

> * 水平切分相对于垂直拆分，水平拆分不是将表做分类，而是按照某个字段的某种规则来分散到多个库之中，每个表中包含一部分数据。简单来说，我们可以将数据的水平切分理解为是按照数据行的切分，就是将表中的某些行切分到一个数据库，而另外的某些行又切分到其他的数据库中

![https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/36AAAFDFE64E49ED8682B3AC8A135EBB/7041C4DF7E83451B976602680177245A/26073](https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/36AAAFDFE64E49ED8682B3AC8A135EBB/7041C4DF7E83451B976602680177245A/26073)

* 将上午写的schema.xml   server.xml还原

```bash
[root@mycat conf]# cp schema.xml.bak schema.xml
cp: overwrite ‘schema.xml’? yes
[root@mycat conf]# cp server.xml.bak server.xml
cp: overwrite ‘server.xml’? yes
```

* 先选择对应的规则和函数

  * rule

  * - 用于定义按表中的哪个字段进行切分

  - function

  - - 用于定义数据切分的规则，共分为2片，每片的长度为512，具体应该分为几片取决于后台有几个数据库
    - 即可以根据id对1024的取模结果，结果在0-512间的落在一台服务器上，512-1024间的落在另外的服务器

```bash
	<mycat:rule xmlns:mycat="http://io.mycat/">
        <tableRule name="splitHWrule">
                <rule>
                #下边的id是代表数据库的数量
                        <columns>id</columns>
                        <algorithm>func1</algorithm>
                </rule>
        </tableRule>

        <function name="func1" class="io.mycat.route.function.PartitionByLong">
                <property name="partitionCount">2</property>
                <property name="partitionLength">512</property>
        </function>
</mycat:rule>
```

* 最后修改这两个配置文件

```bash
[root@mycat conf]# vim schema.xml
       <schema name="phone" checkSQLschema="false" sqlMaxLimit="100">
                <!-- auto sharding by id (long) -->
                <table name="hw" dataNode="dn01,dn02" rule="splitHWrule" />
        </schema>

        <dataNode name="dn01" dataHost="localhost1" database="phone" />
        <dataNode name="dn02" dataHost="localhost2" database="phone" />

        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.152.10:3306" user="mycatuser"
                                   password="redhat">
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>
        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.152.11:3306" user="mycatuser"
                                   password="redhat">
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>
[root@mycat conf]# cat server.xml
	<user name="ljh">
		<property name="password">redhat</property>
		<property name="schemas">phone</property>
	</user>

```

* 在db01上创建相同数据库，相同的表，为mycat授权

```mysql
MariaDB [(none)]> create database phone charset utf8;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use phone;
Database changed
MariaDB [phone]> create table hw(
    -> id int primary key not null auto_increment,
    -> name char(30));
Query OK, 0 rows affected (0.00 sec)

MariaDB [phone]> grant all on phone.* to 'mycatuser'@'192.168.152.12' identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [phone]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [phone]> show tables;
+-----------------+
| Tables_in_phone |
+-----------------+
| hw              |
+-----------------+
```

* 在db02上创建相同数据库，相同的表，为mycat授权

```mysql
MariaDB [(none)]> create database phone charset utf8;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use database phone;
ERROR 1049 (42000): Unknown database 'database'
MariaDB [(none)]> use phone;
Database changed
MariaDB [phone]> create table hw(
    -> id int primary key not null auto_increment,
    -> name char(30));
Query OK, 0 rows affected (0.00 sec)

MariaDB [phone]> grant all on phone.* to 'mycatuser'@'192.168.152.12' identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [phone]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

* 用测试机连接创建不同的ID，来测试规则

```bash
[root@localhost ~]# mysql -uljh -predhat -h 192.168.152.12 -P8066
MySQL [(none)]> show databases;
+----------+
| DATABASE |
+----------+
| phone    |
+----------+
MySQL [phone]> insert into hw(id,name)values(10,"ljh"),(511,"yxj"),(513,"hzy"),(1021,"zyz");
Query OK, 4 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
MySQL [phone]> select * from hw;
+------+------+
| id   | name |
+------+------+
|   10 | ljh  |
|  511 | yxj  |
|  513 | hzy  |
| 1021 | zyz  |
+------+------+


#在db01上查看
MariaDB [phone]> select * from hw;
+-----+------+
| id  | name |
+-----+------+
|  10 | ljh  |
| 511 | yxj  |
+-----+------+
2 rows in set (0.00 sec)
#在db02上查看
MariaDB [phone]> select * from hw;
+------+------+
| id   | name |
+------+------+
|  513 | hzy  |
| 1021 | zyz  |
+------+------+
```

