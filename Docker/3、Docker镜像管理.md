[toc]

## 一、镜像介绍

![image-20220515112545522](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220515112545522.png)

* 分层的文件系统
* 每一层都是只读的
* 在基于镜像创建容器时，镜像最上边会添加一个读写层

### 1、核心技术

* COW机制   Copy On Write  写时复制
  * 创建容器时，镜像会在最上层添加一个读写曾，只有修改文件时，才会将底层的文件复制到读写层进行修改
  * 容器删除时，读写层会随之删除，所以要对容器做持久化存储
* Union FS 联合文件系统
  * 支持将多个不同的文件系统挂载到同一个虚拟文件系统中
  * overlay2

```bash
[root@localhost ~]# docker info 
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.5.1-docker)
  scan: Docker Scan (Docker Inc., v0.8.0)

Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 9
 Server Version: 20.10.7
 Storage Driver: overlay2
```

## 二、镜像管理操作

### 1、镜像命令格式

* 仓库名称/镜像名称:[标记]
  * 默认标记是 latest

- - test.linux.com/tomcat:8.9
  - polinux/stress
  - nginx
    - hub.docker.com/nginx:latest

### 2、查看下载镜像

```bash
[root@localhost ~]# docker pull mysql:5.6
```

```bash
[root@localhost ~]# docker image ls
```

### 3、导出镜像

```bash
[root@localhost ~]# docker save -o MySQL56.tar mysql:5.6
```

### 4、导入镜像

```bash
[root@localhost ~]# docker load -i MySQL56.tar 
```

### 5、删除镜像

```bash
# docker rmi mysql:5.6
```

### 6、查看镜像的详细信息

```bash
[root@localhost ~]# docker image inspect haproxy:latest 
```

## 三、Dockerfile定制镜像

### 1、Dockerfile使用流程

#### 1）规划目录

```bash
[root@localhost ~]# mkdir demoDockerFile
```

#### 2）编写Dockerfile

```bash
[root@localhost ~]# cd demoDockerFile/
[root@localhost demoDockerFile]# vim Dockerfile
FROM centos:latest
RUN yum install -y net-tools
```

#### 3）构建镜像

```bash
[root@localhost demoDockerFile]# docker build -t centos:v1 ./ 
```

### 2、Dockerfile常用指令

#### 1）FROM指令

指定基础镜像，基础镜像不在，自动联网下载

```bash
FROM  <基础镜像名称>
```

#### 2）MAINTAINER

```bash
MAINTAINER  <user>  <email>
```

指定镜像的作者, 可以使用docker image inspect查看

#### 3）RUN

执行定制的操作，RUN指定的命令在基础镜像中得事先存在

在docker build的时候运行

```bash
RUN   cmd1 && cmd2 && cmd3
```

#### 4）CMD指令

指定创建容器时，自动执行的命令

1. 一个Dockerfile中只能出现一个CMD
2. `写前台启动服务的指令`
3. 创建容器时指定的命令会覆盖CMD指令的命令
4. 在docker run 的时候运行

```bash
CMD  /usr/sbin/httpd -D FOREGROUND
CMD [ "/usr/sbin/httpd", "-D", "FOREGROUND" ]
```

#### 5）ENTRYPOINT

指定创建容器时，自动执行的命令 

创建容器时指定的命令不会覆盖ENTRYPOINT指令的命令

```bash
ENTRYPOINT /usr/sbin/httpd -D FOREGROUND
ENTRYPOINT [ "/usr/sbin/httpd", "-D", "FOREGROUND" ]
```

CMD指令可以作为ENTRYPOINT的参数用

```bash
CMD [ "-D", "FOREGROUND" ]
ENTRYPOINT /usr/sbin/httpd
```

#### 6）COPY

拷贝本地文件、目录

源文件以相对路径的方式写

```bash
COPY <源文件> <目的文件>
```

#### 7）ADD指令

拷贝文件、目录

1. 源文件可以是本地文件、URL地址
2. 源文件是压缩包，相当于自动解压缩

```bash
ADD file01.txt   /tmp/file01.txt 
ADD http://nginx.org/download/nginx-1.21.1.tar.gz   /tmp
```

#### 8）EXPOSE

说明容器要发布的服务端口`仅仅起到了说明作用`

```bash
EXPOSE 80
```

配置-p选项、-P选项真正发布服务

-P  随机发布端口

```bash
[root@localhost testDockerFile]# docker run -tid -P centos:v7 
```

#### 9）VOLUME

指定持久化存储的指令

```bash
VOLUME /webdata
```

