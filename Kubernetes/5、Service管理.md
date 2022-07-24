[toc]

![image-20220427205504485](https://s2.loli.net/2022/04/27/ZfBJwWIEMYV2ONi.png)

## 一、Service类型

* ClusterIP
  * 仅在集群内布可以被访问，通过kube-proxy自动修改iptables实现在访问ClusterIP时转发到其中一个POD上。
* NodePort
  * 将服务发布到物理机，
* LoadBalance
  * 适用于公有云场景

## 二、ClusterIP类型

* 有ClusterIP的服务
  * 集群内部服务A访问服务B时，DNS组件会将其解析到服务B对应的IP上
* 无ClusterIP的服务
  * 集群内部服务A访问服务B时，由于服务B没有IP，DNS会直接将其解析到对应 的POD地址上，
  * 缺点
    * 由于直接解析到到POD上，失去了kube-proxy提供的负载均衡，在该POD宕机时，会导致服务不可用

### 1、创建ClusterIP服务

> 两种创建方式
>
> * 命令创建
> * yaml文件创建

#### 1、命令行创建

* 创建service

```bash
#创建了一个端口为8080的svc，具体POD端口为80的svc，可以在集群内部通过访问IP:8080 的方式访问到POD
kubectl expose deployment test  --port=8080 --target-port=80
```

* 查看svc

![image-20220427221503928](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220427221503928.png)

* 查看服务的可用性

![image-20220427221540925](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220427221540925.png)

#### 2、yaml文件创建

* 这里为了方便，我把创建的service和deployment放在了一起

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
        name: test1-nginx-deployment		#deployment的名字
spec:
        replicas: 2							#创建了两个后端POD
        selector:
                matchLabels:
                        name: test1-nginx-deployment	#指定创建出来的RS要维护的标签名字
        template:
                metadata:
                        labels:
                                name: test1-nginx-deployment #创建出来的POD标签，要与上边一致
                spec:
                        containers:
                        - name: test1-nginx
                          image: nginx:1.14
#两个类型之间用"---"来隔开
---
apiVersion: v1
kind: Service
metadata:
  name: test-service		#service的名字
  labels:					#service的标签
    app: test-service
spec:
  ports:
  - port: 80				#指定service对外提供的80端口的服务
  selector:					#通过这个标签选择器，来建立POD与service之间的关系
    name: test1-nginx-deployment

```

```bash
[root@k8s-master01 demo]# kubectl get service
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   23h
test-service      ClusterIP   10.101.174.42   <none>        80/TCP    45m
```

* 创建集群内部pod测试访问服务

```bash
[root@k8s-master01 demo]# cat test-client-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
    name: test-client
    labels:
        app: client
spec:
    containers:
    - name: test-client
      image: centos
      command:
      - sleep
      - "36000"
```

```bash
[root@k8s-master01 demo]# kubectl exec -ti test-client /bin/bash

#这个POD使用coreDNS的IP当作集群内部的DNS
[root@test-client /]# cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local linux.com
options ndots:5

# 由于clusterIP只是个标识，所以不能直接ping；但ping的时候可以看到解析结果
[root@test-client /]# ping test-service
PING test-service.default.svc.cluster.local (10.101.174.42) 56(84) bytes of data.
^C
--- test-service.default.svc.cluster.local ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1ms

# 通过curl访问测试正常，可以正常访问nginx
[root@test-client /]# curl test-service
```

### 2、创建无clusterIP的服务

```bash
apiVersion: v1
kind: Service
metadata:
  name: test-service-02
  labels:
    app: test-service-02
spec:
  clusterIP: None
  ports:
  - port: 80
  selector:
    app: nginx
```

```bash
[root@k8s-master01 demo]# kubectl get service
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   23h
test-service      ClusterIP   10.101.174.42   <none>        80/TCP    49m
test-service-02   ClusterIP   None            <none>        80/TCP    45m
```

**登陆测试访问无ClusterIP服务**

```bash
[root@k8s-master01 demo]# kubectl exec -ti test-client /bin/bash

# 由于无clusterIP服务是直接解析到对应的Pod地址上，所以可直接通信
[root@test-client /]# ping test-service-02
PING test-service-02.default.svc.cluster.local (172.168.242.136) 56(84) bytes of data.
64 bytes from 172-168-242-136.test-service.default.svc.cluster.local (172.168.242.136): icmp_seq=1 ttl=62 time=0.521 ms
64 bytes from 172-168-242-136.test-service.default.svc.cluster.local (172.168.242.136): icmp_seq=2 ttl=62 time=0.487 ms
64 bytes from 172-168-242-136.test-service.default.svc.cluster.local (172.168.242.136): icmp_seq=3 ttl=62 time=0.542 ms

# curl访问服务也正常
[root@test-client /]# curl test-service-02
```

**验证查看对应的POD地址**

```bash
[root@k8s-master01 demo]# kubectl get pods  -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP                NODE                   NOMINATED NODE   READINESS GATES
test-client                        1/1     Running   0          51m   172.168.201.201   k8s-node01.linux.com   <none>           <none>
test-deployment-57bb468744-2jf68   1/1     Running   0          56m   172.168.201.200   k8s-node01.linux.com   <none>           <none>
test-deployment-57bb468744-6n6rm   1/1     Running   0          56m   172.168.242.135   k8s-node02.linux.com   <none>           <none>
test-deployment-57bb468744-hqlt6   1/1     Running   0          56m   172.168.242.136   k8s-node02.linux.com   <none>           <none>
test-pod-01                        1/1     Running   0          18m   172.168.201.203   k8s-node01.linux.com   <none>           <none>
test-pod-02                        1/1     Running   0          12m   172.168.242.138   k8s-node02.linux.com   <none>           <none>
```

## 三、NodePort配型服务

**该类型的服务会将服务端口暴露在物理机上，可以借助物理机访问**

> 也可以通过kubectl来创建。

![image-20220427221657236](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220427221657236.png)

### 1、创建Deployment

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment-02
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx02
  template:
    metadata:
      labels:
        app: nginx02
    spec:
      containers:
      - name: test-deployment-02
        image: nginx:1.14
        imagePullPolicy: IfNotPresent

```

```bash
[root@k8s-master01 demo]# kubectl get pods 
NAME                                  READY   STATUS    RESTARTS   AGE
test-client                           1/1     Running   0          58m
test-deployment-02-5987f896b5-j4b2b   1/1     Running   0          2m57s
test-deployment-02-5987f896b5-jqzwf   1/1     Running   0          2m57s
```

### 2、创建服务

```bash
[root@k8s-master01 demo]# cat test-service-03.yaml 
apiVersion: v1
kind: Service
metadata:
  name: test-service-03
  labels:
    app: test-service-03
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30010
  selector:
    app: nginx02
```

```bash
[root@k8s-master01 demo]# kubectl get service
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        24h
test-service      ClusterIP   10.101.174.42   <none>        80/TCP         60m
test-service-02   ClusterIP   None            <none>        80/TCP         57m
test-service-03   NodePort    10.103.0.180    <none>        80:30010/TCP   2m36s
```

### 3、验证服务访问

#### 1）查看服务对应的POD节点IP

* 因为service是绑定在后端的两个POD上的，所以输入任何一个POD节点的IP都可以

```bash
[root@k8s-master01 demo]# kubectl get pods -o wide
NAME                                  READY   STATUS    RESTARTS   AGE     IP                NODE                   NOMINATED NODE   READINESS GATES
test-client                           1/1     Running   0          60m     172.168.201.201   k8s-node01.linux.com   <none>           <none>
test-deployment-02-5987f896b5-j4b2b   1/1     Running   0          4m49s   172.168.242.139   k8s-node02.linux.com   <none>           <none>
test-deployment-02-5987f896b5-jqzwf   1/1     Running   0          4m49s   172.168.201.204   k8s-node01.linux.com   <none>           <none>
```

#### 2）在物理机通过IP+端口访问

![image-20220404184125134](https://s2.loli.net/2022/04/04/N2ZKzYIRjCSh9qM.png)