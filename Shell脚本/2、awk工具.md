[toc]

# awk工具

## 一、awk工具介绍

* 工作流程
  * 默认情况下，awk可以使用空白字符分割文本，awk内部可以分别$1,\$2变量代表第一段内容
  * -F 
    * 指定分隔符
  * 逐行处理
* 作用nm
  * 显示文本

## 二、awk基本使用

```bash
# awk [option] 'script' 文件名称  文件名称 

# awk [option] 'PATTERN{action}'  文件名称  文件名称
```

* awk自带循环
* option
  * 常用选项操作
* PATTERN
  * 条件
* action：
  * print
  * printf
    * 为输出设置格式，可以更整齐
* $0
  * **没有被分割的整行内容**
* $1...\$6
  * 分割的第一列。。。6列

### 1、print操作

```bash
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -F: '{print $1}'
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -F: '{print $0}'
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -F: '{print $1, $3}'

[root@localhost ~]# sed -n '1,3p' /etc/passwd | awk -F: '{print "用户名:",$1}'
[root@localhost ~]# rpm -qa | grep php | awk -F-5 '{print "rpm -e --nodeps", $1}' | bash
```

### 2、printf操作

* 格式化输出内容
* 使用格式
  * printf “格式格式”，内容，内容
  * 一个格式对2应一个内容
  * **printf不会自动换行使用\n,想要整齐输出使用\t**
    * 占位符
      * %s	字符串
      * %d    整数
      * %f     浮点数

```bash
[root@localhost ~]# awk '{printf "%s", $2}' /tmp/file01 
[root@localhost ~]# awk '{printf "%s%s", $2, $3}' /tmp/file01 
```

## 三、awk常用内置变量

1、FS

* 相当于-F  作用
* 保存行分隔符，默认空白

```bash
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -v FS=":" '{print $1}'
```

2、OFS

* 在输出多个变量时，在多个变量中间指定分割符，而非默认空白

```bash
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -v OFS="--->" -F: '{print $1, $7}'
root--->/bin/bash
```

3、**NR**

* 记录操作行的行号
* 在处理多个文件时，所有文件的行号是连续记录的，非连续记录用FNR

```bash
[root@localhost ~]# awk '{print NR}' /etc/hosts /etc/redhat-release 
```

* NR变量也可以当作条件使用

```bash
[root@localhost ~]# awk 'NR==3{print $0}' /etc/passwd
```

4、FNR

* 记录行号
* 处理多个文件时，所有文件的行号时独立记录的

```bash
[root@localhost ~]# awk '{print FNR}' /etc/hosts /etc/redhat-release 
```

5、**NF**

* 记录awk分割后的段数

```bash
[root@localhost ~]# sed -n '1p' /etc/passwd | awk -F: '{print NF}'
```

* 也可以使用$NF来输出每段的最后一段

```bash
[root@localhost ~]# awk '{print $NF}' /tmp/file01 
```

6、FILENMAME

* 保存awk正在处理的文件名

```bash
[root@localhost ~]# awk '{print FILENAME}' /tmp/file01 
/tmp/file01
/tmp/file01
/tmp/file01

[root@localhost ~]# awk '{print FILENAME}' /tmp/file01 | uniq
/tmp/file01
```

## 四、awk自定义变量

​                -v 变量名称=值               

```bash
[root@localhost ~]# awk -v name="martin" '{print "hello", name}' /etc/fstab 
[root@localhost ~]# awk -v number=100 '{print number}' /etc/fstab 
```

## 五、awk常用文本处理模式

1、表达式

* 比较运算符

* - ==, >, >=, <, <=, !=
- 数据 ~ “正则表达式”
  - - 判断数据是否可以被正则表达式
- 数据  !~ 正则表达式
    - 判断数据是否不可以被正则表达式

* 逻辑运算符 

* - &&    并且 
  - ||	或者

```bash
[root@localhost ~]# awk -F: '$3>=1 && $3 <= 999{print $1}' /etc/passwd
[root@localhost ~]# awk -F: '$7=="/sbin/nologin"{print $1}' /etc/passwd
[root@localhost ~]# awk -F: '$7 ~ "nologin$"{print $1}' /etc/passwd
[root@localhost ~]# df -hT | grep "^/dev" | awk '+$6>20{print "磁盘名称:", $1, "剩余空间", $5, "挂载目录:", $NF}'
```

2、正则表达式

```bash
[root@localhost ~]# netstat -antp | awk '/^tcp/{print $0}'
[root@localhost ~]# awk -F: '/^[rhg]/{print $1}' /etc/passwd
[root@localhost ~]# ls -l /etc/ | awk '/^-/{print $NF}'
```

3、/正则/，/正则/

* 常用于分析系统日志

```bash
[root@localhost ~]# awk '/Sep 26 09:02:30/,/Sep 26 09:18:04/{print $0}' /var/log/messages
```

4、BEGIN{}

* 定于awk开始处理内容前执行的操作

```bash
[root@localhost ~]# sed -n '1,3p' /etc/passwd | awk -F: 'BEGIN{print "----数据开始----"}{print $1,$7}' 
[root@localhost ~]# awk 'BEGIN{name="martin"; print name}'·	
```

5、END{}

* 定义awk处理内容后要执行的操作

```bash
[root@localhost ~]# sed -n '1,3p' /etc/passwd | awk -F: 'BEGIN{print "----数据开始-----"}{print $1, $7}END{print "----数据结束----"}'
```

## 六、awk条件判断

### 1、语法

