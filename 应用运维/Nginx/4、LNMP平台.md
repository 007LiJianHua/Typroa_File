[toc]

## 一、源码安装

[吴老板源码安装笔记]: https://note.youdao.com/ynoteshare/index.html?id=2dd243e3793f68efa7840971eb8d1a90&amp;type=notebook&amp;_time=1646707554037#/D9896554BA264CA28EC57F243791CC75

## 二、软件仓库安装

> * 名称
>   * linux+nginx+mysql+php
>   * 部署php动态应用
> * 与LAMP平台区别
>   * php支持以php-fpm的方式运行
>     * php作为独立的应用程序、进程、端口
>     * tcp/9000
>   * nginx通过fastcgi机制调用php，完成php动态页面的处理

### 1、安装部署

* 下载nginx，php需要的epel源

```bash
[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo
[root@localhost ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

* 安装平台相关软件

```bash
 [root@localhost ~]# yum install nginx mariadb-server php-fpm php-mysql gd php-gd php-devel php-curl php-intl php-mcrypt php-mbstring php-xml php-dom
```

### 2、平台优化

* 修改php配置文件来提高php性能

```bash
[root@localhost ~]# rpm -qc php-fpm
/etc/logrotate.d/php-fpm     
/etc/php-fpm.conf     #php主配置文件
/etc/php-fpm.d/www.conf      #php与网页相关的配置文件
/etc/sysconfig/php-fpm
[root@localhost ~]# vim /etc/php-fpm.d/www.conf 

listen = 127.0.0.1:9000
pm.max_children = 50			// 允许启动的最多的进程数
pm.start_servers = 8			//启动时，默认启动的进程数
pm.min_spare_servers = 8		//最小空闲进程数
pm.max_spare_servers = 16		//最大空闲进程数
pm.max_requests = 4096
```

* 修改php的错误日志级别

```bash
[root@localhost ~]# vim /etc/php-fpm.conf 

error_log = /var/log/php-fpm/error.log
log_level = error
```

### 3、将php加载到nginx中

```bash
[root@localhost ~]# vim /etc/nginx/nginx.conf

            location / {
            index index.php index.html;
        }

        location ~ \.php$ {
            root           /usr/share/nginx/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
```



### 4、启动

```bash
[root@localhost ~]# systemctl start php-fpm
[root@localhost ~]# systemctl enable php-fpm

[root@localhost ~]# systemctl start mariadb
[root@localhost ~]# systemctl enable mariadb

[root@localhost ~]# systemctl start nginx
[root@localhost ~]# systemctl enable nginx

```

### 5、测试

* 测试php是否成功

```bash
[root@localhost ~]# cat /usr/share/nginx/html/test1.php
<?php
   phpinfo();
?>
```

* 测试数据库是否连接成功

```bash
[root@localhost ~]# cat /usr/share/nginx/html/test2.php
<?php
  $link=mysql_connect("localhost","root","");
  if($link)
      echo "OK.";
  else
      echo "Bad.";
  mysql_close()
?>
```

## 三、为PHP添加功能模块

### 1、查看PHP加载的模块

```bash
[root@localhost ~]# php -m 
[PHP Modules]
bz2
calendar
Core
ctype
.......
```

### 2、添加xcache模块

* xcache
  * php加速器
* 生成configure配置工具
  * 默认模块中没有configure工具

```bash
[root@localhost ~]# tar xf xcache-3.2.0.tar.gz 
[root@localhost ~]# cd xcache-3.2.0/
[root@localhost xcache-3.2.0]# /usr/bin/phpize 
Configuring for:
PHP Api Version:         20100412
Zend Module Api No:      20100525
Zend Extension Api No:   220100525
```

* 安装模块 

```bash
[root@localhost xcache-3.2.0]# ./configure --enable-xcache --with-php-config=/usr/bin/php-config 
[root@localhost xcache-3.2.0]# make && make install 

Installing shared extensions:     /usr/lib64/php/modules/  //  模块安装路径
```

* 配置php加载模块、并验证

```bash
[root@localhost xcache-3.2.0]# vim /etc/php.ini 
extension=/usr/lib64/php/modules/xcache.so

[root@localhost xcache-3.2.0]# systemctl restart php-fpm
[root@localhost xcache-3.2.0]# php -m | grep SCache
```

