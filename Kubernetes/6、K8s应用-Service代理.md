[toc]

## 一、四层负载均衡 Service

### 1、四层负载均衡 Service：概念、原理解读 

#### 1.1、为什么要有 Service？ 

> 在 kubernetes 中，Pod 是有生命周期的，如果 Pod 重启它的 IP 很有可能会发生变化。如果我们的服务都是将 Pod 的 IP 地址写死，Pod 挂掉或者重启，和刚才重启的 pod 相关联的其他服务将会找不到它所关联的 Pod，为了解决这个问题，在 kubernetes 中定义了 service 资源对象，Service 定义了一个服务访问的入口，客户端通过这个入口即可访问服务背后的应用集群实例，service 是一组 Pod 的逻辑集合，这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector 实现的。 

![image-20220919193456663](https://tva1.sinaimg.cn/large/e6c9d24ely1h6c58w62b5j20yh0ckq3v.jpg)

**1、pod ip 经常变化，service 是 pod 的代理，我们客户端访问，只需要访问 service，就会把请求代理到 Pod** 

**2、pod ip 在 k8s 集群之外无法访问，所以需要创建 service，这个 service 可以在 k8s 集群外访问的。** 

#### 1.2、Service概述

> service 是一个固定接入层，客户端可以通过**访问 service 的 ip 和端口访问到 service 关联的后端pod**，这个 service 工作依赖于在 kubernetes 集群之上部署的一个附件，就是 kubernetes 的 dns 服务（不同 kubernetes 版本的 dns 默认使用的也是不一样的，1.11 之前的版本使用的是 kubeDNs，较新的版本使用的是 coredns），service 的名称解析是依赖于 dns 附件的，因此在部署完 k8s 之后需要再部署 dns附件，kubernetes 要想给客户端提供网络功能，需要依赖第三方的网络插件（flannel，calico 等）。每个 K8s 节点上都有一个组件叫做 kube-proxy，**kube-proxy 这个组件将始终监视着 apiserver 中有关service 资源的变动信息**，需要跟 master 之上的 apiserver 交互，随时连接到 apiserver 上获取任何一个与 service 资源相关的资源变动状态，这种是通过 kubernetes 中固有的一种请求方法 watch（监视）来实现的，一旦有 service 资源的内容发生变动（如创建，删除），kube-proxy 都会将它转化成当前节点之上的能够实现 service 资源调度，把我们请求调度到后端特定的 pod 资源之上的规则，这个规则可能是 iptables，也可能是 ipvs，取决于 service 的实现方式。 

#### 1.3、Service工作原理

> k8s 在创建 Service 时，会根据标签选择器 selector(lable selector)来查找 Pod，据此创建与Service 同名的 endpoint 对象，当 Pod 地址发生变化时，endpoint 也会随之发生变化，service 接收前端 client 请求的时候，就会通过 endpoint，找到转发到哪个 Pod 进行访问的地址。(至于转发到哪个节点的 Pod，由负载均衡 kube-proxy 决定) 

#### 1.4、kubernetes 集群中有三类 IP 地址

* Node Network（节点网络）：物理节点或者虚拟节点的网络，如 ens33 接口上的网路地址

* Pod network（pod 网络），创建的 Pod 具有的 IP 地址 
  * Node Network 和 Pod network 这两种网络地址是我们实实在在配置的，其中节点网络地址是配置在节点接口之上，而 pod 网络地址是配置在 pod 资源之上的，因此这些地址都是配置在某些设备之上的，这些设备可能是硬件，也可能是软件模拟的 

* Cluster Network（集群地址，也称为 service network），这个地址是虚拟的地址（virtual ip），没有配置在某个接口上，只是出现在 service 的规则当中。

```bash
#对应service的IP无法ping通
[root@master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   35d
```

### 2、创建Service资源

```bash
#查看定义 Service 资源需要的字段有哪些？ 
[root@xianchaomaster1 ~]# kubectl explain service 
KIND: Service 
VERSION: v1 
DESCRIPTION: 
 Service is a named abstraction of software service (for example, mysql) 
 consisting of local port (for example 3306) that the proxy listens on, and 
 the selector that determines which pods will answer requests sent through 
 the proxy. 
FIELDS: 
 apiVersion <string> #service 资源使用的 api 组 
 kind <string> #创建的资源类型 
 metadata <Object> #定义元数据 
 spec <Object>
 
 #查看 service 的 spec 字段如何定义？ 
[root@xianchaomaster1 ~]# kubectl explain service.spec 
KIND: Service 
VERSION: v1 
RESOURCE: spec <Object> 
DESCRIPTION: 
 Spec defines the behavior of a service.
  https://git.k8s.io/community/contributors/devel/sig-architecture/apiconventions.md#spec-and-status 
 ServiceSpec describes the attributes that a user creates on a service. 
FIELDS: 
 allocateLoadBalancerNodePorts <boolean> 
 clusterIP <string>
 #动态分配的地址，也可以自己在创建的时候指定，创建之后就改不了了 
 clusterIPs <[]string> 
 externalIPs <[]string> 
 externalName <string> 
 externalTrafficPolicy <string> 
 healthCheckNodePort <integer> 
 ipFamilies <[]string> 
 ipFamilyPolicy <string> 
 loadBalancerIP <string> 
 loadBalancerSourceRanges <[]string> 
 ports <[]Object> #定义 service 端口，用来和后端 pod 建立联系 
 publishNotReadyAddresses <boolean> 
 selector <map[string]string> #通过标签选择器选择关联的 pod 有哪些 
 sessionAffinity <string> 
 sessionAffinityConfig <Object>
 #service 在实现负载均衡的时候还支持 sessionAffinity，sessionAffinity 什么意思？会话联系，默认是 none，随机调度的（基于 iptables 规则调度的）；如果我们定义sessionAffinity 的 client ip，那就表示把来自同一客户端的 IP 请求调度到同一个 pod 上 
 topologyKeys <[]string> 
 type <string> #定义 service 的类型
```

#### 2.1、Service的四种类型

* **ExternalName：** 
  * 适用于 k8s 集群内部容器访问外部资源，它没有 selector，也没有定义任何的端口和 Endpoint。 以下 Service 定义的是将 prod 名称空间中的 my-service 服务映射到 `my.database.example.com` 

```bash
[root@master services]# cat service-demo.yaml 
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

> **当查询主机 my-service.prod.svc.cluster.local 时，群集 DNS 将返回值为my.database.example.com 的 CNAME 记录。** 

service的FQDN是：`<service_name>.<namespace>.svc.cluster.local`

* **ClusterIP：**
  * 通过 k8s 集群内部 IP 暴露服务，选择该值，服务只能够在集群内部访问，这也是默认的ServiceType。

* **NodePort:** 
  * 通过每个 Node 节点上的 IP 和静态端口暴露 k8s 集群内部的服务。通过请求<NodeIP>:<NodePort>可以把请求代理到内部的 pod

`客户端`-`Node节点:Port`-`ServiceIP:Port`-`PodIP:Port`-`ContainerPort`

* **LoadBalancer:** 
  * 使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和ClusterIP 服务。 

#### 2.2、Service的端口

```bash
#查看 service 的 spec.ports 字段如何定义？ 
[root@xianchaomaster1 ~]# kubectl explain service.spec.ports 
KIND: Service 
VERSION: v1
RESOURCE: ports <[]Object> 
DESCRIPTION: 
 The list of ports that are exposed by this service. More info: 
 https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ipsand-service-proxies 
 ServicePort contains information on service's port. 
FIELDS: 
 appProtocol <string> 
 name <string> #定义端口的名字 
 nodePort <integer>
 #宿主机上映射的端口，比如一个 Web 应用需要被 k8s 集群之外的其他用户访问，那么需要配置type=NodePort，若配置 nodePort=30001，那么其他机器就可以通过浏览器访问 scheme://k8s 集群中的任何一个节点 ip:30001 即可访问到该服务，例如 http://192.168.1.63:30001。如果在 k8s 中部署MySQL 数据库，MySQL 可能不需要被外界访问，只需被内部服务访问，那么就不需要设置 NodePort 
 port <integer> -required- #service 的端口，这个是 k8s 集群内部服务可访问的端口 
 protocol <string> 
 targetPort <string> 
# targetPort 是 pod 上的端口，从 port 和 nodePort 上来的流量，经过 kube-proxy 流入到后端 pod的 targetPort 上，最后进入容器。与制作容器时暴露的端口一致（使用 DockerFile 中的 EXPOSE），例如官方的 nginx 暴露 80 端口。
```

#### 2.3、创建Service：type类型是ClusterIP

* 创建Pod
  * 将`nginx.tar.gz `上传到三台node节点

```bash
[root@node1 ~]# docker load -i nginx.tar.gz 
root@master services]# cat pod-test.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
[root@master services]# kubectl apply -f pod-test.yaml
#查看Pod标签
[root@master services]# kubectl get pods --show-labels
NAME                        READY   STATUS    RESTARTS   AGE   LABELS
my-nginx-5b56ccd65f-7lq7g   1/1     Running   0          41m   pod-template-hash=5b56ccd65f,run=my-nginx
my-nginx-5b56ccd65f-m9l97   1/1     Running   0          41m   pod-template-hash=5b56ccd65f,run=my-nginx
```

* 创建Service

```bash
[root@master services]# cat service-demo.yaml 
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    run: my-service
spec:
  type: ClusterIP
  ports:
  - port: 80		#service 的端口，暴露给 k8s 集群内部服务访问
    protocol: TCP
    targetPort: 80	#pod 容器中定义的端口
  selector:
    run: my-nginx		 #选择拥有 run=my-nginx 标签的 pod

