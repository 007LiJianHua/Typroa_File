[toc]

## 一、数据库类型

根据存储的类型分为两种，数据的持久化存储与缓存数据，对应的分别是关系型数据库、非关系型数据库

* 关系型数据库（RDBMS）
  * MySQl、Oracle、PostgreSQL
  * 使用场景
    * 用于数据的持久化存储
* 非关系型数据库 （NoSQL）
  * memcached、mongDB、redis
  * 优势：支持高并发，速度快
    * 没有严格的数据约束规范
    * 以key-value键值对的方式进行存储
    * 基于内存进行存储，所以处理速度快
    * 基于O1算法开发
  * 使用场景
    * 缓存服务器
    * 存储业务session

## 二、redis特性

网址：https://redis.io/

* 开源的，跨平台的
* 以key-value存储数据
* 基于内存存储数据
* 支持持久化存储数据据
  * 两种方式
    * rdb数据文件   通过后端手动回滚生成*.rdb文件
    * aof日志：相当于mysql的二进制日志，会记录下redis的写操作

## 三、redis安装部署

### 1、安装

```bash
[root@localhost ~]# yum install -y gcc 
```

### 2、安装redis

```bash
[root@localhost ~]# tar xf redis-5.0.12.tar.gz 
#或者
[root@localhost ~]# wget https://download.redis.io/releases/redis-6.2.6.tar.gz
[root@localhost ~]# cd redis-5.0.12/
[root@localhost redis-5.0.12]# make 
[root@localhost redis-5.0.12]# make PREFIX=/usr/local/redis install 
```

### 3、准备redis配置文件,并添加环境变量

```bash
[root@localhost redis-5.0.12]# mkdir /usr/local/redis/conf
[root@localhost redis-5.0.12]# cp redis.conf /usr/local/redis/conf
[root@localhost ~]# sed -ri '$a \export PATH=$PATH:/usr/local/redis/bin' /etc/profile
[root@localhost ~]# source /etc/profile
```

### 4、启动redis

* redis默认在前台启动

```bash
[root@localhost ~]# nohup redis-server /usr/local/redis/conf/redis.conf & 
[1] 11260

[root@localhost ~]# netstat -antp | grep redis
tcp        0      0 127.0.0.1:6379          0.0.0.0:*    
```

### 5、测试数据读写

```bash
[root@localhost ~]# redis-cli 
127.0.0.1:6379> set name martin
OK
127.0.0.1:6379> get name
"martin"
127.0.0.1:6379> exit
```

### 启动管理

#### 启动redis

```bash
# nohup redis-server /usr/local/redis/conf/redis.conf &
```

#### 关闭redis

```bash
[root@localhost ~]# redis-cli shutdown
```

## 四、数据管理操作

### 1、增加、删除、修改（覆盖）、查看

```bash
#增加
127.0.0.1:6379> set url http://www.baidu.com
OK
#删除
127.0.0.1:6379> del url
(integer) 1
#修改(覆盖)
127.0.0.1:6379> set url http://www.jd.com
OK
#查看
127.0.0.1:6379> get url
"http://www.baidu.com"
```

### 2、切换数据库

redis内部有16个数据库，编号为0~15

* 测试在当前数据库存入键值，切换数据库会发现没有之前写入的键值

```bash
127.0.0.1:6379> set name martin
OK
127.0.0.1:6379> get name
"martin"
127.0.0.1:6379> 
127.0.0.1:6379> SELECT 2
OK
127.0.0.1:6379[2]> get name
(nil)
```

### 3、数学运算+ - 

* 加法

```bash
127.0.0.1:6379> set data 10
OK
127.0.0.1:6379> INCR data
(integer) 11
#增加10
127.0.0.1:6379> INCRBY data 10
(integer) 21
```

* 减法

```bash
127.0.0.1:6379> DECR data
(integer) 11
#减小10
127.0.0.1:6379> DECRBY data 10
(integer) 1
```

### 4、支持事务

```bash
127.0.0.1:6379> get data
"10"
# 启动事务
127.0.0.1:6379> multi 			
OK
127.0.0.1:6379> incr data
QUEUED
127.0.0.1:6379> incr data
QUEUED
127.0.0.1:6379> get data
QUEUED
# 提交事务
127.0.0.1:6379> exec			
1) (integer) 11
2) (integer) 12
```

## 五、通过PHP连接redis进行数据读写

### 1、部署php平台

```bash
[root@localhost ~]# yum install -y httpd php php-gd php php-devel  
```

### 2、php测试代码

* 在网页目录下编写测试代码，记得开启redis数据库

```bash
<?php
  $redis = new Redis();
  $redis -> connect("127.0.0.1",6379);
  $redis -> set("name", "Tome");
  $var = $redis -> get("name");
  echo $var;
?>
```

### 3、为php添加redis的模块

* 获取模块并编译安装

```bash
[root@localhost ~]# tar xf redis-php-4.1.1.tgz 
[root@localhost ~]# cd redis-4.1.1/

#获取./configure命令
[root@localhost redis-4.1.1]# /usr/bin/phpize 
Configuring for:
PHP Api Version:         20100412
Zend Module Api No:      20100525
Zend Extension Api No:   220100525
[root@localhost redis-4.1.1]# ./configure --enable-redis --with-php-config=/usr/bin/php-config 
[root@localhost redis-4.1.1]# make 
[root@localhost redis-4.1.1]# make install
Installing shared extensions:     /usr/lib64/php/modules/

[root@localhost redis-4.1.1]# ls /usr/lib64/php/modules/
curl.so  fileinfo.so  gd.so  json.so  phar.so  redis.so  zip.so

```

### 4、配置PHP加载redis模块

```bash
[root@localhost ~]# vim /etc/php.ini 
extension=/usr/lib64/php/modules/redis.so
#因为php时作为模块添加到http的服务中，所以，在修改php之后要重启httpd
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# php -m | grep -i redis
redis
```

### 5、测试

* 在网页输入http://192.168.152.10/test.php

提示   Tome 则PHP连接redis