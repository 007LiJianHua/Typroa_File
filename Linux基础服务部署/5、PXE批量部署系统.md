# PXE批量部署系统

[toc]

## 一、PXE组件介绍

### 1、共享对应的系统安装文件

*  FTP/NFS/HTTP

### 2、提供DHCP服务， 告知tftp地址

### 3、提供tftp服务

* 提供系统安装源的菜单
* 共享内核vmlinuz、初始化镜像文件 initrd.img
* pxelinux.0

### 4、syslinux

* pxelinux.0      tftp服务共享
* 提供系统引导的支持

## 二、PXE服务部署提供centos7.6安装源

> **第一步：PXE Client向DHCP发送请求**
>
> 首先，将支持PXE的网络接口卡（NIC）的客户端的BIOS设置成为网络启动，通过PXE BootROM（自启动芯片）会以UDP（简单用户数据报协议）发送一个广播请求，向网络中的DHCP服务器索取IP地址等信息。
>
> **第二步：DHCP服务器提供信息**
>
> DHCP服务器收到客户端的请求，验证是否来至合法的PXE Client的请求，验证通过它将给客户端一个“提供”响应，这个“提供”响应中包含了为客户端分配的IP地址、pxelinux启动程序（TFTP）位置，以及配置文件所在位置。
>
> **第三步：PXE客户端请求下载启动文件**
>
> 客户端收到服务器的“回应”后，会回应一个帧，以请求传送启动所需文件。这些启动文件包括：pxelinux.0、pxelinux.cfg/default、vmlinuz、initrd.img等文件。
>
> 第四步：Boot Server响应客户端请求并传送文件
>
> 当服务器收到客户端的请求后，他们之间之后将有更多的信息在客户端与服务器之间作应答, 用以决定启动参数。BootROM 由 TFTP 通讯协议从Boot Server下载启动安装程序所必须的文件（pxelinux.0、pxelinux.cfg/default）。default文件下载完成后，会根据该文件中定义的引导顺序，启动Linux安装程序的引导内核。
>
> **第五步：请求下载自动应答文件**
>
> 客户端通过pxelinux.cfg/default文件成功的引导Linux安装内核后，安装程序首先必须确定你通过什么安装介质来安装linux，如果是通过网络安装（NFS, FTP, HTTP），则会在这个时候初始化网络，并定位安装源位置。或许你会说，刚才PXE不是已经获取过IP地址了吗？为什么现在还需要一次？这是由于PXE获取的是安装用的内核以及安装程序等，而安装程序要获取的是安装系统所需的二进制包以及配置文件。由于它们需要的内容不同造成PXE模块和安装程序是相对独立的，PXE的网络配置并不能传递给安装程序。从而进行两次获取IP地址过程。
>
> 接着会读取该文件中指定的自动应答文件ks.cfg所在位置，根据该位置请求下载该文件。
>
> **第六步：客户端安装操作系统**
>
> 将ks.cfg文件下载回来后，通过该文件找到OS Server，并按照该文件的配置请求下载安装过程需要的软件包。
>
> OS Server和客户端建立连接后，将开始传输软件包，客户端将开始安装操作系统。安装完成后，将提示重新引导计算机。这个时候注意，在重新引导的过程中一定要将BIOS修改回从硬盘启动，不然的话又会重复的自动安装操作系统。
>
> 在上面介绍中PXE client是需要安装Linux的计算机，TFTP Server、DHCP Server和NFS Server运行在另外一台Linux Server上。Bootstrap文件、配置文件、Linux内核都放置在Linux Server上TFTP服务器的根目录下。而Linux根文件系统存放于NFS Server的共享目录中。
>
> PXE client在工作过程中，需要三个二进制文件：bootstrap、Linux 内核和Linux根文件系统。Bootstrap文件是可执行程序，它向用户提供简单的控制界面，并根据用户的选择，下载合适的Linux内核以及Linux根文件系统。

### 1、扩展根分区

```bash
[root@pxe ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

[root@pxe ~]# vgextend centos /dev/sdb
  Volume group "centos" successfully extended
[root@pxe ~]# 
[root@pxe ~]# lvextend -L +50G /dev/centos/root
  Size of logical volume centos/root changed from <17.51 GiB (4482 extents) to <67.51 GiB (17282 extents).
  Logical volume centos/root successfully resized.
[root@pxe ~]# 
[root@pxe ~]# xfs_growfs /dev/centos/root
```

