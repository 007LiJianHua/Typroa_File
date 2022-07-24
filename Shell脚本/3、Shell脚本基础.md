[toc]

# SHELL脚本基础

## 一、shell脚本

* 应用场景
  * 重复工作、自动执行
* 本质上就是个文本文件

## 二、输出语句

### 1、echo

* 单引号
  * 所有字符会被作为普通字符显示
* 双引号
  * **配合 -e 选项**
  * 特殊字符会被转义比如（!!会调用最后一个历史命令）

```bash
[root@localhost ~]# echo -e "SHELL\nPython\nGOlang"
SHELL
Python
GOlang

[root@localhost ~]# echo -e "Linux\tWindows\tUNIX"
Linux	Windows	UNIX

[root@localhost ~]# echo "hello!!"
echo "hellocat /etc/hosts"
hellocat /etc/hosts

[root@localhost ~]# echo 'hello!!'
hello!!

```

### 2、here document

* 便于显示多行内容

```bash
[root@localhost ~]# cat << eof
> 1. 创建虚拟机
> 2. 查看虚拟机
> 3. 销毁虚拟机
> 4. 查看系统资源
> eof
```

* 编写脚本

```bash
[root@localhost ~]# vim 1.sh 
cat << eof
1、创建虚拟机
2、查看虚拟机
3、删除虚拟机
4、恢复虚拟机
5、回到上一步
eof
[root@localhost ~]# bash 1.sh 
1、创建虚拟机
2、查看虚拟机
3、删除虚拟机
4、恢复虚拟机
5、回到上一步
```

## 三、重定向符号

### 1、类型

* 输出重定向

  ```bash
  > >> 2> &>
  ```

* 输入重定向

  ```bash
  <
  ```

### 2、三个设备文件

- /dev/stdin		标准输入设备 			键盘、鼠标 		0
- /dev/stdout	标准输出设备			显示器			1
- /dev/stderr	标准错误输出设备		显示器			2

### 3、输出重定向

* \> 覆盖
  * 仅能重定向标准输出

```bash
[root@localhost ~]# ls -ldh /etc/ > /tmp/file01

[root@localhost ~]# cat /tmp/file01 
drwxr-xr-x. 76 root root 8.0K Sep 17 09:02 /etc/

[root@localhost ~]# ls -ldh /dev/ > /tmp/file01 

[root@localhost ~]# cat /tmp/file01
drwxr-xr-x 20 root root 3.2K Sep 17 09:02 /dev/
```

* \>> 追加
  * 仅能标准重定向输出

```bash
[root@localhost ~]# ls -ldh /dev/ > /tmp/file01 

[root@localhost ~]# cat /tmp/file01
drwxr-xr-x 20 root root 3.2K Sep 17 09:02 /dev/
[root@localhost ~]# 
[root@localhost ~]# ls -ldh /etc/ >> /tmp/file01 
[root@localhost ~]# 
[root@localhost ~]# cat /tmp/file01
drwxr-xr-x 20 root root 3.2K Sep 17 09:02 /dev/
drwxr-xr-x. 76 root root 8.0K Sep 17 09:02 /etc/
```

* /dev/null  黑洞文件

```bash
[root@localhost ~]# ls -ldh /etc/ > /dev/null 
```

* 2> 
  * 仅能重定向错误信息

```bash
[root@localhost ~]# ls -ldh /dsjlfdsjlfdsjlf 2> /dev/null 

[root@localhost ~]# ls -ldh /etc/ 2> /dev/null
drwxr-xr-x. 76 root root 8.0K Sep 17 09:02 /etc/
```

* &> 
  * 重写所有信息

### 4、输入重定向

* 将文件的内容作为键盘输入使用

```bash
[root@localhost ~]# tr 'a-z' 'A-Z' < /tmp/file01
ABCDE
```

## 四、变量的使用

### 1、变量类型

* 自定义变量
* 环境变量
* 特殊变量

### 2、自定义变量

* 定义变量

```bash
变量名称=值
```

