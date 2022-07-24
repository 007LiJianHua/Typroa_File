[toc]

## 一、仓库类型

* 公有仓库
  * DockerHub

- - https://hub.docker.com/

* 私有仓库
  * registry
  * harbor

## 二、DockerHub的使用

### 1、登录DockerHub

* 前提是在DockerHub官网自己得先注册一个账号

```bash
[root@localhost ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: martinwjc
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### 2、上传镜像

```bash
[root@localhost ~]# docker tag nginx:1.14 martinwjc/nginx:1.14
[root@localhost ~]# docker push martinwjc/nginx:1.14 
```

### 3、退出

```bash
# docker logout
```

## 三、harbor介绍

* Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，通过添加一些企业必需的功能特性，例如安全、标识和管理等
* 基于Vmware开源 

### 1、特性

- 基于角色的访问控制
- 镜像复制
- 图形化用户界面
- AD/LDAP 支持
- 审计管理
- 国际化
- RESTful API
- 部署简单

### 2、部署方式

- on-line	在线
- off-line	离线

## 四、Harbor部署安装

* 环境描述

```bash
192.158.140.10	docker服务器
192.168.140.11	harbor服务器
192.168.140。12	CA服务器
```



### 1、添加主机名解析

```bash
[root@harbor ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.140.11	harbor.linux.com
```

### 2、安装docker

```bash
[root@localhost ~]# cat /etc/yum.repos.d/docker.repo 
[docker-ce]
name=docker-ce
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7.9/x86_64/stable/
enabled=1
gpgcheck=0
[root@localhost ~]# yum -y install docker-ce
....
[root@localhost ~]# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
docker version >= 1.12
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
Success.
You need to restart docker to take effect: sudo systemctl restart docker 
[root@localhost ~]# cat /etc/docker/daemon.json 
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
[root@localhost ~]# systemctl restart docker
[root@localhost ~]# systemctl daemon-reload 
```



### 3、安装docker-compose

```bash
[root@harbor ~]# mv docker-compose /usr/local/bin/
[root@harbor ~]# chmod a+x /usr/local/bin/docker-compose 
```

### 4、安装harbor

```bash
[root@harbor ~]# tar xf harbor-offline-installer-v2.2.2.tgz 
[root@harbor ~]# cp harbor/harbor.yml.tmpl harbor/harbor.yml
```

### 5、配置CA为harbor签发证书

```bash
[root@ca ~]# touch /etc/pki/CA/index.txt
[root@ca ~]# echo 01 > /etc/pki/CA/serial

[root@ca ~]# openssl genrsa -out /etc/pki/CA/private/cakey.pem 1024
Generating RSA private key, 1024 bit long modulus
......++++++
....................++++++
e is 65537 (0x10001)

[root@ca ~]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
```

```bash
[root@harbor ~]# mkdir /root/harbor/ssl
[root@harbor ~]# openssl genrsa -out /root/harbor/ssl/harbor.key 2048
[root@harbor ~]# openssl req -new -key /root/harbor/ssl/harbor.key -out /root/harbor/ssl/harbor.csr
```

```bash
[root@harbor ~]# rsync -av /root/harbor/ssl/harbor.csr root@192.168.140.12:/tmp/
[root@ca ~]# openssl ca -in /tmp/harbor.csr  -out /etc/pki/CA/certs/harbor.crt -days 3650
[root@ca ~]# rsync -av /etc/pki/CA/certs/harbor.crt root@192.168.140.11:/root/harbor/ssl

[root@harbor ~]# ls /root/harbor/ssl/
harbor.crt  harbor.csr  harbor.key
```

### 6、编辑harbor配置文件

```bash
[root@harbor harbor]# vim harbor.yml

hostname: harbor.linux.com			#将这个名字修改为CA申请的服务器名字

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /root/harbor/ssl/harbor.linux.com.crt		#修改证书位置
  private_key: /root/harbor/ssl/harbor.linux.com.key		#修改密钥位置
```

### 7、启动harbor

```bash
[root@harbor harbor]# ./prepare 
prepare base dir is set to /root/harbor
Unable to find image 'goharbor/prepare:v2.2.2' locally
v2.2.2: Pulling from goharbor/prepare
b31150c04016: Pull complete 
d504272addf9: Pull complete 
a9c2d9be0ec7: Pull complete 
ba14108b237f: Pull complete 
888a2dd12a77: Pull complete 
08591f736052: Pull complete 
e9a06c50605c: Pull complete 
fcc257111f80: Pull complete 
Digest: sha256:d12185f2c925416fa260d2af8764d8c27d35b4f66d9bcff67bf7e35d9409789e
Status: Downloaded newer image for goharbor/prepare:v2.2.2
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
```

```bash
[root@harbor harbor]# ./install.sh 

Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-db     ... done
Creating harbor-portal ... done
Creating redis         ... done
Creating registryctl   ... done
Creating registry      ... done
Creating harbor-core   ... done
Creating nginx             ... done
Creating harbor-jobservice ... done
✔ ----Harbor has been installed and started successfully.----
```

```bash

[root@harbor harbor]# docker image ls
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
goharbor/harbor-exporter        v2.2.2    d63334489be4   2 months ago   76.4MB
goharbor/chartmuseum-photon     v2.2.2    c3c6b2f81c7c   2 months ago   165MB
goharbor/redis-photon           v2.2.2    11a777ead643   2 months ago   69MB
goharbor/trivy-adapter-photon   v2.2.2    a0cd0b967c47   2 months ago   120MB
goharbor/notary-server-photon   v2.2.2    f963e34d9130   2 months ago   102MB
goharbor/notary-signer-photon   v2.2.2    c3ce4df1b104   2 months ago   98.5MB
goharbor/harbor-registryctl     v2.2.2    461e67c4ec3d   2 months ago   128MB
goharbor/registry-photon        v2.2.2    fb9295e771e6   2 months ago   77.3MB
goharbor/nginx-photon           v2.2.2    6744b15891f6   2 months ago   40.4MB
goharbor/harbor-log             v2.2.2    3837bbad1328   2 months ago   108MB
goharbor/harbor-jobservice      v2.2.2    c9db96b1363f   2 months ago   163MB
goharbor/harbor-core            v2.2.2    2e6b040afa40   2 months ago   148MB
goharbor/harbor-portal          v2.2.2    c240f7374709   2 months ago   51.1MB
goharbor/harbor-db              v2.2.2    e6b67be16b5b   2 months ago   177MB
goharbor/prepare                v2.2.2    eb938b7f85aa   2 months ago   165MB
```

```bash
[root@harbor harbor]# docker ps -a
CONTAINER ID   IMAGE                                COMMAND                  CREATED         STATUS                        PORTS                                                                            NAMES
2c20abb4d255   goharbor/harbor-jobservice:v2.2.2    "/harbor/entrypoint.…"   2 minutes ago   Up About a minute (healthy)                                                                                    harbor-jobservice
b5393b959230   goharbor/nginx-photon:v2.2.2         "nginx -g 'daemon of…"   2 minutes ago   Up About a minute (healthy)   0.0.0.0:80->8080/tcp, :::80->8080/tcp, 0.0.0.0:443->8443/tcp, :::443->8443/tcp   nginx
64f26bafbd90   goharbor/harbor-core:v2.2.2          "/harbor/entrypoint.…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         harbor-core
6c8a836e6c10   goharbor/registry-photon:v2.2.2      "/home/harbor/entryp…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         registry
048bb45ebb42   goharbor/redis-photon:v2.2.2         "redis-server /etc/r…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         redis
bf345e4dbce7   goharbor/harbor-registryctl:v2.2.2   "/home/harbor/start.…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         registryctl
26eaff3ead77   goharbor/harbor-portal:v2.2.2        "nginx -g 'daemon of…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         harbor-portal
b57fdf68b23a   goharbor/harbor-db:v2.2.2            "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes (healthy)                                                                                         harbor-db
61261e5bb47c   goharbor/harbor-log:v2.2.2           "/bin/sh -c /usr/loc…"   2 minutes ago   Up 2 minutes (healthy)        127.0.0.1:1514->10514/tcp     
```

### 8、通过WebUI访问harbor

https://harbor.linux.com

测试：

* 创建项目
* 创建用户
* 将用户添加到项目中
* 用户授权

### 9、上传镜像

* 登录仓库

```bash
Harbor仓库启用了https后，docker服务器要能正常登录访问仓库，需要将harbor的证书拷贝到docker服务器
#这里存放证书的位置是死的，不能变更
[root@test ~]# mkdir /etc/docker/certs.d/harbor.linux.com -p

[root@harbor ~]# rsync -av harbor/ssl/harbor.crt root@192.168.140.10:/etc/docker/certs.d/harbor.linux.com
```

<font color="red">**出现错误:** </font>

<font color="red">Error response from daemon: Get "https://harbor.linux.com/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0</font>

<font color="red">**原因说明:** </font>

<font color="red">**go从1.15版本后，不支持传统证书；需要使用SAN证书进行认证**</font>

<font color="red">**使用openssl生成证书的方式得变**  [**https://www.cnblogs.com/jackluo/p/13841286.html**](https://www.cnblogs.com/jackluo/p/13841286.html)></font>



**临时解决办法**

* --insecure-registry=harbor.linux.com

* - docker服务登录harbor使用http协议 

```bash
[root@node01 ~]# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock $DOCKER_NETWORK_OPTIONS --insecure-registry=harbor.linux.com
```

```bash
[root@node01 ~]# systemctl daemon-reload
[root@node01 ~]# systemctl restart docker
```

```bash
[root@localhost ~]# docker login harbor.linux.com
Username: martin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

* 上传镜像

```bash
[root@localhost ~]# docker tag nginx:1.14 harbor.linux.com/project_yw/nginx:1.14 
[root@localhost ~]# docker push harbor.linux.com/project_yw/nginx:1.14
```

## 五、harbor组件说明

![https://note.youdao.com/yws/public/resource/177109985ddced8703a06ef7db3ec83d/xmlnote/32FB114D1E0E4DC9A44CFE9987E8744E/447DD8D6E7714539BAE5E273AB600381/17332](https://s2.loli.net/2022/04/05/mAR7Kvwg1LhEb3T.png)

- redis

- - 负责存储会话信息

- harbor-db

- - 负责存储仓库数据(项目名称、镜像名称)

- registry

- - 镜像的上传、下载

- Log collector

- - 记录日志

- harbor-core

- - 提供web UI界面 

  - 颁发令牌token

  - webhook

  - - 负责搜集镜像变化的信息，通知给UI进行展示 

- proxy

- - 提供反向代理

- Job service

- - 提供镜像复制功能



```bash

[root@harbor harbor]# docker images | awk '{print $1}'
REPOSITORY
goharbor/harbor-exporter	
goharbor/chartmuseum-photon
goharbor/redis-photon				#用来存储会话信息
goharbor/trivy-adapter-photon
goharbor/notary-server-photon
goharbor/notary-signer-photon
goharbor/harbor-registryctl
goharbor/registry-photon			#镜像的上传、下载
goharbor/nginx-photon
goharbor/harbor-log					#记录日志
goharbor/harbor-jobservice
goharbor/harbor-core
goharbor/harbor-portal
goharbor/harbor-db					#负责存储仓库的数据
goharbor/prepare

```

