[toc]

## 一、Kubernetes核心资源Pod介绍

**K8s 官方文档：****https://kubernetes.io/** 

**K8s 中文官方文档：** **https://kubernetes.io/zh/** 

**K8s Github 地址：****https://github.com/kubernetes/** 

### 1、Pod是什么?

> * **Pod 是 Kubernetes 中的最小调度单元，k8s 是通过定义一个 Pod 的资源，然后在 Pod 里面运行容器，容器需要指定一个镜像，这样就可以用来运行具体的服务。一个 Pod 封装一个容器（也可以封装多个容器），Pod 里的容器共享存储、网络等。也就是说，应该把整个 pod 看作虚拟机，然后每个容器相当于运行在虚拟机的进程。** 

![image-20220817194528915](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220817194528915.png)

* Pod 是需要调度到 k8s 集群的工作节点来运行的，具体调度到哪个节点，是根据 `scheduler `调度器实现的。 
* pod 相当于一个逻辑主机--比方说我们想要部署一个 tomcat 应用，如果不用容器，我们可能会部署到物理机、虚拟机或者云主机上，那么出现 k8s 之后，我们就可以定义一个 pod 资源，在 pod 里定义一个把 tomcat 容器，所以 pod 充当的是一个逻辑主机的角色。 

#### 1.1、Pod如何管理多个容器

![image-20220817195359005](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220817195359005.png)

> Pod 中可以同时运行多个容器。同一个 Pod 中的容器会自动的分配到同一个 node 上。同一个 Pod 中的容器共享资源、网络环境，它们总是被同时调度，在一个 Pod 中同时运行多个容器是一种比较高级的用法，只有当你的容器需要紧密配合协作的时候才考虑用这种模式。例如，你有一个容器作为 web 服务器运行，需要用到共享的 volume，有另一个“sidecar”容器来从远端获取资源更新这些文件。 

* **一些 Pod 有 init 容器和应用容器。 在应用程序容器启动之前，运行初始化容器。** 

#### 1.2、Pod网络

* Pod 是有 IP 地址的，每个 pod 都被分配`唯一的 IP 地址`（IP 地址是靠网络插件 calico、flannel、weave 等分配的），POD 中的容器`共享网络名称空间`，包括 `IP 地址和网络端口`。 Pod 内部的容器可以使用 localhost 相互通信。Pod 中的容器也可以通过`网络插件 calico 与其他节点的 Pod 通信`。 

### 2、Pod工作方式

> 在 K8s 中，所有的资源都可以使用一个` yaml 文件`来创建，创建 Pod 也可以使用 yaml 配置文件。或者使用 kubectl run 在命令行创建 Pod（**不常用**）。

#### 2.1、自主式Pod

* 所谓的自主式 Pod，就是直接定义一个 Pod 资源，如下：

```yaml
[root@master1 ~]# vim pod-tomcat.yaml 
apiVersion: v1 
kind: Pod 
metadata: 
 name: tomcat-test 
 namespace: default 
 labels: 
   app:  tomcat 
spec: 
 containers: 
 - name:  tomcat-java 
   ports: 
   - containerPort: 8080 
   image: xianchao/tomcat-8.5-jre8:v1 
   imagePullPolicy: IfNotPresent 
```

> * 把 `xianchao-tomcat.tar.gz `上传到 xianchaonode1 和 xianchaonode2 节点，手动解压： 

```bash
#导入镜像 
[root@xianchaonode1 ~]# docker load -i xianchao-tomcat.tar.gz 
[root@xianchaonode2 ~]# docker load -i xianchao-tomcat.tar.gz 
```

![image-20220817200943801](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220817200943801.png)

```bash
[root@master1 ~]# kubectl apply -f pod-tomcat.yaml 
#查看 pod 是否创建成功 
[root@master1 ~]# kubectl get pods -o wide -l app=tomcat 
NAME          READY   STATUS       IP              NODE 
tomcat-test      1/1        Running   10.244.121.45   xianchaonode1 
但是自主式 Pod 是存在一个问题的，假如我们不小心删除了 pod： 
[root@master1 ~]# kubectl delete pods tomcat-test 
#查看 pod 是否还在 
[root@master1 ~]# kubectl get pods -l app=tomcat 
#结果是空，说明 pod 已经被删除了 
```

> 通过上面可以看到，如果**直接定义一个 Pod 资源**，那 Pod 被删除，就彻底被删除了，不会再创建一个新的 Pod，这在生产环境还是具有非常大风险的，所以今后我们接触的 Pod，都是`控制器`管理的。 

#### 2.2、Deployment控制的Pod

> * 常见的管理 Pod 的控制器：`Replicaset`、`Deployment`、`Job`、`CronJob`、`Daemonset`、`Statefulset`。 控制器管理的 Pod 可以确保 Pod 始终维持在指定的副本数运行。 如，通过 Deployment 管理 Pod 

