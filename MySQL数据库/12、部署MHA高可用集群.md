[toc]

# 	MHA高可用集群

## 一、MHA集群

### 1、作用

> * MHA    Master High Avavibility   主服务器高可用 
> * 应用场景:
>   * 一主多从的环境 
> * 作用：
>   * 提升主从复制中**主服务器的可用性**，尽量减少故障时间
>   * **周期性的检查主服务器的运行状态**，一但主服务器故障后，会在从服务器挑选一个延迟最低的服务器升级为主服务器，并将剩余的从服务器连接到新的主服务器上，继续维持主从复制的运行
>   * 同时MHA还会尝试检测旧服务器的二进制日志文件，将二进制文件在新主服务器运行，**保持数据的完整性**

![img](https://note.youdao.com/yws/public/resource/b7c8a66478a291b31e0f530ec74ae8c3/xmlnote/856E670810E747C081F8EBC46B6575F9/6CB2C01F4C4342759714C3EA53BA487E/20405)

### 2、MHA角色

* mha-manager	mha管理者
  * 建议将其部署独立的服务器，用于检测主服务器的运行状态 
* mha-node     mba客户端
  * 部署在所有的服务器上

### 3、开始部署MHA集群

##### 1、环境描述

> ​                192.168.152.11		MHA Manager + MHA node              
>
> ​                192.168.152.12		Master服务器 		mha_node              
>
> ​                192.168.152.13		Slave服务器		mha_node              
>
> ​                192.168.152.14		Slave服务器 		mha_node              

##### 2、关闭selinux，防火墙

##### 3、配置免密ssh

```bash
[root@localhost ~]# ssh-keygen -t rsa
[root@localhost ~]# mv .ssh/id_rsa.pub .ssh/authorized_keys
[root@localhost ~]# for i in 12 13 14
do rsync -av /root/.ssh/* root@192.168.152.$i:/root/.ssh/
done

[root@localhost ~]# for i in 11 12 13 14
> do
> ssh root@192.168.152.$i date
> done
Thu Mar  3 05:42:14 CST 2022
Thu Mar  3 05:43:14 CST 2022
Thu Mar  3 05:43:14 CST 2022
Thu Mar  3 05:43:14 CST 2022

```

##### 4、为所有主机修改名字，并添加所有主机名解析

```bash
[root@mha ~]# hostnamectl set-hostname mha
[root@master ~]# hostnamectl set-hostname master
[root@slave01 ~]# hostnamectl set-hostname slave01
[root@slave02 ~]# hostnamectl set-hostname slave02

[root@mha ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.152.11	mha
192.168.152,12	master
192.168.152.13	slave01
192.168.152.14	slave02

[root@mha ~]# for i in 12 13 14; do rsync -av /etc/hosts root@192.168.152.$i:/etc/hosts; done

```

##### 5、检测所有机器时间同步

```bash
[root@mha ~]# for i in 11 12 13 14
> do
> ssh root@192.168.152.$i ntpdate 120.25.115.20;date
> done
 2 Mar 21:54:13 ntpdate[1401]: step time server 120.25.115.20 offset -28742.418114 sec
Wed Mar  2 21:54:13 CST 2022
 2 Mar 21:54:20 ntpdate[1359]: step time server 120.25.115.20 offset -28802.214742 sec
Wed Mar  2 21:54:20 CST 2022
 2 Mar 21:54:26 ntpdate[1336]: step time server 120.25.115.20 offset -28802.727525 sec
Wed Mar  2 21:54:26 CST 2022
 2 Mar 21:54:33 ntpdate[1340]: step time server 120.25.115.20 offset -28802.168359 sec
Wed Mar  2 21:54:33 CST 2022

```

##### 6、在mha管理机器上安装管理软件

```bash
[root@mha ~]# wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@mha ~]# yum install perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes -y
#manager和node包在Github下载后传到虚拟机上
#网址：https://github.com/yoshinorim/mha4mysql-manager/wiki/Downloads
[root@mha ~]# rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mha4mysql-node-0.56-0.el6        ################################# [100%]
[root@mha ~]# rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mha4mysql-manager-0.56-0.el6     ################################# [100%]
#然后分别在每个服务器上安装node节点
```

##### 7、配置主从环境

* 在所有数据库启动二进制日志

```bash
[mysqld]
server_id=11
log_bin=master

[mysqld]
server_id=12
log_bin=slave01

[mysqld]
server_id=13
log_bin=slave02
```

* 创建允许所有主机连接的用户

```bash
MariaDB [(none)]> grant replication slave on *.* to 'repluser'@'192.168.183.12' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant replication slave on *.* to 'repluser'@'192.168.183.13' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant replication slave on *.* to 'repluser'@'192.168.183.14' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

* 配置从服务器连接主服务器
  * 自行查看主服务器的二进制日志事件位置

```bash
MariaDB [(none)]> change master to
    -> master_host="192.168.152.12",
    -> master_user="repluser",
    -> master_password="redhat",
    -> master_log_file="master.000001",
    -> master_log_pos=245;
Query OK, 0 rows affected (0.10 sec)

MariaDB [(none)]> start slave;
```

##### 8、创建MHA连接数据库的用户

* 注意：此时主从已经搭建完成只需要在主服务器上输入即可

```bash
MariaDB [(none)]> grant all on *.* to 'manager'@'192.168.152.11' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on *.* to 'manager'@'192.168.152.14' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on *.* to 'manager'@'192.168.152.12' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on *.* to 'manager'@'192.168.152.13' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

##### 9、创建MHA manager需要的工作目录

```bash
创建MHA manager需要的工作目录
[root@mha ~]# mkdir -p /masterha/app1			//保存日志
[root@mha ~]# mkdir /etc/masterha				//配置文件 
```

##### 10、编辑MHA的配置文件

```bash
#配置文件不存在，自己写一个
[root@mha ~]# cat /etc/masterha/app1.cnf

[server default]
manager_workdir=/masterha/app1
manager_log=/masterha/app1/manager.log
user=manager					//指定MHA管理用户，用于获取数据库服务器的运行状态、二进制日志信息等
password=redhat
ssh_user=root					//指定服务器间免密SSH的用户
repl_user=repluser				//指定连接主服务器复制数据时的远程复制用户
repl_password=redhat
ping_interval=1					//检测服务器运行状态的时间间隔，单位秒 

[server1]
hostname=192.168.152.12
port=3306
master_binlog_dir="/var/lib/mysql"			//指定对应数据库服务器二进制日志存储位置
candidate_master=1							//允许该服务器参选新主服务器 

[server2]
hostname=192.168.152.13
port=3306
master_binlog_dir="/var/lib/mysql"
candidate_master=1

[server3]
hostname=192.168.152.14
port=3306
master_binlog_dir="/var/lib/mysql"
candidate_master=1
```

##### 11、检测ssh免密是否成功

```bash
[root@mha ~]# masterha_check_ssh --conf=/etc/masterha/app1.cnf 

Mon Jan 25 15:16:04 2021 - [info] All SSH connection tests passed successfully.
```

##### 12、检测一主两从是否成功

```bash
[root@mha ~]# masterha_check_repl --conf=/etc/masterha/app1.cnf 

192.168.183.11(192.168.183.11:3306) (current master)
 +--192.168.183.12(192.168.183.12:3306)
 +--192.168.183.13(192.168.183.13:3306)

Mon Jan 25 15:18:08 2021 - [info] Checking replication health on 192.168.183.12..
Mon Jan 25 15:18:08 2021 - [info]  ok.
Mon Jan 25 15:18:08 2021 - [info] Checking replication health on 192.168.183.13..
Mon Jan 25 15:18:08 2021 - [info]  ok.
Mon Jan 25 15:18:08 2021 - [warning] master_ip_failover_script is not defined.
Mon Jan 25 15:18:08 2021 - [warning] shutdown_script is not defined.
Mon Jan 25 15:18:08 2021 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
```

##### 13、启动MHA

```bash
[root@mha ~]# masterha_manager --conf=/etc/masterha/app1.cnf 

[root@mha ~]# ps -elf | grep perl
0 S root       1469   1119  0  80   0 - 74284 hrtime 15:20 pts/0    00:00:00 perl /usr/bin/masterha_manager --conf=/etc/masterha/app1.cnf
```

##### 14、验证MHA的作用

* 停掉主服务器，查看是否切换主服务器

```bash
[root@master ~]# systemctl stop mariadb

在新主服务器查看从服务器信息
MariaDB [(none)]> show slave hosts;
+-----------+------+------+-----------+
| Server_id | Host | Port | Master_id |
+-----------+------+------+-----------+
|        14 |      | 3306 |        13 |
+-----------+------+------+-----------+
1 row in set (0.00 sec)

在从服务器上查看对应的新主服务器 
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.152.13
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: slave01.000003
          Read_Master_Log_Pos: 245
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 527
        Relay_Master_Log_File: slave01.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

##### 15、修复故障的主机，将其连接到新的主服务器

```bash
[root@slave_01 ~]# mysqldump -uroot --lock-all-tables --master-data=2 --all-databases > /tmp/data.sql
[root@slave_01 ~]# rsync -av /tmp/data.sql root@192.168.183.11:/tmp/

MariaDB [(none)]> change master to
    -> master_host="192.168.152.13",
    -> master_user="repluser",
    -> master_password="redhat",
    -> master_log_file="slave01.000003",
    -> master_log_pos=245;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.01 sec)
```

##### 16、再次启动MHA

```bash
[root@mha ~]# masterha_manager --conf=/etc/masterha/app1.cnf
Wed Mar  2 22:54:13 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Mar  2 22:54:13 2022 - [info] Reading application default configuration from /etc/masterha/app1.cnf..
Wed Mar  2 22:54:13 2022 - [info] Reading server configuration from /etc/masterha/app1.cnf..
```

