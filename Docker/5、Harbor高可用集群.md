[toc]

## 一、实现方式

**实现的方式有以下两种**

* 利用harbor自带的复制功能
  * 两个harbor节点在进行复制的时候，如果其中一个节点故障，此时有新的镜像，并不会复制到新的节点，并且在故障修复后也不会自动复制数据，会导致数据不一致的问题
* 基于共享存储的方案实现
  * 将数据库，redis独立出来
    * harbor1.6版本以后，数据库仅支持postgreSQL
  * 利用共享存储（NFS、IPSAN、GlusterFS）存储镜像



## 二、部署Harbor高可用集群

### 0、环境描述

```bash
192.168.140.11		harbor服务器
192.168.140.13		harbor服务器
192.168.140.12		NFS、Redis、PostgreSQL
```

![image-20220405133749081](https://s2.loli.net/2022/04/05/eDqYbHWLKgzi3Qd.png)

### 1、在两台harbor服务器安装docker、dockers-compose、harbor

### 2、配置NFS服务器

```bash
[root@share_storage ~]# mkdir /data/harbor -p
[root@share_storage ~]# chmod o+w /data/harbor/
```

```bash
[root@share_storage ~]# yum install -y nfs-utils rpcbind 
```

```bash
[root@share_storage ~]# vim /etc/exports
[root@share_storage ~]# cat /etc/exports
/data/harbor	192.168.140.11(rw,no_root_squash)   192.168.140.13(rw,no_root_squash)
 
[root@share_storage ~]# systemctl start nfs-server
[root@share_storage ~]# systemctl enable nfs-server.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.

[root@share_storage ~]# showmount -e localhost 
Export list for localhost:
/data/harbor 192.168.140.13,192.168.140.11
```

### 3、配置两台harbor服务器挂载NFS存储

```bash
[root@harbor_01 ~]# yum install -y nfs-utils 

[root@harbor_01 ~]# vim /etc/fstab 
192.168.140.12:/harbordata	/data	nfs	defaults 0 0

[root@harbor_01 ~]# df -hT | grep nfs
192.168.140.12:/harbordata nfs4       18G  3.0G   15G  18% /data
```



### 4、部署redis服务

```bash
[root@share_storage ~]# yum install -y redis
```

```bash
[root@share_storage ~]# vim /etc/redis.conf 
bind 192.168.140.12
daemonize yes
```

```bash
[root@share_storage ~]# systemctl start redis
[root@share_storage ~]# systemctl enable redis
[root@share_storage ~]# netstat -antp | grep redis
tcp        0      0 192.168.140.12:6379     0.0.0.0:*               LISTEN      17493/redis-server  
```

### 5、部署postgresql

#### 1）安装postgresql

```bash
[root@storage ~]# yum install -y cmake gcc gcc-c++ perl readline readline-devel openssl openssl-devel zlib zlib-devel ncurses-devel readline readline-devel zlib zlib-devel
[root@storage ~]# wget https://ftp.postgresql.org/pub/source/v12.2/postgresql-12.2.tar.gz
[root@storage ~]# tar xf postgresql-12.2.tar.gz 
[root@storage ~]# cd postgresql-12.2/
[root@storage ~]#./configure --prefix=/usr/local/postsql
[root@storage ~]# make && make install
```

#### 2）初始化postgresql

```bash
[root@storage ~]# useradd postgres
[root@storage ~]# mkdir -p /work/harbor-db/{data,temp,log}
[root@storage ~]# chown -R postgres.postgres /work/harbor-db
[root@storage ~]# /usr/local/postgresql/bin/initdb --username=postgres -D /work/harbor-db/data/
```

#### 3）编辑postgresql配置文件

```bash
[root@storage ~]# vim /work/harbor-db/data/postgresql.conf 

data_directory = '/work/harbor-db/data'

listen_addresses = '*'          
port = 5432                           
max_connections = 100   

unix_socket_directories = '/work/harbor-db/temp'
unix_socket_group = ''
unix_socket_permissions = 0777

shared_buffers = 128MB

timezone = 'Asia/Shanghai'
 
logging_collector = on
log_directory = '/work/harbor-db/log'
log_rotation_size = 1GB
log_timezone = 'Asia/Shanghai'
log_min_duration_statement = 100
```

#### 4）指定允许远程连接的客户端

```bash
[root@storage ~]# tail -n 2 /work/harbor-db/data/pg_hba.conf 
host    all		harbor          192.168.140.11/24                 trust
host    all		harbor		192.168.140.13/24                 trust
```

#### 5）启动数据库

```bash
[root@storage ~]# su - postgres
[root@storage ~]# /usr/local/postgresql/bin/pg_ctl -D /work/harbor-db/data/ -l /work/harbor-db/log/start.log start

[postgres@storage ~]$ netstat -antp | grep 5432
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      32215/postgres 
```

#### 6）创建harbor需要的数据库以及远程连接用户

```bash
[root@storage ~]# /usr/local/postgresql/bin/psql -h 127.0.0.1 -p 5432 -U postgres

postgres=# \password postgres
Enter new password: 
Enter it again: 

postgres=# create user harbor with password 'redhat';
CREATE ROLE

postgres=# CREATE DATABASE harbor;
CREATE DATABASE

postgres=# create database harbor_clair;
CREATE DATABASE

postgres=# create database harbor_notary_server;
CREATE DATABASE

postgres=# create database harbor_notary_signer; 
CREATE DATABASE
harbor		
postgres=# GRANT ALL ON DATABASE harbor to harbor;           
GRANT

postgres=# GRANT ALL ON DATABASE harbor_clair to harbor;           
GRANT

postgres=# GRANT ALL ON DATABASE harbor_notary_server to harbor;            
GRANT

postgres=# GRANT ALL ON DATABASE harbor_notary_signer to harbor;                    
GRANT
```

#### 7）远程连接测试数据库

```bash
[root@harbor_02 ~]# psql -h 192.168.140.12 -p 5432 -U harbor -W
```

### 6、安装harbor

```bash
[root@harbor_01 ~]# tar xf harbor-offline-installer-v2.2.1.tgz 
```

### 7、编辑harbor配置文件

```bash
[root@harbor_02 ~]# vim harbor/harbor.yml

hostname: harbor.linux.com

注释https相关配置，本案例中没有配置证书
#https:
	# port: 443
	# certificate: /usr/local/harbor/ssl/harbor.ssl	
	# private_key: /usr/local/harbor/ssl/harbor.key

data_volume: /data

harbor_admin_password: Harbor12345


注释或删除harbor自带的数据库配置，后续配置外部数据库连接
#database:
	# password: root123
	# max_idle_conns: 50
	# max_open_conns: 1000

配置harbor连接外部postgresql数据库
external_database:
  harbor:
    host: 192.168.140.12
    port: 5432
    db_name: harbor
    username: harbor
    password: redhat
    ssl_mode: disable
    max_idle_conns: 2
    max_open_conns: 0
  notary_signer:
    host: 192.168.140.12
    port: 5432
    db_name: harbor_notary_signer
    username: harbor
    password: redhat
    ssl_mode: disable
  notary_server:
    host: 192.168.140.12
    port: 5432
    db_name: harbor_notary_server
    username: harbor
    password: redhat
    ssl_mode: disable

配置连接外部redis存储会话信息
external_redis:
   host: 192.168.140.12:6379
   password:
   
   registry_db_index: 1
   jobservice_db_index: 2
   chartmuseum_db_index: 3
   trivy_db_index: 5
   idle_timeout_seconds: 30
```

### 8、启动harbor

```bash
[root@harbor_01 harbor]# ./prepare
[root@harbor_01 harbor]# ./install.sh 
```

```bash
[root@harbor_02 harbor]# docker ps -a
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                                                                            NAMES
7839506e6909   goharbor/nginx-photon:v2.2.2         "nginx -g 'daemon of…"   19 seconds ago   Up 16 seconds (health: starting)   0.0.0.0:80->8080/tcp, :::80->8080/tcp, 0.0.0.0:443->8443/tcp, :::443->8443/tcp   nginx
0685bef08838   goharbor/harbor-jobservice:v2.2.2    "/harbor/entrypoint.…"   19 seconds ago   Up 16 seconds (health: starting)                                                                                    harbor-jobservice
0745cdc06650   goharbor/harbor-core:v2.2.2          "/harbor/entrypoint.…"   19 seconds ago   Up 18 seconds (health: starting)                                                                                    harbor-core
2dca99a9b8d9   goharbor/harbor-portal:v2.2.2        "nginx -g 'daemon of…"   20 seconds ago   Up 19 seconds (health: starting)                                                                                    harbor-portal
3b3f3a4d4192   goharbor/registry-photon:v2.2.2      "/home/harbor/entryp…"   20 seconds ago   Up 19 seconds (health: starting)                                                                                    registry
db630535b7e8   goharbor/harbor-registryctl:v2.2.2   "/home/harbor/start.…"   20 seconds ago   Up 19 seconds (health: starting)                                                                                    registryctl
27103477a551   goharbor/harbor-log:v2.2.2           "/bin/sh -c /usr/loc…"   21 seconds ago   Up 20 seconds (health: starting)   127.0.0.1:1514->10514/tcp 
```

<font color="red">**所有Harbor节点配置保持一致**</font>

### 9、测试harbor集群

> 连接任意一台harbor上传镜像，上传镜像完毕后。修改本地hosts文件解析到其他harbor服务器，再次访问harbor web界面。第一会发现不需要登录直接可以访问数据，第二会看到同样的项目及镜像