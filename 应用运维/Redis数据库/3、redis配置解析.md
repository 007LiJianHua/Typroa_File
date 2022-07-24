[toc]

## 一、配置文件--redistribution.conf

### 1、加载子配置文件

```bash
# include /path/to/local.conf
# include /path/to/other.conf
```

### 2、加载模块

```bash
loadmodule /path/to/my_module.so
```

### 3、修改redis监听地址

```bash
bind x.x.x.x

[root@localhost ~]# redis-cli -h 192.168.140.10
192.168.140.10:6379> 
192.168.140.10:6379> set a 10
OK
```

### 4、修改redis的端口

```bash
port 6379
```

### 5、修改redis接收请求对列的长度

```bash
tcp-backlog 511
```

* 如果网卡的消息队过短，需要修改内核参数
  * somaxconn: 系统层面，网卡接收请求的队列长度
  * tcp_max_syn_backlog：系统层面，网卡接收TCP第一次握手的队列长度

临时修改系统参数

```bash
[root@localhost ~]# echo 1000 > /proc/sys/net/core/somaxconn
[root@localhost ~]# echo 1000 > /proc/sys/net/ipv4/tcp_max_syn_backlog
[root@localhost ~]# 
[root@localhost ~]# cat /proc/sys/net/core/somaxconn 
1000
[root@localhost ~]# cat /proc/sys/net/ipv4/tcp_max_syn_backlog 
1000
```

永久修改系统参数

```bash
[root@localhost ~]# cat /etc/sysctl.conf 
net.core.somaxconn = 1000
net.ipv4.tcp_max_syn_backlog = 1000
#重新加载内核
[root@localhost ~]# sysctl -p
net.core.somaxconn = 1000
net.ipv4.tcp_max_syn_backlog = 1000
#查看所有参数
[root@localhost ~]# sysctl -a
```

### 6、设置客户端空闲的超时时间

```bash
timeout 30
```

### 7、设置redis在后台启动

```bash
daemonize yes
```

### 8、redis的PID文件

```bash
pidfile /var/run/redis_6379.pid
```

### 9、错误日志

```bash
loglevel warning
logfile "/var/log/redis_6379.log"
```

### 10、定义redis自动持久化的频率

```bash
save 900 1
save 300 10
save 60 10000
```

### 11、指定rdb数据文件的存储位置

手动持久化

```bash
>bgsave;
```

```bash
dbfilename dump_6379.rdb
dir /usr/local/redis/data
```

### 12、启动aof日志

```bash
appendonly yes
appendfilename "appendonly_6379.aof"
```

### 13、设置redis密码

```bash
requirepass redhat
```

密码认证

```bash
[root@localhost ~]# redis-cli -h 192.168.140.10
192.168.140.10:6379> auth redhat
OK
192.168.140.10:6379> set aa 100
OK
192.168.140.10:6379> exit
```

或者

```bash
[root@localhost ~]# redis-cli -h 192.168.140.10 -a redhat 
192.168.140.10:6379> set bb 200
OK
192.168.140.10:6379> exit
```



### 14、设置命令别名

```bash
rename-command set add
```

### 15、设置redis最大并发连接数

```bash
maxclients 10000
```

### 16、设置最大内存策略

```bash
maxmemory 200M
maxmemory-policy noeviction
```

* 支持的策略：

> - noeviction    
>
> - -   默认策略 内存空间不足时，添加新的数据会返回报错信息
>
> - allkeys‐lru    
>
> - -  内存不足时，redis会按照LRU(最近最少访问)算法清除缓存数据
>
> - allkeys‐random  
>
> - - 内存不足时，redis会随机删除缓存数据
>
> - volatile‐lru    
>
> - - 内存不足时，redis会在设置了过期时间的缓存数据中，按照LRU算法清除数据
>
> - volatile‐random 
>
> - - 内存不足时，redis会在设置了过期时间的缓存数据中，随机清除数据
>
> - volatile‐ttl    
>
> - - 内存不足时，redis会在设置了过期时间的缓存数据中，优先清除过期时间较早的数据
>
> - allkeys-lfu
>
> - volatile-lfu
>
> - - lfu（Least Frequently Used）算法根据数据的历史访问频率来淘汰数据