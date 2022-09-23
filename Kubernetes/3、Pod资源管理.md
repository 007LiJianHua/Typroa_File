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

#### 5.4、namespace 资源限额 

> * namespace 是命名空间，里面有很多资源，那么我们可以对`命名空间资源`做个限制，防止该命名空间部署的资源超过限制。

```yaml
[root@master ~]# cat namespace-quota.yaml 
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
  namespace: test
spec:
  hard:
    requests.cpu: "2"		#分配的CPU和内存
    requests.memory: 2Gi
    limits.cpu: "4"		#最大限制
    limits.memory: 4Gi
```

> * 创建的 ResourceQuota 对象将在 test 名字空间中添加以下限制： 
>   * 每个容器必须设置内存请求（memory request），内存限额（memory limit），
>   * cpu 请求（cpu request）和 cpu 限额（cpu limit）。 
>
> * 所有容器的内存请求总额不得超过 2GiB。 
> *  所有容器的内存限额总额不得超过 4 GiB。
> *  所有容器的 CPU 请求总额不得超过 2 CPU。 
> *  所有容器的 CPU 限额总额不得超过 4CPU。 

* 创建 pod 时候必须设置资源限额，否则创建失败，如下： 

```yaml
[root@master1 ~]# vim pod-test.yaml 
apiVersion: v1 
kind: Pod 
metadata: 
  name: pod-test 
  namespace: test 
  labels: 
    app: tomcat-pod-test 
spec: 
  containers: 
  - name:  tomcat-test 
  ports: 
  - containerPort: 8080 
  image: xianchao/tomcat-8.5-jre8:v1 
  imagePullPolicy: IfNotPresent 
  resources: 
    requests: 
      memory: "100Mi" 
      cpu: "500m" 
    limits: 
      memory: "2Gi" 
      cpu: "2" 
```

### 6、标签

#### 6.1、什么是标签？ 

> 标签其实就一对 `key/value` ，被关联到对象上，比如 Pod,标签的使用我们倾向于能够表示对象的特殊特点，就是一眼就看出了这个 Pod 是干什么的，**标签可以用来划分特定的对象**（比如版本，服务类型等），标签可以在创建一个对象的时候直接定义，也可以在后期随时修改，每一个对象可以拥有多个标签，但是，key 值必须是唯一的。创建标签之后也可以方便我们对资源进行分组管理。如果对 pod 打标签，之后就可以使用标签来查看、删除指定的 pod。 

#### 6.2、 给 pod 资源打标签 

* 对已经存在的资源打标签

```bash
[root@master1~]# kubectl label pods pod-first  release=v1 
```

* 查看标签是否打成功

```bash
[root@master1~]# kubectl get pods pod-first --show-labels 
#显示如下,显示如下，说明标签达成功了； 
NAME         READY   STATUS    RESTARTS   AGE   LABELS 
pod-first    1/1     Running   1          21h   release=v1, app=tomcat-pod-first 
```

#### 6.3、查看资源标签 

* 查看默认名称空间下所有 pod 资源的标签 

```bash
[root@master1~]# kubectl get pods --show-labels 
```

* 查看默认名称空间下指定 pod 具有的所有标签 

```bash
[root@master1~]# kubectl get pods pod-first --show-labels 
```

* 列出默认名称空间下标签 key 是 release 的 pod，不显示标签 

```bash
[root@master1~]# kubectl get pods -l release 
```

* 列出默认名称空间下标签 key 是 release、值是 v1 的 pod，不显示标签 

```bash
[root@master1~]# kubectl get pods -l release=v1 
```

* 列出默认名称空间下标签 key 是 release 的所有 pod，并打印对应的标签值 

```bash
[root@master1~]# kubectl get pods -L release 
```

* 查看所有名称空间下的所有 pod 的标签 

```bash
[root@master1 ~]# kubectl get pods --all-namespaces --show-labels 
```

### 7、node 节点选择器 

> 我们在创建 pod 资源的时候，pod 会根据 schduler 进行调度，那么默认会调度到随机的一个工作节点，如果我们想要 pod 调度到指定节点或者调度到一些具有相同特点的 node 节点，怎么办呢？ 
>
> 可以使用 pod 中的` nodeName `或者` nodeSelector` 字段指定要调度到的 node 节点 

#### 7.1、nodeName： 

* 把 tomcat.tar.gz 上传到node1 和 node2，手动解压： 

```bash
[root@node1 ~]# docker load -i tomcat.tar.gz 
Loaded image: tomcat:8.5-jre8-alpine 
[root@node2 ~]# docker load -i tomcat.tar.gz 
Loaded image: tomcat:8.5-jre8-alpine 
```

* 把 busybox.tar.gz 上传到 node1 和 node2，手动解压： 

```bash
[root@node1 ~]# docker load -i busybox.tar.gz 
[root@node2 ~]# docker load -i busybox.tar.gz 
```

* 编写pod-node.yaml

```bash
[root@master ~]# cat pod-node.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  namespace: default
  labels:
    app: myapp
    env: dev
spec:
  nodeName: node2.linux.com
  containers:
  - name:  tomcat-pod-java
    ports:
    - containerPort: 8080
    image: tomcat:8.5-jre8-alpine
    imagePullPolicy: IfNotPresent
  - name: busybox
    image: busybox:latest
    command:
    - "/bin/sh"
    - "-c"
    - "sleep 3600"
    
[root@master ~]# kubectl apply -f pod-node.yaml 
pod/demo-pod created
```

```bash
#查看 pod 调度到哪个节点 
[root@master1 ~]# kubectl get pods  -o wide 
NAME             READY   STATUS    RESTARTS 
demo-pod        1/1     Running   0            node1 
```

#### 7.2、nodeSelector： 

> 指定 pod 调度到具有哪些标签的 node 节点上 

* 给 node 节点打标签，打个具有 disk=ceph 的标签 

```bash
[root@master1 ~]# kubectl label nodes node2 disk=ceph 
node/node2 labeled 
```

* 定义 pod 的时候指定要调度到具有 disk=ceph 标签的 node 上

```yaml
[root@master ~]# cat pod-1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod-1
  namespace: default
  labels:
    app: myapp
    env: dev
spec:
  nodeSelector:
    disk: ceph
  containers:
  - name:  tomcat-pod-java
    ports:
    - containerPort: 8080
    image: tomcat:8.5-jre8-alpine
    imagePullPolicy: IfNotPresent
```

* 查看pod调度到哪个节点

```bash
[root@master1 ~]# kubectl apply -f pod-1.yaml 
[root@master1 ~]# kubectl get pods  -o wide 
NAME             READY   STATUS    RESTARTS 
demo-pod-1        1/1     Running   0            node2 
```

### 8、亲和性、污点和容忍度

#### 8.1、node 节点亲和性 

> `node 节点亲和性调度：nodeAffinity `

* 可以通过`explain`来查看yaml语法

