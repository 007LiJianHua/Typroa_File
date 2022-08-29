[toc]

## 一、作用：

> * 在使用php语言时，大部分依托于LNMP或者LAMP平台架构，
>
> * 而在使用java语言时，可以使用tomcat来作为java应用程序的中间键，
>
> * 动态网站:
>
>   PHP语言   *.php  依托于lnmp平台 
>
>   Java语言  *.jsp  依托于tomcat平台

## 二、Tomcat特性

* 开源的、跨平台
* 支持多实例的部署方式
* 支持https
* 官网：https://tomcat.apache.org/

## 三、Tomcat的安装部署

### 1、安装JDK

```bash
[root@localhost ~]# wget https://download.oracle.com/java/17/latest/jdk-17_linux-aarch64_bin.tar.gz
[root@localhost ~]# tar xf jdk-17_linux-aarch64_bin.tar.gz -C /usr/local/
[root@localhost ~]# ls /usr/local/
bin  etc  games  include  jdk1.8.0_91  lib  lib64  libexec  sbin  share  src
[root@localhost ~]# tail -n 2 /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
[root@localhost ~]# source /etc/profile
[root@localhost ~]# java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

### 2、安装Tomcat

```bash
[root@localhost ~]# wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.17/bin/apache-tomcat-10.0.17.tar.gz
[root@localhost ~]# tar xf apache-tomcat-8.5.12.tar.gz -C /usr/local/
[root@localhost ~]# mv /usr/local/apache-tomcat-8.5.12/ /usr/local/tomcat85
[root@localhost ~]# ls /usr/local/
bin  etc  games  include  jdk1.8.0_91  lib  lib64  libexec  sbin  share  src  tomcat85
```

### 3、定义CATALINA_HOME环境变量

```bash
[root@localhost ~]# sed -ri '$a \export CATALINA_HOME=/usr/local/tomcat85' /etc/profile
[root@localhost ~]# source /etc/profile
```



### 4、tomcat相关的目录

* tomcat安装目录
  * bin：一系列的命令
  * conf/server.xml：主配置文件
  * logs：日志目录
  * webapps：数据目录
  * lib：库文件目录

### 5、启动tomcat

* 端口说明
  * 8009：提供tomcat做httpd反向代理的端口
  * 8080：提供http服务的端口
  * 8005：tomcat实例端口

```bash
[root@localhost ~]# /usr/local/tomcat85/bin/catalina.sh start
Using CATALINA_BASE:   /usr/local/tomcat85
Using CATALINA_HOME:   /usr/local/tomcat85
Using CATALINA_TMPDIR: /usr/local/tomcat85/temp
Using JRE_HOME:        /usr/local/jdk1.8.0_91
Using CLASSPATH:       /usr/local/tomcat85/bin/bootstrap.jar:/usr/local/tomcat85/bin/tomcat-juli.jar
Tomcat started.
[root@localhost ~]# netstat -antp | grep java
tcp6       0      0 :::8009                 :::*                    LISTEN      7088/java           
tcp6       0      0 :::8080                 :::*                    LISTEN      7088/java           
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      7088/java  
```

### 6、tomcat开机自启

* 在系统开机启动的时候，会先读取rc.local文件，再读取profile文件，这样会导致设置在profile的环境变量加载不上，所以要把加载环境变量放到rc.local文件中

```bash
[root@localhost ~]# vim /etc/rc.d/rc.local 
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin

/usr/local/tomcat85/bin/catalina.sh start
[root@localhost ~]# chmod a+x /etc/rc.d/rc.local
```

## 四、Tomcat配置解析

### 1、项目目录

默认的项目目录：/webapps

* 注意：一个项目目录中可以部署多个项目
  * 默认情况下tomcat会访问ROOT这个项目

### 2、配置文件---server.xml

* 配置tomcat实例端口

```bash
<Server port="8005" shutdown="SHUTDOWN">
```

* 提供http服务的端口

```bash
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

* 配置虚拟主机

  * name	**虚拟主机名称**

  * appBase	**项目目录**

  * <Context path="/AA" docBase="hello" />

  * ​	项目的访问别名 

  * - path=	别名，空表示通过主机名访问  
    - docBase=	项目路径，相对于appBase写

不写别名浏览器访问的时候输入 http://192.168.152.10:8080/project

写了别名浏览器访问可输入：http://192.168.152.10:8080/pro

```bash
      <Host name="localhost"  appBase="/test"
            unpackWARs="true" autoDeploy="true">
        <Context path="/pro" docBase="profile" />
```

* tomcat访问日志名字及格式
  * directory=""		//访问日志存放路径 
  * prefix=""			//访问日志的开头
  * suffix=""			//定义访问日志的结尾 
  * patter=""			//访问日志格式  
  * tomcat与apache日志变量一样，可以自己添加一些。

```bash
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

* 定义默认的虚拟主机

```bash
    <Engine name="Catalina" defaultHost="localhost">
