[toc]

## 一、作用

* redis 3.X 版本后开始支持
  * 提高缓存数据的读写速度
  * 提高缓存数据的可用性、可靠性

## 二、工作原理

> * redis集群使用哈希槽slot进行数据切片
>
> * Redis集群有16384个哈希槽, 每个key通过CRC16校验后对16384取模来决定放置哪个槽, 集群的每个节点负责一部分hash槽
>
> * 举个例子,比如当前集群有3个节点,
>
>   ​	那么:
>
>   ​		节点 A 包含 0 到 5500号哈希槽.
>
>   ​		节点 B 包含5501 到 11000 号哈希槽.
>
>   ​		节点 C 包含11001 到 16384号哈希槽.
>
>   数据究竟存放到哪个槽上？
>
>   数据做hash运算除以16384取余

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE270de4ef730e68988a4acf1b2b2fc00d/114](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE270de4ef730e68988a4acf1b2b2fc00d/114)



![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE4a4dcd1cc18a012dd33bb1d5ba6bac87/116](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE4a4dcd1cc18a012dd33bb1d5ba6bac87/116)

## 三、部署redis分片集群

### 1、环境描述

```bash
192.168.140.10     3个集群模式的实例     7001  7002  7003
192.168.140.11     3个集群模式的实例     7004  7005  7006
```

### 2、关闭防火墙、SELinux、打开时间同步

### 3、两台服务器分别装上redis

### 4、在第一台服务器创建三个redis实例

```bash
[root@localhost ~]# mkdir /opt/700{1,2,3}/{conf,data} -p
[root@localhost ~]# 
[root@localhost ~]# cp redis-5.0.12/redis.conf /opt/7001/conf/
```

### 5、编辑实例的配置文件

```bash
# [root@localhost ~]# vim /opt/7001/conf/redis.conf 
bind 192.168.140.10
port 7001
daemonize yes
loglevel warning
logfile "/var/log/redis_7001.log"
dbfilename dump_7001.rdb
dir /app/7001/data
appendonly yes
appendfilename "appendonly_7001.aof"
#启动集群架构
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 15000
```

* 依次将修改的配置文件复制给其他的两个实例并修改

```bash
[root@localhost ~]# cp /opt/7001/conf/redis.conf /opt/7002/conf/
[root@localhost ~]# cp /opt/7001/conf/redis.conf /opt/7003/conf/
[root@localhost ~]# 
[root@localhost ~]# sed -ri 's|7001|7002|' /opt/7002/conf/redis.conf 
[root@localhost ~]# sed -ri 's|7001|7003|' /opt/7003/conf/redis.conf 

```

### 6、第二台服务器参考上述配置

### 7、启动redis实例

```bash
[root@localhost ~]# redis-server /opt/7001/conf/redis.conf 
[root@localhost ~]# redis-server /opt/7002/conf/redis.conf 
[root@localhost ~]# redis-server /opt/7003/conf/redis.conf 
```

* 看端口

```bash
[root@localhost ~]# ps -elf | grep redis
5 S root       6895      1  0  80   0 - 38499 ep_pol 09:47 ?        00:00:00 redis-server 192.168.140.10:7001 [cluster]
5 S root       6900      1  0  80   0 - 38499 ep_pol 09:47 ?        00:00:00 redis-server 192.168.140.10:7002 [cluster]
5 S root       6905      1  0  80   0 - 38499 ep_pol 09:47 ?        00:00:00 redis-server 192.168.140.10:7003 [cluster]
0 S root       6910   6776  0  80   0 - 28203 pipe_w 09:47 pts/0    00:00:00 grep --color=auto redis
```

* 来看进程

```bash
[root@localhost ~]# netstat -antp | grep redis
tcp        0      0 192.168.140.10:17001    0.0.0.0:*               LISTEN      6895/redis-server 1 
tcp        0      0 192.168.140.10:17002    0.0.0.0:*               LISTEN      6900/redis-server 1 
tcp        0      0 192.168.140.10:17003    0.0.0.0:*               LISTEN      6905/redis-server 1 
tcp        0      0 192.168.140.10:7001     0.0.0.0:*               LISTEN      6895/redis-server 1 
tcp        0      0 192.168.140.10:7002     0.0.0.0:*               LISTEN      6900/redis-server 1 
tcp        0      0 192.168.140.10:7003     0.0.0.0:*               LISTEN      6905/redis-server 1 

```

* 说明
  * 启动集群功能的redis实例时，该实例会自动产生10000+的端口，用于集群内部相互通信 

### 8、在第二台服务器安装配置redis实例

```bash
[root@localhost ~]# ps -elf | grep redis
5 S root       7044      1  0  80   0 - 38499 ep_pol 09:58 ?        00:00:00 redis-server 192.168.140.11:7004 [cluster]
5 S root       7049      1  0  80   0 - 38499 ep_pol 09:58 ?        00:00:00 redis-server 192.168.140.11:7005 [cluster]
5 S root       7054      1  0  80   0 - 38499 ep_pol 09:58 ?        00:00:00 redis-server 192.168.140.11:7006 [cluster]
0 S root       7059   6790  0  80   0 - 28203 pipe_w 09:58 pts/0    00:00:00 grep --color=auto redis

[root@localhost ~]# netstat -antp | grep redis
tcp        0      0 192.168.140.11:17004    0.0.0.0:*               LISTEN      7044/redis-server 1 
tcp        0      0 192.168.140.11:17005    0.0.0.0:*               LISTEN      7049/redis-server 1 
tcp        0      0 192.168.140.11:17006    0.0.0.0:*               LISTEN      7054/redis-server 1 
tcp        0      0 192.168.140.11:7004     0.0.0.0:*               LISTEN      7044/redis-server 1 
tcp        0      0 192.168.140.11:7005     0.0.0.0:*               LISTEN      7049/redis-server 1 
tcp        0      0 192.168.140.11:7006     0.0.0.0:*     
```

### 9、创建集群

* 找一个机器输入

```bash
[root@localhost ~]# redis-cli --cluster create \
> 192.168.140.10:7001 \
> 192.168.140.10:7002 \
> 192.168.140.10:7003 \
> 192.168.140.11:7004 \
> 192.168.140.11:7005 \
> 192.168.140.11:7006 \
> --cluster-replicas 1 

yes

[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.


```

### 10、查看集群信息

```bash
[root@localhost ~]# redis-cli --cluster info 192.168.140.10:7001
192.168.140.10:7001 (6e2538d4...) -> 0 keys | 5461 slots | 1 slaves.
192.168.140.10:7002 (1192e53c...) -> 0 keys | 5461 slots | 1 slaves.
192.168.140.11:7004 (a695a4cd...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
```

### 11、连接集群、测试数据读写

```bash
[root@localhost ~]# redis-cli -h 192.168.140.11 -p 7004 -c 
192.168.140.11:7004> 
192.168.140.11:7004> set url http://baidu.com 
-> Redirected to slot [12521] located at 192.168.140.10:7002
OK
```

