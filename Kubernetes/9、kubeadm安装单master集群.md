[toc]

## 一、准备工作

### 1、机器规划

| K8S 集群角色 | Ip           | 主机名            | 安装的组件                                                   |
| ------------ | ------------ | ----------------- | ------------------------------------------------------------ |
| 控制节点     | 10.18.32.125 | master1.linux.com | apiserver、controller-manager、scheduler、etcd、docker、calico |
| 工作节点     | 10.18.32.126 | node1.linux.com   | kubelet、kube-proxy、docker、calico、coredns                 |
| 工作节点     | 10.18.32.127 | node2.linux.com   | kubelet、kube-proxy、docker、calico、coredns                 |
| 工作节点     | 10.18.32.128 | node2.linux.com   | kubelet、kube-proxy、docker、calico、coredns                 |



* **适用场景分析**

> * kubeadm 是官方提供的开源工具，是一个开源项目，用于快速搭建 kubernetes 集群，目前是比较方便和推荐使用的。kubeadm init 以及 kubeadm join 这两个命令可以快速创建 kubernetes 集群。
> * Kubeadm 初始化 k8s，所有的组件都是以 pod 形式运行的，具备故障自恢复能力。 kubeadm 是工具，可以快速搭建集群，也就是相当于用程序脚本帮我们装好了集群，属于自动部署，简化部署操作，证书、组件资源清单文件都是自动创建的，自动部署屏蔽了很多细节，使得对各个模块感知很少，如果对 k8s 架构组件理解不深的话，遇到问题比较难排查。
> * kubeadm 适合需要经常部署 k8s，或者对自动化要求比较高的场景下使用。

### 2、初始化

> * 初始化工作同`二进制安装k8s中的   一.2 ` 这里不在赘述

## 二、安装k8s集群

### 1、安装初始化 k8s 需要的软件包 

> * 以下操作在每个节点上执行

```bash
[root@master1 ~]# yum install -y kubelet-1.20.6 kubeadm-1.20.6 kubectl-1.20.6 
[root@master1 ~]# systemctl enable kubelet 
[root@master1]# systemctl status kubelet 
```

> **注：每个软件包的作用** 
>
> * Kubeadm:  kubeadm 是一个工具，用来初始化 k8s 集群的 
> * kubelet:  安装在集群所有节点上，用于启动 Pod 的 
> * kubectl:  通过 kubectl 可以部署和管理应用，查看各种资源，创建、删除和更新各种组件 

### 2、kubeadm 初始化 k8s 集群 

> * 把初始化 k8s 集群需要的`k8simage-1-20-6.tar.gz`离线镜像包上传到 master1、node1、node2 机器上，手动解压： 

```bash
[root@master1 ~]# docker load -i k8simage-1-20-6.tar.gz 
[root@node1 ~]# docker load -i k8simage-1-20-6.tar.gz 
[root@node2 ~]# docker load -i k8simage-1-20-6.tar.gz 
[root@node3 ~]# docker load -i k8simage-1-20-6.tar.gz 
```

#### 2.1、使用 kubeadm 初始化 k8s 集群 

```bash
[root@master1 ~]# kubeadm init --kubernetes-version=1.20.6  --apiserver-advertise-address=10.18.32.125  --image-repository registry.aliyuncs.com/google_containers  --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=SystemVerification 
注：--image-repository registry.aliyuncs.com/google_containers：手动指定仓库地址为registry.aliyuncs.com/google_containers。kubeadm 默认从 k8s.grc.io 拉取镜像，但是 k8s.gcr.io
访问不到，所以需要指定从 registry.aliyuncs.com/google_containers 仓库拉取镜像。
```

* 显示如下：说明安装完成

**![image-20220815162739936](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815162739936.png)**

#### 2.2、配置 kubectl 的配置文件 config

* 相当于对 kubectl 进行授权，这样 `kubectl` 命令可以使用这个证书对 k8s 集群进行管理 

```bash
[root@master1 ~]# mkdir -p $HOME/.kube 
[root@master1 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
[root@master1 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config 
[root@master1 ~]# kubectl get nodes 
NAME               STATUS    ROLES                  AGE   VERSION 
master1   NotReady   control-plane,master    60s   v1.20.6 
#此时集群状态还是 NotReady 状态，因为没有安装网络插件。 	 
```

#### 2.3、扩容 k8s 集群-添加第一个工作节点 

* 在 master1 上查看加入节点的命令：

```bash
[root@master1 ~]# kubeadm token create --print-join-command 
显示如下： 
kubeadm join 10.18.32.125:6443 --token vulvta.9ns7da3saibv4pg1     --discovery-token-ca-cert-hash sha256:72a0896e27521244850b8f1c3b600087292c2d10f2565adb56381f1f4ba7057a 
```

