[toc]

k8s是基于声明式来管理资源，即说明我们要做什么事情，具体的交给k8s来完成

## 一、管理Pod

### 1、yaml文件位置

* 具体的yaml文件位置在哪里都可以，只要自己找得到。名字必须以yaml结尾

```bash]
kubectl create -f test1.yaml
```

### 2、yaml文件格式

* `注意冒号后一定要空格`

![image-20220427201315575](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220427201315575.png)

```bash
apiVersion: v1		#apiVersion：指定api server的版本，目前是v1  资源组/版本号 apps/v1beat2
#通过kubectl api-resouces来查看资源的版本，   vx:达标发行版本，v1beat2 测试版本
kind: Pod			#说明要创建的资源类型
metadata:			#指定pod的元数据信息
    name: test-busybox	#指定pod的名称，名字随意
    namespace: kube-system	#指定pod所在的命名空间，不写就代表在默认空间，default
    labels:			#为该对象分配标签，可以省略不写，但一般情况下在工作中建议使用与项目名相同，就是键值对的数据结构
        app: busybox
spec:				#描述POD的规格，定义pod中包含的是什么，vim的粘贴模式
    containers:
    - name: test-busybox #容器名字
      image: busybox	#容器镜像
      command:
      - sleep
      - "36000"
```

### 3、创建pod

```bash
[root@k8s-master01 demo_pod]# pwd
/opt/demo_pod
[root@k8s-master01 demo_pod]# kubectl create -f test1.yaml 
```

### 4、删除pod

> 命令或者yaml文件

```bash
[root@k8s-master01 demo_pod]# kubectl delete -f test1.yaml 
pod "test1-pod" deleted
[root@k8s-master01 demo_pod]# kubectl get pods 
No resources found in default namespace.

```

```bash
#kubectl delete  pods test POD_Name
```



### 5、查看pod的一些常用命令（排错常用）

![image-20220427195451999](https://s2.loli.net/2022/04/27/GthwDOVyfrR71xg.png)

![image-20220427200917034](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/image-20220427200917034.png)

```bash
#可以参考官方文档创建别名，
[root@k8s-master01 demo_pod]# kubectl exec -it POD_name -- bash
#-i 交互式运行，-t：打开终端
#查看某个POD中运行了哪些容器
[root@k8s-master01 demo_pod]# kubectl describe pod POD_name 
#查看pod的信息，默认查看defaul命名空间，查看其他命名空间使用-n选项
[root@k8s-master01 demo_pod]# kubectl get pod 
NAME        READY   STATUS    RESTARTS   AGE
test1-pod   1/1     Running   0          3m47s

#查看其他命名空间
[root@k8s-master01 demo_pod]# kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6d4b864b-cxxfx   1/1     Running   1          15h
calico-node-4wxxv                          1/1     Running   1          15h
calico-node-fgv7r                          1/1     Running   1          15h
calico-node-jl85z                          1/1     Running   1          15h
coredns-54d67798b7-t84tv                   1/1     Running   1          15h
coredns-54d67798b7-zflcm                   1/1     Running   1          15h
etcd-k8s-master01                          1/1     Running   1          15h
kube-apiserver-k8s-master01                1/1     Running   1          15h
kube-controller-manager-k8s-master01       1/1     Running   1          15h
kube-proxy-7nwvl                           1/1     Running   1          15h
kube-proxy-qk87v                           1/1     Running   1          15h
kube-proxy-qxqpl                           1/1     Running   1          15h
kube-scheduler-k8s-master01                1/1     Running   1          15h
metrics-server-545b8b99c6-ppqjx            1/1     Running   1          15h

#查看某个pod的更为详细的信息
[root@k8s-master01 demo_pod]# kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP                NODE                   NOMINATED NODE   READINESS GATES
test1-pod   1/1     Running   0          3m51s   172.168.201.197   k8s-node01.linux.com   <none>           <none>

#查看某个pod的创建过程
[root@k8s-master01 demo_pod]# kubectl discribe pod test1-pod

#更新了test.yaml文件，并更新pod
kubectl apply -f test1.yaml 
```

## 二、container常用选项

### 1、name：string

> 容器名称

### 2、image

>  容器用到的镜像

### 3、iamgePullPolicy：[Always|Never|IfNotPresent]

> 镜像的拉取策略
>
> * Always：每次执行yaml文件时，都重新拉取镜像，即使本地存在镜像
> * Never：不拉取镜像
> * IfNotPresent：如果本地没有，则拉取镜像，有则直接使用（常用）

### 4、restartPolicy[Always|Never|OnFailure]

> 容器的重启策略
>
> * 资源类型为Pod时无法使用，只有`类型是Deployment`时才有效
> * Always: 只要容器挂掉，删掉这个容器，重新创建，
> * Never: 容器挂掉，忽略不管
> * OnFailure: 只有在容器异常退出时，重启；如果是正常退出，则不管

### 5、commend:string 

> 容器启动时执行的命令

### 6、args：string

> 向commed传递的参数

### 7、ports

> 指名容器的服务端口，并不是发布端口
>

```bash
ports：

\- containerPort: 80
```



### 8、env

>  向容器传递环境变量

```bash
env

\- name: xxxxxx

\- value: xxxxxx
```

### 9、resources

> 对容器的CPU、内存做资源限制
>
> 1000豪核=1CPU

```bash
resources:
	requests:
	memory: "64Mi"
	cpu: "250m"
limits:
	memory: "128Mi"
	cpu: "500m"
```

### 10、健康状态检查

#### 1）检查方式

* LivenessProbe

判断容器是否健康，如果不健康, kubelet将删除该容器并根据重启策略做相应的处理

- ReadinessProbe

判断容器是否启动完成且准备接受请求；如果检测失败，则endpoint会被从service对应的endpoint中删除 

```bash
livenessProbe:
	httpGet:			#http请求
		path: /check	#访问网站的/check资源
		port: 80		#80端口
		httpHeaders:	#定义http请求头部
		- name: X-REAL-IP	#变量名
		value: client_i		#值：真实的IP地址
	initialDelaySeconds: 3	#初始化3s后开始健康状态检查
	periodSeconds: 3	#每3秒健康状态检查一次
```

## 三、POD控制器介绍和使用

> 因为单个POD是不受管理的，就是某个POD坏了，他就坏了，但有控制器不一样，它会自动在别的Node上再创建出来。

![image-20220427203411040](https://s2.loli.net/2022/04/27/9PRvjdxa8EUS1mG.png)