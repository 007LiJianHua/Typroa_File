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