```bash
[root@master node]# kubectl explain pods.spec.affinity 
KIND:     Pod
VERSION:  v1

RESOURCE: affinity <Object>

DESCRIPTION:
     If specified, the pod's scheduling constraints

     Affinity is a group of affinity scheduling rules.

FIELDS:
   nodeAffinity <Object>
     Describes node affinity scheduling rules for the pod.

   podAffinity  <Object>
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).

   podAntiAffinity      <Object>
     Describes pod anti-affinity scheduling rules (e.g. avoid putting this pod
     in the same node, zone, etc. as some other pod(s)).
[root@master1 ~]#  kubectl explain  pods.spec.affinity.nodeAffinity 
KIND:     Pod
VERSION:  v1

RESOURCE: nodeAffinity <Object>
FIELDS:
   preferredDuringSchedulingIgnoredDuringExecution      <[]Object>  # prefered 表示有节点尽量满足这个位置定义的亲和性，这不是一个必须的条件，软亲和性 

   requiredDuringSchedulingIgnoredDuringExecution       <Object>	#require 表示必须有节点满足这个位置定义的亲和性，这是个硬性条件，硬亲和性 
```

* 例 1：使用 `requiredDuringSchedulingIgnoredDuringExecution `硬亲和性 

* 把 myapp-v1.tar.gz 上传到 node2 和 node1 上，手动解压： 

```bash
[root@node1 ~]# docker load -i myapp-v1.tar.gz 
Loaded image: ikubernetes/myapp:v1 
[root@node2 ~]# docker load -i myapp-v1.tar.gz 
Loaded image: ikubernetes/myapp:v1 
```

* 查看yaml文件

```yaml
[root@master node]# cat pod-nodeaffinity-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
        name: pod-node-affinity-demo
        namespace: default
        labels:
            app: myapp
            tier: frontend
spec:
    containers:
    - name: myapp
      image: ikubernetes/myapp:v1
    affinity:
         nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                   nodeSelectorTerms:
                   - matchExpressions:
                     - key: zone		#必须满足key是zone
                       operator: In		#In表示=
                       values:
                       - foo		#value为foo或者bar
                       - bar
```

* 我们检查当前节点中有任意一个节点拥有` zone 标签`的`值是 foo 或者 bar`，就可以把 pod 调度到这个 node 节点的 foo 或者 bar 标签上的节点上 

```bash
[root@master1 ~]# kubectl apply -f pod-nodeaffinity-demo.yaml 
[root@master node]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
demo-pod                      2/2     Running   1          61m
demo-pod-1                    1/1     Running   0          47m
nginx-7f466444dc-f55dn        1/1     Running   0          9d
nginx-7f466444dc-zxpfp        1/1     Running   0          9d
nginx-test-75c685fdb7-njjm2   1/1     Running   0          7d
nginx-test-75c685fdb7-tm98h   1/1     Running   0          7d
pod-node-affinity-demo        0/1     Pending   0          7s
tomcat-test                   1/1     Running   0          7d1h
#发现是pending挂起状态，因为没有一个拥有 zone 的标签的值是 foo 或者 bar，而且使用的是硬亲和性，必须满足条件才能完成调度 
```

* 给这个 node1 节点打上标签 zone=foo，在查看 

```bash
[root@master1 ~]# kubectl label nodes node1 zone=foo 
[root@master1 ~]# kubectl get pods -o wide 显示如下： 
pod-node-affinity-demo             1/1     Running  0   node1 
```

* 例 2：使用` preferredDuringSchedulingIgnoredDuringExecution `软亲和性 

```yaml
[root@master node]# cat pod-nodeaffinity-demo-2.yaml 
apiVersion: v1
kind: Pod
metadata:
        name: pod-node-affinity-demo-2
        namespace: default
        labels:
            app: myapp
            tier: frontend
spec:
    containers:
    - name: myapp
      image: ikubernetes/myapp:v1
    affinity:
        nodeAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - preference:
               matchExpressions:
               - key: zone1
                 operator: In
                 values:
                 - foo1
                 - bar1
              weight: 60
             
```

```bash
[root@master1 ~]# kubectl apply -f pod-nodeaffinity-demo-2.yaml 
[root@master1 ~]# kubectl get pods -o wide |grep demo-2 
pod-node-affinity-demo-2           1/1     Running     0        node1 
#上面说明软亲和性是可以运行这个 pod 的，尽管没有运行这个 pod 的节点定义的 zone1 标签 Node 节点亲和性针对的是 pod 和 node 的关系，Pod 调度到 node 节点的时候匹配的条件 
```

#### 8.2、Pod 节点亲和性 

* pod 自身的亲和性调度有两种表示形式

> * `podaffinity`：pod 和 pod 更倾向腻在一起，把相近的 pod 结合到相近的位置，如同一区域，同一机架，这样的话 pod 和 pod 之间更好通信，比方说有两个机房，这两个机房部署的集群有 1000 台主机，那么我们希望把 nginx 和 tomcat 都部署同一个地方的 node 节点上，可以提高通信效率； 
>
> * `podunaffinity`：pod 和 pod 更倾向不腻在一起，如果部署两套程序，那么这两套程序更倾向于反亲和性，这样相互之间不会有影响。 
> * 第一个 pod 随机选则一个节点，做为评判后续的 pod 能否到达这个 pod 所在的节点上的运行方式，这就称为 **pod 亲和性**；
> * 我们怎么判定哪些节点是相同位置的，哪些节点是不同位置的；我们在定义 pod 亲和性时需要有一个前提，哪些 pod 在同一个位置，哪些 pod 不在同一个位置，这个位置是怎么定义的，标准是什么？
> * **以节点名称为标准**，这个节点名称相同的表示是同一个位置，节点名称不相同的表示不是一个位置，说白了就是当前创建pod的机器名字。

```bash
[root@master node]# kubectl explain pods.spec.affinity.podAffinity 
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinity <Object>

FIELDS:
   preferredDuringSchedulingIgnoredDuringExecution      <[]Object>	#软亲和性 

   requiredDuringSchedulingIgnoredDuringExecution       <[]Object>	#硬亲和性 
```

> 位置拓扑的键，这个是必须字段 
>
> 怎么判断是不是同一个位置： 
>
> `rack=rack1` 
>
> `row=row1` 
>
> 使用 rack 的键是同一个位置 
>
> 使用 row 的键是同一个位置 
>
> * labelSelector： 
>   * 我们要判断 pod 跟别的 pod 亲和，跟哪个 pod 亲和，需要靠 labelSelector，通过 labelSelector
>   * 选则一组能作为亲和对象的 pod 资源 
>
> namespace： 
>
> * labelSelector 需要选则一组资源，那么这组资源是在哪个名称空间中呢，通过 namespace 指定，
> * 如果不指定 namespaces，那么就是**当前创建 pod 的名称空间** 

* **例 1：pod 节点亲和性** 

> 定义两个 pod，第一个 pod 做为基准，第二个 pod 跟着它走 

```yaml
[root@master node]# cat pod-required-affinity-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-first
  labels:
    app2: myapp2		#创建的第一个pod具有两个标签
    tier: frontend
spec:
    containers:
    - name: myapp
      image: ikubernetes/myapp:v1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-second	
  labels:
    app: backend	#创建的第二个pod具有和之前pod不一样的标签
    tier: db
spec:
    containers:
    - name: busybox
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: ["sh","-c","sleep 3600"]
    affinity:
      podAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:	#硬亲和性
         - labelSelector:		
              matchExpressions:		#创建标签选择器
              - {key: app2, operator: In, values: ["myapp2"]}	#指定和app2=myapp2硬亲和
           topologyKey: kubernetes.io/hostname		#跟着第一个pod去选择相同节点的机器
```

* 上面表示创建的 pod 必须与拥有 app=myapp 标签的 pod 在一个节点上 

