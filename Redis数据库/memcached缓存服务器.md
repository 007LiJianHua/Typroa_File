[toc]

## 一 、 MemCache 简介

> memcached  是一款开源的、高性能的纯内存缓存服务软件，mem  是内存的意思 ，cache 是缓存的意思 ， d 是 daemon 的意思
>
> memcache 是项目的名字，诞生于 2003 年，memcached 服务分为 客户端和服务端两部分，C/S 架构 ；
>
> 缓存数据库包含  ； memcache  redis ，它们的 数据是存储在内存中，而内存的速度远远快于磁盘加快网页响应速度 ；
>
> NOSQL 数据库；not  only SQL  （ 非关系型数据库 ） ；
>
> memcache缓存数据库的并发 ； --->> 10w   
>
> memcache的缺点就是内存的缺点 ---->> 断电数据丢失 
>
> 企业中常用的memcache 架构 ： MySQL + memcache   （互补一下 ）
>
> 这里说一下 Redis 优点 ；
>
> Redis 工作区域在内存，但是会定时的将内存的数据保存到磁盘中 。

![img](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7CB086B486C24DA1A4F18F57BFC6B5B8/04CE794CE32E4BF9B8CE6457DB1BBD38/13203)

## 二、Memcached的原理及优点

> 再启动 memcache 的时候 ，需要指定内存大小，根据指定的内存大小会被分配一个内存空间，当我们读取数据库的各类业务数据后，数据会同时放入到memcache 缓存中，当用户下一次请求同样的数据时，程序直接去memcache 缓存中取数据，直接返回给用户 。
>
> 优点 ：
>
> ① 、 对于用户而言，用户访问网页更快了，体验好
>
> ② 、对于网站来说，数据库的压力降低了，只有当内存没有数据的时候才会去请求数据库，第一次写入数据是会请求数据库。  
>
>       因为 memcached 是直接从内存读取数据的很快 ，而MySQL是从磁盘取数据的，上面也说了内存的响应速度比磁盘快很多 。
>       还有就是一般公司会有 “预热” 就是先把MySQL 中的数据放到 memcache 中  。
>
> ③ 、提升了网站的并发访问，减少服务器数量 ；
>
> 

## 三、Memcached的原理及优点

> 数据读取流程 ；
>
> 步骤① 、memcache 中没有发现用户需要的数据，程序只能到MySQL中读取，优先会把数据返回给用户，其次会将这个数据缓存到memcache中 。
>
> 步骤② 、程序会优先判断这个数据是否存在 memcache 中，如果在直接从memcache 中返回给用户，如果不在重复步骤①
>
> memcache 可以作为数据库的前端缓存应用 ；

![img](https://img-blog.csdn.net/20180915210123158?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQxMDc1MTQ2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180915213339294?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQxMDc1MTQ2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 四、Memcached内存分配的方式

> * memcached开启的时候会指定分配的内存大小，在固定的内存中**默认分配的内存大小为48Bytes, 以1.25倍向上递僧**，直到将分配的内存空间占满，根据数据的大小来存储

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7CB086B486C24DA1A4F18F57BFC6B5B8/CD28885566B848DD8131916C8B66BB60/13207](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7CB086B486C24DA1A4F18F57BFC6B5B8/CD28885566B848DD8131916C8B66BB60/13207)

## 五、安装使用memcached

### 1、安装libevent库文件

```bash
[root@localhost ~]# tar xf libevent-2.1.8-stable.tar.gz 
[root@localhost ~]# cd libevent-2.1.8-stable/
[root@localhost libevent-2.1.8-stable]# ./configure --prefix=/usr/local/libevent 
[root@localhost libevent-2.1.8-stable]# make && make install 
```

### 2、安装memcached

```bash
[root@localhost ~]# tar xf memcached-1.5.1.tar.gz 
[root@localhost ~]# cd memcached-1.5.1/
[root@localhost memcached-1.5.1]# ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local/libevent 
[root@localhost memcached-1.5.1]# make && make install 
```

### 3、启动memcached

> 启动参数说明
>
> - -p
>
> - - 端口
>
> - -l
>
> - - 监听地址
>
> - -u
>
> - - 启动用户
>
> - -d
>
> - - 后台启动
>
> - -m
>
> - - 允许memcached所使用的最大内存
>
> - -c
>
> - - 最大并发连接数
>
> - -f
>
> - - 增长因子，默认1.25倍
>
> - -n
>
> - - memcached分配内存的基本大小，默认48B
>
> - -t
>
> - - 线程数

```bash
[root@localhost ~]# useradd -s /sbin/nologin memcached

[root@localhost ~]# /usr/local/memcached/bin/memcached -p 11211 -l 192.168.140.12 -u memcached -d -m 500M -c 2000 -n 20 -f 2 -t 10 

[root@localhost ~]# netstat -antp | grep memca
tcp        0      0 192.168.140.12:11211    0.0.0.0:*               LISTEN      25743/memcached     

[root@localhost ~]# ps -elf | grep memcached
1 S memcach+  25743      1  0  80   0 - 214343 ep_pol 15:53 ?       00:00:00 /usr/local/memcached/bin/memcached -p 11211 -l 192.168.140.12 -u memcached -d -m 500M -c 2000 -n 20 -f 2 -t 10

```

### 4、测试读写

> ping命令用来测试网络连通性
>
> telnet命令用来查看某个端口是否可访问

```bash
[root@localhost ~]# telnet 192.168.140.12 11211
Trying 192.168.140.12...
Connected to 192.168.140.12.
Escape character is '^]'.
set name 0 20 6     20  过期时间  6 数据的长度
martin
STORED
get name
VALUE name 0 6
martin
END
get name
END
quit
```

### 5、PHP代码连接memcached

```bash
[root@localhost ~]# yum install -y httpd php gd php-gd php-devel php-pecl-memcache
[root@localhost ~]# php -m | grep -i mem
memcache

[root@localhost ~]# cat /var/www/html/test1.php
<?php
   $memcache_obj = memcache_connect('192.168.140.10', 11211);
   $memcache_obj->add("name", "Martin");
   echo $memcache_obj->get("name");
?>
```

