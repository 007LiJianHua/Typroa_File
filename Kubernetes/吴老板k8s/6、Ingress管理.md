[toc]

## 一、Ingress介绍

> 之前我们在k8s集群内部对外暴露服务的方式用的是nodePort，但采用 NodePort 方式暴露服务面临问题是，==服务一旦多起来，NodePort 在每个节点上开启的端口会及其庞大，而且难以维护==
>
> 因此，后续出现了一种新的方式，称为ingress；ingress类似于coreDNS是作为k8s的插件存在的，借助ingress一样可对外暴露服务
>
> 基于ingress暴露服务时，类似于七层调度，对外可提供一个主机名，将不同的服务以不同的端口方式映射到该主机名上，客户端3直接通过主机名进行访问，不需要       <节点IP>:<端口>

![image-20220404200858354](https://s2.loli.net/2022/04/04/cFwC3qUThyXkGoR.png)

### 1、Ingress核心组件

* ingress
  * 一种资源对象，用于描述服务规则，在这里就是将serverA的服务发布到db.linux.com上
* ingress-controller

- - Ingress Controller 通过与 Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化，然后读取他，按照他自己模板生成一段 Nginx 配置，再写到 Nginx Pod 里，最后 reload 一下，也就是来匹配客户端的请求的，请求不同，转发的后端也不同

### 2、Ingress实现方式

* nginx、

![image-20220404184056756](https://s2.loli.net/2022/04/04/inQBvjqGb9lk1D3.png)

* haproxy
* traefik

## 二、Ingress部署

### 1、下载ingress-nginx部署文件   

本文档使用事先准备好的文件，不同版本的文件可到github上进行下载 

### 2、修改ingress-nginx-deploy.yaml

#### 1）将ingress-nginx-controller的网络修改为复用主机网络

```bash
274     spec:
275       serviceAccountName: nginx-ingress-serviceaccount
276       hostNetwork: true
277       containers:
278         - name: nginx-ingress-controller
279           image: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:0.20.0
```

#### 2）镜像下载地址修改为registry.cn-hangzhou.aliyuncs.com/google_containers

image: registry.cn-hangzhou.aliyuncs.com/google_containers/defaultbackend-amd64:1.5

 image: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:0.20.0

建议实现准备好镜像，增加部署速度

### 3、部署ingress-nginx，先把这个环境搭建起来

```bash
# kubectl create -f ingress-nginx-deploy.yaml
```

```bash
[root@k8s-master01 opt]# kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS    RESTARTS   AGE
default-http-backend-5c9967d8d6-2dn7k       1/1     Running   0          100m
nginx-ingress-controller-7f74f77cf5-mxcbf   1/1     Running   0          31m
```

### 4、创建测试deploy和service

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
    name: test-nginx-deploy
spec:
    replicas: 2
    selector:
        matchLabels:
            name: test-nginx-deploy
    template:
         metadata:
             labels:
                name: test-nginx-deploy
         spec:
            containers:
            - name: webserver
              image: nginx:1.14
              imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  ports:
  - port: 80
  selector:
    name: test-nginx-deploy
```

### 5、注意：该服务类型为clusterIP

* 只能集群内部通信，但是通过ingress-nginx在外部网络中也可以访问

```bash
[root@k8s-master01 ingress-demo]# kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   10d
test-service      ClusterIP   10.100.68.127   <none>        80/TCP    129m
```

### 6、创建ingress发布服务

```bash
apiVersion: extensions/v1beta1
kind: Ingress 
metadata:
  name: ingress-test
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: 
  - host: test.linux.com
    http:
      paths: 
      - path:  
        backend: 
          serviceName: test-service
          servicePort: 80
```

### 7、在物理机上添加主机名解析，测试访问