```bash
[root@master1 ~]# kubectl apply -f pod-required-affinity-demo.yaml 
kubectl get pods -o wide 显示如下： 
pod-first              running        node1 
pod-second           running        node1 
```

* 上面说明第一个 pod 调度到哪，第二个 pod 也调度到哪，这就是 pod 节点亲和性 

```bash
[root@master1 ~]# kubectl delete -f pod-required-affinity-demo.yaml 
注意： 
[root@master1 ~]# kubectl get nodes --show-labels 
```

* **例 2：pod 节点反亲和性** 

>  定义两个 pod，第一个 pod 做为基准，第二个 pod 跟它调度节点相反 

```yaml
[root@master node]# cat pod-required-anti-affinity-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-first
  labels:
    app1: myapp1
    tier: frontend
spec:
    containers:
    - name: myapp
      image: ikubernetes/myapp:v1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-second
  labels:
    app: backend
    tier: db
spec:
    containers:
    - name: busybox
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: ["sh","-c","sleep 3600"]
    affinity:
      podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
              matchExpressions:
              - {key: app1, operator: In, values: ["myapp1"]}
           topologyKey: kubernetes.io/hostname
           
```

* 两个POD不在同一个机器上

```bash
[root@master ~]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE    IP               NODE              NOMINATED NODE   READINESS GATES
pod-first                     1/1     Running   0          172m   10.244.83.64     node2.linux.com   <none>           <none>
pod-second                    1/1     Running   2          172m   10.244.83.65     node2.linux.com   <none>           <none>
[root@master node]# kubectl delete -f pod-required-anti-affinity-demo.yaml 
pod "pod-first" deleted
pod "pod-second" deleted
```

* **例 3：换一个 topologykey** 

```bash
#为三个节点全部打上zone=foo的标签，来测试反亲和性，
root@master node]# kubectl label nodes node1.linux.com zone=foo
error: 'zone' already has a value (foo), and --overwrite is false
[root@master node]# kubectl label nodes node2.linux.com zone=foo
node/node2.linux.com labeled
[root@master node]# kubectl label nodes node3.linux.com zone=foo
node/node3.linux.com labeled

[root@master node]# cat test1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-first
  labels:
    app1: myapp1
    tier: frontend
spec:
    containers:
    - name: myapp
      image: ikubernetes/myapp:v1

[root@master node]# cat test2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-second
  labels:
    app: backend
    tier: db
spec:
    containers:
    - name: busybox
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: ["sh","-c","sleep 3600"]
    affinity:
      podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
              matchExpressions:
              - {key: app1, operator: In, values: ["myapp1"]}
           topologyKey: zone
           
           
[root@master node]# kubectl apply -f test1.yaml 
pod/pod-first created
[root@master node]# kubectl apply -f test2.yaml 
pod/pod-second created
[root@master node]# kubectl get pods -owide
NAME                          READY   STATUS              RESTARTS   AGE    IP               NODE              NOMINATED NODE   READINESS GATES
demo-pod-1                    1/1     Running             0          3h7m   10.244.20.67     node3.linux.com   <none>           <none>
nginx-7f466444dc-f55dn        1/1     Running             0          23d    10.244.20.69     node3.linux.com   <none>           <none>
nginx-7f466444dc-zxpfp        1/1     Running             0          23d    10.244.83.68     node2.linux.com   <none>           <none>
nginx-test-75c685fdb7-4x7p8   1/1     Running             0          3h7m   10.244.131.130   node1.linux.com   <none>           <none>
nginx-test-75c685fdb7-w62xt   1/1     Running             0          3h7m   10.244.83.66     node2.linux.com   <none>           <none>
pod-first                     0/1     ContainerCreating   0          7s     <none>           node2.linux.com   <none>           <none>
pod-node-affinity-demo        1/1     Running             0          3h7m   10.244.131.133   node1.linux.com   <none>           <none>
pod-node-affinity-demo-2      1/1     Running             0          3h7m   10.244.20.71     node3.linux.com   <none>           <none>
pod-second                    0/1     Pending             0          3s     <none>           <none>            <none>           <none>
tomcat-test                   1/1     Running             0          3h7m   10.244.20.70     node3.linux.com   <none>           <none>
[root@master node]# kubectl get pods -owide
NAME                          READY   STATUS    RESTARTS   AGE    IP               NODE              NOMINATED NODE   READINESS GATES
demo-pod-1                    1/1     Running   0          3h7m   10.244.20.67     node3.linux.com   <none>           <none>
nginx-7f466444dc-f55dn        1/1     Running   0          23d    10.244.20.69     node3.linux.com   <none>           <none>
nginx-7f466444dc-zxpfp        1/1     Running   0          23d    10.244.83.68     node2.linux.com   <none>           <none>
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          3h7m   10.244.131.130   node1.linux.com   <none>           <none>
nginx-test-75c685fdb7-w62xt   1/1     Running   0          3h7m   10.244.83.66     node2.linux.com   <none>           <none>
pod-first                     1/1     Running   0          12s    10.244.83.69     node2.linux.com   <none>           <none>
pod-node-affinity-demo        1/1     Running   0          3h7m   10.244.131.133   node1.linux.com   <none>           <none>
pod-node-affinity-demo-2      1/1     Running   0          3h7m   10.244.20.71     node3.linux.com   <none>           <none>
pod-second                    0/1     Pending   0          8s     <none>           <none>            <none>           <none>
tomcat-test                   1/1     Running   0          3h7m   10.244.20.70     node3.linux.com   <none>           <none>
```

> 由此可知，在部署pod的时候，先通过标签选择器选择到具有`app1:myapp1`标签的pod，然后我要与这个pod做反亲和，那么就根`topologyKey: zone`,有合格标签的我都不创建
>
> 所以`pod-second`一直为pending状态

#### 8.3、污点、容忍度 

> **给了节点选则的主动权，我们给节点打一个污点，不容忍的 pod 就运行不上来，污点就是定义在节点上的键值属性数据，可以定决定拒绝那些 pod；**

> `taints` 是键值数据，用在**节点**上，定义污点； 
>
> `tolerations` 是键值数据，用在 **pod** 上，定义容忍度，能容忍哪些污点 
>
> `pod 亲和性`是 pod 属性；但是污点是节点的属性，污点定义在 nodeSelector 上 

* 可以先看master节点的污点：

```bash
[root@master ~]# kubectl describe nodes master.linux.com | grep -i "taints"
Taints:             node-role.kubernetes.io/master:NoSchedule
```

* 而node节点上没有污点
  * 所以我门创建的pod都不会调度到master上，因为我们创建的pod没有容忍度


```bash
[root@master ~]# kubectl describe nodes node1.linux.com | grep -i "taints"
Taints:             <none>
```

* 查看`Taints`的yaml写法

```bash
[root@master ~]# kubectl explain node.spec.taints
KIND:     Node
VERSION:  v1

RESOURCE: taints <[]Object>

DESCRIPTION:
     If specified, the node's taints.

     The node this Taint is attached to has the "effect" on any pod that does
     not tolerate the Taint.

FIELDS:
   effect       <string> -required-
     Required. The effect of the taint on pods that do not tolerate the taint.
     Valid effects are NoSchedule, PreferNoSchedule and NoExecute.

   key  <string> -required-
     Required. The taint key to be applied to a node.

   timeAdded    <string>
     TimeAdded represents the time at which the taint was added. It is only
     written for NoExecute taints.

   value        <string>
     The taint value corresponding to the taint key.
```

