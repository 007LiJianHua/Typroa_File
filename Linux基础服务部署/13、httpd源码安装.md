# httpd源码安装

[toc]

## 一、安装apr

```bash
[root@bogon ~]# tar xf apr-1.5.2.tar.gz 
[root@bogon ~]# cd apr-1.5.2/
[root@bogon apr-1.5.2]# ./configure --prefix=/usr/local/apr 	//apr和apr-utils是源码安装httpd的运行环境
[root@bogon apr-1.5.2]# make && make install 
```

## 二、安装apr-utils

```bash
[root@bogon ~]# tar xf apr-util-1.5.4.tar.gz 
[root@bogon ~]# cd apr-util-1.5.4/
[root@bogon apr-util-1.5.4]# ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr 
[root@bogon apr-util-1.5.4]# make && make install 
```

## 三、安装httpd

```bash
[root@bogon ~]# tar xf httpd-2.4.12.tar.gz 
[root@bogon ~]# cd httpd-2.4.12/
[root@bogon httpd-2.4.12]# ./configure --prefix=/usr/local/httpd24 --enable-so --enable-rewrite --enable-ssl --enable-cgi --enable-cgid --enable-modules=most --enable-mods-shared-most --enable-mpm-shared=all --with-mpm=event --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util
[root@bogon httpd-2.4.12]# make && make install 
```

## 四、httpd启动管理

### 1、启动httpd

```bash
[root@bogon ~]# /usr/local/httpd24/bin/httpd -k start
[root@bogon ~]# netstat -antp | grep http
tcp6       0      0 :::80                   :::*                    LISTEN      39423/httpd         
```

### 2、设置httpd开机自启动

* /etc/rc.d/rc.local
  * 这个文件会在每次开机的时候加载，所以可以将想要开机自启的命令放到这里
  * 并且为该文件添加执行权限

```bash
[root@bogon ~]# vim /etc/rc.d/rc.local 
/usr/local/httpd24/bin/httpd -k start
[root@bogon ~]# chmod a+x /etc/rc.d/rc.local
```

## 五、httpd相关文件目录 

### 1、httpd相关命令

* httpd -t
* htpasswd -c 
* httpd

```bash
httpd安装目录/bin
```

### 2、主配置文件

```bash
httpd安装目录/conf/httpd.conf
```

### 3、子配置文件目录

* **如果想要添加虚拟主机不仅要在子配置文件目录下创建.conf文件，还要在主配置文件中添加/去掉#号对应的子配置文件**

```bash
httpd安装目录/conf/extra/*.conf 
```

### 4、默认网页目录

```bash
httpd安装目录/htdocs
```

## 六、永久添加环境变量路径

```bash
[root@bogon ~]# vim /etc/profile
export PATH=$PATH:/usr/local/httpd24/bin
[root@bogon ~]# source /etc/profile		//重新读取配置文件
```

