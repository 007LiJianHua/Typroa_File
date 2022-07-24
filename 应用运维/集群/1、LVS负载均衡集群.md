[toc]

## 一、集群的类型

> * 负载均衡集群   Load Balance LB
> * 高可用集群     High  Available  HA
> * 高性能计算集群   High Proformace  Compute   HPC

### 1、负载均衡集群

> 作用：提高任务的并发能力
>
> scale in：提高硬件性能，
>
> scale out ：提高服务器数量

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/F9536BF97075443AA2FC8F3C4DEFE0A6/13279](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/F9536BF97075443AA2FC8F3C4DEFE0A6/13279)

### 2、高可用集群

> 解决了单点故障，提高了可用性
>
> 可用性=运行时间/（运行时间+故障修复时间）
>
> 通过心跳机制来监视副调度器的状态

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/2FB936DE933A4FC3AFCF31DA297530F4/13289](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/2FB936DE933A4FC3AFCF31DA297530F4/13289)

### 3、高性能计算集群

> 作用：在海量数据下提高数据的处理能力
>
> 把海量数据分成多个小芬数据，分别处理，最后再汇总

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/DBB5621D2CAD49439066C8517985B818/13295](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/DBB5621D2CAD49439066C8517985B818/13295)

## 二、LVS--Linux Vritual  Service

> LVS被集成在Linux内核，速度快
>
> 通过ipvsadm命令来管理

### 1、调度算法 scheduler

> *  rr  round robin  (轮询)
> * wrr  基于权重的轮询  weight
>   * 会话保持方案：
>     * 会话共享存储redis
>     * 换调度算法（ip_hash）
> * lc  least connection  最少连接算法
> * wlc   基于权重的最少连接  （默认的）
> * sh   source  hash  源哈希算法
>   * 根据客户端的IP来计算hash值，相同的哈希值转发到相同的后端服务器上
>   * 一定程度上可以解决会话保持的问题
> * dh   destination hash 目的哈希算法
>   * 根据目的IP地址计算hash值，后续所有的请求都定位到同一个目的IP上
>   * 适用于后端是缓存服务器，提升缓存命中率

### 2、LVS的工作模式

#### 1）NAT模式

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/13252B7F7A90422083E607129B0C0514/13314](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/13252B7F7A90422083E607129B0C0514/13314)

#### 2）DR模式

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/E9826123A7CB4D5BA5D69D81739EB3CF/13317](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/E9826123A7CB4D5BA5D69D81739EB3CF/13317)

* TUN模式

### 3、NAT模式核心要素

> * 请求响应都要经过调度器转发
> * DIP、VIP要分属不同的网络
> * 调度器开启路由转发功能
> * 所有real-server的网关都要指向DIP

### 4、NAT模式的工作原理

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/601832B13BA54BDE935B17201BF44DA1/13329](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/601832B13BA54BDE935B17201BF44DA1/13329)

### 5、FULL NAT的工作原理

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/5CA834B451C04E5C8356CFF2C9E0A0BB/13331](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/5CA834B451C04E5C8356CFF2C9E0A0BB/13331)



## 三、案例：基于NAT模式配置web集群

### 1、环境说明

```bash
1、客户端为web浏览器，通过vmnet1网卡与VIP通信，需要CIP与VIP在同一个网段中
2、调度器有两张网卡，一个在vmnet1网段，一个在vmnet2网段
	还要开启路由转发功能
3、后端是两个web服务器，注意要把网关修改为调度器的DIP
```



