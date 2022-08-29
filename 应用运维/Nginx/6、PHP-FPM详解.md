[toc]

## 一、php-fpm与fastcgi的理解

> PHP-FPM(PHP FastCGI Process Manager)意：PHP FastCGI 进程管理器，用于管理PHP 进程池的软件，用于接受web服务器的请求。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置。

### 1、为什么会出现php-fpm

fpm的出现全部因为`php-fastcgi`出现。**为了很好的管理php-fastcgi而实现的一个程序**

### 2、什么是php-fastcgi

`php-fastcgi `只是一个cgi程序,只会解析php请求，并且返回结果，不会管理(因此才出现的php-fpm)。

### 3、为什么不叫php-cgi

其实在php-fastcgi出现之前是有一个php-cgi存在的,只是它的执行效率低下，因此被php-fastcgi取代。

### 4、那fastcgi和cgi有什么区别呢？

* 亲们，这区别就大了，当一个服务web-server(nginx)分发过来请求的时候，通过匹配后缀知道该请求是个动态的php请求，会把这个请求转给php。
* 在cgi的年代，思想比较保守，
  * 总是一个请求过来后,
  * 去读取php.ini里的基础配置信息，
  * 初始化执行环境，
  * 返回数据
  * 提出进程
  * 每次都要不停的去创建一个进程,读取配置，初始化环境，返回数据，退出进程，久而久之，启动进程的工作变的乏味无趣特别累。

当php来到了5的时代，大家对这种工作方式特别反感，想偷懒的人就拼命的想，**我可不可以让cgi一次启动一个主进程(master),让他只读取一次配置，然后在启动多个工作进程(worker),当一个请求来的时候，通过master传递给worker这样就可以避免重复劳动了。于是就产生了fastcgi。**

### 5、fastcgi这么好，启动的worker用完怎么办？

当worker不够的时候，master会通过配置里的信息，动态启动worker，等空闲的时候可以收回worker

### 6、到现在还是没明白php-fpm 是个什么东西?

**就是来管理启动一个master进程和多个worker进程的程序.**

* PHP-FPM 会创建一个主进程，控制何时以及如何将HTTP请求转发给一个或多个子进程处理。PHP-FPM主进程还控制着什么时候创建(处理Web应用更多的流量)和销毁(子进程运行时间太久或不再需要了)
* PHP子进程。PHP-FPM进程池中的每个进程存在的时间都比单个HTTP请求长,可以处理10、50、100、500或更多的HTTP请求。

## 二、php-fpm 安装

### CentOS 6 安装

```bash
PHP-5.3.2：默认不支持fpm机制；需要自行打补丁并编译安装
httpd-2.2：默认不支持fcgi协议，需要自行编译此模块
			
解决方案：编译安装httpd-2.4, php-5.3.3+
```

### CentOS 7

#### 1、安装

```bash
httpd-2.4：rpm包默认编译支持了fcgi模块
php-fpm包：专用于将php运行于fpm模式
```

#### 2、配置文件

```bash
服务配置文件        # /etc/php-fpm.conf,  /etc/php-fpm.d/*.conf（连接池的配置文件）
php环境配置文件     # /etc/php.ini, /etc/php.d/*.ini 
```

#### 3、具体配置

```bash
[root@Neo ~]# cat /etc/php-fpm.conf     # 主配置文件中有关于连接池的配置
;;;;;;;;;;;;;;;;;;;;
; Pool Definitions ; 
;;;;;;;;;;;;;;;;;;;;

; See /etc/php-fpm.d/*.conf              # 连接池的主要配置需要看 /etc/php-fpm.d/*.conf 文件
连接池：
	pm = static|dynamic
		static      # 固定数量的子进程；pm.max_children
		dynamic     # 子进程数据以动态模式管理
			pm.start_servers
			pm.min_spare_servers
			pm.max_spare_servers
			pm.max_requests = 500
```

- **创建session目录，并确保运行php-fpm进程的用户对此目录有读写权限**

```bash
# mkdir  /var/lib/php/session
# chown apache.apache /var/lib/php/session	
```

#### (1) 配置httpd，添加/etc/httpd/conf.d/fcgi.conf配置文件，内容类似：

