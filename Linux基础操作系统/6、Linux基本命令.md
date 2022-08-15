# Linux基本命令

# <kbd>听课</kbd><kbd>理解</kbd><kbd>操作</kbd><kbd>记笔记</kbd>

[toc]

[郭玉晨老师云笔记]: https://note.youdao.com/s/1OhkOFzV



## 一、课上笔记：

> * **<font color="red">	su - 意义和家目录的意义？</font>**
>
> 切换用户的时候，会调用/加载该用户家目录的配置信息，例如：单独给该用户配置命令则只有该用户会识别这个命令，其他用户不可以。
>
> * <font color="red">**相对路径与绝对路径**</font>
>
> 以/ 开始，不以/ 开始
>
> * <font color="red">**cd /home    与  cd /home/ 区别？**</font>
>
> 区别home是目录还是文件，最后一个 / 为目录分隔符
>
> * **<font color="red">alias别名</font>**
>
>   命令行输入alias可以查看系统中所有的别名
>
>   注意：在命令行中定义的别名，仅对当前shell进程有效
>   如果想永久有效，要定义在配置文件中
>   **仅对当前用户：~/.bashrc**
>   **对所有用户有效：/etc/bashrc**
>   
> * <font color="red">**file +文件名		//可以查看文件类型**</font>
>
>   Linux下不看扩展名，

## 二、查看文件more、less、cat、head、tail、

> * **小段文件可以用cat查看，长文件用more和less、或者head和tail**
>
> * **cat -n：显示行号**
>
> * <font color="red">**tail -f /var/log/messages 动态监控日志文件**</font>
>
> * **more ： 浏览稍长文件时使用      **
>
>   **` 回车`：下一行**
>
>   **` 空格`：下一页**
>
>   **` b`：上一页**
>
>   **`q` ：退出**
>
> * **less ：浏览长文档，且需要上下翻页时使用**
>
>   **`上键 `：上一行**
>
>   **`下键` ：下一行**
>
>   **`pageup` ：上一页**
>
>   **`pagedown` ：下一页**
>
>   **`q` ：退出**

## 三、创建文件touch

``` linux
touch a.txt				
touch a.txt b.txt
touch a{1..6}.txt  				//利用通配符{}
touch {a,b,c}{1..3}.txt
a1.txt  a2.txt  a3.txt  b1.txt  b2.txt  b3.txt  c1.txt  c2.txt  c3.txt
```

## 四、创建目录mkdir

``` linux
mkdir aa
mkdir -p -v a/b/c				//-p递归创建目录，-v显示创建过程		
mkdir abc{1..3}
```

## 五、删除rm

``` linux
rm -r 			//删除目录，也起到递归作用
rm -f 			//删除文件,强制删除
rm -rf			//都可以删除
```

## 六、复制cp

``` linux
touch a1 a2
mkdir D1 D2
cp -p a1 a2 /tmp		//保留原文件的属性和权限
cp -r D1 /tmp/D3		//复制目录时加上-r 表示 递归复制，否则会报错
[root@localhost opt]# cp /opt/D1/ /tmp/D3
cp: omitting directory ‘/opt/D1/’
```



## 七、移动mv

> **mv file1 /tmp/;ls /tmp/file**		//;是**<font color="red">命令分隔符</font>**，可以输入多条命令

``` linux
vim test.config.simple
[root@localhost opt]# ll
total 4
drwxr-xr-x 2 root root  6 Dec 15 20:50 D1
drwxr-xr-x 2 root root  6 Dec 15 20:49 D2
drwxr-xr-x 2 root root  6 Dec 15 20:50 D3
-rw-r--r-- 1 root root 10 Dec 15 20:57 test.config
[root@localhost opt]# mv test.config{.simple,}
[root@localhost opt]# ls
D1  D2  D3  test.config
```

## 八、文本编辑器vim

[郭玉晨老师云笔记]: https://note.youdao.com/s/linQypC



### 1、基本语法

> * vi是一个文本编辑器，vi工具在大多数发行版本里都会默认安装
> * vim是vi的升级版
> * vim的特点：
>   * 文本中显示不同颜色，管理员通过颜色辨别文本中错误
>   * 支持块图模式
>   * 很多软件也默认使用了vi的编辑方法，比如crontab（定时任务）

```linux
vim file1
a		//输入模式
:wq		//保存推出
```

### 2、VI编辑器的工作模式

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE3de3ee04376b6d5bb7425c31b4ea0a90/15)