![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/D5982AED89FB41C58C7EE6BC23E35E33/13333](https://s2.loli.net/2022/04/09/9fCPpkWI6Nydmsg.png)

### 1、所有服务器关闭防火墙、SELinux、开启时间同步

### 2、在调度器上事先安装ipvsadm、Web服务器安装httpd

```bash
[root@lvs ~]#  yum install -y ipvsadm
```

```bash
[root@web01 ~]# yum -y install httpd
```

```bash
[root@web02 ~]# yum -y install httpd
```

### 3、在调度器上分别配置VIP、DIP

> 先为调度器添加一个网卡，

```bash
[root@lvs ~]# ip addr show
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:8f:c3:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.146.100/24 brd 192.168.146.255 scope global noprefixroute ens33

3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:8f:c3:72 brd ff:ff:ff:ff:ff:ff
    inet 192.168.177.254/24 brd 192.168.177.255 scope global noprefixroute ens37
```

### 4、所有real server配置地址、网关指向DIP

```bash
[root@web01 ~]# ifconfig ens33
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.177.10  netmask 255.255.255.0  broadcast 192.168.177.255
        inet6 fe80::20c:29ff:fe14:788e  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:14:78:8e  txqueuelen 1000  (Ethernet)
        RX packets 9863  bytes 13214062 (12.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3421  bytes 213516 (208.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@web01 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.177.254 0.0.0.0         UG    100    0        0 ens33
192.168.177.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
```

### 5、在调度器上开启路由转发

```bash
[root@lvs ~]# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1

[root@lvs ~]# sysctl -p
net.ipv4.ip_forward = 1
```

### 6、在调度器上创建虚拟服务

```bash
命令解析：
  --add-service     -A        add virtual service with options
  --edit-service    -E        edit virtual service with options
  --delete-service  -D        delete virtual service
  --clear           -C        clear the whole table
  --restore         -R        restore rules from stdin
  --save            -S        save rules to stdout
  --add-server      -a        add real server with options
  --edit-server     -e        edit real server with options
  --delete-server   -d        delete real server
  --list            -L|-l     list the table
  --zero            -Z        zero counters in a service or all services
  --set tcp tcpfin udp        set connection timeout values
  --start-daemon              start connection sync daemon
  --stop-daemon               stop connection sync daemon
  --help            -h        display this help message
    --tcp-service  -t service-address   service-address is host[:port]
  --udp-service  -u service-address   service-address is host[:port]
  --fwmark-service  -f fwmark         fwmark is an integer greater than zero
  --ipv6         -6                   fwmark entry uses IPv6
  --scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
                                      the default scheduler is wlc.
  --pe            engine              alternate persistence engine may be sip,
                                      not set by default.
  --persistent   -p [timeout]         persistent service
  --netmask      -M netmask           persistent granularity mask
  --real-server  -r server-address    server-address is host (and port)
  --gatewaying   -g                   gatewaying (direct routing) (default)
  --ipip         -i                   ipip encapsulation (tunneling)
  --masquerading -m                   masquerading (NAT)
  --weight       -w weight            capacity of real server
  --u-threshold  -x uthreshold        upper threshold of connections
  --l-threshold  -y lthreshold        lower threshold of connections
  --mcast-interface interface         multicast interface for connection sync
  --syncid sid                        syncid for connection sync (default=255)
  --connection   -c                   output of current IPVS connections
  --timeout                           output of timeout (tcp tcpfin udp)
  --daemon                            output of daemon information
  --stats                             output of statistics information
  --rate                              output of rate information
  --exact                             expand numbers (display exact values)
  --thresholds                        output of thresholds information
  --persistent-conn                   output of persistent connection info
  --nosort                            disable sorting output of service/server entries
  --sort                              does nothing, for backwards compatibility
  --ops          -o                   one-packet scheduling
  --numeric      -n                   numeric output of addresses and ports
  --sched-flags  -b flags             scheduler flags (comma-separated)

```



```bash
[root@lvs ~]# ipvsadm -A -t 192.168.146.100:80 -s rr 

[root@lvs ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.146.100:80 rr
```

### 7、添加real-server

```bash
[root@lvs ~]# ipvsadm -a -t 192.168.146.100:80 -r 192.168.177.10:80 -m
[root@lvs ~]# ipvsadm -a -t 192.168.146.100:80 -r 192.168.177.20:80 -m 
```

### 8、查看负载均衡表

```bash
[root@lvs ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.146.100:80 rr
  -> 192.168.177.10:80            Masq    1      0          0         
  -> 192.168.177.20:80            Masq    1      0          0     
```

### 9、测试访问：http://192.168.152.100

```bash
[root@lvs ~]# ipvsadm -L -n -c
IPVS connection entries
pro expire state       source             virtual            destination
TCP 01:14  TIME_WAIT   192.168.146.1:55276 192.168.146.100:80 192.168.177.20:80
TCP 00:30  TIME_WAIT   192.168.146.1:56266 192.168.146.100:80 192.168.177.20:80
TCP 00:55  TIME_WAIT   192.168.146.1:60898 192.168.146.100:80 192.168.177.10:80
TCP 01:01  TIME_WAIT   192.168.146.1:65024 192.168.146.100:80 192.168.177.10:80
TCP 01:52  TIME_WAIT   192.168.146.1:51311 192.168.146.100:80 192.168.177.20:80

```

### 10、保存、恢复规则

```bash
保存 ipvsadm -S > /opt/web_rule
恢复 ipvsadm -R < /opt/web_rule
```

## 四、DR模式--Direct Route直接路由模式

### 1、核心要素

> * 请求在经过调度器、响应由real-server发送
> * real-server的网关要指向真实的网关、要能联通网络
> * VIP、DIP在同一个网段中
>   * 在所有的real-server 上全部配置VIP 
>   * 为避免局域网内的IP冲突，修改两个参数，在请求IP的时候不予回复，只有调度器回复，从而避免冲突
>     * arp_ignore=1
>       * 只让主机回复关于物理网卡的arp请求
>     * arp_announce=2
>       * 让主机以虚拟的IP发送响应报文，不需要转换为真实的物理网卡IP
>   * 因为会涉及到修改内核，所以后端的real-server必须要是类Linux系统

## 五、基于DR模式配置web系统

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/492223E5968A4384ACD7328731C9E950/13376](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9D8198E2C64A41B4A3C8BE4D32354088/492223E5968A4384ACD7328731C9E950/13376)