```bash
# 创建完 fcgi.conf 后，需要添加以下内容
# 主页支持 index.php
# 关闭正向代理
# 反向代理设置：
	# $1 代表前面 () 内的内容
	# Match 进行正则表达式匹配
	# 运行内容是以 .php 结尾的话，是会反代到 fcgi://127.0.0.1:9000的某个配置文件中
	# /var/www/html 也就相当于 fcgi 的 DocumentRoot ，可以变更
	
DirectoryIndex index.php
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/var/www/html/$1
```

#### (2) 虚拟主机配置（有以上配置后，就不需要配置虚拟主机）

```bash
DirectoryIndex index.php

<VirtualHost *:80>
	ServerName www.b.net
	DocumentRoot /apps/vhosts/b.net
	ProxyRequests Off
	ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/apps/vhosts/b.net/$1

	<Directory "/apps/vhosts/b.net">
		Options None
		AllowOverride None
		Require all granted
	</Directory>
</VirtualHost>
```

#### 4、验证

假设fpm的status页面输出URL为/pmstatus，测试接口的输出位置为/ping ，都会送入到 fpm 去进行处理：

```bash
ProxyPassMatch ^/(ping|pmstatus)$  fcgi://127.0.0.1:9000/$1
```

## httpd + php 实现方式

```bash
module: php
fastcgi : php-fpm 
```

### 第一种 `module: php`

直接加载 httpd 模块。

```bash
[root@Neo ~]# httpd -M | grep fcgi
 proxy_fcgi_module (shared)
```

### 第二种 `fastcgi : php-fpm`

直接安装 php-fpm。这种方式适用于负载较大的场景。

```bash
[root@Neo ~]# yum info php-fpm
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Available Packages
Name        : php-fpm
Arch        : x86_64
Version     : 5.4.16
Release     : 46.el7
Size        : 1.4 M
Repo        : base/7/x86_64
Summary     : PHP FastCGI Process Manager
URL         : http://www.php.net/
License     : PHP and Zend and BSD
Description : PHP-FPM (FastCGI Process Manager) is an alternative PHP FastCGI
            : implementation with some additional features useful for sites of
            : any size, especially busier sites
```

## 一、mariadb 相关安装及配置

### 1、安装 `mariadb-server`

```bash
[root@Neo ~]# yum install mariadb-server -y
```

### 2、修改 `/etc/my.cnf.d/server.cnf`

安装完 mariadb 以后，一般都需要默认添加以下两条命令。

```bash
[root@Neo ~]# vim /etc/my.cnf.d/server.cnf 
[mysqld]
skip_name_resolve=ON             # 跳过名称解析
innodb_file_per_table=ON         # innodb 表每个表使用单独的表空间文件
```

### 3、重启 mariadb 服务，并进行查看

```bash
[root@Neo ~]# systemctl start mariadb.service   
[root@Neo ~]# ss -tnl            # 有3306 监听端口代表启动成功
State       Recv-Q Send-Q                         Local Address:Port          Peer Address:Port        
LISTEN      0      50                                         *:3306                     *:*            
LISTEN      0      128                                        *:22                       *:*   
LISTEN      0      100                                127.0.0.1:25                       *:*            
LISTEN      0      32                                        :::21                      :::*     
LISTEN      0      128                                       :::22                      :::*    
LISTEN      0      100                                      ::1:25 
[root@Neo ~]# mysql               # 默认登陆方式也能登陆
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> quit
Bye
```

### 4、加固 mysql 服务器

加固mysql服务器，在安装完成后，运行`mysql_secure_installation`命令。

* 生产环境必做的安全配置向导

```bash
[root@Neo ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):  # 默认空密码，直接回车
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y                         # 是否设定 root 用户的密码，Y 代表设定
New password:                                      # root密码
Re-enter new password:                             # 重新输入root密码
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y                     # 是否移除默认用户，为了安全，肯定需要移除
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y                # 是否禁止 root 用户远程登陆，肯定禁止
 ... Success!                                        # 为了安全，以后只能使用单独授权的用户进行远程登陆

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] n       # 是都需要删除 test 的数据库
 ... skipping.

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y                     # 是否重载授权表，需要重载
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

### 5、加固以后，只能通过输入密码进行登陆

```bash
[root@Neo ~]# mysql            # 默认登陆失效
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
[root@Neo ~]# mysql -uroot -h127.0.0.1 -proot
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

