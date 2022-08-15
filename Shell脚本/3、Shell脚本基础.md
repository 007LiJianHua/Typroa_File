[toc]

# SHELL脚本基础

## 一、shell脚本

* 应用场景
  * 重复工作、自动执行
* 本质上就是个文本文件



## 一+、数组与JSON

### 1、数组的定义与调用

```bash
local message=($@)	##定义局部数组，将所有的参数列表放到数组message中
echo ${message[@]}	##输出数组message的所有值
```

### 2、JSON格式

>  jq 用于处理JSON输入，将给定过滤器应用于其JSON文本输入并在标准输出上将过滤器的结果生成为JSON。





## 二、输出语句

### 1、echo

* 单引号
  * 所有字符会被作为普通字符显示
* 双引号
  * **配合 -e 选项**
  * 特殊字符会被转义比如（!!会调用最会一个历史命令）

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

### 3、eval用法

> 当我们在命令行前加上eval时，shell就会在执行命令之前扫描它两次.eval命令将首先会先扫描命令行进行所有的置换，然后再执行该命令。该命令适用于那些一次扫描无法实现其功能的变量。该命令对变量进行两次扫描。

* eval可以用来回显简单的变量

  ![img](https://img2022.cnblogs.com/blog/1616576/202206/1616576-20220627134442096-1011754387.png)

* 也可以用来执行含有字符串的命令

  ![img](https://img2022.cnblogs.com/blog/1616576/202206/1616576-20220627134458582-652864432.png)



### 4、xargs

* 单独使用时，将多行输入处理为单行输出

```bash
[root@test-10 opt]# cat test.json 
[
  {
    "lon": 113.30765,
    "name": "广州市",
    "code": "4401",
    "lat": 23.422825
  },
  {
    "lon": 113.59446,
    "name": "韶关市",
    "code": "4402",
    "lat": 24.80296
  }
]
[root@test-10 opt]# cat test.json | xargs
[ { lon: 113.30765, name: 广州市, code: 4401, lat: 23.422825 }, { lon: 113.59446, name: 韶关市, code: 4402, lat: 24.80296 } ]
```

* 配合使用
  * xargs命令可以通过管道接受字符串，并将接收到的字符串通过空格分割成许多参数(默认情况下是通过空格分割) 然后将参数传递给其后面的命令，作为后面命令的命令行参数

```bash
[root@test-10 opt]# systemctl start httpd
[root@test-10 opt]# ps -elf | grep httpd
4 S root      80543      1  0  80   0 - 57610 poll_s 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
5 S apache    80544  80543  0  80   0 - 57610 inet_c 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
5 S apache    80545  80543  0  80   0 - 57610 inet_c 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
5 S apache    80546  80543  0  80   0 - 57610 inet_c 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
5 S apache    80547  80543  0  80   0 - 57610 inet_c 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
5 S apache    80548  80543  0  80   0 - 57610 inet_c 18:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
0 S root      80550  80229  0  80   0 - 28202 pipe_w 18:04 pts/0    00:00:00 grep --color=auto httpd
[root@test-10 opt]# ps -elf | grep httpd | xargs kill
kill: cannot find process "S"
kill: cannot find process "root"
Terminated
[root@test-10 opt]# ps -elf | grep httpd
0 S root      80575  80229  0  80   0 - 28202 pipe_w 18:04 pts/0    00:00:00 grep --color=auto httpd
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

 read -p "提示信息" 变量名称              

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

> * local
>   * 定义局部变量
> * date -d ""
>   * 显示字符串指定的时间，而不是当前的时间
> * $后可以加的字符
>   * $?
>     * 上一条命令的执行状态码
>     * 取值范围 0---255
>       * 0   成功
>       * 非0  失败
>   * \$$
>     * Shell本身的PID
>   * $!
>     * Shell最后运行的后台Process的PID
>   * $@
>     * 所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。获取传递给脚本或函数的所有参数
>   * $?
>     * 最后运行的命令的结束代码（返回值）
>   * $*
>     * 所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。获取传递给脚本或函数的所有参数
>   * $0
>     * shell脚本的文件名字
>   * $#
>     * 添加到shell的参数个数

补充：$@与$*区别？

> * 正常情况下没有什么区别，只有当$*  被双引号包围时，会有区别
>
>   * $@没有什么区别
>
>     ```bash
>     [root@localhost code]# cat test
>         for var in "$@"
>         do
>             echo ${var}
>         done
>      
>     [root@localhost code]# ./test a b c d	#这里输出了4次
>     a
>     b
>     c
>     d
>     [root@localhost code]#
>     ```
>
>   * 当$*被包围时，会将脚本的输入当成一个整体输出
>
>     ```bash
>     [root@localhost code]# cat test1
>         for var in "$*"
>         do
>             echo ${var}
>         done
>     [root@localhost code]#
>     [root@localhost code]# ./test1 a b c d		#也就是说只输出了一次
>     a b c d
>     [root@localhost code]#
>     ```
>
>     



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