taints 的 effect 用来定义对 pod 对象的排斥等级（效果）： 

`NoSchedule`： 

仅影响 pod 调度过程，当 pod 能容忍这个节点污点，就可以调度到当前节点，后来这个节点的污点改了，加了一个新的污点，使得之前调度的 pod 不能容忍了，那现存的pod也不会受到影响

`NoExecute`： 

既影响调度过程，又影响现存的 pod 对象，如果现存的 pod 不能容忍节点后来加的污点，这个 pod就会被驱逐 

`PreferNoSchedule`： 

最好不，也可以，是 NoSchedule 的柔性版本

* 在 pod 对象定义容忍度的时候支持两种操作：
  * 等值密钥：key 和 value 上完全匹配 
  * 存在性判断：key 和 effect 必须同时匹配，value 可以是空

> 在 pod 上定义的容忍度可能不止一个，在节点上定义的污点可能多个，需要琢个检查容忍度和污点能否匹配，每一个污点都能被容忍，才能完成调度，如果不能容忍怎么办，那就需要看 pod 的容忍度了 

例 1：把node2、node3 当成是生产环境专用的，其他 node 是测试的

```bash
[root@master taint]# kubectl taint nodes node3.linux.com node-type:NoExecute-
node/node3.linux.com untainted
[root@master taint]# kubectl taint nodes node2.linux.com node-type:NoExecute-
node/node2.linux.com untainted
```

* 创建测试pod
  * 这个pod没有任何容忍，所以只能调度在node1上

```bash
[root@master taint]# cat pod-taint.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: taint-pod
  namespace: default
  labels:
    tomcat:  tomcat-pod
spec:
  containers:
  - name:  taint-pod
    ports:
    - containerPort: 8080
    image: tomcat:8.5-jre8-alpine
    imagePullPolicy: IfNotPresent 
    
    
[root@master taint]# kubectl apply -f pod-taint.yaml 
pod/taint-pod created
[root@master taint]# kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
demo-pod-1                    1/1     Running             0          5d22h
nginx-7f466444dc-f55dn        1/1     Running             0          28d
nginx-7f466444dc-zxpfp        1/1     Running             0          28d
nginx-test-75c685fdb7-4x7p8   1/1     Running             0          5d22h
nginx-test-75c685fdb7-w62xt   1/1     Running             0          5d22h
pod-node-affinity-demo        1/1     Running             0          5d22h
pod-node-affinity-demo-2      1/1     Running             0          5d22h
taint-pod                     0/1     ContainerCreating   0          4s
tomcat-test                   1/1     Running             0          5d22h
[root@master taint]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP               NODE              NOMINATED NODE   READINESS GATES
demo-pod-1                    1/1     Running   0          5d22h   10.244.20.67     node3.linux.com   <none>           <none>
nginx-7f466444dc-f55dn        1/1     Running   0          28d     10.244.20.69     node3.linux.com   <none>           <none>
nginx-7f466444dc-zxpfp        1/1     Running   0          28d     10.244.83.68     node2.linux.com   <none>           <none>
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          5d22h   10.244.131.130   node1.linux.com   <none>           <none>
nginx-test-75c685fdb7-w62xt   1/1     Running   0          5d22h   10.244.83.66     node2.linux.com   <none>           <none>
pod-node-affinity-demo        1/1     Running   0          5d22h   10.244.131.133   node1.linux.com   <none>           <none>
pod-node-affinity-demo-2      1/1     Running   0          5d22h   10.244.20.71     node3.linux.com   <none>           <none>
taint-pod                     1/1     Running   0          12s     10.244.83.70     node2.linux.com   <none>           <none>
tomcat-test                   1/1     Running   0          5d22h   10.244.20.70     node3.linux.com   <none>           <none>
```

例 2：给node1 也打上污点 

```bash
[root@xianchaomaster1 ~]# kubectl delete -f pod-taint.yaml 
[root@xianchaomaster1 ~]#kubectl taint node xianchaonode1 node-type=dev:NoExecute 
[root@xianchaomaster1 ~]#kubectl get pods -o wide 
显示如下： 
taint-pod termaitering 
上面可以看到已经存在的 pod 节点都被撵走了
```

例三：pod容忍的三种方式

* 匹配`key`和`effect`
  * 就是说，我定义的pod只要有key和effect，他会自动的去找，对应的具有key和effect的节点，找到之后就可以调度。
  * 但是`operator`字段必须为`Exists`
* 单独匹配key也是一样的道理

```bash
[root@master taint]# cat pod-demo-1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: myapp-deploy
  namespace: default
  labels:
    app: myapp
    release: canary
spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v1
        ports:
        - name: http
          containerPort: 80
      tolerations:
      - key: "node-type"
        operator: "Exists"
        value: ""
        effect: "NoSchedule"
[root@master taint]# kubectl get pods 
NAME                          READY   STATUS    RESTARTS   AGE
myapp-deploy                  1/1     Running   0          9s
nginx-7f466444dc-cq7p4        0/1     Pending   0          11m
nginx-7f466444dc-kbb2b        0/1     Pending   0          11m
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          5d22h
nginx-test-75c685fdb7-nvv26   0/1     Pending   0          11m
pod-node-affinity-demo        1/1     Running   0          5d22h
```

* 删除污点
  * 删除污点只需要`key和effect`

```bash
[root@master taint]# kubectl taint nodes node3.linux.com node-type:NoExecute-
node/node3.linux.com untainted
[root@master taint]# kubectl taint nodes node2.linux.com node-type:NoExecute-
node/node2.linux.com untainted
```

### 9、Pod异常处理方案

#### 9.1、Pod常见的状态

> Pod 的 status 定义在 PodStatus 对象中，其中有一个 phase 字段。它简单描述了 Pod 在其生命周期的阶段。熟悉 Pod 的各种状态对我们理解如何设置 Pod 的调度策略、重启策略是很有必要的。下面是 phase 可能的值，也就是 pod 常见的状态：

**挂起（Pending）**：我们在请求创建 pod 时，条件不满足，调度没有完成，没有任何一个节点能满足调度条件，已经创建了 pod 但是没有适合它运行的节点叫做挂起，调度没有完成，处于 `pending`的状态会持续一段时间：包括调度 Pod 的时间和通过网络下载镜像的时间。 

**运行中（Running）**：Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。 

**成功（Succeeded）**：Pod 中的所有容器都被成功终止，并且不会再重启。

**失败（Failed）**：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。

**未知（Unknown）**：未知状态，所谓 pod 是什么状态是 apiserver 和运行在 pod 节点的 kubelet 进行通信获取状态信息的，如果节点之上的 kubelet 本身出故障，那么 apiserver 就连不上kubelet，得不到信息了，就会看 Unknown 

**扩展：还有其他状态，如下：** 

Evicted 状态：出现这种情况，多见于系统内存或硬盘资源不足，可 df-h 查看 docker 存储所在目录的资源使用情况，如果百分比大于 85%，就要及时清理下资源，尤其是一些大文件、docker 镜像。 

`CrashLoopBackOff`：容器曾经启动了，但可能又异常退出了 

`Error` 状态：Pod 启动过程中发生了错误 

#### 9.2、Pod重启策略

