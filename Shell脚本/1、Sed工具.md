[toc]

# Sed工具

## 一、sed工具介绍

* 是一款流编辑器
* 作用
  * 编辑文本文件
* 工作原理
  * 将文件逐行读入到模式空间（内存），在内存中对文件进行修改，修改完毕后，默认会把模式空间所有内容显示到屏幕上
  * 逐行处理
  * 并不会真正修改文件

## 二、sed常用操作

### 1、sed命令格式

```bash
# sed [option] 'script' 文件名称
---[option]：选项
---'script'：脚本

//通俗的来说
# sed [option] 'lineCMD' 文件名称
---[option]：选项
---'lineCMD'：
	line：代表第几行，不写表示对文件每一行都进行操作
	CMD：命令操作
```

### 2、line常用的写法

* 行号	10
* 起始行号，终止行号
  * 5，9
  * 5，+3   //从第五行开始数，往后三行
* /正则表达式/      ：由一系列元字符构成
* /正则表达式/,/正则表达式/
  * 从第一个匹配的正则表达式到第二个匹配的正则表达式中间的所有行

### 3、常用操作

* d	删除整行

```bash
[root@localhost ~]# sed '1d' /etc/fstab 
[root@localhost ~]# sed '1,5d' /etc/fstab 
[root@localhost ~]# sed '/^#/d' /etc/fstab 
[root@localhost ~]# sed '/^\//d' /etc/fstab 
[root@localhost ~]# netstat -antp | sed '1,2d‘
```

* 显示整行
  * -n：取消默认显示模式空间的内容

```bash
[root@localhost ~]# sed -n '/^#/p' /etc/fstab 

[root@localhost ~]# df -hT | sed -n '/^\/dev/p'
```

* a \内容     
  * 追加内容
  * $：在结尾追加

```bash
[root@localhost ~]# sed '$a \10.1.1.1  node01.linux.com' /etc/hosts
```

* i  \内容
  * 插入内容

```bash
[root@localhost ~]# sed 'i \172.16.10.1 www.linux.com' /etc/hosts
```

* 整行替换
  * c  \内容

```bash
[root@localhost ~]# sed '/#Port/c \Port 55555' /etc/ssh/sshd_config 
```

* w 文件名称		
  * 另存为

```bash
[root@localhost ~]# sed '/^#/w /tmp/test01' /etc/fstab 
```

* r	文件名称	
  * 合并文件

```bash
[root@localhost ~]# sed '$r /etc/hosts' /etc/redhat-release 
```

* =	显示行号/统计行数

```bash
[root@localhost ~]# sed -n '$=' /etc/fstab 
```

* n  读取下一行

```bash
[root@localhost ~]# sed -n '/^$/{n;p}' /etc/fstab 
```

* s/旧内容/新内容/[修饰符]
  * 文件内容查找替换
  * 旧内容支持正则表达式

```bash
[root@localhost ~]# sed '/^UUID=/s/UUID/uuid/' /etc/fstab 
[root@localhost ~]# sed '$s/4/8/' /etc/fstab 

[root@localhost ~]# sed '$s/[0-9]/!/g' /etc/fstab 
[root@localhost ~]# sed 's/[0-9]/!/g' /etc/fstab 

[root@localhost ~]# sed '$s/\//?/g' /etc/fstab 
[root@localhost ~]# sed '$s|/|?|g' /etc/fstab 
```

* 反向引用
  * \1   \2   \3
    * 依次表示引用正则表达式中的第一个分组内容，第二个分组内容，以此类推

```bash
[root@www ~]# cat /opt/file01 
A like B
B love C
[root@localhost ~]# sed 's|\(l..e\)|\1r|' /tmp/file01
[root@localhost ~]# sed 's|l\(..e\)|L\1|' /tmp/file01 
[root@localhost ~]# sed 's|\(l.\)\(.e\)|o\2|' /opt/file01 
```

* & 匹配所有旧内容

```bash
[root@localhost ~]# sed 's|l..e|&r|' /tmp/file01
```

### 4、常用选项

* -n
  * 取消显示模式空间的内容
  * 配合显示内容使用
* -i
  * 修改源文件

```bash
[root@localhost ~]# sed -i '/^$/d' /etc/fstab 
```

* -e
  * 同时做多个修改

```bash
[root@localhost ~]# sed -e '/^#/d' -e '/UUID/d' /etc/fstab 
```

* -f
  * 操作文件

```bash
[root@localhost ~]# cat /tmp/list
/^#/d
/UUID/d
[root@localhost ~]# sed -f /tmp/list  /etc/fstab 
```

* -r
  * 支持扩展正则表达式

```bash
[root@localhost ~]# sed -r 's|l(..e)|L\1|' /tmp/file01 
```

* --follow-symlinks  
  * 支持修改软连接文件

```bash
[root@localhost ~]# sed -ri --follow-symlinks '2d' /tmp/file01 
```

