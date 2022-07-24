# NFS网络存储

[toc]

## 一、NFS介绍

### 1、介绍

* NFS  网络文件系统
* Network File System
* NFS提供数据存储能力，同时借助RPC机制实现数据共享传输
  * NFS RPC最主要的功能就是记录每个NFS功能所对应的端口号，并且在NFS客户端请求时将该端口和功能对应的信息传递给请求数据的NFS客户端，让客户端可以链接到正确的端口上去，从而实现数据传输。
* 原理：

> 经过前面介绍，NFS系统是通过网络来进行数据传输，所以必定会使用端口来进行网络传输。NFS的传输端口是随机选择的，并且通过RPC协议来实现客户端的协调。
> 什么是RPC服务？
> RPC（远程调用服务）。因为NFS支持的功能特别多，而不同的功能都会使用不同的程序来启动，每启动一个功能端口就会用一些端口来传输数据，因此，NFS的功能所对应的端口才无法固定，而是随机取用一些未被使用的小于1024的端口来进行传输。（常规服务端口：0~655535，1024一下，系统服务常用）
> 所以，客户端要准确的获得NFS服务器所使用的端口，就需要RPC服务。 NFS RPC最主要的功能就是记录每个NFS功能所对应的端口号，并且在NFS客户端请求时将该端口和功能对应的信息传递给请求数据的NFS客户端，让客户端可以链接到正确的端口上去，从而实现数据传输。（本人觉得RPC像是中介）
> RPC怎样知道NFS的每个端口呢？
> 原因是，当NFS服务启动时会随机取用数个端口，并主动向RPC服务注册取用的相关端口信息，这样，RPC服务就可以知道每个端口所对应的NFS功能了，然后RPC服务使用固定的端口号111来监听NFS客户端提交的请求，并将正确的NFS端口答应给NFS客户端，这样一来，就可以让NFS客户端与服务端进行数据传输了。

![img](https://img-blog.csdn.net/20171220174111079?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemZ4MTk5Nw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**关键点**

服务端需要NFS服务和RPC服务，并且需要先启动RPC服务，客户端只需要RPC服务。

并且，当正式连接建立之后，RPC服务可以关掉，但当有新的请求出现时，必须开启RPC服务。
RPC服务在cestos5.x 下名字叫portmap，centos6.x下叫rpcbind

* 作用
  * NFS主要应用于实现NAS网络附加存储
  
  > NAS，指将一台服务器中的存储空间共享给其他服务器来使用，这样的好处是可以在不同服务器之间共享数据，常用的就是NFS服务；例子：当一个网站是由几个Web服务器组成的集群，就只需要将数据存放在一台服务器上，然后共享给其他服务器使用。NAS的缺点是当访问的数据量过大时，就会影响存放数据的那台服务器的性能；NAS是属于文件级的共享，性能和效率比较低
  
  * 只适用于Linux系统之间

### 2、NFS服务器

* 软件
  * nfs-utils、rpcbind
* 配置文件
  * /etc/exports   -----空文件

### 3、配置文件格式

* 格式：
  * 目录名称    客户端地址（权限）客户端地址（权限）客户端地址（权限）----------------借助nfs把我机器上的哪个目录共享出去，目录的权限是什么，
* 客户端地址z
  * 单个地址 192.168.152.1
  * 网段：192.168.152.0/24
  * 所有客户端地址：*
  * 主机名：node01.linux.com
* 常用权限
  * ro：只读
  * rw：读写--------------客户端产生新的文件能同步到我服务器上
  * sync：同步--------------cpu和硬盘同步，数据可靠性高，传输速度慢
  * async：异步----------------**cpu**先将数据在**内存**中存储，一段时间后再存到**硬盘**中，数据可靠性低，传输速度高
  * no_root_squash：保留root权限

## 二、配置NFS只读共享

### 1、关闭SELinux、防火墙、配置时间同步

### 2、创建测试文件

```bash
[root@nfsserver ~]# mkdir /webdata
[root@nfsserver ~]# touch /webdata/{1..10}.html
```

### 3、安装NFS服务端软件

```bash
[root@nfsserver ~]# yum install nfs-utils rpcbind 
```

### 4、编辑配置文件

```bash
[root@nfsserver ~]# cat /etc/exports
/webdata	192.168.140.11(ro)  192.168.140.12(ro)
```

### 5、启动NFS服务

```bash
[root@nfsserver ~]# systemctl start nfs-server
[root@nfsserver ~]# systemctl enable nfs-server
```

### 6、查看共享目录(服务是否配置成功)

```bash
[root@nfsserver ~]# showmount -e localhost 
Export list for localhost:
/webdata 192.168.140.12,192.168.140.11
```

### 7、客户端挂载NFS

```bash
[root@localhost ~]# yum install nfs-utils 
[root@localhost ~]# vim /etc/fstab 
192.168.140.10:/webdata	 /web	nfs	defaults	0 0

[root@localhost ~]# mount -a

[root@localhost ~]# df -hT | grep "web"
192.168.140.10:/webdata nfs4       18G  1.3G   17G   7% /web
```

## 三、配置NFS读写共享

### 1、创建测试文件

```bash
[root@nfsserver ~]# mkdir /dbdata
[root@nfsserver ~]# touch /dbdata/{1..10}.sql
```

### 2、编辑测试文件、添加读写共享

```bash
[root@nfsserver ~]# cat /etc/exports
/dbdata		192.168.140.11(rw,async,no_root_squash)    192.168.140.12(rw,async,no_root_squash)
[root@nfsserver ~]# chmod o+w /dbdata/
```

### 3、重新读取配置文件

```bash
[root@nfsserver ~]# exportfs -rav  
exporting 192.168.140.11:/dbdata
exporting 192.168.140.12:/dbdata
exporting 192.168.140.11:/webdata
exporting 192.168.140.12:/webdata
```

### 4、客户端挂载读写共享

```bash
[root@localhost ~]# vim /etc/fstab 
192.168.140.10:/dbdata	 /db	nfs	defaults	0 0

[root@localhost ~]# mount -a

[root@localhost ~]# df -hT | grep "db"
192.168.140.10:/dbdata  nfs4       18G  1.3G   17G   7% /db
[root@localhost ~]# 
```



