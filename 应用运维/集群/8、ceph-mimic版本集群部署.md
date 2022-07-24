[toc]

## 一、环境描述

> * 192.168.152.10	ceph-node01.linux.com	ceph集群节点/ceph-deploy    /dev/sdb
> * 192.168.152.11	ceph-node02.linux.com	ceph集群节点    /dev/sdb
> * 192.168.152.12	ceph-node03.linux.com	ceph集群节点    /dev/sdb
> * 192.168.183.13	client.linux.com		业务服务器 

## 二 、系统环境准备

### 1、所有节点关闭防火墙、SELinux

### 2、所有节点配置主机名解析

* 每个主机上都是这四个解析，这里不在展示

```bash
[root@ceph-node01 ~]# tail -4 /etc/hosts
192.168.152.10	ceph-node01.linux.com ceph-node01
192.168.152.11	ceph-node02.linux.com ceph-node02
192.168.152.12	ceph-node03.linux.com ceph-node03
192.168.152.13	ceph-node04.linux.com ceph-node04
[root@ceph-node01 ~]# for i in 11 12 13
> do
> rsync -av /etc/hosts root@192.168.152.$i:/etc/hosts
> done
```

### 3、所有节点配置免密SSH

```bash
[root@ceph-node01 ~]# ssh-keygen -t rsa 
Generating public/private rsa key pair.
[root@ceph-node01 ~]# mv .ssh/id_rsa.pub /root/.ssh/authorized_keys
[root@ceph-node01 ~]# for i in 11 12 13 
> do
> rsync -av /root/.ssh/ root@192.168.152.$i:/root/.ssh/
> done
[root@ceph-node01 ~]# for i in 10 11 12 13
> do
> ssh root@192.168.152.$i date
> done
Fri Mar 25 18:01:38 CST 2022
Fri Mar 25 18:01:39 CST 2022
Fri Mar 25 18:01:39 CST 2022
Fri Mar 25 18:01:39 CST 2022
```

### 4、所有节点配置时间同步

```bash
[root@ceph-node01 ~]# for i in 10 11 12 13
> do
> echo "* */2  * * * /usr/sbin/ntpdate 120.25.115.20 &> /dev/null" > /var/spool/cron/root 
> done
[root@ceph-node01 ~]# for i in 10 11 12 13
> do
> ssh root@192.168.152.$i crontab -l
> done
* */2  * * * /usr/sbin/ntpdate 120.25.115.20 &> /dev/null
* */2  * * * /usr/sbin/ntpdate 120.25.115.20 &> /dev/null
* */2  * * * /usr/sbin/ntpdate 120.25.115.20 &> /dev/null
```

### 5、所有节点配置ceph软件仓库

```bash
#第一个包是配置阿里云的软件仓库
#第二个包是添加epel源，提供ceph的依赖包
[root@ceph-node01 ~]# for i in 10 11 12 13
> do
> ssh root@192.168.152.$i wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
> ssh root@192.168.152.$i wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
> done
[root@ceph-node01 ~]# cat /etc/yum.repos.d/ceph.repo 
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/x86_64/
enabled=1
gpgcheck=0
priority=1
 
[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/noarch/
enabled=1
gpgcheck=0
priority=1
 
[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/SRPMS
enabled=1
gpgcheck=0
priority=1
[root@ceph-node01 ~]# for i in 11 12 
> do
> rsync -av /etc/yum.repos.d/ceph.repo root@192.168.152.$i:/etc/yum.repos.d/
> done

```

### 6、所有节点更新仓库包

* 在node01、node02、node03上执行操作

```bash
[root@ceph-node01 ~]# yum update
[root@ceph-node01 ~]# reboot
```

```bash
[root@ceph-node01 ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)

[root@ceph-node01 ~]# uname -r
3.10.0-1160.45.1.el7.x86_64

```

## 三、在主节点安装ceph-deploy工具

### 1、安装ceph-deploy工具

* 定义node01为主节点

```bash
[root@ceph-node01 ~]# yum install -y ceph-deploy
```

### 2、创建集群目录

```bash
[root@ceph-node01 ~]# mkdir /etc/ceph
[root@ceph-node01 ~]# cd /etc/ceph
```

## 四、创建ceph集群

### 1、开始创建

