[toc]

## 一、作用

* 主要记录文本文件的变化（像代码等），便于回退
* 便于多人协同开发

**集中式与分布式的区别**
除了git还有svn、cvs这样的版本控制系统，它们的区别在于一个是分布式一个是集中式

集中式就是svn和csv这样的版本控制系统，分布式是git

区别在于集中式的版本控制系统每次在写代码时都需要从服务器中拉取一份下来，并且如果服务器丢失了，那么所有的就都丢失了，你本机客户端仅保存当前的版本信息，换句话说，集中式就是把代码放在一个服务器上集中管理，你的所有回滚等操作都需要服务器的支持。

分布式的区别在于，每个人的电脑都是服务器，当你从主仓库拉取一份代码下来后，你的电脑就是服务器，无需担心主仓库被删或者找不到的情况，你可以自由在本地回滚，提交，当你想把自己的代码提交到主仓库时，只需要合并推送到主仓库就可以了，同时你可以把自己的代码新建一份仓库分享给其它人。

像集中式它们都有一个主版本号，所有的版本迭代都以这个版本号为主，而分布式因为每个客户端都是服务器，git没有固定的版本号，但是有一个由哈希算法算出的id，用来回滚用的，同时也有一个master仓库，这个仓库是一切分支仓库的主仓库，我们可以推送提交到master并合并到主仓库上，主仓库的版本号会迭代一次，我们客户端上的git版本号无论迭代多少次，都跟master无关，只有合并时，master才会迭代一次。
## 二、安装git工具

```bash
[root@salt-syndic ~]# yum install -y git 
```

### 1、初始化设置

```bash
[root@salt-syndic ~]# git config --global user.name "Martin"
[root@salt-syndic ~]# git config --global user.email "Martin@qq.com"
[root@salt-syndic ~]# git config --global color.ui true
```

## 三、Git常用管理操作

### 1、创建git仓库

```bash
# cd /opt/work
# git init
Initialized empty Git repository in /opt/work/.git/
```

### 2、提交修改

```bash
# git add 文件名称
# git commit -m "说明信息" 文件名称
```

### 3、查看仓库的状态

```bash
[root@salt-syndic work]# git status
# On branch master
nothing to commit, working directory clean
```

### 4、版本回退

```bash
# git reflog
# git reset --hard <版本ID>
```

### 5、删除文件

```bash
# git rm 
# git commit -m "说明信息"
```

### 6、重命名文件

```bash
# git mv 
# git commit -m "说明信息"
```

## 四、暂存区、缓存区

### 1、撤销未提交的修改

*  此时还没有git add

```bash
# git checkout -- <文件名称>
```

### 2、撤销暂存区的修改

* 已经git add
* 先执行 reset
* 再执行  checkout

```bash
# git reset HEAD file02.txt
# git checkout -- file02.txt
```

### 3、撤销工作区的修改

* 直接回滚上一个版本

```bash
# git reset --hard df7e7f7
```

## 五、分支

* 分支之间是互相隔离的

### 1、查看分支

```bash
[root@salt-syndic work]# git branch 
* master
```

### 2、创建、切换分支

```bash
[root@salt-master linux]# git branch AA
[root@salt-master linux]# git branch 
  AA
* master

[root@salt-master linux]# git checkout AA     //切换分支
Switched to branch 'AA'
[root@salt-master linux]# git branch 
* AA
  master
```

### 3、创建分支

```bash
[root@salt-syndic work]# git checkout -b test

[root@salt-syndic work]# git branch 
  master
* test
  update
```

### 4、删除分支

```bash
[root@salt-syndic work]# git branch -d test
```

### 5、合并分支

* **将指定的分支合并到当前分支**

```bash
# git merge <分支名称>
```

## 六、git服务器

* github/gitee

* - https://github.com/
  - https://gitee.com
  - 适用于个人使用 

* gitlab软件

  * 适合公司使用

### 1、部署gitlab服务器

#### 1）添加主机名解析

```bash
[root@gitlab ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.140.15	gitlab.linux.com
```

#### 2）安装gitlab

```bash
[root@gitlab ~]# yum install -y gitlab-ce-10.1.5-ce.0.el7.x86_64.rpm 
```

#### 3）编辑gitlab配置文件，指定web界面的访问地址

```bash
[root@gitlab ~]# vim /etc/gitlab/gitlab.rb 

external_url 'http://192.168.140.15'
```

#### 4）配置启动gitlab

```bash
# gitlab-ctl reconfigure
```

#### 5）访问gitlab

默认用户名root

http://192.168.140.15

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/9194A22982E945258831BCAC55AD4E32/0A2DC4D591F2431BA8551E6D97F3248C/15137](https://s2.loli.net/2022/04/10/9YJOcEsUvyZqP34.png)

**克隆仓库**

```bash
# git clone 仓库地址
```

**上传文件**

> push：将本地仓库与远程仓库合并
>
> -u：将本地仓库分支与远程仓库分支一起合并，就是说将master的分支也提交上去，这样你就可以在远程仓库上看到你在本地仓库的master中创建了多少分支，不加这个参数只将当前的master与远程的合并，没有分支的历史记录，也不能切换分支
>
> origin：远程仓库的意思，如果这个仓库是远程的那么必须使用这个选项
>
> master：提交本地matser分支仓库

```bash
# git push -u origin master 
```

