[toc]

## 一、概述

- Redis是什么？what？
  Redis（Remote Dictionary Server )，即远程字典服务 !
  是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118103556927.png)
  redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
  免费和开源！是当下最热门的 NoSQL 技术之一！也被人们称之为结构化数据库！
- Redis能干嘛？
  1、内存存储、持久化，内存中是断电即失、所以说持久化很重要（rdb、aof） 2、效率高，可以用于高速缓存
  3、发布订阅系统
  4、地图信息分析
  5、计时器、计数器（浏览量！）
  6、…

## 二、Redis的基本了解

需要**安装教程**请参考本人的另一篇博客文章：[Widows和Linux下如何安装Redis](https://blog.csdn.net/weixin_43829443/article/details/112638809).

1. 首先我们可以看下官方文档是如何介绍Redis的：
   ①、英文文档 [点击跳转](https://redis.io/).
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118104750172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
   ②、中文文档 [点击跳转](http://www.redis.cn/).
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118104900750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
2. Redis-Key
   简单介绍一下Redis中队Key的操作命令。希望大家可以跟着注释敲一遍，简单记一下，都是最常用的命令！

```c
127.0.0.1:6379> ping  #查看当前连接是否正常，正常返回PONG
PONG
127.0.0.1:6379> clear  #清楚当前控制台（为了更好的看到下面输入的命令）
127.0.0.1:6379> keys *  #查看当前库里所有的key
1) "db"
127.0.0.1:6379> FLUSHALL  #清空所有库的内容
OK
127.0.0.1:6379> keys * 
(empty array)
127.0.0.1:6379> set name dingdada  #添加一个key为‘name’ value为‘dingdada’的数据
OK
127.0.0.1:6379> get name  #查询key为‘name’的value值
"dingdada"
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set name1 dingdada2
OK
127.0.0.1:6379> get name1
"dingdada2"
127.0.0.1:6379> keys *  #查看当前库里所有的key
1) "name1"
2) "name"
127.0.0.1:6379> EXISTS name  #判断当前key是否存在
(integer) 1
127.0.0.1:6379> move name 1  #移除当前库1的key为‘name‘的数据
(integer) 1
127.0.0.1:6379> keys *
1) "name1"
127.0.0.1:6379> FLUSHALL  #再次清空所有库的内容
OK

## 多加几条数据 下面测试设置key的过期时间
127.0.0.1:6379> set name dingdada
OK
127.0.0.1:6379> set name1 dingdada1
OK
127.0.0.1:6379> set name2 dingdada2
OK
127.0.0.1:6379> EXPIRE name 15  #设置key为’name‘的数据过期时间为15秒 单位seconds
(integer) 1
127.0.0.1:6379> ttl name  #查看当前key为’name‘的剩余生命周期时间
(integer) 13
127.0.0.1:6379> ttl name
(integer) 12
127.0.0.1:6379> ttl name
(integer) 11
127.0.0.1:6379> ttl name
(integer) 8
127.0.0.1:6379> ttl name
(integer) 6
127.0.0.1:6379> ttl name
(integer) 3
127.0.0.1:6379> ttl name
(integer) 2
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) 0
127.0.0.1:6379> ttl name  #如若返回-2，证明key已过期
(integer) -2
127.0.0.1:6379> get name    #再次查询即为空
(nil)
127.0.0.1:6379> type name1
string
127.0.0.1:6379> 
```

***如若遇到不会的命令！记得查看Redis中文官网，上面有官方文档！链接上面有，可以点击跳转~***

## 二、Redis的五[大数据](https://so.csdn.net/so/search?q=大数据&spm=1001.2101.3001.7020)类型

### 1、**String**（字符串）

#### ①添加、查询、追加、获取长度，判断是否存在的操作

```c
127.0.0.1:6379> set name dingdada  #插入一个key为‘name’值为‘dingdada’的数据
OK
127.0.0.1:6379> get name  #获取key为‘name’的数据
"dingdada"
127.0.0.1:6379> get key1
"hello world!"
127.0.0.1:6379> keys *  #查看当前库的所有数据
1) "name"
127.0.0.1:6379> EXISTS name  #判断key为‘name’的数据存在不存在，存在返回1
(integer) 1
127.0.0.1:6379> EXISTS name1  #不存在返回0
(integer) 0
127.0.0.1:6379> APPEND name1 dingdada1  #追加到key为‘name’的数据后拼接值为‘dingdada1’，如果key存在类似于java中字符串‘+’，不存在则新增一个，类似于Redis中的set name1 dingdada1 ，并且返回该数据的总长度
(integer) 9
127.0.0.1:6379> get name1
"dingdada1"
127.0.0.1:6379> STRLEN name1  #查看key为‘name1’的字符串长度
(integer) 9
127.0.0.1:6379> APPEND name1 ,dingdada2  #追加，key存在的话，拼接‘+’，返回总长度
(integer) 19
127.0.0.1:6379> STRLEN name1
(integer) 19
127.0.0.1:6379> get name1
"dingdada1,dingdada2"
127.0.0.1:6379> set key1 "hello world!"  #注意点：插入的数据中如果有空格的数据，请用“”双引号，否则会报错！
OK
127.0.0.1:6379> set key1 hello world!  #报错，因为在Redis中空格就是分隔符，相当于该参数已结束
(error) ERR syntax error
127.0.0.1:6379> set key1 hello,world!  #逗号是可以的
OK
```

#### ②自增、自减操作

```c
127.0.0.1:6379> set num 0  #插入一个初始值为0的数据
OK
127.0.0.1:6379> get num
"0"
127.0.0.1:6379> incr num  #指定key为‘num’的数据自增1，返回结果  相当于java中 i++
(integer) 1
127.0.0.1:6379> get num  #一般用来做文章浏览量、点赞数、收藏数等功能
"1"
127.0.0.1:6379> incr num
(integer) 2
127.0.0.1:6379> incr num
(integer) 3
127.0.0.1:6379> get num
"3"
127.0.0.1:6379> decr num  #指定key为‘num’的数据自减1，返回结果  相当于java中 i--
(integer) 2
127.0.0.1:6379> decr num
(integer) 1
127.0.0.1:6379> decr num
(integer) 0
127.0.0.1:6379> decr num  #可以一直减为负数~
(integer) -1
127.0.0.1:6379> decr num  #一般用来做文章取消点赞、取消收藏等功能
(integer) -2
127.0.0.1:6379> decr num
(integer) -3
127.0.0.1:6379> INCRBY num 10  #后面跟上by  指定key为‘num’的数据自增‘参数（10）’，返回结果
(integer) 7
127.0.0.1:6379> INCRBY num 10
(integer) 17
127.0.0.1:6379> DECRBY num 3  #后面跟上by  指定key为‘num’的数据自减‘参数（3）’，返回结果
(integer) 14
127.0.0.1:6379> DECRBY num 3
(integer) 11
```

#### ③截取、替换字符串操作

```c
#截取
127.0.0.1:6379> set key1 "hello world!"
OK
127.0.0.1:6379> get key1
"hello world!"
127.0.0.1:6379> GETRANGE key1 0 4  #截取字符串，相当于java中的subString，下标从0开始，不会改变原有数据
"hello"
127.0.0.1:6379> get key1
"hello world!"
127.0.0.1:6379> GETRANGE key1 0 -1  #0至-1相当于 get key1，效果一致，获取整条数据
"hello world!"
#替换
127.0.0.1:6379> set key2 "hello,,,world!"
OK
127.0.0.1:6379> get key2
"hello,,,world!"
127.0.0.1:6379> SETRANGE key2 5 888  #此语句跟java中replace有点类似，下标也是从0开始，但是有区别：java中是指定替换字符，Redis中是从指定位置开始替换，替换的数据根据你所需替换的长度一致，返回值是替换后的长度
(integer) 14
127.0.0.1:6379> get key2
"hello888world!"
127.0.0.1:6379> SETRANGE key2 5 67  #该处只替换了两位
(integer) 14
127.0.0.1:6379> get key2
"hello678world!"
```

#### ④设置过期时间、不存在设置操作

```c
#设置过期时间，跟Expire的区别是前者设置已存在的key的过期时间，而setex是在创建的时候设置过期时间
127.0.0.1:6379> setex name1 15  dingdada  #新建一个key为‘name1’，值为‘dingdada’，过期时间为15秒的字符串数据
OK
127.0.0.1:6379> ttl name1  #查看key为‘name1’的key的过期时间
(integer) 6
127.0.0.1:6379> ttl name1
(integer) 5
127.0.0.1:6379> ttl name1
(integer) 3
127.0.0.1:6379> ttl name1
(integer) 1
127.0.0.1:6379> ttl name1
(integer) 0
127.0.0.1:6379> ttl name1  #返回为-2时证明该key已过期，即不存在
(integer) -2
#不存在设置
127.0.0.1:6379> setnx name2 dingdada2  #如果key为‘name2’不存在，新增数据，返回值1证明成功
(integer) 1
127.0.0.1:6379> get name2
"dingdada2"
127.0.0.1:6379> keys *
1) "name2"
127.0.0.1:6379> setnx name2 "dingdada3"  #如果key为‘name2’的已存在，设置失败，返回值0，也就是说这个跟set的区别是：set会替换原有的值，而setnx不会，存在即不设置，确保了数据误操作~
(integer) 0
127.0.0.1:6379> get name2
"dingdada2"
```

#### ⑤mset、mget操作

```c
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3  #插入多条数据
OK
127.0.0.1:6379> keys *  #查询所有数据
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> mget k1 k2 k3  #查询key为‘k1’，‘k2’，‘k3’的数据
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> MSETNX k1 v1 k4 v4  #msetnx是一个原子性的操作，在一定程度上保证了事务！要么都成功，要么都失败！相当于if中的条件&&（与）
(integer) 0
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> MSETNX k5 v5 k4 v4  #全部成功
(integer) 1
127.0.0.1:6379> keys *
1) "k2"
2) "k4"
3) "k3"
4) "k5"
5) "k1"
```

#### ⑥添加获取对象、getset操作

```c
#这里其实本质上还是字符串，但是我们讲其key巧妙的设计了一下。
##mset student:1:name  student 相当于类名，1 相当于id，name 相当于属性
#如果所需数据全部这样设计，那么我们在java的业务代码中，就不需要关注太多的key
#只需要找到student类，下面哪个id，需要哪个属性即可，减少了代码的繁琐，在一定程度上可以理解为这个一个类的对象！
127.0.0.1:6379> mset student:1:name dingdada student:1:age 22  #新增一个key为‘student:1:name’，value为‘dingdada ’。。等数据
OK
127.0.0.1:6379> keys *  #查看所有的key
1) "student:1:age"
2) "student:1:name"
127.0.0.1:6379> mget student:1:age student:1:name  #获取数据
1) "22"
2) "dingdada"

##getset操作
127.0.0.1:6379> getset name1 dingdada1  #先get再set，先获取key，如果没有，set值进去，返回的是get的值
(nil)
127.0.0.1:6379> get name1
"dingdada1"
127.0.0.1:6379> getset name1 dingdada2  ##先获取key，如果有，set（替换）最新的值进去，返回的是get的值
"dingdada1"
127.0.0.1:6379> get name1  #替换成功
"dingdada2"
```

#### ⑦总结

*String是Redis中最常用的一种数据类型，也是Redis中最简单的一种数据类型。首先，表面上它是字符串，但其实他可以灵活的表示字符串、整数、浮点数3种值。Redis会自动的识别这3种值。*

***路漫漫其修远兮，吾必将上下求索~***
因为篇幅所限，另开一篇文章写剩下的内容哦!如果你认为博主写的不错！写作不易，请点赞、关注、评论给博主一个鼓励吧**转载请注明出处哦**

### 2、**List**（列表）

#### ①lpush（左插入）、lrange（查询集合）、rpush（右插入）操作

```c
#lpush
127.0.0.1:6379> lpush list v1  #新增一个列表
(integer) 1
127.0.0.1:6379> lpush list v2
(integer) 2
127.0.0.1:6379> lpush list v3
(integer) 3
#lrange
127.0.0.1:6379> LRANGE list 0 -1  #查询list的所有元素值
1) "v3"
2) "v2"
3) "v1"
127.0.0.1:6379> lpush list1 v1 v2 v3 v4 v5  #批量添加列表元素
(integer) 5
127.0.0.1:6379> LRANGE list1 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
5) "v1"
###这里大家有没有注意到，先进去的会到后面，也就是我们的lpush的意思是左插入，l--left
#rpush
127.0.0.1:6379> LRANGE list 0 1  #指定查询列表中的元素，从下标零开始，1结束，两个元素
1) "v3"
2) "v2"
127.0.0.1:6379> LRANGE list 0 0  #指定查询列表中的唯一元素
1) "v3"
127.0.0.1:6379> rpush list rv0  #右插入，跟lpush相反，这里添加进去元素是在尾部！
(integer) 4
127.0.0.1:6379> lrange list 0 -1  #查看集合所有元素
1) "v3"
2) "v2"
3) "v1"
4) "rv0"
##联想：这里我们是不是可以做一个，保存的记录值（如：账号密码的记录），
每次都使用lpush，老的数据永远在后面，我们每次获取 0 0 位置的元素，是不是相当于更新了数据操作，但是数据记录还在？想要查询记录即可获取集合所有元素！
```

#### ②lpop（左移除）、rpop（右移除）操作

```c
#lpop
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
5) "v1"
127.0.0.1:6379> lpop list  #从头部开始移除第一个元素
"v5"
##################
#rpop
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
127.0.0.1:6379> rpop list
"v1"
127.0.0.1:6379> LRANGE list 0 -1  #从尾部开始移除第一个元素
1) "v4"
2) "v3"
3) "v2"
```

#### ③lindex（查询指定下标元素）、llen（获取集合长度） 操作

```c
#lindex
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
2) "v3"
3) "v2"
127.0.0.1:6379> lindex list 1  #获取指定下标位置集合的元素，下标从0开始计数
"v3"
127.0.0.1:6379> lindex list 0  #相当于java中的indexof
"v4"
#llen
127.0.0.1:6379> llen list  #获取指定集合的元素长度，相当于java中的length或者size
(integer) 3
```

#### ④lrem（根据value移除指定的值）

```c
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
2) "v3"
3) "v2"
127.0.0.1:6379> lrem list 1 v2  #移除集合list中的元素是v2的元素1个
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
2) "v3"
127.0.0.1:6379> lrem list 0 v3 #移除集合list中的元素是v2的元素1个,这里的0和1效果是一致的
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
127.0.0.1:6379> lpush list  v3 v2 v2 v2
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "v2"
2) "v2"
3) "v2"
4) "v3"
5) "v4"
127.0.0.1:6379> lrem list 3 v2  #移除集合list中元素为v2 的‘3’个，这里的参数数量，如果实际中集合元素数量不达标，不会报错，全部移除后返回成功移除后的数量值
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "v3"
2) "v4"
```

#### ⑥ltrim（截取元素）、rpoplpush（移除指定集合中最后一个元素到一个新的集合中）操作

```c
#ltrim
127.0.0.1:6379> lpush list v1 v2 v3 v4
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
127.0.0.1:6379> ltrim list 1 2  #通过下标截取指定的长度，这个list已经被改变了，只剩下我们所指定截取后的元素
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "v3"
2) "v2"
################
#rpoplpush
127.0.0.1:6379> lpush list v1 v2 v3 v4 v5
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
5) "v1"
127.0.0.1:6379> rpoplpush list newlist  #移除list集合中的最后一个元素到新的集合newlist中，返回值是移除的最后一个元素值
"v1"
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
127.0.0.1:6379> LRANGE newlist 0 -1  #确实存在该newlist集合并且有刚刚移除的元素，证明成功
1) "v1"
```

#### ⑦lset（更新）、linsert操作

```c
#lset
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
127.0.0.1:6379> 
127.0.0.1:6379> lset list 1 newV5  #更新list集合中下标为‘1’的元素为‘newV5’
OK
127.0.0.1:6379> LRANGE list 0 -1  #查看证明更新成功
1) "v5"
2) "newV5"
3) "v3"
4) "v2"
##注意点：
127.0.0.1:6379> lset list1 0 vvvv  #如果指定的‘集合’不存在，报错
(error) ERR no such key
127.0.0.1:6379> lset list 8 vvv  #如果集合存在，但是指定的‘下标’不存在，报错
(error) ERR index out of range
########################
#linsert
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "newV5"
3) "v3"
4) "v2"
127.0.0.1:6379> LINSERT list after v3 insertv3  #在集合中的‘v3’元素 ‘(after)之后’ 加上一个元素
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "newV5"
3) "v3"
4) "insertv3"
5) "v2"
127.0.0.1:6379> LINSERT list before v3 insertv3  #在集合中的‘v3’元素 ‘(before)之前’ 加上一个元素
(integer) 6
127.0.0.1:6379> LRANGE list 0 -1
1) "v5"
2) "newV5"
3) "insertv3"
4) "v3"
5) "insertv3"
6) "v2"
```

#### ⑧小结：

- 实际上是一个链表，before Node after ， left，right 都可以插入值
- 如果key 不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表，也代表不存在！
- 在两边插入或者改动值，效率最高！ 中间元素，相对来说效率会低一点~
- 消息排队！消息队列 （Lpush Rpop）， 栈（ Lpush Lpop）！

### 3、**Set**（集合）元素唯一不重复

#### ①sadd（添加）、smembers（查看所有元素）、sismember（判断是否存在）、scard（查看长度）、srem（移除指定元素）操作

```c
#set中所有的元素都是唯一的不重复的！
127.0.0.1:6379> sadd set1 ding da mian tiao  #添加set集合（可批量可单个，写法一致，不再赘述）
(integer) 4
127.0.0.1:6379> SMEMBERS set1  #查看set中所有元素
1) "mian"
2) "da"
3) "tiao"
4) "ding"
127.0.0.1:6379> SISMEMBER set1 da  #判断某个值在不在set中，在返回1
(integer) 1
127.0.0.1:6379> SISMEMBER set1 da1  #不在返回0，在返回1
(integer) 0
127.0.0.1:6379> SCARD set1  #查看集合的长度，相当于size、length
(integer) 4
127.0.0.1:6379> srem set1 da  #移除set中指定的元素
(integer) 1
127.0.0.1:6379> SMEMBERS set1  #移除成功
1) "mian"
2) "tiao"
3) "ding"
```

#### ②srandmember（抽随机）操作

```c
127.0.0.1:6379> sadd myset 1 2 3 4 5 6 7  #在set中添加7个元素
(integer) 7
127.0.0.1:6379> SMEMBERS myset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
127.0.0.1:6379> SRANDMEMBER myset 1  #随机抽取myset中1个元素返回
1) "4"
127.0.0.1:6379> SRANDMEMBER myset 1  #随机抽取myset中1个元素返回
1) "1"
127.0.0.1:6379> SRANDMEMBER myset 1  #随机抽取myset中1个元素返回
1) "5"
127.0.0.1:6379> SRANDMEMBER myset  #不填后参数，默认抽1个值，但是下面返回不会带序号值
"3"
127.0.0.1:6379> SRANDMEMBER myset 3  #随机抽取myset中3个元素返回
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> SRANDMEMBER myset 3  #随机抽取myset中3个元素返回
1) "6"
2) "3"
3) "5"
```

#### ③spop（随机删除元素）、smove（移动指定元素到新的集合中）操作

```c
127.0.0.1:6379> SMEMBERS myset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
127.0.0.1:6379> spop myset  #随机删除1个元素，不指定参数值即删除1个
"2"
127.0.0.1:6379> spop myset 1  #随机删除1个元素
1) "7"
127.0.0.1:6379> spop myset 2  #随机删除2个元素
1) "3"
2) "5"
127.0.0.1:6379> SMEMBERS myset  #查询删除后的结果
1) "1"
2) "4"
3) "6"
127.0.0.1:6379> smove myset myset2 1  #移动指定set中的指定元素到新的set中
(integer) 1
127.0.0.1:6379> SMEMBERS myset  #查询原来的set集合
1) "4"
2) "6"
127.0.0.1:6379> SMEMBERS myset2  #查询新的set集合，如果新的set存在，即往后加，如果不存在，则自动创建set并且加入进去
1) "1"
```

#### ④sdiff（差集）、sinter（交集）、sunion（并集）操作

```c
127.0.0.1:6379> sadd myset1 1 2 3 4 5
(integer) 5
127.0.0.1:6379> sadd myset2 3 4 5 6 7
(integer) 5
127.0.0.1:6379> SMEMBERS myset1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6379> SMEMBERS myset2
1) "3"
2) "4"
3) "5"
4) "6"
5) "7"
127.0.0.1:6379> SDIFF myset1 myset2  #查询指定的set之间的差集，可以是多个set
1) "1"
2) "2"
127.0.0.1:6379> SINTER myset1 myset2  #查询指定的set之间的交集，可以是多个set
1) "3"
2) "4"
3) "5"
127.0.0.1:6379> sunion myset1 myset2  #查询指定的set之间的并集，可以是多个set
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
```

#### ⑤总结：可实现共同好友、共同关注等需求。

### 4、**Hash**（哈希）

#### ①hset（添加hash）、hget（查询）、hgetall（查询所有）、hdel（删除hash中指定的值）、hlen（获取hash的长度）、hexists（判断key是否存在）操作

```c
127.0.0.1:6379> hset myhash name dingdada age 23  #添加hash，可多个
(integer) 2
127.0.0.1:6379> hget myhash name  #获取hash中key是name的值
"dingdada"
127.0.0.1:6379> hget myhash age  #获取hash中key是age的值
"23"
127.0.0.1:6379> hgetall myhash  #获取hash中所有的值，包含key
1) "name"
2) "dingdada"
3) "age"
4) "23"
127.0.0.1:6379> hset myhash del test  #添加
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "name"
2) "dingdada"
3) "age"
4) "23"
5) "del"
6) "test"
127.0.0.1:6379> hdel myhash del age  #删除指定hash中的key（可多个），key删除后对应的value也会被删除
(integer) 2
127.0.0.1:6379> hgetall myhash
1) "name"
2) "dingdada"
127.0.0.1:6379> hlen myhash  #获取指定hash的长度，相当于length、size
(integer) 1
127.0.0.1:6379> HEXISTS myhash name  #判断key是否存在于指定的hash，存在返回1
(integer) 1
127.0.0.1:6379> HEXISTS myhash age  #判断key是否存在于指定的hash，不存在返回0
(integer) 0
```

#### ②hkeys（获取所有key）、hvals（获取所有value）、hincrby（给值加增量）、hsetnx（存在不添加）操作

```c
127.0.0.1:6379> hset myhash age 23 high 173
(integer) 2
127.0.0.1:6379> hgetall myhash
1) "name"
2) "dingdada"
3) "age"
4) "23"
5) "high"
6) "173"
127.0.0.1:6379> hkeys myhash  #获取指定hash中的所有key
1) "name"
2) "age"
3) "high"
127.0.0.1:6379> hvals myhash   #获取指定hash中的所有value
1) "dingdada"
2) "23"
3) "173"
127.0.0.1:6379> hincrby myhash age 2  #让hash中age的value指定+2(自增)
(integer) 25
127.0.0.1:6379> hincrby myhash age -1  #让hash中age的value指定-1(自减)
(integer) 24
127.0.0.1:6379> hsetnx myhash nokey novalue  #添加不存在就新增返回新增成功的数量（只能单个增加哦）
(integer) 1 
127.0.0.1:6379> hsetnx myhash name miaotiao  #添加存在则失败返回0
(integer) 0
127.0.0.1:6379> hgetall myhash
1) "name"
2) "dingdada"
3) "age"
4) "24"
5) "high"
6) "173"
7) "nokey"
8) "novalue"
```

#### ③总结：比String更加适合存对象~

### 5、**zSet**（有序集合）

#### ①zadd（添加）、zrange（查询）、zrangebyscore（排序小-大）、zrevrange（排序大-小）、zrangebyscore withscores（查询所有值包含key）操作

```c
127.0.0.1:6379> zadd myzset 1 one 2 two 3 three  #添加zset值，可多个
(integer) 3
127.0.0.1:6379> ZRANGE myzset 0 -1  #查询所有的值
1) "one"
2) "two"
3) "three"
#-inf 负无穷  +inf 正无穷
127.0.0.1:6379> ZRANGEBYSCORE myzset -inf +inf  #将zset的值根据key来从小到大排序并输出
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> ZRANGEBYSCORE myzset 0 1  #只查询key<=1的值并且排序从小到大
1) "one"
127.0.0.1:6379> ZREVRANGE myzset 1 -1  #从大到小排序输出
1) "two"
2) "one"
127.0.0.1:6379> ZRANGEBYSCORE myzset -inf +inf withscores  #查询指定zset的所有值，包含序号的值
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
```

#### ②zrem（移除元素）、zcard（查看元素个数）、zcount（查询指定区间内的元素个数）操作

```c
127.0.0.1:6379> zadd myset 1 v1 2 v2 3 v3 4 v4
(integer) 4
127.0.0.1:6379> ZRANGE myset 0 -1
1) "v1"
2) "v2"
3) "v3"
4) "v4"
127.0.0.1:6379> zrem myset v3  #移除指定的元素，可多个
(integer) 1
127.0.0.1:6379> ZRANGE myset 0 -1
1) "v1"
2) "v2"
3) "v4"
127.0.0.1:6379> zcard myset  #查看zset的元素个数，相当于长度，size。
(integer) 3
127.0.0.1:6379> zcount myset 0 100  #查询指定区间内的元素个数
(integer) 3
127.0.0.1:6379> zcount myset 0 2  #查询指定区间内的元素个数
(integer) 2
```

#### ③总结：成绩表排序，工资表排序，年龄排序等需求可以用zset来实现！

***路漫漫其修远兮，吾必将上下求索~***
到此关于Redis的五大类型的讲解就算告一段落了，如果你认为i博主写的不错！写作不易，请点赞、关注、评论给博主一个鼓励吧**转载请注明出处哦**