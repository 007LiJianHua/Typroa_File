[toc]

## NATS是什么?

NATS是一个现代化分布式的消息系统。支持*<u>消息发现</u>*，<u>消息发布</u>，`消息问答`等。
NATS支持消息处理微服务部署。

NATS用go语言开发，不需要运行环境，下载对应平台编译好的执行文件可直接使用。

## NATS 功能架构图

此图展示nats提供的4个特性不同的功能。 分别是`pub/sub`、 `reqeust/replay`、`stream`和 `K/V store`。

CoreNATS消息是实时的，支持消息订阅和消息请求回复。
JetStream是Core NATS的上层建筑，支持消息持久化，消息队列功能。保障消息可靠性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a367e0f2934a45aea4d1d9530dc0f6d1.png)

## NATS 各功能特性说明

`NATS-server`的功能由 <u>CoreNATS和JetStream两大部分提供</u>。

> CoreNATS是基础，提供Publish/Subscribe和Request/Replay功能。
> JetStream基于CoreNATS，提供stream和K/V store功能。

下面介结每个功能的特性。

## Publish/Subscribe

Publish/Subscribe功能如同 redis 的 pub/sub功能，支持多订阅者同时获取到消息。如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/57d684123cc54e7fb7bf93fb1b3595d2.png)

比 redis 的 pub/sub功能强大处在于以下几点:

- 订阅者 `支持通配符`，可订阅多个subject消息。
- Publish/Subscribe 支持同名组队列，组内随机某个成员获得消息。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/fb245b78adf74c02bb0066ace0ac3f19.png)
  对于Subject,组队列被认为是一个订阅者， 组队列内某随机成员可获取消息。组队列用来支持业务系统的高可用。

## Request/Replay

Request/Replay 这个功能让我眼前一亮，之前没在哪个消息系统见到这个功能。
此功能NATS作为中间桥梁，助程序A和程序B之间实现通信。
![在这里插入图片描述](https://img-blog.csdnimg.cn/28d457a7a0624ef0a45211501f052809.png)
第一步： replay进程订阅指定key
第二步： request进程向指定key发布消息
第三步： replay进程收到消息后，应答结果
第四步： request进程收到replay进程的应答

Request/Replay在使业务系统不关心上游服务是可用， 解耦请求与回复系统之间的依赖关系。

## Stream

在Pub/Sub下，订阅者服务挂掉或者重启会导致部分消息丢失。stream功能类型于rabbitMQ的消息队列功能。可保证下游消费端不丢失消息。

stream特性有：

- 消费如同存在队列中，是有序的
- 消息可持久化到文件中
- 消息可重复消费
- 新增加的消费者可指定序号或者时间节点消费历史消息
- stream支持push和pull双模式
- stream有ack机制，确保消费者正确处理每一条消息。

## K/V store

K/V store功能，初看名字如redis存储一般，但不完全是。
NATS 的K/V store支持的操作有：

- put: 设置key的value
- get: 获取key的value
- delete: 删除key
- purge: 清除key的value
- create: 如果key不存在，创建key
- update: compare and set (aka compare and swap) 对比更新
- keys: 获取所有key对应的value

NATS 的K/V store 特性如下：

- 支持put, get ，delete 等基础操作
- 支持watch 监听 value的变化
- 支持watch all 监听bucket内所有keys的变化
- history 查看一个key中value的变更历史，此功能是redis不具备的。

## NATS的一些概念说明

### Queue Groups

有些场景消费需要高可用， 因此多个进程组合成为queue groups, 随机一个进程获取到消息。

### Subject Mapping and Traffic Shaping

> Subject mapping is a very powerful feature of the NATS server, useful for canary deployments, A/B testing, chaos testing, and migrating to a new subject namespace.

Subject Mapping and Traffic Shaping是NATS一个重要的功能，可用于金丝雀部署、A/B测试、混沌测试，以及迁移到新的主题名称空间

就是把某个或者某些subject转发到另外的某个或者某些subject中，还可设置转发百分比。 如同nginx upstream把流量打到不同机器可指定百分比一样。



## NATS pub/sub 与 redis pub/sub 和nsq对比

redis pub/sub和 nsq区别不大， nsq只是比redis多了高可用特点外nsq没有一个监听者时消息会保留到队列，只要有一个监听者，消息就不可找回。

因此以下nats具备的优点：

- NATS 本身是分布式服务，不用担心节时是否挂掉
- Queue Groups 允许多个进程作为一个监听者，以支持业务系统的高可用
- Subject Mapping and Traffic Shaping 支持A/B测试, 迭代升级。
- JetStream支持消息持久化，支持消息重复消费，新建监听者可消费历史消息

## NATS stream 与 rabbitMQ 对比

- rabbitMq消息存多份， stream消息存一份，节省空间
- rabbitMq后挂在exchange的队列只能接收最新消息， stream新建的消费者可消费历史消息
- stream 支持Queue Groups

## 官方 消息服务对比说明

https://docs.nats.io/nats-concepts/overview/compare-nats

# 支持哪些语言的客户端

- Golang
​ - Java
​ - JavaScript
​ - Python
​ - Ruby
​​ - C
​ - Rust
​ - Elixir
​ - Zig
​ - SwiftyNats
​ - Kotlin
​ - Dart and Dart
​ - Tcl
​ - Crystal
​ - PHP and PHP
​ - and many more…