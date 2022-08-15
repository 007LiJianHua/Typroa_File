# FTP服务

[toc]

## 一、FTP介绍

### 1、介绍

* FTP应用层协议
* 文件传输协议
* 作用
  * 共享文件

### 2、FTP服务端

* 软件
  * vsftpd
* 配置文件
  * /etc/vsftpd/vdftpd.conf

### 3、FTP数据目录

* 存放共享文件的位置
* 位置可自定义
* 默认位置
  * /var/ftp

```bash
[root@localhost ~]# yum -y install vsftpd
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
[root@localhost ~]# ls /var/ftp/
pub
```

### 4、FTP的访问模式

* 匿名认证
  * ftp://192.168.152.10  
* 本地用户认证

## 二、匿名认证相关配置

### 1、配置文件

* /etc/vsftpd/vsftp.conf

### 2、启用匿名认证

```bash
anonymous_enable=YES
```

### 3、允许上传文件

```bash
anon_upload_enabled=YES
chmod o+w /var/ftp/pub
```

* 不允许直接在数据目录下上传文件，需要实现规划子目录
  * **不能直接给数据目录给w权限，否则会拒绝匿名登录**
* 确保服务器本地匿名用户（FTP用户）对目录拥有读写权限
  * 上传的文件属性为600：没有执行权限
  * 上传的目录属性为700：

### 4、允许上传目录

```bash
anon_mkdir_write_enable=YES
```

### 5、允许其他修改操作

* 删除、改名字

```bash
anon_other_write_enable=YES
```

### 6、允许匿名用户可正常下载上传的文件

```bash
anon_umask=022
```

### 7、修改匿名用户的数据目录

```bash
anon_root=/data
```

## 三、本地用户认证配置

### 1、本地用户默认目录

* 对应用户的家目录

### 2、启用本地用户认证

```bash
local_enable=YES
```

### 3、修改数据目录

```bash
local_root=数据目录
```

### 4、本地用户想要上传文件或者目录

> * 要么把本地用户设置为数据目录子目录的所有者
> * 要么创建一个组，将本地用户全部加入组中，再将所属组修改为对应的组+写权限即可

### 5、将本地用户锁在家目录的办法

```bash
chroot_local_user=YES
chroot_list_enable=YES
```



## 四、FTP客户端

### 1、windows客户端

* filezilla

### 2、Linux客户端

```bash
[root@localhost ~]# lftp 192.168.140.10 
lftp 192.168.140.10:~> ls              
-rw-r--r--    1 0        0               0 Jan 07 02:11 1.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 10.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 2.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 3.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 4.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 5.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 6.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 7.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 8.mp3
-rw-r--r--    1 0        0               0 Jan 07 02:11 9.mp3
drwxr-xrwx    2 0        0              32 Jan 07 02:12 AA

lftp 192.168.140.10:/> get 10.mp3 				// 下载单个文件
lftp 192.168.140.10:/> put /etc/fstab			//上传文件
lftp 192.168.140.10:/> mget 2.mp3 3.mp3 4.mp3 5.mp3 		//下载多个文件
Total 4 files transferred

lftp 192.168.140.10:/> mirror AA			//下载目录
Total: 1 directory, 2 files, 0 symlinks                   
New: 2 files, 0 symlinks
lftp 192.168.140.10:/> 
lftp 192.168.140.10:/> exit
```

