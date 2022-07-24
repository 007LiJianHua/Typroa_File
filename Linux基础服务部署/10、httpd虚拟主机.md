# httpd虚拟主机

[toc]

## 一、虚拟主机介绍

### 1、作用

* 在一台服务器可以部署多套网站
* 注意：
  * 虚拟主机配置后，主配置文件中部署的网站会失效！！！

### 2、类型

* 基于名称的虚拟主机（常用）
  * 不同的网站指定不同的ServerName、DocumentRoot
* 基于IP地址的虚拟主机
  * 不同的网站监听在不同的IP地址上
* 基于端口的虚拟主机

### 3、配置虚拟主机

```bash
<VirtualHost IP:port>
    DocumentRoot xxxxx
    ServerName xxxxx
    ErrorLog xxxxxxxxxxxx
    CustomLog xxxxxxxxxxxxxxxx
</VirtualHost>
```

## 二、基于名称的虚拟主机



网站名称:  vedio.linux.com	网页目录: /var/www/html/vedio

网站名称:  mp3.linux.com	网页目录: /mp3

### 1、配置vedio网站

* 创建网页目录、测试首页

```bash
[root@localhost ~]# mkdir /var/www/html/vedio
[root@localhost ~]# vim /var/www/html/vedio/index.html
<h1> Welcome To Vedio's Web </h1>
```

* 编辑虚拟主机配置文件

```bash
[root@localhost ~]# vim /etc/httpd/conf.d/vedio.conf
<VirtualHost 192.168.150.10:80>
    ServerName vedio.linux.com
    DocumentRoot /var/www/html/vedio
    ErrorLog /var/log/httpd/vedio_error.log
    CustomLog /var/log/httpd/vedio_access.log combined
</VirtualHost>
```

* 检查httpd配置语法命令

```bash
[root@localhost ~]# httpd -t 
Syntax OK
```

* 重启

```bash
[root@localhost ~]# systemctl restart httpd
```

* 测试
  * 计算机测试记得要修改hosts文件
    * C:\Windows\System32\drivers\etc\hosts

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd3a2d5fdd27c0cc45991f574259ed3e3/68](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd3a2d5fdd27c0cc45991f574259ed3e3/68)



![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd5eb13d524093bbf36f0fddc84479ba0/64](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd5eb13d524093bbf36f0fddc84479ba0/64)

### 2、配置mp3网站

* 创建网页目录、测试目录

```bash
[root@localhost ~]# mkdir /mp3
[root@localhost ~]# vim /mp3/index.html
<h1> Welcome To MP3's Web </h1>
```

* 编辑配置文件

```bash
<VirtualHost 192.168.152.10:80>
        ServerName mp3.linux.com
        DocumentRoot    /mp3
        ErrorLog        /var/log/httpd/mp3_error.log
        CustomLog       /var/log/httpd/mp3_access.log   combined
</VirtualHost>

<Directory "/mp3">		//给非默认网页目录进行授权
        Require all granted
</Directory>

```

* 语法检查、并重启

```bash
[root@localhost ~]# httpd -t
Syntax OK
[root@localhost ~]# systemctl restart httpd
```

* 测试访问

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE96da69cebda8b0daa390689d19e1c78d/66)

## 三、基于IP的虚拟主机



网站: python.linux.com		网页目录: /python		192.168.152.128

网站: shell.linux.com		网页目录: /shell		192.168.152.129



### 1、为主机添加网卡、配置多个地址

```bash
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.10  netmask 255.255.255.0  broadcast 192.168.152.255

ens37: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.128  netmask 255.255.255.0  broadcast 192.168.152.255

ens38: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.129  netmask 255.255.255.0  broadcast 192.168.152.255
```

### 2、配置Python网站

```bash
[root@localhost ~]# mkdir /python
[root@localhost ~]# vim /python/index.html 

<h1>Welcome To Python's Web </h1>
[root@localhost ~]# vim /etc/httpd/conf.d/python.conf 

<VirtualHost 192.168.152.128:80>
        ServerName      python.linux.com
        DocumentRoot    /python
        ErrorLog        /var/log/httpd/python_error.log
        CustomLog       /var/log/httpd/python_access.log combined
</VirtualHost>

<Directory "/python">
        Require all granted
</Directory>
[root@localhost ~]# httpd -t 
Syntax OK
[root@localhost ~]# systemctl restart httpd
```

* 测试（先修改计算机hosts文件）

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE9939dc70782e7b2eb905e18e0766688e/75)

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE581cb97bffcd1ca1b861422eb9c7969e/71)

### 3、配置shell网站

```bash
[root@localhost ~]# mkdir /shell
[root@localhost ~]# vim /shell/index.html 

<h1> Welcome To Shell's Web <h1>
[root@localhost ~]# vim /etc/httpd/conf.d/shell.conf

<VirtualHost 192.168.152.129:80>
        ServerName      shell.linux.com
        DocumentRoot    /shell
        Errorlog        /var/log/httpd/shell_error.log
        CustomLog       /var/log/httpd/shell_access.log combined
</VirtualHost>

<Directory "/shell">
        Require all granted
</Directory>
[root@localhost ~]# httpd -t
Syntax OK
[root@localhost ~]# systemctl restart httpd
```

* 测试（先修改计算机hosts文件）

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE8635c3a4499b596eadea3f38f2e80459/73)