[root@master services]# kubectl apply -f service-demo.yaml 
service/my-service created
[root@master services]# kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
my-nginx-5b56ccd65f   2         2         2       18m
[root@master services]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   35d
my-service   ClusterIP   10.108.144.231   <none>        80/TCP    26s
#这里的endpoint就是后端PodIP
[root@master services]# kubectl get endpoints
NAME         ENDPOINTS                         AGE
kubernetes   10.18.32.125:6443                 35d
my-service   10.244.20.97:80,10.244.83.94:80   3m54s
[root@master services]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
my-nginx-5b56ccd65f-7lq7g   1/1     Running   0          22m   10.244.83.94   node2.linux.com   <none>           <none>
my-nginx-5b56ccd65f-m9l97   1/1     Running   0          22m   10.244.20.97   node3.linux.com   <none>           <none>
##在 k8s 控制节点访问 service 的 ip:端口就可以把请求代理到后端 pod
[root@master services]# curl 10.108.144.231
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

> 上述 yaml 文件将创建一个 Service，具有标签 run=my-nginx 的 Pod，目标 TCP 端口 80，并且在一个抽象的 Service 端口（targetPort：容器接收流量的端口；port：抽象的 Service 端口，可以使任何其它 Pod 访问该 Service 的端口）上暴露。 

