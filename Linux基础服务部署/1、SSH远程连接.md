# SSH远程连接

[toc]

## 一、SSH介绍

### 1、SSH介绍

* SSH是一个应用层协议
  * 实现Linux的远程加密连接
  * 适用于Linux服务器远程管理
* Telnet
  * 远程连接、不加密
  * 适用于局域网网络设备的远程连接

### 2、管理Linux服务器的方式

* 本地管理
  * 安装系统、故障修复
* 远程管理（使用频率较高）

### 3、确保Linux服务器正常启动SSH服务

* 确保SSH服务正常运行
* SSH服务端口22/tcp

```bash
[root@localhost ~]# ps aux | grep ssh
root        937  0.0  0.4 112900  4324 ?        Ss   17:56   0:00 /usr/sbin/sshd -D
root       1196  0.0  0.6 161512  6084 ?        Ss   17:57   0:00sshd: root@pts/0
root       1237  0.0  0.0 112812   964 pts/0    S+   18:20   0:00 grep --color=auto ssh
[root@localhost ~]# ps -elf |grep ssh
4 S root        937      1  0  80   0 - 28225 poll_s 17:56 ?        00:00:00 /usr/sbin/sshd -D
4 S root       1196    937  0  80   0 - 40378 poll_s 17:57 ?        00:00:00 sshd: root@pts/0
0 S root       1240   1198  0  80   0 - 28202 pipe_w 18:20 pts/0    00:00:00 grep --color=auto ssh

```

* 对应服务端软件

```bash
[root@localhost ~]# rpm -qf /usr/sbin/sshd
openssh-server-7.4p1-21.el7.x86_64
```

### 4、客户端软件

* Windows
  * XShell
  * SecureCRT
  * Putty
* Linux
  * ssh命令

```bash
[root@localhost ~]# which ssh
/usr/bin/ssh
[root@localhost ~]# rpm -qf /usr/bin/ssh
openssh-clients-7.4p1-21.el7.x86_64

```

## 二、ssh远程连接操作

### 1、ssh远程连接

```bash
// ssh 用户名@服务器地址
[root@client ~]# ssh root@192.168.152.11
The authenticity of host '192.168.152.11 (192.168.152.11)' can't be established.
ECDSA key fingerprint is SHA256:bxSRt15R3GMzRu2+B9CU/dOHgaH/bQNlWmrSpWHNed4.
ECDSA key fingerprint is MD5:58:73:3c:d6:06:c2:58:a1:fc:7d:b9:c7:1d:bd:97:e7.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.152.11' (ECDSA) to the list of known hosts.
root@192.168.152.11's password: 
Permission denied, please try again.
root@192.168.152.11's password: 
Last login: Thu Jan  6 18:31:25 2022 from 192.168.152.1

```

### 2、执行远程命令

```bash
[root@client ~]# ssh root@192.168.152.11 uptime
root@192.168.152.11's password: 
Permission denied, please try again.
root@192.168.152.11's password: 
 18:33:07 up 2 min,  1 user,  load average: 0.13, 0.19, 0.09
[root@client ~]# ssh root@192.168.152.11 ifconfig 
root@192.168.152.11's password: 
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.11  netmask 255.255.255.0  broadcast 192.168.152.255
        inet6 fe80::2332:e633:8b27:d20a  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:b9:db:e8  txqueuelen 1000  (Ethernet)
        RX packets 392  bytes 45076 (44.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 272  bytes 41766 (40.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 3、远程拷贝文件

* scp命令使用
  * 参数
    * -r：拷贝目录
    * -P：指定端口

#scp 源文件 用户名@服务器地址：目录 

```bash
[root@client ~]# vim /opt/file10.txt
[root@client ~]# scp /opt/file10.txt root@192.168.152.11:/tmp
root@192.168.152.11's password: 
file10.txt                      100%    4     1.9KB/s   00:00
[root@server ~]# ls /tmp/
file10.txt  ks-script-mP33Qu  yum.log

//以tank用户登录并将服务器文件夹blog及以下的所有文件夹copy到本地
[root@localhost www]# scp -r tank@192.16.1.108:/var/www/blog /home/www/blog  

//将本地文件copy到服务器，添填写用户，默认当前用户
[root@localhost www]# scp /home/www/blog/index.php 192.16.1.108:/var/www/blog  
```

* rsync
  * 前提
    * **两方之间必须都有rsync软件**
  * 参数
    * -a：保留文件权限
    * -v：显示拷贝的详情
  * **增量数据**
    * 只会拷贝变化的数据，

```bash
[root@client ~]# rsync -av /opt/python/ root@192.168.152.11:/tmp/
root@192.168.152.11's password: 
sending incremental file list
./
1.py
10.py
2.py
3.py
4.py
5.py
6.py
7.py
8.py
9.py

sent 581 bytes  received 209 bytes  316.00 bytes/sec
total size is 0  speedup is 0.00
[root@server ~]# ls /tmp/
10.py  2.py  4.py  6.py  8.py  file10.txt        yum.log
1.py   3.py  5.py  7.py  9.py  ks-script-mP33Qu