### 6、授权一个普通用户可以登陆 mysql

- 授权一个普通用户可以登陆 mysql 访问 testdb 开头的任意数据库，生产中只授权具体需要访问的数据库
- 授权用户名和密码：`用户 myuser `密码` mypass`
- 只能使用主机地址是 192.168 段的地址进行登陆，生产中使用最小权限原则（只有两台主机允许访问的话，决不允许第三台进行访问）
- 使用新的用户名和密码进行登陆，进行验证

```bash
MariaDB [(none)]> GRANT ALL ON testdb.* TO 'myuser'@'192.168.%.%' IDENTIFIED BY 'mypass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;             # 刷新立马生效
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@Neo ~]# mysql -umyuser -h192.168.1.10 -pmypass
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 12
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> quit
Bye
```

### 7、根据实际情况，设定开机启动 mariadb 服务

```bash
[root@Neo ~]# systemctl enable mariadb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
```

## 二、php-fpm 相关安装及配置

### 1、确认 php 是否安装

- 安装 php-fpm 之前要确认是否安装了 php，php 和 php-fpm 不是同一套服务或组件
- 使用 `yum info php` 进行查看
- `Available Packages` 代表有可用包，但是没安装
- `Installed Packages` 代表已安装

```bash
[root@Neo ~]# yum info php
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Available Packages
Name        : php
Arch        : x86_64
Version     : 5.4.16
Release     : 46.el7
Size        : 1.4 M
Repo        : base/7/x86_64
Summary     : PHP scripting language for creating dynamic web sites
URL         : http://www.php.net/
License     : PHP and Zend and BSD
Description : PHP is an HTML-embedded scripting language. PHP attempts to make it
            : easy for developers to write dynamically generated web pages. PHP also
            : offers built-in database integration for several commercial and
            : non-commercial database management systems, so writing a
            : database-enabled webpage with PHP is fairly simple. The most common
            : use of PHP coding is probably as a replacement for CGI scripts.
            : 
            : The php package contains the module (often referred to as mod_php)
            : which adds support for the PHP language to Apache HTTP Server.
```

### 2、php 常用扩展程序介绍

```bash
[root@Neo ~]# yum list all | grep php  # 查看 php 的程序及所有扩展程序包
[root@Neo ~]# yum info php-mysql       # 连接 mysql 使用的扩展程序
Available Packages
Name        : php-mysql
Arch        : x86_64
Version     : 5.4.16
Release     : 46.el7
Size        : 101 k
Repo        : base/7/x86_64
Summary     : A module for PHP applications that use MySQL databases
URL         : http://www.php.net/
License     : PHP
Description : The php-mysql package contains a dynamic shared object that will add
            : MySQL database support to PHP. MySQL is an object-relational database
            : management system. PHP is an HTML-embeddable scripting language. If
            : you need MySQL support for PHP applications, you will need to install
            : this package and the php package.

[root@Neo ~]# yum info php-mbstring     # 支持多字节，常用于中文数据库
Available Packages
Name        : php-mbstring
Arch        : x86_64
Version     : 5.4.16
Release     : 46.el7
Size        : 505 k
Repo        : base/7/x86_64
Summary     : A module for PHP applications which need multi-byte string handling
URL         : http://www.php.net/
License     : PHP and LGPLv2 and BSD and OpenLDAP
Description : The php-mbstring package contains a dynamic shared object that will add
            : support for multi-byte string handling to PHP.

[root@Neo ~]# yum info php-mcrypt     # 实现加密解密的扩展程序
```

### 3、php-fpm 及相关扩展程序安装

```bash
[root@Neo ~]# yum -y install php-fpm php-mysql php-mbstring 
```

