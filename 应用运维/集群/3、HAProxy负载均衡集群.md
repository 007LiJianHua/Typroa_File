[toc]

# HAProxy负载均衡集群

## 一、haproxy介绍

> HAProxy是一个开源的、高性能的、基于TCP和HTTP应用的负载均衡软件，借助HAProxy可快速、可靠地提供基于TCP和HTTP应用的负载均衡解决方案

### 1、调度器的类型

* 四层调度
  * 仅能根据IP地址、端口号进行调度
  * 比如LVS：某个服务提供什么端口，通过端口调度
* 七层调度
  * 根据应用数据（HTTP协议）进行调度
  * nginx，比如某个客户端要访问网站的视频文件，音频文件、文本文件、可通过location 正则来匹配，再进行转发

### 2、haproxy特性

> * 可靠性和稳定性非常好，可以与硬件的F5相媲美
> * 最高可以同时维护40000--50000个并发连接、单位时间内处理的最大请求数为20000，最大数据处理能力可达10Gbps
> * 支持对于8种负载均衡算法、同时也支持session保持
> * 支持虚拟主机功能
> * 从HAProxy 1.3 版本后开始支持连接拒绝、全透明代理等功能
> * HAproxy拥有一个功能强大的服务状态监控页面
> * HAProxy由用户功能强的ACL支持

### 3、haoroxy配置文件

> * global	
>   * 全局配置、进程数、日志等
> * defalts
>   * 默认参数
> * frontend
>   * 定义前端的虚拟服务、调度算法
> * backend
>   * 定义后端的real-server 
> * listen
>   * 配置i监听、用于展示监控页面、用于配置虚拟服务

## 二、haproxy配置文件解析

### 1、global全局配置

```bash
global
	#定义haproxy
	log 127.0.0.1 local info  
	#指定haproxy启动的进程数
	nbproc 1
	maxconn 4096  #最大连接数
	user nobody  #指定haproxy进程启动的用户
	group nobody   
	daemon  # 后台启动
	pidfile   /usr/local/haproxy/logs/haproxy.pid  

	日志级别
		debug, info, notice, warn, error, alert, emercy
```

### 2、defaults  默认配置

```bash
defaluts
	retires  3		#指定后端服务健康状态检查时，最大的失败次数
	timeout check 5	#健康状态检查的时间间隔
	timeout connect 10	#连接real-serverd的超时时	间，单位毫秒
	timeout client 20s	#连接客户端的超时时间
	timeout server 30s 	#real-server连接客户端的超时时间
```

### 3、frontend 定义虚拟服务

```bash
frontend xxxxx						# 定义虚拟服务的名称
	bind IP:port					# 定义虚拟服务监听的IP及端口
	mode {tcp|http} 				# 工作模式; http: 七层调度    tcp：四层调度
	option forwardfor				# 后端服务器记录日志时，记录真实客户端地址 
	option httpclose				# 优化参数, 高并发场景下，haproxy会自动断开连接时间过长的客户端请求
	log global
	use_backend XXX				# 定义后端服务器
	default_backend XXX			# 定义默认后端服务器
	
```

### 4、backend 定义后端real-server

```bash
backend xxxxxxx		# 定义后端服务器的名称 
	mode {tcp|http}
	option abortonclose		# 优化参数, 高并发场景下让后端服务器自动结束处理时间较长的请求
	option redispatch
	cookie SERVERID
	balance roundrobin 		# 调度算法
	server web1 10.1.1.1:80 cookie s1 weight 6 check inter 2000 rise 2 fall 3
	server web2 10.1.1.2:80 cookie s2 weight 6 check inter 2000 rise 2 fall 3
```



#### 1)支持的调度算法

> * roundrobin
>   * 基于权重进行轮询调度的算法
> * static-rr
>   * 基于权重进行轮询调度的算法，不过次算法为静态算法，再运行时调整其服务器权重不会生效
> * source
>   * 源hash，将同一个客户端 的请求转发到同一个后端服务器
> * leastconn
>   * 最少连接
> * uri
>   * 此算法会对部分或整个URI进行hash运算，在经过与服务器的总权重向相除，最后转发到某台匹配的后端服务器上
> * uri_param
>   * 此算法会根据URL路径中的参数进行转发，这样可保证再后端真实服务器数据不变时，同一个用户的请求始终转发到同一个机器上

#### 2)健康状态检测的参数

```bash
server web1 10.1.1.1:80 cookie server 1 weight 6 check inter 2000 rise 2 fall 3 
	web1 后端服务器名称
	weight 6 权重
	check inter 2000 rise 2 fall 3 定义健康状态检查参数
```

#### 3)会话保持的配置