* 看到下面说明 node1 节点已经加入到集群了,充当工作节点

![image-20220815163144212](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815163144212.png)

* 在 master1 上查看集群节点状况：

```bash
[root@master1 ~]# kubectl get nodes 
NAME              STATUS     ROLES                  AGE     VERSION 
master1   NotReady   control-plane,master   53m     v1.20.6 
node1     NotReady   <none>                 59s     v1.20.6 
```

#### 2.4、扩容 k8s 集群-添加第二个工作节点

* 在 master1 上查看加入节点的命令： 

```bash
[root@master1 ~]# kubeadm token create --print-join-command 
 显示如下： 
kubeadm join 10.18.32.125:6443 --token i3u8gu.n1d6fy40jdxgqjpu     --discovery-token-ca-cert-hash sha256:72a0896e27521244850b8f1c3b600087292c2d10f2565adb56381f1f4ba7057a 
```

* 把 node2 加入 k8s 集群：

```bash
[root@node2~]# kubeadm join 10.18.32.125:6443 --token i3u8gu.n1d6fy40jdxgqjpu --discovery-token-ca-cert-hash sha256:72a0896e27521244850b8f1c3b600087292c2d10f2565adb56381f1f4ba7057a --ignore-preflight-errors=SystemVerification 
```

* 看到下面说明 node2 节点已经加入到集群了,充当工作节点 

![image-20220815163424665](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815163424665.png)

#### 2.5、扩容 k8s 集群-添加第三个工作节点

* 与上两个节点一致，不在演示

#### 2.6、绑定ROLES 为work

* 可以看到 node1、node2 的 ROLES 角色为空，`<none>`就表示这个节点是工作节点。 

```bash
[root@master1 ~]# kubectl get nodes 
NAME              STATUS     ROLES                  AGE     VERSION 
master1   NotReady   control-plane,master   53m     v1.20.6 
node1     NotReady   <none>                 59s     v1.20.6 
node2     NotReady   <none>                 59s     v1.20.6 
#注意：上面状态都是 NotReady 状态，说明没有安装网络插件 
```

* 绑定角色

```bash
[root@master1 ~]# kubectl label node node1.linux.com node-role.kubernetes.io/worker=worker 
[root@master1 ~]# kubectl label node node2.linux.com node-role.kubernetes.io/worker=worker 
[root@master1 ~]# kubectl label node node3.linux.com node-role.kubernetes.io/worker=worker 
```

![image-20220815163823850](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815163823850.png)

### 3、安装 kubernetes 网络组件-Calico 

* Calico本身作用
  * 做一些网络策略划分、隔离
  * 分配IP

> * 上传 `calico.yaml `到 master1 上，使用 yaml 文件安装 calico 网络插件 。

```bash
[root@master1 ~]# kubectl apply -f  calico.yaml 
注：在线下载配置文件地址是： https://docs.projectcalico.org/manifests/calico.yaml 
。 
[root@master1 ~]# kubectl get pod -n kube-system 
再次查看集群状态。 
[root@master ~]# kubectl get pods -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE               NOMINATED NODE   READINESS GATES
calico-kube-controllers-6949477b58-zcpc5   1/1     Running   0          15m   10.244.83.65     node2.linux.com    <none>           <none>	#实现pod网络入网出网流量
calico-node-8b6kx                          1/1     Running   0          15m   10.18.32.126     node1.linux.com    <none>           <none>	#具体calico的组件划分IP
calico-node-kwq5m                          1/1     Running   0          15m   10.18.32.128     node3.linux.com    <none>           <none>	#具体calico的组件划分IP
calico-node-s9blj                          1/1     Running   0          15m   10.18.32.127     node2.linux.com    <none>           <none>	#具体calico的组件划分IP
calico-node-tmwwh                          1/1     Running   0          15m   10.18.32.125     master.linux.com   <none>           <none>	#具体calico的组件划分IP
coredns-7f89b7bc75-57qzl                   1/1     Running   0          26m   10.244.131.129   node1.linux.com    <none>           <none>
coredns-7f89b7bc75-vqqq4                   1/1     Running   0          26m   10.244.20.65     node3.linux.com    <none>           <none>
etcd-master.linux.com                      1/1     Running   0          26m   10.18.32.125     master.linux.com   <none>           <none>
kube-apiserver-master.linux.com            1/1     Running   0          26m   10.18.32.125     master.linux.com   <none>           <none>
kube-controller-manager-master.linux.com   1/1     Running   0          26m   10.18.32.125     master.linux.com   <none>           <none>
kube-proxy-bbw4z                           1/1     Running   0          25m   10.18.32.126     node1.linux.com    <none>           <none>
kube-proxy-j4vbp                           1/1     Running   0          25m   10.18.32.128     node3.linux.com    <none>           <none>
kube-proxy-kmcjg                           1/1     Running   0          26m   10.18.32.125     master.linux.com   <none>           <none>
kube-proxy-mzfgf                           1/1     Running   0          25m   10.18.32.127     node2.linux.com    <none>           <none>
kube-scheduler-master.linux.com            1/1     Running   0          26m   10.18.32.125     master.linux.com   <none>           <none>
```

