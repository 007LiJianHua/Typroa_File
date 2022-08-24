[toc]

## 一、环境描述

```bash
通过客户端访问浏览器来实现：相同会话ID负载到不同的后端tomcat服务器还能保持会话

nginx上设置反向代理+负载均衡   192.168.152.10
tomcat设置双实例   192.168.152.11:9000  
				 192.168.152.11:9001
后端设一个单独的哨兵来管理三个redis实例  192.168.152.12
三个redis实例   192.168.152.13:7001
			   192.168.152.13:7002
			   192.168.152.13:7003
```



![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/C69183C9021241C490E19D6F073E1ED2/455A411E72AC4E75B7C20BE5F1085040/13164](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/C69183C9021241C490E19D6F073E1ED2/455A411E72AC4E75B7C20BE5F1085040/13164)



## 二、安装配置

### 1、解压tomcat与jdk环境

```bash
[root@localhost ~]# ll
total 185860
-rw-r--r--  1 root root   8931288 Jul 11  2017 apache-tomcat-7.0.72.tar.gz
-rw-r--r--  1 root root 181367942 Aug 25  2017 jdk-8u91-linux-x64.tar.gz
[root@localhost ~]# tar -xf apache-tomcat-7.0.72.tar.gz  -C /usr/local/
[root@localhost ~]# tar -xf jdk-8u91-linux-x64.tar.gz  -C /usr/local/
#改个名字，好记
[root@localhost ~]# mv /usr/local/apache-tomcat-7.0.72/ /usr/local/tomcat70/
```

### 3、添加java环境变量

```bash
[root@localhost ~]# tail -2 /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
[root@localhost ~]# source /etc/profile
[root@localhost ~]# java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

### 3、规划目录复制配置文件

```bash
[root@localhost ~]# mkdir -p /opt/tomcat0{1,2}/{conf,logs,webapps}
[root@localhost ~]# cp /usr/local/tomcat70/conf/* /opt/tomcat01/conf/
[root@localhost ~]# cp /usr/local/tomcat70/conf/* /opt/tomcat02/conf/
```

### 4、修改两个实例的配置文件

```bash
[root@localhost ~]# vim /opt/tomcat01/conf/server.xml
#实例端口
<Server port="8005" shutdown="SHUTDOWN">
#实例1提供httpd服务的端口
<Connector port="9000" protocol="HTTP/1.1"
#指定实例1的数据文件位置
<Host name="localhost"  appBase="/opt/tomcat01/webapps"
#指定实例1的日志文件位置
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="/opt/tomcat01/logs"
[root@localhost ~]# vim /opt/tomcat02/conf/server.xml
<Server port="8006" shutdown="SHUTDOWN">
<Connector port="9001" protocol="HTTP/1.1"
<Host name="localhost"  appBase="/opt/tomcat02/webapps"
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="/opt/tomcat02/logs"

```

### 5、编写两个tomcat启动脚本

```bash
[root@localhost ~]# cat /opt/tomcat01/daemon.sh
#!/bin/bash
#

export CATALINA_HOME=/usr/local/tomcat70
export CATALINA_BASE=/opt/tomcat01

case $1 in 
   start)
	$CATALINA_HOME/bin/startup.sh 
	;;
   stop)
	$CATALINA_HOME/bin/shutdown.sh
	;;
   reload)
	$CATALINA_HOME/bin/shutdown.sh
	sleep 1
	$CATALINA_HOME/bin/startup.sh
	;;
   *)
	echo "请输入 <start> <stop> <reload>"
	;;