### 2、部署ftp服务提供系统安装源

```bash
[root@pxe ~]# yum install -y vsftpd dhcp tftp-server xinetd syslinux 
[root@pxe ~]# mkdir /var/ftp/centos7.6
[root@pxe ~]# mount /dev/sr0 /mnt/
mount: /dev/sr0 is write-protected, mounting read-only
[root@pxe ~]# cp -r /mnt/* /var/ftp/centos7.6/ &
[root@pxe ~]# systemctl start vsftpd
[root@pxe ~]# systemctl enable vsftpd

[root@pxe ~]# netstat -antp | grep ftp
tcp6       0      0 :::21                   :::*                    LISTEN      7235/vsftpd   
```

### 3、部署tftp服务

* 共享内核、初始化镜像文件、菜单文件

```bash
[root@pxe ~]# cp /mnt/isolinux/* /var/lib/tftpboot/
[root@pxe ~]# mkdir /var/lib/tftpboot/centos7.6
[root@pxe ~]# mv /var/lib/tftpboot/vmlinuz /var/lib/tftpboot/initrd.img /var/lib/tftpboot/centos7.6/
[root@pxe ~]# ls /var/lib/tftpboot/
boot.cat  boot.msg  centos7.6  grub.conf  isolinux.bin  isolinux.cfg  memtest  splash.png  TRANS.TBL  vesamenu.c32
```

* 共享pxelinux.0

```bash
[root@pxe ~]# cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
```

* 编辑菜单文件

```bash
[root@pxe ~]# mkdir /var/lib/tftpboot/pxelinux.cfg
[root@pxe ~]# mv /var/lib/tftpboot/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
[root@pxe ~]# vim /var/lib/tftpboot/pxelinux.cfg/default
label centos76
  menu label Install CentOS7.6
  kernel centos7.6/vmlinuz	//以相对路径【相对于tftp的数据目录】方式指定对应菜单项的内核
  append initrd=centos7.6/initrd.img inst.stage2=ftp://192.168.140.10/centos7.6 inst.repo=ftp://192.168.140.10/centos7.6
```

---------------initrd=       初始化镜像文件，相对路几个

---------------inst.stage2=    指定系统安装文件的存储路径

----------------inst.repo=		指定软件安装包的存储位置

* 启动tftp服务

```bash
[root@pxe ~]# vim /etc/xinetd.d/tftp 

​        disable                 = no

[root@pxe ~]# systemctl restart xinetd
[root@pxe ~]# systemctl enable xinetd
[root@pxe ~]# netstat -anup | grep 69
udp        0      0 0.0.0.0:69              0.0.0.0:*       
```

### 4、配置DHCP服务

```bash
[root@pxe ~]# vim /etc/dhcp/dhcpd.conf 

subnet 192.168.140.0 netmask 255.255.255.0 {
    range 192.168.140.100 192.168.140.200;
    option routers 192.168.140.2;
    option domain-name-servers 114.114.114.114,223.5.5.5;
    next-server 192.168.140.10;
    filename "pxelinux.0";
}
[root@pxe ~]# systemctl start dhcpd
[root@pxe ~]# systemctl enable dhcpd
Created symlink from /etc/systemd/system/multi-user.target.wants/dhcpd.service to /usr/lib/systemd/system/dhcpd.service.
[root@pxe ~]# 
[root@pxe ~]# netstat -tunlp | grep dhcp
udp        0      0 0.0.0.0:67              0.0.0.0:*      
```

### 5、测试

**centos 7测试机至少2G内存**

## 三、添加centos6.6安装源

### 1、通过ftp共享centos6.6光盘文件

```bash
[root@pxe ~]# mount /dev/sr0 /mnt/
mount: /dev/sr0 is write-protected, mounting read-only
[root@pxe ~]# ls /mnt/
CentOS_BuildTag  EULA  images    Packages                  repodata              RPM-GPG-KEY-CentOS-Debug-6     RPM-GPG-KEY-CentOS-Testing-6
EFI              GPL   isolinux  RELEASE-NOTES-en-US.html  RPM-GPG-KEY-CentOS-6  RPM-GPG-KEY-CentOS-Security-6  TRANS.TBL
[root@pxe ~]# mkdir /var/ftp/centos6.6
[root@pxe ~]# cp -r /mnt/* /var/ftp/centos6.6
```

