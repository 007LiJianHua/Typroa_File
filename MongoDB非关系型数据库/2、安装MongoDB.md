[toc]

环境：

CentOS 7
MongoDB 3.4.9

------

### 1、下载MongoDB

首先去MongoDB官网下载MongoDB，地址https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.9.tgz。将下载的文件放到/opt目录下。  

### 2、解压

解压下载到的tgz文件,并给文件夹重命名为mongodb，然后创建db、logs目录分别用来存放数据和日志。如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIoHGMHhHZqJjeaXFK4jW4Xzm7p5qHu1cRwGyPXqnL7HicSDdI31VjdTqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 3、配置

进入到bin目录下，编辑mongodb.conf文件，内容如下：

```bash
dbpath=/opt/mongodb/db
logpath=/opt/mongodb/logs/mongodb.log
port=27017
fork=true
nohttpinterface=true
```

执行结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIociaxK5EcZpKbvp0iabZJSbxeqlibialJthoiajL5IV2kBsH8ibsBzjrdiaVGQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 4、测试

做完这一切之后，我们就可以启动MongoDB了，还是在bin目录下，执行`./mongod -f mongodb.conf`命令表示启动MongoDB，然后执行`mongo`命令表示表示进入到MongDB的控制台，进入到控制台之后，我们输入`db.version()`命令，如果能显示出当前MongoDB的版本号，说明安装成功了。如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIowGrRy7G9GJQlAu88iavFrKAZKggBk3shUAqw4cbiccoLE5SyXiaXoVrpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

默认情况下，连接地址是127.0.0.1:27017，连接的数据库是test数据库，我们也可以手动指定连接地址和连接的数据库：

```bash
mongo 127.0.0.1:27017/admin
```

此时连接成功之后，输入db命令，我们可以看到 库是admin。

### 5、配置开机启动

我们也可以配置开机启动，编辑/etc/rc.d/rc.local文件，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIok1AGymDfeygCpfvsYMqE1PDlZ6epBcXoXuvGGY4tujXGYbKuf611RQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

配置完成之后自行关机重启测试。

### 6、配置环境变量

每次都要进入到安装目录中去输入命令，麻烦，我们直接配置环境变量即可，编辑当前用户目录下的.bash_profile文件，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIopicu3GZmFPAr2UTqqJGKHbgc1wcneibtX2yaLgK7essoQRKQw4Wzjgwg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 7、关闭MongoDB服务

使用`db.shutdownServer();`命令可以关闭到MongoDB服务，但是这个命令的执行要在admin数据库下，所以先切换到admin，再关闭服务，完整运行过程如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmCxXGHl7ashxFG4jENCVIoFiaOzzVVgMY9KSyVqibLCYhxxKJF6E7gzzWWh2kWZWd7S6q4YJ8Gb7ZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 8、安全管理

上面我们所做的所有的操作都没有涉及到用户，我们在用Oracle、MySQL或者MSSQL时都有用户名密码需要登录才可以操作，MongoDB中当然也有，但是需要我们手动添加。在添加之前，我们先来说说MongoDB中用户管理的几个特点：

> 1.MongoDB中的账号是在某一个库里边进行设置的，**我们在哪一个库里边进行设置，就要在哪一个库里边进行验证**。
> 2.创建用户时，我们需要指定用户名、用户密码和用户角色，用户角色表示了该用户的权限。

OK，假设我给admin数据库创建一个用户，方式如下：

```bash
use admin
db.createUser({user:"root",pwd:"123",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
```

user表示`用户名`，pwd表示`密码`，role表示`角色`，db表示这个`用户应用在哪个数据库`上。用户的角色，有如下几种：
1.`Read`：允许用户读取指定数据库
2.`readWrite`：允许用户读写指定数据库
3.`dbAdmin`：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
4.`userAdmin`：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
5.`clusterAdmin`：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限
6.`readAnyDatabase`：只在admin数据库中可用，赋予用户所有数据库的读权限
7.`readWriteAnyDatabase`：只在admin数据库中可用，赋予用户所有数据库的读写权限
8.`userAdminAnyDatabase`：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
9.`dbAdminAnyDatabase`：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限
10.`root`：只在admin数据库中可用。超级账号，超级权限

用户创建成功之后，我们关闭掉当前MongoDB服务实例，然后重新启动新的实例，启动方式如下：

```bash
mongod -f /opt/mongodb/bin/mongodb.conf --auth
```

启动成功之后，如果我们直接执行如下命令，会提示没有权限：

```bash
show dbs
```

执行结果如下：

```bash
"errmsg" : "not authorized on admin to execute command { listDatabases: 1.0 }",
"code" : 13,
"codeName" : "Unauthorized"
```

此时我们需要先进入到admin数据库中，然后授权，操作如下：

```bash
use admin
db.auth("root","123")
```

auth方法执行结果返回1表示认证成功。然后再去执行show dbs就可以看到预期结果了。此时我再在sang库下创建一个只读用户，如下：

```bash
use sang
db.createUser({user:"readuser",pwd:"123",roles:[{role:"read",db:"sang"}]})
```

创建成功之后，再按照上面的流程进入到sang库中，使用readuser用户进行认证，认证成功之后一切我们就可以在sang库中执行查询操作了，步骤如下：

```bash
use sang
db.auth("readuser","123")
```

做完这两步之后再执行查询操作就没有任何问题了，但是此时如果执行插入操作会提示没有权限，那我们可以创建一个有读写功能的用户执行相应的操作，这里就不再赘述。