* 查看service详细信息

```bash
[root@master services]# kubectl describe svc my-service
Name:              my-service
Namespace:         default
Labels:            run=my-service
Annotations:       <none>
Selector:          run=my-nginx
Type:              ClusterIP
IP Families:       <none>
IP:                10.108.144.231
IPs:               10.108.144.231
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.20.97:80,10.244.83.94:80
Session Affinity:  None
Events:            <none>
```

> service 可以对外提供统一固定的 ip 地址，并将请求重定向至集群中的 pod。其中“`将请求重定向至集群中的 pod`”就是通过 endpoint 与 selector 协同工作实现。selector 是用于选择 pod，由selector 选择出来的 pod 的 ip 地址和端口号，将会被记录在 endpoint 中。`endpoint 便记录了所有 pod的 ip 地址和端口号`。当一个请求访问到 service 的 ip 地址时，就会从 endpoint 中选择出一个 ip 地址和端口号，然后将请求重定向至 pod 中。具体把请求代理到哪个 pod，需要的就是 kube-proxy 的轮询实现的。**service 不会直接到 pod，service 是直接到 endpoint 资源，再由 endpoint 再关联到 pod**。

> service 只要创建完成，我们就可以直接解析它的服务名，每一个服务创建完成后都会在集群 dns 中动态添加一个资源记录，添加完成后我们就可以解析了，资源记录格式是： 

`SVC_NAME.NS_NAME.DOMAIN.LTD.`

`服务名.命名空间.域名后缀`

集群默认的域名后缀是 svc.cluster.local. 

就像我们上面创建的 my-nginx 这个服务，它的完整名称解析就是 

```bash
my-nginx.default.svc.cluster.local
```

#### 2.4、 创建 Service：type 类型是 NodePort

* 先创建一个pod资源

```bash
[root@master services]# cat pod-nodeport.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-nodeport
spec:
  selector:
    matchLabels:
      run: my-nginx-nodeport
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx-nodeport
    spec:
      containers:
      - name: my-nginx-nodeport-container
        image: nginx
        ports:
        - containerPort: 80
[root@master services]# kubectl apply -f pod-nodeport.yaml 
deployment.apps/my-nginx-nodeport unchanged
[root@master services]# kubectl get pods -l run=my-nginx-nodeport
NAME                                 READY   STATUS    RESTARTS   AGE
my-nginx-nodeport-6f8c64fc6c-jhwwp   1/1     Running   0          25m
my-nginx-nodeport-6f8c64fc6c-qrxd2   1/1     Running   0          25m
```

* 创建一个service来代理pod

