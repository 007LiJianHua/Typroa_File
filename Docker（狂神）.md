[toc]

## 一、Docker 概述

### 1、Docker为什么会出现

> 一款产品： 开发–上线 两套环境！应用环境，应用配置！
>
> 环境配置是十分的麻烦，每一个及其都要部署环境(集群Redis、ES、Hadoop…) !费事费力。
>
> 发布一个项目( jar + (Redis MySQL JDK ES) ),项目能不能带上环境安装打包！
>
> 之前在服务器配置一个应用的环境 Redis MySQL JDK ES Hadoop 配置超麻烦了，不能够跨平台。
>
> 开发环境Windows，最后发布到Linux！
>
> 传统：开发jar，运维来做！
>
> 现在：开发打包部署上线，一套流程做完！
>
> 安卓流程：java — apk —发布（应用商店）一 张三使用apk一安装即可用！
>
> docker流程： java-jar（环境） — 打包项目帯上环境（镜像） — ( Docker仓库：商店）-----
>
> Docker给以上的问题，提出了解决方案！

![image-20220326193911736](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326193911736.png)

### 2、Docker历史

> 2010年，几个的年轻人，就在美国成立了一家公司 `dotcloud`
>
> 做一些pass的云计算服务！LXC（Linux Container容器）有关的容器技术！
>
> Linux Container`容器`是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源。
>
> 他们将自己的技术（容器化技术）命名就是 Docker
> Docker刚刚延生的时候，没有引起行业的注意！dotCloud，就活不下去！
>
> `开源`
>
> 2013年，Docker开源！
>
> 越来越多的人发现docker的优点！火了。Docker每个月都会更新一个版本！
>
> 2014年4月9日，Docker1.0发布！
>
> docker为什么这么火？十分的轻巧！
>
> 在容器技术出来之前，我们都是使用虚拟机技术！
>
> **虚拟机：在window中装一个VMware，通过这个软件我们可以虚拟出来一台或者多台电脑！笨重！**
>
> 虚拟机也属于虚拟化技术，Docker容器技术，也是一种虚拟化技术！
>
> Docker基于Go语言开发的！开源项目！
>
> Docker官网：https://www.docker.com/
>
> 文档：https://docs.docker.com/ Docker的文档是超级详细的！
>
> 仓库：https://hub.docker.com/

### 3、Docker能干嘛

> * 之前的虚拟机技术
> * 在一个操作系统内核上运行需要的库，再跑一些应用，
> * 资源占用多，冗余步骤多，启动慢
>
> ![image-20220326193929899](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326193929899.png)
>
> * 容器化技术
>   * 容器化技术不是模拟一个完整的操作系统。
> * **比较Docker容器与虚拟机技术的不同**
>   * 传统虚拟机，虚拟出一个硬件，运行一个完整的操作系统，然后在这个操作系统上安装和运行软件。
>   * 容器内的应用直接运行在宿主机上，容器是没有自己的内核的，也没有虚拟的硬件，所以轻便了，
>   * 每个容器间相互隔离，每个容器间都有i自己的文件系统，互不影响。
>
> ![image-20220326193946248](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326193946248.png)
>
> * **DevOps(开发---运维)**
>   * ==应用更快的交付和部署，==
>     * 传统：阅读帮助文档，安装程序。
>     * Docker：打包镜像，发布测试，一件运行，
>   * ==更便捷的升级和扩缩容==
>     * 使用Docker之后，我们部署应用就像搭积木一样，
>     * 项目打包成一个镜像，扩展服务器A，服务器B
>   * ==更简单的系统运维==
>     * 在容器化之后，我们的开发，测试环境都是高度一致的，
>   * ==更高效的利用系统资源==
>     * Docker是内核级别的虚拟化，可以在一个物理机上运行很多的容器示例，服务器的性能可以被压榨到极致



## 二、Docker安装



### 1、Docker的基本组成

![image-20220326194004714](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326194004714.png)

* `镜像（image）`：
  * docker镜像就好比一个目标，可以通过这个目标来创建容器服务，==nginx镜像----> run----->容器==，通过这个镜像可以创建多个容器（最终服务运行或者项目运行是在容器中的）。
* `容器（container）`:
  * Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的
  * 启动、停止、删除、基本命令
  * 目前就可以把这个容器理解为就是一个简易的Linux系统
* `仓库（repository）`：
  * 仓库就是存放镜像的地方
  * 仓库分为私有仓库、共有仓库
  * Docker Hub是国外的
  * 可以通过阿里云、来配置镜像加速

### 2、安装Docker

Docker安装帮助文档：https://docs.docker.com/engine/install/

* Linux要求`内核3.0`以上

```bash
[root@localhost ~]# uname -r
3.10.0-1160.el7.x86_64
```

* 卸载旧版本

```bash
[root@localhost ~]# yum remove docker \
                   docker-client \
                   docker-client-latest \
                   docker-common \
                   docker-latest \
                   docker-latest-logrotate \
                   docker-logrotate \
                   docker-engine
Loaded plugins: fastestmirror
No Match for argument: docker
No Match for argument: docker-client
No Match for argument: docker-client-latest
No Match for argument: docker-common
No Match for argument: docker-latest
No Match for argument: docker-latest-logrotate
No Match for argument: docker-logrotate
No Match for argument: docker-engine
No Packages marked for removal
```

* 需要的安装包

```bash
[root@localhost ~]# yum install -y yum-utils
```

* 设置镜像的仓库

```bash
[root@localhost ~]# yum-config-manager \
     --add-repo \
     https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo

[root@localhost ~]# ll /etc/yum.repos.d/ | grep docker
-rw-r--r--  1 root root 2081 Mar 26 12:49 docker-ce.repo

```

* 更新yum软件包索引

```bash
[root@localhost ~]# yum makecache fast
```

* 安装Docker社区版

```bash
#docker-ce 社区版 而ee是企业版
[root@localhost ~]# yum install docker-ce docker-ce-cli containerd.io
```

* 启动docker并测试

```bash
[root@localhost ~]# systemctl start docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

#查看docker版本信息
[root@localhost ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.14
 API version:       1.41
 Go version:        go1.16.15
 Git commit:        a224086
 Built:             Thu Mar 24 01:49:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.14
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.15
  Git commit:       87a90dc
  Built:            Thu Mar 24 01:48:24 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.11
  GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc:
  Version:          1.0.3
  GitCommit:        v1.0.3-0-gf46b6ba
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

#测试
[root@localhost ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
[root@localhost ~]# 

```

* 查看下载的镜像

```bash
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   6 months ago   13.3kB
```

* 如何卸载docker

```bash
#先卸载软件包
[root@localhost ~]# yum remove docker-ce docker-ce-cli containerd.io
#再删除docker的工作目录
[root@localhost ~]# rm -rf /var/lib/docker
```

### 3、阿里云镜像加速

![image-20220326132719050](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326132719050.png)

`管理控制台` `容器镜像服务` `镜像工具` `镜像加速器` `CentOS`

* 创建daemon.json文件

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zmja481a.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 4、docker run运行图

![image-20220326194054915](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326194054915.png)

### 5、docker底层原理

* docker是怎样工作的？
  * Docker是一个CS架构的系统，Docker的==守护进程==运行在主机上，==通过Socket==从客户端访问。
  * Docker服务端接收到一个Docker客户端的指令就会执行这个命令

![image-20220326133451025](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326133451025.png)



* 为什么docker比VMware块
  * docker有着比虚拟机更少的抽象层，由于==docker不需要Hypervisor实现硬件资源虚拟化==，运行在docker容器上的程序直接使用的都==是实际物理机的硬件资源==，因此在CPU、内存利用率上docker会有更明显的优势。
  * docker利用的是宿主机的内核，而不需要Guest OS
    * Guest OS：VM（虚拟机）里的操作系统（OS）
    * HostOS：物理机里的操作系统（OS）
  * 因此，当新建一个容器时，不需要和虚拟机一样重新加载操作系统的内核，避免引导、加载操作系统内核返个比较费时费资源的过程。当新建一个虚拟机时,虚拟机软件需要加载GuestOS,返个新建过程是**分钟级别**的。
  * 而docker由于直接利用宿主机的操作系统,则省略了这个复杂的过程,因此新建一个docker容器只需要**几秒钟**

![image-20220326194124433](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326194124433.png)



## 三、Docker命令

### 1、帮助命令

Docker帮助文档的地址：https://docs.docker.com/engine/reference/commandline/build/

```bash
docker version    #显示docker的版本信息。
docker info       #显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help #帮助命令
```

### 2、镜像命令

```bash
docker images #查看所有本地主机上的镜像 可以使用docker image ls代替

docker search 搜索镜像

docker pull 下载镜像 docker image pull

docker rmi 删除镜像 docker image rm
```

### 3、docker images解释

```bash
[root@localhost ~]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
mysql                 5.7                 e73346bdf465        24 hours ago        448MB

# 解释
REPOSITORY			# 镜像的仓库源
TAG				# 镜像的标签
IMAGE ID			# 镜像的id
CREATED			# 镜像的创建时间
SIZE				# 镜像的大小
 可选项
Options:
  -a, --all             Show all images (default hides intermediate images) #列出所有镜像
  -q, --quiet           Only show numeric IDs # 只显示镜像的ID
[root@localhost ~]# docker images -aq ＃显示所有镜像的id
e73346bdf465
d03312117bb0
d03312117bb0
602e111c06b6
2869fc110bf7
470671670cac
bf756fb1ae65
5acf0e8da90b
```

### 4、docker search 搜索镜像

```bash
[root@localhost ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9500                [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3444                [OK]  
# --filter=STARS=3000 #搜索出来的镜像就是STARS大于3000的
Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
#过滤出starts数量大于等于3000的
➜  ~ docker search mysql --filter=STARS=3000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9500                [OK]             
mariadb             MariaDB is a community-developed fork of MyS…   3444                [OK]

```

### 5、docker pull 下载镜像

```bash
# 下载镜像 docker pull 镜像名[:tag]
[root@localhost ~]# docker pull tomcat:8
8: Pulling from library/tomcat #如果不写tag，默认就是latest
90fe46dd8199: Already exists   #分层下载： docker image 的核心 联合文件系统,下载过的不会再次下载
35a4f1977689: Already exists 
bbc37f14aded: Already exists 
74e27dc593d4: Already exists 
93a01fbfad7f: Already exists 
1478df405869: Pull complete 
64f0dd11682b: Pull complete 
68ff4e050d11: Pull complete 
f576086003cf: Pull complete 
3b72593ce10e: Pull complete 
Digest: sha256:0c6234e7ec9d10ab32c06423ab829b32e3183ba5bf2620ee66de866df640a027  # 签名 防伪
Status: Downloaded newer image for tomcat:8
[root@localhost ~]# docker.io/library/tomcat:8 #真实地址

#等价于
[root@localhost ~]# docker pull tomcat:8
[root@localhost ~]# docker pull docker.io/library/tomcat:8
```

### 6、docker rmi 删除镜像

```bash
[root@localhost ~]# docker rmi -f 镜像id #删除指定的镜像
[root@localhost ~]# docker rmi -f 镜像id 镜像id 镜像id 镜像id#删除指定的镜像
[root@localhost ~]# docker rmi -f $(docker images -aq) #删除全部的镜像
```

### 7、容器命令

```bash
docker run 镜像id 新建容器并启动

docker ps 列出所有运行的容器 docker container list

docker rm 容器id 删除指定容器

docker start 容器id #启动容器
docker restart 容器id #重启容器
docker stop 容器id #停止当前正在运行的容器
docker kill 容器id #强制停止当前容器
```

#### 1）docker run 选项说明

```bash
docker run [可选参数] image | docker container run [可选参数] image 
#参书说明
--name="Name"		容器名字 tomcat01 tomcat02 用来区分容器
-d					后台方式运行
-it 				使用交互方式运行，进入容器查看内容
-p					指定容器的端口 -p 8080(宿主机):8080(容器)
		-p ip:主机端口:容器端口
		-p 主机端口:容器端口(常用)
		-p 容器端口
		容器端口
-P(大写) 				随机指定端口
# 测试、启动并进入容器
#注意，容器的运行是依赖于一个持续的命令，命令或者进程结束，容器就会退出
#容器就是为了执行任务而生，执行任务结束，容器退出，就像虚拟机关机一样
#所以在run后边可以执行一些命令，
[root@localhost ~]#  docker run -it centos /bin/bash
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
8a29a15cefae: Already exists 
Digest: sha256:fe8d824220415eed5477b63addf40fb06c3b049404242b31982106ac204f6700
Status: Downloaded newer image for centos:latest
[root@95039813da8d /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

#### 2）列出容器命令

```bash
#docker ps命令
#列出当前正在运行的容器
  -a, --all             #包括曾经使用过的所有容器
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -q, --quiet           #以容器ID的形式来显示容器
  
  ➜  ~ docker ps   
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
68729e9654d4        portainer/portainer   "/portainer"             14 hours ago        Up About a minute   0.0.0.0:8088->9000/tcp   funny_curie
d506a017e951        nginx                 "nginx -g 'daemon of…"   15 hours ago        Up 15 hours         0.0.0.0:3344->80/tcp     nginx01
➜  ~ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                       PORTS                    NAMES
95039813da8d        centos                "/bin/bash"              3 minutes ago       Exited (0) 2 minutes ago                              condescending_pike
1e46a426a5ba        tomcat                "catalina.sh run"        11 minutes ago      Exited (130) 9 minutes ago                            sweet_gould
14bc9334d1b2        bf756fb1ae65          "/hello"                 3 hours ago         Exited (0) 3 hours ago                                amazing_stonebraker
f10d60f473f5        bf756fb1ae65          "/hello"                 3 hours ago         Exited (0) 3 hours ago                                dreamy_germain
68729e9654d4        portainer/portainer   "/portainer"             14 hours ago        Up About a minute            0.0.0.0:8088->9000/tcp   funny_curie
677cde5e4f1d        elasticsearch         "/docker-entrypoint.…"   15 hours ago        Exited (143) 8 minutes ago                            elasticsearch
33eb3f70b4db        tomcat                "catalina.sh run"        15 hours ago        Exited (143) 8 minutes ago                            tomcat01
d506a017e951        nginx                 "nginx -g 'daemon of…"   15 hours ago        Up 15 hours                  0.0.0.0:3344->80/tcp     nginx01
24ce2db02e45        centos                "/bin/bash"              16 hours ago        Exited (0) 15 hours ago                               hopeful_faraday
42267d1ad80b        bf756fb1ae65          "/hello"                 16 hours ago        Exited (0) 16 hours ago                               ecstatic_sutherland
➜  ~ docker ps -aq
95039813da8d
1e46a426a5ba
14bc9334d1b2
f10d60f473f5
68729e9654d4
677cde5e4f1d
33eb3f70b4db
d506a017e951
24ce2db02e45
42267d1ad80b
```

#### 3）退出容器命令

```bash
exit #容器直接退出
ctrl +P +Q #容器不停止退出
```

#### 4）删除容器

```bash
docker rm 容器id   #删除指定的容器，不能删除正在运行的容器，如果要强制删除 rm -rf
docker rm -f $(docker ps -aq)  #删除指定的容器
docker ps -a -q|xargs docker rm  #删除所有的容器
```

#### 5）启动和停止容器的操作

```bash
docker start 容器id	#启动容器
docker restart 容器id	#重启容器
docker stop 容器id	#停止当前正在运行的容器
docker kill 容器id	#杀死的不是容器本身，杀死的是运行容器的命令，就是让容器停掉
```

### 8、其他常用命令

#### 1）后台启动命令

```bash
# 命令 docker run -d 镜像名
[root@localhost ~]#  docker run -d centos
a8f922c255859622ac45ce3a535b7a0e8253329be4756ed6e32265d2dd2fac6c
[root@localhost ~]# docker ps           
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
# 问题docker ps. 发现centos 停止了
# 常见的坑，docker容器使用后台运行，就必须要有要一个前台进程，docker发现没有应用，就会自动停止,就是说必须得让容器做点什么
# nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
ctrl +P +Q #容器不停止退出
```

#### 2）查看日志

```bash
[root@localhost ~]# docker logs --help
Options:
      --details        Show extra details provided to logs 
*  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
*      --tail string    Number of lines to show from the end of the logs (default "all")
*  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
   
docker run -d centos /bin/sh -c "while true;do echo 6666;sleep 1;done" #模拟日志      
#显示日志
-tf		#显示日志信息（一直更新）
--tail number #需要显示日志条数
[root@localhost ~]# docker logs -t --tail n 容器id #查看n行日志
[root@localhost ~]# docker logs -ft 容器id #跟着日志

```

#### 3）查看容器中的进程信息

```bash
# 命令 
[root@localhost ~]# docker top 容器id
```

#### 4）查看镜像的元数据信息

```bash
# 命令
[root@localhost ~]# docker inspect 容器id

#测试
[root@localhost ~]# docker inspect 55321bcae33d
...............
..............

```

#### 5）进入当前正在运行的容器

```bash
# 我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 方式一
#bashshell这个命令也可以换成ls 、mkdir 等命令
[root@localhost ~]# docker exec -it 容器id bashshell

#测试
[root@localhost ~]#  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
55321bcae33d        centos              "/bin/sh -c 'while t…"   10 minutes ago      Up 10 minutes                           bold_bell
a7215824a4db        centos              "/bin/sh -c 'while t…"   13 minutes ago      Up 13 minutes                           zen_kepler
55a31b3f8613        centos              "/bin/bash"              15 minutes ago      Up 15 minutes                           lucid_clarke
[root@localhost ~]# 	 docker exec -it 55321bcae33d /bin/bash
[root@55321bcae33d /]# 

# 方式二
docker attach 容器id
#测试
docker attach 55321bcae33d 
正在执行当前的代码...
区别
#docker exec #进入当前容器后开启一个新的终端，可以在里面操作。（常用）
#docker attach # 进入容器正在执行的终端
```

#### 6）导出容器

```bash
docker export -o test.tar 23te 
```

#### 7）导出容器

```bash
docker import test.tar
```



#### 8）从容器内拷贝到主机上

```bash
docker cp 容器id:容器内路径   主机目的路径
#进入docker容器内部
➜  ~ docker exec -it  55321bcae33d /bin/bash 
[root@55321bcae33d /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
#新建一个文件
[root@55321bcae33d /]# echo "hello" > java.java
[root@55321bcae33d /]# cat java.java 
hello
[root@55321bcae33d /]# exit
exit

#这里拷贝的时候不管容器是否正在运行，只要这个容器中存在数据，就可以复制出来
➜  ~ docker cp 55321bcae33d:/java.java /    #拷贝
➜  ~ cd /              
➜  / ls  #可以看见java.java存在
bin   home            lib         mnt   run       sys  vmlinuz
boot  initrd.img      lib64       opt   sbin      tmp  vmlinuz.old
dev   initrd.img.old  lost+found  proc  srv       usr  wget-log
etc   java.java       media       root  swapfile  var  

```

### 9、docker的所有命令

![image-20220326144348969](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326144348969.png)

* docker命令帮助文档

```bash
attach      Attach local standard input, output, and error streams to a running container
  #当前shell下 attach连接指定运行的镜像
  build       Build an image from a Dockerfile # 通过Dockerfile定制镜像
  commit      Create a new image from a container's changes #提交当前容器为新的镜像
  cp          Copy files/folders between a container and the local filesystem #拷贝文件
  create      Create a new container #创建一个新的容器
  diff        Inspect changes to files or directories on a container's filesystem #查看docker容器的变化
  events      Get real time events from the server # 从服务获取容器实时时间
  exec        Run a command in a running container # 在运行中的容器上运行命令
  export      Export a container's filesystem as a tar archive #导出容器文件系统作为一个tar归档文件[对应import]
  history     Show the history of an image # 展示一个镜像形成历史
  images      List images #列出系统当前的镜像
  import      Import the contents from a tarball to create a filesystem image #从tar包中导入内容创建一个文件系统镜像
  info        Display system-wide information # 显示全系统信息
  inspect     Return low-level information on Docker objects #查看容器详细信息
  kill        Kill one or more running containers # kill指定docker容器
  load        Load an image from a tar archive or STDIN #从一个tar包或标准输入中加载一个镜像[对应save]
  login       Log in to a Docker registry #
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

```

### 10、作业练习

Docker Hub仓库  https://registry.hub.docker.com/，不知道想要安装的版本可以去Docker Hub上参考

> 1、Docker安装nginx

* 先search搜索nginx，想要查看nginx的具体版本，可以去Docker Hub 上查看

```bash
[root@localhost ~]# docker search nginx
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                             Official build of Nginx.                        16516     [OK]       
bitnami/nginx                                     Bitnami nginx Docker Image                      120                  [OK]
```

* 安装下载nginx

```bash
[root@localhost ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

* 后台启动nginx并将nginx的80端口映射到本地的3333端口

```bash

[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 months ago   141MB
mysql         latest    3218b38490ce   3 months ago   516MB
hello-world   latest    feb5d9fea6a5   6 months ago   13.3kB
[root@localhost ~]# 
[root@localhost ~]# docker run -d --name nginx01 -p 3333:80
# -d 后台启动
# --name  起了一个别名
# -p 暴露端口:容器端口
"docker run" requires at least 1 argument.
See 'docker run --help'.

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
[root@localhost ~]# docker run -d --name nginx01 -p 3333:80 nginx
b7917891e36c0c93ff2054049ac7a95bc53098326a4db317211adbd362befb60
[root@localhost ~]# curl 127.0.0.1:3333

<title>Welcome to nginx!</title>

[root@localhost ~]#  
```

* 端口暴露的概念
  * 这个是以在阿里云的服务器为例，docker内的多个容器是相互隔离的，通过-p 选项，使得外部网络可以通过3344这个端口，穿过防火墙，连接到服务器上，再连接到docker中的nginx容器
  * 如果端口不暴露，那么只有本机和本机上的容器可以访问

![image-20220326153106856](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326153106856.png)



* 进入nginx01

```bash
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                   NAMES
b7917891e36c   nginx     "/docker-entrypoint.…"   13 minutes ago   Up 13 minutes   0.0.0.0:3333->80/tcp, :::3333->80/tcp   nginx01
[root@localhost ~]# docker exec -it b7917891e36c /bin/bash
root@b7917891e36c:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@b7917891e36c:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@b7917891e36c:/# cd /etc/nginx/
root@b7917891e36c:/etc/nginx# ls
conf.d	fastcgi_params	mime.types  modules  nginx.conf  scgi_params  uwsgi_params

```

> 2、Docker 安装tomcat

* 先搜索一下tomcat镜像，想下载其他的版本，可以去Docker Hub

```bash
[root@localhost ~]# docker search tomcat
NAME                                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
tomcat                                         Apache Tomcat is an open source implementati…   3293      [OK]       
```

* 下载tomcat镜像

```bash
[root@localhost ~]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
668f6fcc5fa5: Pull complete 
dc120c3e0290: Pull complete 
8f7c0eebb7b1: Pull complete 
77b694f83996: Pull complete 
0f611256ec3a: Pull complete 
4f25def12f23: Pull complete 
Digest: sha256:9dee185c3b161cdfede1f5e35e8b56ebc9de88ed3a79526939701f3537a52324
Status: Downloaded newer image for tomcat:latest
docker.io/library/tomcat:latest
```

* 启动tomcat容器

```bash
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 months ago   141MB
tomcat        latest    fb5657adc892   3 months ago   680MB
mysql         latest    3218b38490ce   3 months ago   516MB
hello-world   latest    feb5d9fea6a5   6 months ago   13.3kB
[root@localhost ~]# 
[root@localhost ~]# docker run -d -p 4444:8080 --name tomcat01 tomcat 
de947902a673cee8e9c00c8e574fbe0521fc0aeb83abca4496076449a2ab89ce
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                       NAMES
de947902a673   tomcat    "catalina.sh run"        5 minutes ago    Up 5 minutes    0.0.0.0:4444->8080/tcp, :::4444->8080/tcp   tomcat01
b7917891e36c   nginx     "/docker-entrypoint.…"   29 minutes ago   Up 29 minutes   0.0.0.0:3333->80/tcp, :::3333->80/tcp       nginx01
```

* 这个时候访问会发现服务器报错，这时进入tomcat的webapps目录，查看没有东西
  * 这是因为阿里云镜像的原因，默认是最小的镜像，所有不必要的都踢出，保证最小的环境实用性

```bash
[root@localhost ~]# 
[root@localhost ~]# docker exec -it tomcat01 /bin/bash
root@de947902a673:/usr/local/tomcat# ls -l
total 132
-rw-r--r-- 1 root root 18994 Dec  2 22:01 BUILDING.txt
-rw-r--r-- 1 root root  6210 Dec  2 22:01 CONTRIBUTING.md
-rw-r--r-- 1 root root 60269 Dec  2 22:01 LICENSE
-rw-r--r-- 1 root root  2333 Dec  2 22:01 NOTICE
-rw-r--r-- 1 root root  3378 Dec  2 22:01 README.md
-rw-r--r-- 1 root root  6905 Dec  2 22:01 RELEASE-NOTES
-rw-r--r-- 1 root root 16517 Dec  2 22:01 RUNNING.txt
drwxr-xr-x 2 root root  4096 Dec 22 17:07 bin
drwxr-xr-x 1 root root    22 Mar 26 15:46 conf
drwxr-xr-x 2 root root  4096 Dec 22 17:06 lib
drwxrwxrwx 1 root root    80 Mar 26 15:46 logs
drwxr-xr-x 2 root root   159 Dec 22 17:07 native-jni-lib
drwxrwxrwx 2 root root    30 Dec 22 17:06 temp
drwxr-xr-x 2 root root     6 Dec 22 17:06 webapps
drwxr-xr-x 7 root root    81 Dec  2 22:01 webapps.dist
drwxrwxrwx 2 root root     6 Dec  2 22:01 work
root@de947902a673:/usr/local/tomcat# cd webapps
root@de947902a673:/usr/local/tomcat/webapps# ls
root@de947902a673:/usr/local/tomcat/webapps# 
#实际的网页文件在webapps.dist中，将webapps.dist下边的文件复制过去，即可

root@de947902a673:/usr/local/tomcat# 
root@de947902a673:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@de947902a673:/usr/local/tomcat# ls webapps.dist/
ROOT  docs  examples  host-manager  manager
root@de947902a673:/usr/local/tomcat# cp -r webapps.dist/* webapps
webapps/      webapps.dist/ 
root@de947902a673:/usr/local/tomcat# cp -r webapps.dist/* webapps/
root@de947902a673:/usr/local/tomcat# ls webapps/
ROOT  docs  examples  host-manager  manager
#刷新网页http://192.168.152.15:4444即可访问
```

### 11、可视化面板

* portainer
  * docker的图形化管理工具，提供后台图形面板供我们操作
  * 中间的是一些配置参数，先跳过

```bash
[root@localhost ~]# docker run  -d -p 8080:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

可以在浏览器输入： http://192.168.152.10:8080

创建用户名-----> 密码------>本地监控

## 四、Docker镜像

### 1、镜像是什么

> 镜像是一种轻量级，可执行 的独立软件包，==用来打包软件基本运行环境，和基于运行环境开发的软件，它包含运行某个软件所需要的所有内容，==包括代码，库、环境变量和配置文件

### 2、Docker镜像加载原理

> UnionFS（联合文件系统）

UnionFS（联合文件系统）：是一种分层，轻量级并且高性能的文件系统，他==支持对文件系统的修改作为一次提交来层层叠加==，同时可以将不同的目录挂载到同一个文件系统下，

UnionFS文件系统是Docker镜像的基础，镜像可以通过分层来继承，基于基础镜像，可以制作各种具体的应用镜像

两个容器==共用==同一个操作系统centos

![image-20220326173657292](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326173657292.png)



> Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。
bootfs(boot file system）主要包含 bootloader和 Kernel, bootloader主要是引导加 kernel, Linux刚启动时会加bootfs文件系统，在 Docker镜像的最底层是 bootfs。这一层与我们典型的Linux/Unix系统是一样的，==包含boot加載器和内核==。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs转交给内核，此时系统也会卸载bootfs。
rootfs（root file system),在 bootfs之上。包含的就是典型 Linux系统中的/dev,/proc,/bin,/etc等标准目录和文件。 rootfs就是各种不同的操作系统发行版，比如 Ubuntu, Centos等等。

是一个层级概念

![阿松大](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326173844451.png)

* 平时安装的虚拟机的centos都好几个G，而容器中的centos只有200M左右
  * 这是因为对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层会直接使用Host的kernel，自己只需要提供rootfs就可以了，
  * 由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此，不同的发行版本可以公用bootfs

### 3、分层理解

> 分层的镜像

在下载镜像的时候会发现，镜像是分层下载的

```bash
[root@localhost ~]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
668f6fcc5fa5: Pull complete 
dc120c3e0290: Pull complete 
8f7c0eebb7b1: Pull complete 
77b694f83996: Pull complete 
0f611256ec3a: Pull complete 
4f25def12f23: Pull complete 
Digest: sha256:9dee185c3b161cdfede1f5e35e8b56ebc9de88ed3a79526939701f3537a52324
Status: Downloaded newer image for tomcat:latest
docker.io/library/tomcat:latest
```

这样下载的最大的好处就是==资源共享==，假如有多个镜像都是从相同的base镜像构建而来，那么宿主机只需要在磁盘上保存一份base镜像即可，同时内存中只需要加载一份base镜像，这样就可以为所有容器服务了，而且镜像的每一层都可以被共享

* `理解`
  * 所有的 Docker镜像都起始于一个基础镜像层，当进行修改或培加新的内容时，就会在当前镜像层之上，创建新的镜像层。
  * 举一个简单的例子，假如基于 Ubuntu Linux16.04创建一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加 Python包，就会在基础镜像层之上创建第二个镜像层；
  * 如果继续添加一个安全补丁，就会创健第三个镜像层该像当前已经包含3个镜像层，如下图所示（这只是一个用于演示的很简单的例子）。

![image-20220326180159929](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326180159929.png)

* 在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要。下图中举了一个简单的例子，每个镜像层包含3个文件，而镜像包含了来自两个镜像层的6个文件。

![image-20220326180249468](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326180249468.png)

* 上图中的镜像层跟之前图中的略有区別，主要目的是便于展示文件
* 下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像==只有6个文件==，这是因为最上层中的文件7是文件5的一个更新版

![image-20220326180520144](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326180520144.png)

* 这种情況下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中。
* Docker通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。
* Linux上可用的存储引撃有AUFS、 Overlay2、 Device Mapper、Btrfs以及ZFS。顾名思义，每种存储引擎都基于 Linux中对应的文件系统或者块设备技术，井且每种存储引擎都有其独有的性能特点。
* Docker在 Windows上仅支持 windowsfilter 一种存储引擎，该引擎基于NTFS文件系统之上实现了分层和CoW 
* 下图展示了与系统显示相同的三层镜像。所有镜像层堆并合井，对外提供统一的视图。

![image-20220326180846301](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326180846301.png)

> 特点：

Docker镜像是只读的，当容器启动时，一个新的可写层加载到镜像的顶部，

这一层就是我们常说的容器层，容器之下的都叫镜像层

![image-20220326181217398](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220326181217398.png)

### 4、commit镜像

> docker commit ：提交自己的容器

``` bash
docker commit -m="提交的描述信息" -a="作者" 容器ID 目标镜像名:[TAG]
```

* 实战：启用一个tomcat、加入网页目录文件，commit

```bash
[root@localhost ~]# docker run -d -p 4444:8080 --name tomcat01 tomcat
2bbf2580a7719b707e58e4b4eb759a3a4b88c51683b643f7543a6c355bc2a378
[root@localhost ~]# docker exec -it 2bbf2580a771 /bin/bash
root@2bbf2580a771:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@2bbf2580a771:/usr/local/tomcat# cp -r webapps.dist/* webapps/
root@2bbf2580a771:/usr/local/tomcat# exit
exit
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED         STATUS         PORTS                                       NAMES
2bbf2580a771   tomcat    "catalina.sh run"   2 minutes ago   Up 2 minutes   0.0.0.0:4444->8080/tcp, :::4444->8080/tcp   tomcat01
[root@localhost ~]# 
[root@localhost ~]# docker commit -a="LiJianHua" -m="Add a tomcat's webapps" 2bbf2580a771 tomcat02:1.0
sha256:ed1c6bdc91cc00be114f87960501ce44a8af4c3a97974a5bdae3283878eff43f
[root@localhost ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
tomcat02              1.0       ed1c6bdc91cc   10 seconds ago   684MB
nginx                 latest    605c77e624dd   2 months ago     141MB
tomcat                latest    fb5657adc892   3 months ago     680MB
mysql                 latest    3218b38490ce   3 months ago     516MB
hello-world           latest    feb5d9fea6a5   6 months ago     13.3kB
portainer/portainer   latest    580c0e4e98b0   12 months ago    79.1MB

```

## 五、容器数据卷

### 1、什么是数据卷

docker**的理念是将应用和环境打包成一个镜像在**

然而，如果数据都在容器中，在删除容器的时候，数据就会永久丢失，`需求：数据的持久化`

容器之间有一个可以数据共享的技术：`数据卷：`可以实现Docker容器中产生的数据，同步到本地！

简而言之就是将容器的目录，挂载到Linux上面

![image-20220326202620558](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/202203262026635.png)

**容器的持久化和同步操作，容器间也是可以数据共享的**

### 2、使用数据卷

> 方式一：直接使用命令来挂载 -v

```bash
docker run -it -v 主机目录:容器目录
```

```bash
[root@localhost ~]# docker run -it -v /tmp/:/home centos /bin/bash
```

因为挂载的是同一个数据节点，所以可以相互写入文件

![image-20220326210306571](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/202203262103699.png)

```bash
#在这里可以看到挂载的源目录和目的目录
inspect：可以查看容器的详细文件
[root@localhost ~]# docker inspect 4dd7b03c4796 | grep -C 7 "Mount" | tail -10
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/tmp",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
[root@localhost ~]# 
```

**好处：在本地修改文件后，容器内就会自动同步**

### 3、实战：实现MySQL的数据持久化

* 以安装MySQL5.7为例

* -d：后台运行

* -p：映射端口

* -v：创建数据卷（挂载多个目录同样的操作）

* -e：修改环境配置

* --name：别名

* 最后指定镜像名称:[TAG]

* [TAG]:标签

```bash
    [root@localhost ~]# docker run -d -p 3306:3306 -v /etc/mysql/conf:/etc/mysql/conf.d -v /etc/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=redhat --name mysql01 mysql:5.7

Unable to find image 'mysql:5.7' locally
5.7: Pulling from library/mysql
72a69066d2fe: Pull complete 
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
0ceb82207cd7: Pull complete 
37f2405cae96: Pull complete 
e2482e017e53: Pull complete 
70deed891d42: Pull complete 
Digest: sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f
Status: Downloaded newer image for mysql:5.7
79b91e4803d9198f3a755caa09fd1852468c38aac3980632a5eaea75778b78d5

```

* 这个时候使用Navicat连接`docker_mysql01`的数据库，连接成功

![image-20220326213006450](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/202203262130599.png)

* 再图形化创建一个数据库，在虚拟机的`/etc/mysql/data`上可以看到新建的数据库信息

![image-20220326213831749](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/202203262138835.png)

* 就算这个时候把docker_mysql01删掉，存在主机上的信息也不会删除，从而实现了==数据的持久化存储==

![image-20220326214259916](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/202203262142994.png)

### 4、容器的具名和匿名挂载

`匿名挂载：`-v：容器内的路径

-P：指定本机的一个随机端口进行映射，

```bash
[root@localhost ~]# docker run -d -P --name nginx01 -v /etc/nginx nginx
```

可以使用docker volume ls查看匿名

```bash
[root@localhost ~]# docker volume ls
DRIVER    VOLUME NAME
local     3bb7e6ad57cf16e3b7089e497a39bd2a963ab7ada6defc304767537de79798a7
local     3980efc18a8a60839a7e46d8153028108341fc225d1542b0f7c9e68ea5836eb9
local     b6a2a7573f411b40ecca9260a0368d9173508735580f3b5412ffb1e975335905
local     c214a4bbd68689bab468f62d42e759b582c0fdfe99ed2c6c8a8877b3cd67962d
```

 `具名挂载：` -v：数据卷名称：容器内路径

使用docker inspect 具名：来查看具体挂载到了本机哪个目录

```bash
[root@localhost ~]# docker run -d -P --name nginx02 -v test_volume:/etc/nginx nginx
25cf5472e9bb9be27d2f9beacad1f8095344a5b16ab1fc8d149168c218152538
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
25cf5472e9bb   nginx     "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:49154->80/tcp, :::49154->80/tcp   nginx02
90794276c824   nginx     "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:49153->80/tcp, :::49153->80/tcp   nginx01
[root@localhost ~]# 
[root@localhost ~]# docker volume ls
DRIVER    VOLUME NAME
local     3bb7e6ad57cf16e3b7089e497a39bd2a963ab7ada6defc304767537de79798a7
local     3980efc18a8a60839a7e46d8153028108341fc225d1542b0f7c9e68ea5836eb9
local     b6a2a7573f411b40ecca9260a0368d9173508735580f3b5412ffb1e975335905
local     c214a4bbd68689bab468f62d42e759b582c0fdfe99ed2c6c8a8877b3cd67962d
local     test_volume

#可以看到Mountpoint的位置就是挂载的位置
[root@localhost ~]# docker inspect test_volume 
[
    {
        "CreatedAt": "2022-03-27T06:07:58+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/test_volume/_data",
        "Name": "test_volume",
        "Options": null,
        "Scope": "local"
    }
]
```

所有docker容器内的卷，没有指定目录的情况下，都是在`/var/lib/docker/volumes/xxxxxx/_data`

通过具名挂载可以很方便的找打需要的卷，==工作中使用的更多的也是具名挂载==

> 如何确定是具名挂载还是匿名挂载，还是指定路径挂载
>
> -v：容器内路径		               匿名挂载
>
> -v：卷名：容器内路径    		具名挂载
>
> -v：宿主机路径：容器内路径   指定路径挂载

**扩展**

```bash
-v 容器内路径:ro		只读  只能在容器外部改变
-v 容器内路径:rw		读写  可以相互改变

#一旦设定了容器的权限，容器对挂载出来的内容就有限定了
[root@localhost ~]# docker run -d -P --name nginx02 -v test_volume:/etc/nginx:ro nginx
[root@localhost ~]# docker run -d -P --name nginx02 -v test_volume:/etc/nginx:rw nginx
```

### 5、初识DockerFile

> DockerFile就是用来构建Dokcer镜像的构建文件！命令的脚本，
>
> 通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的执行，每个命令都是一层

```bash
#创建一个dockerfile文件，名字随机，建议dockerfile
FROM centos
VOLUME ["volume01","volume02"]
CMD echo "...........end................"
CMD  /bin/bash
```



通过docker build命令使用刚才创建的文件，也能实现数据卷的挂载

-f：写dockerfile的绝对路径，

-t：target。写出想要创建的容器名字和版本 最后还有一个   .

```bash
[root@localhost docker_test]# docker build -f /tmp/docker_test/dockerfile.txt -t ljh_centos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 5d0da3dc9764
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in b5d7052120c8
Removing intermediate container b5d7052120c8
 ---> 4546a63ab451
Step 3/4 : CMD echo "...........end................"
 ---> Running in b77e6805e779
Removing intermediate container b77e6805e779
 ---> e2f8c6580403
Step 4/4 : CMD  /bin/bash
 ---> Running in e943ab64519c
Removing intermediate container e943ab64519c
 ---> 0c4c59b13688
Successfully built 0c4c59b13688
Successfully tagged ljh_centos:1.0
[root@localhost docker_test]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ljh_centos   1.0       0c4c59b13688   6 seconds ago   231MB
nginx        latest    605c77e624dd   2 months ago    141MB
mysql        5.7       c20987f18b13   3 months ago    448MB
centos       latest    5d0da3dc9764   6 months ago    231MB
[root@localhost docker_test]# docker run -it ljh_centos:1.0 /bin/bash
[root@6cb13930bf8b /]# 
[root@6cb13930bf8b /]# ls -l
total 0
......
drwxr-xr-x   2 root root   6 Mar 27 13:36 volume01
drwxr-xr-x   2 root root   6 Mar 27 13:36 volume02
[root@6cb13930bf8b /]# cd volume01/
[root@6cb13930bf8b volume01]# ls
[root@6cb13930bf8b volume01]# touch test.txt
[root@6cb13930bf8b volume01]# ls
test.txt
[root@localhost docker_test]# docker inspect 0c4c59b13688  | grep "Mount"
       "Mounts": [
            {
                "Type": "volume",
                "Name": "09fc84998d4c5883c51ec12a24fe8e505c8a2a18014fd9f3ffead2c92e87dd80",
                "Source": "/var/lib/docker/volumes/09fc84998d4c5883c51ec12a24fe8e505c8a2a18014fd9f3ffead2c92e87dd80/_data",
                "Destination": "volume01",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "9272926bc496c178d58adb8645a102c6a611a362fa021f7d267ed93d3a53bf15",
                "Source": "/var/lib/docker/volumes/9272926bc496c178d58adb8645a102c6a611a362fa021f7d267ed93d3a53bf15/_data",
                "Destination": "volume02",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        
[root@localhost docker_test]# cd /var/lib/docker/volumes/9272926bc496c178d58adb8645a102c6a611a362fa021f7d267ed93d3a53bf15/
[root@localhost 9272926bc496c178d58adb8645a102c6a611a362fa021f7d267ed93d3a53bf15]# ll
drwxr-xr-x 2 root root 6 Mar 27 22:10 _data
[root@localhost 9272926bc496c178d58adb8645a102c6a611a362fa021f7d267ed93d3a53bf15]# cd _data/
[root@localhost _data]# ls
test.txt
#可以看到创建test。txt文件
```

### 6、容器数据卷

* 可以利用父容器来实现多个容器同步数据

![image-20220327135213715](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327135213715.png)

> 启动三个容器，通过刚才写的镜像启动，从而实现这三个容器的数据同步

* 先启动docker01，在volume01的目录下留下自己的名字docker01

```bash
[root@localhost ~]# docker run -it --name docker01 ljh_centos:1.1 /bin/bash
[root@e3e7b82abbae /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@e3e7b82abbae /]# cd volume01
[root@e3e7b82abbae volume01]# ls
[root@e3e7b82abbae volume01]# touch docker01
[root@e3e7b82abbae volume01]# ls
docker01
#CTRL+P+Q：可以不停止容器退出
[root@e3e7b82abbae volume01]# [root@localhost ~]# 
```

* 再通过刚才的镜像启动一个新的容器，也在volume01的目录下留下自己的名字docker02
  * --volume-from：继承父容器的数据目录，通过这个可以选项实现多个容器之间的数据同步

```bash
[root@localhost ~]# docker run -it --name docker02  --volumes-from=docker01 ljh_centos:1.1 
[root@bbec79fad742 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@bbec79fad742 /]# cd volume01
[root@bbec79fad742 volume01]# ls
docker01
[root@bbec79fad742 volume01]# touch docker02
[root@bbec79fad742 volume01]# ls
docker01  docker02
[root@bbec79fad742 volume01]# [root@localhost ~]# 
```

* 最后再通过这个镜像启动一个容器docker03，在volume01的目录下留下自己的名字docker03

```bash
[root@localhost ~]# docker run -it --name docker03  --volumes-from=docker01 ljh_centos:1.1 
[root@b0e4985e096b /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@b0e4985e096b /]# cd volume01
[root@b0e4985e096b volume01]# ls
docker01  docker02
[root@b0e4985e096b volume01]# touch docker03
[root@b0e4985e096b volume01]# ls
docker01  docker02  docker03
[root@b0e4985e096b volume01]# [root@localhost ~]# 
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS     NAMES
b0e4985e096b   ljh_centos:1.1   "/bin/sh -c /bin/bash"   37 seconds ago       Up 36 seconds                 docker03
bbec79fad742   ljh_centos:1.1   "/bin/sh -c /bin/bash"   About a minute ago   Up About a minute             docker02
e3e7b82abbae   ljh_centos:1.1   "/bin/bash"              3 minutes ago        Up 3 minutes                  docker01
```

## 六、DockerFile

### 1、DockerFile介绍

==DockerFile是用来构建Docker镜像的文件！命令的参数脚本==

> 构建步骤
>
> 1、编写一个DockerFile文件
>
> 2、docker build 构建成为一个镜像
>
> 3、docker run 运行镜像
>
> 4、docker push发布镜像（Docker Hub、阿里云）

* 可以参考一下官方的，在浏览器输入https://hub.docker.com/  输入centos

![image-20220327150852257](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327150852257.png)

* 随便点击一个版本



![image-20220327150827494](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327150827494.png)

* 会跳转到GitHub上，框起来的就是一个DockerFile

![image-20220327150735657](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327150735657.png)

* 很多官方的镜像都是基础包，很多功能都没有，我们通常会搭建自己的镜像，

### 2、DockerFile构建过程

> ==基础知识==
>
> 1、每个保留关键字（指令）都是必须是大写字母
>
> 2、执行顺序从上到下
>
> 3、#代表注释
>
> 4、每个指令都会提交创建一个新的镜像层，并提交

![image-20220327151539926](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327151539926.png)

### 3、DockerFile的指令

```bash
FROM    		#基础镜像，一切从这里开始
MAINTAINER      #镜像是谁写，姓名+邮箱
RUN				#镜像 构建的时候需要运行的命令
ADD				#当构建tomcat镜像的时候可以将tomcat的压缩包添加进来
WORKDIR			#镜像的工作目录
VOLUME			#挂载的目录
EXPOSE			#暴露端口配置
CMD				#指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		#指定这个容器启动的时候要运行的命令，不可被替代，属于追加
ONBUILD			#当构建一个被继承的DockerFile，这个时候就会运行ONBUILD的指令，触发指令
COPY			#类似ADD，将我们的文件拷贝到镜像中
ENV				#构建的时候设置环境变量
```



![image-20220327151816694](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327151816694.png)



### 4、实战测试

> Docker HUb中99%的镜像都是从这个基础镜像过来的 FROM scratch 然后配置需要的软件来进行构建

![image-20220327150735657](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220327150735657.png)

> 编写一个自己的centos的dockerfile

* 为了方便期间，我创建一了一个/tmp/my_dockerfile_centos的目录，在这里边写了一个DockerFile文件

```bash
[root@localhost my_dockerfile_centos]# cat dockerfile 
#这里如果不指定版本的话，会自动去Docker Hub上下载最新的版本，而最新的centos8已经不维护了，所以只能指定一个7的版本
FROM centos:7
#姓名+邮箱（用来指定维护者）
MAINTAINER LiJianHua<saita_ma@163.com>

#指定一个新的环境变量
ENV MYPATH /usr/local
#将打开容器后进入的目录也就是工作目录指定为/usr/local
WORKDIR $MYPATH

#为centos7的镜像添加一个vim、和net-tools的安装包
RUN yum -y install vim
RUN yum -y install net-tools

#指定暴露端口
EXPOSE 80

#在容器运行时执行的指令
CMD echo $MYPATH
CMD echo "---------------------------end-----------------------"
CMD /bin/bash
```

* 使用docker build 命令加载dockerfile文件

```bash
[root@localhost my_dockerfile_centos]# docker build -f dockerfile  -t mycentos:1.0 .
Sending build context to Docker daemon  2.048kB
#本地没有centos:7的镜像，所以要先去下载
Step 1/10 : FROM centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete 
Digest: sha256:9d4bcbbb213dfd745b58be38b13b996ebb5ac315fe75711bd618426a630e0987
Status: Downloaded newer image for centos:7
 ---> eeb6ee3f44bd
Step 2/10 : MAINTAINER LiJianHua<saita_ma@163.com>
 ---> Running in f69fdc93a092
Removing intermediate container f69fdc93a092
 ---> 220adaf46fb7
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in 63085173aa67
Removing intermediate container 63085173aa67
 ---> 2d21768522e3
Step 4/10 : WORKDIR $MYPATH
 ---> Running in db7304f70198
Removing intermediate container db7304f70198
 ---> f552b0239592
Step 5/10 : RUN yum -y install vim
 ---> Running in c35d2bca07b6
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: mirrors.bupt.edu.cn
 * extras: mirrors.bupt.edu.cn
 * updates: mirrors.bfsu.edu.cn
Resolving Dependencies
--> Running transaction check
。。。。。。
```

* 此时，镜像已经创建完毕

```bash
[root@localhost my_dockerfile_centos]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mycentos     1.0       b72131e8dc96   29 seconds ago   580MB
ljh_centos   1.0       0c4c59b13688   4 hours ago      231MB
ljh_centos   1.1       0c4c59b13688   4 hours ago      231MB
nginx        latest    605c77e624dd   2 months ago     141MB
mysql        5.7       c20987f18b13   3 months ago     448MB
centos       7         eeb6ee3f44bd   6 months ago     204MB
centos       latest    5d0da3dc9764   6 months ago     231MB
[root@localhost my_dockerfile_centos]# docker run -it b72131e8dc96 /bin/bash
[root@3c021a56094d local]# ls
bin  etc  games  include  lib  lib64  libexec  sbin  share  src
[root@3c021a56094d local]# pwd
/usr/local
[root@3c021a56094d local]# vim aa.txt
[root@3c021a56094d local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 8  bytes 656 (656.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@3c021a56094d local]# pwd
/usr/local
```

### 5、列出镜像变更历史的命令

docker history 镜像ID

```bash
[root@localhost my_dockerfile_centos]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mycentos     1.0       b72131e8dc96   16 minutes ago   580MB
ljh_centos   1.0       0c4c59b13688   5 hours ago      231MB
ljh_centos   1.1       0c4c59b13688   5 hours ago      231MB
nginx        latest    605c77e624dd   2 months ago     141MB
mysql        5.7       c20987f18b13   3 months ago     448MB
centos       7         eeb6ee3f44bd   6 months ago     204MB
centos       latest    5d0da3dc9764   6 months ago     231MB
[root@localhost my_dockerfile_centos]# docker history b72131e8dc96
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b72131e8dc96   16 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
3de660c2bfaa   16 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
dbba201a21f1   16 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
b73b9e44c70b   16 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
0539236dc78a   16 minutes ago   /bin/sh -c yum -y install net-tools             161MB     
fe2d488733ea   16 minutes ago   /bin/sh -c yum -y install vim                   216MB     
f552b0239592   16 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
2d21768522e3   16 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
220adaf46fb7   16 minutes ago   /bin/sh -c #(nop)  MAINTAINER LiJianHua<sait…   0B        
eeb6ee3f44bd   6 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      6 months ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      6 months ago     /bin/sh -c #(nop) ADD file:b3ebbe8bd304723d4…   204MB     
[root@localhost my_dockerfile_centos]# 
```

### 6、DockerFile中CMD与ENTRYPOINT的区别

* CMD

```bash
[root@localhost my_dockerfile_centos]# cat cmd_dockerfile 
FROM centos:7
CMD ["ls","-a"]

[root@localhost my_dockerfile_centos]# docker build -f cmd_dockerfile -t cmd_test .
[root@localhost my_dockerfile_centos]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
cmd_test     latest    874a5ee43e12   About a minute ago   204MB
mycentos     1.0       b72131e8dc96   30 minutes ago       580MB
ljh_centos   1.0       0c4c59b13688   5 hours ago          231MB
ljh_centos   1.1       0c4c59b13688   5 hours ago          231MB
nginx        latest    605c77e624dd   2 months ago         141MB
mysql        5.7       c20987f18b13   3 months ago         448MB
centos       7         eeb6ee3f44bd   6 months ago         204MB
centos       latest    5d0da3dc9764   6 months ago         231MB

#如果将新建的这个镜像运行在另外一个bash环境中，则不能看到dockerfile中写的CMD命令，直接运行即刻
[root@localhost my_dockerfile_centos]# docker run -it cmd_test /bin/bash
[root@44eb282ab75b /]# exit
exit

[root@localhost my_dockerfile_centos]# docker run cmd_test
.
..
.dockerenv
anaconda-post.log
bin
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

#报错的原因是因为  -l 把dockerfile中的命令替换掉了，不想被替换可以是用 ENTYYPOINT
[root@localhost my_dockerfile_centos]# docker run cmd_test -l
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled
```

* ENTRYPOINT
  * 由下可见CMD与ENTRYPOINT的区别，一个是替换命令，一个是追加命令

```bash
[root@localhost my_dockerfile_centos]# cat entrypoin_dockerfile
FROM centos:7
ENTRYPOINT ["ls","-a"]

[root@localhost my_dockerfile_centos]# docker build -f entrypoin_dockerfile -t entrypoint_centos .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos:7
 ---> eeb6ee3f44bd
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in b6ab130800ea
Removing intermediate container b6ab130800ea
 ---> 7ba434938ae3
Successfully built 7ba434938ae3
Successfully tagged entrypoint_centos:latest
[root@localhost my_dockerfile_centos]# docker images
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
entrypoint_centos   latest    7ba434938ae3   5 seconds ago    204MB
cmd_test            latest    874a5ee43e12   8 minutes ago    204MB
mycentos            1.0       b72131e8dc96   37 minutes ago   580MB
<none>              <none>    62015a92ab73   40 minutes ago   231MB
ljh_centos          1.0       0c4c59b13688   5 hours ago      231MB
ljh_centos          1.1       0c4c59b13688   5 hours ago      231MB
nginx               latest    605c77e624dd   2 months ago     141MB
mysql               5.7       c20987f18b13   3 months ago     448MB
centos              7         eeb6ee3f44bd   6 months ago     204MB
centos              latest    5d0da3dc9764   6 months ago     231MB
[root@localhost my_dockerfile_centos]# docker run entrypoint_centos
.
..
.dockerenv
anaconda-post.log
bin
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
[root@localhost my_dockerfile_centos]# docker run entrypoint_centos -l
total 12
drwxr-xr-x   1 root root     6 Mar 27 18:37 .
drwxr-xr-x   1 root root     6 Mar 27 18:37 ..
-rwxr-xr-x   1 root root     0 Mar 27 18:37 .dockerenv
-rw-r--r--   1 root root 12114 Nov 13  2020 anaconda-post.log
lrwxrwxrwx   1 root root     7 Nov 13  2020 bin -> usr/bin
drwxr-xr-x   5 root root   340 Mar 27 18:37 dev
drwxr-xr-x   1 root root    66 Mar 27 18:37 etc
drwxr-xr-x   2 root root     6 Apr 11  2018 home
lrwxrwxrwx   1 root root     7 Nov 13  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Nov 13  2020 lib64 -> usr/lib64
drwxr-xr-x   2 root root     6 Apr 11  2018 media
drwxr-xr-x   2 root root     6 Apr 11  2018 mnt
drwxr-xr-x   2 root root     6 Apr 11  2018 opt
dr-xr-xr-x 135 root root     0 Mar 27 18:37 proc
dr-xr-x---   2 root root   114 Nov 13  2020 root
drwxr-xr-x  11 root root   148 Nov 13  2020 run
lrwxrwxrwx   1 root root     8 Nov 13  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root     6 Apr 11  2018 srv
dr-xr-xr-x  13 root root     0 Mar 27 13:36 sys
drwxrwxrwt   7 root root   132 Nov 13  2020 tmp
drwxr-xr-x  13 root root   155 Nov 13  2020 usr
drwxr-xr-x  18 root root   238 Nov 13  2020 var

```



### 7、实战：发布tomcat镜像

* 准备镜像文件，tomcat压缩包，jdk压缩包

```bash
[root@localhost my_dockerfile_centos]# ll
total 185856
-rw-r--r-- 1 root root   8931288 Mar 28 04:06 apache-tomcat-7.0.72.tar.gz
-rw-r--r-- 1 root root        30 Mar 28 02:28 cmd_dockerfile
-rw-r--r-- 1 root root       254 Mar 28 01:59 dockerfile
-rw-r--r-- 1 root root        37 Mar 28 02:36 entrypoin_dockerfile
-rw-r--r-- 1 root root 181367942 Mar 28 04:06 jdk-8u91-linux-x64.tar.gz
```

* 编写DockerFile文件，

```bash
FROM centos:7
MAINTAINER LiJianHua<saita_ma@163.com>

COPY README.txt /usr/local/README.txt

ADD apache-tomcat-10.0.17.tar.gz  /usr/local/
ADD jdk-8u181-linux-x64.tar.gz /usr/local/

RUN yum -y install vim net-tools

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_181
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-10.0.17
ENV CATALINA_BASE /usr/local/apache-tomcat-10.0.17
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_HOME/lib

EXPOSE 8080

CMD ["/usr/local/apache-tomcat-10.0.17/bin/catalina.sh","run"]

```

* 构建镜像
  * DockerFIle是官方指定默认的文件名字，所以也可以不需要指定文件，直接   -t   即可

```bash
[root@localhost my_dockerfile_centos]# docker build -t tomcat_test05 ./
Sending build context to Docker daemon  190.3MB
Step 1/15 : FROM centos:7
 ---> eeb6ee3f44bd
Step 2/15 : MAINTAINER LiJianHua<saita_ma@163.com>
 ---> Using cache
 ---> 220adaf46fb7
Step 3/15 : COPY README.txt /usr/local/README.txt
 ---> Using cache
 ---> 3424559e459b
Step 4/15 : ADD apache-tomcat-7.0.72.tar.gz /usr/local/
 ---> Using cache
 ---> 527b3f31e981
Step 5/15 : ADD jdk-8u91-linux-x64.tar.gz /usr/local/
 ---> Using cache
 ---> 77baec20cbd3
Step 6/15 : RUN yum -y install vim net-tools
 ---> Using cache
 ---> c07773b22e37
Step 7/15 : ENV MYPATH /usr/local
 ---> Using cache
 ---> b3830f74fba3
Step 8/15 : WORKDIR $MYPATH
 ---> Using cache
 ---> b62a078b41b0
Step 9/15 : ENV JAVA_HOME /usr/local/jdk1.8.0_91
 ---> Using cache
 ---> f93ec292c81b
Step 10/15 : ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.72
 ---> Using cache
 ---> 33dc0dbf7085
Step 11/15 : ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.72
 ---> Using cache
 ---> dd91ee001a22
Step 12/15 : ENV CLASSPATH $CATALINA_HOME/bin/bootstrap.jar:$CATALINA_HOME/bin/tomcat-juli.jar
 ---> Using cache
 ---> ff9bc6a46734
Step 13/15 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_HOME/lib
 ---> Using cache
 ---> f9af1c3bddba
Step 14/15 : EXPOSE 8080
 ---> Using cache
 ---> 58dd5408ca59
Step 15/15 : CMD ["/usr/local/apache-tomcat-10.0.17/bin/catalina.sh","run"]
 ---> Running in 4c445083f63b
Removing intermediate container 4c445083f63b
 ---> 7c0a350e4388
Successfully built 7c0a350e4388
Successfully tagged tomcat_test05:latest
```

* 启动镜像

```bash
[root@localhost my_dockerfile_centos]# docker run -d -p 3333:8080 --name dasdafasdasdasdandaskn -v /usr/local/tomcat/webapps:/usr/local/apache-tomcat-7.0.72/webapps -v /usr/local/tomcat/tomcatlogs:/usr/local/apache-tomcat-7.0.72/logs tomcat_test09
ef054a35be1c6436d4990f7945905a3007f241733ac6f1f13b4d769a145ddf11

```

* 访问测试（第二次测试有问题，没有网页）

![](https://s2.loli.net/2022/03/28/OkqX4hVLBf3mpWr.png)













## 七、Docker网络原理

## 八、IDEA整合Docker

## 九、Docker Compose

## 十、Docker Swarm

## 十一、CI/CD Jenkins