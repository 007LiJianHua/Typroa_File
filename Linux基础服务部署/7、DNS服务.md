# DNS服务

[toc]

## 一、DNS介绍

* DNS应用层协议
* Domain Name System    域名系统
* 端口
  * TCP/53
    * 配置DNS主从，保证数据的可靠性传输
  * UDP/53
    * 正常的客户端来请求服务器使用的端口，
* 作用
  * 正向解析
    * 根据主机名解析查询对应的IP
  * 反向解析
    * 根据IP查询对应的主机名
* DNS服务器层级的概念
* 一级域有七个：
  * 包括用于科研机构的ac；
  * 用于工商金融企业的com；
  * 用于教育机构的edu；
  * 用于政府部门的gov；
  * 用于互联网络信息中心和运行中心的net；
  * 用于非盈利组织的org；
  * 国家顶级域名 cn；

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE5496243871ac501d6c102d311e23dd6e/62](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE5496243871ac501d6c102d311e23dd6e/62)

### 1、区域的概念

* 正向区域
  * 一般就是二级域名
    * jd.com
    * baidu.com
    * taobao.com
* 反向区域
  * x.x.x.in-addr.arpa
    * 1.168.192.in-addr-arpa
    * 1.1.10.in-addr-arpa

### 2、记录文件Record

* A记录

  * 记录主机名与IP地址的对应关系
  * www.baidu.com A 3.3.3.3

* PTR记录

  * 反向指针记录

  * 记录IP地址、主机名的对应关系

    10.168.192.in-addr.arpa

    ​	8	PTR  test1.linux.com

    ​	12	PTR test2.linux.com

* CNAME记录

  * 别名记录
  * www.baidu.com    CNAME   www.a.shifen.com

* MX记录

  * 标识邮件服务器

  * **必须配合A记录同时使用**

    MX	5	mail0.linux.com

    mail0.linux.com	A	5.5.5.5

    MX	6	mail1.linux.com

    mail1.linux.com	A	6.6.6.6

    数字5：邮件服务器的优先级、数字越小、优先级越高

* NS记录

  * 标识DNS服务器

  * **必须配合A记录同时使用**

    NS		ns01.linux.com

    ns01.linux.com	A	6.6.6.6

    

### 3、DNS服务端

* 软件
  * bind、bind-chroot
    * bind-chroot可增强系统安全性
