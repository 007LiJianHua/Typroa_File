[toc]

# MySQL的安装部署

## 一、MySQL版本

* 5.6
* 5.7
* 8.0

## 二、MySQL安装方式

* rpm软件

* 源码编译

  * []: https://www.mysql.com/

    

## 三、MySQL安装

### 1、配置MySQL软件仓库

```bash
[root@localhost ~]# cat /etc/yum.repos.d/mysql.repo
[mysql57]
name=mysql57
baseurl=ftp://10.10.11.100/software/mysql/
enabled=1
gpgcheck=0
```

### 2、安装MySQL

```bash
[root@localhost ~]# yum install -y mysql-community-server 
```

### 3、启动mysql服务

```bash
[root@localhost ~]# systemctl start mysqld

#中间可能会停留一会，是mysql在初始化数据库


[root@localhost ~]# systemctl enable mysqld
[root@localhost ~]# 
[root@localhost ~]# netstat -antp | grep mysql
tcp6       0      0 :::3306                 :::*                    LISTEN      7620/mysqld         
[root@localhost ~]# 
[root@localhost ~]# ps -elf | grep mysql
1 S mysql      7620      1  0  80   0 - 296199 poll_s 10:42 ?       00:00:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

### 4、修改root用户临时密码

```bash
[root@localhost ~]# grep -i "password" /var/log/mysqld.log 
2021-05-28T02:42:52.290430Z 1 [Note] A temporary password is generated for root@localhost: cSNlQY/Qr1KU


如果使用短选项形式(-p)，该选项和 密码之间不能有空格
[root@localhost ~]# mysqladmin -u root -p password "WWW.1.com"
Enter password: 
```

### 5、链接mysql数据库

```bash
[root@localhost ~]# mysql -uroot -pWWW.1.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.17 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit

```

### 6、mysql相关文件

> * 错误日志
>   * /var/log/mysqld.log
> * 默认数据目录
>   * /var/lib/mysql
> * 配置文件
>   * /etc/my.cnf
>     * datadir=		数据目录
>     * socket=        套接字文件socket
>       * **本地客户端连接数据库时通过套接字文件连接，服务套接字, 程序使用套接字链接比使用`域名:端口`链接要快, 因为这样就不需要协议解析了**
>       * MySQL是C/S 架构，如果是本地客户端则通过socket连接，如果是其他的网络客户端则通过`域名:端口`
>     * log-error=       错误日志
>     * pid-file=         pid文件
>     * long_query_time=10      慢查询时间
>     * slow_query_log=ON    开启慢查询
>     * slow_query_log_file=slow.log     慢查询日志文件，默认在数据目录下
>     * server_id=10    mysql服务器ID号
>     * l**og_bin=/data/master        开启二进制日志文件，非常重要**

