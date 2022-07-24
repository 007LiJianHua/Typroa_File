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
  * /etc/vsftpd/vsftpd.conf

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

### 8、不允许匿名用户下载

```bash
anon_world_readable_only=YES
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

### 6、删除数据目录下的文件和文件夹

> * 对数据目录具有x权限，即可删除文件
> * 对数据目录有x权限，必须也是文件夹的属主，才可删除文件夹

## 三（+）：全局配置

```bash
//上传
write_enable=YES
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

## 五、作业

目的：搭建学生上传作业FTP服务器

要求：

> 1.学生可以直接访问FTP服务器，无需登录；
>
> 2.只能上传文件，不能创建文件夹、重命名、删除等；
>
> 3.不能下载文件（防抄袭）；
>
> 4.不能在线查看文件。

```bash
配置
# 创建upload目录
mkdir /var/ftp/upload
# 改变属主
chown ftp /var/ftp/upload

第一个要求是通过anonymous_enable=YES参数实现的，即允许匿名用户登录，因为这个参数的值默认就是YES，所以可以不写入配置文件。

第二个要求是通过anon_upload_enable=YES、anon_mkdir_write_enable=NO、anon_other_write_enable=NO这三个参数实现，即允许匿名用户上传，不允许匿名用户创建目录和其他写入权限（删除和重命名），因为anon_mkdir_write_enable和anon_other_write_enable这两个参数默认值都是NO，所以也可以不写入配置文件。

第三个要求是通过anon_world_readable_only=YES、anon_umask=077这两个参数实现的，默认情况下，匿名用户所有上传下载，所使用的用户都是ftp用户的权限，若要上传文件，则需要ftp用户有写的权限，若要下载，则需要ftp用户有读的权限，即一般情况下，ftp用户对文件有读权限就对文件有下载权限了

文件有三种权限，文件所有人，文件所有组，文件的其他人，anon_world_readable_only的意思是，当他为YES时候，文件的其他人必须有读的权限才允许下载，单单文件所有人为ftp且有读权限是无法下载的，必须其他人也有读权限，才允许下载；若为NO则只要ftp用户对文件有读权限即可下载

//?不懂
第四个要求是针对txt文档而言，如果学生上传的作业不是以txt文档保存的话就不需要配置了，当然，你可以利用file_open_mode=0000来控制上传文件的权限，这样就算是txt文件，也无法在线查看了。
```

