

[toc]

## 一、ceph介绍

* ceph是一个统一的分布式文件存储系统，设计的初衷是提供较好的性能、可靠性和扩展性。
* 目前已经得到众多厂商的支持并广泛应用，RedHat及OpenStack都可与Ceph整合以支持虚拟机镜像的后端存储。

## 二、ceph特点

* 高性能，
  * 采用==CRUSH算法==实现数据均衡分布
  * 支持上千存储节点，支持TB到PB级别的数据
* 高可用性
* 高扩展性
* 特性丰富
  * 同时**支持三种存储接口：块存储，文件系统存储，对象存储**
    * 块存储：共享的是数据块，在客户端进行格式化、挂载使用
    * 文件系统存储：共享的是文件系统，客户端直接挂载使用
    * 对象存储：通过使用编程语言来进行通信
  * 支持多种语言驱动

![](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/B7FBBE6C3353447094C77507F62FB774/19722)

> `CEPH FS:	`基于NFS/CIFS协议提供ceph文件系统的存储
>
> `RBD:`提供块存储
>
> `RADOSGW:`提供对象存储，
>
> `LIBRADOS:`提供支持的各种库文件，和编程语言
>
> `RADOS:`整个CEPH集群最重要的系统，实现数据的分配、管理各个主从切换等操作

## 三、ceph的核心概念

![](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/C344D8C6707C480C8B6BD8EB98286B13/22003)

> `Rados：`可靠的，自动化的，分布式对象存储文件系统，RADOS是ceph存储的精华，用户实现数据分配，Failover（故障转移）等集群操作
>
> `Librados:`Rados的提供库，上边的RBD、CephFS、RGW、通过librados来访问Rados，目前提供PHP、Ruby、Java、Python、C和C++支持
>
> `Crush:`Crush算法是ceph的两大创新之一，通过crush的==寻址操作==，ceph得以摒弃了传统的集中式存储元数据寻址方案，而Crush算法在一致性哈希基础上很好的考虑了容灾域的隔离，使得==Ceph能够实现各类负载的副本放置规则==，例如跨机房、机架感知等。同时，Crush算法有相当强大的==扩展性==，理论上可以支持数千个存储节点，这为Ceph在大规模云环境中的应用提供了先天的便利
>
> `Pool:`Pool是存储对象的逻辑分区，它规定了数据冗余的类型和对应的副本分布策略，支持两种类型：副本（replicated）和 纠删码（ Erasure Code）
>
> `PG:`PG是一个放置策略组，它是对象的集合，该集合里的所有对象都具有相同的放置策略，简单点说就是相同PG内的对象都会放到相同的硬盘上，PG是 ceph的逻辑概念，服务端数据均衡和恢复的最小粒度就是PG，一个PG包含多个OSD。引入PG这一层其实是为了更好的分配数据和定位数据
>
> `Object:`对象存储，最底层的存储单元，包含元数据与原始数据
>
> --------------------------------------------------------------------==以下为核心组件：==----------------------------------------------------
>
> `OSD:`OSD是负责物理存储的**进程**，**一般配置成和磁盘一一对应**，一块磁盘启动一个OSD进程。主要功能是**存储数据、复制数据、平衡数据、恢复数据**，以及与其它OSD间进行**心跳检查**，负责响应客户端请求**返回具体数据的进程**等
>
> ==Pool、PG、OSD的关系如下：==
>
> 一个Pool里有很多PG；
> 一个PG里包含一堆对象，一个对象只能属于一个PG；
> PG有主从之分，一个PG分布在不同的OSD上（针对三副本类型）
>
> `Monitor:`一个Ceph集群需要多个Monitor组成的小集群，它们通过**Paxos**同步数据，用来**保存OSD的元数据**。负责监视整个**Ceph集群运行的Map视图**（如OSD Map、Monitor Map、PG Map和CRUSH Map），维护集群的健康状态，维护展示集群状态的各种图表，管理**集群客户端认证与授权**；
>
> ​					Paxos协议：最开始，几个mon启动起来，并且建立连接之后，会开始竞选leader，其实paxos协议是允许多个leader，但在系统正常的情况下，没有必要有多个leader，多个leader各自运行自己的流程，会增加整个系统的负载。而竞选的主要依据就是每个Mon都有一个全局唯一的数字rank，所以这个数字中肯定存在最小的，竞选的流程类似于paxos协商值时候的prepare阶段，rank最小的Mon会竞选成功成为leader，不成功的就是peon。
>
> `MDS:`MDS是CephFS服务依赖的元数据服务。负责保存文件系统的元数据，管理目录结构。对象存储和块设备存储不需要元数据服务
>
> `MGR:`主要目标实现 ceph 集群的管理，减小Monitor节点的压力，为外界提供统一的入口。例如cephmetrics、zabbix、calamari、promethus
>
> `RGW:`是Ceph对外提供的对象存储服务，接口与S3和Swift兼容
>
> `Admin:`Ceph常用管理接口通常都是命令行工具，如rados、ceph、rbd等命令，另外Ceph还有可以有一个专用的管理节点，在此节点上面部署专用的管理工具来实现近乎集群的一些管理工作，如集群部署，集群组件管理等



## 四、ceph的三种存储类型

