[toc]

## 前言

Redis 是**内存数据库**，如果不将[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了**持久化功能** !

## 一. RDB（Redis DataBase）

### ①首先我们进入服务器找到[dump](https://so.csdn.net/so/search?q=dump&spm=1001.2101.3001.7020).rdb文件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126100842863.png)

### ②测试触发rdb操作：[vim](https://so.csdn.net/so/search?q=vim&spm=1001.2101.3001.7020)打开redis.conf配置文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126101300315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
为了方便测试，我们将其改为 ：

```bash
save 60 5  #意思是在60秒内进行了5次操作，即写入rdb文件中进行持久化保存
1
```

如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126101402476.png)

### ③触发机制：

#### 1、save的规则满足的情况下，会自动触发rdb规则,测试如下：

先手动删除dump.rdb文件，实验触发规则！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126101910784.png)
在Redis中操作5次命令！
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012610201838.png)
查看是否生成dump.rdb文件!
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102109611.png)
成功！

#### 2、执行flushall命令，也会触发rdb规则

再次删除dump.rdb文件！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102154282.png)
执行flushall操作命令！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102240697.png)
正常生成成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102228945.png)

#### 3、退出Redis，也会触发rdb规则

删除：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102322872.png)
退出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102413886.png)
生成成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126102401338.png)

### ④恢复rdb文件

1、只需将备份的rdb文件放在我们的redis启动目录即可，Redis启动的时候会自动检查dump.rdb文件并恢复其中的数据！
2、查找文件位置的命令：

```java
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin"  # 如果在这个目录下存在 dump.rdb 文件，启动就会自动恢复其中的数据
123
```

### ⑤优缺点：

**优点：**
1、适合大规模的数据恢复！
2、对数据的完整性要求不高！
**缺点：**
1、需要一定的时间间隔进程操作！如果redis意外宕机了，这个最后一次修改数据就没有的了！
2、fork进程的时候，会占用一定的内容空间！

⑥总结：
Redis会单独创建（fork）一个**子进程**来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是**不进行任何IO操作**的。
这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置！

**在生产环境我们会将这个文件进行备份！**

## 二. AOF（Append Only File）

### ①Redis默认使用的是RDB模式，所以需要手动开启AOF模式！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126103608150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
十分简单！将no改为yes即可！

重启服务器！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104004148.png)
发现新文件appendonly.aof！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104025417.png)

### ②aof文件内容：

先进行一些添加的操作：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104237643.png)
然后我们可以vim打开appendonly.aof文件看看里面是什么？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104248767.png)
有木有发现，里面存储的就是我们先前操作的命令！

### ③修复aof文件：

#### 1、如果有个**坏蛋** 将我们的aof文件给修改了，加了点乱七八糟的东西，我们该如何修复呢？如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104505332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)

#### 2、重启下Redis看看：发现重启失败！报错**配置信息加载失败**！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104646486.png)

#### 3、我们可以使用redis-check-aof文件来进行修复！

```bash
redis-check-aof --fix appendonly.aof  #修复appendonly.aof文件
1
```

返回修复成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104914673.png)

#### 4、我们再看看aof文件里面的内容！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126104956815.png)

#### 5、细心的同学可能会发现虽然错误的内容少了，但是正确的也有一定的丢失！所以这个修复无法做到百分百修复！但是丢弃一小部分和丢失全部

#### 6、再次重启试试！成功！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126105158830.png)

### ④AOF重写规则！

aof默认的就是文件的无限追加，文件会越来越大！在配置文件中可以设置文件的大小！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126105344293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
解释：

```bash
# appendfsync always # 每次修改都会 sync。消耗性能 
appendfsync everysec # 每秒执行一次 sync，可能会丢失这1s的数据！ # appendfsync no # 不执行 sync，这个时候操作系统自己同步数据，速度最快！

appendfilename "appendonly.aof" # 持久化的文件的名字
appendonly no # 默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下， rdb完全够用！

auto-aof-rewrite-percentage 100  #写入百分比
auto-aof-rewrite-min-size 64mb  #写入的文件最大值是多少，一般在实际工作中我们会将其设置为5gb左右！
12345678
```

### ⑤优缺点!

**优点：**
1、每一次修改都同步，文件的完整性会更加好！
2、每秒同步一次，最多会丢失一秒的数据！
3、从不同步，效率最高的！

**缺点：**
1、相对于数据文件来说，aof远远大于 rdb，修复的速度也比 rdb慢！
2、Aof 运行效率也要比 rdb 慢，所以我们redis默认的配置就是rdb持久化！

⑥**总结**：以下来自伟大的网友总结！
1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化
4、同时开启两种持久化方式

- 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
- RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

5、性能建议

- 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
- 如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
- 如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。