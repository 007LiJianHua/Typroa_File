# MPM模型及ab压力测试

[toc]

## 一、MPM模型

* 处理并发连接的处理方式

### 1、MPM的方式

* prefork

* worker
* event

### 2、查看httpd当前正在使用的MPM模型

```bash
[root@music ~]# httpd -V 
Server version: Apache/2.4.6 (CentOS)
Server built:   Nov 16 2020 16:18:20
Server's Module Magic Number: 20120211:24
Server loaded:  APR 1.4.8, APR-UTIL 1.5.2
Compiled using: APR 1.4.8, APR-UTIL 1.5.2
Architecture:   64-bit
Server MPM:     prefork		//所使用的MPM方式
```

### 3、prefork

> * **<font color="red">预先生成子进程，由不同的子进程处理不同的客户端请求</font>**，预派生模式，有⼀个主控制进程，然后⽣成多个⼦进程，使用select模型，最⼤并发1024，每个子进程有⼀个独立的线程响应用户请求，相对⽐较占用内存，但是比较稳定，可以设置最⼤和最小进程数，是最古⽼的⼀种模式，也是最稳定的模式，适用于访问量不是很大的场景。
>
> * 优势
>   * 稳定
> * 劣势
>   * 易过度消耗内存，大量用户访问慢，占用资源，1024个进程不适⽤于高并发场景
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019122919432434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rob21hc19DWVE=,size_16,color_FFFFFF,t_70)

```bash
<IfModule prefork.c>
StartServers 16 //默认启动子进程数
MinSpareServers 5 //最小空闲的子进程
MaxSpareServers 5 //最大空闲的子进程
ServerLimit 256 //服务器允许启动的最多的进程数
MaxClients 256 //服务器允许的最大连接数
MaxRequestsPerChild 4000 //每个子进程最多允许处理多少个请求
</IfModule>
```

### 4、worker

> * 预先生成子进程，每个子进程会产生多个线程，由每个线程来处理客户端请求，⼀种多进程和多线程混合的模型，有⼀个控制进程，启动多个⼦进程，每个⼦进程里面包含固定的线程，使⽤线程 程来处理请求，当线程不够使⽤的时候会再启动⼀个新的⼦进程，然后在进程⾥⾯再启动线程处理请求，由于其使⽤了线程处理请求，因此可以承受更⾼的并发。
> * 优势
>   * 速度快
>   * 由同一个进程产生的多个线程可以共享内存
>   * 相⽐prefork占⽤的内存较少，可以同时处理更多的请求
> * 劣势
>   * 不稳定
>   * worker模型和其他应用程序兼用型有问 题
>   * 使⽤keepalive的⻓连接⽅式，某个线程会⼀直被占据，即使没有传输数据，也需要⼀直等待到超时才会被释放。如果过多的线程，被这样占据，也会导致在⾼并发场景下的⽆服务线程可⽤。（该问题在prefork模式下，同样会发⽣）
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019122919433997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rob21hc19DWVE=,size_16,color_FFFFFF,t_70)

### 5、event

> * 预先产生子进程，每个子进程会产生多个线程，由每个线程来处理客户端请求
> * 在长连接、高并发的场景，解决了线程耗尽的问题
> * 属于事件驱动模型(epoll)，每个进程响应多个请求，在现在版本⾥的已经是稳定可⽤的模式。它和worker模式很像，最⼤的区别在于，它解决了 keepalive场景下，长期被占⽤的线程的资源浪费问题（某些线程因为被keepalive，空挂在哪⾥等待，中间⼏乎没有请求过来，甚⾄等到超时）。
>   event MPM中，会有⼀个专⻔的线程来管理这些keepalive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执⾏完毕后，⼜允许它释放。这样增强了⾼并发场景下的请求处理能力。
> * 优点：
>   * 单线程响应多请求，占据更少的内存，高并发下表现更优秀，会有⼀个专门的线程来管理keep-alive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执⾏完毕后，⼜允许它释放 。
> * 缺点
>   * 没有线程安全控制
>
> ![](https://img-blog.csdnimg.cn/20191229194352734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rob21hc19DWVE=,size_16,color_FFFFFF,t_70)

```bash
<IfModule event.c>
StartServers 4 //默认启动子进程数
MaxClients 300 //最大的并发连接数
MinSpareThreads 25 //最小空闲的线程数
MaxSpareThreads 75 //最大空闲的线程数
ThreadsPerChild 25 //每个子进程生成的线程数
MaxRequestsPerChild 100 //每个线程允许处理的最大请求数
</IfModule>
```

## 二、ab压力测试

* **ab常用参数的介绍：**

  -n ：总共的请求执行数，缺省是1；

  -c： 并发数，缺省是1；

  -t：测试所进行的总时间，秒为单位，缺省50000s

  -p：POST时的数据文件

  -w: 以HTML表的格式输出结果

```bash
#
#ab ‐c 100 ‐n 1000 http://vedio.linux.com/index.html
2 This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
3 Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
4 Licensed to The Apache Software Foundation, http://www.apache.org/
5
6 Benchmarking vedio.linux.com (be patient)
7 Completed 100 requests
8 Completed 200 requests
9 Completed 300 requests
10 Completed 400 requests
11 Completed 500 requests
12 Completed 600 requests
13 Completed 700 requests
14 Completed 800 requests
15 Completed 900 requests
16 Completed 1000 requests
17 Finished 1000 requests
18
19
20 Server Software: Apache
21 Server Hostname: vedio.linux.com
22 Server Port: 80
23
24 Document Path: /index.html
25 Document Length: 17 bytes
26
27 Concurrency Level: 100 //并发连接数
28 Time taken for tests: 1.252 seconds //服务器完成所有请求的花费的时间 【重要指标】
29 Complete requests: 1000
30 Failed requests: 0
31 Write errors: 0
32 Total transferred: 268000 bytes //传输了多少字节的数据; 真实数据+http首部
33 HTML transferred: 17000 bytes //传输了多少字节的数据; 真实数据
34 Requests per second: 798.78 [#/sec] (mean) //每秒处理的请求数， 吞吐率;【重要指标】
35 Time per request: 125.190 [ms] (mean) //用户平均请求等待时间，单位ms 【重要指标】
36 Time per request: 1.252 [ms] (mean, across all concurrent requests) //用户请求的处理时间 【重要指标】
37 Transfer rate: 209.06 [Kbytes/sec] received //衡量带宽 【重要指标】
38
39 Connection Times (ms)
40 min mean[+/‐sd] median max
41 Connect: 0 7 11.9 2 56
42 Processing: 4 114 66.7 108 1071
43 Waiting: 2 96 46.0 93 1071
44 Total: 6 121 68.3 111 1104
45
46 Percentage of the requests served within a certain time (ms)
47 50% 111
48 66% 124
49 75% 140
50 80% 151
51 90% 174
52 95% 194
53 98% 267
54 99% 301
55 100% 1104 (longest request)
```