> 命令模式（一般模式）：用于光标移动，复制删除行
>
> 输入模式（编辑模式）：编辑文本
>
> 末行模式（命令行模式）：用于设置环境，行号，制表符，缩进。

**命令模式进入输入模式方法：**

|  字母  |        含义        |
| :----: | :----------------: |
|   o    | 当前字符下一行输入 |
|   i    |   当前字符前输入   |
|   a    |   当前字符后输入   |
|   I    |   当前行行首输入   |
|   A    |   当前行行尾输入   |
|   O    |  当前行上一行输入  |
|   s    | 删除当前字符后输入 |
|   S    |  删除当前行后输入  |
| Home键 |        行首        |
| End键  |        行尾        |

**命令模式光标控制：**

|       字母        |             光标移动             |
| :---------------: | :------------------------------: |
|       ↑↓←→        |             上下左右             |
|      0 or ^       |         将光标定位到行首         |
|         $         |         将光标定位到行尾         |
|        gg         |       将光标定位到文章首部       |
|         G         |       将光标定位到文章尾部       |
|        nG         | n代表数字，就是将光标移动到第n行 |
|        yy         |               复制               |
|        nyy        |        n代表数字，复制n行        |
| shift+v+↑↓←→+yy+p |     可视化界面选择想复制的行     |
|         v         |            可视化模式            |
|      ctrl+v       |           可视化模式块           |

**Tips：如何给每行文本添加#？**

> ### <kbd>ctrl+v选中</kbd><kbd>shift+i</kbd><kbd>输入#</kbd><kbd>ESC</kbd>

**复制黏贴删除**

| 字母 |        含义        |
| :--: | :----------------: |
|  y   |        复制        |
|  yy  |      复制一行      |
|  p   |        粘贴        |
|  u   |        撤销        |
| dd+p |        剪切        |
|  d^  | 删除当前字符到行首 |
|  d$  | 删除当前字符到行尾 |
| dgg  |  删除当前行到首行  |
|  dG  |  删除当前行到尾行  |

**末行模式**

|       字符        |              含义              |
| :---------------: | :----------------------------: |
|     shift+；      |               ：               |
|        :w         |            保存退出            |
|        :q         |              退出              |
|        :wq        |            保存退出            |
|        :w!        |            强制保存            |
|        :q!        |            强制退出            |
|       :wq!        |          强制保存退出          |
|  :w /tmp/cc.txt   |             另存为             |
|    :! commend     |       临时输入linux命令        |
|  :e /root/aa.txt  |          打开一个文件          |
|        :e!        |    重新打开这个文件（刷新）    |
|       :set        |        查看所有环境设置        |
| :set nu/:set nonu |       查看行号/取消行号        |
|  :set tabstop=16  |    设置制表符为16个字节长度    |
|  :set autoindent  |            自动缩进            |
|   /关键字，n，N   | 从上往下搜索，n下一个，N上一个 |
|     ？关键字      | 从下往上搜索，n上一个，N下一个 |
|       :noh        |          取消高亮模式          |
|                   |                                |

### 3、vim配置文件：

```linux
vim ~/.vimrc			//为个人永久添加开启行号功能
vim /etc/vimrc			//为所有人永久添加，在最后一行添加
```

### 4、替换语法：

> **/old/new/**
>
> s：行
>
> g：到该行结尾
>
> %：全局

|   末行模式    |            含义            |
| :-----------: | :------------------------: |
|   :s/ab/xx/   | 替换当前行第一个匹配字符串 |
|  :s/ab/xx/g   | 替换当前行所有匹配的字符串 |
|  :%s/ab/xx/g  |  替换所有行中匹配的字符串  |
| :3,5s/ab/xx/g |        替换当3到5行        |

### 5、vim的交换文件：

**前提：两个终端打开了同一个文件会提示**

> 如果机器出现down机，交换文件会记录文件的操作并进行提示回复、删除等操作	

> 窗口1：

```linux
cd ~
vim 1.txt
ctrl+z		//将vim的应用放到后台
```

> 窗口2

```linux
[root@localhost /]# cd ~;vim 1.txt		//此时会出现交换文件
```

> 解决方法：
>
> 删除重复打开的文件进程，并且删除交换文件

```linux
jobs -l 		//详细查看后台程序目录
ps -ef 			//查看进程
kill -9 24234
```

> 再次打开任意窗口，删除交换文件，保存退出即可。

## 九、脚本bash



```linux
vim a2.sh		//脚本必须给权限和第一行
chmod +x a2.sh
#!/bin/bash		//脚本第一行
date			//内容
./a2.sh			//运行
Thu 16 Dec 09:43:39 CST 2021
```