### 4、测试k8s 创建 pod 是否可以正常访问网络

> * 把 `busybox-1-28.tar.gz `上传到 node1、node2、node3 节点，手动解压 
>   * `busybox：内置了一些网络环境，专门用来测试网络连通信

```bash
[root@node1 ~]# docker load -i busybox-1-28.tar.gz 
432b65032b94: Loading layer [==================================================>]   1.36MB/1.36MB
Loaded image: busybox:1.28
```

```bash
[root@node1 ~]# docker load -i busybox-1-28.tar.gz 
[root@node2 ~]# docker load -i busybox-1-28.tar.gz 
[root@master1 ~]# kubectl run busybox --image busybox:1.28 --restart=Never --rm 
-it busybox -- sh 
/ # ping www.baidu.com 
PING www.baidu.com (39.156.66.18): 56 data bytes 
64 bytes from 39.156.66.18: seq=0 ttl=127 time=39.3 ms 
#通过上面可以看到能访问网络，说明 calico 网络插件已经被正常安装了 
```

### 5、测试 k8s 集群中部署 tomcat 服务 

> * 把 `tomcat.tar.gz   `上传到 node1、node2、node3，手动解压 
> * `tomcat.yaml、tomcat-service.yaml`上传到master1

```bash
[root@node1 ~]# docker load -i tomcat.tar.gz 
[root@node2 ~]# docker load -i tomcat.tar.gz 
[root@master1 ~]# kubectl apply -f tomcat.yaml 
[root@master1 ~]#  kubectl get pods 
NAME       READY   STATUS    RESTARTS   AGE 
demo-pod   1/1     Running   0          10s 
[root@master1 ~]# kubectl apply -f tomcat-service.yaml 	#创建一个svc
[root@master1 ~]# kubectl get svc 
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE 
kubernetes   ClusterIP   10.255.0.1       <none>        443/TCP          158m 
tomcat       NodePort    10.255.227.179   <none>        8080:30080/TCP   19m 
在浏览器访问 node1 节点的 ip:30080 即可请求到浏览器 
```

![image-20220815180712695](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815180712695.png)

### 6、测试coredns是否正常

> *  用来做域名解析的组件
>   * 能将对应的pod名字解析成IP

```bash
[root@master1 ~]# kubectl run busybox --image busybox:1.28 --restart=Never --rm -it busybox -- sh 
/ # nslookup kubernetes.default.svc.cluster.local 
#10.96.0.10 就是我们 coreDNS 的 clusterIP，说明 coreDNS 配置好了。 解析内部 Service 的名称，是通过 coreDNS 去解析的。 
Server:    10.96.0.10 
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local 

#解析出来的kubernetes.default.svc.cluster.local 和对应的IP
Name:      kubernetes.default.svc.cluster.local 
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local 
```

## 三、安装 k8s 可视化 UI 界面 dashboard 

### 1、安装dashboard

> * 把安装 `dashboard_2_0_0.tar.gz 、metrics-scrapter-1-0-1.tar.gz  `需要的镜像上传到工作节点 node1 和 node2，node3，手动解压： 

```bash
[root@node1 ~]# docker load -i dashboard_2_0_0.tar.gz 
[root@node1 ~]# docker load -i metrics-scrapter-1-0-1.tar.gz 
```

> * 把`kubernetes-dashboard.yaml `上传到master节点

```bash
[root@master1 ~]# kubectl apply -f kubernetes-dashboard.yaml 
#查看 dashboard 的状态 
[root@master1 ~]# kubectl get pods -n kubernetes-dashboard 
显示如下，说明 dashboard 安装成功了 
NAME                                         READY   STATUS    RESTARTS   AGE 
dashboard-metrics-scraper-7445d59dfd-n5krt   1/1     Running   0          66s 
kubernetes-dashboard-54f5b6dc4b-mhd2c        1/1     Running   0          66s 
```

### 2、查看 dashboard 前端的 service 

```bash
[root@master1 ~]#  kubectl get svc -n kubernetes-dashboard 
显示如下： 
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE 
dashboard-metrics-scraper   ClusterIP   10.98.221.57   <none>        8000/TCP 
kubernetes-dashboard        ClusterIP   10.97.37.171   <none>        443/TCP 
#类型为ClusterIP的，只能在集群内部访问，所以要修改为NodePort，在物理机映射出一个端口进行访问、