### 1、所有的real-server配置VIP

```bash
[root@web01 ~]# ip addr add dev lo 192.168.140.100/32
[root@web02 ~]# ip addr add dev lo 192.168.140.100/32
[root@web01 ~]# ip addr show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 192.168.140.100/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
       
[root@web02 ~]# ip addr show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 192.168.140.100/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

### 2、所有的real-server修改内核

```bash
[root@web01 ~]# vim /etc/sysctl.conf 
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2

[root@web01 ~]# sysctl -p
[root@web02 ~]# vim /etc/sysctl.conf 
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2

[root@web02 ~]# sysctl -p
```

### 3、所有real server安装配置httpd

### 4、在调度器上配置VIP

```bash
[root@lvs ~]# ip addr add dev lo 192.168.140.100/32
[root@lvs ~]# ip addr show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 192.168.140.100/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

### 5、创建虚拟服务

```bash
[root@lvs ~]# yum install -y ipvsadm.x86_64  
[root@lvs ~]# ipvsadm -A -t 192.168.140.100:80 -s rr

[root@lvs ~]# ipvsadm -a -t 192.168.140.100:80 -r 192.168.140.11:80 -g
[root@lvs ~]# ipvsadm -a -t 192.168.140.100:80 -r 192.168.140.12:80 -g
[root@lvs ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.140.100:80 rr
  -> 192.168.140.11:80            Route   1      0          0         
  -> 192.168.140.12:80            Route   1      0          0    
```

### 6、测试访问

```bash
[root@redis_server ~]# curl 192.168.140.100
<h1> web01</h1>
[root@redis_server ~]# curl 192.168.140.100
<h1> web02 </h1>

[root@redis_server ~]# curl 192.168.140.100
<h1> web01</h1>
[root@redis_server ~]# curl 192.168.140.100
<h1> web02 </h1>
```

## 六、持久性连接

### 1、作用

> 在同一时间范围内，同一个客户端的请求会被转发到同一个real-server
>
> 实现会话持久

```bash
[root@lvs ~]# ipvsadm -E -t 192.168.140.100:80 -s rr -p 300
[root@lvs ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.140.100:80 rr persistent 300
  -> 192.168.140.11:80            Route   1      0          0         
  -> 192.168.140.12:80            Route   1      0          0    
```