* 配置文件
  * 主配置文件
    * /var/named/chroot/etc/named.conf		**//主配置文件主要用来   指定记录文件的存放位置+指定区域名字所对应的文件记录名称**
  * 数据记录文件
    * /var/named/chroot/var/named/*
      * *：代表的是所指定的区域名字对应的文件记录名称

## 二、配置DNS实现正向解析

### 1、要求：

​	二级域名： linux.com 

DNS服务器	192.168.140.10	ns01.linux.com

FTP服务器		10.1.1.1		ftpserver.linux.com 

WEB服务器		10.1.1.2		web.linux.com

MAIL服务器		10.1.1.3		mail01.linux.com

### 2、关闭防火墙、SElinux、时间同步服务器打开

### 3、安装DNS相关软件

```bash
[root@dns_server ~]# yum install bind bind-chroot 
//1、BIND配置文件保存在两个位置：
　　　　/etc/named.conf　　- BIND服务主配置文件
　　　　/var/named/　　　　- zone文件（域的dns信息）

　　如果安装了bind-chroot（其中chroot是 change root 的缩写），BIND会被封装到一个伪根目录内，配置文件的位置变为：
　　　　/var/named/chroot/etc/named.conf 　　- BIND服务主配置文件
　　　　/var/named/chroot/var/named/　　　　- zone文件
　　chroot是通过相关文件封装在一个伪根目录内，已达到安全防护的目的，一旦程序被攻破，将只能访问伪根目录内的内容，而不是真实的根目录

　　2、BIND安装好之后不会有预制的配置文件，但是在BIND的文档文件夹内（/usr/share/doc/bind-9.9.4），BIND为我们提供了配置文件模板，我们可以直接拷贝过来：
　　　　cp -r /usr/share/doc/bind-9.9.4/sample/etc/* /var/named/chroot/etc/
　　　　cp -r /usr/share/doc/bind-9.9.4/sample/var/* /var/named/chroot/var/
```

### 4、在主配置文件中创建正向区域

```bash
[root@dns_server ~]# vim /var/named/chroot/etc/named.conf
options {
    directory "/var/named";      //指定记录文件的存放位置 
};

zone "linux.com" {             // 指定正向区域名
    type master;               // 指定区域类型为主区域
    file "linux.com.zone";     // 指定linux.com区域对应的记录文件名称，这个文件的位置就是上边指定的位置
};
```

### 5、创建记录文件、添加记录

```bash
//存在记录文件模板，复制粘贴即可
[root@dns_server ~]# cp /usr/share/doc/bind-9.11.4/sample/var/named/named.localhost  /var/named/chroot/var/named/linux.com.zone	
[root@dns_server ~]# vim /var/named/chroot/var/named/linux.com.zone
$TTL 1D
@	IN SOA	linux.com. 454452000.qq.com. (		//其中linux.com
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	ns01.linux.com.
ns01	A	192.168.140.10
ftpserver	A	10.1.1.1
web		A	10.1.1.2
	MX  5	mail01.linux.com.
mail01	A	10.1.1.3
```

### 6、启动DNS服务

```bash
[root@dns_server ~]# systemctl start named  named-chroot 
[root@dns_server ~]# systemctl enable named named-chroot

//进程和端口都能查到即可
[root@dns_server ~]# ps -elf | grep name
[root@dns_server ~]# netstat -tunlp | grep name
```

### 7、验证DNS---nslookup工具

```bash
C:\>nslookup
默认服务器:  gjjline.bta.net.cn
Address:  202.106.0.20

> server 192.168.140.10
默认服务器:  [192.168.140.10]
Address:  192.168.140.10

> ftpserver.linux.com
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    ftpserver.linux.com
Address:  10.1.1.1

> web.linux.com
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    web.linux.com
Address:  10.1.1.2

> mail01.linux.com
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    mail01.linux.com
Address:  10.1.1.3
```

## 三、配置DNS实现反向解析

### 1、添加反向域

```bash
//反向域和正向域在同一个named.conf文件中，用不同的zone隔开
[root@dns_server named]# vim /var/named/chroot/etc/named.conf

zone "linux.com" {             
    type master;               
    file "linux.com.zone";     
    
zone "1.1.10.in-addr.arpa" {
    type master;
    file "10.1.1.zone";
};

```

### 2、添加记录文件

```bash
//直接将正向域的记录文件拷贝修改即可
[root@dns_server named]# cp linux.com.zone 10.1.1.zone
[root@dns_server named]# vim 10.1.1.zone
$TTL 1D
@	IN SOA	linux.com. 454452000.qq.com. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	ns01.linux.com.
ns01	A	192.168.140.10
1	PTR	ftpserver.linux.com.
2	PTR	web.linux.com.
3	PTR	mail01.linux.com.
```

### 3、重启并验证反向解析

```bash
[root@dns_server named]# systemctl restart named named-chroot
C:\Users\admin>nslookup
默认服务器:  gjjline.bta.net.cn
Address:  202.106.0.20

> server 192.168.140.10
默认服务器:  [192.168.140.10]
Address:  192.168.140.10

>
> 10.1.1.1
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    ftpserver.linux.com
Address:  10.1.1.1

> 10.1.1.2
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    web.linux.com
Address:  10.1.1.2

> 10.1.1.3
服务器:  [192.168.140.10]
Address:  192.168.140.10

名称:    mail01.linux.com
Address:  10.1.1.3

> exit
```

## 四、测试DNS的常用工具

* nslookup

* dig

  * dig需要在网卡指定DNS服务器地址

    ```bash
    [root@localhost ~]# dig -t A mail01.linux.com
    ```



## 五、配置DNS主从

### 1、环境描述

* 192.168.152.10	主DNS
* 192.168.152.11    备DNS

### 2、两台服务器关闭防火墙、SELinux、时间同步

### 3、修改主DNS配置文件、添加从服务器地址

```bash
options {
        directory "/var/named";
};

zone "linux.com" {
        type master;
        file "linux.com.zone";
        allow-transfer { 192.168.152.11; };
};

zone "1.1.10.in-addr.arpa" {
        type master;
        file "10.1.1.zone";
};
```

### 4、修改主DNS配置文件，添加主、从两台服务器NS记录

```bash
[root@dns_server ~]# vim /var/named/chroot/var/named/linux.com.zone 

        NS      ns01.linux.com.
        NS      ns02.linux.com.
ns01    A       192.168.140.10
ns02    A       192.168.140.11
[root@dns_server ~]# systemctl restart named named-chroot
```

### 5、在从服务器安装DNS软件

```basj
[root@localhost ~]# yum install bind bind-chroot 
```

### 6、在从服务器上创建相同的区域

```bash
[root@localhost ~]# cat /var/named/chroot/etc/named.conf
options {
   directory "/var/named";
};

zone "linux.com" {
    type slave;         //指定区域类型为从，代表要从主服务器上复制记录文件,不能自己修改
    file "slaves/linux.com.zone";     //记录文件的存放位置 
    masters { 192.168.140.10; };     // 指定主服务器地址
};
```



### 7、验证主从同步成功

```bash
[root@localhost ~]# ls /var/named/chroot/var/named/slaves/
linux.com.zone
```

* 测试使用从服务器解析：

```bash
C:\Users\admin>nslookup
默认服务器:  gjjline.bta.net.cn
Address:  202.106.0.20

>
> server 192.168.140.11
默认服务器:  [192.168.140.11]
Address:  192.168.140.11

>
> ftpserver.linux.com
服务器:  [192.168.140.11]
Address:  192.168.140.11

名称:    ftpserver.linux.com
Address:  10.1.1.1

> web.linux.com
服务器:  [192.168.140.11]
Address:  192.168.140.11

名称:    web.linux.com
Addresses:  10.1.1.2
          10.1.1.10

> mail01.linux.com
服务器:  [192.168.140.11]
Address:  192.168.140.11

名称:    mail01.linux.com
Address:  10.1.1.3
```

### 8、测试同步变化的记录信息

```bash
[root@dns_server ~]# cat /var/named/chroot/var/named/linux.com.zone 
$TTL 1D
@	IN SOA	linux.com. 454452000.qq.com. (
					37	; serial 
                                     // 序列号，让从服务器识别记录变化
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	ns01.linux.com.
	NS	ns02.linux.com.
ns01	A	192.168.140.10
ns02	A	192.168.140.11
AA	A	10.1.1.100
BB	A	10.1.1.200
```



## 六、DNS查询类型

### 1、递归查询

* 只发送一次DNS查询请求
* 客户端和DNS服务器间

### 2、迭代查询

* 发送多次DNS请求
* 多个不同的DNS服务器间发送请求

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/F5ACD984C25B44E1A9C568F0D0FECA72/EC6B2EF716574FDEBF4745352D6068B5/25803)

### 3、本地hosts文件

* 记录主机名、IP地址的对应关系
* 格式
  * IP地址    主机名



Linux：

​	/etc/hosts

Windows

​	C:\Windows\System32\drivers\etc