#修改 service type 类型变成 NodePort 
[root@master1 ~]# kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard 
把 type: ClusterIP 变成 type: NodePort，保存退出即可。 
[root@master ~]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.109.159.208   <none>        8000/TCP        8m40s
kubernetes-dashboard        NodePort    10.106.117.238   <none>        443:31305/TCP   8m40s
```

* 在浏览器输入：`https://10.18.32.125:31305`
  * 如果提示私密连接在页面输入
    * `thisisunsafe`
    * 注意不是在地址栏输入

![image-20220815183302636](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815183302636.png)

### 3、通过Token令牌访问DashBoard

#### 3.1、查看系统自动生成的SA

```bash
[root@master ~]# kubectl get sa -n kubernetes-dashboard
NAME                   SECRETS   AGE
default                1         16m
kubernetes-dashboard   1         16m
```

#### 3.2、对服务账号进行RABC授权，让其具有管理员权限

```bash
[root@master ~]# kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:kubernetes-dashboard 
clusterrolebinding.rbac.authorization.k8s.io/dashboard-cluster-admin created

#把kubernetes-dashboard命名空间下的kubernetes-dashboard 的SA通过clusterrolebinding绑定到cluster-admin（超级管理员权限）
#这样绑定好的SA就可以对所有名称空间下的所有pod进行操作
```

#### 3.3、查看 kubernetes-dashboard 名称空间下的 secret 

```bash
[root@master ~]# kubectl get secret -n kubernetes-dashboard
NAME                               TYPE                                  DATA   AGE
default-token-4jv8k                kubernetes.io/service-account-token   3      24m
kubernetes-dashboard-certs         Opaque                                0      24m
kubernetes-dashboard-csrf          Opaque                                1      24m
kubernetes-dashboard-key-holder    Opaque                                2      24m
kubernetes-dashboard-token-8nf7g   kubernetes.io/service-account-token   3      24m
#查看详细信息
[root@master ~]# kubectl describe secret kubernetes-dashboard-token-8nf7g -n kubernetes-dashboard
Name:         kubernetes-dashboard-token-8nf7g
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 6d37700d-d5c2-400b-80e7-d0b22f216ef3

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Im5FZnpJdHFwWWRzWW13bkU1WUk5VjF0ZWZuU1l0RU9wYThvUnJOUklPUUEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04bmY3ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZkMzc3MDBkLWQ1YzItNDAwYi04MGU3LWQwYjIyZjIxNmVmMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.LvUAJtXUz9jPNxJoWwkxWN_XKlCUfw_F1Ms6ywtW9wLdaG7J3d4z26MCOvcT7unTkggFvG-5AcB_zuhLQTuHn9TAmTtY7SJR8JaMp76NRJwLB5Xftu_uEGH8YPVVreUJB9QB0DV81V_K4ns2ucGy5CQxM5JkHvppTWyO-4f1vxr7kSE6BuQjia44ctqpwZdw6gmWZ3vH7Vy7qB8i0tFiqq-juYOc721ua0iaMgCzubgAh_hm2AlWzAo9E-uxSAKWAy8K9LjhUz4O4zFJvPgkYvxjJZSyZyMdq-bPACAlflzVNEBhtm5vwbye02atyG0p_EoTTzaoEHHM6mA-dnVJzA

```

### 4、通过kubeconfig访问dashboard

#### 4.1、创建 cluster 集群 

```bash
[root@master ~]# cd /etc/kubernetes/pki/
[root@master pki]# ls
apiserver.crt              apiserver.key                 ca.crt  front-proxy-ca.crt      front-proxy-client.key
apiserver-etcd-client.crt  apiserver-kubelet-client.crt  ca.key  front-proxy-ca.key      sa.key
apiserver-etcd-client.key  apiserver-kubelet-client.key  etcd    front-proxy-client.crt  sa.pub
[root@master pki]# kubectl config set-cluster kubernetes --certificate-authority=./ca.crt --server="https://10.18.32.125:6443" --embed-certs=true --kubeconfig=/root/dashboard-admin.conf
Cluster "kubernetes" set.
```

#### 4.2、创建 credentials 

> * 创建 credentials 需要使用上面的 `kubernetes-dashboard-token-ppc8c `对应的 token 信息

* 先将其赋值给变量

