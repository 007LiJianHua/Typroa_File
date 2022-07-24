[toc]

## 一、环境描述

- 192.168.140.10	jenkins服务器
- 192.168.140.11	gitlab服务器
- 192.168.140.12	Tomcat服务器

## 二、在Gitlab服务器上传代码

### 1、在Gitlab服务器创建仓库

![image-20220423204603474](https://s2.loli.net/2022/04/23/xChUVFtsgL3DdmE.png)

### 2、下载项目到本地

```bash
$ git clone http://192.168.140.11/root/projectHelloWorld.git
```

### 3、添加项目代码，上传项目

```bash
$ ls
pom.xml  src/  target/

$ git add ./*
$ git commit -m "add Hello world

$ git push -u origin master
```

![image-20220423204713679](https://s2.loli.net/2022/04/23/dt9Rh5klLPujr3q.png)

## 三、部署Tomcat服务器

![image-20220423204854920](https://s2.loli.net/2022/04/23/nrpcxmhAXwl7Ku1.png)

## 四、安装插件

- git插件

- - 用来自动clone   gitlab仓库的插件

- gitlab插件

- - 作用: 

  - - 连接gitlab服务器拉取代码 

- maven intergration插件

- - 支持maven编译、发布(为什么选择maven编译？)

- publish over ssh插件

- - 借助SSH协议连接tomcat服务器，发布项目 

## 五、在Jenkins与Tomcat服务器之间配置免密SSH

* 否则jenkins在自动发布项目的时候还需要输入密码

```bash
[root@jenkins ~]# ssh-keygen -t rsa
[root@jenkins ~]# ssh-copy-id root@192.168.140.12 

[root@jenkins ~]# ssh root@192.168.140.12 hostname
tomcat_server.linux.com
```

## 六、在jenkins服务器安装git工具

```bash
[root@jenkins ~]# yum install -y git 
```

## 七、在Jenkins服务器安装maven编译工具

* c++写的包解压需要gcc，对应maven写的工具，需要maven编译工具

```bash
[root@jenkins ~]# tar xf apache-maven-3.6.3-bin.tar.gz -C /usr/local/
[root@jenkins ~]# mv /usr/local/apache-maven-3.6.3/ /usr/local/maven36
[root@jenkins ~]# vim /etc/profile
export PATH=$PATH:$JAVA_HOME/bin:/usr/local/maven36/bin
[root@jenkins ~]# source /etc/profile

[root@jenkins ~]# mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/local/maven36
Java version: 1.8.0_91, vendor: Oracle Corporation, runtime: /usr/local/jdk1.8.0_91/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.el7.x86_64", arch: "amd64", family: "unix"
```

## 八、配置Jenkins连接Gitlab

* 基于gitlab令牌token进行认证，就是为jenkins连接gitlab指定个地址

### 1、在Gitlab生成令牌

![image-20220423205328458](https://s2.loli.net/2022/04/23/K8zaWisBkIANmX2.png)

![image-20220423205332675](https://s2.loli.net/2022/04/23/BhJeyzouqjd7fZX.png)

![image-20220423205337183](https://s2.loli.net/2022/04/23/b5RBn4viO2PVGhp.png)

## 2、在Jenkins中添加Gitlab地址、认证令牌

![image-20220423205441543](https://s2.loli.net/2022/04/23/dHU4bkWm9hf2vx8.png)

![image-20220423205446021](https://s2.loli.net/2022/04/23/iMclXfdWqIaTbNB.png)

![image-20220423205449797](https://s2.loli.net/2022/04/23/6sHZdRxnEo1QlIf.png)

## 九、配置Jenkins全局工具

### 1、配置JDK

![image-20220423205524435](https://s2.loli.net/2022/04/23/4IYx3adRX2LPqen.png)

### 2、配置Git客户端工具

![image-20220423205615620](https://s2.loli.net/2022/04/23/DlhmIV7jv9uQAs3.png)

### 配置maven编译工具

![](https://s2.loli.net/2022/04/23/ioNtqOXSsW4IdU5.png)

## 十、配置远程tomcat地址

### 1、配置tomcat服务器地址

![image-20220423205713100](https://s2.loli.net/2022/04/23/zdyKoiUnqmDACTS.png)

### 2、配置SSH认证的密钥

![image-20220423205738099](https://s2.loli.net/2022/04/23/g37dJGnozjWybQ6.png)

## 十一、创建项目发布任务

### 1、创建任务

![image-20220423205817476](C:%5CUsers%5CAdministrator%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20220423205817476.png)

### 2、设置Gitlab连接

* 当有多个Gitlab仓库时，选择一个

![image-20220423205857387](https://s2.loli.net/2022/04/23/KWC7Z42aiuevGVM.png)

### 3、源码管理

* 指定项目代码仓库的位置

![image-20220423210057203](https://s2.loli.net/2022/04/23/dxlOMDPSmoXgVC9.png)

### 4、构建触发器