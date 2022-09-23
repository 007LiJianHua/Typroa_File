[toc]

## 一、Deployment控制器

### 1、Deployment控制器：概念、原理解读

#### 1.1、Deployment概述

Deployment官方文档：

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

> * Deployment是kubernetes中最常用的资源对象，为ReplicaSet和Pod的创建提供了一种声明式的定义方法，在Deployment对象中描述一个期望的状态，Deployment控制器就会按照一定的控制速率把实际状态改成期望状态，通过定义一个Deployment控制器会创建一个新的ReplicaSet控制器，通过ReplicaSet创建pod，删除Deployment控制器，也会删除Deployment控制器下对应的ReplicaSet控制器和pod资源.
> * 使用Deployment而不直接创建ReplicaSet是因为Deployment对象拥有许多ReplicaSet没有的特性，例如滚动升级和回滚
> * **扩展：声明式定义是**指直接修改资源清单yaml文件，然后通过`kubectl apply-f` 资源清单yaml文件，就可以更改资源
> * Deployment控制器是建立在rs之上的一个控制器，可以管理多个rs，每次更新镜像版本，都会生成一个新的rs，把旧的rs替换掉，多个rs同时存在，但是只有一个rs运行。

![image-20220915173348147](https://tva1.sinaimg.cn/large/e6c9d24ely1h67f9nkkmtj20wu0bx75t.jpg)

​		rs v1控制三个pod，删除一个pod，在rsv2上重新建立一个，依次类推，直到全部都是由rs v2控制，如果rsv2有问题，还可以回滚，Deployment是建构在rs之上的，多个rs组成一个Deployment，**但是只有一个rs处于活跃状态.**

​		如果想要回滚，原来的v1就是第一个版本，以相同的方法逐渐把pod资源在v1上创建出来

### 2、Deployment工作原理：如何管理rs和Pod？

​	Deployment可以使用声明式定义，**直接在命令行通过纯命令的方式完成对应资源版本的内容的修改，也就是通过打补丁的方式进行修改；**Deployment能提供滚动式自定义自控制的更新；对Deployment来讲，我们在实现更新时还可以实现控制更新节奏和更新逻辑。

> 更新逻辑：
>
> 比如说Deployment控制5个pod副本，pod的期望值是5个，但是升级的时候需要额外多几个pod，那我们控制器可以控制在5个pod副本之外还能再增加几个pod副本；比方说能多一个，但是不能少，那么升级的时候就是先增加一个，再删除一个，增加一个删除一个，始终保持pod副本数是5个；
>
> 还有一种情况，最多允许多一个，最少允许少一个，也就是最多6个，最少4个，第一次加一个，删除两个，第二次加两个，删除两个，依次类推，可以自己控制更新方式，**这种滚动更新需要加readinessProbe和livenessProbe探测**，确保pod中容器里的应用都正常启动了才删除之前的pod。
>
> 更新节奏：
>
> 启动第一步，刚更新第一批就暂停了也可以；假如目标是5个，允许一个也不能少，允许最多可以10个，那一次加5个即可；这就是我们可以自己控制节奏来控制更新的方法。

通过Deployment对象，你可以轻松的做到以下事情：

1、创建ReplicaSet和Pod

2、滚动升级（不停止旧服务的状态下升级）和回滚应用（将应用回滚到之前的版本）

3、平滑地扩容和缩容

4、暂停和继续Deployment

### 3、Deployment资源清单文件编写技巧

* 查看Deployment资源对象由哪几部分组成

```bash
#查看 Deployment 资源对象由哪几部分组成 
[root@xianchaomaster1 ~]# kubectl explain deployment 
KIND: Deployment 
VERSION: apps/v1 
DESCRIPTION: 
 Deployment enables declarative updates for Pods and ReplicaSets. 
FIELDS: 
 apiVersion <string> #该资源使用的 api 版本 
 kind <string> #创建的资源是什么？ 
 metadata <Object> #元数据，包括资源的名字和名称空间 
 spec <Object> #定义容器的 
 status <Object> #状态，不可以修改

[root@master deploy]# kubectl explain deploy.spec
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the Deployment.

     DeploymentSpec is the specification of the desired behavior of the
     Deployment.

FIELDS:
   minReadySeconds      <integer>#k8s在等待设置的时间后才进行升级，如果没有设置该值，k8s会假设该容器起来后就提供服务了。
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   paused       <boolean>#暂停，当我们更新的时候，创建pod先暂停，不是立即更新。# k8s在升级过程中有可能由于各种原因升级卡住（这个时候还没有明确的升级失败），比如在拉取被墙的镜像，权限不够等错误。那么这个时候就需要有个 deadline ，在 deadline之内如果还卡着，那么就上报这个情况，这个时候这个Deployment 状态就被标 记为False ，并且注明原因。但是它并不会阻止 Deployment 继续进行卡住后面的操作。完全由用户进行控制。
     Indicates that the deployment is paused.

   progressDeadlineSeconds      <integer>
     The maximum time in seconds for a deployment to make progress before it is
     considered to be failed. The deployment controller will continue to process
     failed deployments and a condition with a ProgressDeadlineExceeded reason
     will be surfaced in the deployment status. Note that progress will not be
     estimated during the time a deployment is paused. Defaults to 600s.

   replicas     <integer>#副本数
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.

   revisionHistoryLimit <integer>#保留的历史版本，默认是10
     The number of old ReplicaSets to retain to allow rollback. This is a
     pointer to distinguish between explicit zero and not specified. Defaults to
     10.

   selector     <Object> -required-#标签选择器，选择它关联的pod
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment. It must match the pod
     template's labels.

   strategy     <Object>#更新策略
     The deployment strategy to use to replace existing pods with new ones.

   template     <Object> -required-#定义的pod模版
     Template describes the pods that will be created.
     
     
 	#查看 Deployment 下的 spec.strategy 字段 
[root@xianchaomaster1 ~]# kubectl explain deploy.spec.strategy 
KIND: Deployment 
VERSION: apps/v1 
RESOURCE: strategy <Object> 
DESCRIPTION: 
 The deployment strategy to use to replace existing pods with new ones. 
 DeploymentStrategy describes how to replace existing pods with new ones. 
FIELDS: 
 rollingUpdate <Object> 
 type <string> 
 Type of deployment. Can be "Recreate" or "RollingUpdate". Default is 
 RollingUpdate. #支持两种更新，Recreate 和 RollingUpdate 
#Recreate 是重建式更新，删除一个更新一个

#RollingUpdate 滚动更新，定义滚动更新方式，也就是 pod 能多几个，少几个 #查看 Deployment 下的 spec.strategy.rollingUpdate 字段 
[root@xianchaomaster1 ~]# kubectl explain deploy.spec.strategy.rollingUpdate 
KIND: Deployment 
VERSION: apps/v1 
RESOURCE: rollingUpdate <Object> 
DESCRIPTION: 
 Rolling update config params. Present only if DeploymentStrategyType = 
 RollingUpdate. 
 Spec to control the desired behavior of rolling update. 
FIELDS: 
 maxSurge <string> #我们更新的过程当中最多允许超出的指定的目标副本数有几个； 它有两种取值方式，第一种直接给定数量，第二种根据百分比，百分比表示原本是 5 个，最多可以超出 20%，那就允许多一个，最多可以超过 40%，那就允许多两个 
 maxUnavailable <string> #最多允许几个不可用 假设有 5 个副本，最多一个不可用，就表示最少有 4 个可用
 
 
 #查看 Deployment 下的 spec.template 字段 #template 为定义 Pod 的模板，Deployment 通过模板创建 Pod 
[root@xianchaomaster1 ~]# kubectl explain deploy.spec.template 
KIND: Deployment 
VERSION: apps/v1 
RESOURCE: template <Object> 
DESCRIPTION: 
 Template describes the pods that will be created. 
 PodTemplateSpec describes the data a pod should have when created from a 
template 
FIELDS: 
 metadata <Object> #定义模板的名字 
 spec <Object> 
#deployment.spec.template 为 Pod 定义的模板，和 Pod 定义不太一样，template 中不包含apiVersion 和 Kind 属性，要求必须有 metadata。deployment.spec.template.spec 为容器的属性信息，其他定义内容和 Pod 一致。


#查看 Deployment 下的 spec.template.spec 字段 
[root@xianchaomaster1 ~]# kubectl explain deploy.spec.template.spec 
KIND: Deployment 
VERSION: apps/v1 
RESOURCE: spec <Object> 
DESCRIPTION: 
 Specification of the desired behavior of the pod. More info: 
 https://git.k8s.io/community/contributors/devel/sig-architecture/apiconventions.md#spec-and-status 
 PodSpec is a description of a pod. 
FIELDS: 
 activeDeadlineSeconds <integer> #activeDeadlineSeconds 表示 Pod 可以运行的最长时间，达到设置的该值后，Pod 会自动停
止。 
 affinity <Object> #定义亲和性，跟直接创建 pod 时候定义亲和性类似 
 automountServiceAccountToken <boolean> #身份认证相关的 
 containers <[]Object> -required- #定义容器属性 
 dnsConfig <Object> #设置 Pod 的 DNS 
dnsConfig: 
 nameservers: 
 - 192.xxx.xxx.6 
 searches: 
 - xianchao.svc.cluster.local 
 - my.dns.search.xianchao 
 dnsPolicy <string> # dnsPolicy 决定 Pod 内预设的 DNS 配置策略
 None 无任何策略：使用自定义的策略 
Default 默认：使用宿主机的 dns 配置，/etc/resolv.conf 
ClusterFirst 集群 DNS 优先，与 Default 相反，会预先使用 kube-dns (或 CoreDNS ) 的
信息当预设置参数写入到该 Pod 内的 DNS 配置。 
ClusterFirstWithHostNet 集群 DNS 优先，并伴随着使用宿主机网络：同时使用 
hostNetwork 与 kube-dns 作为 Pod 预设 DNS 配置。
```

**enableServiceLinks <boolean>** 

 **ephemeralContainers <[]Object> #定义临时容器** 

临时容器与其他容器的不同之处在于，它们缺少对资源或执行的保证，并且永远不会自动重启，因此不适用于构建应用程序。临时容器使用与常规容器相同的 ContainerSpec 段进行描述，但许多字段是不相容且不允许的。 临时容器没有端口配置，因此像 ports，livenessProbe，readinessProbe 这样的字段是不允许的。 

 Pod 资源分配是不可变的，因此 resources 配置是不允许的。 

**临时容器用途：** 

**当由于容器崩溃或容器镜像不包含调试应用程序而导致 kubectl exec 无用时，临时容器对于交互式故障排查很有用。** 

```bash
hostAliases <[]Object> #在 pod 中增加域名解析的 
例如
  hostAliases: 
  – ip: "10.1.2.2" 
    hostnames: 
    – "mc.local"
    – "rabbitmq.local" 
  – ip: "10.1.2.3" 
    hostnames: 
    – "redis.local" 
    – "mq.local" 
 hostIPC <boolean> #使用主机 IPC 
 hostNetwork <boolean> #是否使用宿主机的网络 
 hostPID <boolean> #可以设置容器里是否可以看到宿主机上的进程。True 可以 
 hostname <string> 
 imagePullSecrets <[]Object> 
 initContainers <[]Object> #定义初始化容器 
 nodeName <string> #定义 pod 调度到具体哪个节点上 
 nodeSelector <map[string]string> #定义节点选择器 
 overhead <map[string]string> #overhead 是 1.16 引入的字段，在没有引入 Overhead 之前，只要一个节点的资源可用量大于等于 Pod 的 requests 时，这个 Pod 就可以被调度到这个节点上。引入 Overhead 之后，只有节点的资源可用量大于等于 Overhead 加上requests 的和时才能被调度上来。
 preemptionPolicy <string>
 priority <integer> 
 priorityClassName <string> 
 readinessGates <[]Object> 
 restartPolicy <string> #Pod 重启策略 
 runtimeClassName <string> 
 schedulerName <string> 
 securityContext <Object> #是否开启特权模式 
 serviceAccount <string> 
 serviceAccountName <string> 
 setHostnameAsFQDN <boolean> 
 shareProcessNamespace <boolean> 
 subdomain <string> 
 terminationGracePeriodSeconds <integer> #在真正删除容器之前，K8S 会先发终止信号（kill -15 {pid}）给容器，默认 30s 
 tolerations <[]Object> #定义容忍度 
 topologySpreadConstraints <[]Object 
 volumes <[]Object> #挂载存储卷
```

### 4、Deployment 使用案例：创建一个 web 站点 

> **deployment 是一个三级结构，deployment 管理 replicaset，replicaset 管理 pod，** 

* 把 `myapp-blue-v1.tar.gz` 和 `myapp-blue-v2.tar.gz` 上传到 三哥node节点上，

手动解压： 

```bash
[root@node1 ~]# docker load -i myapp-blue-v1.tar.gz
cdb3f9544e4c: Loading layer [==================================================>]  58.44MB/58.44MB
190f3188c8aa: Loading layer [==================================================>]  54.24MB/54.24MB
d1bade4185fe: Loading layer [==================================================>]  3.584kB/3.584kB
3602844318e8: Loading layer [==================================================>]  2.048kB/2.048kB
1dc7ed42d5a8: Loading layer [==================================================>]  4.608kB/4.608kB
Loaded image: janakiramm/myapp:v1
[root@node1 ~]# docker load -i myapp-blue-v2.tar.gz
af2ad77d9e26: Loading layer [==================================================>]  4.608kB/4.608kB
Loaded image: janakiramm/myapp:v2
```

* 在编写deploymen的文件时，可以一个窗口利用`explain`命令查看帮助，另一个窗口来编写文件

```bash
[root@master deploy]# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      name: test
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: janakiramm/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        
[root@master deploy]# kubectl apply -f deployment.yaml 
deployment.apps/myapp-v1 created
[root@master deploy]# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
myapp-v1-86f6b6d8c8-dpbd7   1/1     Running   0          111s
myapp-v1-86f6b6d8c8-r9wlv   1/1     Running   0          111s
[root@master deploy]# kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
myapp-v1   2/2     2            2           115s
#创建的控制器名字是 myapp-v1 
#1.NAME ：列出名称空间中 deployment 的名称。 
#2.READY：显示 deployment 有多少副本数。它遵循 ready/desired 的模式。 
#3.UP-TO-DATE： 显示已更新到所需状态的副本数。 
#4.AVAILABLE： 显示你的可以使用多少个应用程序副本。 
#5.AGE ：显示应用程序已运行的时间。
[root@master deploy]# kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-86f6b6d8c8   2         2         2       2m4s
#创建 deploy 的时候也会创建一个 rs（replicaset），67fd9fc9c8 这个随机数字是我们引用pod 的模板 template 的名字的 hash 值 
#1.NAME： 列出名称空间中 ReplicaSet 资源 2DESIRED：显示应用程序的所需副本数，这些副本数是在创建时定义的。这是所需的状态。 
#3.CURRENT： 显示当前正在运行多少个副本。 
#4.READY： 显示你的用户可以使用多少个应用程序副本。 5.AGE ：显示应用程序已运行的时间。
[root@master deploy]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
myapp-v1-86f6b6d8c8-dpbd7   1/1     Running   0          4m28s   10.244.20.86   node3.linux.com   <none>           <none>
myapp-v1-86f6b6d8c8-r9wlv   1/1     Running   0          4m28s   10.244.83.84   node2.linux.com   <none>           <none>
[root@master deploy]# curl 10.244.20.86\
> ^C
[root@master deploy]# curl 10.244.20.86
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Sample Deployment</title>
  <style>
    body {
      color: #ffffff;
      background-color: blue;	#蓝色背景，部署成功
      font-family: Arial, sans-serif;
      font-size: 14px;
    }
    
    h1 {
      font-size: 500%;
      font-weight: normal;
      margin-bottom: 0;
    }
    
    h2 {
      font-size: 200%;
      font-weight: normal;
      margin-bottom: 0;
    }
  </style>
</head>
<body>
  <div align="center">
    <h1>Welcome to V1 of the web application</h1>
    <h2>This application will be deployed on Kubernetes.</h2>
  </div>
</body>
</html>
```

* 添加存活性探测和就绪性探测

```bash
#两种探测方式都采用了httpGet的方式
[root@master deploy]# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      name: test
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: janakiramm/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            port: 80
        readinessProbe:
          httpGet:
            port: 80
            
#这两种探测方式都是先启动一个pod，两种探测方式都通过之后，Running状态成功，再删除一个pod，依次循环
[root@master deploy]# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
myapp-v1-86f6b6d8c8-dpbd7   1/1     Running   0          27m
myapp-v1-86f6b6d8c8-r9wlv   1/1     Running   0          27m
myapp-v1-c5549578-t4rx4     0/1     Running   0          11s
[root@master deploy]# kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
myapp-v1-86f6b6d8c8-dpbd7   1/1     Terminating         0          27m
myapp-v1-86f6b6d8c8-r9wlv   1/1     Running             0          27m
myapp-v1-c5549578-t4rx4     1/1     Running             0          12s
myapp-v1-c5549578-xtx88     0/1     ContainerCreating   0          0s

#在新生成的pod中查看两种探测
[root@master deploy]# kubectl describe pods myapp-v1-c5549578-t4rx4 | grep Liveness
    Liveness:       http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
[root@master deploy]# kubectl describe pods myapp-v1-c5549578-t4rx4 | grep Readiness
    Readiness:      http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
```

### 5、Deployment 管理 pod：扩容、缩容、滚动更新、回滚

#### pod扩容

```bash
#将deployment.yaml中replicas的数量修改为3，原来是2
[root@master ~]# kubectl get pods -w 
NAME                      READY   STATUS    RESTARTS   AGE
myapp-v1-c5549578-t4rx4   1/1     Running   0          14m
myapp-v1-c5549578-xtx88   1/1     Running   0          13m
myapp-v1-c5549578-qhxrf   0/1     Pending   0          0s	#调度状态，找node去创建
myapp-v1-c5549578-qhxrf   0/1     Pending   0          0s
myapp-v1-c5549578-qhxrf   0/1     ContainerCreating   0          0s	#容器创建状态，正在创建容器中
myapp-v1-c5549578-qhxrf   0/1     ContainerCreating   0          3s
myapp-v1-c5549578-qhxrf   0/1     Running             0          5s	#容器创建成功，Running，正在进行就绪性和存活性探测
myapp-v1-c5549578-qhxrf   1/1     Running             0          14s#探测成功。
```

#### pod缩容

```bash
#将deployment.yaml中replicas的数量修改为2，原来是3
myapp-v1-c5549578-qhxrf   1/1     Terminating         0          3m41s	#容器删除状态
myapp-v1-c5549578-qhxrf   1/1     Terminating         0          3m42s
myapp-v1-c5549578-qhxrf   0/1     Terminating         0          3m43s
myapp-v1-c5549578-qhxrf   0/1     Terminating         0          3m44s
myapp-v1-c5549578-qhxrf   0/1     Terminating         0          3m44s
```

#### pod滚动更新

* 这个就和replicast不一样了，replicast需要我们手动的一个个的去删除，再去自动创建
* 而deployment自动更新

```bash

#将deployment.yaml中的image: janakiramm/myapp:v1的版本修改为janakiramm/myapp:v2
[root@master ~]# kubectl get pods -w 
NAME                      READY   STATUS    RESTARTS   AGE
myapp-v1-c5549578-t4rx4   1/1     Running   0          19m
myapp-v1-c5549578-xtx88   1/1     Running   0          19m
myapp-v1-c5549578-965q4   0/1     Pending   0          0s			#先创建个pod
myapp-v1-c5549578-965q4   0/1     Pending   0          0s
myapp-v1-787b9bc78c-wjr22   0/1     Pending   0          0s		#再来一个pod
myapp-v1-787b9bc78c-wjr22   0/1     Pending   0          0s
myapp-v1-c5549578-965q4     0/1     ContainerCreating   0          0s
myapp-v1-787b9bc78c-wjr22   0/1     ContainerCreating   0          0s		#创建容器
myapp-v1-c5549578-965q4     0/1     ContainerCreating   0          1s
myapp-v1-c5549578-965q4     0/1     Running             0          2s		#进行就绪性探测
myapp-v1-787b9bc78c-wjr22   0/1     ContainerCreating   0          2s
myapp-v1-787b9bc78c-wjr22   0/1     Running             0          4s
myapp-v1-c5549578-965q4     1/1     Running             0          10s
myapp-v1-787b9bc78c-wjr22   1/1     Running             0          12s
myapp-v1-c5549578-965q4     1/1     Terminating         0          12s
myapp-v1-787b9bc78c-7m95v   0/1     Pending             0          0s
myapp-v1-787b9bc78c-7m95v   0/1     Pending             0          0s
myapp-v1-787b9bc78c-7m95v   0/1     ContainerCreating   0          0s
myapp-v1-c5549578-965q4     1/1     Terminating         0          12s
myapp-v1-c5549578-965q4     0/1     Terminating         0          14s
myapp-v1-787b9bc78c-7m95v   0/1     ContainerCreating   0          3s
myapp-v1-787b9bc78c-7m95v   0/1     Running             0          5s
myapp-v1-c5549578-965q4     0/1     Terminating         0          17s
myapp-v1-c5549578-965q4     0/1     Terminating         0          17s
myapp-v1-787b9bc78c-7m95v   1/1     Running             0          10s
myapp-v1-c5549578-xtx88     1/1     Terminating         0          19m
myapp-v1-787b9bc78c-mlg5v   0/1     Pending             0          0s
myapp-v1-787b9bc78c-mlg5v   0/1     Pending             0          0s
myapp-v1-787b9bc78c-mlg5v   0/1     ContainerCreating   0          0s
myapp-v1-c5549578-xtx88     1/1     Terminating         0          19m
myapp-v1-787b9bc78c-mlg5v   0/1     ContainerCreating   0          1s
myapp-v1-c5549578-xtx88     0/1     Terminating         0          19m
myapp-v1-787b9bc78c-mlg5v   0/1     Running             0          2s
myapp-v1-c5549578-xtx88     0/1     Terminating         0          19m
myapp-v1-c5549578-xtx88     0/1     Terminating         0          19m
myapp-v1-787b9bc78c-mlg5v   1/1     Running             0          5s
myapp-v1-c5549578-t4rx4     1/1     Terminating         0          20m
myapp-v1-c5549578-t4rx4     1/1     Terminating         0          20m		#最后再将剩余的三个pod删除
myapp-v1-c5549578-t4rx4     0/1     Terminating         0          20m
myapp-v1-c5549578-t4rx4     0/1     Terminating         0          20m
myapp-v1-c5549578-t4rx4     0/1     Terminating         0          20m

[root@master deploy]# kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-787b9bc78c   3         3         3       11m		#由新的rs接收
myapp-v1-86f6b6d8c8   0         0         0       58m		#原来的deplicast下线
myapp-v1-c5549578     0         0         0       31m
```

* 再次访问可以看到更新后的背景颜色为绿色

```bash
[root@master deploy]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
myapp-v1-787b9bc78c-7m95v   1/1     Running   0          3m50s   10.244.83.88   node2.linux.com   <none>           <none>
myapp-v1-787b9bc78c-mlg5v   1/1     Running   0          3m40s   10.244.20.89   node3.linux.com   <none>           <none>
myapp-v1-787b9bc78c-wjr22   1/1     Running   0          4m2s    10.244.83.87   node2.linux.com   <none>           <none>
[root@master deploy]# curl 10.244.83.88
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Sample Deployment</title>
  <style>
    body {
      color: #ffffff;
      background-color: green;
      font-family: Arial, sans-serif;
      font-size: 14px;
    }
    
    h1 {
      font-size: 500%;
      font-weight: normal;
      margin-bottom: 0;
    }
    
    h2 {
      font-size: 200%;
      font-weight: normal;
      margin-bottom: 0;
    }
  </style>
</head>
<body>
  <div align="center">
    <h1>Welcome to vNext of the web application</h1>
    <h2>This application will be deployed on Kubernetes.</h2>
  </div>
</body>
</html>
```

* 查看滚动更新策略

```bash
[root@master deploy]# kubectl describe deployment myapp-v1
Name:                   myapp-v1
Namespace:              default
CreationTimestamp:      Mon, 19 Sep 2022 10:12:12 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=myapp,version=v1
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge  
#滚动更新策略最多：现有pod数量+0.25*（（现有pod数量）向上取整）=3
#滚动更新策略最少：现有pod数量-0.25*（（现有pod数量）向下取整）=2
```

* 以上这些都是默认的滚动更新策略，现在来自定义更新策略

```bash
[root@master deploy]# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v1
  strategy:
    rollingUpdate:		#自定义滚动更新策略
      maxSurge: 1			#每次最多更新1个，就
      maxUnavailable: 1	#我最多只能有一个不可用，要么就全都可用
  template:
    metadata:
      name: test
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: janakiramm/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - protocol: TCP
          containerPort: 80
        resources:
          limits:
            cpu: 1
            memory: 1Gi
          requests:
            cpu: 0.5
            memory: 1Gi
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 1
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 1
```







#### 回滚

* 有命令不熟悉可以使用`help`参数

```bash
[root@master deploy]# kubectl rollout --help
Manage the rollout of a resource.
  
 Valid resource types include:

  *  deployments
  *  daemonsets
  *  statefulsets

Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc
  
  # Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo

Available Commands:
  history     显示 rollout 历史
  pause       标记提供的 resource 为中止状态
  restart     Restart a resource
  resume      继续一个停止的 resource
  status      显示 rollout 的状态
  undo        撤销上一次的 rollout
[root@master deploy]# kubectl rollout undo --help
Rollback to a previous rollout.

Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc
  
  # Rollback to daemonset revision 3
  kubectl rollout undo daemonset/abc --to-revision=3
  
  # Rollback to the previous deployment with dry-run
  kubectl rollout undo --dry-run=server deployment/abc
[root@master deploy]# kubectl rollout history deployment/myapp-v1		查看可回滚的历史
deployment.apps/myapp-v1 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
[root@master deploy]# kubectl rollout undo deployment/myapp-v1 --to-revision=2	回滚到第二个版本
deployment.apps/myapp-v1 rolled back
[root@master deploy]# kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
myapp-v1-c5549578-9pdsr   1/1     Running   0          28s   10.244.83.89   node2.linux.com   <none>           <none>
myapp-v1-c5549578-s6pwp   1/1     Running   0          20s   10.244.20.91   node3.linux.com   <none>           <none>
myapp-v1-c5549578-sg8st   1/1     Running   0          37s   10.244.20.90   node3.linux.com   <none>           <none>
[root@master deploy]# curl 10.244.83.89
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Sample Deployment</title>
  <style>
    body {
      color: #ffffff;
      background-color: blue;		#已经从绿色回滚到蓝色了
      font-family: Arial, sans-serif;
      font-size: 14px;
    }6、定义Pod资源配额
```

### 6、定义Pod资源配额

* 可用于生产环境

```bash
[root@master deploy]# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
  namespace: ms
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      name: test
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: janakiramm/myapp:v2
        imagePullPolicy: IfNotPresent
        ports:
        - protocol: TCP
          containerPort: 80
        resources:		#新加了资源限额
          limits:
            cpu: 1		#这个pod，最多使用一个cpu
            memory: 1Gi	#最多使用一G内存
          requests:		#在调度的时候，node节点上至少有0.5核cpu。1G内存
             cpu: 0.5
            memory: 1Gi
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
```