> cookie SERVERID
>
> * server ... cookie server 1
>   * 服务器在给客户端响应时，要在响应数据中插入实现指定的serverid，用于实现会话保持
> * option redispatch
>   * 此参数用于cookie保持的环境中，在默认的情况下，HAProxy会将其请求的后端服务器的serverID插入cookie中，以保证会话的session持久性，而如果后端服务器出现故障，客户端的cookie是不会刷新的，这就会造成无法访问，此时，如果设置了此参数，就会将客户的请求强制定向到另外一台健康的后端服务器上，以保证服务正常

#### 4)、会话保持的解决方案：

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE19073363616c4cb27e87e1bdc7584013/119](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE19073363616c4cb27e87e1bdc7584013/119)

### 5、listen 配置监听

```bash
listen admin_status		#定义服务名称
	bind 0.0.0.0:9188	#服务监听的地址及端口
	stats refresh 30s	#数据刷新的时间
	stats uri/haproxy-status	#访问监控页面的uri地址
	stats realm Welcome login 		# 用户名对话框提示信息
	stats auth admin:admin 			# 认证的用户名、密码
	stats hide-version 				# 监控页面不显示软件版本信息
```

## 三、haproxy ACL功能

### 1、ACL介绍

> * ACL：访问控制列表
> * 用于匹配客户端的请求
> * 只能应用于frondend、listen中
> * 作用：
>   * 访问控制
>   * 七层调度

### 2、ACL的语法

acl  <acl名称>  <[匹配请求的方法]> -i   <具体条件>

-i：忽略大小写

#### 1)常用acl方法：

> * hdr_reg(host)		
>
> * - 以正则表达式的方式匹配主机名
>   - acl test1 hdr_reg(host) -i www
>
> - hdr_dom(host)		
>
> - - 精确匹配主机名
>   - acl test2 hdr_dom(host) -i music.linux.com 
>
> - hdr_beg(host)		
>
> - - 匹配主机名以xxx开头
>   - acl test3 hdr_beg(host) -i download. 
>
> - path_end
>
> - - URL以xxxx结尾
>   - acl test4 path_end -i .jpg  .jpeg  .gif  .png   
>
> - path_beg			
>
> - - url以xxxx开头
>   - acl test5 path_beg -i https://
>
> - url_ip
>
> - - 匹配数据目的IP	
>   - acl test5 url_ip 10.1.1.1 
>
> - src 
>
> - - 匹配数据源IP
>
> - method 
>
> - - 匹配HTTP请求的方法；GET, POST
>   - acl test7 method POST 

### 3、ACL实例

#### 1)实现七层调度实例1

```bash
        acl www_policy	hdr_reg(host)  -i 	^(www.z.cn|z.cn)
        acl bbs_policy		hdr_dom(host)  -i   bbs.z.cn 
        acl url_policy      url_sub		   -i       buy_sid=

        use_backend   server_www  if   www_policy
        use_backend   server_app   if  bbs_policy
        use_backend   server_bbs   if url_policy
```

#### 2)七层调度实例2

```bash
        acl  host_www   hdr_beg(host)   -i www 
        acl  host_static   hdr_beg(host)   -i img. vedio. download. ftp.

        use_backend  static   if host_static || host_www 
```

#### 3)实现数据过滤1

```bash
        acl forbidden_dst   url_ip  192.168.0.0/16
        acl forbidden_dst   url_ip  172.16.0.0/12
        acl forbidden_dst   url_ip  10.0.0.0/8

        http-request deny if forbidden_dst
```

#### 4)实现数据过滤2

```bash
        acl  allow_host  src  192.168.200.150/32
        http-request  deny  if  !  allow_host
```

## 四、案例：基于HAproxy实现web服务调度

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/9F0F76ABB72B4160BA36017CEFC5ED62/13648](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/9F0F76ABB72B4160BA36017CEFC5ED62/13648)

### 1、在所有网站服务器上安装httpd，建立测试页面

### 2、安装HAproxy

```bash
[root@haproxy_01 ~]# yum install -y haproxy 
```

### 3、编辑haproxy配置文件