```bash
[root@master pki]# kubectl get secret -n kubernetes-dashboard
NAME                               TYPE                                  DATA   AGE
default-token-4jv8k                kubernetes.io/service-account-token   3      77m
kubernetes-dashboard-certs         Opaque                                0      77m
kubernetes-dashboard-csrf          Opaque                                1      77m
kubernetes-dashboard-key-holder    Opaque                                2      77m
kubernetes-dashboard-token-8nf7g   kubernetes.io/service-account-token   3      77m
[root@master pki]# DEF_NS_ADMIN_TOKEN=$(kubectl get secret kubernetes-dashboard-token-8nf7g -n kubernetes-dashboard  -o jsonpath={.data.token}|base64 -d) 
[root@master pki]# echo $DEF_NS_ADMIN_TOKEN
eyJhbGciOiJSUzI1NiIsImtpZCI6Im5FZnpJdHFwWWRzWW13bkU1WUk5VjF0ZWZuU1l0RU9wYThvUnJOUklPUUEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04bmY3ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZkMzc3MDBkLWQ1YzItNDAwYi04MGU3LWQwYjIyZjIxNmVmMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.LvUAJtXUz9jPNxJoWwkxWN_XKlCUfw_F1Ms6ywtW9wLdaG7J3d4z26MCOvcT7unTkggFvG-5AcB_zuhLQTuHn9TAmTtY7SJR8JaMp76NRJwLB5Xftu_uEGH8YPVVreUJB9QB0DV81V_K4ns2ucGy5CQxM5JkHvppTWyO-4f1vxr7kSE6BuQjia44ctqpwZdw6gmWZ3vH7Vy7qB8i0tFiqq-juYOc721ua0iaMgCzubgAh_hm2AlWzAo9E-uxSAKWAy8K9LjhUz4O4zFJvPgkYvxjJZSyZyMdq-bPACAlflzVNEBhtm5vwbye02atyG0p_EoTTzaoEHHM6mA-dnVJzA
```

* 创建用户dashboard-admin
  * 用户的Token就是上一步创建的变量，也同样写到cluster集群的/root/dashboard-admin.conf

```bash
[root@master pki]# kubectl config set-credentials dashboard-admin --token=$DEF_NS_ADMIN_TOKEN --kubeconfig=/root/dashboard-admin.conf 
User "dashboard-admin" set.
[root@master pki]# cat /root/dashboard-admin.conf 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EZ3hOVEEzTkRZME9Gb1hEVE15TURneE1qQTNORFkwT0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTUhaCkp6dURSaFlJMnRwWHVaQVpwQTJmWWNXbUk2OURCUzNMdFZpL3NQNjRiS25WcXo3bkRNb0FEUVo5alQzcUY5NEcKQ25pRU5McURPVy9wcDlIUGQ2VklKTHc0ek1wTk9KeG0zSEpQTFlKRVlGUzJIZmNhSGhyWEpxcE5ka2NVemRrLwpjSG84dHpHWmcxSXExOE9QTmlvNlVpeFhkeHB6TzdVdXE4N2hHTUxzSlYyamtIc2s4VXRndDJ2R3FWRlJEdEtZCjlzcm5iemJaUlZtY0w4dW8zMUNWbERXSC9uNEcvTm1UMGhzUGV6cnAyZEZPbDZCcHJzTU0rZW0xbDlBb1gzY2cKeFRTYnVQcnVFcEw1aWkvWkh4OXVsOFhUTm9scEFUSXB1WEIvWkNwN2o3eGpqNXpRSnBYWmpNa2xnRjMvOS9ETwp2cmdmZUdRTm9nejBRbnpBV1dzQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZPKysvK04walAwMXUrZWhTam5UVmY1UEtsSjFNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCcW9rVmpjYmdZQlMycnNkVXZvT3dya0QyYXB6eHVZSFBZMzluckdtVlpiSTNLeEZ0cAo3b1ZIMVpBS0pFWFNmZnVseU9YcHEvdk1GTXFaNEl2clhZMzRldGczNFEvOWY0dFIxTnRncTErUWFvQnZIYnRSClg5WVlEVnJscjlvVkQ2UWplV0t2ZWJhWlBkWG5QUVA0WWNRUVIzSnVPMlNQVXZOVmpBYUI0ckV2SDF2QkR1SC8KMmZGYWlKTG02MmU2UE5rbThxdU1GNHI2Z3NONnk4VHBseW1jQVRPR2tRbCt4S3VDSlhYcHZGTmhkeU5Hck5rdApsRm5mdXFweWN3MDVJTnJhU0ZWLytVUEZYb3Uwa3l5bTVDMEdTMDJCM01sdWlhQTdUZ1BKcUFHY3AwMDlRL1hOCnZVSC9kOVZQOVhUcXVNNG5pQ2JlaTZIU00xM2JyWWY3bEU0awotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://10.18.32.125:6443
  name: kubernetes
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6Im5FZnpJdHFwWWRzWW13bkU1WUk5VjF0ZWZuU1l0RU9wYThvUnJOUklPUUEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04bmY3ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZkMzc3MDBkLWQ1YzItNDAwYi04MGU3LWQwYjIyZjIxNmVmMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.LvUAJtXUz9jPNxJoWwkxWN_XKlCUfw_F1Ms6ywtW9wLdaG7J3d4z26MCOvcT7unTkggFvG-5AcB_zuhLQTuHn9TAmTtY7SJR8JaMp76NRJwLB5Xftu_uEGH8YPVVreUJB9QB0DV81V_K4ns2ucGy5CQxM5JkHvppTWyO-4f1vxr7kSE6BuQjia44ctqpwZdw6gmWZ3vH7Vy7qB8i0tFiqq-juYOc721ua0iaMgCzubgAh_hm2AlWzAo9E-uxSAKWAy8K9LjhUz4O4zFJvPgkYvxjJZSyZyMdq-bPACAlflzVNEBhtm5vwbye02atyG0p_EoTTzaoEHHM6mA-dnVJzA
```

