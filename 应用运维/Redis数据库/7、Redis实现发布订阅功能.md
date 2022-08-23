[toc]

## 前言

**Redis发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息。**
Redis客户端可以订阅任意数量的频道！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126113040932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)

## 一、实现方式：

### ①命令：

这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126113311555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)

### ②发布订阅的实现：

#### 1、订阅端：

```java
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> SUBSCRIBE dingdada  #订阅名字为 dingdada 的频道
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "dingdada"
3) (integer) 1
#等待推送的信息
1) "message"  #消息
2) "dingdada"  #来自哪个频道的消息
3) "hello world\xef\xbc\x81"  # 消息的具体内容
1) "message"
2) "dingdada"
3) "my name is dyj\x81"
```

#### 2、发送端：

```java
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> PUBLISH dingdada "hello world！"  #发送消息到dingdada 频道
(integer) 1
127.0.0.1:6379> PUBLISH dingdada "my name is dyj"  #发送消息到dingdada 频道
(integer) 1
```

如图所示：
订阅端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126114738275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyOTQ0Mw==,size_16,color_FFFFFF,t_70)
发送端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126114749120.png)

### ③PSUBSCRIBE 命令：订阅指定频道！

```java
PSUBSCRIBE + 频道。。 #订阅给定的模式，可多个
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126132759536.png)

### ④PUBLISH 命令：发送消息至指定频道！

```java
PUBLISH + 频道 +消息  #将信息 message 发送到指定的频道 channel
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126133253627.png)

### ⑤PUNSUBSCRIBE命令：退订！

```java
#指示客户端退订指定模式，若果没有提供模式则退出所有模式。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126133958978.png)

### ⑥SUBSCRIBE：订阅，同上一致。不细讲！

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012613430393.png)

### ⑦UNSUBSCRIBE：退订，同上一致，不细讲！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126134337144.png)

### **⑧总结：**

Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe），在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的**即时聊天**，**群聊**等功能。