esac
[root@localhost ~]# chmod a+x /opt/tomcat01/daemon.sh
[root@localhost ~]# /opt/tomcat01/daemon.sh asd
请输入 <start> <stop> <reload>
[root@localhost ~]# /opt/tomcat01/daemon.sh start
Using CATALINA_BASE:   /opt/tomcat01
Using CATALINA_HOME:   /usr/local/tomcat70
Using CATALINA_TMPDIR: /opt/tomcat01/temp
Using JRE_HOME:        /usr/local/jdk1.8.0_91
Using CLASSPATH:       /usr/local/tomcat70/bin/bootstrap.jar:/usr/local/tomcat70/bin/tomcat-juli.jar
Tomcat started.
[root@localhost ~]# cp /opt/tomcat01/daemon.sh /opt/tomcat02/
[root@localhost ~]# chmod a+x /opt/tomcat02/daemon.sh
[root@localhost ~]# /opt/tomcat02/daemon.sh start
Using CATALINA_BASE:   /opt/tomcat02
Using CATALINA_HOME:   /usr/local/tomcat70
Using CATALINA_TMPDIR: /opt/tomcat02/temp
Using JRE_HOME:        /usr/local/jdk1.8.0_91
Using CLASSPATH:       /usr/local/tomcat70/bin/bootstrap.jar:/usr/local/tomcat70/bin/tomcat-juli.jar
Tomcat started.
[root@localhost ~]# netstat -tnulp | grep java
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      1777/java           
tcp6       0      0 127.0.0.1:8006          :::*                    LISTEN      1833/java           
tcp6       0      0 :::9000                 :::*                    LISTEN      1777/java           
tcp6       0      0 :::9001                 :::*                    LISTEN      1833/java  
```

### 6、创建默认访问目录，放入jsp文件

```bash
[root@localhost ~]# ls
1.sh  anaconda-ks.cfg  apache-tomcat-7.0.72.tar.gz  crontab  index.jsp  jdk-8u91-linux-x64.tar.gz  original-ks.cfg
[root@localhost ~]# mkdir /opt/tomcat01/webapps/ROOT
[root@localhost ~]# mkdir /opt/tomcat02/webapps/ROOT
[root@localhost ~]# cp index.jsp /opt/tomcat01/webapps/ROOT
[root@localhost ~]# cp index.jsp /opt/tomcat02/webapps/ROOT
#方便区别，将实例2的jsp文件修改一些明显的字，方便展示负载均衡
```

火狐浏览器输入http://192.168.152.11:9000  和 http://192.168.152.11:9001  会看到两个不同的会话ID界面

### 7、在nginx服务器上配置负载均衡

```bash
[root@nginx ~]#  ls
anaconda-ks.cfg  a.tx  nginx-1.20.1.tar.gz  original-ks.cfg  s.txt
[root@nginx ~]# tar -xf nginx-1.20.1.tar.gz 
[root@nginx ~]# cd nginx-1.20.1/
[root@nginx nginx-1.20.1]# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src
#安装gcc编译器等其他依赖
[root@nginx ~]# yum -y install gcc pcre pcre-devel openssl-devel zlib-devel
[root@nginx ~]# cd nginx-1.20.1/
[root@nginx nginx-1.20.1]# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src
#开始编译安装
[root@nginx nginx-1.20.1]# ./configure --prefix=/usr/local/nginx && make && make install
#启动nginx，为nginx添加负载均衡
[root@nginx nginx-1.20.1]# vim /usr/local/nginx/conf/nginx.conf
    upstream tomcat_server {
        server 192.168.152.11:9000;
        server 192.168.152.11:9001;
    }
    server {
        listen       80;
        server_name  localhost;
        location / {
                proxy_pass http://tomcat_server;
        }  
[root@nginx nginx-1.20.1]# /usr/local/nginx/sbin/nginx -s reload
```

此时在浏览器输入http://192.168.152.10  多次刷新，会发现会话ID一直在变，为了实现会话保持，后端搭建redis集群通过redis哨兵进行管理

### 8、在redis服务器上实现三实例

```bash
[root@red_servers ~]# ls
anaconda-ks.cfg  redis-2.6.14.tar.gz
[root@red_servers ~]# tar -xf redis-2.6.14.tar.gz 
[root@red_servers ~]#cd redis-2.6.14/
[root@red_servers redis-2.6.14]# ls
00-RELEASENOTES  BUGS  CONTRIBUTING  COPYING  deps  INSTALL  Makefile  MANIFESTO  README  redis.conf  runtest  sentinel.conf  src  tests  utils
[root@red_servers redis-2.6.14]# make 
[root@red_servers  redis-2.6.14]# make PREFIX=/usr/local/redis install 
```

### 9、创建三实例目录、拷贝配置文件

```bash
[root@red_servers ~]# mkdir -p /opt/700{1,2,3}/{conf,data}
[root@red_servers ~]# cp redis-2.6.14/
00-RELEASENOTES  CONTRIBUTING     deps/            INSTALL          MANIFESTO        redis.conf       sentinel.conf    tests/           
BUGS             COPYING          .gitignore       Makefile         README           runtest          src/             utils/           
[root@red_servers ~]# cp redis-2.6.14/redis.conf /opt/7001/conf/
[root@red_servers ~]# cp redis-2.6.14/redis.conf /opt/7002/conf/
[root@red_servers ~]# cp redis-2.6.14/redis.conf /opt/7003/conf/
[root@red_servers ~]# vim /opt/7001/conf/redis.conf 
daemonize yes
pidfile /var/run/redis_7001.pid
port 7001
bind 192.168.140.12
loglevel warning
logfile "/var/log/redis_7001.log"
dbfilename dump_7001.rdb
dir /app/7001/data
appendonly yes
appendfilename appendonly_7001.aof
[root@red_servers ~]# sed -ri 's|7001|7002|' /opt/7002/conf/redis.conf 
[root@red_servers ~]# sed -ri 's|7001|7003|' /opt/7003/conf/redis.conf 
[root@red_servers data]# redis-server /opt/7002/conf/redis.conf 
[root@red_servers data]# redis-server /opt/7003/conf/redis.conf 
[root@red_servers data]# redis-server /opt/7001/conf/redis.conf
[root@red_servers ~]# netstat -tunlp | grep redis
tcp        0      0 192.168.152.13:7001     0.0.0.0:*               LISTEN      6017/redis-server   
tcp        0      0 192.168.152.13:7002     0.0.0.0:*               LISTEN      6036/redis-server   
tcp        0      0 192.168.152.13:7003     0.0.0.0:*               LISTEN      6040/redis-server   
```

### 10、配置实例主从

```bash
#在两个从实例上修改
slaveof 192.168.152.13 7001
```

### 11、测试负载均衡

```bash
[root@red_servers ~]# redis-cli -h 192.168.152.13 -p 7001
redis 192.168.152.13:7001> set aa 10
OK
redis 192.168.152.13:7001> get aa
"10"
redis 192.168.152.13:7001> exit
[root@red_servers ~]# redis-cli -h 192.168.152.13 -p 7002
redis 192.168.152.13:7002> get aa
"10"
redis 192.168.152.13:7002> exit
[root@red_servers ~]# redis-cli -h 192.168.152.13 -p 7003
redis 192.168.152.13:7003> get aa
"10"
redis 192.168.152.13:7003> exit

```

### 12、配置哨兵

```bash
#将编译好的redis发送给哨兵进行安装
[root@red_servers ~]# ls
anaconda-ks.cfg  redis-2.6.14  redis-2.6.14.tar.gz
[root@red_servers ~]# rsync -av /root/redis-2.6.14 root@192.168.152.12:/root/
 #创建哨兵文件夹
 [root@bing redis-2.6.14]# mkdir /usr/local/redis/conf -p
[root@bing redis-2.6.14]# cp sentinel.conf /usr/local/redis/conf/
[root@localhost ~]# vim /usr/local/redis/conf/sentinel.conf 
port 26379
sentinel monitor mymaster 192.168.152.12 7001 1
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 60000
[root@bing redis-2.6.14]# make
[root@bing redis-2.6.14]# make PREFIX=/usr/local/redis install
[root@bing ~]# ./redis-2.6.14/src/redis-sentinel /usr/local/redis/conf/sentinel.conf 
[12809] 16 Mar 21:35:45.352 * Max number of open files set to 10032
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.6.14 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 12809
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               
```

### 13、配置tomcat连接哨兵保持会话

```bash
#将必要的jar包放入tomcat的库文件
[root@localhost ~]# cp commons-logging-1.2.jar commons-pool2-2.4.2.jar jedis-2.9.0.jar tomcat-cluster-redis-session-manager-2.0.jar /usr/local/tomcat/lib/
```

### 14、修改两个tomcat实例，指定redis集群信息

```bash
[root@localhost ~]# vim /opt/tomcat01/conf/context.xml 
   <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
   <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         maxInactiveInterval="60"
         sentinelMaster="mymaster"
         sentinels="192.168.152.13:26379"
   />
[root@localhost ~]# vim /opt/tomcat02/conf/context.xml 
   <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
   <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         maxInactiveInterval="60"
         sentinelMaster="mymaster"
         sentinels="192.168.152.13:26379"
   />
[root@localhost ~]# netstat -antp | grep java
tcp6       0      0 :::9000                 :::*                    LISTEN      17377/java          
tcp6       0      0 :::9001                 :::*                    LISTEN      17456/java          
tcp6       0      0 :::8009                 :::*                    LISTEN      17377/java          
tcp6       0      0 127.0.0.1:8010          :::*                    LISTEN      17377/java          
tcp6       0      0 127.0.0.1:8020          :::*                    LISTEN      17456/java          
tcp6       0      0 192.168.140.11:47648    192.168.140.13:26379    ESTABLISHED 17456/java          
tcp6       0      0 192.168.140.11:47640    192.168.140.13:26379 
```

### 15、测试访问tomcat