- 查看安装 php-fpm 生成的文件：

  ```bash
  [root@Neo ~]# rpm -ql php-fpm
  /etc/logrotate.d/php-fpm
  /etc/php-fpm.conf             # 主配置文件
  /etc/php-fpm.d
  /etc/php-fpm.d/www.conf       # 模块化配置文件
  /etc/sysconfig/php-fpm
  /run/php-fpm
  /usr/lib/systemd/system/php-fpm.service
  /usr/lib/tmpfiles.d/php-fpm.conf
  /usr/sbin/php-fpm             # php-fpm 主程序
  /usr/share/doc/php-fpm-5.4.16
  /usr/share/doc/php-fpm-5.4.16/fpm_LICENSE
  /usr/share/doc/php-fpm-5.4.16/php-fpm.conf.default
  /usr/share/fpm
  /usr/share/fpm/status.html    # 内置的状态信息该如何输出
  /usr/share/man/man8/php-fpm.8.gz
  /var/log/php-fpm
  ```

### 4、php-fpm 相关配置文件更改

#### 1、主配置文件

主配置文件一般不需要修改。

```bash
[root@Neo ~]# cat /etc/php-fpm.conf     # 主配置文件中有关于连接池的配置
;;;;;;;;;;;;;;;;;;;;
; Pool Definitions ; 
;;;;;;;;;;;;;;;;;;;;

; See /etc/php-fpm.d/*.conf              # 连接池的主要配置需要看 /etc/php-fpm.d/*.conf 文件
```

#### 2、连接池配置文件

修改连接池配置文件时，最好先进行下备份。

```bash
[root@Neo ~]# cd /etc/php-fpm.d/
[root@Neo php-fpm.d]# ls
www.conf
[root@Neo php-fpm.d]# cp www.conf{,.bak}
[root@Neo php-fpm.d]# ls
www.conf  www.conf.bak
```

##### 3、连接池配置文件 www 模块

此模块适用于与 HTTPD 服务进行通信的相关配置。

```bash
[www]
# 有三类通信方式可以进行设置
# 1、监听地址 + 监听端口 
	# listen = 127.0.0.1:9000 只能与本机进行通信
# 2、监听端口
	# 使用本机的任何地址都能进行监听
# 3、使用 UNIX 套接字
	# 直接使用共享内存进行进行通信，不经过协议栈
; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses on a
;                            specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = 127.0.0.1:9000

 # listen.backlog，侯援队列，表示最多可以有多少个请求进行等待， -1 表示无限制
; Set listen(2) backlog. A value of '-1' means unlimited.
; Default Value: -1
;listen.backlog = -1   

# 授权连接，允许来自于哪个地址段的请求进行连接
; List of ipv4 addresses of FastCGI clients which are allowed to connect.
; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
; must be separated by a comma. If this value is left blank, connections will be
; accepted from any ip address.
; Default Value: any
listen.allowed_clients = 127.0.0.1   

# 定义运行连接进程的属主和属组
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
; RPM: apache Choosed to be able to access some dir as httpd
user = apache
; RPM: Keep a group allowed to write in log dir.
group = apache

# pm = static|dynamic   # 选择 pm 的运行方式
#		static          # 固定数量的子进程；pm.max_children
#		dynamic         # 子进程数据以动态模式管理
#           pm.max_children = 50    # 并发数
#			pm.start_servers
#			pm.min_spare_servers
#			pm.max_spare_servers
#			pm.max_requests = 500   # 运行多少次请求时将其进程 kill

; Choose how the process manager will control the number of child processes.
; Possible Values:
;   static  - a fixed number (pm.max_children) of child processes;
;   dynamic - the number of child processes are set dynamically based on the
;             following directives:
;             pm.max_children      - the maximum number of children that can
;                                    be alive at the same time.
;             pm.start_servers     - the number of children created on startup.
;             pm.min_spare_servers - the minimum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is less than this
;                                    number then some children will be created.
;             pm.max_spare_servers - the maximum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is greater than this
;                                    number then some children will be killed.
; Note: This value is mandatory.
pm = dynamic

; The number of child processes to be created when pm is set to 'static' and the
; maximum number of child processes to be created when pm is set to 'dynamic'.
; This value sets the limit on the number of simultaneous requests that will be
; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
; CGI.
; Note: Used when pm is set to either 'static' or 'dynamic'
; Note: This value is mandatory.
pm.max_children = 50

; The number of child processes created on startup.
; Note: Used only when pm is set to 'dynamic'
; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
pm.start_servers = 5

; The desired minimum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.min_spare_servers = 5

; The desired maximum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.max_spare_servers = 35
 
; The number of requests each child process should execute before respawning.
; This can be useful to work around memory leaks in 3rd party libraries. For
; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
; Default Value: 0
;pm.max_requests = 500
其它配置参数：
ping.path = /ping                 # ping 检测
pm.status_path = /pmstatus        # 状态页路径
ping.response = pong              # ping 的回应模式为 pong，能正常回应说明连接正常

# 在动态模式下，对于会话数保持的持久连接配置参数
# mkdir  /var/lib/php/session -pv           # 此目录不存在需要事先创建
# chown apache:apache /var/lib/php/session	# 属主和属组需要修改为运行 HTTPD 服务的属主和属组
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session  
```

