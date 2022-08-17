[toc]

## 一、Zookeeper leader选举

2020-11-19 14:38 更新



Leader 选举是保证分布式数据一致性的关键所在。Leader 选举分为 Zookeeper 集群初始化启动时选举和 Zookeeper 集群运行期间 Leader 重新选举两种情况。在讲解 Leader 选举前先了解一下 Zookeeper 节点 4 种可能状态和事务ID概念。

### 1、Zookeeper 节点状态

- LOOKING：寻找 Leader 状态，处于该状态需要进入选举流程
- LEADING：领导者状态，处于该状态的节点说明是角色已经是 Leader
- FOLLOWING：跟随者状态，表示 Leader 已经选举出来，当前节点角色是 follower
- OBSERVER：观察者状态，表明当前节点角色是 observer

### 2、事务ID

ZooKeeper 状态的每次变化都接收一个 ZXID（ZooKeeper 事务 id）形式的标记。ZXID 是一个 64 位的数字，由 Leader 统一分配，全局唯一，不断递增。ZXID 展示了所有的ZooKeeper 的变更顺序。每次变更会有一个唯一的 zxid，如果 zxid1 小于 zxid2 说明 zxid1 在 zxid2 之前发生。

### 3、Zookeeper 集群初始化启动时 Leader 选举

若进行 Leader 选举，则至少需要两台机器，这里选取 3 台机器组成的服务器集群为例。初始化启动期间 Leader 选举流程如下图所示。

![20180128213955030](https://atts.w3cschool.cn/attachments/image/20201119/1605767881800718.png)

在集群初始化阶段，当有一台服务器 ZK1 启动时，其单独无法进行和完成 Leader 选举，当第二台服务器 ZK2 启动时，此时两台机器可以相互通信，每台机器都试图找到 Leader，于是进入 Leader 选举过程。选举过程开始，过程如下：　　

(1) 每个Server发出一个投票。由于是初始情况，ZK1 和 ZK2 都会将自己作为 Leader 服务器来进行投票，每次投票会包含所推举的服务器的 myid 和 ZXID，使用(myid, ZXID)来表示，此时 ZK1 的投票为(1, 0)，ZK2 的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。　　

(2) 接受来自各个服务器的投票。集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自 LOOKING 状态的服务器。　　

(3) 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行比较，规则如下　　　　

- 优先检查 ZXID。ZXID 比较大的服务器优先作为 Leader。
- 如果 ZXID 相同，那么就比较 myid。myid 较大的服务器作为Leader服务器。

  对于 ZK1 而言，它的投票是(1, 0)，接收 ZK2 的投票为(2, 0)，首先会比较两者的 ZXID，均为 0，再比较 myid，此时 ZK2 的 myid 最大，于是 ZK2 胜。ZK1 更新自己的投票为(2, 0)，并将投票重新发送给 ZK2。　　



(4) 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于 ZK1、ZK2 而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出 ZK2 作为Leader。　　

(5) 改变服务器状态。一旦确定了 Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为 FOLLOWING，如果是 Leader，就变更为 LEADING。当新的 Zookeeper 节点 ZK3 启动时，发现已经有 Leader 了，不再选举，直接将直接的状态从 LOOKING 改为 FOLLOWING。

### 4、Zookeeper 集群运行期间 Leader 重新选

在 Zookeeper 运行期间，如果 Leader 节点挂了，那么整个 Zookeeper 集群将暂停对外服务，进入新一轮Leader选举。假设正在运行的有 ZK1、ZK2、ZK3 三台服务器，当前 Leader 是 ZK2，若某一时刻 Leader 挂了，此时便开始 Leader 选举。选举过程如下图所示。

![201802](https://atts.w3cschool.cn/attachments/image/20201119/1605767775894718.png)

 

(1) 变更状态。Leader 挂后，余下的非 Observer 服务器都会讲自己的服务器状态变更为 LOOKING，然后开始进入 Leader 选举过程。　　

(2) 每个Server会发出一个投票。在运行期间，每个服务器上的 ZXID 可能不同，此时假定 ZK1 的 ZXID 为 124，ZK3 的 ZXID 为 123；在第一轮投票中，ZK1 和 ZK3 都会投自己，产生投票(1, 124)，(3, 123)，然后各自将投票发送给集群中所有机器。　　

(3) 接收来自各个服务器的投票。与启动时过程相同。　　

(4) 处理投票。与启动时过程相同，由于 ZK1 事务 ID 大，ZK1 将会成为 Leader。　　

(5) 统计投票。与启动时过程相同。　　

(6) 改变服务器的状态。与启动时过程相同。

leader选举是一个复杂的过程，但 ZooKeeper 服务使它非常简单。让我们在下一章中继续学习 ZooKeepe r 安装，以用于开发目的。