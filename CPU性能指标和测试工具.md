[toc]

### 首先确定cpu性能的指标有哪些

###### 1.cpu使用率

- 用户cpu使用率
- 系统使用率
- 等待I/O的cpu使用率
- 软中断和硬中断的cpu使用率

###### 2.平均负载

即系统的平均活跃进程数。理想情况下，平均负载等于逻辑cpu个数，表示每个cpu都被使用到。

###### 3.进程上下文切换

- 无法获取资源而导致的自愿上下文切换
- 被系统强制调度导致的非自愿上下文切换

###### 4.cpu缓存命中率

命中率表示了cpu缓存的复用情况，命中率越高，性能越好。

### 对应的性能工具有哪些

###### 1.top

```bash
$ top
top - 22:26:51 up 2 days,  5:19,  1 user,  load average: 2.85, 2.50, 2.34
Tasks: 288 total,   2 running, 234 sleeping,   0 stopped,   0 zombie
%Cpu(s): 18.8 us,  7.0 sy,  0.0 ni, 69.4 id,  4.7 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem :  7868584 total,   443100 free,  3107988 used,  4317496 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  3629296 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND

 1581 root      20   0  424244  90788  53076 S  10.0  1.2   2:49.44 Xorg
```

注意第一行最右边的数据，

```bash
load average后面三个数据，分别表示过去1分钟、5分钟、15分钟的平均负载数据
```

注意第三行数据，

```bash
%us：表示用户空间程序的cpu使用率（没有通过nice调度）
%sy：表示系统空间的cpu使用率，主要是内核程序。
%ni：表示用户空间且通过nice调度过的程序的cpu使用率。
%id：空闲cpu
%wa：cpu运行时在等待io的时间
%hi：cpu处理硬中断的数量
%si：cpu处理软中断的数量
%st：被虚拟机偷走的cpu
```

###### 2.uptime

```bash
$ uptime
 22:32:03 up 2 days,  5:25,  1 user,  load average: 1.27, 1.88, 2.14
```

load average后面三个数据，也是分别表示过去1分钟、5分钟、15分钟的平均负载数据

###### 3.mpstat

每个1秒显示数据，一共显示3组数据。

```bash
$ mpstat 1 3
Linux 4.15.0-115-generic (buaa) 	2020年09月15日 	_x86_64_	(4 CPU)

22时33分32秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
22时33分33秒  all   11.53    0.00    7.52    0.00    0.00    0.00    0.00    0.00    0.00   80.95
22时33分34秒  all    8.85    0.00    7.55    0.78    0.00    0.00    0.00    0.00    0.00   82.81
22时33分35秒  all   12.15    0.00    7.09    0.00    0.00    0.00    0.00    0.00    0.00   80.76
Average:     all   10.87    0.00    7.39    0.25    0.00    0.00    0.00    0.00    0.00   81.49
```

注意每个字段的含义：

```bash
%user		在间隔时间段里，用户态的CPU时间(%)，不包含nice值为负进程
%nice		在间隔时间段里，nice值为负进程的CPU时间(%)
%sys		在间隔时间段里，内核时间(%)
%iowait		在间隔时间段里，硬盘IO等待时间(%)
%irq		在间隔时间段里，硬中断时间(%)
%soft		在间隔时间段里，软中断时间(%)
%idle		在间隔时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%)
```

mpstat -P 0表示显示cpu0的数据
mpstat -P ALL显示每个cpu的数据。

###### 4.vmstat

主要用来分析系统的内存使用情况，也常用来分析 CPU 上下文切换和中断的次数

```bash
$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 404408 570356 3796212    0    0   171   934 1587 1520 27  8 59  7  0
```

字段含义：