### 4、php-fpm 服务启动和检查确认

服务启动后，有 9000 监听端口代表程序服务启动成功。

```bash
[root@Neo php-fpm.d]# systemctl start php-fpm.service
[root@Neo php-fpm.d]# ss -tnl
State       Recv-Q Send-Q                         Local Address:Port           Peer Address:Port   
LISTEN      0      128                                127.0.0.1:9000                      *:* 
LISTEN      0      50                                         *:3306                      *:* 
LISTEN      0      128                                        *:22                        *:*  
LISTEN      0      100                                127.0.0.1:25                        *:*       
LISTEN      0      32                                        :::21                       :::*
LISTEN      0      128                                       :::22                       :::*  
LISTEN      0      100                                      ::1:25                       :::*      
```

### 5、php-fpm 进程说明

```bash
# php-fpm: master process   # 主控进程，运行属主和属组都是 root
# php-fpm: pool www         # 启动5个 php 进程，运行属主和属组是 apache 
[root@Neo php-fpm.d]# ps -aux | grep php
root       8823  0.0  1.0 333356 10560 ?        Ss   23:18   0:00 php-fpm: master process (/etc/php-fpm.conf)
apache     8825  0.0  0.4 333356  4868 ?        S    23:18   0:00 php-fpm: pool www
apache     8826  0.0  0.4 333356  4868 ?        S    23:18   0:00 php-fpm: pool www
apache     8827  0.0  0.4 333356  4868 ?        S    23:18   0:00 php-fpm: pool www
apache     8828  0.0  0.4 333356  4868 ?        S    23:18   0:00 php-fpm: pool www
apache     8829  0.0  0.4 333356  4868 ?        S    23:18   0:00 php-fpm: pool www
root       8859  0.0  0.0 112708   976 pts/0    R+   23:21   0:00 grep --color=auto php
```

## 三、HTTPD 相关安装及配置

### 1、安装 HTTPD

```bash
[root@Neo ~]# yum info httpd | grep Installed
Installed Packages
[root@Neo ~]# systemctl start httpd.service
[root@Neo ~]# ss -tnl
State       Recv-Q Send-Q                   Local Address:Port       Peer Address:Port              
LISTEN      0      128                          127.0.0.1:9000                  *:*                  
LISTEN      0      50                                   *:3306                  *:*                  
LISTEN      0      128                                  *:22                    *:*                  
LISTEN      0      100                          127.0.0.1:25                    *:*                `在这里插入代码片`
LISTEN      0      128                                 :::8080                 :::*                  
LISTEN      0      128                                    :::80                   :::*                  
LISTEN      0      32                                     :::21                   :::*                  
LISTEN      0      128                                    :::22                   :::*                  
LISTEN      0      100                                   ::1:25                   :::* 
```

### 2、确认 fcgi 模块

安装完 HTTPD 后，还要确认 fcgi 模块是否加载。否则无法与 php 进行通信。

```bash
[root@Neo ~]# httpd -M | grep fcgi
 proxy_fcgi_module (shared)
```

### 3、配置虚拟主机（与 php 进行通信）

- **进行 HTTPD 主文件配置**

```bash
# php 动态链接文件放置在与虚拟主机一样的根目录下

[root@Neo ~]# vim /etc/httpd/conf/httpd.conf 
DirectoryIndex index.php

<VirtualHost 192.168.1.10:80>
        ServerName www.neo.com
        DocumentRoot "/data/web/neo"
        ProxyRequests Off
        ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/data/web/neo/$1
        <Directory  "/data/web/neo">
                AllowOverride None
                Options None
                Require all granted
        </Directory>
</VirtualHost>

Listen 80
[root@Neo ~]# httpd -t
Syntax OK
[root@Neo ~]# systemctl restart httpd.service
```

