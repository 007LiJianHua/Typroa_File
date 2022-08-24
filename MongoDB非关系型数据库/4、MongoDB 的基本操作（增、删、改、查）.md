[toc]

## 一、客户端安装配置

上篇文章我们提到可以在[MongoDB](https://so.csdn.net/so/search?q=MongoDB&spm=1001.2101.3001.7020)启动成功之后通过mongo命令进入MongoDB客户端，然后在客户端输入操作命令执行增删改查等操作。当然，我们也可以通过一些客户端工具来连接MongoDB，比如Robo 3T。

首先我们下载Robo 3T(下载地址https://robomongo.org/download)，下载成功之后解压，找到`.exe`[可执行文件](https://so.csdn.net/so/search?q=可执行文件&spm=1001.2101.3001.7020)双击启动，启动后新建一个连接，输入ip地址(**注意连接之前要确保Linux上的MongoDB已经启动**)，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215144565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)

连接成功之后，我们就可以看到数据库的信息了，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215209518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)

## 二、shell 简介

暂时我们所有的操作都先放在test数据库中进行(默认情况下，test数据库为空，这里不显示空的数据库，此时执行可以选中CentOS菜单，右键单击点击Open Shell，默认打开test数据库)，选中test，右键单击，选择Open Shell，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215303207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)
这里的shell是一个功能完整的JavaScript解释器，可以运行JavaScript程序，这个很好玩，如下我定义一个函数然后调用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215322120.png#pic_center)

函数定义和调用的代码写好之后，按左上角的三角符号表示运行，也可以按F5或者Ctrl+Enter组合键。我们也可以调用JavaScript的标准函数库，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215339933.png#pic_center)
再比如调用Date函数，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215356998.png#pic_center)
如果我们没有使用Robo 3T工具，而是直接在命令行通过mongo命令来启动shell，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215417758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)
此时，shell会连接到MongoDB服务器的test数据库，并将数据库连接赋值给全局变量db，我们将通过db这个变量实现很多功能，我们也可以查看db当前指向哪个数据库，直接使用db命令，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215442491.png#pic_center)
再比如我们可以通过use命令来切换数据库（上文中也有用到过），如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215509770.png#pic_center)

## 三、增加操作

在添加之前我们先来说说数据库的创建，上文我们提到了use命令，表示切换到某一个数据库中去，如果我们想切换到一个并不存在的数据库中去，系统就会自动的帮我们创建这个数据库。但是一个空的数据库系统并不会显示出来，往这个数据库中插入一条记录，我们就可以看到数据库存在了，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824215549511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)

```java
db.sang_collect.insert({x:3})
```

在MongoDB中，我们插入的每一条记录都是一个json字符串，这个json字符串我们称作文档，多个文档可以组成一个集合，这个文档就类似于我们关系型数据库中的一行数据，而集合就类似于关系型数据库中的一张表，**集合也不用专门去创建，直接输入向哪个集合中插入数据即可，此时集合就会被自动的创建出来了**。

当然我们也可以批量的添加文档，如下(批量添加一样也可以使用insert方法来完成)：

```java
db.sang_collect.insertMany([{x:1},{x:2},{x:3}])
```

如果在插入某一个文档时出错，则其后面的文档就会插入失败，而在其之前已经插入的文档则不受影响，如下：

```java
db.sang_collect.insertMany([{_id:99,x:99},{_id:99,x:98},{_id:97,x:97}])
```

由于第二个文档的_id字段与前面的重复，所以第二第三个文档插入失败，第一个文档则插入成功。

这里的 sang_collect 表示的该数据库下的一个集合，这里是对该集合进行的操作

## 四、查找操作

数据添加成功之后我们再来看看查询，利用db.sang.find()方法我们可以查看所有文档(所有记录)，如果只查看一个文档(一条记录)，可以通过db.sang.findOne()命令，在查看之前我先用一个for循环多插入几条数据，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824220716264.png#pic_center)

```java
> for(var i=4;i<100;i++)db.sang_collect.insert({x:i})
```

然后分别调用find和findOne方法查看，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824220732527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmh1YWliZWlhbg==,size_16,color_FFFFFF,t_70#pic_center)

```java
> db.sang_collect.find()
```

查出来的数据除了我们插入的 x 之外，还有一个_id字段，这是系统自动为我们添加的字段，我们也可以自己传入`_id`,但是`_id`字段不能重复，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824220822809.png#pic_center)

```java
> db.sang_collect.insert({_id:1,x:3})
WriteResult({ "nInserted" : 1 })
> db.sang_collect.insert({_id:1,x:3})
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 11000,
                "errmsg" : "E11000 duplicate key error collection: sang.sang_collect index: _id_ dup key: { : 1.0 }"
        }
})
```

find和findOne中也可以传入查询参数，这个我们后面再详细说。

## 五、修改操作

update操作可以用来更新数据，它接收两个参数，第一个参数表示更新条件，第二个参数表示要更新的数据，比如我将所有x:1的数据改为x:999,如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824220854303.png#pic_center)

```java
> db.sang_collect.update({x:99},{x:1000})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```



## 六、删除操作

remove操作可以用来删除数据，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824220921871.png#pic_center)

```java
> db.sang_collect.remove({x:99})
WriteResult({ "nRemoved" : 1 })
> db.sang_collect.find({x:99})
>
```



## 七、shell 其他操作

我们也可以将要执行的脚本放在一个js文件中，在使用shell脚本时指定要执行的js文件，如下：

```java
mongo ~/myjs.js
```

shell会依次执行js中的脚本，并在执行完成后退出。如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824221037515.png#pic_center)
我的js脚本中是两行插入语句，此时我们重新进入到shell中，就可以看到刚刚的数据已经插入成功了。

如果有每次启动都要加载的js文件，我们可以将其内容放在`.mongorc.js`文件中，该文件放在当前用户目录下，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200824221053856.png#pic_center)
这样，每次启动都会打印一个"你好，欢迎使用MongoDB".