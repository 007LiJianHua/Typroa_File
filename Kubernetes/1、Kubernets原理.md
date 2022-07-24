[toc]

## 一、Kubernets介绍

### 1、kubernets是什么

* 由Goole公司开源的应用，基于Go语言编写
* 简称k8s

### 2、kubernets作用

* 服务发现和负载均衡
* 存储编排
* 自动部署和回滚
* 资源调度分配
* 自我修复

总而言之，kubernets的目的就是让部署应用变得更简单、高效

## 二、Kubernets架构

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/A772628F1E2F428C9AEB580ACFF1FE8A/0111A1A68D83420E8A75EF3A482E7482/8427](https://s2.loli.net/2022/04/02/dNO52nWKMegFDXS.png)

### 1、`Master节点`

![image-20220427195040317](https://s2.loli.net/2022/04/27/m7f5YFMs92cRTo3.png)

* Master 负责管理整个集群，==Master负责协调集群中的所有活动==，例如调度应用，维护应用的所需状态，应用扩容，以及退出新的更新，

#### 1）API Server组件

* API:应用程序接口

* 与etcd数据库进行交互，读写集群状态信息，
* 接收客户端操作请求，验证身份
* 接收kubelet发送过来的注册请求

#### 2）Scheduler 组件

* 调度客户端操作请求，选择合适的Node节点运行资源

#### 3）Controller Manager组件

* 管理集群控制器，例如replication controller负责维护容器的副本数量

### 2、`Node节点`

![image-20220427195128951](https://s2.loli.net/2022/04/27/JC2lzdkNMrswIh5.png)

运行pod的主机，可以是物理服务器，也可以是虚拟机；处理生产级流量的Kubernets集群至少要有三个Node

#### 1）container engine

#### 2）kubelet

* 负责整个POD的生命周期
* 以守护进程的方式存在，并不是以POD的形式存在。（K8s中所有的集群节点都是以POD的方式部署，所以不管是Master节点还是Node节点都会存在）

#### 3）kube-proxy

* 也会负责端口的自动映射，（服务自动发现，修改防火墙规则，建立映射关系）

* 再多个容器间实现负载均衡

### 3、`etcd数据库`

* 记录集群装填数据，例如node节点信息，pod信息（IP、网关），service等
* 集群出现错误，会去重新选主
* 负责元数据的保存
* 负责分布式协同，保证数据库的数据保持同步，

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/A772628F1E2F428C9AEB580ACFF1FE8A/80EF9F63D8ED4E5580431EBDD3B75CA0/8471](https://s2.loli.net/2022/04/02/ADz4hGPoaxIKWpi.png)

### 4、通过K8s集群创建一个pod的过程说明组件间的交互

* 用户通过客户端工具向API server组件发送创建pod的请求
* api-server接收到该请求后，会将请求信息（POD名称，镜像，卷，网络等信息）记录到数据库中
* scheduler组件会周期性的请求api-server，以询问是否有操作请求；
* api-server组件查询etcd数据库是否有新的操作请求，来响应scheduler组件，scheduler组件会得知创建pod的请求
* scheduler按照一定的算法，选择一个合适的node节点计划创建POD，并将选定的节点信息返回给API-server，api-server会将该node节点与要创建的pod对应关系写入到etcd数据库
* Node节点选择完毕，启动kubelet，在kubelet启动后，会再向api-server注册自己，以让api-server得知有运行kubelet服务的节点存在，并将node节点信息记录到etcd数据库，这样scheduler组件才能根据数据库的记录来选择合适的节点来创建pod
* kubelet组件也会周期性的请求api-server，以询问是否有自己要做的操作，api-server查询数据库，响应kubelet，kubelet得知要创建的pod信息后，调用container engine 创建容器
* 容器创建完成后会，为了便于访问，有kube-proxy提供负载均衡

## 三、kubernets集群对象

### 1、Node

* `运行pod的主机`，可以是物理机，也可以是虚拟机；每个node节点上都要运行容器引擎（docker、rts、podman）、kubelet、组件以及kube-proxy组件

### 2、Namespace

* `命名空间`，是资源或对象的抽象集合
* kubernets集群默认会创建三个namespace
  * defaults-----默认namespace
    * 创建的资源默认命名空间
  * kube-system
    * k8s集群内部组件
  * kube-public
    * 所有的用户都可以访问，没有经过身份验证的也可以访问
  * kube-node-lease
    * 集群中的心跳检查

### 3、container

* 容器

### 4、Pod

* kubernets所能管理的最小单元，一个pod中可以放一个容器，也可以放多个容器
* `pod的设计理念`是支持多个容器再同一个pod中共享网络和文件系统（挂载）
* k8s集群会为每个pod分配IP、成为podIP
* 从实际应用角度讲，多数应用都是一个pod中只放一个容器
* 本质上讲，k8s创建一个pod后，每个pod中存放两个容器，一个是用户自定义的容器，另一个是k8s自动创建的名为pause-amd64的容器，实际上pod的IP是配置在pause容器上的

### 5、Label 标签

* 一个key-value的键值对的数据结构，用于在k8s集群中识别标识对象，例如每创建一个POD都应该指定一个或多个label
* 一个对象可以被分配一个label或多个label
* label分配

### 6、Annotations

* key-value键值对的数据结构，用于为对象添加注释说明信息

* 默认情况下可以不写

* 在特殊应用场景下，可以通过Annotations辅助部署应用；例如特定的镜像结合特定的说明信息时

### 7、Replication Controller复制控制器RC

* RC是k8s集群中保证POD高可用的对象，通过监控运行中的POD来保证集群中运行指定数量的POD副本
* RC通过 Label Selector机制实现对POD副本的自动控制

### 8、Replica Set副本集RS

* RS是新一代的RC，提供相同的高可用能力

### 9、Deployement部署

* 通过deployement来管理集群中的RS、POD
* 在实际操作中，很少直接去操作RS、或者POD在k8s集群中完成对RS和POD 的操作都是通过Deployment完成
* Deployment自动创建RS保证POD副本
* 在升级操作时，Deployment会做滚动升级
* 在升级操作时，如果检测到失败，会自动回滚

### 10、HPA  HorizontalPodAutoscaler

* 用于POD实现自动横向扩容
  * HPA会实时监控POD的负载情况，完成自动扩展
  * 可通过CPU百分比、自定义指标（QPS）实现

### 11、Services服务

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/A772628F1E2F428C9AEB580ACFF1FE8A/42A7519258BE4E86A8B248FC2E6FD8E9/8528](https://s2.loli.net/2022/04/02/hkwlKMGuA8qcpa7.png)

* 将具有相同的label分为一个组，有集群为services分配固定IP，便于用户访问
* 可以理解为一个services是一组具有相同标签的的POD的集合
* 由kube-proxy组件实现，kube-proxy创建service类似负载均衡器，后端POD类似real-server
* POD宕机产生新的POD时，kube-proxy会自动更新etcd数据库关于service与POD的对应关系

### 12、服务发现

* 借助DNS实现，coreDNS组件
* 创建service 时，k8s集群会为其分配一个域名 www.default.svc.cluster.local
* 域名格式
  * 服务名称.命名空间.svc.cluster.local
* service创建完成后，会自动形成 IP域名的对应关系，集群内多个service互相通信时，依靠域名实现，

### 13、Job任务

* k8s集群执行处理批量任务的对象
* 用于运行一次性任务，任务完成后POD自动退出，类似于一次性任务

### 14、DaemonSet

* 保证选定的业务在所有的节点运行一个Pod
* 典型场景：监控agent，日志收集工具  filebeat

### 15、StatefulSet 有状态服务集

* 通过Statefulset创建的Pod，集群会为其分配一个固定的名称，而且该名称创建后是不能修改的、也是集群全局唯一的；用于让构建业务集群的多个Pod间通过名称通信，而不是借助IP，来保证服务的有状态性
* stateful的实现要依赖于存储volume，例如一个Pod挂载一个独立的存储，该Pod宕机后，自动创建一个新的Pod，新Pod一样会去挂载存储获取数据，这就相当于新Pod继承的原有Pod的所有数据及状态
* 典型应用场景：数据库集群

### 16、Volume 卷

* 用于实现数据持久化存储

### 17、Federation	集群联邦

* 存在多个不同的k8s集群时，常规情况下，我们连接哪个集群创建资源，资源就会被创建在哪个集群里
* Federation基于全局控制器，可实现将多个不同的集群放到一个全局控制器中，创建资源时，向该全局控制器发送请求，全局控制器会选择一个集群创建资源
* Fderation基于全局调度器，在访问资源时，向全局调度器发送请求，它会将请求转交到合适的集群处理

## 四、访问控制概述

![image-20220428195055955](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428195055955.png)

### 1、认证（Authentication）

### 2、授权（Authorization）

### 3、准入控制（Admission Control）

### 4、基于角色访问控制（RBAC）

![image-20220428195530115](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428195530115.png)

### 5、kubernetes中的用户

![image-20220428195745590](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428195745590.png)

#### 1、用户账号

> 对所有命名空间生效，k8s的运维人员，不通过用户名密码，通过密钥配合证书登录，

#### 2、服务账号

![image-20220428200522753](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428200522753.png)

（Token controller）令牌控制器、

> 绑定一个命名空间生效，
>
> 当某一个POD需要使用API来查看其他资源信息时
>
> 常用于DashBoard

#### 3、Secret概述

![image-20220428201025992](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428201025992.png)

**大概流程**

> 先创建一个NS---->再创建一个Default SA----再通过Token控制器来创建Secret------将必要信息注入POD----最后通过这个POD来访问这个API

### 6、Role和ClusterRole

![image-20220428201937010](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428201937010.png)

**Role**：只作用于单个命名空间，

**ClusterROle**：作用于整个集群

![image-20220428203250574](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220428203250574.png)