> 把 `xianchao-nginx.tar.gz` 上传到 xianchaonode1 和 xianchaonode2 节点 

```bash
#解压镜像： 
[root@xianchaonode1 ~]# docker load -i xianchao-nginx.tar.gz 
[root@xianchaonode2 ~]# docker load -i xianchao-nginx.tar.gz 
```

* 创建一个资源清单文件

```yaml
[root@master ~]# cat nginx-deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  labels:
    app: nginx-deploy
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: my-nginx
        image: xianchao/nginx:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

```bash
#更新资源清单文件 
[root@master1 ~]# kubectl apply -f nginx-deploy.yaml 
#查看 Deployment 
[root@master1 ~]# kubectl get deploy -l app=nginx-deploy 
NAME         READY   UP-TO-DATE   AVAILABLE   AGE 
nginx-test       2/2        2            2           16s 
#查看 Replicaset 
[root@master1 ~]# kubectl get rs -l app=nginx 
NAME                    DESIRED   CURRENT   READY   AGE 
nginx-test-75c685fdb7   2         2         2       71s 
#查看 pod 
[root@master1 ~]# kubectl get pods -o wide -l app=nginx 
NAME                           READY   STATUS        IP 
nginx-test-75c685fdb7-6d4lx    1/1     Running       10.244.102.69 
nginx-test-75c685fdb7-9s95h    1/1     Running       10.244.102.68 
#删除 nginx-test-75c685fdb7-9s95h 这个 pod 
[root@master1 ~]# kubectl delete pods nginx-test-75c685fdb7-9s95h 
[root@master1 ~]# kubectl get pods -o wide -l app=nginx 
NAME                           READY   STATUS              IP 
nginx-test-75c685fdb7-6d4lx    1/1       Running            10.244.102.69 
nginx-test-75c685fdb7-pr8gh    1/1      Running             10.244.102.70 
#发现重新创建一个新的 pod 是 nginx-test-75c685fdb7-pr8gh 
通过上面可以发现通过 deployment 管理的 pod，可以确保 pod 始终维持在指定副本数量 
```

### 3、如何创建一个Pod

#### 3.1、K8s创建Pod流程

![image-20220817201947604](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220817201947604.png)

> ` Pod`是 Kubernetes 中最基本的部署调度单元，可以包含 container，逻辑上表示某种应用的一个实例。例如一个 web 站点应用由前端、后端及数据库构建而成，这三个组件将运行在各自的容器中，那么我们可以创建包含三个 container 的 pod。 

* 创建 pod 流程：

![image-20220817202211365](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220817202211365.png)

> * master 节点：`kubectl `-> `kube-api` -> `kubelet `->` CRI 容器环境初始化` 

> * 第一步： 
>   * 客户端提交创建 `Pod `的请求，可以通过调用 API Server 的 `Rest API `接口，也可以通过 kubectl 命令行工具。如 kubectl apply -f filename.yaml(资源清单文件) 
> * 第二步： 
>   * apiserver 接收到 `pod 创建请求`后，会将 yaml 中的属性信息(metadata)写入 etcd。 
> * 第三步：
>   *  apiserver 触发 `watch 机制`准备创建 pod，信息转发给`调度器 scheduler`，调度器使用调度算法`选择node`，调度器将 node 信息给 apiserver，apiserver 将绑定的 node 信息写入 etcd 
>
> 调度器用一组规则过滤掉不符合要求的主机。比如 Pod 指定了所需要的资源量，那么可用资源比 Pod 需要的资源量少的主机会被过滤掉。 
>
> scheduler 查看 k8s api ，类似于通知机制。 
>
> * 首先判断：`pod.spec.Node == null? `
>   * 若为 null，表示这个 Pod 请求是新来的，需要创建；
>   * 因此先进行调度计算，找到最“闲”的 node。 
>   * 然后将信息在 etcd 数据库中更新分配结果：pod.spec.Node = nodeA (设置一个具体的节点) 
>
> ps:**同样上述操作的各种信息也要写到 etcd 数据库中中** 
>
> * 第四步： 
>   * apiserver 又通过 watch 机制，调用 kubelet，指定 pod 信息，调用 Docker API 创建并启动 pod 内的容器。 
> * 第五步： 
>   * 创建完成之后反馈给 kubelet, kubelet 又将 pod 的状态信息给 apiserver, apiserver 又将 pod 的状态信息写入 etcd。 

### 4、资源清单 YAML 文件书写技巧 

* 如下

```yaml
[root@master1 ~]# vim pod-tomcat.yaml 
apiVersion: v1  #api 版本 可以通过kubectl api-versions查看所有的版本
kind: Pod       #创建的资源 
metadata: 
  name: tomcat-test  #Pod 的名字 
  namespace: default   #Pod 所在的名称空间 
  labels: 
  	app:  tomcat     #Pod 具有的标签 
