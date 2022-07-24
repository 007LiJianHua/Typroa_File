[toc]

## Jenkins持续集成工具

> 自动化运维工具一共有两种：

* ansible、saltsstack
  * 都是运维来使用
  * 用来实现服务器的批量管理
* Jekins+Gitlab(CI/CD持续集成交付工具)
  * 主要用来项目更新、发布



## 一、Jenkins介绍

* 基于java开发的、开源的、持续集成/持续交付工具（CI/CD）
* 一个插件话工具
* 在安装的时候最好安装最新的，要不有的时候会因为版本问题导致插件不可用
* 在官网下载的是war包，使用的时候先部署tomcat----> 再将war包放入webapps目录下即可



## 二、Jenkins安装部署

### 1、下载epel源

```bash
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### 2、安装JDK

* 注意JDK的版本

```bash
# yum install java-11-openjdk-devel
```

### 3、安装Jenkins最新版本

* Jenkins最新下载地址

- - https://mirrors.aliyun.com/jenkins/redhat/

```bash
# yum install -y jenkins-2.318-1.1.noarch.rpm
```

### 4、启动Jenkins

```shell
# systemctl start jenkins
# systemctl enable jenkins
[root@jenkins jenkins]# netstat -tunlp | grep java
tcp6       0      0 :::8080                 :::*                    LISTEN      19655/java

```

### 5、修改插件下载地址

```bash
[root@jenkins ~]# cat /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
#这个网址就是jenjins官方的插件地址https://updates.jenkins.io/update-center.json最好改成国内的清华大学地址
    <url>http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
​
[root@jenkins ~]# systemctl restart jenkins
```

### 6、访问Jenkins界面

```bash
http://192.168.140.10:8080        
```

![image-20220423204159603](https://s2.loli.net/2022/04/23/kLQj7OFrvYtJoi6.png)

![image-20220423204214802](https://s2.loli.net/2022/04/23/43qAEfjKdT5x2NV.png)

![](https://s2.loli.net/2022/04/23/f2rIhzP8p3x5OUK.png)

### 7、修改default.json文件，将国内的地址修改为国内插件地址

```bash
[root@jenkins ~]# sed -i 's/https:\/\/updates.jenkins.io\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json
​
[root@jenkins ~]# systemctl restart jenkins
```

### 8、最后在网页测试

```bash
http://192.168.152.12:8080
```

