[toc]

## 一、nginx反向代理

> * 主要的应用范围
>   * Web应用，和mail邮件服务
> * 语法
>   * proxy_pass 后端服务器地址

### 1、何为反向代理？

在介绍反向代理之前，先来了解一下正向代理。

**正向代理**：如果把局域网外的`Internet`想象成一个巨大的资源库，则局域网中的客户端要访问`Internet`，则需要通过[代理服务器](https://so.csdn.net/so/search?q=代理服务器&spm=1001.2101.3001.7020)来访问，这种代理服务就称为正向代理，下面是正向代理的原理图。

由于工作环境原因，日常工作只能局限于单位的局域网，如果想要访问互联网，怎么办呢？这就需要用到正向代理，本人经常用正向代理来进行上网。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013121717337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4ZDE0MzU1MTM3NzU=,size_16,color_FFFFFF,t_70)
**反向代理**：看下面原理图，就一目了然。其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 `IP`地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013122029747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p4ZDE0MzU1MTM3NzU=,size_16,color_FFFFFF,t_70)

> 正向代理和反向代理的区别，一句话就是：如果我们客户端自己用，就是正向代理。如果是在服务器用，用户无感知，就是反向代理。



### 2、三个简单示例

* 示例1：将nginx下/mp3请求转交给后端httpd服务器/music地址
  * 注意：这个例子写在了nginx的主配置文件中，需要将加载的子配置文件注释掉，否则无法访问

```bash
location /mp3 {
     proxy_pass http://192.168.140.11/music;
}
```

* 示例2：将/testA的请求转交给后端首页处理
  * 注意：http:地址`/`这个`/`就代表了网页首页

```bash
location /testA {
	proxy_pass http://192.168.140.11/;
}
```

```bash
location /testA {
     proxy_pass http://192.168.140.11;
}
```



* 示例3：location匹配正则
  * **默认情况下，nginx会将uri地址自动拼接到后端服务器地址上，而且当location以正则的方式匹配请求时，反向代理只支持写道后端服务器地址**，所以后端服务器也应该有对应的文件目录

```bash
location ~ /testB {
     proxy_pass http://192.168.140.11;
} 
```

### 3、配置后端服务器记录真实的客户端地址

* 以后端服务器访问日志为例
  * 在nginx代理服务器中添加客户端的地址
  * **$remote_addr**是nginx内置变量，记录了真实的客户端地址

```bash
location /MP3 {
	proxy_pass http://192.168.152.10/music;
	proxt_set_header  X-REAL-IP $remote_addr;
}
```

* 最后修改后端httpd的访问日志格式
  * %{X-REAL-IP}i   固定格式

```bash
LogFormat "%{X-REAL-IP}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

## 二、利用upstream模块

### 1、upstream作用

> * 实现后端服务的负载均衡
> * 实现对后端服务的健康检查状态

### 2、负载均衡

* 采用两种不同的算法
  * rr：轮询算法
    * 默认
    * 把来自用户的请求轮流分配给内部的服务器
    * **支持权重weight**
    * 解决会话保持/持久
      * 会话共享
        * 通过NoSQL数据库（redis）来记录会话信息
  
  
  
  ![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE6d89cb4e7b9a8566df6f49c70a1ce885/109](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE6d89cb4e7b9a8566df6f49c70a1ce885/109)
  
  * ip_hash
    * 根据客户端地址进行hash运算
    * 将同一个客户端的请求在一段时间内转交到后端同一个服务器进行处理
  * least_conn
    * 最少连接：哪个服务器连接少，分配给哪个

### 3、语法

* **注意：upstream 写在http的全局配置中，不能写在 某个server虚拟主机上**

```bash
upstream  组名 {
    [调度算法];
    server  IP:port  [weight=数字]  [max_fails=2]  [fail_timeout=2];
    server  IP:port  [weight=数字]  [max_fails=2]  [fail_timeout=2];
    server  IP:port  [weight=数字]  [max_fails=2]  [fail_timeout=2] [backup];
}

location {
    proxy_pass  http://组名;
}
```

**参数说明:**

**upstream组不能定义在某个server{}虚拟主机内部！！！！！！！！**

- IP:port

- - 后端真实服务器的地址

- weight

- - 权重，默认为1

- max_fails

- - 健康状态检查时，最大的失败次数

- fail_timeout

- - 失败次数的间隔时间，单位为秒

- backup

- - 指定备份服务器

### 4、示例

* nginx负载均衡服务器

```bash
    upstream webserver {
       server 192.168.140.11:80 weight=1 max_fails=2 fail_timeout=2;
       server 192.168.140.12:80 weight=1 max_fails=2 fail_timeout=2;
       server 127.0.0.1:8000 backup;
    }
    
           location / {
           proxy_pass http://webserver;
           proxy_set_header X-REAL-IP $remote_addr;
        }
```

* 当后台服务器全部宕机时，使用nginx服务器本机
* 当作页面展示

```bash
    server {
        listen       8000;
        server_name  localhost;

        location / {
            root   /test2;
            index  index.html index.htm;
        }
    }
```