```

## 五、Tomcat多实例部署

### 1、环境描述

```bash
安装目录: /opt/tomcat01	实例端口: 8006	HTTP服务: 9000	项目目录: /opt/tomcat01/webapps
安装目录: /opt/tomcat02	实例端口: 8007	HTTP服务: 9001	项目目录: /opt/tomcat02/webapps
```

### 2、规划目录

* 每一个实例都必须要有这四个目录

```bash
[root@localhost ~]# mkdir /opt/tomcat0{1,2}/{conf,logs,webapps} -p
```

### 3、复制实例配置文件

```bash
[root@localhost ~]# cp -r /usr/local/tomcat85/conf/* /opt/tomcat01/conf/
[root@localhost ~]# cp -r /usr/local/tomcat85/conf/* /opt/tomcat02/conf/
```

### 4、编辑实例配置文件

```bash
[root@localhost ~]# vim /opt/tomcat01/conf/server.xml 
#避免端口重复，使用不同的端口
<Server port="8006" shutdown="SHUTDOWN">
    <Connector port="9000" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
      <Host name="localhost"  appBase="/opt/tomcat01/webapps"
            unpackWARs="true" autoDeploy="true">
        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="/opt/tomcat01/logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

* 其他实例参考上述文件

### 5、多个实例如何启动

* 利用CATALINA_BASE变量，这个变量会自动的找  安装目录/conf/server.xml
* 由此来区别不同的实例
* 可编写脚本

```bash
[root@localhost ~]# cat /opt/tomcat01/daemon.sh
#!/bin/bash
#
export CATALINA_HOME=/usr/local/tomcat85
# 指定启动时查找配置文件的路径
export CATALINA_BASE=/opt/tomcat01
case $1 in 
   start)
       $CATALINA_HOME/bin/startup.sh
       ;;
   stop)
       $CATALINA_HOME/bin/shutdown.sh
       ;;
   restart)
       $CATALINA_HOME/bin/shutdown.sh
       sleep 1
       $CATALINA_HOME/bin/startup.sh
       ;;
esac

[root@localhost ~]# chmod a+x /opt/tomcat01/daemon.sh
```

### 6、启动

```bash
[root@localhost ~]# /opt/tomcat01/daemon.sh start
[root@localhost ~]# /opt/tomcat02/daemon.sh start
[root@localhost ~]# netstat -antp | grep java
tcp6       0      0 127.0.0.1:8007          :::*                    LISTEN      18336/java          
tcp6       0      0 :::9000                 :::*                    LISTEN      18297/java          
tcp6       0      0 :::9001                 :::*                    LISTEN      18336/java          
tcp6       0      0 127.0.0.1:8006          :::*                    LISTEN      18297/java   
```

## 六、实现nginx负载均衡与反向代理

### **反向代理**

#### 1、环境描述

* 接着上面操作

```bash
192.168.152.10 tomcat服务器
192.168.152.11	nginx服务器
```

#### 2、在nginx端配置

```bash
#这个location写在默认的虚拟主机
root /etc # vim /etc/nginx/nginx.conf
        location /sj {
                proxy_pass http://192.168.152.10:8080/phone_war;
                proxy_set_header X-REAL-IP $remote_addr;
        }
        location /pro {
                proxy_pass http://192.168.152.10:8081/project;
                proxy_set_header X-REAL-IP $remote_addr;
        }
root /etc # systemctl restart nginx.service
```

在windows浏览器输入 http://192.168.152.11/sj   和 http://192.168.152.11/pro  来访问这两个项目文件

负载

### 实现对phone_war项目的负载均衡

#### 1、环境描述

```bash
192.168.152.10   tomcat01   phone_war
192.168.152.12	 tomcat02	phone_war
192.168.152.11	 nginxf反向代理   
192.168.152.11:8000  当两个服务器全部down机，由nginx的web服务来提示两个tomcat服务器都不可用 
```



#### 2、在http全局配置模块添加upstream模块

```bash
root /etc # vim /etc/nginx/nginx.conf


http {
...
	upstream sjus {
        server 192.168.152.10:8081 weight=1 max_fails=2  fail_timeout=2;
        server 192.168.152.12:8081 weight=1 max_fails=2  fail_timeout=2;
        server 127.0.0.1:8000 backup;
    }
...
	server{
	....
	location /sj {
                proxy_pass http://sjus/phone_war;
                proxy_set_header X-REAL-IP $remote_addr;
                }

	}
    server {
        listen 8000;
        server_name localhost;

        location / {
                root /test;
                index   index.html;
        }
    }
}

root /etc # cat /test/phone_war/index.html 
<h1>All tomcat is down</h1>

root /etc # systemctl restart nginx.service 
```

#### 3、12的tomcat服务器配置与10一样，不再赘述

* 为了显示负载均衡，修改了项目文件的网页名子‘



#### 4、测试

在浏览器输入http://192.168.152.11/sj   多点几次，会发现首页名字不停的切换，

将两个tomcat停掉，测试