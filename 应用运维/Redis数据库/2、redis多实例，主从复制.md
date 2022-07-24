[toc]

## 一、redis多实例

### 1、环境描述

```bash
redis实例:   /opt/6380	数据目录: /opt/6380/data		日志: /var/log/redis_6380.log
redis实例:   /opt/6381	数据目录: /opt/6381/data		日志: /var/log/redis_6381.log
```

### 2、规划实例目录

```bash
[root@localhost ~]# mkdir /opt/638{0,1}/{conf,data} -p
[root@localhost ~]# tree /opt/
/opt/
├── 6380
│   ├── conf
│   └── data
└── 6381
    ├── conf
    └── data
```

### 3、复制配置文件

```bash
[root@localhost ~]# cp redis-5.0.12/redis.conf /opt/6380/conf/
[root@localhost ~]# cp redis-5.0.12/redis.conf /opt/6381/conf/
```

### 4、修改实例配置文件

```bash
[root@localhost ~]# vim /opt/6380/conf/redis.conf 
#redis绑定的IP地址
bind 192.168.140.10
#redis的实例端口
port 6380
#开启后台启动，
daemonize yes
#redis的PID文件
pidfile /var/run/redis_6380.pid
#redis的日志级别
loglevel warning
#启用redis的手动持久化存储
rdbchecksum yes
#redis持久化存储的文件名
dbfilename dump_6380.rdb
#redis持久化存储的文件路径
dir /opt/6380/data
#启动redis的aof日志文件
appendonly yes
#aof日志文件的名字，路径默认是上边的dir路径
appendfilename "appendonly_6380.aof"
```

* 另一个实例配置文件就直接复制修改了

```bash
[root@localhost ~]# cp /opt/6380/conf/redis.conf /opt/6381/conf/redis.conf 
[root@localhost ~]# sed -ri 's/6380/6381/g' /opt/6381/conf/redis.conf
```

### 5、启动多实例

* 不同的多实例启动跟的是不同的配置文件

```bash
[root@localhost ~]# redis-server /opt/6380/conf/redis.conf 
[root@localhost ~]# redis-server /opt/6381/conf/redis.conf 

[root@localhost ~]# netstat -antp | grep redis
tcp        0      0 192.168.140.10:6380     0.0.0.0:*               LISTEN      33830/redis-server  
tcp        0      0 192.168.140.10:6381     0.0.0.0:*               LISTEN      33838/redis-server  

[root@localhost ~]# ps -elf | grep redis
5 S root      33830      1  0  80   0 - 38499 ep_pol 15:09 ?        00:00:00 redis-server 192.168.140.10:6380
5 S root      33838      1  0  80   0 - 38499 ep_pol 15:09 
```

### 6、测试数据读写

```bash
[root@localhost ~]# redis-cli -h 192.168.140.10 -p 6380
192.168.140.10:6380> 
192.168.140.10:6380> set a 10
OK
192.168.140.10:6380> exit

[root@localhost ~]# redis-cli -h 192.168.140.10 -p 6381
192.168.140.10:6381> 
192.168.140.10:6381> set b 20
OK
192.168.140.10:6381> exit
```

## 二、主从复制

* 作用：避免单点故障、提高可用性

### 1、核心要素

* 基于异步进行数据复制
  * 同步：客户在进行写操作的时候必须等到从服务器复制完毕，才会给客户返回确认
  * 异步：客户在进行写操作的时候，主服务器收到就给客户端一个响应确认，之后再进行主从复制
* 支持一主多从
* 复制数据时不影响前端业务
* redis2.0版本开始，从服务器为只读模式

### 2、配置主从复制

```bash
#随便找一台机器修改他的配置文件，当作从服务器
vim /opt/6381/conf/redis.conf
#修改
replicaof 192.168.140.10  6380

[root@localhost ~]# redis-cli -h 192.168.140.10 -p 6381 shutdown
[root@localhost ~]# redis-server /opt/6381/conf/redis.conf
```

### 3、查看

```bash
[root@localhost ~]# redis-cli -h 192.168.140.10 -p 6480 
[root@localhost ~]# INFO replication
role:master
connected_slaves:1
slave0:ip=192.168.140.10,port=6381,state=online,offset=296,lag=1
master_replid:0e72f90eafdbf8cb5f9fafea49199181d2d2bb90
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:296
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:296

[root@localhost ~]# redis-cli -h 192.168.140.10 -p 6481
192.168.140.10:6381> info replication
# Replication
role:slave
master_host:192.168.140.10
master_port:6380
master_link_status:up
master_last_io_seconds_ago:4
```

## 三、主从切换

### 1、将从服务器提升为新主

```bash
192.168.140.10:6381> slaveof no one
OK
192.168.140.10:6381> info replication
# Replication
role:master
connected_slaves:0
master_replid:c478b8a4d12e8da7e9cffb89fd75c2d3eab5f5d5
master_replid2:0e72f90eafdbf8cb5f9fafea49199181d2d2bb90
master_repl_offset:1229
second_repl_offset:1230
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1229
```

### 2、将从服务器的主从配置删除

```bash
#eplicaof 192.168.140.10  6380
```

### 3、重启、查看



