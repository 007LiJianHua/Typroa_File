[toc]

## 一、检测服务是否正常

* 每隔一秒访问nginx，能否正常运行

```bash
#!/bin/bash
#

while true; do
   curl 127.0.0.1 &> /dev/null
   if [ $? -eq 0 ]; then
        echo "ok" >> /tmp/check 
   else
        echo "error" >> /tmp/check
   fi
   sleep 1
done
```

## 二、查看nginx支持的参数

```bash
[root@web_server ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.14.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi/ --http-scgi-temp-path=/var/tmp/nginx/scgi/ --with-pcre --with-file-aio --with-http_secure_link_module --with-threads

```

## 三、以相同的版本参数配置新版本nginx

```bash
[root@web_server ~]# wget http://nginx.org/download/nginx-1.18.0.tar.gz
[root@web_server ~]# tar xf nginx-1.18.0.tar.gz 
[root@web_server ~]# cd nginx-1.18.0/
[root@web_server nginx-1.18.0]# ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi/ --http-scgi-temp-path=/var/tmp/nginx/scgi/ --with-pcre --with-file-aio --with-http_secure_link_module --with-threads
[root@web_server nginx-1.18.0]# make 
==========不要执行make install=====================
```

* 此时可以看到新版本的nginx

```bash
[root@web_server nginx-1.18.0]# ls objs/
autoconf.err  Makefile  nginx  nginx.8  ngx_auto_config.h  ngx_auto_headers.h  ngx_modules.c  ngx_modules.o  src
[root@web_server nginx-1.18.0]# 
[root@web_server nginx-1.18.0]# objs/nginx -v
nginx version: nginx/1.18.0
```

## 四、将旧版本的nginx二进制命令备份

```bash
[root@web_server nginx-1.18.0]# mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
[root@web_server nginx-1.18.0]# ls /usr/local/nginx/sbin/
nginx.bak
[root@web_server nginx-1.18.0]# cp objs/nginx /usr/local/nginx/sbin/
[root@web_server nginx-1.18.0]# ls /usr/local/nginx/sbin/
nginx  nginx.bak
```

## 五、启动新版本nginx

* 不能直接启动，否则会报错
  * 给新生成的二进制命令发送USR2信号

```bash
[root@web_server ~]# kill -USR2 $(cat /usr/local/nginx/logs/nginx.pid) 
[root@web_server ~]# 
[root@web_server ~]# ps -elf | grep nginx
1 S root      91827      1  0  80   0 - 11754 sigsus 16:40 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
5 S nginx     91828  91827  0  80   0 - 11850 ep_pol 16:40 ?        00:00:00 nginx: worker process
0 S root      92169  91827  0  80   0 - 11758 sigsus 16:43 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
5 S nginx     92170  92169  0  80   0 - 11853 ep_pol 16:43 ?        00:00:00 nginx: worker process
0 S root      92190   6857  0  80   0 - 28203 pipe_w 16:43 pts/0    00:00:00 grep --color=auto nginx
[root@web_server ~]# 
[root@web_server ~]# ls /usr/local/nginx/logs/
access.log  error.log  nginx.pid  nginx.pid.oldbin
[root@web_server ~]# 
[root@web_server ~]# cat /usr/local/nginx/logs/nginx.pid
92169
[root@web_server ~]# cat /usr/local/nginx/logs/nginx.pid.oldbin 
91827

```

## 六、优雅的关闭nginx旧工作进程

* 注意关闭顺序，否则就没有平滑升级的意义了
  * 先关闭子进程，再关闭父进程

```bash
[root@web_server ~]# kill -WINCH $(cat /usr/local/nginx/logs/nginx.pid.oldbin)
[root@web_server ~]# ps -elf | grep nginx
1 S root      91827      1  0  80   0 - 11754 sigsus 16:40 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
0 S root      92169  91827  0  80   0 - 11758 sigsus 16:43 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
5 S nginx     92170  92169  0  80   0 - 11853 ep_pol 16:43 ?    
```

* 清理旧的主进程

```bash
[root@web_server ~]# kill 91827
[root@web_server ~]# ps -elf | grep nginx
0 S root      92169      1  0  80   0 - 11758 sigsus 16:43 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
5 S nginx     92170  92169  0  80   0 - 11853 ep_pol 16:43 ?        00:00:00 nginx: worker process
0 S root      92758   6857  0  80   0 - 28203 pipe_w 16:47 pts/0    00:00:00 grep --color=auto nginx
[root@web_server ~]# rm -rf /usr/local/nginx/sbin/nginx.bak 

```

## 七、为nginx添加第三方模块

### 1、添加第三方模块

```bash
--add-module=PATH
```

### 2、添加nginx-purge-cache模块

* nginx-purge-cache
  * 清理nginx缓存

* 注意在最后导入模块的时候，给出模块的路径

```bash
[root@game ~]# wget  http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz
[root@game ~]# tar xf ngx_cache_purge-2.3.tar.gz 
[root@game nginx-1.20.1]# ./configure -prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-file-aio --with-http_secure_link_module --with-threads --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi/ --http-scgi-temp-path=/var/tmp/nginx/scgi/ --add-module=/root/ngx_cache_purge-2.3 
[root@game nginx-1.20.1]# make
```

### 3、剩下与（四）相同，不再赘述