```bash
[root@ceph-node01 ceph]# ceph-deploy new ceph-node01
.......
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
#集群创建完毕后会在当前目录生成如下文件
-rw-r--r-- 1 root root    236 Mar 25 14:55 ceph.conf			#集群配置文件
-rw-r--r-- 1 root root 213004 Mar 25 14:56 ceph-deploy-ceph.log	 #ceph-deploy日志
-rw------- 1 root root     73 Mar 25 14:34 ceph.mon.keyring		 # ceph monitor认证 的令牌

```

### 2、所有ceph集群节点安装ceph相关软件

* 可以分别在每个节点上安装

```bash
[root@ceph-node01 ~]# yum install -y ceph ceph-radosgw 
[root@ceph-node01 ~]# ceph -v
ceph version 13.2.10 (564bdc4ae87418a232fc901524470e1a0f76d641) mimic (stable)
```

* 也可以在主节点上利用ceph-deploy工具在集群节点安装软件**(前提网络必须要畅通)**

```bash
[root@ceph-node01 ~]#  ceph-deploy install ceph-node01 ceph-node02 ceph-node03
```

### 3、在客户端安装ceph-common软件

* client.linux.com

```bash
[root@client ~]# yum install -y ceph-common
```

### 4、创建ceph monitor

* 编辑ceph-node01配置文件，添加public network配置

```bash
[root@ceph-node01 ceph]# cat ceph.conf 
[global]
fsid = 1ca6c582-8598-4300-93c4-4da0f2b16d32
mon_initial_members = ceph-node01
mon_host = 192.168.183.10
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
public network = 192.168.183.0/24
```

* monitor初始化，将ceph-node01配置为monitor

```bash
[root@ceph-node01 ceph]# ceph-deploy mon create-initial
[ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
..........
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-osd.keyring
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-rgw.keyring
[ceph_deploy.gatherkeys][INFO  ] Destroy temp directory /tmp/tmp9GPugf

#生成如下配置文件
[root@ceph-node01 ceph]# ls
ceph.bootstrap-mds.keyring  ceph.bootstrap-osd.keyring  ceph.client.admin.keyring  ceph-deploy-ceph.log  rbdmap
ceph.bootstrap-mgr.keyring  ceph.bootstrap-rgw.keyring  ceph.conf                  ceph.mon.keyring

```

* 查看monitor健康状态

```bash
[root@ceph-node01 ceph]# ceph health
HEALTH_OK
```

### 5、将配置信息同步到所有节点

```bash
[root@ceph-node01 ceph]# ceph-deploy admin ceph-node01 ceph-node02 ceph-node03 
[ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy admin ceph-node01 ceph-node02 ceph-node03
..........
[ceph-node03][DEBUG ] detect machine type
[ceph-node03][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf

```

* 集群中的其他节点会出现如下信息

```bash
[root@ceph-node02 ~]# ls /etc/ceph/
ceph.client.admin.keyring  ceph.conf  rbdmap  tmp7nnJHQ
```

### 6、查看集群的健康状态

```bash
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum ceph-node01         # 1个monitor服务
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs: 
```

### 7、为避免monitor成为单点故障，配置node02\node03配置成monitor

