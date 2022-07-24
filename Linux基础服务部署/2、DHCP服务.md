# DHCP服务

[toc]

## 一、DHCP介绍

### 1、DHCP介绍

* DHCP应用层协议
* 动态主机配置协议
* 作用
  * 为网络中的主机自动分配IP信息

## 二、配置DHCP服务

### 1、关闭防火墙、SElinux、时间同步

略

### 2、安装DHCP软件

```bash
[root@node01 ~]# yum install dhcp 

[root@node01 ~]# rpm -q dhcp
dhcp-4.2.5-83.el7.centos.1.x86_64
```

### 3、复制dhcp配置文件

```bash
[root@node01 ~]# cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf 
cp: overwrite ‘/etc/dhcp/dhcpd.conf’? yes
```

### 4、编辑配置文件

```bash
[root@node01 ~]# vim /etc/dhcp/dhcpd.conf 

subnet 192.168.140.0 netmask 255.255.255.0 {
  range 192.168.140.100 192.168.140.188;
  option routers 192.168.140.2;
  option domain-name-servers 114.114.114.114, 223.5.5.5;
}
```

### 5、启动dhcp服务

```bash
[root@node01 ~]# systemctl start dhcpd
[root@node01 ~]# systemctl enable dhcpd
Created symlink from /etc/systemd/system/multi-user.target.wants/dhcpd.service to /usr/lib/systemd/system/dhcpd.service.
[root@node01 ~]# ps -elf | grep dhcp
4 S dhcpd     17620      1  0  80   0 - 26591 poll_s 15:53 ?        00:00:00 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
0 S root      17642   6927  0  80   0 - 28177 pipe_w 15:55 pts/0    00:00:00 grep --color=auto dhcp
[root@node01 ~]# netstat -tunlp | grep dhcp
udp        0      0 0.0.0.0:67              0.0.0.0:*                           17620/dhcpd         
[root@node01 ~]# 
```

## 三、验证dhcp服务

### 1、关闭vmnet虚拟网络自带的 DHCP

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/BECFCCFC964B458DB6C3AC4F78E56BF8/832303EF715846E58D5FD2A78DCAA39D/25448](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/BECFCCFC964B458DB6C3AC4F78E56BF8/832303EF715846E58D5FD2A78DCAA39D/25448)

### 2、将主机修改为自动获取IP，验证

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/BECFCCFC964B458DB6C3AC4F78E56BF8/AC4CD2F758984D8FB66177E2CBCF7CB2/25452](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/BECFCCFC964B458DB6C3AC4F78E56BF8/AC4CD2F758984D8FB66177E2CBCF7CB2/25452)

## 四、DHCP工作原理

### 1、DHCP Discovery	发现

- 检测网络中的dhcp服务器

### 2、DHCP Offer	提供

- 包含dhcp服务器准备分配IP信息

### 3、DHCP Request	请求

- 询问是否可配置IP 

### 4、DHCP ACK

* 服务器确认
