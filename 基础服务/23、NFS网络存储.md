# NFS网络存储

[toc]

## 一、NFS介绍

### 1、介绍

* NFS  网络文件系统
* Network File System
* NFS提供数据存储能力，同时借助RPC机制实现数据共享传输
* 作用
  * 中小型业务的数据存储能力
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
  * 单个地址 192。168.152.1
  * 网段：1921.68.152.0/24
  * 所有客户端地址：*
  * 主机名：node01.linux.com
* 常用权限
  * ro：只读
  * rw：读写--------------客户端产生新的文件能同步到我服务器上
  * sync：同步--------------cpu和硬盘同步，数据可靠性6高，传输速度慢
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
[root@nfsserver ~]# systemctl enable nfs-serve
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