#### 4.3、创建上下文context

```bash
[root@master pki]# kubectl config set-context dashboard-admin@kubernetes --cluster=kubernetes --user=dashboard-admin --kubeconfig=/root/dashboard-admin.conf 
Context "dashboard-admin@kubernetes" created.
[root@master pki]# vim /root/dashboard-admin.conf 
```

#### 4.4、使用当前上下文

```bash
[root@master pki]# kubectl config use-context dashboard-admin@kubernetes --kubeconfig=/root/dashboard-admin.conf 
Switched to context "dashboard-admin@kubernetes".
[root@master pki]# 
[root@master pki]# 
[root@master pki]# 
[root@master pki]# cat /root/dashboard-admin.conf 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EZ3hOVEEzTkRZME9Gb1hEVE15TURneE1qQTNORFkwT0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTUhaCkp6dURSaFlJMnRwWHVaQVpwQTJmWWNXbUk2OURCUzNMdFZpL3NQNjRiS25WcXo3bkRNb0FEUVo5alQzcUY5NEcKQ25pRU5McURPVy9wcDlIUGQ2VklKTHc0ek1wTk9KeG0zSEpQTFlKRVlGUzJIZmNhSGhyWEpxcE5ka2NVemRrLwpjSG84dHpHWmcxSXExOE9QTmlvNlVpeFhkeHB6TzdVdXE4N2hHTUxzSlYyamtIc2s4VXRndDJ2R3FWRlJEdEtZCjlzcm5iemJaUlZtY0w4dW8zMUNWbERXSC9uNEcvTm1UMGhzUGV6cnAyZEZPbDZCcHJzTU0rZW0xbDlBb1gzY2cKeFRTYnVQcnVFcEw1aWkvWkh4OXVsOFhUTm9scEFUSXB1WEIvWkNwN2o3eGpqNXpRSnBYWmpNa2xnRjMvOS9ETwp2cmdmZUdRTm9nejBRbnpBV1dzQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZPKysvK04walAwMXUrZWhTam5UVmY1UEtsSjFNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCcW9rVmpjYmdZQlMycnNkVXZvT3dya0QyYXB6eHVZSFBZMzluckdtVlpiSTNLeEZ0cAo3b1ZIMVpBS0pFWFNmZnVseU9YcHEvdk1GTXFaNEl2clhZMzRldGczNFEvOWY0dFIxTnRncTErUWFvQnZIYnRSClg5WVlEVnJscjlvVkQ2UWplV0t2ZWJhWlBkWG5QUVA0WWNRUVIzSnVPMlNQVXZOVmpBYUI0ckV2SDF2QkR1SC8KMmZGYWlKTG02MmU2UE5rbThxdU1GNHI2Z3NONnk4VHBseW1jQVRPR2tRbCt4S3VDSlhYcHZGTmhkeU5Hck5rdApsRm5mdXFweWN3MDVJTnJhU0ZWLytVUEZYb3Uwa3l5bTVDMEdTMDJCM01sdWlhQTdUZ1BKcUFHY3AwMDlRL1hOCnZVSC9kOVZQOVhUcXVNNG5pQ2JlaTZIU00xM2JyWWY3bEU0awotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://10.18.32.125:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: dashboard-admin	#这个用户可以访问kubernetes集群
  name: dashboard-admin@kubernetes	#上下文的名字
current-context: dashboard-admin@kubernetes		#使用当前上下文
kind: Config
preferences: {}
users:
- name: dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6Im5FZnpJdHFwWWRzWW13bkU1WUk5VjF0ZWZuU1l0RU9wYThvUnJOUklPUUEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04bmY3ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZkMzc3MDBkLWQ1YzItNDAwYi04MGU3LWQwYjIyZjIxNmVmMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.LvUAJtXUz9jPNxJoWwkxWN_XKlCUfw_F1Ms6ywtW9wLdaG7J3d4z26MCOvcT7unTkggFvG-5AcB_zuhLQTuHn9TAmTtY7SJR8JaMp76NRJwLB5Xftu_uEGH8YPVVreUJB9QB0DV81V_K4ns2ucGy5CQxM5JkHvppTWyO-4f1vxr7kSE6BuQjia44ctqpwZdw6gmWZ3vH7Vy7qB8i0tFiqq-juYOc721ua0iaMgCzubgAh_hm2AlWzAo9E-uxSAKWAy8K9LjhUz4O4zFJvPgkYvxjJZSyZyMdq-bPACAlflzVNEBhtm5vwbye02atyG0p_EoTTzaoEHHM6mA-dnVJzA
```

