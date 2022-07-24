[toc]

# MySQL主从复制、读写分离

## 二、主从复制

### 1、工作原理

> **Master服务器将写操作保存到二进制文件中，通过网络将事件发送给slave服务器，slave服务器产生I/O thread线程接受二进制日志我事件，并将该事件写入到本地的中继日志relay log ，同时产生SQL thread 线程从中继日志中读取执行操作，确保数据同步**

![https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/BA8BCDC37A6043908AE1E2FCAF90A24D/B75497E4FDB043A484FB45A7D3252A48/10563](https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/BA8BCDC37A6043908AE1E2FCAF90A24D/B75497E4FDB043A484FB45A7D3252A48/10563)

> * 核心关注点
>   * 数据同步
>   * 复制延迟的时间
> * 实现流程
>   * 两个MySQL服务器配置server_id
>   * master服务器启动二进制日志
>   * master服务器要存在允许从服务器远程连接的用户

### 2、作用

* 避免数据库单点故障
* 便于冷备份
* 读写分离
  * 实现方式
    * 开发代码
    * 数据库中间件
      * mysql-proxy
      * mycat

### 3、常见主从复制架构

* 一主多从
  * 实现读操作的负载均衡
* 一主一从
* 双主复制

### 4、主从复制工作方式

* 异步
  * 默认
* 同步
* 半同步
  * 借助插件goole公司semi



## 二、案例：AB复制

1、环境描述

> 192.168.152.10		master服务器
>
> 192.168.152.11		slave服务器

2、关闭SELinux，防火墙，时间同步

3、在Master服务器上安装MySQL，更改二进制数据目录，开启二进制日志，导入jiaowu数据库

```bash
[root@localhost ~]# yum install -y mariadb-server 
[root@localhost ~]# systemctl start mariadb
[root@localhost ~]# systemctl enable mariadb
[root@localhost ~]# mysql -uroot < jiaowu.sql 

[root@localhost ~]# mkdir /data/
[root@localhost ~]# chown mysql.mysql /data/
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
server_id=10
log_bin=/data/master
[root@localhost ~]# systemctl restart mariadb
[root@localhost ~]# mysql -uroot < jiaowu.sql 
```

4、在master服务器上创建复制的远程用户，授权为从服务器

```mysql
MariaDB [(none)]> create user "ljh"@"192.168.152.12" identified by "WWW.1.com";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant replication slave on *.* to "ljh"@"192.168.152.13";

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

5、将master服务器的数据库文件生成.sql文件发送给从服务器

```bash
[root@mysql-master ~]# mysqldump -uroot --lock-all-tables --master-data=2 --all-databases > /tmp/all.sql
[root@mysql-master ~]# rsync -av /tmp/data.sql root@192.168.140.11:/tmp/
```

**6、在slave安装数据库，导入.sql文件,修改配置文件设置从服务器为只读**

* 如果从服务器只读情况下，root用户 还在从服务器写入了数据，当出现主服务器写入相同数据的时候，SQL线程会报错，
* 解决办法：
  * 先停止slave，删掉相同的数据，再开启slave

```bash
[root@mysql-slave ~]# yum -y install mariadb-server
[root@mysql-slave ~]# vim /etc/my.cnf
[mysqld]
server_id=11
read_only=1
[root@mysql-slave ~]# systemctl start mariadb
[root@mysql-slave ~]# systemctl enable mariadb
 [root@mysql-slave ~]# mysql -u root < /tmp/all.sql 

```

7、在slave服务器指定连接主服务器

```mysql
MariaDB [(none)]> change master to
    -> master_host="192.168.152.12",
    -> master_user="ljh",
    -> master_password="WWW.1.com",
    -> master_log_file="master.000001",
    -> master_log_pos=245;
Query OK, 0 rows affected (0.01 sec)

```

8、启动slave服务器的复制线程,并查看

```mysql
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.140.10
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000003
          Read_Master_Log_Pos: 5249			# 读取到主服务器哪个事件
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 526
        Relay_Master_Log_File: master.000003	
             Slave_IO_Running: Yes				# IO线程状态 
            Slave_SQL_Running: Yes				# SQL线程状态 
.........................................
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 5249			# 同步到主服务器的哪个事件
              Relay_Log_Space: 822
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
.........................................
        Seconds_Behind_Master: 0				# 复制延迟，单位为秒
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 10
```

9、测试主从同步与从服务器宕机自动同步

* **复制线程会随着数据库服务启动而自动启动**

* 在从库的数据目录下，会自动生成两个文件

* - master.info		记录连接主服务器的信息
  - relay-log.info     记录主服务器二进制日志，本地中继日志信息

## 三、主从切换 failover

> * 出错场景
>   * 主从已经搭建好，主服务器宕机，从服务器正常运行
>   * 把主从调换，重新配置

*  从服务器先停止复制线程，在将从服务器提升为新主

  * ```mysql
    > stop slave;
    
    > reset slave all;
    ```

* 其他操作同上

```mysql
> stop slave;
> reset slave all;
```

## 四、案例：基于MyCAT实现读写分离

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEe195dc899699112ee24816c59fb14728/96](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEe195dc899699112ee24816c59fb14728/96)

* 添加mycat主机名解析

```bash
[root@localhost ~]# hostnamectl set-hostname mycat.linux.com
[root@mycat ~]# vim /etc/hosts
192.168.152.14   mycat.linux.com
```

* 获取mycat软件包和jdk，并安装

```bash
[root@mycat ~]# lftp 10.10.21.100
lftp 10.10.21.100:/software/mycat> mget jdk-8u91-linux-x64.tar.gz Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz 
197030222 bytes transferred in 32 seconds (5.89M/s)                                                  
Total 2 files transferred
lftp 10.10.21.100:/software/mycat> exit

