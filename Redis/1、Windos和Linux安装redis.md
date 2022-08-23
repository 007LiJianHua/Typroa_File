[toc]

###  一.Widows下如何安装Redis?

* (1) .下载地址: [点击跳转](https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100).
  如下图:
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210114212314327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
* (2) .下载成功后将其移动到我们想要安装的目录下并且解压:
  如下图:
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011510312788.png)
* (3) .进入后有以下内容文件：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115103245463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
* (4) .先点击redis-server.exe启动Redis服务，显示如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115103336859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(5) .再点击redis-cli.exel连接Redis，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011510354034.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)

(6) .至此，在Windows下的安装就结束了。不过Redis官方文档不建议我们在Widows环境下搭建Redis服务。
最好是Linux环境下搭建并使用Redis服务！

### 二.Linux下如何安装Redis?

**前提条件**：我这边安装的Linux系统是CentOS7.7版本，并且是云服务器，使用的远程工具是Xshell 6。
*(1) .首先下载安装包，下载地址：[点击跳转](https://redis.io/).
如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115104532646.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(2) .一般我们都是将这些服务安装在指定文件夹下，在/opt目录下新建soft文件夹，再在/soft文件下建我们需要安装的服务名称文件夹，redis：如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115110715908.png)
*(3) .将包通过rz命令上传到远程服务器上的redis文件夹下：如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115110831490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115110841618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(4) .等待上传成功，速度一般根据你的服务器带宽而定：然后解压redis安装包：
命令：`tar -zxvf 需要解压的包名`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115111055757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(5) .解压后，进入文件夹，正常情况下解压后有19个文件，如若不对，请删除，重新解压：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115111435466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(6) .接下来我们安装基本环境gcc,安装命令：`yum install gcc-c++`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115111614361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115111640822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
等待其安装完成，输入`gcc -v`查看当前gcc的版本：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115111748343.png)
*(7) .这里是**重点**：我们安装的Redis版本是6.0.10.对gcc环境的版本有一定的要求，所以我们要升级gcc的版本，输入以下命令：

```bash
#第一步
sudo yum install centos-release-scl
#第二步
sudo yum install devtoolset-7-gcc*
#第三步
    scl enable devtoolset-7 bash
```

中途遇到输入时不用管太多，直接yes即可。如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115112309347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115112337483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
全部执行完毕后，再次查看版本变为：7.3.1即代表环境升级成功。

*(8) .然后在我们解压的redis目录下执行命令：`make`，，出现以下显示即成功make下载！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115112701406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
再执行命令：`make install`,安装文件！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115112837706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(9) .大家应该会发现我们并没有指定安装的路径，但是Redis有一个默认的路径，得记住：

```bash
/usr/local/bin
```

进入该文件夹内，可以发现，有两个文件和我们再Windows下的文件一样，分别时启动和连接。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115113248933.png)
*(10) .我们将配置文件拷贝过来，以后就用这个配置文件来启动Redis服务。
命令： `cp /opt/soft/redis/redis-6.0.10/redis.conf mtconfig/`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115113519596.png)
*(11) .redis默认不是后台启动的，修改配置文件！

```bash
#打开配置文件
vim redis.conf
#保存
wq!
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115113835457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(12) 接下来就是.启动和连接Redis服务！

```bash
#启动redis服务
redis-server mtconfig/redis.conf
#连接redis服务
redis-cli -p 6379
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115114124653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
*(13).测试redis服务：
先测试set和get基本操作：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115114407982.png)
再查看redis服务的进程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210115114453196.png)

**大功告成！**
自此，关于Redis的安装就全部结束了！欢迎观看转载请注明出处