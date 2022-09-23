[toc]

## 一、k8s控制器-Replicaset

 前面我们学习了Pod，那我们在定义pod资源时，可以直接创建一个kind：Pod类型的自主式pod，但是这存在一个问题，假如pod被删除了，那这个pod就不能自我恢复，就会彻底被删除，线上这种情况非常危险，所以今天就给大家讲解下pod的控制器，所谓控制器就是能够管理pod，监测pod运行状况，当pod发生故障，可以自动恢复pod。也就是说能够代我们去管理pod中间层，并帮助我们确保每一个pod资源始终处于我们所定义或者我们所期望的目标状态，一旦pod资源出现故障，那么控制器会尝试重启pod或者里面的容器，如果一直重启有问题的话那么它可能会基于某种策略来进行重新布派或者重新编排；如果pod副本数量低于用户所定义的目标数量，它也会自动补全；如果多余，也会自动终止pod资源。

### 1、Replicaset控制器：概念、原理解读

#### 1.1、Replicaset概述

> `ReplicaSet`是kubernetes中的一种副本控制器，简称rs，主要作用是控制由其管理的pod，使pod副本的数量始终维持在预设的个数。它的主要作用就是保证一定数量的Pod能够在集群中正常运行，它会持续监听这些Pod的运行状态，在Pod发生故障时重启pod，pod数量减少时重新运行新的Pod副本。**官方推荐不要直接使用ReplicaSet，用Deployments取而代之**，Deployments是比ReplicaSet更高级的概念，它会管理ReplicaSet并提供很多其它有用的特性，最重要的是Deployments支持声明式更新，声明式更新的好处是不会丢失历史变更。所以Deployment控制器不直接管理Pod对象，而是**由Deployment 管理ReplicaSet，再由ReplicaSet负责管理Pod对象**。

#### 1.2、Replicaset工作原理：如何管理Pod？

Replicaset核心作用在于代用户创建指定数量的pod副本，并确保pod副本一直处于满足用户期望的数量，起到多退少补的作用，并且还具有自动扩容缩容等机制。

Replicaset控制器主要由三个部分组成：

1、用户期望的pod副本数：用来定义由这个控制器管控的pod副本有几个 

2、标签选择器：选定哪些pod是自己管理的，如果通过标签选择器选到的pod副本数量少于我们指定的数量，需要用到下面的组件 

3、pod资源模板：如果集群中现存的pod数量不够我们定义的副本中期望的数量怎么办，需要新建pod，这就需要pod模板，新建的pod是基于模板来创建的。 

### 2、Replicaset资源清单文件编写技巧

* 查看定义Replicaset资源需要的字段有哪些？ 
  * 可以一级一级的逐渐查看

```bash
[root@master ~]# kubectl explain rs
KIND:     ReplicaSet
VERSION:  apps/v1

DESCRIPTION:
     ReplicaSet ensures that a specified number of pod replicas are running at
     any given time.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     If the Labels of a ReplicaSet are empty, they are defaulted to be the same
     as the Pod(s) that the ReplicaSet manages. Standard object's metadata. More
     info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Spec defines the specification of the desired behavior of the ReplicaSet.
     More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Status is the most recently observed status of the ReplicaSet. This data
     may be out of date by some window of time. Populated by the system.
     Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

```bash
[root@master ~]# kubectl explain rs.spec.template
KIND:     ReplicaSet
VERSION:  apps/v1

RESOURCE: template <Object>

DESCRIPTION:
     Template is the object that describes the pod that will be created if
     insufficient replicas are detected. More info:
     https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller#pod-template

     PodTemplateSpec describes the data a pod should have when created from a
     template

FIELDS:
   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

通过上面可以看到，ReplicaSet资源中有两个spec字段。**第一个spec声明的是ReplicaSet定义多少个Pod副本**（默认将仅部署1个Pod）、匹配Pod标签的选择器、创建pod的模板。**第二个spec是spec.template.spec：主要用于Pod里的容器属性等配置**。

.spec.template里的内容是声明Pod对象时要定义的各种属性，所以这部分也叫做PodTemplate（Pod模板）。还有一个值得注意的地方是：在.spec.selector中定义的标签选择器必须能够匹配到spec.template.metadata.labels里定义的Pod标签，否则Kubernetes将不允许创建ReplicaSet。

#### 2.1、Replicaset使用案例：部署Guestbook留言板

