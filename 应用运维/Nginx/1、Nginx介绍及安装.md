-

[toc]

#                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            Nginx介绍及安装

##      一、nginx介绍

> * 介绍     
>   
>   * 一般情况下系统会产生两个进程，一个系统进程，一个用户进程，当某个客户端来访问我硬盘的文件时，用户进程不允许直接硬盘内容，交给    系统进程来调取硬盘内容，                                                                                                                                                                                            
>   
> * 作用
>   * 可以配置Web服务
>     * 具有高并发
>       * C10k：即可同时处理10k的数据
>     * 具有高性能
>       * 在处理10k个不活跃的进程时，消耗的内存较低，在2.5M左右
>   * 更多的时候当作代理服务器
>     * 可保证后端服务器的安全性，
>   
> * 特性
>   * 开源、跨平台
>   * 高度模块化
>   * 高并发、高性能
>   * 支持虚拟主机
>   * 支持https
>   * 支持URL重写
>   
> * Nginx高效的原因
>   
>   * 操作系统为了支持多个应用同时运行，需要保证不同进程之间相对独立（一个进程的崩溃不会影响其他的进程 ， 恶意进程不能直接读取和修改其他进程运行时的代码和数据）。 因此操作系统内核需要拥有高于普通进程的权限， 以此来调度和管理用户的应用程序。
>     
>     于是内存空间被划分为两部分，一部分为内核空间，一部分为用户空间，内核空间存储的代码和数据具有更高级别的权限。内存访问的相关硬件在程序执行期间会进行访问控制（ Access Control），使得用户空间的程序不能直接读写内核空间的内存。
>     
>   * 基于异步非阻塞/异步IO模型
>     * 异步、同步
>       * 异步：
>         * 将用户请求放入消息队列（即缓存机制），并反馈给用户，系统迁移程序已经启动，然后用户便可以关闭浏览器，然后程序再慢慢的写到数据库中。此时用户不会有卡死的感觉，系统会告诉用户请求系统已经响应了，可以关闭界面。
>       * 同步：
>         * 所有的操作都做完，才返回给用户。这样用户在线等待的时间太长，会有一种卡死的感觉，这种情况下，用户不可以关闭界面，一旦关闭，则迁移程序就中断了。
>       * **同步，同步和异步概念描述的是用户线程与内核的交互方式：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行**
>       * **异步，而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。**
>       
>     * 非阻塞、阻塞
>       * **阻塞和非阻塞的概念描述的是用户线程调用内核IO操作的方式：阻塞是指IO操作需要彻底完成后才返回到用户空间；**
>       * **非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成**
>       
>     * 进程间的通信时通过 send() 和 receive() 两种基本操作完成的。具体如何实现这两种基础操作，存在着不同的设计。
>       消息的传递有可能是**阻塞的**或**非阻塞的** -- 也被称为**同步**或**异步**的：
>   
>       阻塞式发送（blocking send）. 发送方进程会被一直阻塞， 直到消息被接受方进程收到。
>       非阻塞式发送（nonblocking send）。 发送方进程调用 send() 后， 立即就可以其他操作。
>       阻塞式接收（blocking receive） 接收方调用 receive() 后一直阻塞， 直到消息到达可用。
>       非阻塞式接受（nonblocking receive） 接收方调用 receive() 函数后， 要么得到一个有效的结果， 要么得到一个空值， 即不会被阻塞。
>       上述不同类型的发送方式和不同类型的接收方式，可以自由组合。
>   
>     * 也就是说， 从进程级通信的维度讨论时， 阻塞和同步（非阻塞和异步）就是一对同义词， 且需要针对**发送方**和**接收方**作区分对待。
>   
>   * 基于epoll事件驱动模型设计
>     * select
>       * 周期性轮询询问内核是否调用结束，限制最大文件数1024
>     * poll
>       * 周期性轮询询问，取消了最大文件的限制数
>     * **epoll**
>       * **由内核来通知进程**

## 二、Nginx安装部署

### 1、下载nginx安装包

```bash
[root@localhost ~]# wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

### 2、安装依赖

```bash
[root@localhost ~]# yum install -y gcc openssl-devel pcre-devel zlib-devel 
```

### 3、创建nginx需要的临时目录

```bash
[root@localhost ~]# mkdir -p /var/tmp/nginx/{client,proxy,fastcgi,uwsgi,scgi}
```

### 4、创建nginx用户

```bash
[root@localhost ~]# useradd -s /sbin/nologin nginx 
```

### 5、编译安装nginx

```bash
[root@localhost ~]# tar xf nginx-1.18.0.tar.gz 
[root@localhost ~]# cd nginx-1.18.0/
[root@localhost nginx-1.18.0]# ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-file-aio --with-http_secure_link_module --with-threads --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi/ --http-scgi-temp-path=/var/tmp/nginx/scgi/
[root@localhost nginx-1.18.0]# make && make install 
```

## 三、nginx相关文件目录

> * nginx安装目录/conf
>   * 配置文件    nginx.conf主配置文件
> * nginx安装目录/logs
>   * 用来存放日志
> * nginx安装目录/html
>   * 默认网页目录
> * nginx安装目录/sbin
>   * 二进制文件

## 四、nginx启动管理

### 1、启动nginx

```bash
[root@localhost ~]# /usr/local/nginx/sbin/nginx 

[root@localhost ~]# netstat -antp | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      9726/nginx: master  

[root@localhost ~]# ps -elf | grep nginx 
1 S root       9726      1  0  80   0 - 11499 sigsus 15:56 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
5 S nginx      9727   9726  0  80   0 - 11612 ep_pol 15:56 ?  
```

* **master process 主进程**

  * 用来生成子进程，记录日志，重新加载配置文件

* **worker process  工作进程**

  * 接收、处理客户端访问请求

  

### 2、设置nginx开机自启

```bash
[root@localhost ~]# sed -ri '$a \/usr/local/nginx/sbin/nginx' /etc/rc.d/rc.local
[root@localhost ~]# chmod a+x /etc/rc.d/rc.local
```

### 3、停止、重新加载、检测语法、查看版本和配置参数

```bash
[root@localhost ~]# /usr/local/nginx/sbin/nginx -s stop
[root@localhost ~]# /usr/local/nginx/sbin/nginx -s reload
[root@localhost ~]# /usr/local/nginx/sbin/nginx -t
[root@localhost ~]# /usr/local/nginx/sbin/nginx -v
[root@localhost ~]# /usr/local/nginx/sbin/nginx -V
```