#### 4.5、登陆

> * 把刚才的 kubeconfig 文件 `dashboard-admin.conf `复制到桌面
> * 浏览器访问时使用 kubeconfig 认证，把刚才的 dashboard-admin.conf 导入到 web 界面，那么就可以登陆了 

## 四、通过 kubernetes-dashboard 创建容器

### 1、上传镜像

> * 把 `nginx.tar.gz `镜像压缩包上传到 node1 和 node2 ，node3，手动解压：

```bash
[root@node1 ~]# docker load -i nginx.tar.gz 
02c055ef67f5: Loading layer [==================================================>]  72.53MB/72.53MB
766fe2c3fc08: Loading layer [==================================================>]   64.8MB/64.8MB
83634f76e732: Loading layer [==================================================>]  3.072kB/3.072kB
134e19b2fac5: Loading layer [==================================================>]  4.096kB/4.096kB
5c865c78bc96: Loading layer [==================================================>]  3.584kB/3.584kB
075508cf8f04: Loading layer [==================================================>]  7.168kB/7.168kB
Loaded image: nginx:latest
```

### 2、创建pod

* 点开右上角红色箭头标注的 “+”，如下图所示：

![image-20220815195722000](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815195722000.png)

* 选择 Create  from form 

![image-20220815195808291](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815195808291.png)

![image-20220815200355695](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815200355695.png)

* 等待一会，刷新，可以看到全部创建成功

