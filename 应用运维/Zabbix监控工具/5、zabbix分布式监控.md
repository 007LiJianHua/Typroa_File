[toc]

## 一、分布式监控介绍

在被监控设备数量过多时，可以zabbix proxy减缓server的工作负载 

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/BA9E7A7CE312418794266FFBB84268DA/14727](https://s2.loli.net/2022/04/06/QAUC9afXLlejqDG.png)

## 二、分布式监控部署

### 1、安装部署zabbix-proxy

```bash
[root@zabbix-server ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

[root@zabbix_proxy ~]# yum install -y zabbix-proxy-mysql mariadb-server
```

### 2、创建zabbix proxy数据库

```bash
MariaDB [(none)]> create database zabbix_proxy charset utf8 collate utf8_bin;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on zabbix_proxy.* to 'proxyuser'@'localhost' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### 3、导入proxy需要的表

```bash
[root@zabbix_proxy ~]# cd /usr/share/doc/zabbix-proxy-mysql-4.4.10/
[root@zabbix_proxy zabbix-proxy-mysql-4.4.10]# ls
AUTHORS  ChangeLog  COPYING  NEWS  README  schema.sql.gz
[root@zabbix_proxy zabbix-proxy-mysql-4.4.10]# 
[root@zabbix_proxy zabbix-proxy-mysql-4.4.10]# zcat schema.sql.gz | mysql -uroot zabbix_proxy
```

### 4、编辑zabbix_proxy配置文件

```bash
[root@zabbix_proxy ~]# vim /etc/zabbix/zabbix_proxy.conf 
Server=192.168.140.10
Hostname=zabbix_proxy.linux.com
DBHost=localhost
DBName=zabbix_proxy
DBUser=proxyuser
DBPassword=redhat
DBSocket=/var/lib/mysql/mysql.sock
```

### 5、启动zabbix-proxy

```bash
[root@zabbix_proxy ~]# systemctl start zabbix-proxy.service 
[root@zabbix_proxy ~]# systemctl enable zabbix-proxy.service

[root@zabbix_proxy ~]# netstat -antp | grep zabbix
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      7437/zabbix_proxy   
tcp6       0      0 :::10051                :::*                    LISTEN      7437/zabbix_proxy   
```

### 6、在Web界面添加代理

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/553C0BD1B3E242C6A5070FD18A8A3538/14752](https://s2.loli.net/2022/04/06/CO5dH46V1icKJjB.png)

### 7、在被监控端安装zabbix-agent

```bash
Server=代理地址
ServerActive=代理地址
Hostname=node04.linux.com
```

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/7DD2B85B66BE43479A5BB538D7E6C839/14759](https://s2.loli.net/2022/04/06/bkX9OGiSNR1dsHt.png)

## 三、Web监测

web服务的访问质量

1、访问速度 

2、响应时间

### 1、创建Web场景

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/BBDB51C8B8244ECCA7DCAA0C66EBF5E1/14766](https://s2.loli.net/2022/04/06/N5cJ81RYWdbeTIp.png)

### 2、创建步骤

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/8666E50C70F945E184B7F3A36C85B5AF/14769](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/8666E50C70F945E184B7F3A36C85B5AF/14769)

### 3、查看

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/E2A59B29502047348FF6E20CCE06FF8C/5DA992ACA37340BE9CA5CA6334683011/14772](https://s2.loli.net/2022/04/06/hd8rIUXGPpFzxO7.png)