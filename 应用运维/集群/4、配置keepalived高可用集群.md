[toc]

## 一、keepalived高可用集群

### 1、keepalived介绍

> * keepalived基于VRRP协议开发
> * VRRP协议
>   * 分为主备路由
>   * 优先级1--255
>     * 数字越大，优先级越高
>   * 周期性以组播的形式发送心跳
>   * 默认组播地址：224.0.0.18
>   * 组ID：用来生成MAC地址
>   * 将组ID转换成16进制数，组成虚拟MAC地址
>     * 00-00-5E-00-01-xx

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/70AE79106B3F4949AA1A2FBA858A7BEB/A792DEE4DFAE41A8B889C0DD1D9DE57C/13784](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/70AE79106B3F4949AA1A2FBA858A7BEB/A792DEE4DFAE41A8B889C0DD1D9DE57C/13784)

## 二、keepalived配置案例

> * 环境描述：
>   * 后端配置两个web界面
>   * 配置两个nginx调度器
>   * 这两个调度器通过keepalived配置高可用，形成虚拟IP

### ![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/70AE79106B3F4949AA1A2FBA858A7BEB/06345164807C47C59DCF47AA9D716809/26398](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/70AE79106B3F4949AA1A2FBA858A7BEB/06345164807C47C59DCF47AA9D716809/26398)1、配置虚拟主机两台虚拟主机

```bash
<VirtualHost 192.168.140.12:80>
    ServerName test.linux.com
    DocumentRoot /web1
    CustomLog /var/log/httpd/test_access.log combined
    ErrorLog /var/log/httpd/test_error.log
</VirtualHost>
<Directory "/web1">
   Require all granted
</Directory>
```

```bash
<VirtualHost 192.168.140.13:80>
    ServerName test.linux.com
    DocumentRoot /web2
    CustomLog /var/log/httpd/test_access.log combined
    ErrorLog /var/log/httpd/test_error.log
</VirtualHost>

<Directory "/web2">
   Require all granted
</Directory>
```

### 2、配置2个相同的nginx调度器

```bash
    upstream web {
       server 192.168.140.12:80;
       server 192.168.140.13:80;
    }

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://web;
        }
```

### 3、配置2台机器安装keepalived

```bash
# yum install -y keepalived
```

### 4、编辑keepalived的配置文件

#### 1)主调度

```bash
! Configuration File for keepalived

global_defs {
   router_id nginx_01
}

vrrp_instance web_service {
    state MASTER
    interface ens33
    virtual_router_id 88
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

#### 2)备调度

```bash
! Configuration File for keepalived

global_defs {
   router_id nginx_02
}