```bash
r		绪队列的长度，也就是正在运行和等待 CPU 的进程数
b		处于不可中断睡眠状态的进程数
swpd	正在使用虚拟内存的大小，单位k
free	空闲内存大小
buff	已使用的buffer大小
cache	已使用的cache大小
si		每秒从交换区写入内存的大小（单位：kb/s）
so		每秒从内存写到交换区的大小
bi		每秒读取的块数（读磁盘）
bo		每秒写入的块数（写磁盘）
in		每秒中断数
cs		每秒上下文切换的次数
us		用户进程执行消耗cpu时间
sy		系统进程消耗cpu时间
id		空闲时间(包括IO等待时间)
wa		等待IO时间
st		从虚拟机窃取的时间
```

###### 5.pidstat

vmstat 只给出了系统总体的上下文切换情况，要想查看每个进程的详细情况需要使用pidstat。
下面指定进程号为805的进程数据，每个1秒显示数据，一共显示3组数据。

```bash
$ pidstat -p 805 1 3
Linux 4.15.0-115-generic (buaa) 	2020年09月15日 	_x86_64_	(4 CPU)

22时43分04秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
22时43分05秒     0       805    2.00    0.00    0.00    2.00     2  kubelet
22时43分06秒     0       805    2.00    3.00    0.00    5.00     2  kubelet
22时43分07秒     0       805    5.00    2.00    0.00    7.00     2  kubelet
Average:        0       805    3.00    1.67    0.00    4.67     -  kubelet
```

注意每个字段的含义：

```bash
PID：		进程ID
%usr：		进程在用户空间占用cpu的百分比
%system：	进程在内核空间占用cpu的百分比
%guest：	进程在虚拟机占用cpu的百分比
%CPU：		进程占用cpu的百分比
CPU：		处理进程的cpu编号
```

使用-w 选项，查看进程上下文切换的情况

```bash
pidstat -p 547 -w
Linux 4.15.0-115-generic (buaa) 	2020年09月15日 	_x86_64_	(4 CPU)

22时58分38秒   UID       PID   cswch/s nvcswch/s  Command
22时58分38秒  1000       547      3.51      0.94  MainThread
```

字段含义：

```bash
cswch	表示每秒自愿上下文切换（voluntary context switches）的次数
nvcswch	表示每秒非自愿上下文切换（non voluntary context switches）的次数
```

###### 6.execsnoop

在实际运行环境中，可能会遇到系统的CPU使用率和系统平均负载很高，但却找不到导致指标很高的应用。需要考虑产生这个问题的原因：进程有可能在不断的崩溃、重启。execsnoop是专门用于为追踪短时进程（瞬时进程）设计的工具，通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括进程 PID、父进程 PID、命令行参数以及执行的结果。

###### 7.perf

安装

```bash
sudo apt install linux-tools-4.15.0-117-generic
```

perf主要有两种常用使用方式：
a.实时查看占用 CPU 时钟最多的函数或者指令，因此可以用来查找热点函数。

```bash
$ perf top
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091622012956.png#pic_center)字段含义：

```bash
第一行中有三个数据，分别表示示采样数、事件类型和事件总数。
下面列字段的含义：
 - Overhead ，是该符号的性能事件在所有采样中的比例，用百分比来表示
 - Shared ，是该函数或指令所在的动态共享对象（Dynamic Shared Object），如内核、进程名、动态链接库名、内核模块名等
 - Object ，是动态共享对象的类型。比如 [.] 表示用户空间的可执行程序、或者动态链接库，而 [k] 则表示内核空间
 - Symbol 是符号名，也就是函数名。当函数名未知时，用十六进制的地址来表示
123456
```

b.使用perf record记录，再使用perf report解析展示结果
先使用perf record -g开启调用关系的采样。记录一定时间后，使用ctrl+c停止record。

```bash
$ sudo perf record -g
^C[ perf record: Woken up 11 times to write data ]
[ perf record: Captured and wrote 4.248 MB perf.data (22856 samples) ]
123
```

再使用perf report查看报告

```bash
$ sudo pref report
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918215658611.png#pic_center)