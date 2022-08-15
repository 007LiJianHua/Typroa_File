# cobbler批量部署系统

[toc]

## 一、cobbler介绍

* 由RedHat公司基于python语言开发
* 作用：快速批量部署Linux系统

### 1、cobbler特性：

* 部署简单
* 默认使用http共享系统安装文件
* 支持系统定制
* 提供API接口
* 提供web管理界面

## 二、cobbler服务安装部署

### 1、关闭防火墙、SELinux、确保正常访问互联网

```bash
[root@localhost ~]# systemctl status firewalld.service 
● firewalld.service
   Loaded: masked (/dev/null; bad)
   Active: inactive (dead)
[root@localhost ~]# getenforce 
Disabled
[root@localhost ~]# ping baidu.com
PING baidu.com (220.181.38.251) 56(84) bytes of data.
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=1 ttl=128 time=10.2 ms
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=2 ttl=128 time=9.43 ms
```

### 2、扩展centos逻辑卷、导入centos7.6镜像

```bash
//文件较大，可放入后台运行
[root@localhost ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@localhost ~]# vgcreate centos /dev/sdn
  /dev/centos: already exists in filesystem
  Run `vgcreate --help' for more information.
[root@localhost ~]# 
[root@localhost ~]# vgcreate centos /dev/sdb
  /dev/centos: already exists in filesystem
  Run `vgcreate --help' for more information.
[root@localhost ~]# vgextend centos /dev/sdb
  Volume group "centos" successfully extended
[root@localhost ~]# lvextend -L +45G /dev/centos/root 
  Size of logical volume centos/root changed from <17.51 GiB (4482 extents) to <62.51 GiB (16002 extents).
  Logical volume centos/root successfully resized.
[root@localhost ~]# xfs_growfs /dev/centos/root 
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1147392 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4589568, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4589568 to 16386048
[root@cobbler ~]# mount /dev/sr0 /mnt/
[root@cobbler ~]# cobbler import --path=/mnt --name=CentOS74 --arch=x86_64 
```



### 3、安装cobbler相关软件

> * 本地源中没有cobbler软件，需要在阿里云开源镜像站下载一个常用的epel.repo
> * 每次安装完软件都要**启动并配置开机自启**
> * 最后安装cobbler相关软件

```bash
[root@cobbler ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@cobbler ~]# yum install -y cobbler cobbler-web dhcp tftp-server xinetd httpd 
[root@cobbler ~]# systemctl start cobblerd.service httpd 
[root@cobbler ~]# systemctl enable cobblerd.service httpd 
```

### 4、配置cobbler参数

```bash
[root@cobbler ~]# cobbler check
The following are potential configuration items that you may want to fix:

//在cobbler设置文件中必须指定kickstarting功能的服务器地址
1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.

//'next_server'字段要与与PXE网络上引导服务器的IP匹配
2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.

//取消禁用tftp
3 : change 'disable' to 'no' in /etc/xinetd.d/tftp

//  在/var/lib/cobbler/loaders 中存放网络引导加载程序，运行命令cobbler get-loaders，如果失败手动下载存放
# yum install -y syslinux
# cp /usr/share/syslinux/pxelinux.0 /var/lib/cobbler/loaders/
# cp /usr/share/syslinux/menu.c32 /var/lib/cobbler/loaders
4 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.

//启动服务
5 : enable and start rsyncd.service with systemctl

//可跳过安装debmirror的过程
6 : debmirror package is not installed, it will be required to manage debian deployments and repositories

//安装pykickstart
7 : ksvalidator was not found, install pykickstart

//设置自动部署的服务器的密码，医用命令openssl passwd -1 -salt 'random-phrase-here' 'your-password-here' 来生成随机字符+密码的组合，在/etc/cobbler/setting 中找到default_passwd_crtpted并修改
8 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one

//centos6以下安装cman、centos7以上安装fence-agents
9 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

//运行cobbler sync应用cobbler的更改
Restart cobblerd and then run 'cobbler sync' to apply changes.
```

### 5、配置DHCP服务

```bash
//在cobbler设置文件中开启DHCP服务
[root@cobbler ~]# vim /etc/cobbler/settings 
manage_dhcp: 1

//编辑cobbler的dhcp模板文件，写完conbbler会自动复制到DHCP的配置文件下
[root@cobbler ~]# vim /etc/cobbler/dhcp.template 
subnet 192.168.183.0 netmask 255.255.255.0 {
     option routers             192.168.183.2;
     option domain-name-servers 114.114.114.114,202.106.0.20;
     range dynamic-bootp        192.168.183.100 192.168.183.200;
     default-lease-time         21600;
     max-lease-time             43200;
     next-server                $next_server;
     
//这个时候的DHCP服务是由DHCP来控制，重启cobbler即重启DHCP     
[root@cobbler ~]# systemctl restart cobblerd
[root@cobbler ~]# cobbler sync 
```

### 6、启动TFTP服务

```bash
[root@cobbler ~]# systemctl restart xinetd
[root@cobbler ~]# systemctl enable xinetd
```

### 7、测试

```bash
//在服务器上如果有这个目录则配置正确
[root@cobbler ~]# cobbler profile list
CentOS74-x86_64

//测试机2G内存，新建测试
```

### 8、再次添加centos66的安装源

* 替换centos 6的安装光盘

```bash
[root@cobbler ~]# mount /dev/sr0 /mnt/
[root@cobbler ~]# cobbler import --path=/mnt --name=CentOS66 --arch=x86_64
```



## 三、系统绑定

### 1、添加系统绑定名字、要绑定的系统、网卡接口、MAC地址

````bash
[root@cobbler ~]# cobbler system add --name=vm01_centos7 --profile=CentOS74-x86_64 --mac-address=00:50:56:21:AD:B1 --interface=eth0
````

### 2、查看绑定的系统名单

```bash
[root@cobbler ~]# cobbler system list
   vm01_centos7
```