spec: 
 containers: 
 - name:  tomcat-java   #Pod 里容器的名字 
   ports: 
   - containerPort: 8080  #容器暴露的端口 
     image: xianchao/tomcat-8.5-jre8:v1  #容器使用的镜像 
     imagePullPolicy: IfNotPresent    #镜像拉取策略 
```

#### 4.1、更新资源清单文件 

```bash
[root@master1 ~]# kubectl apply -f pod-tomcat.yaml 
```

#### 4.2、Pod 资源清单编写技巧 

```bash
#通过 kubectl explain 查看定义 Pod 资源包含哪些字段。 
[root@master1 ~]# kubectl explain pod 
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.
#[Pod 是可以在主机上运行的容器的集合。此资源是由客户端创建并安排到主机上。] 

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
#[APIVersion 定义了对象,代表了一个版本。] 

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
#[Kind 是字符串类型的值，代表了要创建的资源。服务器可以从客户端提交的请求推断出这个资源。] 

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
#[metadata 是对象，定义元数据属性信息的] 

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
#[spec 制定了定义 Pod 的规格，里面包含容器的信息] 

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
#[status 表示状态，这个不可以修改，定义 pod 的时候也不需要定义这个字段] 

也可以通过
[root@master ~]# kubectl explain pod.metadata 来查看pod下的某一个资源如何定义
```

> * 如果某个资源后是`<string>`则说明该资源对象是个列表，
>
>   * 直接在`:`后写值即可
>
> * 如果某个资源后是`<Object>`则说明该资源对象是个对象，
>
>   * 它下面会有字段，多个字段直接`换行空2格`就可以
>
> * 如果某个资源后是`<[]Object>`则说明该资源对象是个对象列表，
>
>   * 它下面也会有很多的对象列表，多个对象列表`换行`+`- 对象`
>
> * 如果某个资源后是`-required-`则说明该资源对象必须要加，
>
> * 如果某个资源后是`<map[string]string>`则说明该资源对象的值是一个键值对，
>
>   * ```json
>     "metadata": { 
>      "annotations": { 
>      "key1" : "value1", 
>      "key2" : "value2" 
>      } 
>     } 
>     ```

#### 4.3、更新资源清单文件 

`[root@master1 ~]# kubectl apply -f pod-first.yaml` 

查看 pod 是否创建成功 

```bash
[root@master1 ~]# kubectl get pods -o wide -l app= tomcat-pod-first 

NAME      READY  STATUS    IP        NODE 

pod-first    1/1     Running  10.244.121.45  xianchaonode1 
```

#### 4.4、查看 pod 日志 

```bash
kubectl logs pod-first 
```

#### 4.5、查看 pod 里指定容器的日志 

```bash
kubectl logs pod-first  -c tomcat-first 
```

#### 4.6、进入到刚才创建的 pod，刚才创建的 pod 名字是 web 

```bash
kubectl exec -it pod-first  -- /bin/bash 
```

#### 4.7、假如 pod 里有多个容器，进入到 pod 里的指定容器，按如下命令： 

```bash
kubectl exec -it pod-first  -c  tomcat-first -- /bin/bash 
```

> 我们上面创建的 pod 是一个自主式 pod，也就是通过 pod 创建一个应用程序，如果 pod 出现故障停掉，那么我们通过 pod 部署的应用也就会停掉，不安全， 还有一种控制器管理的 pod，通过控制器创建pod，可以对 pod 的生命周期做管理，可以定义 pod 的副本数，如果有一个 pod 意外停掉，那么会自动起来一个 pod 替代之前的 pod，之后会讲解 pod 的控制器 

#### 4.8、通过 kubectl run 创建 Pod 

```bash
kubectl run tomcat --image=xianchao/tomcat-8.5-jre8:v1  --image-pull-policy='IfNotPresent'  --port=8080 
```

### 5、命名空间 

#### 5.1、什么是命名空间？ 

> * Kubernetes 支持多个虚拟集群，它们底层依赖于同一个物理集群。 这些虚拟集群被称为命名空间。 
> * 命名空间 namespace 是 k8s 集群级别的资源，可以给不同的用户、租户、环境或项目创建对应的命名空间，例如，可以为 test、devlopment、production 环境分别创建各自的命名空间。 

#### 5.2、namespace 应用场景 