### 2、通过tftp共享centos6.6内核、初始化文件

```bash
[root@pxe ~]# mkdir /var/lib/tftpboot/centos6.6
[root@pxe ~]# 
[root@pxe ~]# cp /mnt/isolinux/vmlinuz /mnt/isolinux/initrd.img /var/lib/tftpboot/centos6.6
```

### 3、添加centos6菜单项

```bash
[root@pxe ~]# vim /var/lib/tftpboot/pxelinux.cfg/default 

label centos66
  menu label Install CentOS6.6
  kernel centos6.6/vmlinuz
  append initrd=centos6.6/initrd.img 
```

### 4、测试centos6.6

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE9ee6ecda20c837d77572e07317a51e88/49](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE9ee6ecda20c837d77572e07317a51e88/49)

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEbde153c9ec9253a3150c84bc3c056504/51](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEbde153c9ec9253a3150c84bc3c056504/51)

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE983aaf79da24e4619975daf8c00eaaa5/53](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE983aaf79da24e4619975daf8c00eaaa5/53)

下图不做任何处理直接OK

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEe49374bf7579ef71d519e65a799e4dcf/55](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEe49374bf7579ef71d519e65a799e4dcf/55)

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE19bf355fc3170d74fd0bd31213e4c65e/57](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE19bf355fc3170d74fd0bd31213e4c65e/57)

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEf25842bbd0ddd7a2f085d8a1de0e758e/59)

## 四、自动应答文件kickstart

### 1、kickstart文件介绍

* kickstart文件，自动应答文件、简称ks文件
* 文件中记录在安装系统过程中的操作（磁盘分区、root密码等）

### 2、kickstart文件格式

* 常规配置

  * 设置语言、时区
  * 设置root密码
  * 磁盘分区
  * 网卡

* 安装软件

  ​		%packages

  ​		软件名称

  ​		软件名称

  ​		@软件组名称

  ​		%end

* 系统安装完毕后自动执行的操作

  ​		%post

  ​		操作命令

  ​		操作命令

  ​		%end

### 3、kickstart文件的获取

*  anaconda-ks.cfg
* system-config-kickstart图形化工具
  * 最小化系统需要安装x11图形转发工具
  * yum -y install *x11*

### 4、配置ks文件实现centos7.6自动安装

* 通过ftp共享ks文件

```bash
[root@pxe ~]# cp centos76_ks.cfg /var/ftp/
```

* 编辑菜单文件

```bash
label centos76
  menu label Install CentOS7.6
  kernel centos7.6/vmlinuz
  append initrd=centos7.6/initrd.img inst.stage2=ftp://192.168.140.10/centos7.6 inst.repo=ftp://192.168.140.10/centos7.6 ks=ftp://192.168.140.10/centos76_ks.cfg
```

* centos 7.6 ks文件参考

```bash
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$pHhwGQlp$FnSejX6/.MBUrHVJlIUTb/
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
firstboot --disable
# SELinux configuration
selinux --disabled


# Firewall configuration
firewall --disabled
# Network information
network  --bootproto=dhcp --device=ens33
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# Use network installation
url --url="ftp://192.168.140.10/centos7.6"
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="xfs" --size=500
part swap --fstype="swap" --size=2048
part / --fstype="xfs" --grow --size=1

%packages
@core
vim-enhanced
net-tools
psmisc
lftp
wget
rsync
ntpdate
bash-completion
%end


%post
sed -ri '/^#Port/c \Port 33333' /etc/ssh/sshd_config
systemctl restart sshd
%end
```

* centos 6.6 ks文件参考

```bash
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$pHhwGQlp$FnSejX6/.MBUrHVJlIUTb/
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
firstboot --disable
# SELinux configuration
selinux --disabled


# Firewall configuration
firewall --disabled
# Network information
network  --bootproto=dhcp --device=eth0
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# Use network installation
url --url="ftp://192.168.140.10/centos6.6"
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="ext4" --size=500
part swap --fstype="swap" --size=2048
part / --fstype="ext4" --grow --size=1

%packages
@core
vim-enhanced
net-tools
psmisc
lftp
wget
rsync
ntpdate
%end


%post
sed -ri '/^#Port/c \Port 33333' /etc/ssh/sshd_config
service sshd restart
%end
```