- **添加php-info页面，进行测试**

```bash
[root@Neo neo]# pwd 
/data/web/neo
[root@Neo neo]# ls
index.php  neo.html
[root@Neo neo]# cat index.php 
<?php
	phpinfo()
?>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810163734485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk4MzY1Mw==,size_16,color_FFFFFF,t_70#pic_center)

## 四、安装 phpMyAdmin（示例）

### 1、官网上进行版本下载，注意和本机 php 版本的兼容关系

```bash
https://www.phpmyadmin.net/files/4.0.10.20/
```

### 2、上传至 Linux 服务器，并解压到 HTTPD 的文档根目录下

```bash
[root@Neo neo]# pwd
/data/web/neo
[root@Neo neo]# ls
index.php  neo.html  phpMyAdmin-4.0.10.20-all-languages.tar.gz
[root@Neo neo]# 
[root@Neo neo]# tar -xf phpMyAdmin-4.0.10.20-all-languages.tar.gz 
[root@Neo neo]# ls
index.php  neo.html  phpMyAdmin-4.0.10.20-all-languages  phpMyAdmin-4.0.10.20-all-languages.tar.gz
[root@Neo neo]# ll
total 7048
-rw-r--r--. 1 root root      20 Aug 10 04:30 index.php
-rw-r--r--. 1 root root      32 Aug 10 04:29 neo.html
drwxr-xr-x. 9 root root    4096 Mar 28  2017 phpMyAdmin-4.0.10.20-all-languages
-rw-r--r--. 1 root root 7203314 Aug 10 04:48 phpMyAdmin-4.0.10.20-all-languages.tar.gz
```

### 3、因 php 名字过长，不便于访问，可添加软链接进行访问

```bash
[root@Neo neo]# ln -sv phpMyAdmin-4.0.10.20-all-languages pma
‘pma’ -> ‘phpMyAdmin-4.0.10.20-all-languages’
[root@Neo neo]# ll
total 7048
-rw-r--r--. 1 root root      20 Aug 10 04:30 index.php
-rw-r--r--. 1 root root      32 Aug 10 04:29 neo.html
drwxr-xr-x. 9 root root    4096 Mar 28  2017 phpMyAdmin-4.0.10.20-all-languages
-rw-r--r--. 1 root root 7203314 Aug 10 04:48 phpMyAdmin-4.0.10.20-all-languages.tar.gz
lrwxrwxrwx. 1 root root      34 Aug 10 04:51 pma -> phpMyAdmin-4.0.10.20-all-languages
```

### 4、编辑 config.inc.php 文件

```bash
[root@Neo pma]# ls
browse_foreigners.php  examples               prefs_forms.php            server_status.php            
... ...
db_triggers.php        pmd_relation_upd.php   server_status_advisor.php  tbl_relation.php
doc                    pmd_save_pos.php       server_status_monitor.php  tbl_replace.php

# 一般编辑默认的配置文件时，可以先复制一份进行备份（养成此习惯，保存原文件信息）
[root@Neo pma]# cp config.sample.inc.php config.inc.php 

[root@Neo pma]# vim config.inc.php 
# 在“”里面添加一些随机数即可
$cfg['blowfish_secret'] = 'asdasdadwg58b7c6d'; 

[root@Neo pma]# ll config.inc.php 
-rw-r--r--. 1 root root 3820 Aug 10 04:53 config.inc.php
```

### 5、进行网页访问，提示禁止访问

```bash
http://192.168.1.10/pmabash
```

### 6、再次修改 HTTPD 的主配置文件

```bash
[root@Neo ~]# vim /etc/httpd/conf/httpd.conf 
DirectoryIndex index.php

<VirtualHost 192.168.1.10:80>
        ServerName www.neo.com
        DocumentRoot "/data/web/neo"
        ProxyRequests Off
        ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/data/web/neo/$1
        <Directory  "/data/web/neo">
                AllowOverride None
                Options FollowSymLinks
# 因为我们访问的 pma 是一个软链接文件，所以要允许访问链接文件
                Require all granted
        </Directory>
</VirtualHost>