> * 命名空间适用于存在很多跨多个团队或项目的用户的场景。对于只有几到几十个用户的集群，根本不需要创建或考虑命名空间。 
> * 查看名称空间及其资源对象 
>   * k8s 集群默认提供了几个名称空间用于特定目的，例如，`kube-system `主要用于运行系统级资源，存放 k8s 一些组件的。
>   * 而 default 则为那些未指定名称空间的资源操作提供一个默认值。 
>   * 使用 `kubectl get namespace `可以查看 namespace 资源，
>   * 使用 `kubectl describe namespace $NAME` 可以查看特定的名称空间的详细信息。 
> * 管理 namespace 资源 
>   * namespace 资源属性较少，通常只需要指定名称即可创建，如`kubectl create namespace qa`。
>   * namespace 资源的名称仅能由字母、数字、下划线、连接线等字符组成。删除 namespace 资源会级联删除其包含的所有其他资源对象。 

#### 5.3、namespacs 使用案例分享

* 创建一个 test 命名空间 

```bash
[root@master1~]# kubectl create ns test 
```

* 切换命名空间

```bash
[root@master ~]# kubectl  config set-context --current --namespace=kube-system 
Context "kubernetes-admin@kubernetes" modified.
#在切换了命名空间之后，get pods就不需要制定命名空间了
[root@master ~]# kubectl get pods
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6949477b58-zcpc5   1/1     Running   0          7d5h
calico-node-8b6kx                          1/1     Running   0          7d5h
calico-node-kwq5m                          1/1     Running   0          7d5h
calico-node-s9blj                          1/1     Running   0          7d5h
calico-node-tmwwh                          1/1     Running   0          7d5h
coredns-7f89b7bc75-57qzl                   1/1     Running   0          7d5h
coredns-7f89b7bc75-vqqq4                   1/1     Running   0          7d5h
etcd-master.linux.com                      1/1     Running   0          7d5h
kube-apiserver-master.linux.com            1/1     Running   0          7d1h
kube-controller-manager-master.linux.com   1/1     Running   0          7d
kube-proxy-bbw4z                           1/1     Running   0          7d5h
kube-proxy-j4vbp                           1/1     Running   0          7d5h
kube-proxy-kmcjg                           1/1     Running   0          7d5h
kube-proxy-mzfgf                           1/1     Running   0          7d5h
kube-scheduler-master.linux.com            1/1     Running   0          7d
metrics-server-6595f875d6-wbgjs            2/2     Running   0          7d1h
#切换命名空间后，kubectl get pods 如果不指定-n，查看的就是 kube-system 命名空间的资源了。 
```

* 查看哪些资源属于命名空间级别的 

```bash
[root@master1~]# kubectl api-resources --namespaced=true 
#就是说下面的资源在定义的时候都必须要指定命名空间
NAME                        SHORTNAMES   APIVERSION                     NAMESPACED   KIND
bindings                                 v1                             true         Binding
configmaps                  cm           v1                             true         ConfigMap
endpoints                   ep           v1                             true         Endpoints
events                      ev           v1                             true         Event
limitranges                 limits       v1                             true         LimitRange
persistentvolumeclaims      pvc          v1                             true         PersistentVolumeClaim
pods                        po           v1                             true         Pod
podtemplates                             v1                             true         PodTemplate
replicationcontrollers      rc           v1                             true         ReplicationController
resourcequotas              quota        v1                             true         ResourceQuota
secrets                                  v1                             true         Secret
serviceaccounts             sa           v1                             true         ServiceAccount
services                    svc          v1                             true         Service
controllerrevisions                      apps/v1                        true         ControllerRevision
daemonsets                  ds           apps/v1                        true         DaemonSet
deployments                 deploy       apps/v1                        true         Deployment
replicasets                 rs           apps/v1                        true         ReplicaSet
statefulsets                sts          apps/v1                        true         StatefulSet
localsubjectaccessreviews                authorization.k8s.io/v1        true         LocalSubjectAccessReview
horizontalpodautoscalers    hpa          autoscaling/v1                 true         HorizontalPodAutoscaler
cronjobs                    cj           batch/v1beta1                  true         CronJob
jobs                                     batch/v1                       true         Job
leases                                   coordination.k8s.io/v1         true         Lease
networkpolicies                          crd.projectcalico.org/v1       true         NetworkPolicy
networksets                              crd.projectcalico.org/v1       true         NetworkSet
endpointslices                           discovery.k8s.io/v1beta1       true         EndpointSlice
events                      ev           events.k8s.io/v1               true         Event
ingresses                   ing          extensions/v1beta1             true         Ingress
pods                                     metrics.k8s.io/v1beta1         true         PodMetrics
ingresses                   ing          networking.k8s.io/v1           true         Ingress
networkpolicies             netpol       networking.k8s.io/v1           true         NetworkPolicy
poddisruptionbudgets        pdb          policy/v1beta1                 true         PodDisruptionBudget
rolebindings                             rbac.authorization.k8s.io/v1   true         RoleBinding
roles                                    rbac.authorization.k8s.io/v1   true         Role
```