![image-20220815200651709](https://saita-ma.oss-cn-beijing.aliyuncs.com/image-20220815200651709.png)

* 最后通过访问master的IP:32158端口，即可访问nginx

### 3、关于 port、targetport、nodeport 的说明： 

> * `nodeport `是集群外流量访问集群内服务的端口，比如客户访问 nginx，apache， 
> * `port `port 是 service 的的端口 , 是集群内的 pod 互相通信用的端口类型，比如 nginx 访问 mysql，而 mysql 是不需要让客户访问到的，
> * `targetport` 目标端口，也就是最终端口，也就是 pod 的端口。 

## 五、安装metrics-server组件

### 1、上传文件

> * metrics-server 是一个集群范围内的资源数据集和工具，同样的，metrics-server 也只是显示数据，并不提供数据存储服务，主要关注的是资源度量 API 的实现，比如 CPU、文件描述符、内存、请求延时等指标，metric-server 收集数据给 k8s 集群内使用，
> * 在通过kubecrl-top 来获取node节点的CPU、内存、等指标的时候，通过metrics-server组件来实现
> * HPA扩缩容
> * Scheduler调度

> * 把`metrics-server-amd64-0-3-6.tar.gz` 、`addon.tar.gz ` 上传到集群的**各个节点**

```bash
[root@master ~]# docker load -i metrics-server-amd64-0-3-6.tar.gz 
932da5156413: Loading layer [==================================================>]  3.062MB/3.062MB
7bf3709d22bb: Loading layer [==================================================>]  38.13MB/38.13MB
Loaded image: k8s.gcr.io/metrics-server-amd64:v0.3.6
[root@master ~]# docker load -i addon.tar.gz 
8a788232037e: Loading layer [==================================================>]   1.37MB/1.37MB
cd05ae2f58b4: Loading layer [==================================================>]   37.2MB/37.2MB
Loaded image: k8s.gcr.io/addon-resizer:1.8.4
```

### 2、部署 metrics-server 服务 

> * 在`/etc/kubernetes/manifests `里面改一下 apiserver 的配置 
> * 注意：这个是 k8s 在 1.17 的新特性，如果是 1.16 版本的可以不用添加，1.17 以后要添加。这个参数的作用是 Aggregation 允许在不修改 Kubernetes 核心代码的同时扩展 Kubernetes API。

```bash
[root@master ~]# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
#增加如下内容
- --enable-aggregator-routing=true 
[root@master ~]# kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml 
pod/kube-apiserver created
[root@master ~]# kubectl get pods -n kube-system
NAME                                       READY   STATUS             RESTARTS   AGE
calico-kube-controllers-6949477b58-zcpc5   1/1     Running            0          4h34m
calico-node-8b6kx                          1/1     Running            0          4h34m
calico-node-kwq5m                          1/1     Running            0          4h34m
calico-node-s9blj                          1/1     Running            0          4h34m
calico-node-tmwwh                          1/1     Running            0          4h34m
coredns-7f89b7bc75-57qzl                   1/1     Running            0          4h45m
coredns-7f89b7bc75-vqqq4                   1/1     Running            0          4h45m
etcd-master.linux.com                      1/1     Running            0          4h46m
kube-apiserver                             0/1     CrashLoopBackOff   5          4m23s	#这个是自动生成的pod，真正的apiserver的pod是下边 的，可以删掉
kube-apiserver-master.linux.com            1/1     Running            0          4m24s
kube-controller-manager-master.linux.com   1/1     Running            1          4h46m
kube-proxy-bbw4z                           1/1     Running            0          4h44m
kube-proxy-j4vbp                           1/1     Running            0          4h44m
kube-proxy-kmcjg                           1/1     Running            0          4h45m
kube-proxy-mzfgf                           1/1     Running            0          4h44m
kube-scheduler-master.linux.com            1/1     Running            1          4h46m
#把 CrashLoopBackOff 状态的 pod 删除 
[root@master1 ~]# kubectl delete pods kube-apiserver -n kube-system 
```

* 上传`metrics.yaml`到master节点

```bash
[root@master ~]# kubectl apply -f metrics.yaml 
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
configmap/metrics-server-config created
deployment.apps/metrics-server created
service/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
root@master ~]# kubectl get pods -n kube-system | grep metrics 
metrics-server-6595f875d6-wbgjs            2/2     Running   0          37s
```

* 测试kubectl top 命令 

```bash
[root@master ~]# kubectl top pods -n kube-system 
NAME                                       CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-6949477b58-zcpc5   2m           20Mi            
calico-node-8b6kx                          22m          118Mi           
calico-node-kwq5m                          31m          114Mi           
calico-node-s9blj                          25m          112Mi           
calico-node-tmwwh                          24m          114Mi           
coredns-7f89b7bc75-57qzl                   2m           15Mi            
coredns-7f89b7bc75-vqqq4                   3m           14Mi            
etcd-master.linux.com                      14m          70Mi            
kube-apiserver-master.linux.com            58m          422Mi           
kube-controller-manager-master.linux.com   13m          51Mi            
kube-proxy-bbw4z                           1m           20Mi            
kube-proxy-j4vbp                           1m           20Mi            
kube-proxy-kmcjg                           1m           20Mi            
kube-proxy-mzfgf                           1m           21Mi            
kube-scheduler-master.linux.com            3m           24Mi            
metrics-server-6595f875d6-wbgjs            1m           17Mi  
[root@master ~]# kubectl top nodes
NAME               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master.linux.com   156m         7%     2782Mi          39%       
node1.linux.com    75m          3%     2620Mi          37%       
node2.linux.com    98m          4%     2726Mi          38%       
node3.linux.com    86m          4%     2596Mi          36%  
```

## 六、把 scheduler、controller-manager 端口变成物理机可以监听的端口

### 1、修改

> * 默认在 1.19 之后 10252 和 10251 都是绑定在 127 的，如果想要通过 prometheus 监控，会采集不到数据，所以可以把端口绑定到物理机 

```bash
[root@master ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
etcd-0               Healthy     {"health":"true"}     
```

### 2、可以按照如下方法处理

#### 2.1、修改如下内容

```bash
vim /etc/kubernetes/manifests/kube-scheduler.yaml 
修改如下内容： 
把--bind-address=127.0.0.1 变成--bind-address=10.18.32.125 
把 httpGet:字段下的 hosts 由 127.0.0.1 变成 10.18.32.125 
把—port=0 删除 
#注意：10.18.32.125 是 k8s 的控制节点 master1 的 ip 
systemctl restart kubelet
```

```bash
vim /etc/kubernetes/manifests/kube-controller-manager.yaml 
把--bind-address=127.0.0.1 变成--bind-address=10.18.32.125 
把 httpGet:字段下的 hosts 由 127.0.0.1 变成 10.18.32.125 
把—port=0 删除 
```

```bash
修改之后在 k8s 各个节点重启下 kubelet 
systemctl restart kubelet 
[root@master1 prometheus]# kubectl get cs 
Warning: v1 ComponentStatus is deprecated in v1.19+ 
NAME                 STATUS    MESSAGE             ERROR 
scheduler            Healthy   ok 
controller-manager   Healthy   ok 
etcd-0               Healthy   {"health":"true"} 
```

```bash
[root@master ~]# netstat -tunlp | grep :10251
tcp6       0      0 :::10251                :::*                    LISTEN      33967/kube-schedule 
[root@master ~]# netstat -tunlp | grep :10252
tcp6       0      0 :::10252                :::*                    LISTEN      35947/kube-controll 

#可以看到这两台机器已经被物理机监听了
```