* 上传`frontend.tar.gz `到三台node节点

```bash
[root@node1 ~]# docker load -i frontend.tar.gz 
c69ae1aa4698: Loading layer [==================================================>]    131MB/131MB
5f70bf18a086: Loading layer [==================================================>]  1.024kB/1.024kB
651ea5361fc7: Loading layer [==================================================>]  19.48MB/19.48MB
ea43de674974: Loading layer [==================================================>]  180.4MB/180.4MB
c306e0a83bae: Loading layer [==================================================>]  3.584kB/3.584kB
78b5f7f46ca3: Loading layer [==================================================>]  7.855MB/7.855MB
a218acec1fa6: Loading layer [==================================================>]   7.68kB/7.68kB
3a3e37555242: Loading layer [==================================================>]  9.728kB/9.728kB
9acf9c00aead: Loading layer [==================================================>]  14.34kB/14.34kB
e33833ae235f: Loading layer [==================================================>]  4.096kB/4.096kB
a10a1016a983: Loading layer [==================================================>]  22.53kB/22.53kB
fcca76fc56af: Loading layer [==================================================>]  166.9MB/166.9MB
e0eb8c7c584d: Loading layer [==================================================>]  7.168kB/7.168kB
3102a5e90cfa: Loading layer [==================================================>]  3.584kB/3.584kB
cda66b465bac: Loading layer [==================================================>]  9.536MB/9.536MB
1255fb5f5013: Loading layer [==================================================>]  17.45MB/17.45MB
dc03d3bdb471: Loading layer [==================================================>]   12.8kB/12.8kB
6e6dbd6c359e: Loading layer [==================================================>]  2.572MB/2.572MB
782897a48cb0: Loading layer [==================================================>]  4.096kB/4.096kB
e57a0035af53: Loading layer [==================================================>]  4.096kB/4.096kB
fa34d13336c7: Loading layer [==================================================>]  4.096kB/4.096kB
Loaded image: yecc/gcr.io-google_samples-gb-frontend:v3
```

* 编写yaml文件
  * 写的时候可以开两个控制终端，一个写yaml，另一个查看帮助
  * `<Object>	`:	代表是对象，这个字段回车后并行写，不需要-
  * `<string>`：字符串，这个字段后直接跟字符串即可
  * `<integer>`：整形数字，后边直接跟数字
  * `<[]Object>`：对象列表，回车后需要-连接起来

```yaml
[root@master replicaset]# cat replicaset.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guest
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: test
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: yecc/gcr.io-google_samples-gb-frontend:v3
        imagePullPolicy: IfNotPresent
    
```

* 启动

```bash
[root@master replicaset]# kubectl apply -f replicaset.yaml ^C
[root@master replicaset]# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-2p8kj   1/1     Running   0          10m
frontend-br9mp   1/1     Running   0          16m
frontend-rmrw7   1/1     Running   0          16m
```

#### 2.2、Replicaset管理pod：扩容、缩容、更新

> ReplicaSet最核心的功能是可以动态扩容和回缩，如果我们觉得两个副本太少了，想要增加，只需要修改配置文件replicaset.yaml里的replicas的值即可，原来replicas: 3，现在变成replicaset: 4，修改之后，执行如下命令更新：

```bash
[root@master ~]# kubectl get pods -w
#在这里可以清楚的看到pod的变化过程
NAME             READY   STATUS    RESTARTS   AGE
frontend-2p8kj   1/1     Running   0          46m
frontend-br9mp   1/1     Running   0          53m
frontend-rmrw7   1/1     Running   0          53m
frontend-jnhpx   0/1     Pending   0          0s
frontend-jnhpx   0/1     Pending   0          0s
frontend-jnhpx   0/1     ContainerCreating   0          0s
frontend-jnhpx   0/1     ContainerCreating   0          0s
frontend-jnhpx   1/1     Running             0          1s
```

* 或者通过edit参数来修改

```bash
[root@master replicaset]# kubectl edit rs frontend
#修改完之后保存推出即可
replicaset.apps/frontend edited
```

* 实时更新镜像
  * 有两种方式，不管是哪种方式，都是通过**先删除原来的pod，创建一个新的pod，才会使用新的image**
    * 通过edit参数动态修改维护数量
    * 直接删除一个pod，让其自动创建
    * **提示：直接修改yaml文件中的image，apply之后不生效**