> `Pod 的重启策略（RestartPolicy）`应用于 Pod 内的所有容器，并且仅在 Pod 所处的 Node 上由kubelet 进行判断和重启操作。当某个容器异常退出或者健康检查失败时，kubelet 将根据 RestartPolicy 的设置来进行相应的操作。 

**Pod 的重启策略包括 Always、OnFailure 和 Never，默认值为 Always。** 

`Always`：当容器失败时，由 kubelet 自动重启该容器。 

`OnFailure`：当容器终止运行且退出码不为 0 时，由 kubelet 自动重启该容器。 

`Never`：不论容器运行状态如何，kubelet 都不会重启该容器。

```bash
[root@master taint]# cat pod.yaml 
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

#### 9.3、初始化容器

* Pod生命周期

![image-20220913171858969](https://tva1.sinaimg.cn/large/e6c9d24ely1h653lluhmnj21ao0nc0us.jpg)

* Init容器

> Pod 里面可以有一个或者多个容器，部署应用的容器可以称为主容器，在创建 Pod 时候，Pod 中可以有一个或多个先于主容器启动的 Init 容器,这个 init 容器就可以成为初始化容器，初始化容器一旦执行完，它从启动开始到初始化代码执行完就退出了，它不会一直存在，所以在主容器启动之前执行初始化，初始化容器可以有多个，多个初始化容器是要串行执行的，
>
> 先执行初始化容器 1，
>
> 在执行初始化容器 2 等，
>
> 等初始化容器执行完初始化就退出了，然后再执行主容器，
>
> 主容器一退出，pod 就结束了，主容器退出的时间点就是 pod 的结束点，它俩时间轴是一致的； 

> Init 容器就是做初始化工作的容器。可以有一个或多个，如果多个按照定义的顺序依次执行，只有所有的初始化容器执行完后，主容器才启动。
>
> 由于一个 Pod 里的存储卷是共享的，所以 Init Container 里产生的数据可以被主容器使用到
>
> Init Container 可以在多种 K8S 资源里被使用到，如 Deployment、DaemonSet, StatefulSet、Job 等，但都是在 Pod 启动时，在主容器启动前执行，做初始化工作。 

**Init 容器与普通的容器区别是:** 

1、Init 容器不支持 Readiness,因为它们必须在 Pod 就绪之前运行完成 

2、每个 Init 容器必须运行成功,下一个才能够运行 

3、如果 Pod 的 Init 容器失败,Kubernetes 会不断地重启该 Pod,直到 Init 容器成功为止，然而,如果 Pod 对应的 restartPolicy 值为 Never,它不会重新启动。 

#### 9.4、定义初始化容器

* 初始化容器的官方地址

`https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use`

* 先定义一个容器` myapp-container`
  * 在定义两个初始化容器
    * 初始化容器需要使用`nslookuop`工具来解析myservice，所以在apply之后，会一直处于调度的状态

```bash
[root@master ~]# mkdir  init
[root@master ~]# cd init/
[root@master init]# ls
[root@master init]# vim init.yaml
[root@master init]# 
[root@master init]# 
[root@master init]# cat init.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
[root@master init]# kubectl apply -f init.yaml 
pod/myapp-pod created
[root@master init]# kubectl get pods
NAME                          READY   STATUS     RESTARTS   AGE
myapp-deploy                  1/1     Running    0          118m
myapp-pod                     0/1     Init:0/2   0          5s
nginx-7f466444dc-cq7p4        1/1     Running    0          129m
nginx-7f466444dc-kbb2b        1/1     Running    0          129m
nginx-test-75c685fdb7-4x7p8   1/1     Running    0          6d
nginx-test-75c685fdb7-nvv26   1/1     Running    0          129m
pod-node-affinity-demo        1/1     Running    0          6d
```

* 创建service

```bash
[root@master init]# cat services.yaml 
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
[root@master init]# kubectl apply -f services.yaml 
service/myservice created
service/mydb created
[root@master init]# kubectl get pods
NAME                          READY   STATUS     RESTARTS   AGE
myapp-deploy                  1/1     Running    0          119m
myapp-pod                     0/1     Init:1/2   0          99s
nginx-7f466444dc-cq7p4        1/1     Running    0          130m
nginx-7f466444dc-kbb2b        1/1     Running    0          130m
nginx-test-75c685fdb7-4x7p8   1/1     Running    0          6d
nginx-test-75c685fdb7-nvv26   1/1     Running    0          130m
pod-node-affinity-demo        1/1     Running    0          6d
[root@master init]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
myapp-deploy                  1/1     Running   0          119m
myapp-pod                     1/1     Running   0          103s
nginx-7f466444dc-cq7p4        1/1     Running   0          130m
nginx-7f466444dc-kbb2b        1/1     Running   0          130m
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          6d
nginx-test-75c685fdb7-nvv26   1/1     Running   0          130m
pod-node-affinity-demo        1/1     Running   0          6d
```

#### 9.5、主容器和初始化容器的关系

* 容器钩子

> 初始化容器启动之后，开始启动主容器，在主容器启动之前有一个 post start hook（容器启动后钩子）和 pre stop hook（容器结束前钩子），无论启动后还是结束前所做的事我们可以把它放两个钩子，这个钩子就表示用户可以用它来钩住一些命令，来执行它，做开场前的预设，结束前的清理，如 awk 有 begin，end，和这个效果类似；

![image-20220913171858969](https://tva1.sinaimg.cn/large/e6c9d24ely1h653lluhmnj21ao0nc0us.jpg)

**postStart**：该钩子在容器被创建后立刻触发，通知容器它已经被创建。<u>如果该钩子对应的 hookhandler 执行失败，则该容器会被杀死</u>，并根据该容器的重启策略决定是否要重启该容器，这个钩子不需要传递任何参数。 

**preStop**：该钩子在容器被删除前触发，其所对应的 hook handler 必须在删除该容器的请求发送给 Docker daemon 之前完成。在该钩子对应的 hook handler 完成后不论执行的结果如何，<u>Docker daemon 会发送一个 SGTERN 信号量给 Docker daemon 来删除该容器</u>，这个钩子不需要传递任何参数。 

> 在 k8s 中支持两类对 pod 的检测，
>
> 第一类叫做 livenessprobe（pod 存活性探测）： 
>
> 存活探针主要作用是，<u>用指定的方式检测 pod 中的容器应用是否正常运行</u>，如果检测失败，则认为容器不健康，那么 Kubelet 将根据 Pod 中设置的 restartPolicy 来判断 Pod 是否要进行重启操作，**如果容器配置中没有配置 livenessProbe，Kubelet 将认为存活探针探测一直为成功状态**。 
>
> 第二类是状态检 readinessprobe（pod 就绪性探测）：
>
> <u>用于判断容器中应用是否启动完成</u>，当探测成功后才使 Pod 对外提供网络访问，设置容器 Ready 状态为 true，如果探测失败，则设置容器的Ready 状态为 false。 

#### 9.6、创建pod需要经过哪些阶段

> 当用户创建 pod 时，
>
> * 这个请求给 apiserver，apiserver 把创建请求的状态保存在 etcd 中；
> * 接下来 apiserver 会请求 scheduler 来完成调度，如果调度成功，会把调度的结果（如调度到哪个节点上了，运行在哪个节点上了，把它更新到 etcd 的 pod 资源状态中）保存在 etcd 中，
> * 一旦存到 etcd 中并且完成更新以后，如调度到 xianchaonode1 上，那么 xianchaonode1 节点上的kubelet 通过 apiserver 当中的状态变化知道有一些任务被执行了，所以此时此 kubelet 会拿到用户创建时所提交的清单，这个清单会在当前节点上运行或者启动这个 pod，
> * 如果创建成功或者失败会有一个当前状态，当前这个状态会发给 apiserver，
> * apiserver 在存到 etcd 中；
> * 在这个过程中，etcd 和 apiserver 一直在打交道，不停的交互，scheduler 也参与其中，负责调度 pod 到合适的 node 节点上，这个就是 pod 的创建过程 

**pod 在整个生命周期中有非常多的用户行为：** 

1、初始化容器完成初始化 

2、主容器启动后可以做启动后钩子 

3、主容器结束前可以做结束前钩子 

4、在主容器运行中可以做一些健康检测，如 liveness probe，readness probe 

### 10、Pod容器探测和钩子

#### 10.1、容器钩子：postStart和preStop

**postStart：容器创建成功后，运行前的任务，用于资源部署、环境准备等。** 

**preStop：在容器被终止前的任务，用于优雅关闭应用程序、通知其他系统等。** 

* postStart用法

```bash
[root@master ~]# kubectl explain pods.spec.containers.lifecycle.postStart
KIND:     Pod
VERSION:  v1