- if (条件) {操作}
- if (条件) {操作} else {操作}
- if (条件) {操作} else if (条件) {操作}

示例：统计bash用户数量和nologin数量

```bash
[root@localhost ~]# awk -F: -v bash_number==0 -v nologin_number==0 '{if ($7=="/bin/bash"){bash_number++}else if ($7=="/sbin/nologin"){nologin_number++}}END{print "bash用户数量：",bash_number,"nologin用户数量：",nologin_number}' /etc/passwd
bash用户数量： 1 nologin用户数量： 17

```

## 七、awk循环

### 1、for循环语法

* for循环是对分割出的每一段进行循环

```bash
for(变量定义; 循环条件; 改变变量的值) {操作}
for(i=1; i<=10; i++)
```

* 示例：打印出/etc/paswd中每一行字符长度大于5的
  * 内置变量length(0)

```bash
[root@localhost ~]# sed -n '1,3p' /etc/passwd | awk -F: '{for(i=1; i<=7; i++) {if(length($i)>=5) {print $i}}}'
/root
/bin/bash
/sbin/nologin
daemon
daemon
/sbin
/sbin/nologin
[root@localhost ~]# awk '{for(i=1; i<=NF; i++) {if(length($i)>=4) {print $i}}}' /etc/passwd 
```

## 八、awk数组

### 1、定义数组

* 数组[自定义下标]=”内容“
  * 数组内不同下标用“；” 隔开
* **数组在BEGIN{}中**

```bash
[root@localhost ~]# awk 'BEGIN{test[1]="martin"; test[2]="robin"; test[3]="lz"}'

[root@localhost ~]# awk 'BEGIN{test[1]="martin"; test[2]="robin"; test[3]="lz"; print test[2]}'
robin
```

### 2、获取数组中所有数据

* 通过 test[i] 循环打印出test数组所有的内容

```bash
[root@localhost ~]# awk 'BEGIN{test[1]="martin"; test[2]="robin"; test[3]="lz"; for(i=1; i<=3; i++) {print test[i]}}'
```

### 3、数组支持以任意数据作为下标

```bash
[root@localhost ~]# awk 'BEGIN{test["martin"]="北京"; test["robin"]="河北"; test["lz"]="天津"}'
[root@localhost ~]# awk 'BEGIN{test["martin"]="北京"; test["robin"]="河北"; test["lz"]="天津"; print test["robin"]}'
河北
```

* 示例1：

  打印下标

```bash
[root@localhost ~]# awk 'BEGIN{test["martin"]="北京"; test["robin"]="河北"; test["lz"]="天津"; for(i in test) {print i}}'
robin
lz
martin
```

​	打印下标对应内容

```bash
[root@localhost ~]# awk 'BEGIN{test["martin"]="北京"; test["robin"]="河北"; test["lz"]="天津"; for(i in test) {print test[i]}}'
河北
天津
北京
```

合并：循环打印

```bash
[root@localhost ~]# awk 'BEGIN{test["martin"]="北京"; test["robin"]="河北"; test["lz"]="天津"; for(i in test) {print "姓名:", i, "住.:", test[i]}}'
姓名: robin 住址: 河北
姓名: lz 住址: 天津
姓名: martin 住址: 北京
```

* 示例2:统计bash和nologin数量并输出
  * data[$7]++
    * 每遇到一个不同的下标的时候自加1，因为一开始并没有这个下标，所以默认为0，

```bash
[root@localhost ~]# awk -F: '{data[$7]++}END{for(i in data) {print i, data[i]}}' /etc/passwd
```

* 示例3：统计web服务中UV（用户访问量）和PV（页面访问量）

```bash
[root@localhost ~]#  awk '{uv[$1]++}END{for(i in uv) {print i, uv[i]}}' access_log
[root@localhost ~]#  awk '{pv[$7]++}END{for(i in pv) {print i, pv[i]}}' access_log
```

## 九、awk的内置函数

1、split(字符串、数组、行分隔符)

* 使用分隔符分割字符串，将分割后的字符串保存到指定数组

```bash
[root@localhost ~]#  awk 'BEGIN{split("a-b-c", data, "-"); print data[1]}'
[root@localhost ~]#  awk 'BEGIN{split("a-b-c", data, "-"); print data[2]}'
[root@localhost ~]#  awk 'BEGIN{split("a-b-c", data, "-"); print data[3]}'
```

2、length(string)

* 返回string字符串的字符个数

```bash
[root@localhost ~]#  awk 'BEGIN{print length("shell")}'
```

3、substr(string,start,[length])

* 取出string字符串中的字串，从start开始，取length个；start从1开始计数

```bash
[root@localhost ~]# awk 'BEGIN{data=substr("hello",2, 3); print data}'
ell
```

4、system(commend)

* 支持在awk内部调用shell命令

```bash
[root@localhost ~]# awk 'BEGIN{system("ifconfig ens33")}'
```

5、systime()

* 返回系统当前时间

```bash
[root@localhost ~]# awk 'BEGIN{print systime()}'
1645035840
```

* 指定返回的格式

```bash
[root@localhost ~]# awk 'BEGIN{now=systime(); print strftime("%F_%T", now)}'
2022-02-17_02:24:36
```

6、tolower(string)

* 将string中的所有字母转为小写

```bash
[root@localhost ~]# awk 'BEGIN{print tolower("aBcD")}'
abcd
```

7、toupper(string)

* 将string中所有字母转换为大写

```bash
[root@localhost ~]# awk 'BEGIN{print toupper("aBcD")}'
ABCD
```



