[toc]

## 一、企业级服务器监控的方式

* 人工巡检
  * 效率低
* 专业的监控工具
  * `zabbix`
    * IT基础设施设备（网络设备、服务器、存储）
  * promethus
    * 适用于容器化的监控

## 二、zabbix特性

* 开源的、跨平台的
* zabbix用来获取监控数据的方式
  * `zabbix-agent`
    * 适用于主机，服务器
    * 分为主动模式、被动模式
  * `SNMP协议`  161/udp
    * 分为三个版本，v1，v2，v3
    * snmp协议使用了MIB库，库中写了很多要监控的项目信息
  * `IPMI`协议
    * 适用于设备的硬件信息（温度，序列号）
    * 在人工智能机器人领域经常用到
  * `JMX`协议
    * 适用于java应用
* 支持自动监控
* 支持多种报警方式
  * 邮件、电话、微信
* 支持分布式监控
* 提供API接口
  * 适合前端二次开发

![image-20220329191434700](https://s2.loli.net/2022/03/29/TlW6MZ32KEcQrxh.png)

## 三、zabbix-server部署

### 1、关闭防火墙、SELinux、开启时间同步

### 2、配置epel仓库、zabbix仓库

* epel仓库主要是用来提供zabbix-server的依赖

```bash
[root@zabbix_server ~]# wget  http://mirrors.aliyun.com/repo/epel-7.repo -O /etc/yum.repos.d/epel.repo 
[root@zabbix_server ~]# cat /etc/yum.repos.d/zabbix.repo
[zabbix]
name=zabbix
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/4.4/rhel/7/x86_64/
enabled=1
gpgcheck=0
```

### 3、安装zabbix-server服务端软件

```bash
[root@zabbix_server ~]# yum install -y zabbix-server-mysql zabbix-web-mysql
```

### 4、安装配置MySQL

```bash
[root@zabbix_server ~]# yum install -y mariadb-server 
[root@zabbix_server ~]# systemctl start mariadb
[root@zabbix_server ~]# systemctl enable mariadb
[root@zabbix_server ~]# mysql -uroot
MariaDB [(none)]> create database zabbix charset utf8 collate utf8_bin;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on zabbix.* to 'zabbixuser'@'localhost' identified by 'redhat';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye

```

### 5、导入zabbix需要的表

* 表中的数据主要也是写一些需要的表

```bash
[root@zabbix_server zabbix-server-mysql-4.4.10]# pwd
/usr/share/doc/zabbix-server-mysql-4.4.10
[root@zabbix_server zabbix-server-mysql-4.4.10]# zcat create.sql.gz | mysql -uroot zabbix
```

### 6、编辑zabbix的配置文件，指定数据库连接信息

```bash
[root@zabbix_server ~]# vim /etc/zabbix/zabbix_server.conf 
DBHost=localhost
DBName=zabbix
DBUser=zabbixuser
DBPassword=redhat
DBSocket=/var/lib/mysql/mysql.sock

[root@zabbix_server ~]# systemctl start httpd
[root@zabbix_server ~]# systemctl start zabbix-server
[root@zabbix_server ~]# systemctl enable zabbix-server

[root@zabbix_server ~]# netstat -antp | grep zabbix
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      7544/zabbix_server  
tcp6       0      0 :::10051                :::*                    LISTEN      7544/zabbix_server 
```

### 7、初始化web界面

**访问地址:**

http://zabbix服务器/zabbix

* 将时区修改为Asia/Shanghai
* **默认用户名: Admin， 密码: zabbix**

```bash
[root@zabbix_server ~]# grep "date" /etc/httpd/conf.d/zabbix.conf
        php_value date.timezone Asia/Shanghai
[root@zabbix_server ~]# 
```

### 8、修改界面为中文

![img](https://s2.loli.net/2022/03/29/M6E4zbJQj2SICUT.png)

![img](https://s2.loli.net/2022/03/29/IdcblWZtLy83VUp.png)



## 四、配置监控本机

### 1、安装zabbix-agent

```bash
[root@zabbix_server ~]# yum install -y zabbix-agent 
```

### 2、编辑zabbix-agent的配置文件

```bash
[root@zabbix_server ~]# vim /etc/zabbix/zabbix_agentd.conf 

Server=192.168.140.10			# 被动模式下，等待哪个server要数据
ServerActive=192.168.140.10	# 主动模式下，主动向哪个server汇报数据
Hostname=Zabbix server		# 被监控端名称，惟一 
[root@zabbix_server ~]# systemctl start zabbix-agent
[root@zabbix_server ~]# systemctl enable zabbix-agent
[root@zabbix_server ~]# netstat -antp | grep zabbix
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      19071/zabbix_agentd 
```

### 3、在web界面修改本机监控接口地址

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/5F68FD5EE7434706A32E053623F7A7C9/67805D5F18804AC7A4E9AD3C98758B5C/14330](https://s2.loli.net/2022/03/29/J473NZ8jkPbwC6R.png)

![img](https://s2.loli.net/2022/03/29/CaL1yFpHxkQmjBZ.png)

### 4、解决图形乱码问题

#### 1）上传Windows的中文楷体以.ttf结尾的文件

```bash
C:\Windows\Fonts 这里边有一个  “黑体 常规” 的文件，复制到桌面，再上传到服务器的 /usr/share/zabbix/assets/fonts/ 这个目录下
[root@zabbix_server ~]# ls /usr/share/zabbix/assets/fonts/
graphfont.ttf  [root@zabbix_server ~]# ls /usr/share/zabbix/assets/fonts/
graphfont.ttf  simhei.ttf
```

#### 2）修改zabbix显示图形的配置文件

* 注意：只修改名字，不加后缀

```bash
[root@zabbix_server ~]# vim /usr/share/zabbix/include/defines.inc.php 
define('ZBX_GRAPH_FONT_NAME',           'simhei'); // font file name
define('ZBX_FONT_NAME', 'simhei');
```

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/5F68FD5EE7434706A32E053623F7A7C9/29013671FCF3462B802892788CFD5222/14346](https://s2.loli.net/2022/03/29/xup67Zr3ei5WlRm.png)

## 五、运维工作中的监控指标

### 1、CPU

* `CPU的上下文切换`        Context Switch	CS