```bash
[root@master services]# cat service-nodeport.yaml 
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-nodeport
  labels:
    run: my-nginx-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30380
  selector:
    run: my-nginx-nodeport
[root@master services]# kubectl apply -f service-nodeport.yaml 
service/my-nginx-nodeport unchanged
#现在集群中访问pod
[root@master services]# kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
my-nginx-nodeport-6f8c64fc6c-jhwwp   1/1     Running   0          27m   10.244.20.98   node3.linux.com   <none>           <none>
my-nginx-nodeport-6f8c64fc6c-qrxd2   1/1     Running   0          27m   10.244.83.95   node2.linux.com   <none>   
[root@master services]# kubectl get svc
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP        36d
my-nginx-nodeport   NodePort    10.108.230.199   <none>        80:30380/TCP   22m
my-service          ClusterIP   10.108.144.231   <none>        80/TCP         142m
#可以看到访问我成功
[root@master services]# curl 10.108.230.199
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

在浏览器中访问任意k8s节点:30380

```bash
[root@master services]# netstat -tunlp  | grep 30380
tcp        0      0 0.0.0.0:30380           0.0.0.0:*               LISTEN      42708/kube-proxy    
```

![image-20220920175801041](https://tva1.sinaimg.cn/large/e6c9d24ely1h6d82e0x37j21jr0j7gno.jpg)

#### 2.5、创建 Service：type 类型是 ExternalName 

**应用场景**：跨名称空间访问 

**需求**：default 名称空间下的 client 服务想要访问 nginx-ns 名称空间下的 nginx-svc 服务 

* 先创建client的pod

```bash
[root@master services]# cat client.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client
spec:
  selector:
    matchLabels:
      app: busybox
  replicas: 1
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["/bin/sh","-c","sleep 36000"]
        
[root@master services]# kubectl apply -f client.yaml 
```

* 在创建一个默认名称空间的service

```bash
[root@master services]# cat service-client.yaml 
apiVersion: v1
kind: Service
metadata:
  name: client-svc
spec:
  type: ExternalName
  externalName: nginx-svc.nginx-ns.svc.cluster.local
  ports:
  - name: http
    port: 80
    targetPort: 80
 
[root@master services]# kubectl apply -f service-client.yaml 
```

* 创建一个namespace

```bash
[root@master services]# kubectl create ns nginx-ns
namespace/nginx-ns created
```

* 在这个命名空间下创建pod

```bash
[root@master services]# cat pod-server.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: nginx-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        
[root@master services]# kubectl apply -f pod-server.yaml 
```

* 在这个命名空间下创建service关联上一步创建的pod

```bash
[root@master services]# cat service-server-nginx.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: nginx-ns
spec:
  selector:
    app: nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
[root@master services]# kubectl apply -f service-server-nginx.yaml 
```

* 进入第一步创建的pod，访问`nginx-svc.nginx-ns.svc.cluster.local`访问其他命名空间的FQDN，发现是通的。

```bash
[root@master ~]# kubectl get pods 
NAME                      READY   STATUS    RESTARTS   AGE
client-7d8bfb6fcf-h5hvj   1/1     Running   0          23m
[root@master ~]# kubectl exec -it client-7d8bfb6fcf-h5hvj -- /bin/sh
/ # wget -q -O - nginx-svc.nginx-ns.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

* 也是在这个pod中，访问自己命名空间下的FQDN，发现被转发到了其他命名空间的Pod

```bash
[root@master services]# cat service-client.yaml 
apiVersion: v1
kind: Service
metadata:
  name: client-svc		#自己命名空间的service
spec:
  type: ExternalName
  externalName: nginx-svc.nginx-ns.svc.cluster.local		#相当于做了一个软连接，连接到其他命名空间下的pod
  ports:
  - name: http
    port: 80
    targetPort: 80
    
/ # wget -q -O - client-svc.default.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

> **两次请求结果一样，说明ExternalName类型的Service创建成功**
>
> 完成需求：**default 名称空间下的 client 服务想要访问 nginx-ns 名称空间下的 nginx-svc 服务** 

#### 2.6、k8s实践：映射外部服务

* k8s引用外部的mysql数据库
  * 现在node3节点上部署一个mysql数据库

```bash
[root@node3 ~]# yum install mariadb-server
[root@node3 ~]# systemctl start mariadb
[root@node3 ~]# netstat -tunlp | grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      48595/mysqld      
```

* 部署mysql的service

```bash
[root@master services]# cat mysql_service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: ClusterIP
  ports:
  - port: 3306
[root@master services]# kubectl apply -f !$
kubectl apply -f mysql_service.yaml
service/mysql unchanged
[root@master services]# kubectl describe svc mysql
Name:              mysql
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.241.47
IPs:               10.101.241.47
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         <none>		#这里为空，说明还没有配置后端
Session Affinity:  None
Events:            <none>
```

* 部署endpoint

```bash
[root@master services]# cat endpoint_service.yaml 
apiVersion: v1
kind: Endpoints
metadata:
  name: mysql
subsets:
- addresses:
  - ip: 10.18.32.128
  ports:
  - port: 3306
[root@master services]# kubectl apply -f !$
kubectl apply -f endpoint_service.yaml
endpoints/mysql configured

[root@master services]# kubectl describe svc mysql
Name:              mysql
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.101.241.47
IPs:               10.101.241.47
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.18.32.128:3306		#已经成功关联，在k8s内部访问FQDN：mysql.default.svc.cluster.local，即可访问外部的数据库
Session Affinity:  None
Events:            <none>
```

