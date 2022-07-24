[toc]

## 一、环境描述

### 1、节点说明

* 计划部署三个master节点，通过haproxy+keepalived来实现master的高可用，来提供一个虚拟的100VIP
* 每个虚拟机4G内存、4个逻辑CPU
* 如果资源限制，后边会介绍如何开3个机器，1个master，2个NOde节点来部署kubernetes集群

![image-20220403230124897](https://s2.loli.net/2022/04/03/QjVAzlHTrvuJhOk.png)

> 192.168.140.13	k8s-master01.linux.com      
>
> 192.168.140.14	k8s-master02.linux.com
>
> 192.168.140.15	k8s-master03.linux.com
>
> 192.168.140.16	k8s-node01.linux.com
>
> 192.168.140.18	k8s-node02.linux.com
>
> 192.168.140.100	master VIP

### 2、所用到的软件版本

> 系统: CentOS 7.6
>
> kubeadm版本: 1.20.7
>
> docker版本: 19.03
>
> Pod网段: 172.168.0.0/16
>
> Service网段: 10.96.0.0/16

## 二、基础环境配置

### 1、所有节点配置免密SSH

### 2、关闭所有结点的SELinux、防火墙、时间同步

```bash
[root@k8s-master01 ~]# echo "* */2  * * * /usr/sbin/ntpdate 120.25.115.20 &> /dev/null" &> /var/spool/cron/root
[root@k8s-master01 ~]# crontab -l
*/30 * * * * /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
```

### 3、所有节点添加主机名解析

```bash
[root@k8s-master01 ~]#  cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.140.13	k8s-master01.linux.com k8s-master01
192.168.140.14	k8s-master02.linux.com k8s-master02
192.168.140.15	k8s-master03.linux.com k8s-master03
192.168.140.16	k8s-node01.linux.com k8s-node01
192.168.140.18	k8s-node02.linux.com k8s-node02
192.168.140.100	k8s-master-vip
```

### 4、所有节点添加docker软件仓库

```bash
[root@k8s-master01 ~]# cat /etc/yum.repos.d/docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```

### 5、所有节点添加kubernetes软件仓库

```bash
[root@k8s-master01 ~]# cat /etc/yum.repos.d/kubernetes.repo 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

#由于官网未开放同步方式, 可能会有索引gpg检查失败的情况, 这时请用 yum install -y --nogpgcheck kubelet kubeadm kubectl 安装
```

### 6、所有节点禁用swap

* 因为kubernetes会使用大量内存，建议关闭交换分区，否则会使kubernetes效率降低

```bash
#临时关闭，同归关闭就注释/etc/fstab 中的 挂载交换分区
# swapoff -a

# sysctl -w vm.swappiness=0
```

### 7、所有节点调整系统资源限制

* 过低k8s跑不起来 

```bash
[root@k8s-master01 ~]# ulimit -SHn 65535

#在文件的最后边加入
[root@k8s-master01 ~]# vim /etc/security/limits.conf 
#系统支持的打开最大文件数
* soft nofile 655360
* hard nofile 131072
#最大能打开的进程数
* soft nproc 655350
* hard nproc 655350
#对时使用的内存不做限制
* soft memlock unlimited
* hard memlock unlimited
```

### 8、所有节点升级系统、并重启

* 保持所有软件包是最新的
* 保持系统的发行版本和内核都是最新的

```bash
# yum update 
# reboot
[root@k8s-master01 ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)

[root@k8s-master01 ~]# uname -r
3.10.0-1160.25.1.el7.x86_64
```

## 三、内核参数调整

* 安装ipvs是因为在1.18版本之后，services组件在对后端pod节点做负载均衡的时候，改用了ipvs

### 1、所有节点安装ipvs

* 这些大部分都是官方指定的资料，直接复制即可

```bash
# yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

### 2、所有节点加载ipvs模块

* 就是一些调度算法

```bash
[root@k8s-master01 ~]#  /etc/modules-load.d/ipvs.conf 
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack_ipv4
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

```bash
# systemctl enable --now systemd-modules-load.service
```

### 3、调整内核参数

```bash
[root@k8s-master01 ~]# cat /etc/sysctl.d/k8s.conf 
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
[root@k8s-master01 ~]# sysctl --system
```

## 四、所有节点安装docker

```bash
[root@k8s-master01 ~]# yum install -y docker-ce-19.03* 

[root@k8s-master01 ~]# systemctl start docker
[root@k8s-master01 ~]# systemctl enable docker
```

* 阿里云镜像加速

```bash
[root@k8s-master01 ~]# cat /etc/docker/daemon.json
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
 
[root@k8s-master01 ~]# systemctl restart docker
```

## 五、所有节点安装kubeadm工具

### 1、安装kubeadm 1.20.7

```bash
[root@k8s-master01 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
```

### 2、kubelet默认下载pause镜像地址为gcr.io，修改其为国内地址

```bash
[root@k8s-master01 ~]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2"
```

### 3、开机自启

```bash
[root@k8s-master01 ~]# systemctl enable kubelet
```



## 六、在三个master节点分别安装Haproxy和keepalived(机器不行可跳过)

### 1、安装相关软件

```bash
# yum install -y haproxy keepalived
```

### 2、所有Master节点配置Haproxy提供负载均衡

```bash
[root@k8s-master01 ~]# cat /etc/haproxy/haproxy.cfg 
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor

frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01	192.168.140.13:6443  check
  server k8s-master02	192.168.140.14:6443  check
  server k8s-master03	192.168.140.15:6443  check
```

**注意: 三个Master节点haproxy配置保持一致!!!**

### 3、三个节点配置keepalived实现高可用

**检查kube-apiserver状态脚本** 

```bash
[root@k8s-master01 ~]# cat /etc/keepalived/check_apiserver.sh 
#!/bin/bash

err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
```

**注意: 三个master节点keepalived配置优先级、状态区分**

**k8s-master01.linux.com**

```bash
[root@k8s-master01 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

**k8s-master02.linux.com**

```bash
[root@k8s-master02 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

**k8s-master03.linux.com**

```bash
[root@k8s-master03 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

### 4、分别启动haproxy和keepalived

```bash
[root@k8s-master01 ~]# systemctl enable haproxy keepalived
[root@k8s-master01 ~]# systemctl start haproxy keepalived
```



### 5、测试VIP工作正常，可正常通信

```bash
[root@k8s-master01 ~]# ping 192.168.140.100
PING 192.168.140.100 (192.168.140.100) 56(84) bytes of data.
64 bytes from 192.168.140.100: icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from 192.168.140.100: icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from 192.168.140.100: icmp_seq=3 ttl=64 time=0.048 ms

```

## 七、kubernetes集群初始化

### 1、在k8s-master01节点准备初始化集群文件

```bash
[root@k8s-master01 ~]# cat new.yaml 
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 96h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.140.13
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 192.168.140.100
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.140.100:16443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.7
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

### 2、事先在所有Master节点下载需要的镜像

```bash
# kubeadm config images pull --config /root/new.yaml
```

* 因为我自己准备了，就直接上传到服务器使用了

### 3、在集群所有节点导入镜像（忘记了master和node该用什么镜像，索性全部导入）

```bash
[root@k8s-master01 ~]# for i in $(ls *.tar); do docker load -i $i; done
```

### 4、在k8s-master01节点初始化集群

```bash
# kubeadm init --config /root/new.yaml  --upload-certs
.......
.......
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.183.100:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:840ee303f9bfa42f5e2d9f20fb3c2c9ab35b62b01201c3e778153f1de1e0e9cd \
    --control-plane --certificate-key 8f86add4cce0bb5ba1ad55ce5bd2da55fcbebe6d720570105c0fbbcd507091e3

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.183.100:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:840ee303f9bfa42f5e2d9f20fb3c2c9ab35b62b01201c3e778153f1de1e0e9cd 
```

### 5、在k8s-master01节点上设置环境变量，便于访问操作集群

```bash
[root@k8s-master01 ~]# tail -1 /etc/profile
export KUBECONFIG=/etc/kubernetes/admin.conf
[root@k8s-master01 ~]# source /etc/profile

#此时可以看到主节点已经加入了集群当中
[root@k8s-master01 ~]# kubectl get nodes
NAME           STATUS     ROLES                  AGE    VERSION
k8s-master01   NotReady   control-plane,master   102s   v1.20.7
```

## 八、将集群的其他master节点加入到集群中

```bash
# kubeadm join 192.168.140.100:16443 --token 7t2weq.bjbawausm0jaxury     --discovery-token-ca-cert-hash sha256:978fa35c37b8813d4d47a27747a6d88b77372d9eedfe5667da13c436cfa2a127     --control-plane --certificate-key 16a7bedd682685bf70a49d59ce1f10627c9219d5387f6476c26e481e400422f0
```

## 九、将其他node节点加入集群

```bash
# kubeadm join 192.168.140.100:16443 --token 7t2weq.bjbawausm0jaxury     --discovery-token-ca-cert-hash sha256:978fa35c37b8813d4d47a27747a6d88b77372d9eedfe5667da13c436cfa2a127
```

* 此时可以看到所有节点都加入到了集群当中，但是并没有准备好

```bash
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS     ROLES                  AGE     VERSION
k8s-master01             NotReady   control-plane,master   3m43s   v1.20.7
k8s-master02.linux.com   NotReady   control-plane,master   93s     v1.20.7
k8s-master03.linux.com   NotReady   control-plane,master   44s     v1.20.7
k8s-node01.linux.com     NotReady   <none>                 33s     v1.20.7
k8s-node02.linux.com     NotReady   <none>                 9s      v1.20.7
```

## 十、为了实现跨物理机的容器之间通信，部署calico网络

该操作只需要在k8s-master01节点执行

* 之所有不使用  fannel+etcd是因为   fannel使用于小型集群，而kubernetes并不适合，所以这里使用calico

### 1、将k8s-ha-install 安装源文件上传到服务器

```bash
[root@k8s-master01 calico]# pwd
/root/k8s-ha-install/calico
```

### 2、修改calico-etcd.yaml配置，添加etcd数据库连接地址

```bash
# sed -i 's#etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"#etcd_endpoints: "https://192.168.140.13:2379,https://192.168.140.14:2379,https://192.168.140.15:2379"#g' calico-etcd.yaml
```

### 3、在文件中添加证书相关信息

```bash
k8s中的所有组件，在相互通信的时候需要证书加密传输
# ETCD_CA=`cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'`
# ETCD_CERT=`cat /etc/kubernetes/pki/etcd/server.crt | base64 | tr -d '\n'`
# ETCD_KEY=`cat /etc/kubernetes/pki/etcd/server.key | base64 | tr -d '\n'`

#这里也是添加证书加密，在calico和etcd数据库在传输数据时，需要的证书

# sed -i "s@# etcd-key: null@etcd-key: ${ETCD_KEY}@g; s@# etcd-cert: null@etcd-cert: ${ETCD_CERT}@g; s@# etcd-ca: null@etcd-ca: ${ETCD_CA}@g" calico-etcd.yaml
```

* 真正调用这些证书，让其生效

```bash
# sed -i 's#etcd_ca: ""#etcd_ca: "/calico-secrets/etcd-ca"#g; s#etcd_cert: ""#etcd_cert: "/calico-secrets/etcd-cert"#g; s#etcd_key: "" #etcd_key: "/calico-secrets/etcd-key" #g' calico-etcd.yaml
```

### 4、在文件中修改pod网段信息

* 就是告诉calico这个组件，我以后要在172.168.0.0/16这个网段通信

```bash
# sed -i 's@# - name: CALICO_IPV4POOL_CIDR@- name: CALICO_IPV4POOL_CIDR@g; s@#   value: "172.168.0.0/16"@  value: '"${POD_SUBNET}"'@g' calico-etcd.yaml
```

### 5、部署caclico网络

```bash
# kubectl apply -f calico-etcd.yaml
```

### 6、等待kube-system命名空间中的所有pod均为running状态，再次查看集群节点

```bash
[root@k8s-master01 calico]# kubectl get nodes
NAME                     STATUS   ROLES                  AGE     VERSION
k8s-master01             Ready    control-plane,master   11m     v1.20.7
k8s-master02.linux.com   Ready    control-plane,master   9m44s   v1.20.7
k8s-master03.linux.com   Ready    control-plane,master   8m55s   v1.20.7
k8s-node01.linux.com     Ready    <none>                 8m44s   v1.20.7
k8s-node02.linux.com     Ready    <none>                 8m20s   v1.20.7
```



==**至此，kubernetes集群部署完毕，下边是加入Metric和dashboard组件，来实现对kubernetes的监控和web界面**==



## 十一、部署Metric，用于对node节点进行监控、CPU内存资源采集

### 1、将frontend证书拷贝到所有node节点

* k8s可以利用这个组件来收集后端的node节点信息，为了实现通信，必须拷贝证书

```bash
# scp /etc/kubernetes/pki/front-proxy-ca.crt 192.168.140.16:/etc/kubernetes/pki/front-proxy-ca.crt
# scp /etc/kubernetes/pki/front-proxy-ca.crt 192.168.140.18:/etc/kubernetes/pki/front-proxy-ca.crt
```

### 2、部署metric

```bash
[root@k8s-master01 metrics-server-0.4.x-kubeadm]# pwd
/root/k8s-ha-install/metrics-server-0.4.x-kubeadm

[root@k8s-master01 metrics-server-0.4.x-kubeadm]# kubectl  create -f comp.yaml 
```

### 3、测试metric工作正常

* 可以搜集到信息

```bash
[root@k8s-master01 ~]# kubectl top node
NAME                     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master01             166m         4%     1054Mi          27%       
k8s-master02.linux.com   173m         4%     1126Mi          29%       
k8s-master03.linux.com   140m         3%     924Mi           24%       
k8s-node01.linux.com     102m         2%     647Mi           16%       
k8s-node02.linux.com     84m          2%     679Mi           18%   
```

## 十二、部署dashboard

### 1、部署dashboard

```bash
[root@k8s-master01 dashboard]# cd /root/k8s-ha-install/dashboard/

#在这个文件目录下直接运行即可
[root@k8s-master01 dashboard]# kubectl  create -f .
```

### 2、修改dashboard的运行类型为nodePort

```bash
# kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

     25     port: 443
     26     protocol: TCP
     27     targetPort: 8443
     28   selector:
     29     k8s-app: kubernetes-dashboard
     30   sessionAffinity: None
     31   type: NodePort
```

### 3、查看dashboard的运行

```bash
[root@k8s-master01 ~]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.107.41.129   <none>        8000/TCP        8d
kubernetes-dashboard        NodePort    10.108.189.40   <none>        443:32372/TCP   8d
```

### 4、访问dashboard

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/8E2B18BB87F74B1FB4E0D7ED894A6A4B/6C163207055A40DB8D6894FC2288084C/12044](https://s2.loli.net/2022/04/04/7DgxaL5MmTGN6Ut.png)

### 5、获取dashboard相关用户登录token

```bash
[root@k8s-master01 ~]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

### 6、登录成功后，dashboard界面如下

![https://note.youdao.com/yws/public/resource/a897d5553baa8a773767a822821012b0/xmlnote/8E2B18BB87F74B1FB4E0D7ED894A6A4B/2C7BE56B4A784B72A46AD75F3A60DB37/12052](https://saita-ma.oss-cn-beijing.aliyuncs.com/img/VdJOCc4iFIN2PBg.png)

## 十三、查看集群状态

### 1、查看所有节点

```bash
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS   ROLES                  AGE   VERSION
k8s-master01             Ready    control-plane,master   8d    v1.20.7
k8s-master02.linux.com   Ready    control-plane,master   8d    v1.20.7
k8s-master03.linux.com   Ready    control-plane,master   8d    v1.20.7
k8s-node01.linux.com     Ready    <none>                 8d    v1.20.7
k8s-node02.linux.com     Ready    <none>                 8d    v1.20.7
```

### 2、查看集群所有组件

```bash
#加上 -o wide 看的更为详细
[root@k8s-master01 ~]# kubectl get pods -n kube-system
NAME                                             READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6d4b864b-s4bsq         1/1     Running   5          8d
calico-node-6j9cr                                1/1     Running   5          8d
calico-node-bh2c6                                1/1     Running   5          8d
calico-node-fzd4t                                1/1     Running   7          8d
calico-node-qds9m                                1/1     Running   8          8d
calico-node-x8s4c                                1/1     Running   5          8d
coredns-54d67798b7-5pv7g                         1/1     Running   5          8d
coredns-54d67798b7-pfvlr                         1/1     Running   5          8d
etcd-k8s-master01                                1/1     Running   6          8d
etcd-k8s-master02.linux.com                      1/1     Running   5          8d
etcd-k8s-master03.linux.com                      1/1     Running   5          8d
kube-apiserver-k8s-master01                      1/1     Running   17         8d
kube-apiserver-k8s-master02.linux.com            1/1     Running   5          8d
kube-apiserver-k8s-master03.linux.com            1/1     Running   8          8d
kube-controller-manager-k8s-master01             1/1     Running   8          8d
kube-controller-manager-k8s-master02.linux.com   1/1     Running   5          8d
kube-controller-manager-k8s-master03.linux.com   1/1     Running   5          8d
kube-proxy-4skph                                 1/1     Running   5          8d
kube-proxy-g8kvq                                 1/1     Running   5          8d
kube-proxy-j7n87                                 1/1     Running   5          8d
kube-proxy-ldw4k                                 1/1     Running   5          8d
kube-proxy-lt876                                 1/1     Running   5          8d
kube-scheduler-k8s-master01                      1/1     Running   8          8d
kube-scheduler-k8s-master02.linux.com            1/1     Running   5          8d
kube-scheduler-k8s-master03.linux.com            1/1     Running   5          8d
metrics-server-545b8b99c6-jcj2n                  1/1     Running   6          8d
```

### 3、以coredns为例，查看某个pod的日志信息

```bash
# kubectl logs --tail 100 -f coredns-7ff77c879f-bnlpb -n kube-system
```



## <font color="red">**以上为k8s高可用集群部署，如果物理机资源不足的话，可以按照以下操作，将集群架构修改为三个节点，即一个Master节点，两个Node节点**</font>



### 1、重置k8s集群

* 按顺序分别在node节点、master节点上执行以下命令

```bash
# kubeadm reset
```

### 2、关闭两个master节点，将剩余master节点的haproxy和keepalived也停掉

```bash
# systemctl stop haproxy keepalived
# systemctl disable haproxy keepalived
```

### 3、修改new初始化集群文件

```bash
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 96h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: <Master节点地址>
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - <Master节点地址>
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: <Master节点地址>:6443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.7
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

### 4、重新初始化集群

```bash
# kubeadm init --config /root/new.yaml  --upload-certs
```

``` bash
# tail -n 1 /etc/profile
export KUBECONFIG=/etc/kubernetes/admin.conf
# source /etc/profile
```

### 5、按提示将node节点加入到集群中

```bash
# kubeadm join 192.168.183.13:6443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:840ee303f9bfa42f5e2d9f20fb3c2c9ab35b62b01201c3e778153f1de1e0e9cd 
```

### 6、部署caclico网络

- 部署方式与高可用集群部署方式大体一致
- 唯一不一样的就是calico-etcd.yaml文件中etcd数据库连接地址，将地址改为master节点地址即可

### 7、验证集群pod运行正常

```bash
# kubectl get pods -n kube-system 
```