创建容器时，自动生成匿名卷进行持久化 

```bash
[root@localhost testDockerFile]# docker inspect 96f0 | grep -i volume
```

#### 10）ENV

在容器中定义环境变量

```bash
ENV 变量名称  值 
```

#### 11）WORKDIR

指定容器当前目录

```bash
WORKDIR <目录名称>
```

#### 12）USER

用于指定执行后续命令的用户和用户组，用户和用户组必须提前存在

```bash
USER <用户名>
```

#### 13）ONBUILD

Dockerfile中指定ONBUILD指定的命令，本次构建镜像的过程中不会执行（test-images），当有新的Dockerfile把test-images作为基础镜像时，就会触发ONBUILD指令，

## 四、实战

### 1、定制源码nginx镜像

* 编写Dockerfile

```bash
FROM centos:7
MAINTAINER  LJH
ADD nginx-1.21.6.tar.gz /tmp
RUN yum -y install gcc openssl-devel pcre-devel zlib-devel  make

RUN cd /tmp/nginx-1.21.6 && ./configure --prefix=/usr/local/nginx && make && make install
EXPOSE 80

RUN rm -rf /tmp/nginx*

ENTRYPOINT /usr/local/nginx/sbin/nginx -g "daemon off;"
```

* 生成镜像

```bash
docker build -t nginx:v2 ./
```

* 生成容器

```bash
[root@localhost ~]# docker run -tid -p 3333:80 --name=nging02 nginx:v2
```

* 访问测试

http://192.168.152.15:3333

### 2、定制tomcat镜像

* 编写Dockerfile

```bash
[root@localhost ~]# cat Dockerfile 
FROM centos:7
MAINTAINER  LJH 

ADD apache-tomcat-10.0.17.tar.gz  /usr/local/
ADD jdk-8u181-linux-x64.tar.gz /usr/local/

ENV JAVA_HOME /usr/local/jdk1.8.0_181
ENV CATALINA_HOME /usr/local/apache-tomcat-10.0.17

EXPOSE 8080

ENTRYPOINT /usr/local/apache-tomcat-10.0.17/bin/catalina.sh run

```

* 生成镜像

```bash
[root@localhost ~]# docker build -t tomcat:v1 ./
Sending build context to Docker daemon  809.8MB
Step 1/8 : FROM centos:7
 ---> eeb6ee3f44bd
Step 2/8 : MAINTAINER  LJH
 ---> Using cache
 ---> b0f7e7b49920
Step 3/8 : ADD apache-tomcat-10.0.17.tar.gz  /usr/local/
 ---> 3e77840e3977
Step 4/8 : ADD jdk-8u181-linux-x64.tar.gz /usr/local/
 ---> a5d86ebd5b06
Step 5/8 : ENV JAVA_HOME /usr/local/jdk1.8.0_181
 ---> Running in 127f02499f95
Removing intermediate container 127f02499f95
 ---> 15cdd82c9061
Step 6/8 : ENV CATALINA_HOME /usr/local/apache-tomcat-10.0.17
 ---> Running in feb5b8d9aebf
Removing intermediate container feb5b8d9aebf
 ---> d66288ecebeb
Step 7/8 : EXPOSE 8080
 ---> Running in ab94efe96595
Removing intermediate container ab94efe96595
 ---> 09cca73811b0
Step 8/8 : ENTRYPOINT /usr/local/apache-tomcat-10.0.17/bin/catalina.sh run
 ---> Running in a070904ef9f4
Removing intermediate container a070904ef9f4
 ---> 120211fe18cb
Successfully built 120211fe18cb
Successfully tagged tomcat:v1

```

* 生成容器

```bash
[root@localhost ~]# docker run -tid -p 4444:8080 --name=tomcat01 tomcat:v1
9d533c7a5323aeeb7704ed757f4dc728c25df84665456c2981f0d1166751aafe
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS                        PORTS                                       NAMES
9d533c7a5323   tomcat:v1   "/bin/sh -c '/usr/lo…"   3 seconds ago    Up 2 seconds                  0.0.0.0:4444->8080/tcp, :::4444->8080/tcp   tomcat01
1e4b9eb5b6ed   nginx:v2    "/bin/sh -c '/usr/lo…"   12 minutes ago   Up 12 minutes                 0.0.0.0:3333->80/tcp, :::3333->80/tcp       nging02
8ba14b36bd67   nginx:v1    "/bin/sh -c '/usr/lo…"   13 minutes ago   Exited (126) 13 minutes ago                                               nging01

```

* 访问测试

http://192.168.152.15:4444