```bash
[root@ceph-node01 ceph]# ceph-deploy mon add ceph-node02
[root@ceph-node01 ceph]# ceph-deploy mon add ceph-node03
 [root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

## 五、创建ceph mgr

> * ceph自L版本之后，添加Ceph Manager Daemon，简称ceph-mgr
> * 该组件的出现可以缓解对monitor的压力，分担了monitor的工作，例如：插件管理等，以更好地管理集群

### 1、在ceph-node01节点创建mgr服务

```bash
[root@ceph-node01 ~]# ceph-deploy mgr create ceph-node01
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active)      # 1个mgr服务
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs: 
```

### 2、添加多个mgr避免单点故障

```bash
[root@ceph-node01 ceph]# ceph-deploy mgr create ceph-node02
[root@ceph-node01 ceph]# ceph-deploy mgr create ceph-node03
 [root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs: 
```

## 六、创建OSD（数据盘）

### 1、初始化磁盘，清空磁盘数据

```bash
[root@ceph-node01 ceph]# ceph-deploy disk zap ceph-node01 /dev/sdb 
[root@ceph-node01 ceph]# ceph-deploy disk zap ceph-node02 /dev/sdb 
[root@ceph-node01 ceph]# ceph-deploy disk zap ceph-node03 /dev/sdb 
```

### 2、将磁盘创建为OSD

```bash
[root@ceph-node01 ceph]# ceph-deploy osd create --data /dev/sdb ceph-node01
[root@ceph-node01 ceph]# ceph-deploy osd create --data /dev/sdb ceph-node02
[root@ceph-node01 ceph]# ceph-deploy osd create --data /dev/sdb ceph-node03
```

### 3、查看状态

```bash
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    osd: 3 osds: 3 up, 3 in     # 3个osd
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail    # 容量为所有磁盘之和
    pgs:
```

## 七、启用ceph dashboard

### 1、确认主节点

```bash
[root@ceph-node02 ~]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   6 pools, 48 pgs
    objects: 194  objects, 2.4 KiB
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail
    pgs:     48 active+clean
```

### 2、查看开启或者关闭的模块

```bash
[root@ceph-node01 ceph]# ceph mgr module ls
{
    "enabled_modules": [
        "balancer",
        "crash",
        "iostat",
        "restful",
        "status"
    ],
    "disabled_modules": [
        {
            "name": "dashboard",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "hello",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "influx",
            "can_run": false,
            "error_string": "influxdb python module not found"
        },
        {
            "name": "localpool",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "prometheus",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "selftest",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "smart",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "telegraf",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "telemetry",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "zabbix",
            "can_run": true,
            "error_string": ""
        }
    ]
}
```

### 3、开启ceph dashboard模块

```bash
[root@ceph-node01 ceph]# ceph mgr module enable dashboard
```

* 如果报错Error ENOENT: all mgr daemons do not support module 'dashboard', pass --force to force enablement，需要在所有mgr节点安装ceph-mgr-dashboard软件

### 4、创建自签证书

* 因为图形面板也是基于HTTPS协议，所以要先创建自签证书，密钥，

```bash
[root@ceph-node01 ceph]# ceph dashboard create-self-signed-cert 
Self-signed certificate created
```

### 5、生成dashboard需要的证书，key

```bash
[root@ceph-node01 ceph]# mkdir /etc/mgr-dashboard
[root@ceph-node01 ceph]# cd /etc/mgr-dashboard/

[root@ceph-node01 mgr-dashboard]# openssl req -new -nodes -x509 -subj "/O=IT-ceph/CN=cn" -days 3650 -keyout dashboard.key -out dashboard.crt -extensions v3_ca
Generating a 2048 bit RSA private key
.............................................................................................................+++
........+++
writing new private key to 'dashboard.key'
-----

[root@ceph-node01 mgr-dashboard]# ls
dashboard.crt  dashboard.key
```

### 6、定义dashboard的访问地址

* 默认的端口是：8443

```bash
[root@ceph-node01 mgr-dashboard]# ceph config set mgr mgr/dashboard/server_addr 192.168.183.10
[root@ceph-node01 mgr-dashboard]# ceph config set mgr mgr/dashboard/server_port 8080
```

### 7、重启dashboard生效

```bash
[root@ceph-node01 mgr-dashboard]# ceph mgr module disable dashboard
[root@ceph-node01 mgr-dashboard]# ceph mgr module enable dashboard
```

### 8、查看mgr service

* 可以看到dashboard已经开启

```bash
[root@ceph-node01 mgr-dashboard]# ceph mgr services
{
    "dashboard": "https://192.168.183.10:8080/"
}

[root@ceph-node01 mgr-dashboard]# ceph mgr module ls
{
    "enabled_modules": [
        "balancer",
        "crash",
        "dashboard",
        "iostat",
        "restful",
        "status"
    ],
    "disabled_modules": [
        {
            "name": "hello",
            "can_run": true,
            "error_string": ""
        },
        {
            "name": "influx",
            "can_run": false,
            "error_string": "influxdb python module not found"
        },
....................................................................................................
```

### 9、设置dashboard认证用户名、密码

```bash
[root@ceph-node01 mgr-dashboard]# ceph dashboard set-login-credentials martin redhat
Username and password updated
```

### 10、访问dashboard

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/4ECB6287B3B2485B8F3972316DA7120C/0009C57E11F74388A9400AB2E8FDB5EE/22768](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/4ECB6287B3B2485B8F3972316DA7120C/0009C57E11F74388A9400AB2E8FDB5EE/22768)