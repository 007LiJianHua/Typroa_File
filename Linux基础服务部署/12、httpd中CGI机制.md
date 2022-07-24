# CGI机制

[toc]

## 一、CGI机制介绍

> * Nginx不支持对外部动态程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。FastCGI接口在linux下是socket，（这个socket可以是文件socket，也可以是IP socket）。为了调用CGI程序，还需要wrapper绑定在某个固定socket上，如端口或者文件socket。当Nginx将CGI请求发送给这个socket的时候，通过FastCGI接口，warpper接収到请求，然后派出一个新的线程，这个线程调用解释器或者外部程序处理脚本并读取返回数据，接着，wrapper再将返回的数据通过FastCGI接口，沿着固定socket传递给Nginx；最后，nginx将返回的数据发送给客户端，这就是Nginx+fastcgi的整个运行过程。

### 1、流程图

![img](https://s2.loli.net/2022/04/14/XvHCFlmjIxs2kKM.png)

### 2、Nginx访问fastcgi访问PHP的过程

```bash
    1、用户发送http请求报文给nginx服务器

　　2、nginx会根据文件url和后缀来判断请求

　　3、如果请求的是静态内容,nginx会将结果直接返回给用户

　　4、如果请求的是动态内容,nginx会将请求交给fastcgi客户端,通过fastcgi_pass将这个请求发送给php-fpm

　　5、php-fpm会将请求交给wrapper

　　6、wrapper收到请求会生成新的线程调用php动态程序解析服务器

　　7、如果用户请求的是博文、或者内容、PHP会请求MySQL查询结果

　　8、如果用户请求的是图片、附件、PHP会请求nfs存储查询结果

　　9、php会将查询到的结果交给Nginx

　　10、nginx会生成一个响应报文返还给用户
#知识补充：

　　网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。

　　Socket的英文原义是“孔”或“插座”。作为BSDUNIX的进程通信机制，取后一种意思。通常也称作"套接字"，用于描述IP地址和端口，是一个通信链的句柄，可以用来实现不同虚拟机或不同计算机之间的通信。在Internet上的主机一般运行了多个服务软件，同时提供几种服务。每种服务都打开一个Socket，并绑定到一个端口上，不同的端口对应于不同的服务。Socket正如其英文原意那样，像一个多孔插座。一台主机犹如布满各种插座的房间，每个插座有一个编号，有的插座提供220伏交流电，有的提供110伏交流电，有的则提供有线电视节目。客户软件将插头插到不同编号的插座，就可以得到不同的服务。
```



## 二、具体应用

### 1、测试PHP页面

* 在一个虚拟主机下创建test1.php
* 浏览器访问是看不到phpinfo()界面的

```bash
[root@localhost ~]# cat /mp3/test1.php 
AAAAAAAAAAAAA
<?php
   phpinfo();
?>
```

### 2、安装PHP软件

```bash
# yum install -y php
# systemctl restart httpd 
```

* PHP实际上是作为httpd的功能模块存在着的

```bash
[root@localhost ~]# ls /etc/httpd/modules/
libphp5.so 
```

* 再次在浏览器上测试，

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa44a90b69599e96f6edf3f03c67ad5/82](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa44a90b69599e96f6edf3f03c67ad5/82)