Listen 80
[root@Neo ~]# httpd -t
Syntax OK
[root@Neo ~]# systemctl restart httpd.service
```

### 7、再次进行访问，可成功访问

**注意：登陆所使用的用户名和密码是在刚才配置 mysql 时指定得，用户名和密码全是 root 。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810170554211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk4MzY1Mw==,size_16,color_FFFFFF,t_70#pic_center)

### 8、至此，HTTPD + PHP +MARIADB 搭建完成

```bash
[root@Neo ~]# php info php-xcache
# 生产环境中可安装此模块，进行 php 加速
```

### 9.可对此站点进行压力测试

```bash
[root@LeeMumu ~]# ab -n 1000 -c 20 http://192.168.1.10/pma
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.1.10 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        Apache/2.4.6
Server Hostname:        192.168.1.10
Server Port:            80

Document Path:          /pma
Document Length:        232 bytes

Concurrency Level:      20
Time taken for tests:   0.167 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Non-2xx responses:      1000
Total transferred:      455000 bytes
HTML transferred:       232000 bytes
Requests per second:    5999.30 [#/sec] (mean)
Time per request:       3.334 [ms] (mean)
Time per request:       0.167 [ms] (mean, across all concurrent requests)
Transfer rate:          2665.71 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   0.3      1       2
Processing:     1    2   0.5      2       4
Waiting:        1    2   0.5      2       4
Total:          2    3   0.5      3       6

Percentage of the requests served within a certain time (ms)
  50%      3
  66%      3
  75%      3
  80%      4
  90%      4
  95%      4
  98%      5
  99%      5
 100%      6 (longest request)
[root@LeeMumu ~]# ab -n 10000 -c 20 http://192.168.1.10/pma
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.1.10 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        Apache/2.4.6
Server Hostname:        192.168.1.10
Server Port:            80

Document Path:          /pma
Document Length:        232 bytes

Concurrency Level:      20
Time taken for tests:   1.531 seconds
Complete requests:      10000
Failed requests:        0
Write errors:           0
Non-2xx responses:      10001
Total transferred:      4550455 bytes
HTML transferred:       2320232 bytes
Requests per second:    6532.13 [#/sec] (mean)
Time per request:       3.062 [ms] (mean)
Time per request:       0.153 [ms] (mean, across all concurrent requests)
Transfer rate:          2902.75 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   0.2      1       3
Processing:     1    2   0.4      2       6
Waiting:        1    2   0.5      2       6
Total:          2    3   0.4      3       6

Percentage of the requests served within a certain time (ms)
  50%      3
  66%      3
  75%      3
  80%      3
  90%      3
  95%      4
  98%      4
  99%      5
 100%      6 (longest request)
```

### 10、整体验证

**修改 HTTPD 的主配置文件：**

```bash
DirectoryIndex index.php

<VirtualHost 192.168.1.10:80>
        ServerName www.neo.com
        DocumentRoot "/data/web/neo"
        ProxyRequests Off
        ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/data/web/neo/$1
        ProxyPassMatch ^/(ping|pmstatus)$  fcgi://127.0.0.1:9000/$1
        <Directory  "/data/web/neo">
                AllowOverride None
                Options FollowSymLinks
                Require all granted
        </Directory>
</VirtualHost>

Listen 80
[root@Neo ~]# httpd -t
Syntax OK
[root@Neo ~]# systemctl restart httpd.service
```

**进行状态页查看：**

```bash
http://192.168.1.10/pmstatus        # 显示php的状态信息

pool:                 www
process manager:      dynamic
start time:           09/Aug/2019:23:18:49 -0400
start since:          22319
accepted conn:        26
listen queue:         0
max listen queue:     0
listen queue len:     128
idle processes:       5
active processes:     1
total processes:      6
max active processes: 5
max children reached: 0
slow requests:        0

http://192.168.1.10/pmstatus?full   # 显示更全的状态信息
http://192.168.1.10/pmstatus?xml    # 以 xml 格式进行输出
http://192.168.1.10/pmstatus?json   #  以 json 格式进行输出
```

**进行 ping 查看：**

```bash
http://192.168.1.10/ping
回显 pong ，代表服务正常
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810173316531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk4MzY1Mw==,size_16,color_FFFFFF,t_70#pic_center)