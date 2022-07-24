[toc]

## 一、ansible介绍

* 基于python语言开发，自动化运维工具
* 实现IT基础设施设备进行批量管控

### 1、ansible特性

* 开源的、跨平台的
* 底层基于ssh协议通信的
* no server
  * 就是没有服务器
* no agent
  * 对应的也没有客户端
* 支持playbook剧本
* 提供API接口

## 二、ansible安装部署

* 环境说明

```bash
192.168.140.10	ansible管理机
192.168.140.11	服务器（免密）
192.168.140.12	服务器（免密）
192.168.140.13	服务器（无）
```



### 1、配置SSH免密

* 做的不是机器的互相之间的免密，只是做了ansible与其他机器之间的免密

```bash
[root@zabbix_server ~]# ssh-keygen -t rsa
[root@zabbix_server ~]# ssh-copy-id root@192.168.140.11
[root@zabbix_server ~]# ssh-copy-id root@192.168.140.12
```

### 2、在管理机上安装ansible

```bash
[root@zabbix_server ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-
[root@zabbix_server ~]# yum install -y ansible 
```

### 3、主机清单文件inventory文件----/etc/ansible/hosts

#### 1）单个主机

```bash
192.168.152.11
192.168.152.12
```

* 测试

**<font color="red">在测试之前，要先与服务器登录一次，记录下服务器的公钥，否则会报错</font>**

```bash
[root@zabbix_server ~]# ansible all -m shell -a 'free -m'
```

#### 2）定义主机组

```bash
[appserver]
192.168.140.11
192.168.140.12
```

```bash
[appserver]
192.168.140.11
192.168.140.12

[dbserver]
192.168.140.11
```

```bash
[test:children]
appserver
dbserver
```

* 测试

```bash
[root@zabbix_server ~]# ansible appserver -m shell -a 'free -m'
```

### 4、添加非免密服务器

```bash
[appserver]
192.168.140.11
192.168.140.12
192.168.140.13 ansible_ssh_user="root" ansible_ssh_pass="redhat" ansible_ssh_port=22
```

* 测试

```bash
[root@zabbix-server ~]# ansible webserver -m ping
#无明显红色报错即可
```

### 5、查看清单文件中的主机

```bash
[root@zabbix_server ~]# ansible appserver --list-hosts
  hosts (3):
    192.168.140.11
    192.168.140.12
    192.168.140.13
```

**ansible管理主机，默认会在本地的known_hosts文件中检查是否存在对方主机的密钥，可修改ansible.cfg配置文件，禁用行为**

```bash
[root@zabbix_server ~]# vim /etc/ansible/ansible.cfg 
host_key_checking = False
```

## 三、ansible AD HOC

* 调用模块

### 1、ansible命令使用格式

```bash
# ansible <被管理机> -m <模块名称> [-a <模块参数>]
```

**被管理机的写法：**

- all	所有主机 

- - [root@zabbix_server ~]# ansible all -m ping

- 单个主机 

- - [root@zabbix_server ~]# ansible 192.168.140.11 -m ping

- 主机组名 

- - [root@zabbix_server ~]# ansible dbserver -m ping

- 多个主机组 

- - [root@zabbix_server ~]# ansible 'appserver:!dbserver' -m ping 	

  - - 差集

  - [root@zabbix_server ~]# ansible 'appserver:dbserver' -m ping 

  - - 并集

  - [root@zabbix_server ~]# ansible 'appserver:&dbserver' -m ping 

  - - 交集

### 2、查看模块帮助

#### 1）查看模块名字

```bash
[root@zabbix_server ~]# ansible-doc -l
```

#### 2）查看模块使用

```bash
[root@zabbix_server ~]# ansible-doc service 
```

## 四、ansible常用模块

### 1、ping模块

* 也常用来测试通信

```bash
[root@zabbix_server ~]# ansible appserver -m ping
```

### 2、shell模块

* 在被管理机上统一执行shell命令
* 参数
  * chdir=目录名称

```bash
[root@zabbix_server ~]# ansible appserver -m shell -a 'uptime'
```

```bash
[root@zabbix_server ~]# ansible appserver -m shell -a 'chdir=/tmp ls'
```

### 3、copy模块

* 推送文件
* 常用参数

- - src=			源文件
  - dest=		目的文件
  - mode=		权限
  - owner=		属主
  - group=		属组

```bash
[root@ansible ~]# ansible appserver -m copy -a 'src=/opt/file01.txt dest=/tmp'
```