* **通过edit修改**
  * 可以看到，在吧pod改为两个之后，只是删除了原有的3个pod

```bash
[root@master replicaset]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
frontend-br9mp   1/1     Running   0          87m   10.244.83.79   node2.linux.com   <none>           <none>
frontend-rmrw7   1/1     Running   0          87m   10.244.83.78   node2.linux.com   <none>           <none>
[root@master replicaset]# curl 10.244.83.79
<html ng-app="redis">
  <head>
    <title>Guestbook</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.12/angular.min.js"></script>
    <script src="controllers.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.0/ui-bootstrap-tpls.js"></script>
  </head>
  <body ng-controller="RedisCtrl">
    <div style="width: 50%; margin-left: 20px">
      <h2>Guestbook</h2>
    <form>
    <fieldset>
    <input ng-model="msg" placeholder="Messages" class="form-control" type="text" name="input"><br>
    <button type="button" class="btn btn-primary" ng-click="controller.onRedis()">Submit</button>
    </fieldset>
    </form>
    <div>
      <div ng-repeat="msg in messages track by $index">
        {{msg}}
      </div>
    </div>
    </div>
  </body>
</html>
[root@master replicaset]# curl  10.244.83.78
<html ng-app="redis">
  <head>
    <title>Guestbook</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.12/angular.min.js"></script>
    <script src="controllers.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.0/ui-bootstrap-tpls.js"></script>
  </head>
  <body ng-controller="RedisCtrl">
    <div style="width: 50%; margin-left: 20px">
      <h2>Guestbook</h2>
    <form>
    <fieldset>
    <input ng-model="msg" placeholder="Messages" class="form-control" type="text" name="input"><br>
    <button type="button" class="btn btn-primary" ng-click="controller.onRedis()">Submit</button>
    </fieldset>
    </form>
    <div>
      <div ng-repeat="msg in messages track by $index">
        {{msg}}
      </div>
    </div>
    </div>
  </body>
</html>
```

* 在这里rs维护的pod数量是2个，我们改成3个，让他自动新创建一个
  * 可以看到，新创建的pod已经使用了新的image

```bash
[root@master replicaset]# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-br9mp   1/1     Running   0          91m
frontend-rmrw7   1/1     Running   0          91m
[root@master replicaset]# 
[root@master replicaset]# kubectl edit rs frontend
replicaset.apps/frontend edited
[root@master replicaset]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
frontend-5vmhn   1/1     Running   0          37s   10.244.20.80   node3.linux.com   <none>           <none>
frontend-br9mp   1/1     Running   0          94m   10.244.83.79   node2.linux.com   <none>           <none>
frontend-rmrw7   1/1     Running   0          94m   10.244.83.78   node2.linux.com   <none>           <none>
[root@master replicaset]# curl 10.244.20.80
Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>
```

* **删除pod修改**

```bash
[root@master replicaset]# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-5vmhn   1/1     Running   0          8m14s
frontend-br9mp   1/1     Running   0          102m
frontend-rmrw7   1/1     Running   0          102m
[root@master replicaset]# kubectl delete pods frontend-br9mp
pod "frontend-br9mp" deleted
[root@master replicaset]# kubectl  get pods 
NAME             READY   STATUS    RESTARTS   AGE
frontend-5vmhn   1/1     Running   0          8m35s
frontend-rmrw7   1/1     Running   0          102m
frontend-xfwt8   1/1     Running   0          10s
[root@master replicaset]# kubectl  get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
frontend-5vmhn   1/1     Running   0          8m38s   10.244.20.80   node3.linux.com   <none>           <none>
frontend-rmrw7   1/1     Running   0          102m    10.244.83.78   node2.linux.com   <none>           <none>
frontend-xfwt8   1/1     Running   0          13s     10.244.20.81   node3.linux.com   <none>           <none>
[root@master replicaset]# curl 10.244.20.81  
Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>
```

> 生产环境如果升级，可以删除一个pod，观察一段时间之后没问题再删除另一个pod，但是这样需要人工干预多次；实际生产环境一般采用蓝绿发布，原来有一个rs1，再创建一个rs2（控制器），通过修改service标签，修改service可以匹配到rs2的控制器，这样才是蓝绿发布，这个也需要我们精心的部署规划，我们有一个控制器就是建立在rs之上完成的，叫做Deployment
