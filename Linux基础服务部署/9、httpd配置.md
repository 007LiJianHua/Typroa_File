# httpd安装配置

[toc]

## 一、httpd软件介绍

### 1、部署web服务器的软件

* linux平台
  * httpd、nginx、tomcat、
* Windows server
  * IIS

### 2、httpd介绍：

* 构建web服务器
  * 处理静态页面
  * http://httpd.apache.org/

### 3、httpd软件版本

* 2.2
* 2.4

### 4、httpd部署方式

* RPM安装包
* ​    源码编译

### 5、httpd特性

* 开源、跨平台的
* 模块化软件
* 支持虚拟主机功能
* 支持https虚拟主机
* 支持url重写
* 支持缓存

## 二、安装启动httpd

### 1、安装httpd

```bash
[root@localhost ~]# yum install -y httpd 
```

### 2、启动httpd

```bash
[root@localhost ~]# systemctl start httpd
[root@localhost ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@localhost ~]# netstat -antp | grep http
tcp6       0      0 :::80                   :::*                    LISTEN      6860/httpd          
[root@localhost ~]# ps -elf | grep http
```

* 父进程
  * 管理子进程、记录日志、读取配置文件(kill -1 父进程ID)
* 子进程
  * 用于接收、处理客户端请求

### 3、测试访问httpd

```bash
http://192.168.140.10/
```

### 4、删除测试页

```bash
[root@localhost ~]# rm -rf /etc/httpd/conf.d/welcome.conf 
[root@localhost ~]# systemctl restart httpd
```

### 5、建立测试网页

```bash
[root@localhost ~]# ls /var/www/html/
index.html  music.html
```

## 三、httpd相关文件目录

### 1、主配置文件

```bash
/etc/httpd/conf/httpd.conf
```

### 2、子配置文件

```bash
/etc/httpd/conf.d/*.conf
```

### 3、模块路径、配置文件

```bash
/etc/httpd/modules

//httpd服务启动所需要加载的模块
/etc/httpd/conf.modules.d/*.conf 
```

### 4、存放PID文件，主进程ID

```bash
/etc/httpd/run
```

### 5、日志目录

```bash
/var/log/httpd 
访问日志：access_log	
错误日志：error_log
```

### 6、默认网页数据目录

```bash
/var/www/html
```

## 四、配置文件解析----httpd.conf

### 1、配置文件解析

```bash
31 ServerRoot "/etc/httpd"		//指定httpd的工作目录

42 Listen 8888			//指定监听的端口

56 Include conf.modules.d/*.conf		//加载子配置文件

66 User apache
67 Group apache				//指定启动进程的用户身份

86 ServerAdmin root@localhost		//指定管理员邮箱

96 ServerName test.linux.com		//指定网站的主机名

120 DocumentRoot "/var/www/html"		//指定网页目录

164 <IfModule dir_module>		//指定默认的网页名称
165     DirectoryIndex index.html
166 </IfModule>

183 ErrorLog "logs/error_log"		//错误日志文件、级别
187 # Possible values include: debug, info, notice, warn, error, crit,
188 # alert, emerg.
190 LogLevel error

218     CustomLog "logs/access_log" combined		//访问日志名称、记录模式
		
		web服务器访问量由两个指标衡量:
            1、PV	Page View 页面访问量
            2、UV	User View 用户访问量
            
197     LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined			//定义访问日志的格式
				变量说明: 
                                %h 		客户端地址
                                %l		web认证用户
                                %u		windows用户
                                %t		访问时间
                                %r		请求方法  文件名称  HTTP协议版本
                                %s		状态码
                                %b		响应数据的大小 
                                %{Referer}i	超链接地址	
                                %{User-Agent}i   浏览器类型
                                
KeepAliveTimeout 5			//长连接配置
MaxKeepAliveRequests 100
```



```bash
132 <Directory "/var/www/html">		//针对目录授权	
145     Options  FollowSymLinks		

152     AllowOverride None

157     Require all granted
158 </Directory>

//针对目录配置说明

```

> * 配置说明
>
>   * options
>
>   * 1. 客户端访问对应目录下的文件时受什么影响 
>
>     2. 1. Indexes存在时
>           1. 有index.html首页时，正常访问，
>           2. 无index.html首页时，会列出所有文件 
>        2. Indexes不存在时
>           1. 有index.html首页时，更安全
>
>   * 2. FollowSymLinks
>
>        允许网页目录下的软连接文件被正常访问
>
>   * require 客户端访问认证 （**以下任何一种类型只能出现一次**）
>
>     * 基于客户端地址进行认证
>
>     ```bash
>     Require all granted			//允许所有客户端访问
>     
>     Require ip 10.1.1.1			//仅允许10.1.1.1访问
>     
>     <RequireAll>				//允许所有人访问，除了10.252.46.165
>         Require all granted
>         Require not ip 10.252.46.165
>     </RequireAll>
>     
>     Require all denied
>     ```
>
>     
>
>     * 基于用户名、密码进行认证
>       * 创建认证用户
>        
>     ```bash
>     [root@localhost ~]# htpasswd -c /etc/httpd/.webuser martin
>     New password: 
>     Re-type new password: 
>     Adding password for user martin
>     [root@localhost ~]# 
>     [root@localhost ~]# cat /etc/httpd/.webuser 
>     martin:$apr1$6KV5I5w.$6iQ5ip.1bF3la2pHq9lX4/
>     ```
>        
>     * 编辑httpd.conf
>        
>     ```bash
>     [root@localhost ~]# vim /etc/httpd/conf/httpd.conf 
>             
>     <Directory "/var/www/html">
>         ........................
>         AuthType Basic
>         AuthName "Need to login: "
>         AuthUserFile "/etc/httpd/.webuser"
>         Require valid-user
>     </Directory>
>     ```
>        
>     * 重启服务