* 变量名称规范
  * 只能出现字母，数字，下划线
  * **不能出现与shell关键字冲突**
  * 见名知义
* 调用变量(**注意不要将变量名写错**)
  * $变量名称
  * ${变量名称}

```bash
[root@localhost ~]# ip_address=192.168.152.10
[root@localhost ~]# echo $ip_address
192.168.152.10
[root@localhost ~]# echo $(ip_address)

[root@localhost ~]# data=server
[root@localhost ~]# echo "There are some $datas"
There are some 
[root@localhost ~]# echo "There are some ${data}s"
There are some servers
```

#### 1、交互式定义变量

* ​                \# read -p "提示信息" 变量名称              

```bash
[root@localhost ~]# read -p "用户名: " name
用户名: userA
[root@localhost ~]# echo $name
userA

[root@localhost ~]# vim 1.sh 
read -p "用户名：" user
echo "您好：" $user
[root@localhost ~]# bash 1.sh 
用户名：lijianhua
您好： lijianhua

```

#### 2、shell中存储的所有变量都是字符

```bash
[root@localhost ~]# a=10
[root@localhost ~]# b=20

[root@localhost ~]# c=a+b
[root@localhost ~]# 
[root@localhost ~]# echo $c
a+b
[root@localhost ~]# c=$a+$b
[root@localhost ~]# echo $c
10+20
```

#### 3、数学运算

* \+ \-  \*  %(取余) 
* $(())

```bash
[root@localhost ~]# a=10
[root@localhost ~]# b=20
[root@localhost ~]# echo $((a+b))
30
```

* let 关键字

```bash
[root@localhost ~]# let c=a+b
[root@localhost ~]# echo $c
30
```

* declare

```bash
[root@localhost ~]# declare -i c=a+b
[root@localhost ~]# echo $c
30
```

#### 4、获取100以内随机数

```bash
[root@localhost ~]# let rand=$RANDOM%100
[root@localhost ~]# echo $rand
43
```

### 3、环境变量

* 保存系统当前操作环境

#### 1、查看环境变量

```bash
env
```

#### 2、定义、修改环境变量

* 环境变量名称必须大写

```bash
临时生效：export 环境变量=值
永久生效：vim /etc/profile
		export 环境变量=值
```

示例一：定义历史命令的时间

```bash
export HISTTIMEFORMAT="%F_%T  "
```

示例二：定义当前用户历史命令行数，默认1000

```bash
export HISTSIZE=10
```

示例三：修改系统语言

```bash
export LANG=en_us.UTF-8
```

### 4、特殊变量

* $?
  * 上一条命令的执行状态码
  * 取值范围 0---255
    * 0   成功
    * 非0  失败

## 五、shell脚本示例

### 1、配置本地yum源

```bash
#!/bin/bash
#
mkdir /mnt/cdrom
sed -i '$a \/dev/sr0 /mnt/cdrom iso9660 defaults 0 0' /etc/fstab
mount -a
mkdir /etc/yum.repos.d/backup
mv /etc/yum.repos.d/CentOS* /etc/yum.repos.d/backup/
touch /etc/yum.repos.d/yum.repo
cat << eof > /etc/yum.repos.d/yum.repo
[centos]
name=centos
baseurl=file:///mnt/cdrom
enabled=1
gpgcheck=0
eof
yum repolist
```

### 2、配置网络参数

```bash
#!/bin/bash
#

read -p "手动配置IP地址，请输入：" ip
nmcli connection modify ens160 autoconnect yes ipv4.method manual ipv4.addresses $ip ipv4.gateway 192.168.152.2 ipv4.dns 223.5.5.5
nmcli connection reload
nmcli connection up ens160 >> /dev/null
IP=$(ifconfig ens160 | sed -n '2p' | awk '{print $2}')
ROUTE=$(route -n | sed -n '3p' | awk '{print $2}')
DNS=$(cat /etc/resolv.conf|sed -n '2p' | awk '{print $2}')
echo "=============================="
echo -e "更改后的IP：$IP\n更改后的网关为：$ROUTE\n更改后的DNS为：$DNS"

```

