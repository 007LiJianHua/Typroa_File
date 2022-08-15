[toc]

## 一、容器介绍

> 容器就是为了保证可正常运行必备的应用程序/指令
>
> * 基于操作系统级别的虚拟化

### 1、容器管理工具

* Docker
  * 基于Go语言开发
  * 开源的
  * C/S架构  Client/Server
    * B/S：Brose/Server
  * `docker-ce：社区版本，不收费`
  * docker-ee：企业版本，收费
* podman
  * docker与podman最主要的且别就在于docker在安装完后要`systemctl start docker`，而podman不需要，podman实际上就是个命令，直接使用即可
  * 其他的使用方式与选项与docker并无区别
* rts

### 2、容器的优势

* 创建速度块、秒级
* 便于业务的部署，迁移
* `容器间共享物理机的内核`、安全性低

### 3、核心技术

* namespace：命令空间
  * `实现容器间资源隔离（文件，进程，用户）`
* cgroup
  * `对容器做资源限制（CPU、内存）`
  * 如果不资源限制，就会发生OOM
    * OOM：内存资源不足，系统会随机杀进程		

## 二、安装docker-ce

### 1、配置docker软件仓库

```bash
https://mirrors.aliyun.com/repo/Centos-7.repo

[root@node01 ~]# cat /etc/yum.repos.d/docker.repo
[docker-ce]
name=docker-ce
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7.9/x86_64/stable/
enabled=1
gpgcheck=0
```

### 2、安装docker-ce

```bash
[root@localhost ~]# yum install -y docker-ce
```

### 2+、开启包转发功能和修改内核参数 

* **内核参数修改：br_netfilter 模块用于将桥接流量转发至 iptables 链，br_netfilter 内核参数需要开启转发。** 

```bash
[root@test-10 yum.repos.d]# modprobe br_netfilter	#加载模块
[root@test-10 yum.repos.d]# cat > /etc/sysctl.d/docker.conf << EOF
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> net.ipv4.ip_forward = 1
> EOF
[root@test-10 yum.repos.d]# sysctl -p /etc/sysctl.d/docker.conf 	#使得模块生效
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```

* 但是开机后加载模块回失效，建议将命令放在/etc/rc.d/rc.local   中

### 3、启动docker

```bash
[root@localhost ~]#systemctl start docker
[root@localhost ~]#systemctl enable docker
```

### 4、查看版本

```bash
[root@localhost ~]# docker version 
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:58:10 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:56:35 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.8
  GitCommit:        7eba5930496d9bbe375fdf71603e610ad737d2b2
 runc:
  Version:          1.0.0
  GitCommit:        v1.0.0-0-g84113ee
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### 5、修改国内镜像下载地址

* https://get.daocloud.io
  * 点击docker-hub加速器。选择linux，复制黏贴即可
  * 会自动帮我们生成配置文件，写上国内的镜像下载地址，并提示我们重启

```bash
[root@localhost ~]# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

```bash
[root@localhost ~]# cat /etc/docker/daemon.json
{
    "registry-mirrors": ["http://f1361db2.m.daocloud.io"]
}
[root@localhost ~]# systemctl restart docker
```

### 6、查看主机网络环境的变化

* 会开启路由转换

```bash
[root@localhost ~]# cat /proc/sys/net/ipv4/ip_forward
1
```

* 在防火墙NAT表中，会生成当容器内172.17.0.0/16 的所有IP，在访问外网的所有资源是，会做NAT网络地址转换

```bash
[root@localhost ~]# iptables -t nat -nL
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0           

Chain DOCKER (2 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0     
```

* 会生成Bridge的虚拟网络,NAT模式，开启DHCP服务