RESOURCE: postStart <Object>

DESCRIPTION:
     PostStart is called immediately after a container is created. If the
     handler fails, the container is terminated and restarted according to its
     restart policy. Other management of the container blocks until the hook
     completes. More info:
     https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#container-hooks

     Handler defines a specific action that should be taken

FIELDS:
   exec <Object>
     One and only one of the following should be specified. Exec specifies the
     action to take.

   httpGet      <Object>
     HTTPGet specifies the http request to perform.

   tcpSocket    <Object>
     TCPSocket specifies an action involving a TCP port. TCP hooks not yet
     supported
```

```yaml
containers: - image: sample:v2 
 name: war 
 lifecycle： 
 postStart: 
 exec: 
 command: 
 - “cp” 
 - “/sample.war” 
 - “/app”
 prestop: 
 httpGet: 
 host: monitor.com 
 path: /waring 
 port: 8080 
 scheme: HTTP
```

> 以上示例中，定义了一个 Pod，包含一个 JAVA 的 web 应用容器，其中设置了 PostStart 和PreStop 回调函数。即在容器创建成功后，复制/sample.war 到/app 文件夹中。而在容器终止之前，发送 HTTP 请求到 http://monitor.com:8080/waring，即向监控系统发送警告。 

#### 10.2、优雅的删除资源对象

**当用户请求删除含有 pod 的资源对象时（如 RC、deployment 等），K8S 为了让应用程序优雅关闭（即让应用程序完成正在处理的请求后，再关闭软件），K8S 提供两种信息通知：** 

> 1）、默认：K8S 通知 node 执行 docker stop 命令，docker 会先向容器中 PID 为 1 的进程发送系统信号 SIGTERM，然后等待容器中的应用程序终止执行，如果等待时间达到设定的超时时间，或者默认超时时间（30s），会继续发送 SIGKILL 的系统信号强行 kill 掉进程。 

> 2）、使用 pod 生命周期（利用 PreStop 回调函数），它执行在发送终止信号之前。 默认情况下，所有的删除操作的优雅退出时间都在 30 秒以内。kubectl delete 命令支持--grace-period=的选项，以运行用户来修改默认值。0 表示删除立即执行，并且立即从 API 中删除 pod。在节点上，被设置了立即结束的的 pod，**仍然会给一个很短的优雅退出时间段**，才会开始被强制杀死。如下： 

![image-20220913194022998](https://tva1.sinaimg.cn/large/e6c9d24ely1h657opds42j21bu0pimze.jpg)

> 如上所示，在执行`kubectl delete`命令的时候，先执行command，再推出容器

#### 10.3、存活性探测 livenessProbe 和就绪性探测 readinessProbe

* **livenessProbe：存活性探测** 

> 许多应用程序经过长时间运行，最终过渡到无法运行的状态，除了重启，无法恢复。通常情况下，K8S 会发现应用程序已经终止，然后重启应用程序 pod。有时应用程序可能因为某些原因（后端服务故障等）导致暂时无法对外提供服务，但应用软件没有终止，导致 K8S 无法隔离有故障的pod，调用者可能会访问到有故障的 pod，导致业务不稳定。K8S 提供 livenessProbe 来检测容器是否正常运行，并且对相应状况进行相应的补救措施。

* **readinessProbe：就绪性探测** 

> 在没有配置 readinessProbe 的资源对象中，pod 中的容器启动完成后，就认为 pod 中的应用程序可以对外提供服务，该 pod 就会加入相对应的 service，对外提供服务。但`有时一些应用程序启动后，需要较长时间的加载才能对外服务`，如果这时对外提供服务，执行结果必然无法达到预期效果，影响用户体验。比如使用 tomcat 的应用程序来说，并不是简单地说 tomcat 启动成功就可以对外提供服务的，还需要等待 spring 容器初始化，数据库连接上等等。 

* **目前 LivenessProbe 和 ReadinessProbe 两种探针都支持下面三种探测方法：** 

1、ExecAction：在容器中执行指定的命令，如果执行成功，退出码为 0 则探测成功。 

2、TCPSocketAction：通过容器的 IP 地址和端口号执行 TCP 检 查，如果能够建立 TCP 连接，则表明容器健康。 

3、HTTPGetAction：通过容器的 IP 地址、端口号及路径调用 HTTP Get 方法，如果响应的状态码大于等于 200 且小于 400，则认为容器健康 

* **探针探测结果有以下值：** 

1、Success：表示通过检测。 

2、Failure：表示未通过检测。 

3、Unknown：表示检测没有正常进行。 

* **Pod 探针相关的属性：** 

> 探针(Probe)有许多可选字段，可以用来更加精确的控制 Liveness 和 Readiness 两种探针的行为 

* initialDelaySeconds： Pod 启动后首次进行检查的等待时间，单位“秒”。 
* periodSeconds： 检查的间隔时间，默认为 10s，单位“秒”。 
* timeoutSeconds： 探针执行检测请求后，等待响应的超时时间，默认为 1s，单位“秒”。 
* successThreshold：连续探测几次成功，才认为探测成功，默认为 1，在 Liveness 探针中必须为 1，最小值为 1。 
* failureThreshold： 探测失败的重试次数，重试一定次数后将认为失败，在 readiness 探针中，Pod 会被标记为未就绪，默认为 3，最小值为 1 

* **两种探针区别：** 

>  ReadinessProbe 和 livenessProbe 可以使用相同探测方式，只是对 Pod 的处置方式不同： 

readinessProbe 当检测失败后，将 Pod 的 IP:Port 从对应的 EndPoint 列表中删除，就形式nginx的upstream模块不再将请求转发到某个节点

livenessProbe 当检测失败后，将杀死容器并根据 Pod 的重启策略来决定作出对应的措施。 

#### 10.4、Pod探针使用案例 

* **LivenessProbe 探针使用示例**

1）、通过 exec 方式做健康探测 

![image-20220913202357258](https://tva1.sinaimg.cn/large/e6c9d24ely1h658y171olj20u00w1acb.jpg)

容器启动设置执行的命令： 

`/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600" `