| 存储类型 | 优点                                     | 缺点                                        | 使用场景                         |
| -------- | ---------------------------------------- | ------------------------------------------- | -------------------------------- |
| 块存储   | 通过Raid与LVM等手段，对数据提供了保护    | 采用SAN架构组网时，光纤交换机，造价成本高； | docker容器、虚拟机磁盘存储分配； |
|          | 多块廉价的硬盘组合起来，提高容量         | 主机之间无法共享数据；                      | 日志存储；                       |
|          | 多块磁盘组合出来的逻辑盘，提升读写效率； |                                             | 文件存储；                       |
| 文件存储 | 造价低，随便一台机器就可以了；           | 读写速率低；                                | 日志存储；FTP、NFS；             |
|          | 方便文件共享；                           | 传输速率慢；                                | 其它有目录结构的文件存储         |
| 对象存储 | 具备块存储的读写高速；                   |                                             | 图片存储；                       |
|          | 具备文件存储的共享等特性；               |                                             | 视频存储；                       |

## 五、ceph的IO流程

![](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/53BA3C35E31249B39A5E074B3ABC8B5F/19769)

### 1、正常IO流程如下

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/36789CC0FB3A4DC9874573A67612DEDA/19765](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/36789CC0FB3A4DC9874573A67612DEDA/19765)

> * client创建cluster handler
> * client读取配置文件，
> * client连接Monitor，获取集群信息，
> * client读写IO，根据crush算法请求对应的OSD数据节点，
> * 主OSD数据节点为两个副本写入相同的信息，
> * 等待两个副节点以及主节点写完数据，
> * 主和两个备节点写入状态都成功，返回给client，IO写入完成。

### 2、当加入新主时的IO流程

* 如果新加入的OSD1取代了原有的 OSD4成为 Primary OSD, 由于 OSD1 上未创建 PG , 不存在数据，那么 PG 上的 I/O 无法进行，怎样工作的呢？

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/DE1EC94722DB4FAAAFBD0D66F90DA45B/19775](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/DE1EC94722DB4FAAAFBD0D66F90DA45B/19775)

> * client连接Monitor获取集群的Map信息，
> * 新主OSD1由于没有PG数据，会上报Monitor，告知让OSD2当作临时主，来先接收客户端的数据
> * 当OSD1获取到PG组的信息时，OSD2开始把数据全量同步给OSD1
> * client的IO会直接连接OSD2进行读写，
> * OSD2接收到数据，开始向另外的两个副本写入信息，
> * 等待OSD2以及两个副本写入信息成功，
> * OSD2三分数据都写入成功返回给client，此时client IO完毕
> * 如果OSD1同步数据完毕，临时主OSD2会交出主角色。
> * 如果OSD1成为主节点，OSD2变成副本

### 3、ceph的IO算法流程

* 就是一份数据是怎么存储到后端的OSD上的流程

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/9984558CE351468CA703D386A36060CC/19781](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/9984558CE351468CA703D386A36060CC/19781)

* **FIle：用户需要读写的文件（File---->Object的映射）**
  * ino（File的元数据，File的唯一ID）
  * ono（File切分产生的某个Object的序号，因为会切分出很多的Object，每一个都会有自己的序号，默认以4M为一个块大小）
  * oid（ino+ono）
* **Object是RADOS需要的对象，ceph指定一个静态的hash函数计算oid的值，将oid映射成为一个近似均匀分布的伪随机值，然后和mask按位相与，得到pid（Object----->PG的映射）**
  * hash（oid）& mask-------->  pgid
  * mask=PG总数m(m是2的整数幂)-1
* **PG的用途是对Object的存储进行组织和位置映射，一个PG中有多个Object，采用CRUSH算法，将pgid带入其中，然后得到一组OSD（PG-----> OSD的映射）**
  * CRUSH（pgid）------> (OSD1,OSD2,OSD3)

## 六、ceph的心跳机制

### 1、OSD节点监听端口

> * OSD节点会监听pubilc、cluster、front、back四个端口
>   * public端口
>     * 监听来自Monitor和client的连接
>   * cluster端口
>     * 监听来自OSD peer的连接
>   * front端口
>     * 供客户端连接集群使用的网卡，临时给集群内部间进行心跳，
>   * back端口
>     * 供给集群内部使用的端口，用于集群内部间进行心跳
>   * hbclient端口
>     * 发送ping心跳的message的信息

### 2、OSD间的心跳

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/A96D5F78DEB54DA4BA4110B280B032DD/19810](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/A96D5F78DEB54DA4BA4110B280B032DD/19810)

> * 同一个PG内的OSD相互心跳，他们互相发送ping/pong信息
> * 每隔6s检测一次，（实际上会在这个基础上加一个随机事件来避免峰值）
> * 20s没检测到心跳，加入failure队列

### 3、OSD与Monitor的心跳

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/1DA475BB653E402EB19F1D0EDF83C381/19815](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/3FE93A9F80244E7E8D5B8CE3697E1422/1DA475BB653E402EB19F1D0EDF83C381/19815)

> * OSD有事件发生时，（比如故障，PG变更等）
> * 自身启动5内
> * OSD周期性的上报给Monitor
>   * OSD检查失败队列中的伙伴OSD失败信息
>   * 向Mointor发送失效报告，并将失败信息加入failure_pending队列，然后将其从失败队列中删除，
>   * 收到来自failure_queue或者failure_pending中的OSD心跳信息时，将其从两个队列中踢出，并告知Monitor取消之前的失效报告
>   * 当与Monitor网络重连时，会将failure_pending中的错误报告加回到failure_queue中，并再次发送给Monitor
> * Monitor统计下线OSD
>   * Monitor收集来自OSD的伙伴失效报告，
>   * 当错误报告指向的OSD失效超过一定阈值，当有足够多的OSD报告其失效时，将该OSD踢下线。