```bash
[root@ansible ~]# ansible appserver -m copy -a 'src=/opt/file02.txt dest=/tmp mode=0600 owner=nobody group=nobody'
```

### 4、fetch模块

* 从被管理机上拉取文件
* 常用参数

- - src=			源文件
  - dest=		目的文件

```bash
[root@ansible ~]# ansible appserver -m fetch -a 'src=/etc/hosts dest=/tmp'
```

### 5、file模块

* 管理服务器文件目录
* 常用参数

- - path= 	指定文件、目录名称
  - owner= 	指明文件的属主
  - group= 	指明文件的属组
  - mode= 	指明文件的权限
  - state=directory|link|absent|touch 		表示创建的文件是目录还是软链接
  - src= 		指定软链接的原文件，配合state=link时有用

```bash
[root@ansible ~]# ansible appserver -m file -a 'path=/tmp/file02.txt mode=0666 owner=root group=root'
```

```bash
[root@ansible ~]# ansible appserver -m file -a 'path=/tmp/nginx state=directory'
```

```bash
[root@ansible ~]# ansible appserver -m file -a 'path=/tmp/file01.txt state=absent'
```

```bash
[root@ansible ~]# ansible appserver -m file -a 'path=/tmp/hosts src=/etc/hosts state=link'
```

### 6、cron模块

* 管理周期性计划任务
* 常用参数：

- - minute= 指明计划任务的分钟，支持格式：0-59

  - hour= 指明计划任务的小时，支持的语法：0-23

  - day= 指明计划任务的天，支持的语法：1-31

  - month= 指明计划任务的月，支持的语法为：1-12

  - weekday= 指明计划任务的星期几，支持的语法为：0-6

  - name= 给该计划任务取个名称,必须要给明。每个任务的名称不能一样。

  - job= 执行的任务是什么

  - state=present|absent

  - - present	创建		absent	删除 

```bash
[root@ansible ~]# ansible appserver -m cron -a 'name=TimeSync minute=*/30 job="/usr/sbin/ntpdate  120.25.115.20 &> /dev/null" state=present'
```

### 7、yum模块

* 管理RPM软件
* 常用参数

- - name=		软件名称
  - state={present|absent|latest}
  - enablerepo=		启用yum仓库
  - disablerepo=		禁用yum仓库

```bash
[root@ansible ~]# ansible appserver -m yum -a 'name=vsftpd state=present' 
```

### 8、service服务

* **管理系统服务**
  * 即由systemd管理的服务
* 常用参数

- - name=	服务名称
  - state={started|restarted|stopped}
  - enabled={yes|no}

```bash
[root@ansible ~]# ansible appserver -m service -a 'name=vsftpd state=started enabled=yes'
```

### 9、user模块

* 管理用户
* 常用参数

- - name= 				指明要管理的账号名称
  - state=present|absent 	present表示创建，absent表示删除
  - system=yes|no 		指明是否为系统账号
  - uid= 		指明用户UID
  - group= 	指明用户的基本组
  - groups= 	指明用户的附加组
  - shell= 	指明默认的shell
  - home= 	指明用户的家目录
  - password= 	指明用户的密码，最好使用加密好的字符
  - comment= 	指明用户的注释信息
  - remove=yes|no 当state=absent时，也就是删除用户时，是否要删除用户的家目录

```bash
[root@ansible ~]# ansible appserver -m user -a 'name=king groups=jishu shell=/sbin/nologin state=present'
```

### 10、group模块

* 管理用户组
* 常用参数

- - name= 				被管理的组名
  - state=present|absent 	是添加还是删除,不指名默认为添加
  - gid= 				指明GID
  - system=yes|no 	是否为系统组

```bash
[root@ansible ~]# ansible appserver -m group -a 'name=jishu state=present'
```

### 11、script

* 推送脚本、自动执行脚本
* 常用参数
  * 脚本名称

```bash
[root@ansible ~]# ansible appserver -m script -a '/root/test.sh' 
```

### 12、lineinfile模块

* 管理文件内容

```bash
[root@ansible ~]# ansible appserver -m lineinfile -a 'path=/etc/hosts line="10.1.1.1 agent01.linux.com"'
```

### 13、setup模块

* 搜集被管理机 的状态信息（IP地址、主机名、系统版本、内存大小、CPU型号），统称为facts变量

```bash
[root@ansible ~]# ansible appserver -m setup 
```

