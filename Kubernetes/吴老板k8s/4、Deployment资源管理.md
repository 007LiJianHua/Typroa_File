[toc]

### Deployment管理

> 在实际生产化境当中，很少直接去操作pod，都是通过最常用的控制器，Deployment
>
> Deployment-------自动创建------> RS/RC----------自动创建------>POD

## 一、创建Deployment

> * 通过kubectl create deploymnet test --image=http:2.2
> * 或者通过yaml文件，
> * 当创建deployment时，自动创建出rs，pod
> * kubectl get rs/pod

### 1、编写deployement.yaml文件

```bash
apiVersion: apps/v1					#指定api server接口版本
kind: Deployment					#指定资源类型
metadata:							#指定deplpyment的元数据信息
    name: hello-deployment			#名字
spec:								#定义deployment的具体信息
    replicas: 4						#定义维持的副本数量
    selector:						#指定标签选举器的工作方式，让选择器通过哪个标签来确认容器的副本数量正常
        matchLabels:				#指定创建的RS要维护哪个标签的副本
            name: hello-deployment	#标签名字
    strategy:						#指定容器的滚动升级
        type: RollingUpdate			#指定回滚类型
        rollingUpdate:				#具体的回滚策略
									#支持的升级方式：
									#Rollupdateing	一次更新一批，一批完成后再继续更新
									#ReCreate		干掉所有容器，重新创建
            maxSurge: 20%			#每次回滚的最大POD数量，每次升级的百分比，也可以是个绝对数字，假如有10个pod，每次更新2个
            maxUnavailable: 0		#允许的最大失败次数，即不允许失败，升级失败的百分比，也可以是个绝对数字
    template:						#定义pod中运行的容器的具体信息
         metadata:					#下面所有配置与直接定义pod信息一样
             labels:
                name: hello-deployment
         spec:
            containers:
            - name: webserver
              image: nginx:1.14
              ports:
              - containerPort: 80
```

## 二、Deployment指令

### 1、查看deployment

```bash
# kubectl get deployment
```

### 2、查看deployment创建详情

```bash
# kubectl describe deployment <deployment_name>
```

### 3、查看RS/RC

```bash
# kubectl get rs

测试更新时，可以反复执行该命令查看更新的过程 
```

### 4、扩展、缩减POD数量

```bash
#扩展
kubectl scale  deployment test --replicas=5
#缩减
kubectl scale  deployment test 	--replicas=5
```

### 5、在线修改deployment文件

```bash
kubectl edit deployment test 
```

### 6、查看更新事件

```bash
kubectl describe deployment test
```

### 7、查看deployment的更新记录

```bash
kubectl rollout history deployment test
```

### 8、查看历史版本1的详细信息

```bash
kubectl rollout history deployment test --revision=1
```

### 9、回滚到历史版本1

```bash
kubectl rollout undo deployment test --to-revision=1
```



## 三、Deployment滚动更新

### 1、查看更新历史

```bash
# kubectl create -f <yaml文件> --record       记录创建信息

# kubectl rollout history deploy <deployment_name>
```

### 2、版本回退

```bash
# kubectl rollout undo deployment <deployment_name> --to-revision=<版本ID>
```

## 四、DaemonSet

### 1、作用

* 保证每个节点运行，并且只运行一个node
  * 例如监控系统，每个节点上都需要agent

### 2、创建DaemonSet

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: test-daemon-pod
  labels:
    k8s-app: test-daemon-pod 
spec:
  selector:
    matchLabels:
      name: test-daemon-pod
  template:
    metadata:
      labels:
        name: test-daemon-pod
    spec:
      containers:
      - name: test-daemon-pod
        image: busybox
        imagePullPolicy: IfNotPresent
        command:
           - sleep
           - "36000"
```

```bash
[root@k8s-master01 volume-demo]# kubectl get ds
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
test-daemon-pod   2         2         2       2            2           <none>          5s
```

```bash
[root@k8s-master01 volume-demo]# kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP                NODE                   NOMINATED NODE   READINESS GATES
test-daemon-pod-j8jcv          1/1     Running   0          14s   172.168.201.207   k8s-node01.linux.com   <none>           <none>
test-daemon-pod-t7c97          1/1     Running   0          14s   172.168.242.150   k8s-node02.linux.com   <none>           <none>
```

