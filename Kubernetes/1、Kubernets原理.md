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
* 与etcd数据库进行交互，读写记录集群状态信息，
* 接收客户端操作请求，验证身份
* 接收kubelet发送过来的注册请求
* 认证授权，访问控制

#### 2）Scheduler 组件

* 调度客户端操作请求，选择合适的Node节点运行资源，调度POD资源

#### 3）Controller Manager组件

* 管理集群控制器，例如replication controller负责维护POD、Node的副本数量、
* 监控集群中的组件信息，

#### 4）etcd组件

* 用来存储k8s的资源状态信息及网络信息，所以要确保etcd高可用
* 键值对数据库
* 

#### 5）kubectl组件

* 命令行工具，用来操作k8s中的资源组件，增删改查
* 可以安装在任何位置，默认控制节点
* 通过加载config配置文件，来控制整个K8s集群

### 2、`Node节点`

![image-20220427195128951](https://s2.loli.net/2022/04/27/JC2lzdkNMrswIh5.png)

运行pod的主机，可以是物理服务器，也可以是虚拟机；处理生产级流量的Kubernets集群至少要有三个Node

#### 1）container engine

#### 2）kubelet

* 负责整个POD的生命周期
* 以守护进程的方式存在，并不是以POD的形式存在。（K8s中所有的集群节点都是以POD的方式部署，所以不管是Master节点还是Node节点都会存在）
*  与API server通信，报告自身状态信息，

#### 3）kube-proxy

* 也会负责端口的自动映射，（服务自动发现，修改防火墙规则，建立映射关系）
* 再多个容器间实现负载均衡
* pod前端有services，访问的时候先访问services，再由services将请求代理到pod上，其中就是通过kube-proxy，每创建一个serives，kube-proxy就会将serives的IP和端口，记录到iptables/ipvs.

#### 4) Core-dns

* k8s中实现域名解析的组件，
* 创建services的时候生成FQDN，在k8s内部访问FQDN和访问services的IP是一样的效果，也是通过core-dns实现的

#### 5）Calico

* 实现k8s中不同节点的网络通信
* 比如我有三个命名空间：开发、测试、生产
  * 可以通过calico来实现不同的命名空间之间的通信

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
* `pod的设计理念`是支持**多个容器再同一个pod中共享网络和文件系统**（挂载）
* k8s集群会为每个pod分配IP、成为podIP
* 从实际应用角度讲，多数应用都是一个pod中只放一个容器
* 本质上讲，k8s创建一个pod后，每个pod中存放两个容器，一个是用户自定义的容器，另一个是k8s自动创建的名为pause-amd64的容器，实际上pod的IP是配置在pause容器上的

```bash
root@xianchaomaster1 ~]# cat pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: tomcat-pod
  namespace: default
  labels:
    tomcat:  tomcat-pod
spec:
  containers:
  - name:  tomcat-pod-java
    ports:
    - containerPort: 8080
    image: tomcat:8.5-jre8-alpine
  imagePullPolicy: IfNotPresent

```



### 5、Label 标签

* 一个key-value的键值对的数据结构，用于在k8s集群中识别标识对象，例如每创建一个POD都应该指定一个或多个label
* 一个对象可以被分配一个label或多个label
* k8s 通过 Label 可实现多维度的资源分组管理，后续可通过 Label Selector 查询和筛选拥有某些 Label 的资源对象，例如创建一个 Pod，给定一个 Label是app=tomcat，那么service可以通过label selector选择拥有app=tomcat的pod，和其相关联，也可通过 app=tomcat 删除拥有该标签的 Pod 资源。

### 6、Annotations

* key-value键值对的数据结构，用于为对象添加注释说明信息

* 默认情况下可以不写

* 在特殊应用场景下，可以通过Annotations辅助部署应用；例如特定的镜像结合特定的说明信息时

### 7、Replication Controller复制控制器RC

* RC是k8s集群中保证POD高可用的对象，通过监控运行中的POD来保证集群中运行指定数量的POD副本
* RC通过 Label Selector机制实现对POD副本的自动控制

### 8、Replicaset副本集RS

* RS是新一代的RC，提供相同的高可用能力

### 9、Deployement部署

* 通过deployement来管理集群中的RS、POD
* 在实际操作中，很少直接去操作RS、或者POD在k8s集群中完成对RS和POD 的操作都是通过Deployment完成
* 在创建deployement 的时候自动创建RS，再通过rs制定一定数量的副本来创建POD
* 在升级操作时，Deployment会做滚动升级
* Deployment能对Pod扩容、缩容、滚动更新和回滚、维持Pod数量。

```bash
cat deployment.yaml

apiVersion: apps/v1
kind: Deployment	#指定要创建的资源类型是deployment
metadata:
  name: my-nginx
spec:
  selector:	#制定标签选择器
    matchLabels:	#过滤run：my-nginx的标签
      run: my-nginx
  replicas: 2	#制定2个副本数量
  template:
    metadata:
      labels:	#制定标签，基于这个deployment控制器创建的POd，全都是这个标签
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
[root@xianchaomaster1 ~]# kubectl apply -f deployment.yaml	#应用yaml文件

[root@xianchaomaster1 ~]# kubectl get deploy | grep my-nginx	#这个时候会创建出来一个deploymen控制器my-nginx
my-nginx         2/2     2            2           2m52s

[root@xianchaomaster1 ~]# kubectl get rs | grep my-nginx	#也会自动生成一个rs、名字组成是deployment+一串随机数
my-nginx-5b56ccd65f         2         2         2       3m26s

[root@xianchaomaster1 ~]# kubectl get pods -l run=my-nginx    #最后查看pod的时候可以看到两个pod，因为制定的数量就是两个
```

![image-20220803203740879](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220803203740879.png)

### 10、HPA  HorizontalPodAutoscaler

* 用于POD实现自动横向扩容
  * HPA会实时监控POD的负载情况，完成自动扩展
  * 可通过CPU百分比、自定义指标（QPS）实现

### 11、Services服务

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/A772628F1E2F428C9AEB580ACFF1FE8A/42A7519258BE4E86A8B248FC2E6FD8E9/8528](https://s2.loli.net/2022/04/02/hkwlKMGuA8qcpa7.png)

* 将具有相同 的label分为一个组，有集群为services分配固定IP，便于用户访问
* 可以理解为一个services是一组具有相同标签的的POD的集合
* 由kube-proxy组件实现，kube-proxy创建service类似负载均衡器，后端POD类似real-server
* POD宕机产生新的POD时，kube-proxy会自动更新etcd数据库关于service与POD的对应关系
* 客户端在访问的时候先访问services，配合标签选择器选择制定标签的pod，在客户端看来，访问的就是serviexs，

```bash
#先创建一个pod
cat pod_test.yaml
apiVersion: apps/v1
kind: Deployment	#制定资源类型是deployment
metadata:
  name: my-nginx	#deployment的名字
spec:
  selector:			#创建一个叫做run: my-nginx的标签选择器
    matchLabels:
      run: my-nginx
  replicas: 2		#制定2个副本
  template:
    metadata:
      labels:		#创建出来的pod全部都打上run：my-nginx 的标签
        run: my-nginx	
    spec:
      containers:
      - name: my-nginx	#容器名字
        image: nginx	#镜像
        ports:
        - containerPort: 80
#再创建一个services
cat service.yaml
apiVersion: v1
kind: Service	#制定资源类型是services
metadata:
  name: my-nginx	#services的名字
  labels:
    run: my-nginx	#指定serives本身的标签 run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:			#指定关联的标签选择器，即：serives关联哪个标签，这里指定run: my-nginx 的标签选择器。
run: my-nginx
#客户端在访问services的IP的时候，识别到serives的标签是run: my-nginx ，再通过标签选择器，去找标签为run：my-nginx 的所有POD
[root@xianchaomaster1 ~]# kubectl apply -f service.yaml		#应用yaml

[root@xianchaomaster1 ~]# kubectl get svc -l run=my-nginx	#查看services信息
```

![image-20220803205540995](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220803205540995.png)

```bash
[root@xianchaomaster1 ~]# curl 10.105.104.137		#访问，可以获取到数据
```

![image-20220803205624988](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220803205624988.png)



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