> 容器在初始化后，首先创建一个 /tmp/healthy 文件，然后执行睡眠命令，睡眠 30 秒，到时间后执行删除 /tmp/healthy 文件命令。而设置的存活探针检检测方式为执行 shell 命令，用 cat 命令输出 healthy 文件的内容，如果能成功执行这条命令，存活探针就认为探测成功，否则探测失败。在前 30 秒内，由于文件存在，所以存活探针探测时执行 cat /tmp/healthy 命令成功执行。30 秒后 healthy 文件被删除，所以执行命令失败，Kubernetes 会根据 Pod 设置的重启策略来判断，是否重启 Pod。 

2）、通过 HTTP 方式做健康探测 

```bash
[root@master liveness]# cat liveness-http.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
  labels:
    test: liveness
spec:
  containers:
  - name: liveness
    image: mydlqclub/springboot-helloworld:0.0.1
    livenessProbe:
      initialDelaySeconds: 20
      periodSeconds: 5
      timeoutSeconds: 10
      httpGet:
        scheme: HTTP
        port: 8081
        path: /actuator/health
        #在这里没有写host字段，是因为默认就是pod的IP，相当于访问的就是http://pod-ip:8081/actuator/health
[root@master liveness]# vim liveness-http.yaml
[root@master liveness]# kubectl apply -f liveness-http.yaml 
pod/liveness-http created
[root@master liveness]# kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
liveness-http                 0/1     ContainerCreating   0          5s
myapp-deploy                  1/1     Running             0          5h2m
myapp-pod                     1/1     Running             3          3h4m
nginx-7f466444dc-cq7p4        1/1     Running             0          5h13m
nginx-7f466444dc-kbb2b        1/1     Running             0          5h13m
nginx-test-75c685fdb7-4x7p8   1/1     Running             0          6d3h
nginx-test-75c685fdb7-nvv26   1/1     Running             0          5h13m
pod-node-affinity-demo        1/1     Running             0          6d3h
[root@master liveness]# curl -I http://10.244.83.74:8081/actuator/health
HTTP/1.1 200 
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Tue, 13 Sep 2022 12:49:47 GMT
```

* 如果把访问的端口不对，则pod会一直按照重启策略重启，

```bash
[root@master liveness]# cat liveness-http.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
  labels:
    test: liveness
spec:
  containers:
  - name: liveness
    image: mydlqclub/springboot-helloworld:0.0.1
    livenessProbe:
      initialDelaySeconds: 20
      periodSeconds: 5
      timeoutSeconds: 10
      httpGet:
        scheme: HTTP
        port: 8082
        path: /actuator/health
    readinessProbe:
      initialDelaySeconds: 20
      periodSeconds: 5
      timeoutSeconds: 10
      httpGet:
        scheme: HTTP
        port: 8081
        path: /actuator/health
[root@master liveness]# kubectl get pods
\NAME                          READY   STATUS    RESTARTS   AGE
liveness-http                 0/1     Running   0          11s
myapp-deploy                  1/1     Running   0          5h6m
myapp-pod                     1/1     Running   3          3h8m
nginx-7f466444dc-cq7p4        1/1     Running   0          5h17m
nginx-7f466444dc-kbb2b        1/1     Running   0          5h17m
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          6d3h
nginx-test-75c685fdb7-nvv26   1/1     Running   0          5h17m
pod-node-affinity-demo        1/1     Running   0          6d3h
[root@master liveness]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
liveness-http                 0/1     Running   2          90s
myapp-deploy                  1/1     Running   0          5h7m
myapp-pod                     1/1     Running   3          3h9m
nginx-7f466444dc-cq7p4        1/1     Running   0          5h18m
nginx-7f466444dc-kbb2b        1/1     Running   0          5h18m
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          6d3h
nginx-test-75c685fdb7-nvv26   1/1     Running   0          5h18m
pod-node-affinity-demo        1/1     Running   0          6d3h

```

3）、通过TCP方式做健康探测

```bash
[root@master liveness]# cat liveness-tcp.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp
  labels:
    app: liveness
spec:
  containers:
  - name: liveness
    image: nginx
    livenessProbe:
      initialDelaySeconds: 15
      periodSeconds: 20
      tcpSocket:
        port: 80
[root@master liveness]# kubectl apply -f liveness-tcp.yaml ^C
[root@master liveness]# kubectl describe pods liveness-tcp | grep Liveness
    Liveness:       tcp-socket :80 delay=15s timeout=1s period=20s #success=1 #failure=3
[root@master liveness]# 
```

> TCP 检查方式和 HTTP 检查方式非常相似，在容器启动 initialDelaySeconds 参数设定的时间后，kubelet 将发送第一个 livenessProbe 探针，尝试连接容器的 80 端口，如果连接失败则将杀死 Pod 重启容器。

* **readinessProbe 探针使用示例**

> Pod 的 ReadinessProbe 探针使用方式和 LivenessProbe 探针探测方法一样，也是支持三种，只是一个是用于探测应用的存活，一个是判断是否对外提供流量的条件。这里用一个 Springboot 项目，设置 ReadinessProbe 探测 SpringBoot 项目的 8081 端口下的 `/actuator/health` 接口，如果探测成功则代表内部程序以及启动，就开放对外提供接口访问，否则内部应用没有成功启动，暂不对外提供访问，直到就绪探针探测成功。

```bash
[root@master readiness]# cat readiness-http.yaml 
apiVersion: v1 
kind: Service 
metadata: 
  name: springboot 
  labels: 
    app: springboot 
spec: 
  type: NodePort 
  ports: 
  - name: server 
    port: 8080 
    targetPort: 8080 
    nodePort: 31180 
  - name: management 
    port: 8081 
    targetPort: 8081 
    nodePort: 31181 
  selector: 
    app: springboot
---
apiVersion: v1
kind: Pod
metadata:
  name: springboot
  labels:
    app: springboot
spec:
  containers:
  - name: springboot
    image: mydlqclub/springboot-helloworld:0.0.1
    ports:
    - name: server
      containerPort: 8080
    - name: management
      containerPort: 8081
    readinessProbe:
      initialDelaySeconds: 20
      periodSeconds: 5
      timeoutSeconds: 10
      httpGet:
        scheme: HTTP
        port: 8081
        path: /actuator/health

#这里开始创建pod
[root@master readiness]# kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
liveness-tcp                  1/1     Running             0          26m
myapp-deploy                  1/1     Running             0          25h
nginx-7f466444dc-cq7p4        1/1     Running             0          26h
nginx-7f466444dc-kbb2b        1/1     Running             0          26h
nginx-test-75c685fdb7-4x7p8   1/1     Running             0          7d
nginx-test-75c685fdb7-nvv26   1/1     Running             0          26h
pod-node-affinity-demo        1/1     Running             0          7d
springboot                    0/1     ContainerCreating   0          4s
#pod创建成功，开始进行就绪性探测
[root@master readiness]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
liveness-tcp                  1/1     Running   0          26m
myapp-deploy                  1/1     Running   0          25h
nginx-7f466444dc-cq7p4        1/1     Running   0          26h
nginx-7f466444dc-kbb2b        1/1     Running   0          26h
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          7d
nginx-test-75c685fdb7-nvv26   1/1     Running   0          26h
pod-node-affinity-demo        1/1     Running   0          7d
springboot                    0/1     Running   0          12s
[root@master readiness]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
liveness-tcp                  1/1     Running   0          26m
myapp-deploy                  1/1     Running   0          25h
nginx-7f466444dc-cq7p4        1/1     Running   0          26h
nginx-7f466444dc-kbb2b        1/1     Running   0          26h
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          7d
nginx-test-75c685fdb7-nvv26   1/1     Running   0          26h
pod-node-affinity-demo        1/1     Running   0          7d
springboot                    0/1     Running   0          14s
#就绪性探测成功，可以对外服务
[root@master readiness]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
liveness-tcp                  1/1     Running   0          26m
myapp-deploy                  1/1     Running   0          26h
nginx-7f466444dc-cq7p4        1/1     Running   0          26h
nginx-7f466444dc-kbb2b        1/1     Running   0          26h
nginx-test-75c685fdb7-4x7p8   1/1     Running   0          7d
nginx-test-75c685fdb7-nvv26   1/1     Running   0          26h
pod-node-affinity-demo        1/1     Running   0          7d
springboot                    1/1     Running   0          40s
```