```bash
global
       maxconn 2000
       nbproc 4
       user nobody
       group nobody
       log 127.0.0.1 local0 info
       daemon
       pidfile /var/run/haproxy.pid

defaults
       retries 3
       timeout connect 5s
       timeout client 10s
       timeout server 30s
       timeout check 2s

frontend web_service
       # 监听地址修改为所有地址，便于通过VIP访问服务
       bind 0.0.0.0:80
       mode http
       log global
       option forwardfor


       acl vedio_policy hdr_dom(host) -i vedio.linux.com
       use_backend vedio_web_server if vedio_policy

       acl music_policy hdr_beg(host) -i music.
       use_backend music_web_server if music_policy


backend vedio_web_server
       mode http
       option redispatch
       cookie SERVERID
       option abortonclose
       balance roundrobin
       server v01 192.168.140.11:80 cookie v01 weight 1 check inter 2000 rise 2 fall 2
       server v02 192.168.140.12:80 cookie v02 weight 1 check inter 2000 rise 2 fall 2

backend music_web_server
       mode http
       option redispatch
       cookie SERVERID
       option abortonclose
       balance roundrobin
       server m01 192.168.140.13:80 cookie m01 weight 1 check inter 2000 rise 2 fall 2
       server m02 192.168.140.14:80 cookie m02 weight 1 check inter 2000 rise 2 fall 2

```

### 4、启动haproxy

```bash
[root@haproxy_01 ~]# systemctl start haproxy
[root@haproxy_01 ~]# systemctl enable haproxy

[root@haproxy_01 ~]# netstat -antp | grep haproxy
tcp        0      0 192.168.140.10:80       0.0.0.0:*               LISTEN      6985/haproxy  
```

### 5、测试访问

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/D9CFF89EBD23432EA2DFE30EB6E3087B/13659](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/D9CFF89EBD23432EA2DFE30EB6E3087B/13659)

### 6、通过haproxyIP地址直接访问时，返回503，可通过配置default_backend解决

```bash
#写在frontend 中
	default_backend music_web_server
```

### 7、配置httpd记录真实客户端地址

```bash
[root@web01 ~]# grep -i "logformat" /etc/httpd/conf/httpd.conf 
    LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

### 8、启用haproxy监控页面

```bash
listen admin_status
      bind 0.0.0.0:9188 
      mode http
      stats refresh 30s  
      stats uri /haproxy-status  
      stats realm Welcome login  
      stats auth admin:redhat
      stats hide-version  
```

```bash
[root@haproxy_01 ~]# netstat -antp | grep haproxy
tcp        0      0 192.168.140.10:80       0.0.0.0:*               LISTEN      7042/haproxy        
tcp        0      0 0.0.0.0:9188            0.0.0.0:*               LISTEN      7042/haproxy  
```

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/026C94B893A240A3887510B9DA17465D/13670](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E96E0480305B4B8EB15C0F78EDBD536B/026C94B893A240A3887510B9DA17465D/13670)

### 9、配置第二台haproxy

```bash
[root@haproxy02 ~]# yum install -y haproxy
[root@haproxy02 ~]# vim /etc/haproxy/haproxy.cnf
global
       maxconn 2000
       nbproc 4
       user nobody
       group nobody
       log 127.0.0.1 local0 info
       daemon
       pidfile /var/run/haproxy.pid

defaults
       retries 3
       timeout connect 5s
       timeout client 10s
       timeout server 30s
       timeout check 2s

frontend web_service
       bind 0.0.0.0:80
       mode http
       log global
       option forwardfor


       acl vedio_policy hdr_dom(host) -i vedio.linux.com
       use_backend vedio_web_server if vedio_policy

       acl music_policy hdr_beg(host) -i music.
       use_backend music_web_server if music_policy


       default_backend music_web_server


backend vedio_web_server
       mode http
       option redispatch
       cookie SERVERID
       option abortonclose
       balance roundrobin
       server v01 192.168.140.11:80 cookie v01 weight 1 check inter 2000 rise 2 fall 2
       server v02 192.168.140.12:80 cookie v02 weight 1 check inter 2000 rise 2 fall 2

backend music_web_server
       mode http
       option redispatch
       cookie SERVERID
       option abortonclose
       balance roundrobin
       server m01 192.168.140.13:80 cookie m01 weight 1 check inter 2000 rise 2 fall 2
       server m02 192.168.140.14:80 cookie m02 weight 1 check inter 2000 rise 2 fall 2


listen admin_status
      bind 0.0.0.0:9188
      mode http
      stats refresh 30s                               
      stats uri /haproxy-status               
      stats realm Welcome login               
      stats auth admin:redhat
      stats hide-version  
```

### 10、在两台调度器上分别安装keepalived

```bash
# yum install -y keepalived
```

### 11、主备调度的配置文件

```bash
! Configuration File for keepalived

global_defs {
   router_id haproxy01
}

vrrp_instance haproxy_service {
    state MASTER
    interface ens33
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
}
```

```bash
! Configuration File for keepalived

global_defs {
   router_id haproxy02
}

vrrp_instance haproxy_service {
    state BACKUP
    interface ens33
    virtual_router_id 66
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
}
```

### 12、启动keepalived服务、客户端通过VIP访问测试

```bash
# systemctl start keepalived
# systemctl enable keepalived
```

