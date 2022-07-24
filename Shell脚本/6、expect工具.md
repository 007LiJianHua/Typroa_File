[toc]

# expect工具

## 一、expect工具

* 作用
  * 让脚本中的交互式命令自动执行

### 1、安装expect工具

```bash
[root@localhost ~]# yum -y install expect
```

### 2、expect独立使用

```bash
#!/usr/bin/expect
#

set timeout 10

spawn passwd martin
expect "New password:"
send "www.1.com\n"
expect "Retype new password:"
send "www.1.com\n"
expect eof
```

**指令说明：**

- set timeout 10		

- - 设置超时时间, 单位秒

- spawn

- - 指定交互式命令

- expect

- - 指定捕获的提示信息

- send

- - 发送指令

- expect eof

- - 捕获结束

### 3、结合shell自动输入密码

```bash
#!/bin/bash
#

useradd AAA

/usr/bin/expect << eof
set timeout 10
spawn passwd AAA
expect "New password:"
send "123\n"
expect "Retype new password:"
send "123\n"
expect eof
eof
```

### 4、配置免密ssh

```bash
#!/bin/bash
#

if [ -e ~/.ssh/known_hosts ];then
	> ~/.ssh/known_hosts
fi
if [ -e ~/.ssh/id_rsa ]; then
	rm -rf ~/.ssh/id_rsa*
fi
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa &> /dev/null

for i in 11 12; do
1
done
ssh root@192.168.152.11 ifconfig ens33
ssh root@192.168.152.12 ifconfig ens33
```