正常情况下，我们会在 pod template 中配置 livenessProbe 来探测容器是否正常运行，如果异常则会触发

restartPolicy 重启容器（因为默认情况下 restartPolicy 设置的是 always）。

### 11、Kubernetes 的三种探针

* `livenessProbe`：用于探测容器是否运行。如果存活探测失败，则kubenetes杀死容器，并且容器将受到其重启策略的影响决定是否重启。如果容器不提供存活探针，则默认状态为Success。

* `readinessProbe`：一般用于探测容器内的程序是否健康，容器是否准备好服务请求。如果就绪探测失败，endpoint 将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success。
* `startupProbe`：探测容器中的应用是否已经启动。如果提供了启动探测(startupprobe)，则禁用所有其他探测，直到它成功为止。如果启动探测失败，kubelet 将杀死容器，容器服从其重启策略进行重启。如果容器没有提供启动探测，则默认状态为成功 Success。

**在启动pod的时候可以自定义是否启动这些探测，如果不设置，则检测结果均为默认通过，如果设置，则顺序为：**

*startupProbe>readinessProbe>livenessProbe*

#### 11.1、为什么要用startupProbe

* 在k8s 中，通过控制器管理pod，如果更新pod 的时候，会创建新的pod，删除老的pod，但是如果新的pod 创建了，pod 里的容器还没完成初始化，老的pod 就被删除了，会导致访问service 或者ingress 时候，访问到的pod 是有问题的，所以k8s 就加入了一些存活性探针：livenessProbe、就绪性探针readinessProbe 以及这节课要介绍的启动探针startupProbe。 

* startupProbe 是在k8s v1.16 加入了alpha 版，官方对其作用的解释是： 

Indicates whether the application within the Container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. If a Container does not provide a startup probe, the default state is Success 

翻译：判断容器内的应用程序是否已启动。如果提供了启动探测，则禁用所有其他探测，直到它成功为止。如果启动探测失败，kubelet 将杀死容器，容器将服从其重启策略。如果容器没有提供启动探测，则默认状态为成功。 

**注意：不要将startupProbe 和readinessProbe 混淆。** 

#### 11.2、什么时候用startupProbe呢

> 正常情况下，我们会在pod template 中配置livenessProbe 来探测容器是否正常运行，如果异常则会触发restartPolicy 重启容器（因为默认情况下restartPolicy 设置的是always）。 

如下：

```bash
livenessProbe:
httpGet:
path: /test
prot: 80
failureThreshold: 1
initialDelay：10
periodSeconds: 10
#上面配置的意思是容器启动10s 后每10s 检查一次，允许失败的次数是1 次。如果失败次数超过1 则会触发restartPolicy。
```

> 但是有时候会存在特殊情况，比如服务A 启动时间很慢，需要60s。这个时候如果还是用上面的探针就会进入死循环，因为上面的探针10s 后就开始探测，这时候我们服务并没有起来，发现探测失败就会触发restartPolicy。这时候有的朋友可能会想到把initialDelay 调成60s 不就可以了？但是我们并不能保证这个服务每次起来都是60s，假如新的版本起来要70s，甚至更多的时间，我们就不好控制了。有的朋友可能还会想到把失败次数增加，比如下面配置： 

```bash
livenessProbe:
httpGet:
path: /test
prot: 80
failureThreshold: 5
initialDelay：60
periodSeconds: 10
```

> 这在启动的时候是可以解决我们目前的问题，但是如果这个服务挂了呢？如果failureThreshold=1 则10s 后就会报警通知服务挂了，如果设置了failureThreshold=5，那么就需要5*10s=50s 的时间，在现在大家追求快速发现、快速定位、快速响应的时代是不被允许的。 

在这时候我们把startupProbe 和livenessProbe 结合起来使用就可以很大程度上解决我们的问题。 

```bash
livenessProbe:
httpGet:
path: /test
prot: 80
failureThreshold: 1
initialDelay：10
periodSeconds: 10
startupProbe:
httpGet:
path: /test
prot: 80
failureThreshold: 10
initialDelay：10
periodSeconds: 10
```

> 上面的配置是只有startupProbe 探测成功后再交给livenessProbe。我们startupProbe 配置的是10*10s，也就是说只要应用在100s 内启动都是OK 的，而且应用挂掉了10s 就会发现问题。 
>
> 其实这种还是不能确定具体时间，只能给出一个大概的范围。我个人认为对服务启动时间的影响因素太多了，有可能是应用本身，有可能是外部因素，比如主机性能等等。我们只有在最大程度上追求高效、稳定，但是我们不能保证100%稳定，像阿里这样的大企业对外宣称的也是5 个9，6 个9 的稳定率，如果出问题了，不好意思你恰恰不在那几个9 里面，所以我们自己要做好监控有效性，告警的及时性，响应的快速性，处理的高效性。 

#### 11.4、k8s的livenessProbe和readnessProbe的启动顺序问题

LivenessProbe会导致pod重启，ReadinessProbe只是不提供服务（就是如果容器内服务启动失败，在service中剔除掉失败的pod）

我们最初的理解是LivenessProbe会在ReadinessProbe成功后开始检查，但事实并非如此。

kubelet 使用存活探测器来知道什么时候要重启容器。例如，存活探测器可以捕捉到死锁（应用程序在运行，但是无法继续执行后面的步骤）。这样的情况下重启容器有助于让应用程序在有问题的情况下可用。

kubelet 使用就绪探测器可以知道容器什么时候准备好了并可以开始接受请求流量，当一个Pod 内的所有容器都准备好了，才能把这个Pod 看作就绪了。这种信号的一个用途就是控制哪个Pod 作为Service 的后端。在Pod 还没有准备好的时候，会从Service 的负载均衡器中被剔除的。

kubelet 使用启动探测器(startupProbe)可以知道应用程序容器什么时候启动了。如果配置了这类探测器，就可以控制容器在启动成功后再进行存活性和就绪检查，确保这些存活、就绪探测器不会影响应用程序的启动。这可以用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被杀掉。

真正的启动顺序

https://github.com/kubernetes/kubernetes/issues/60647



https://github.com/kubernetes/kubernetes/issues/27114

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes

官方文档：Caution: Liveness probes do not wait for readiness probes to succeed. If you want to wait before executing a liveness probe you should use initialDelaySeconds or a startupProbe.

也就是Liveness probes并不会等到Readiness probes成功之后才运行

根据上面的官方文档，Liveness 和readiness 应该是某种并发的关系
