[toc]

## 一、网络工作模式

```bash
[root@localhost ~]# docker network ls 
NETWORK ID     NAME      DRIVER    SCOPE
81d5242b39b2   bridge    bridge    local
c3a1ff721693   host      host      local
356904005d05   none      null      local
```

* bridge
  * 就是熟悉的NAT模式
    * 容器在访问外部网络的时候，借助主机上的路由转发功能和SNAT规则，可以实现容器访问外部网络，默认情况下外部网络不能访问内部网络（DNAT）
  * DNAT模式
    * -p 暴露端口，使得外部网络可以访问容器

```bash
[root@localhost ~]# iptables -t nat -nL
```

* host
  * 这个模式共享主机的IP、网段等信息、
  * 服务跑在容器中，容器运行在主机中
  * 注意端口冲突，在启动相同的镜像的时候会使用相同的端口，会发生端口冲突的问题
* null
  * 容器不会有任何网络连接
  * 不会有自己的网络命名空间

## 二、网络管理操作

### 1、创建网络

```bash
[root@localhost ~]# docker network create --subnet=10.1.1.0/24 --gateway=10.1.1.254 mynet_01
```

### 2、查看网络

```bash
[root@localhost ~]# docker network ls 
NETWORK ID     NAME       DRIVER    SCOPE
81d5242b39b2   bridge     bridge    local
c3a1ff721693   host       host      local
d19e77d8d96d   mynet_01   bridge    local
356904005d05   none       null      local
```

### 3、指定容器的网络

```bash
[root@localhost ~]# docker run -tid --net=mynet_01 centos /bin/bash
[root@localhost ~]# docker run -tid --net=mynet_01 --ip=10.1.1.100 centos /bin/bash
```

### 4、将某个容器加入到指定的网络中

```bash
[root@localhost ~]# docker network connect mynet_01 286e
```

### 5、删除网络

```bash
[root@localhost ~]# docker network rm mynet_01 mynet_02  	
```

## 三、flannel+etcd实现网络管理

### 1、flannel+etcd介绍

* 作用
  * 跨物理机的容器相互通信
  * 改变容器分配IP的方式
* etcd
  * 非关系型数据库的一种（键值数据库）
  * 记录网络分配信息

> * bridge模式下：每个物理机上都会有docker0网卡，由这个网卡来往外分配IP
>   * 此时如果想要与外部的物理机通信，不能直接拿个网线连在一起，会出现IP地址冲突的现象，
> * 此时就用到了flannel+etcd数据库，
> * flannel来提供一个巨大的`虚拟网络`，etcd来提供`某个IP`在`哪个物理机的哪个容器上`
> * 每个物理机上都安装flannel数据库，会生成flannel0网卡，
> * 每个flannel网卡都是这个虚拟网络的子网，
> * 再由flannel来接管docker0这个网卡，为这个虚拟机上的所有容器来动态的分配IP

![https://note.youdao.com/yws/public/resource/177109985ddced8703a06ef7db3ec83d/xmlnote/3105EEE841704540BED1ED25580E2768/3F16A4F639D74402AB6EA424CA4ED8E7/17016](https://s2.loli.net/2022/04/01/zXWUNK6Dja5CL3f.png)

### 2、配置flannel+etcd

#### 1）环境描述

> 192.168.152.14     docker-ce  flannel   etcd
>
> 192.168.152.15		docker-ce flannel

#### 2）两台服务器分别安装docker

#### 3）安装配置etcd数据库

```bash
[root@localhost ~]# yum install -y etcd 
#这里将etcd的服务监听在所有的IP上
[root@test ~]# vim /etc/etcd/etcd.conf 

#[Member]
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

#
#[Clustering]
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
```

```bash
[root@localhost ~]# systemctl start etcd
[root@localhost ~]# systemctl enable etcd

[root@localhost ~]# netstat -tunlp | grep etcd
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN      12521/etcd          
tcp6       0      0 :::2379                 :::*                    LISTEN      12521/etcd     
```

#### 4）测试etcd数据库工作正常

```bash
[root@localhost ~]# etcdctl set file01/name martin
martin
[root@localhost ~]# etcdctl get file01/name
martin
```

#### 5）在两台服务器上安装flannel

```bash
[root@localhost ~]# yum install -y flannel
```

#### 6）编辑flannel配置文件

```bash
[root@test ~]# vim /etc/sysconfig/flanneld 

FLANNEL_ETCD_ENDPOINTS="http://192.168.140.10:2379"
FLANNEL_ETCD_PREFIX="/atomic.io/network"  
```

#### 7）在etcd数据写入flannel使用的网段信息

```bash
[root@localhost ~]# etcdctl mk /atomic.io/network/config '{"Network":"10.2.0.0/16"}'
{"Network":"10.2.0.0/16"}
```

#### 8）启动flannel

```bash
[root@localhost ~]# systemctl start flanneld
[root@localhost ~]# systemctl enable flanneld
```

```bash
[root@localhost ~]# ifconfig flannel0
flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1472
        inet 10.2.45.0  netmask 255.255.0.0  destination 10.2.45.0
        inet6 fe80::1f96:1da1:2a1b:3771  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3  bytes 144 (144.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

* 查看flannel的子网环境变量

```bash
[root@localhost ~]# cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.2.0.0/16
FLANNEL_SUBNET=10.2.45.1/24
FLANNEL_MTU=1472
FLANNEL_IPMASQ=false
```

* 查看flannel即将为docker分配的IP

```bash
[root@localhost ~]# cat /run/flannel/docker 
DOCKER_OPT_BIP="--bip=10.2.45.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=true"
DOCKER_OPT_MTU="--mtu=1472"
DOCKER_NETWORK_OPTIONS=" --bip=10.2.45.1/24 --ip-masq=true --mtu=1472"
```

#### 9）修改docker启动脚本，重启docker

* 让flannel0网卡来解手docker0网卡

```bash
[root@test ~]# vim /usr/lib/systemd/system/docker.service 
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock $DOCKER_NETWORK_OPTIONS
```

```bash
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl restart docker
```

```bash
[root@localhost ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.2.45.1  netmask 255.255.255.0  broadcast 10.2.45.255
        ether 02:42:38:a6:0c:c2  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 10）第二台服务器参考上述配置

```bash
[root@localhost ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.2.64.1  netmask 255.255.255.0  broadcast 10.2.64.255
        ether 02:42:87:e8:70:b7  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 11）创建两个容器，测试通信

* 测试应该会ping不通，这是因为rpm版的flannel将防火墙的FORWARD列禁用了，在两个服务器上开启即可

```bash
[root@localhost ~]# iptables -P FORWARD ACCEPT
```

* 此时测试，应该会通

```bash
[root@8478dd5602e7 /]# ping 10.2.64.2
PING 10.2.64.2 (10.2.64.2) 56(84) bytes of data.
64 bytes from 10.2.64.2: icmp_seq=88 ttl=60 time=1.02 ms
64 bytes from 10.2.64.2: icmp_seq=89 ttl=60 time=0.719 ms
64 bytes from 10.2.64.2: icmp_seq=90 ttl=60 time=0.984 ms
64 bytes from 10.2.64.2: icmp_seq=91 ttl=60 time=0.694 ms
```

