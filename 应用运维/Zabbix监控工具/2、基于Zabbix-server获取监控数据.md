[toc]

## 一、监控远程主机

### 1、在被监控端安装zabbix-agent

```bash
[root@node01 ~]# yum install -y zabbix-agent
```

* 修改配置文件

```bash
[root@node01 ~]# vim /etc/zabbix/zabbix_agentd.conf 

Server=192.168.140.10
ServerActive=192.168.140.10
Hostname=node01.linux.com
[root@node01 ~]# systemctl start zabbix-agent
[root@node01 ~]# systemctl enable zabbix-agent

[root@node01 ~]# netstat -antp | grep zabbix
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      6965/zabbix_agentd  
tcp6       0      0 :::10050                :::*                    LISTEN      6965/zabbix_agentd
```

### 2、在web界面添加被监控端

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/6C57ED5402CF4801B439C7548BDADCD5/14396](https://s2.loli.net/2022/03/30/qBIfktyPhRO4pio.png)

### 3、连接模板

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/D84D76A8C0F9478290265597A782296F/14399](https://s2.loli.net/2022/03/30/XrP2vGAQCtLxIk7.png)

### 4、创建图形

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/0FCD1733420049AF9EC993E74B85F4D1/14405](https://s2.loli.net/2022/03/30/A3DSWvgx2dG6cCK.png)

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/B43822652DB944B2BA9B5ECC356570B1/14408](https://s2.loli.net/2022/03/30/RdnaFv5ktwb2gGA.png)

### 5、创建聚合图形

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/E7FA137AF0464F9194BB29F0828BB41F/14413](https://s2.loli.net/2022/03/30/P1qdrW9sN7cyLJE.png)

![](https://s2.loli.net/2022/03/30/P1qdrW9sN7cyLJE.png)

## 二、基于zabbix-agent获取监控数据的三种方式

* 连接模板
  * 适用于监控系统机器自身（CPU、内存、网卡、硬盘）的监控
* 利用zabbix-agent自带的key进行监控
* 自定义key进行监控

### 1、利用zabbix-agent自带的key进行监控

官方文档地址：https://www.zabbix.com/documentation/4.0/zh/manual/config/items/itemtypes/zabbix_agent

**示例一：监控ens33网卡的流入流量**

![](https://s2.loli.net/2022/03/30/28kexbpPruHRF1S.png)

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/9B3D40A3A34B4621AA06F9473C995226/14433](https://s2.loli.net/2022/03/30/Ut8NbuwCoOWTAk1.png)

### 2、常用键值

* `监控网卡流量`

- - net.if.in[if,]

  - - ⇒ net.if.in[eth0,errors]
    - ⇒ net.if.in[eth0]

  - net.if.total[if,]

  - net.if.out[if,]

* `监控TCP的服务状态`

- - net.tcp.listen[port]

  - - ⇒ net.tcp.listen[80]

  - net.tcp.port[,port]

  - - ⇒ net.tcp.port[,80]

* `监控UDP的服务状态`

- - net.udp.listen[port]

  - - => net.udp.listen[123]

* `进程数量`

- - proc.num[,,,]

  - - ⇒ proc.num[]
    - ⇒ proc.num[redis_server,,,]
    - ⇒ proc.num[nginx,www]
    - => proc.num[,,zomb]

* `进程所消耗的内存`

- - proc.mem[,,,,]

  - - => proc.mem[httpd]

* `进程所消耗的CPU使用率`

- - proc.cpu.util[,,,,,]

  - - => proc.cpu.util[httpd]

* `磁盘空间`

- - vfs.fs.size[fs,]

  - - vfs.fs.size[/webdata, free]
    - vfs.fs.size[/var/lib/mysql, pfree]

* `内存大小`

- - vm.memory.size[]

  - - vm.memory.size[free]
    - vm.memory.size[buffers]
    - vm.memory.size[cached]

* `文件内容变化`

- - vfs.file.cksum[file]

  - - ⇒ vfs.file.cksum[/etc/passwd]



## 三、自定义key

* 在被监控端定义
* 配置语法

- - UnsafeUserParameters=1

- - UserParameter=<key>,<shell command>

<font color="red">注意：确保zabbix用户对命令拥有读、执行的权限</font>

### 1、监控MySQL的用户数

#### 1）在被监控端定义key

```bash
[root@node01 ~]# vim /etc/zabbix/zabbix_agentd.conf 

UnsafeUserParameters=1
UserParameter=mysql.user.number,mysql -uroot -e "select count(*) from mysql.user" | sed '1d'

[root@node01 ~]# systemctl restart zabbix-agent
```

#### 2）在zabbix-server测试获取数据

* 命令不存在就安装  yum -y install zabbix-get

```bash
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k mysql.user.number
6
```

#### 3）在web界面创建监控项

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/D48AA1108B9647D980CED678A86AA625/14506](https://s2.loli.net/2022/03/30/YQ1Bo7PEmF68fsh.png)

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/31581347C3664DB8A3973C7DA8ED822F/14510](https://s2.loli.net/2022/03/30/OZAL687wxCH5GDU.png)

### 2、监控MySQL的连接数

```bash
UserParameter=mysql.conn.number,mysql -uroot -e "show processlist" | sed '1d' | wc -l
```

### 3、定义有参数的键值

* \$1是在zabbix的内置变量，$2
  * 依次代表键值中的第一个参数，第二个参数
* $$转义

```bash
UserParameter=memory.size[*], awk '/^$1:/{print $$2}' /proc/meminfo
```

* 重启之后在zabbix-server客户端测试

```bash
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k memory.size[MemTotal]
995924
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k memory.size[Buffers]
2108
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k memory.size[MemFree]
409852
```

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/590F6813F4C446F79725837693A668F7/14521](https://s2.loli.net/2022/03/30/m8ot14k5v2dcx9n.png)

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/30589DD7E5584814AA553DD378364641/14523](https://s2.loli.net/2022/03/30/ljN91TcY4MhPCAF.png)

### 4、监控MySQL复制线程状态

```bash
[root@node01 ~]# vim /etc/zabbix/zabbix_agentd.conf
UserParameter=mysql_io_stat,bash /opt/mysql_io_stat.sh
UserParameter=mysql_sql_stat,bash /opt/mysql_sql_stat.sh
[root@node01 opt]# cat mysql_io_stat.sh 
#!/bin/bash

#
mysql_io_stat=$(mysql -uroot -e "show slave status\G" | awk '/Slave_IO_Running/{print $2}')

if [ "$mysql_io_stat" == "Yes" ];then
	echo 1
else
	echo 0
fi 
[root@node01 opt]# cat mysql_sql_stat.sh 
#!/bin/bash

#
mysql_sql_stat=$(mysql -uroot -e "show slave status\G" | awk '/Slave_SQL_Running/{print $2}')

if [ "$mysql_sql_stat" == "Yes" ];then
	echo 1
else
	echo 0
fi 
#还可以将这两个脚本合起来
[root@node01 opt]# cat mysql_slave_stat.sh 
#!/bin/bash

#
mysql_slave_stat=$(mysql -uroot -e "show slave status\G" | awk "/$1:/{print \$2}")

if [ "$mysql_slave_stat" == "Yes" ];then
	echo 1
else
	echo 0
fi 
[root@node01 opt]# vim /etc/zabbix/zabbix_agentd.conf 
UserParameter=mysql_stat[*],bash /opt/mysql_slave_stat.sh $1

```

* 在zabbix-server测试

```bash
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k mysql.slave[Slave_IO_Running]
1
[root@zabbix_server ~]# zabbix_get -s 192.168.140.11 -k mysql.slave[Slave_SQL_Running]
1
```

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/851630FC2A0546A8999317CE8E8F5137/357F789FB7A8491C84344D6FD7890859/14530](https://s2.loli.net/2022/03/30/jcOET5mDqpSthvi.png)

## 四、MySQL监控指标参考

> 1、服务进程状态 	3306
>
> 2、数据目录剩余容量、二进制目录容量 
>
> 3、用户数、连接数
>
> 4、IO线程、SQL线程、复制延迟 
>
> 5、事务数量 
>
> 6、慢查询数量
>
> 7、运行时长
>
> 8、Innodb buffer缓冲
>
> 9、Innodb Cache

```bash
[root@node01 ~]# mysql -uroot -e "status"
--------------
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:		1957
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server:			MariaDB
Server version:		5.5.68-MariaDB MariaDB Server
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	latin1
Db     characterset:	latin1
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			39 min 31 sec

Threads: 1  Questions: 14319  Slow queries: 0  Opens: 22  Flush tables: 2  Open tables: 48  Queries per second avg: 6.039
[root@node01 ~]# mysqladmin extended-status

  Com_delete：数据库执行delete的次数
  Com_insert：数据库执行insert的次数
  Com_select：数据库执行select的次数
  Com_update：数据库执行update的次数

  Com_commit: 提交的事务的次数

  Threads_created 创建的线程数
  Threads_running 正在处理请求的线程数

  Subquery_cache_hit //子查询的缓存命中数
  Subquery_cache_miss //子查询的缓存丢失数

  Qcache_hits //所有查询的缓存命中

  Cpu_time //数据库消耗的CPU资源

  Bytes_received //网卡接收的数据量
  Bytes_sent //网卡发送的数据量

  Connections //处理的连接数
```

## 五、nginx、redis监控指标