vrrp_instance web_service {
    state BACKUP
    interface ens33
    virtual_router_id 88
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

### 5、启动keepalived，查看日志

```bash
[root@nginx01 ~]# systemctl start keepalived
[root@nginx01 ~]# systemctl enable keepalived
[root@nginx01 ~]# tail -f /var/log/messages 

Jun 30 14:59:18 localhost Keepalived_vrrp[9663]: VRRP_Instance(web_service) Entering MASTER STATE
Jun 30 14:59:18 localhost Keepalived_vrrp[9663]: VRRP_Instance(web_service) setting protocol VIPs.
Jun 30 14:59:18 localhost Keepalived_vrrp[9663]: Sending gratuitous ARP on ens33 for 192.168.140.100
```

### 6、配置keepalived使用外部脚本检测虚拟服务状态

#### 1)编写脚本，检测虚拟服务

```bash
[root@nginx01 ~]# cat /etc/keepalived/check_web_service.sh 
#!/bin/bash
#

curl 127.0.0.1 &> /dev/null
if [ $? -ne 0 ]; then
  systemctl stop keepalived
fi
[root@nginx01 ~]# chmod a+x /etc/keepalived/check_web_service.sh
```

#### 2)编辑keepalived配置，应用脚本

```bash
global_defs {
   router_id nginx_01
}

# 定义外部脚本
vrrp_script check_web_service {
    script "/etc/keepalived/check_web_service.sh"	# 脚本名称
    interval 1					# 脚本执行周期 
}

vrrp_instance web_service {
    state MASTER
    interface ens33
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    # 调用外部脚本 
    track_script {
        check_web_service
    }
}
```

#### 3)通过停止nginx虚拟服务，测试VIP转移

```bash
# /usr/local/nginx/sbin/nginx -s stop 
```

#### 4)VIP转回主调度

* 先修复虚拟服务
* 再启动keepalived

## 三、keepalived的心跳管理

### 1、查看心跳

* tcpdump
  * -i：选择网卡
  * -nn：选择协议

```bash
[root@nginx01 ~]# tcpdump -i ens33 -nn vrrp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
16:29:00.039819 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:29:01.041985 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:29:02.043401 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:29:03.044816 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:29:04.046413 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:29:05.048526 IP 192.168.140.10 > 224.0.0.18: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
```

### 2、配置以单播的形式发送心跳

* 注意两端都要配置

```bash
vrrp_instance web_server {
    .............
    # 指定源地址
    unicast_src_ip 192.168.140.10
    # 指定对端地址
    unicast_peer {
        192.168.140.11
    }
}

[root@nginx01 ~]# tcpdump -i ens33 -nn vrrp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
16:35:59.102235 IP 192.168.140.10 > 192.168.140.11: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:36:00.103577 IP 192.168.140.10 > 192.168.140.11: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:36:01.104872 IP 192.168.140.10 > 192.168.140.11: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:36:02.106144 IP 192.168.140.10 > 192.168.140.11: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
16:36:03.107505 IP 192.168.140.10 > 192.168.140.11: VRRPv2, Advertisement, vrid 88, prio 100, authtype simple, intvl 1s, length 20
```



## 四、配置keepalived不抢占VIP

* keepalived默认行为，在主调度故障修复后，会抢夺VIP
* 只要在高优先级调度配置

```bash
vrrp_instance web_service {
      state BACKUP
      nopreempt
}
```

## 五、案例：配置MySQL高可用读写分离集群

### 1、配置两台MySQL服务器主从

#### 1)主服务器配置

```bash
[root@mysql1 ~]# yum -y install mariadb-server
[mysqld]
server_id=14
log_bin=master
#导入测试数据库
[root@master ~]# systemctl start mariadb
[root@master ~]# mysql -uroot < jiaowu.sql 
#全备mysql，发送给从服务器
[root@master ~]# mysqldump -uroot --lock-all-tables --master-data=2 --all-databases > /tmp/all.sql
[root@master ~]# rsync -av /tmp/all.sql  root@192.168.152.15:/tmp/

#为从服务器授权
MariaDB [(none)]> grant all on *.* to 'slave_15'@'192.168.152.15' identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> 
MariaDB [(none)]> show master status;
+---------------+----------+--------------+------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+---------------+----------+--------------+------------------+
| master.000003 |     5235 |              |                  |
+---------------+----------+--------------+------------------+

```

#### 2)从服务器配置

```bash
[root@mysql2 ~]# yum -y install mariadb-server
[root@mysql2 ~]# vim /etc/my.cnf
[mysqld]
server_id=15
log_bin=slave
[root@mysql2 ~]# systemctl start mariadb
[root@slave ~]# mysql -uroot < /tmp/all.sql  
[root@slave ~]# mysql -uroot 
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jiaowu             |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)
#配置主从
MariaDB [(none)]> change master to
    -> master_host="192.168.152.14",
    -> master_user="slave_15",
    -> master_password="redhat",
    -> master_log_file="master.000003",
    -> master_log_pos=5235;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)

#查看状态
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.152.14
                  Master_User: slave_15
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000003
          Read_Master_Log_Pos: 5235
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 526
        Relay_Master_Log_File: master.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

```

### 2、配置两台MyCAT

#### 1)mycat配置环境

```bash
[root@mycat1 ~]# tail -2 /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
[root@mycat1 ~]# tar -xf jdk-8u91-linux-x64.tar.gz -C /usr/local/  
[root@mycat1 ~]# tar -xf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz  -C /usr/local/ 
```

#### 2)修改schema.xml配置文件

```bash
[root@mycat1 ~]# cat /usr/local/mycat/conf/schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<schema name="jiaowu" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
	</schema>

	<dataNode name="dn1" dataHost="localhost1" database="jiaowu" />

	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="hostM1" url="192.168.152.14:3306" user="mycatuser"
				   password="redhat">
		</writeHost>
		<writeHost host="hostS1" url="192.168.152.15:3306" user="mycatuser"
				   password="redhat" />
	</dataHost>
</mycat:schema>

```

#### 3)创建连接mycat的用户

* 注意要将上边的root全局配置删掉，否则启动不成功

```bash
[root@mycat1 ~]# tail -7 /usr/local/mycat/conf/server.xml

	<user name="ljh">
		<property name="password">redhat</property>
		<property name="schemas">jiaowu</property>
	</user>

</mycat:server>

```

* 启动

```bash
[root@mycat1 ~]# /usr/local/mycat/bin/mycat status
Mycat-server is running (12225).
[root@mycat1 ~]# netstat -tunlp | grep java
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      12227/java          
tcp6       0      0 :::1984                 :::*                    LISTEN      12227/java          
tcp6       0      0 :::8066                 :::*                    LISTEN      12227/java          
tcp6       0      0 :::9066                 :::*                    LISTEN      12227/java          
tcp6       0      0 :::36363                :::*                    LISTEN      12227/java          
tcp6       0      0 :::33273                :::*                    LISTEN      12227/java  
```

* 同样的操作配置mycat2

### 3、测试读写分离是否成功

```bash
[root@slave ~]# mysql -uljh -predhat -h 192.168.152.12 -P 8066
MySQL [jiaowu]> insert into tutors(Tname) values("user1");
Query OK, 1 row affected (0.00 sec)

MySQL [jiaowu]> select * from tutors ;
+-----+--------------+--------+------+
| TID | Tname        | Gender | Age  |
+-----+--------------+--------+------+
|   1 | HongQigong   | M      |   93 |
|   2 | HuangYaoshi  | M      |   63 |
|   3 | Miejueshitai | F      |   72 |
|   4 | OuYangfeng   | M      |   76 |
|   5 | YiDeng       | M      |   90 |
|   6 | YuCanghai    | M      |   56 |
|   7 | Jinlunfawang | M      |   67 |
|   8 | HuYidao      | M      |   42 |
|   9 | NingZhongze  | F      |   49 |
|  10 | user1        | M      | NULL |
+-----+--------------+--------+------+
#mycat1读写正常


[root@slave ~]# mysql -uljh -predhat -h 192.168.152.13 -P 8066
MySQL [jiaowu]> insert into  tutors (Tname) values("user2");
Query OK, 1 row affected (0.01 sec)

MySQL [jiaowu]> select * from tutors ;
+-----+--------------+--------+------+
| TID | Tname        | Gender | Age  |
+-----+--------------+--------+------+
|   1 | HongQigong   | M      |   93 |
|   2 | HuangYaoshi  | M      |   63 |
|   3 | Miejueshitai | F      |   72 |
|   4 | OuYangfeng   | M      |   76 |
|   5 | YiDeng       | M      |   90 |
|   6 | YuCanghai    | M      |   56 |
|   7 | Jinlunfawang | M      |   67 |
|   8 | HuYidao      | M      |   42 |
|   9 | NingZhongze  | F      |   49 |
|  10 | user1        | M      | NULL |
|  11 | user2        | M      | NULL |
+-----+--------------+--------+------+
#mycat2正常
```

### 4、配置前端两台haproxy

#### 1)haproxy1配置

```bash
[root@haproxy01 ~]# yum -y install haproxy
[root@haproxy01 ~]# vim /etc/haproxy/haproxy.cfg 
#其他的都不动，就直接添加一个listen
listen mycat_service
#为了方便VIP的连接，将监听地址成0.0.0.0
    bind 0.0.0.0:8066
    mode tcp
    balance source
    server mycat02  192.168.152.12:8066 check
    server mycat01  192.168.152.13:8066 check
[root@haproxy01 ~]# systemctl start haproxy
[root@haproxy01 ~]# systemctl enable haproxy
```

#### 2)连接测试haproxy1能否代理

```bash
[root@slave ~]# mysql -uljh -predhat -h 192.168.152.10 -P 8066
MySQL [jiaowu]> insert into  tutors (Tname) values("user3");
Query OK, 1 row affected (0.00 sec)

MySQL [jiaowu]> select  * from tutors ;
+-----+--------------+--------+------+
| TID | Tname        | Gender | Age  |
+-----+--------------+--------+------+
|   1 | HongQigong   | M      |   93 |
|   2 | HuangYaoshi  | M      |   63 |
|   3 | Miejueshitai | F      |   72 |
|   4 | OuYangfeng   | M      |   76 |
|   5 | YiDeng       | M      |   90 |
|   6 | YuCanghai    | M      |   56 |
|   7 | Jinlunfawang | M      |   67 |
|   8 | HuYidao      | M      |   42 |
|   9 | NingZhongze  | F      |   49 |
|  10 | user1        | M      | NULL |
|  11 | user2        | M      | NULL |
|  12 | user3        | M      | NULL |
+-----+--------------+--------+------+
```

#### 3)相同方法配置haproxy2，

#### 4)客户端连接并且测试



### 5、利用keepalived实现haproxy高可用

#### 1)先部署haproxy1的机器

```bash
[root@haproxy01 ~]# yum -y install keepalived
[root@haproxy01 ~]# vim /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

#定义keepalived组ID
global_defs {
   router_id keepalived_01
}

#定义检测脚本，检测haproxy状态，宕机就停掉keepalived，实现VIP转移
vrrp_script check_haproxy {
        script "/etc/keepalived/check_haproxy.sh"
        interval 1
}

#定义虚拟组
vrrp_instance mycat_service {
#不设置为MASTER是因为，为了避免宕机恢复后主备之间抢占资源，所以设置为不让VIP转移回来
    state BACKUP
    nopreempt
    
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass redhat
    }
#设置VIP以单播的形式返送心跳
    unicast_src_ip 192.168.152.10
    unicast_peer  {
        192.168.152.11
    }
#调用上边写的脚本
    track_script {
        check_haproxy
    }
#定义虚拟VIP
    virtual_ipaddress {
        192.168.152.100
    }
}
[root@haproxy01 ~]# cat /etc/keepalived/check_haproxy.sh 
#!/bin/bash
#

netstat -tunlp | grep haproxy &> /dev/null

if [ $? -ne 0 ];then
systemctl stop keepalived
fi
 
```

#### 2)为了避免出现问题

> Can’t open PID file /var/run/keepalived.pid (yet?) after start: No such file or directory
>
> 将脚本中的 systemctl stop keepalived
>
>  改为  killall keepalived

#### 3)查看是否生成VIP，

```bash
[root@haproxy01 ~]# ip a
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000

    inet 192.168.152.100/32 scope global ens33
       valid_lft forever preferred_lft forever
```

#### 4)以相同的方式配置keepalived2

#### 5)最后测试VIP漂移，读写分离

