[toc]

## 一、配置文件解析

### 1、Nginx配置文件结构

- server {}

- - 一个server的配置就对应一个虚拟主机 

- location {}

- - 匹配客户端访问请求，以不同的方式给客户端响应

```bash
全局配置

事件驱动模型配置

http服务配置
#http服务配置中可以有多个虚拟主机（server），一个虚拟主机中可以有多个目录（location）
http {
    server {
        
        location {
            响应方式
        }
        
        location {
            
        }
        
        location {
        
        }
    
    }
    
    server {
        
    }
    
    server {
        
    }
}
```

### 2、全局配置

* user nobody；
  * 指定启动工作进程的用户
* worker_processes 8;
  * 定义Nginx默认启动的工作进程数
    * 建议与CPU数量一致，或者2倍
* error_log   logs/error.log   notice
  * 定义错误日志
  * debug，info，notice，warn，error，crit，alert，emerg

### 3、事件驱动模型配置

* worker_connections 1024
  * 定义工作进程所能接收的最大连接数
* use epoll
  * 定义事件驱动模型
  * nginx安装在freeBSDlinux时，名称要改为kqueue

### 4、http服务相关的配置

* include    文件名称；
  * **用来加载子配置文件**
* access_log  logs/access.log  main
  * 定义访问日志

> *  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"';
>
> * **变量说明：**
>
>   $remote_addr：客户端地址
>
>   $remote_user： 基于用户认证访问时，客户端输入的用户名
>
>   $time_local: 访问时间 
>
>   $request： 请求方法(GET/POST)   文件名称  HTTP协议版本 
>
>   $status: http状态码 
>
>   $body_bytes_sent:  响应数据大小，单位字节 
>
>   $http_referer: 超链接地址
>
>   $http_user_agent：客户端浏览器类型

* keepalive_timeout  65

* - 定义长连接的空闲超时时间 

- keepalive_requests  1000;

- - 定义长连接的最大请求数 

- gzip  on;

- - 启用gzip压缩

### 5、虚拟主机配置

```bash
server {
............
}
```

* listen       80;

* - 指定监听地址、端口 
  - listen  IP:port;

- server_name  localhost;

- - 指定网站主机名 
  - server_name  名称1  名称2  名称3;

- location的写法

- - 匹配客户端请求，定义不同响应的方式  

* URL地址与URI地址区别
  * URL：http://www.linux.com/aa/index.html
  * URI：aa/index.html

```bash
location uri地址 {
    响应方式
}
location / {			//匹配所有请求
    root   html;			//指定网页目录
    index  index.html index.htm;		//指定首页名称
        }
```

## 二、虚拟主机认证配置

### 1、虚拟主机经常会使用的模块

```bash
#基于客户端地址进行认证
allow 192.168.152.1;
deny all;
#基于用户名密码的认证,要使用htpasswd -c /usr/local/nginx/.webuser ljh
auth_basic "Need to login:"
auth_basic_user_file /usr/local/nginx/.webuser
#自动列出文件
autoindex on;
```



[吴老板的笔记]: https://note.youdao.com/ynoteshare/index.html?id=2dd243e3793f68efa7840971eb8d1a90&amp;type=notebook&amp;_time=1646707554037#/0A8D9EFC6AB54C79ACD651766392E5CC

* 基于IP的虚拟主机
* 基于域名的虚拟主机
* 基于用户名密码认证的虚拟主机