[root@mycat ~]# tar -xf jdk-8u91-linux-x64.tar.gz -C /usr/local/
[root@mycat ~]# tar -xf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz -C /usr/local/
[root@mycat ~]# sed -ri '$a \export JAVA_HOME=/usr/local/jdk1.8.0_91' /etc/profile
[root@mycat ~]# sed -ri '$a \export PATH=$PATH:$JAVA_HOME/bin' /etc/profile
[root@mycat ~]# source /etc/profile
[root@mycat ~]# cd /usr/local/mycat/
```

* 备份需要修改的文件

```bash
[root@mycat conf]# cp schema.xml schema.xml.bak
[root@mycat conf]# cp server.xml server.xml.bak
[root@mycat conf]# vim schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

        <schema name="jiaowu" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
        </schema>
        <dataNode name="dn1" dataHost="dh1" database="jiaowu" />
        <dataHost name="dh1" maxCon="1000" minCon="10" balance="1"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
 #谁在上边谁就是写，下边的就是读
                <writeHost host="hostM1" url="192.168.152.13:3306" user="mycat"
                                   password="redhat">
                        <!-- can have multi read hosts -->
                </writeHost>
                <writeHost host="hostS1" url="192.168.152.12:3316" user="mycat"
                                   password="redhat" />
                <!-- <writeHost host="hostM2" url="localhost:3316" user="root" password="123456"/> -->
        </dataHost>
</mycat:schema>
~        
[root@mycat conf]# vim server.xml
<mycat:server xmlns:mycat="http://io.mycat/">
.............
        <user name="ljh">
                <property name="password">redhat</property>
                <property name="schemas">jiaowu</property>
        </user>

</mycat:server>

```

* 为master和slave授权

```mysql
MariaDB [(none)]> create user 'mycat'@'192.168.152.14' identified 
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on jiaowu.* to 'mycat'@'192.168.152.14
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit

```



* 启动查看开机自启
  * 8066：用于接收客户端连接
  * 9066：mycat后台管理端口

```bash
[root@mycat conf]# /usr/local/mycat/bin/mycat start
Starting Mycat-server...
[root@mycat conf]# netstat -tunlp | grep java
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      1577/java           
tcp6       0      0 :::45216                :::*                    LISTEN      1577/java           
tcp6       0      0 :::1984                 :::*                    LISTEN      1577/java           
tcp6       0      0 :::8066                 :::*                    LISTEN      1577/java           
tcp6       0      0 :::9066                 :::*                    LISTEN      1577/java           
tcp6       0      0 :::33359                :::*                    LISTEN      1577/java 
[root@mycat ~]# vim /etc/rc.d/rc.local 
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
/usr/local/mycat/bin/mycat start

[root@mycat ~]# chmod a+x /etc/rc.d/rc.local
```

* 连接mycat，测试读写分离

```bash
[root@localhost ~]# mysql -u martin -p -h 192.168.140.12 -P 8066
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.29-mycat-1.6-RELEASE-20161028204710 MyCat Server (OpenCloundDB)
```

## 五、双主复制

### 1、面试出现问题

> * 开启主从复制或者双主复制时，假设id为主键，则可能会出现主服务器写入了id=5，此时从服务器也应该同步创建id=5，因为延迟较高，没有同步成功，又自己在从服务器写入了一条数据id=5，等到同步成功的时候，则会出现IO线程挂掉
>
> * 解决办法：
>
>   * 主服务器：auto_increment_offset=1     初始值
>
>     ​          	  auto_increment_increment=2    步长
>
>     这样在主服务器的主键上就是1，3，5，而从服务器就是2，4，6
>
>   * 从服务器：auto_increment_offset=2     初始值
>   
>     ​          	  auto_increment_increment=2    步长

### 2、在主从的基础上搭建双主复制

* 原来主服务器，改成新从服务器
  * 利用同步，在新的从服务器上为自己创建用户并授权
    * 之前授权都是在主服务器授权

```bash
ariaDB [(none)]> create user 'ljh2'@'192.168.152.13' identified by "WWW.1.com";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant replication slave on *.* to 'ljh2'@'192.168.152.13';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> change master to 
    -> master_host="192.168.152.12",
    -> master_user="ljh2",
    -> master_password="WWW.1.com",
    -> master_log_file="master.000005",
    -> master_log_pos=245;
Query OK, 0 rows affected (0.01 sec)

```

* 原来的从服务器，改成主服务器

```bash
0000000
```

* 测试





```bash
iptables -t nat -A PREROUTING -d 192.168.152.16 -p tcp --dport 80 -j DNAT --to-destination 10.2.90.100:80
```



