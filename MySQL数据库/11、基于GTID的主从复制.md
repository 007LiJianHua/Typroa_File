[toc]

# 基于GTID的主从复制

## 一、GTID介绍

> * MySQL5.6往后版本支持的新特性
> * GTID   Global Transaction Identified
>   * 全局事务ID
> * GTID构成
>   * server_uuid+事务ID
> * 在配置主从的时候不再需要自行去找二进制日志文件位置

## 二、配置基于GTID的主从复制

```bash
环境描述
	192.168.152.12   Master服务器
	192.168.152.13	 Slave服务器
```

1、在服务器安装MySQL8.0，导入jiaowu.sql

* 跳过配置mysql软件仓库，以安装好mysql8.0开始

```bash
[root@master ~]# systemctl start mysqld
[root@master ~]# systemctl enable mysqld
[root@master ~]# mysql -uroot -pWWW.1.com < jiaowu.sql
```

2、完全备份数据库，发送给从服务器

```bash
[root@master ~]# mysqldump -uroot -pWWW.1.com --lock-all-tables --master-data=2 --all-databases > /tmp/all.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.
[root@master ~]# rsync -av /tmp/all.sql root@192.168.152.13:/tmp/
```

3、修改配置文件，启用gtid特性

* gtid=on		启用gtid
* enfore_gtid_consistencytrue   强制gtid一致性

```bash
[root@master ~]# vim /etc/my.cnf
[mysqld]
server_id=10
log_bin=master
gtid_mode=on						//启用gtid
enforce_gtid_consistency=true		//强制gtid一致性
```



4、在主服务器创建从服务器登录用户，为用户授权，刷新

```mysql
mysql> create user 'ljh1'@'192.168.152.13' identified by "WWW.1.com";
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to "ljh1"@"192.168.152.13";
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

5、配置从服务器配置文件，重启

```bash
[root@slave ~]# vim /etc/my.cnf
[mysqld]
server_id=11
log_bin=slave
gtid_mode=on
enforce_gtid_consistency=true
```

6、在从服务器恢复主服务器内容，开机自启数据库，**连接主服务器获取密钥（8.0新特性）**

```bash
[root@slave ~]# mysql -uroot -pWWW.1.com < /tmp/data.sql 
[root@localhost ~]# mysql -u ljh1 -pWWW.1.com -h 192.168.152.12 --get-server-public-key
```

5、连接Master服务器

```mysql
mysql> CHANGE MASTER TO
    -> MASTER_HOST="192.168.152.12",
    -> MASTER_USER="ljh1",
    -> MASTER_PASSWORD="WWW.1.com",
    -> MASTER_AUTO_POSITION=1;
Query OK, 0 rows affected, 7 warnings (0.01 sec)

mysql> stop slave;
mysql> start slave;

```

6、测试即可

## 三、多元复制

### 1、概念

> * MySQL5.7以后支持的
> * 支持多个服务器向同一个从服务器复制数据
> * 通过**channel隧道**来标识不同的主服务器
> * **要将master.info和relay_log.info这两个文件以表的形式放到数据库中**

![img](https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/5ED5186DC4F54402809EF178FAA4B070/EAC8614292B24922909AEB1E8DB0F25A/20384)

### 2、将上午布置的GTID主从还原

### 3、在两个主服务器上为从服务器创建复制用户

```bash
mysql> create user 'ljh14'@'192.168.152.14' identified by "WWW.1.com";
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to 'ljh14'@'192.168.152.14';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

```bash
mysql> create user 'ljh15'@'192.168.152.14' identified by "WWW.1.com";
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to 'ljh15'@'192.168.152.14';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```



### 4、修改从服务器配置文件

```bash
[mysqld]
server_id=14
log_bin=master
gtid_mode=on
enforce_gtid_consistency=true
master_info_repository=TABLE
relay_log_info_repository=TABLE
```

### 5、连接主服务器获取密钥

```bash
[root@localhost ~]# mysql -uljh14 -pWWW.1.com -h 192.168.152.12 --get-server-public-key
[root@localhost ~]# mysql -uljh15 -pWWW.1.com -h 192.168.152.13 --get-server-public-key
```

### 6、连接两个主服务器

```bash
mysql> change master to
    -> master_host="192.168.152.12",
    -> master_user="ljh14",
    -> master_password="WWW.1.com",
    -> master_auto_position=1 for channel "to_master01";
Query OK, 0 rows affected, 7 warnings (0.01 sec)

mysql> change master to
    -> master_host="192.168.152.13",
    -> master_user="ljh14",
    -> master_password="WWW.1.com",
    -> master_auto_position=1 for channel "to_master02";
Query OK, 0 rows affected, 7 warnings (0.02 sec)

mysql> start slave for channel "to_master01";
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> start slave for channel "to_master02";
Query OK, 0 rows affected, 1 warning (0.00 sec)

```

### 