![image-20220401101740107](https://s2.loli.net/2022/04/01/IxNXG4DOwSv2Ru3.png)

* 对应也会生成一个网卡

```bash
[root@localhost ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:a2ff:fe68:1f28  prefixlen 64  scopeid 0x20<link>
        ether 02:42:a2:68:1f:28  txqueuelen 0  (Ethernet)
```

## 三、容器基本管理操作

### 1、查看/拉取镜像

```bash
# docker search <关键字>
# docker pull <镜像名称>
[root@localhost ~]# docker image ls 
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    08b152afcfae   3 days ago     133MB
mysql        latest    c60d96bd2b77   3 days ago     514MB
centos       latest    300e315adb2f   7 months ago   209MB
```

### 2、创建容器

* 容器的后边可以跟命令，命令运行结束，容器也对应停止，因为容器的产生就是为了完成任务，
* `核心要素`
  * **<font color='red'>容器的运行是依赖于持续的进程</font>**
  * **<font color='red'>一个容器只运行一个任务</font>**

```bash
#补：在运行容器的时候以特权模式运行
docker run -d --name=master_mysql --privileged=true test1_mysql:v1 /usr/sbin/init
```



```bash
[root@localhost ~]# docker run -tid centos ping baidu.com
[root@localhost ~]# docker run -tid centos sleep 10
[root@localhost ~]# docker run -tid centos /bin/bash
[root@localhost ~]# docker run -tid centos ls
[root@localhost ~]# docker run -tid nginx
```

### 3、查看容器

```bash
[root@localhost ~]# docker ps -a
```

### 4、删除容器

```bash
[root@localhost ~]# docker rm f296e797464f
```

### 5、停止容器

```bash
[root@localhost ~]# docker stop 8b3d
8b3d
[root@localhost ~]# docker rm 8b3d
```

```bash
[root@localhost ~]# docker ps -qa | awk '{print "docker rm -f", $0}' | bash
98e5ad206be7
d4969c8a7289
8c87f3bcffe5
902ca926ece7
16c899459118
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 6、连接容器

```bash
[root@localhost ~]# docker exec -ti 16c8 bash 
```

### 7、查看容器的详细信息

```bash
[root@localhost ~]# docker inspect 8c87
```

### 8、查看日志

```bash
[root@localhost ~]# docker logs <ID>
```

### 9、导出容器

```bash
[root@localhost ~]# docker export -o test.tar 8478 
```

### 10、导入容器

```bash
[root@localhost ~]# docker import <file>
```

## 四、创建容器常用的选项

### 1、指定容器的名字

* --name

```bash
[root@localhost ~]# docker run -tid --name test_demo centos /bin/bash
```

* --hostname
  * 指定容器的主机名字

```bash
[root@node01 ~]# docker run -tid --name=test2 --hostname=masterDB centos
```

### 2、发布服务

* -p 物理机端口：容器端口

```bash
[root@localhost ~]# docker run -tid -p 80:80 nginx

[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                               NAMES
447257e06d1f   nginx     "/docker-entrypoint.…"   4 seconds ago        Up 3 seconds        0.0.0.0:80->80/tcp, :::80->80/tcp   sharp_noether

[root@localhost ~]# docker exec -ti 4472 bash
root@447257e06d1f:/# 
root@447257e06d1f:/# ls /usr/share/nginx/html/
50x.html  index.html
root@447257e06d1f:/# echo "abc" > /usr/share/nginx/html/index.html 
root@447257e06d1f:/# 
root@447257e06d1f:/# exit
```

### 3、设置容器的重启策略

* --restart=always
  * 开机自启

```bash
[root@localhost ~]# docker run -tid --restart=always centos /bin/bash
```

### 4、资源限制

> * `针对CPU、内存`、做资源限制
> * 默认创建容器时，物理机是不会对容器做资源限制的，如果容器数量过多，导致`OOM现象`，这时，物理机就会启动`OOM-Killer机制`，随机杀进程

#### 1）对CPU进行资源限制

```bash
[root@localhost ~]# docker run -tid --cpus=2 centos /bin/bash
```

```bash
[root@test ~]# docker run -tid --cpus=2 polinux/stress /bin/bash
[root@test ~]# docker exec -ti 3722 bash

bash-5.0# stress --cpu 2 --io 4 --vm 2 --vm-bytes 128M --timeout 60s &

在物理机使用top查看CPU使用率 
```

#### 2）对内存进行资源限制

```bash
[root@localhost ~]# docker run -tid --memory=128M polinux/stress /bin/bash
620512838d0f723d310dca8a9801cf67538f97e4b47cdb2b40991c9d60fd405c

[root@test ~]# docker exec -ti 61e7 bash 
bash-5.0# stress --vm 10 --vm-bytes 200M --verbose 
```

### 5、传递变量

* -e 变量名称=值

```bash
[root@localhost ~]# docker run -tid -e DATA_01=martin centos /bin/bash
222ae6e7cd485ab7d35b21ba1d029b787b284ef8fa9fe74277efd2825299d14b
[root@localhost ~]# 
[root@localhost ~]# docker exec -ti 222a bash
[root@222ae6e7cd48 /]# 
[root@222ae6e7cd48 /]# echo $DATA_01
martin
[root@222ae6e7cd48 /]# exit
exit

[root@localhost ~]# docker run -tid -e MYSQL_ROOT_PASSWORD="redhat" mysql 
[root@localhost ~]# docker run -tid -e MYSQL_ROOT_PASSWORD="redhat" -e MYSQL_DATABASE="jishu" mysql
```

### 6、实现容器数据的持久化存储

**方式一）将容器目录挂载到物理机目录上**

* -v  物理机目录：容器目录

```bash
[root@localhost ~]# docker run -tid -v /opt/projectA/:/webdata centos /bin/bash
```

```bash
[root@localhost ~]# docker run -tid -e MYSQL_ROOT_PASSWORD="redhat" -v /opt/MySQL/my.cnf:/etc/my.cnf mysql
```

* 为数据卷添加访问权限
  * 这样容器内部就不能修改文件了，只能通过再宿主机上进行修改，	

```bash
[root@localhost ~]# docker run -tid -v /opt/projectA/:/webdata:ro  centos /bin/bash
```



**方式二）创建数据卷**

*  数据卷的本质也就是个目录

#### 1）创建数据卷

* 数据卷的所有数据都在
  * /var/lib/docker

```bash
[root@localhost ~]# docker volume create mysql_datadir
mysql_datadir

[root@localhost ~]# docker volume ls
DRIVER    VOLUME NAME
local     mysql_datadir

[root@localhost ~]# ls /var/lib/docker/volumes/
54b56dc8fa1e65dff34e8468c9250ca553538bd58b7830a75db270f1c4d349d2  backingFsBlockDev
68af54ff52b1939ff3ab1f99c3b61bdeb765a0790ce4c982a89b17e4388e5b8f  c1a104e334fbccc79fe35eee4704d607d0a824176dd60daeb38541aed3bd98e0
8c239fd126b222bd348de3643dbf8b0b8fe185b758908a471c991b020a492f0b  d8d0f17f7165a58e0c5bd510c971bb633e25737d267c9b5ccb68ef6d4fd584d9
9c189c0d33de44326303d372ab904024cb21c91822d2e86fa9b6a8600efcaff1  ddbf7c39c3cf79b18b963ac0fce7998f24e8d0c2eb7e30599c8a2f70ca0cf350
acc0732a8fd000628375c157c46420fec6d97323c005372045102183079c9910  metadata.db
baa7f02e60390de92b9237a427008889e624b21e398a8bca9d252f0a5da8bc71  mysql_datadir
```

#### 2）使用卷

```bash
[root@localhost ~]# docker run -tid -e MYSQL_ROOT_PASSWORD="redhat" --mount src=mysql_datadir,dst=/var/lib/mysql mysql
```

```bash
[root@localhost ~]# ls /var/lib/docker/volumes/mysql_datadir/_data/
```

#### 方式三）共享数据卷容器	

可以挂载其他的容器的数据卷，共享到自己身上，那个被挂载的容器就叫做共享数据卷容器。

```bash
docker run -tid --name data-vloume2  --vloumes-from data-volume1 centos /bin/bash
```

### 7、设置容器间的网络别名

* --link=容器的名字：别名
  * 别名的存在是为了避免DHCP的缺点，当启动容器的顺序不同的时候，dhcp配分IP的顺序就会改变，在加上了网络别名之后，容器之间会动态更新网络别名对应的IP，这样就避免了，当IP发生变化的时候，容器之间通信失败

```bash
[root@localhost ~]# docker run -tid --name demo centos /bin/bash

[root@localhost ~]# docker run -tid --link=demo:test_server centos /bin/bash
```

### 8、容器数据备份和还原

* 容器的数据备份
  * 先创建一个容器以data-volume2为共享数据卷，并挂载主机的/backup/目录，
  * 再将容器内的/datavolume2目录挂载到/backup/下，

```bash
docker run -tid --volume-from data-volume2 -v /root/backup:/backup --name datavolume-copy centos tar zcvf /backup/data-volume2.tar.gz /datavolume2
```

* 容器的数据还原
  * 其余的都一样，就是将tar命令换位tar -zxvf 命令。

```bash
docker run -tid --volume-from data-volume2 -v /root/backup:/backup --name datavolume-copy centos tar zxvf /backup/data-volume2.tar.gz -C /
```

### 9、开启特权模式

```bash
#特权模式就是在启动容器的时候添加 --privileged=true ，容器会使用root权限
```



## 五、容器资源限制

### 1、查看配置份额的帮助命令：

```bash
[root@xianchaomaster1 ~]# docker run --help | grep cpu-shares

cpu配额参数：-c, --cpu-shares int
CPU shares (relative weight)在创建容器时指定容器所使用的CPU份额值。cpu-shares的值不能保证可以获得1个vcpu或者多少GHz的CPU资源，仅仅只是一个弹性的加权值,默认每个docker容器的cpu份额值都是1024。在同一个CPU核心上，同时运行多个容器时，容器的cpu加权的效果才能体现出来。
```

例：两个容器A、B的cpu份额分别为1000和500，结果会怎么样？

> 情况1：A和B正常运行，占用同一个CPU，在cpu进行时间片分配的时候，容器A比容器B多一倍
>
> 的机会获得CPU的时间片。
>
> 
>
> 情况2：分配的结果取决于当时其他容器的运行状态。比如容器A的进程一直是空闲的，那么容器B
>
> 是可以获取比容器A更多的CPU时间片的；比如主机上只运行了一个容器，即使它的cpu份额只有
>
> 50，它也可以独占整个主机的cpu资源。
>
> 
>
> cgroups只在多个容器同时争抢同一个cpu资源时，cpu配额才会生效。因此，无法单纯根据某个容
>
> 器的cpu份额来确定有多少cpu资源分配给它，资源分配结果取决于同时运行的其他容器的cpu分
>
> 配和容器中进程运行情况。

### 2、例1：给容器实例分配512权重的cpu使用份额

参数：  --cpu-shares 512

```bash
[root@xianchaomaster1 ~]# docker run -it --cpu-shares 512 centos
/bin/bash
[root@df176dd75bd4 /]# cat /sys/fs/cgroup/cpu/cpu.shares  #查看结果：
512
```

### 3、CPU core核心控制

* 创建两个容器docker10与20 ，docker10使用512份额CPU、docker20使用1024份额CPU，都是使用CPU0和CPU1，使用stress压测工具进行测试，注意，为避免虚拟机被压爆，最少也要有4核CPU。

![img](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/wps1.png)

* 创建两个容器

```bash
[root@xianchaomaster1 ~]# docker run -itd --name docker10 --cpuset-cpus 0,1  --cpu-shares
512 centos  /bin/bash
[root@xianchaomaster1 ~]# docker run -itd --name docker20 --cpuset-cpus 0,1  --cpu-shares
1024 centos  /bin/bash
```

* 进入容器，安装压测工具

```bash
[root@xianchaomaster1 ~]# yum install -y epel-release
[root@xianchaomaster1 ~]# yum install stress -y
```

* 开始压测,在每个容器内部执行。

```bash
[root@xianchaomaster1]# stress -c 2 -i 2 --verbose --timeout 20s
```

* 另开一个终端，使用top命令，查看，

![img](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/wps3.png)

### 4、限制内存

* Docker 提供参数-m, --memory=""限制容器的内存使用量。

```bash
[root@xianchaomaster1 ~]# docker run -it -m 128m centos 
查看： 
[root@40bf29765691 /]# cat /sys/fs/cgroup/memory/memory.limit_in_bytes 
134217728
```

* 例 2：创建一个 docker，只使用 2 个 cpu 核心，只能使用 128M 内存 

```bash
[root@xianchaomaster1 ~]# docker run -it --cpuset-cpus 0,1 -m 128m centos 
```

### 5、限制磁盘IO

```bash
[root@xianchaomaster1 ~]# docker run --help | grep write-b 
 --device-write-bps value Limit write rate (bytes per second) to a device 
(default []) 
#限制此设备上的写速度（bytes per second），单位可以是 kb、mb 或者 gb。 
--device-read-bps value #限制此设备上的读速度（bytes per second），单位可以是 kb、mb 或 者 gb。
```

* 例 1：限制容器实例对硬盘的最高写入速度设定为 2MB/s。 

```bash
--device 参数：将主机设备添加到容器 
[root@xianchaomaster1 ~]# mkdir -p /var/www/html/
#限制容器上的/dev/sda的写速度为2Mb/s
[root@xianchaomaster1 ~]# docker run -it -v /var/www/html/:/var/www/html --device /dev/sda:/dev/sda --device-write-bps /dev/sda:2mb centos /bin/bash
 
 注：dd 参数： 
direct：读写数据采用直接 IO 方式，不走缓存。直接从内存写硬盘上。 
nonblock：读写数据采用非阻塞 IO 方式，优先写 dd 命令的数据
[root@bd79042dbdc9 /]# time dd if=/dev/sda of=/var/www/html/test.out bs=2M count=50 oflag=direct,nonblock
50+0 records in 
50+0 records out 
52428800 bytes (52 MB) copied, 50.1831 s, 2.0 MB/s 
real 0m50.201s 
user 0m0.001s 
sys 0m0.303s 
注： 发现 1 秒写 2M。 限制成功。
```

### 6、自动释放资源

```bash
[root@xianchaomaster1 ~]# docker run --help | grep rm 
 --rm 参数： Automatically remove the container when it exits 
作用：当容器命令运行结束后，自动删除容器，自动释放资源 
例： 
[root@xianchaomaster1 ~]# docker run -it --rm --name xianchao centos sleep 6 
物理上查看： 
[root@xianchaomaster1 ~]# docker ps -a | grep xianchao 
6c75a9317a6b centos "sleep 6" 6 seconds ago Up 4 
seconds mk 等 5s 后，再查看： 
[root@xianchaomaster1 ~]# docker ps | grep xianchao #自动删除了
```











## 六、实战

### 1、部署通过nginx调度器调度tomcat集群

* 创建两个tomcat需要的网页文件

```bash
[root@localhost ~]# cat /opt/java/app1/ROOT/index.jsp 
<h1>Tomcat_01</h1>
[root@localhost ~]# cat /opt/java/app2/ROOT/index.jsp 
<h1>Tomcat_02</h1>
```

* 创建nginx 的配置文件

```bash
[root@localhost ~]# cat /opt/java/nginx.conf 
upstream  java {
	server tomcat_01:8080;
	server tomcat_02:8080;
}

server {
	listen 80;
	server_name localhost;
	location / {
	proxy_pass http://java;
	}
}
```

* 分别创建两个tomcat容器，使用-v来挂载Tomcat的配置文件

```bash
[root@localhost ~]# docker run -tid --name=tomcat_01 -v /opt/java/app1/ROOT/index.jsp:/usr/local/tomcat/webapps/ROOT/index.jsp tomcat
5f8ad1ab0c0064ea9bae3d02cc9146b87afddada01377029df17eff02ed0bf17
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE      COMMAND             CREATED         STATUS                      PORTS      NAMES
5f8ad1ab0c00   tomcat     "catalina.sh run"   7 seconds ago   Up 5 seconds                8080/tcp   tomcat_01
57ffa456c24f   centos:7   "/bin/bash"         15 hours ago    Exited (137) 12 hours ago              hardcore_golick
```

```bash
[root@localhost ~]# docker run -tid --name=tomcat_02 -v /opt/java/app2/ROOT/index.jsp:/usr/local/tomcat/webapps/ROOT/index.jsp tomcat
d693d3ce2c3326f8d35f455fe2b73748dcb7de66c7e6eaabf4281525e38b875d
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE      COMMAND             CREATED          STATUS                      PORTS      NAMES
d693d3ce2c33   tomcat     "catalina.sh run"   3 seconds ago    Up 2 seconds                8080/tcp   tomcat_02
5f8ad1ab0c00   tomcat     "catalina.sh run"   25 seconds ago   Up 23 seconds               8080/tcp   tomcat_01
57ffa456c24f   centos:7   "/bin/bash"         15 hours ago     Exited (137) 12 hours ago              hardcore_golick

```

* 创建nginx调度器

```bash
[root@localhost java]# docker run -tid --name=nginx -v /opt/java/nginx.conf:/etc/nginx/conf.d/default.conf --link=tomcat_01:tomcat_01 --link=tomcat_02:tomcat_02 -p 3333:80 nginx
```

* 在WIndos输入http://192.168.152.10:3333
  * 即可访问

### 2、部署wordpress论坛，后边是mysql主从复制

* 规划目录

```bash
[root@localhost data_dir]# pwd
/opt/data_dir
[root@localhost data_dir]# ll
total 0
#分别是主和从的目录
drwxr-xr-x 3 root root 39 Apr  1 20:53 master
drwxr-xr-x 3 root root 38 Apr  1 20:54 slave
[root@localhost data_dir]# ll master/
total 8
#各自目录包含数据目录和主配置文件
drwxr-xr-x 6 polkitd root 4096 Apr  1 21:50 datadir
-rw-r--r-- 1 root    root   38 Apr  1 20:53 master.cnf
[root@localhost data_dir]# ll slave/
total 8
drwxr-xr-x 6 polkitd root 4096 Apr  1 21:50 datadir
-rw-r--r-- 1 root    root   37 Apr  1 20:54 slave.cnf
```

* 生成master容器

```bash
[root@localhost ~]# docker run -tid --name=mysql_master -v /opt/data_dir/master/datadir/:/var/lib/mysql -v /opt/data_dir/master/master.cnf:/etc/my.cnf -e MYSQL_ROOT_PASSWORD="redhat" mysql:5.7
```

* 生成slave容器

```bash
[root@localhost ~]# docker run -tid --name=mysql_master -v /opt/data_dir/slave/datadir/:/var/lib/mysql -v /opt/data_dir/slave/slave.cnf:/etc/my.cnf --link=mysql_master:mysql_master -e MYSQL_ROOT_PASSWORD="redhat" mysql:5.7
```

* 查看

```bash
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS                      PORTS                                   NAMES
6dffebb3bd2e   mysql:5.7   "docker-entrypoint.s…"   38 minutes ago   Up 38 minutes               3306/tcp, 33060/tcp                     mysql_slave
fbf75bdd244b   mysql:5.7   "docker-entrypoint.s…"   59 minutes ago   Up 59 minutes               3306/tcp, 33060/tcp                     mysql_master
4d3532709fca   nginx       "/docker-entrypoint.…"   3 hours ago      Up 3 hours                  0.0.0.0:3333->80/tcp, :::3333->80/tcp   nginx
d693d3ce2c33   tomcat      "catalina.sh run"        3 hours ago      Up 3 hours                  8080/tcp                                tomcat_02
5f8ad1ab0c00   tomcat      "catalina.sh run"        3 hours ago      Up 3 hours                  8080/tcp                                tomcat_01
57ffa456c24f   centos:7    "/bin/bash"              18 hours ago     Exited (137) 15 hours ago                                           hardcore_golick

```

* 为slave服务器创建授权用户

```bash
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replicate'@'172.17.0.6' IDENTIFIED BY 'redhat';
mysql> FLUSH PRIVILEGES;
```

* 在slave服务器上连接主服务器

```bash
mysql> CHANGE MASTER TO 
MASTER_HOST="mysql_master",
MASTER_USER="REPLICATE",
MASTER_PASSWORD="redhat",
MASTER_LOG_FILE="master.000003",
MASTER_LOG_POS=603;
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql_master
                  Master_User: replicate
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000003
          Read_Master_Log_Pos: 653040
               Relay_Log_File: 6dffebb3bd2e-relay-bin.000002
                Relay_Log_Pos: 652754
        Relay_Master_Log_File: master.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 653040
              Relay_Log_Space: 652968
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 100
                  Master_UUID: 1be928c4-b1bc-11ec-8bc4-0242ac110005
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
```

* 启动wordpress容器

```bash
[root@localhost ~]# docker run -tid --name=wordpress --link=mysql_master:mysql_master 
-p 4444:80
-e WORDPRESS_DB_HOST=mysql_master 
-e WORDPRESS_DB_USER=wpuser 
-e WORDPRESS_DB_PASSWORD=redhat 
-e WORDPRESS_DB_NAME=wordpress 
-e WORDPRESS_TABLE_PREFIX=wp_ wordpress
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED             STATUS                      PORTS                                   NAMES
0976cb64a941   wordpress   "docker-entrypoint.s…"   23 minutes ago      Up 23 minutes               0.0.0.0:4444->80/tcp, :::4444->80/tcp   wordpress
6dffebb3bd2e   mysql:5.7   "docker-entrypoint.s…"   52 minutes ago      Up 52 minutes               3306/tcp, 33060/tcp                     mysql_slave
fbf75bdd244b   mysql:5.7   "docker-entrypoint.s…"   About an hour ago   Up About an hour            3306/tcp, 33060/tcp                     mysql_master

```

网页测试:http://192.168.152.15:4444