```

## 三、ssh怎么实现加密

### 1、数据加密

* 利用数学上的算法+**密钥**实现数据加密

  * 非对称加密算法

  * ssh首次连接主机时，主机会自动发送自己的密钥，询问是否接受 

  * ssh密钥存储位置：

  * - ~/.ssh/known_hosts 

### 2、数据加密类型

* 对称加密算法
  * 数据加密、解密使用相同的密钥

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/7AD5533238E241C5885D1C7CB88975B5/25334)

* 非对称加密算法
  * 数据加密、解密使用不同的密钥
    * 基于密钥对（公钥，私钥）
      * 两个都是数学算法生成的随机数
    * **公钥加密、私钥解密**
      * 一个公钥只能对应一个私钥

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/C4323AE933A3413C8E37C10ED325061D/25347)

## 四、免密SSH

### 1、ssh认证方式

* 注意事项
  * 免密ssh是基于用户的
  * 免密ssh是单向的
* 基于用户名，密码认证【默认】
* 基于密钥的认证
  * 免密的ssh

### 2、基于密钥的认证流程

* 在客户端生成密钥对
* 将公钥拷贝到服务器

### 3、配置免密的ssh

* 在客户端生成密钥对

```bash
[root@client ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:YEk+fF0Ii+R6ZBK6GOeZ4zbhIBXCwtydKeNjwOp3TVA root@client
The key's randomart image is:
+---[RSA 2048]----+
|* o o *E.. ..    |
|.B * % o o..     |
|+ * + & o .      |
|.* B * =         |
|= O o + S        |
|.= + o .         |
|  * .            |
| . .             |
|                 |
+----[SHA256]-----+
```

* 将公钥拷贝到服务器

```bash
[root@client ~]# ssh-copy-id root@192.168.152.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.152.11's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.152.11'"
and check to make sure that only the key(s) you wanted were added.

```

* 也是自动执行了将公钥拷贝到对方服务器用户家目录的这个操作

```bash
/root/.ssh/authorized_keys
```

* 验证

```bash[
[root@client ~]# ssh root@192.168.152.11 ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.11  netmask 255.255.255.0  broadcast 192.168.152.255
        inet6 fe80::2332:e633:8b27:d20a  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:b9:db:e8  txqueuelen 1000  (Ethernet)
        RX packets 4538  bytes 344665 (336.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 914  bytes 111606 (108.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 4、实现windows和linux之间免密登录

* 在windows生成密钥对

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/3B6E6DE95CEC442F844602004A8914C2/25383](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/3B6E6DE95CEC442F844602004A8914C2/25383)

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/EF302CC971134D6C95583A90DA8B3951/25385](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/EF302CC971134D6C95583A90DA8B3951/25385)

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/67384B59F89448248118FDE5359AADF9/25387)

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/A25A099ADB744BC087C2FFECAE5138AB/25391](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/A25A099ADB744BC087C2FFECAE5138AB/25391)

* 将公钥拷贝到服务器

```bash
[root@localhost ~]# rz
[root@localhost ~]# cat id_rsa_2048.pub >> /root/.ssh/authorized_keys 

[root@localhost ~]# cat /root/.ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtQk+Kx4bafhFsSwZC09RQXT2UIux6KDOeXmzYxN+oZFrXqSIGHHforssyiEUqV9dfETmv6kYencgSsU37ecSpn/3QUVuTU6bu26zF4sSF/w+qYuylJ1TuDg5HsqpRWbrJ5Q8eGLg9lwozlCQ0lswLnxmAM1B66SJuY8Lsm1W9f5XBA/RweNiVJwFHoqLHTVwnn0/Gt/LdfL/LnqH+TxbSjnf9mRdBVtogsAITDCBqpzRDCW/XaBkmKOe9mXC/8rZ6saZdLyZKQXefl52Wdh9mYbRVvO32YTnTdLk1mQ3fwOotBlVIPMMrx2HLRIhQXP1L7q9U4f39uPIqYcT3rUvj root@node01.linux.com
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEArrxv/vWwlcsBBNkYCQ0K07nqqDDbbd27eFRIrT+uX0SQ+i1dIXcBgNDf0Zt3nJw9n8bAnksiKxxC6OzRaCMIm/XDr8V75gd3DybeBXO7IVybA/OAr6LVRY5ehAI9OyqMPe6yUxRyekk6uV3KOattMeD2rya0JBTXnjgZjRqc4pApxdh4pAwZGGtaS447TLogLrdnfxAvcEp7dyXl5li/wPRRB2dHZEjyULErWl+R7+Li0nlz2TO790dqHYBcnqdmwMO5iCjIL5trVRqPAgf1HJ7N7/M298h8nCgYR0w80T/E7d/M9/H5x63UYGF/yck891G9tgtqkJsIgukEOXGudQ==[root@localhost ~]# 
```

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/E40427E5104045209196D504813665E1/25397](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/50F435318AAA4F288A34D951A9965767/E40427E5104045209196D504813665E1/25397)

## 五、ssh服务配置

### 1、配置文件

* /etc/ssh/sshd_config

```bash
[root@server ~]# vim /etc/ssh/sshd_config 
```

* 重新启动sshd服务

### 2、常用配置项

* 修改ssh服务端口

```bash
Port 44444
[root@localhost ~]# systemctl restart sshd
```

* 禁用密码认证

```bash
PasswordAuthentication no
```

* 禁止Root用户远程登录

```bash
PermitRootLogin no
```

* 禁用DNS反解，加快ssh连接速度

```bash
UseDNS no 
```

