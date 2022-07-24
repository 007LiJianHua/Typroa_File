# VMware网络模式

[toc]

## 一、VMnet

### 1、vmnet

> * 网络是一切服务的基础
> * 每个新建一个vmnet就是相当于一个新建虚拟交换机

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEa3fa338aa96e5eff44f21fc1b4805de7/24](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEa3fa338aa96e5eff44f21fc1b4805de7/24)

> * 每个vmnet是虚拟交换机
> * 理解成相当于是现实生活中的交换机 

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/09E0D7140AAF451096AE09D8294A06A4/24504)

## 二、windows vmnet虚拟网卡的作用

> * 虚拟网卡由在vmware中创建的虚拟交换机自动产生的
>
> * 作用
>   
>   * 为了让windows和对应虚拟交换机中的虚拟机正常通信
>   
> * 就是相当于windows通过这个两个虚拟网卡可以连接到虚拟交换机上，然后就可以与连接到这个虚拟交换机上的所有虚拟机通信
>
> * 保证windows和对应交换机上的虚拟机可正常通信 
>
> * - 配置合适的IP(vmnet交换机网络规划)
>   - 在windows的vmnet虚拟网卡配置同网段地址

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/15AE8496A104476EAD4ECCC1C03BB09A/24507)

* 不同的虚拟交换机间是隔离的

## 三、虚拟网络的三种工作模式

> * **NAT模式（常用）**
> * 桥接模式
> * 仅主机模式

### 1、NAT模式

> * VMnet8
> * 当虚拟机和外部网络通信时，虚拟机的IP会被转换成物理机IP
> * 当创建多个虚拟交换机，只能由一个NAT模式

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/6FD30A5E22364E73B52D7FA7784EF858/24523)

![https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/EF1B295D7AFB4239B19AA9A38162F9ED/24526](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/EF1B295D7AFB4239B19AA9A38162F9ED/24526)

```bash
先将虚拟机连接到NAT模式
将虚拟机IP、网关、DNS加入到NAT的网段中，手动配置
```



### 2、桥接模式

> * VMnet0
> * 通过一条特殊的链路，直接连接到互联网
> * 也会获得一个真实的IP地址
> * 就相当于一台真实的计算机
> * 但是安全性较低

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/0A57D7AF280F4C3D9323FAA8D7DC3345/24531)

> * 先虚拟机改为桥接模式，查看计算机的IP，网关，DNS
> * 再将虚拟机改为自动获取IP，再为其加上网关和DNS
>
> ```bash
> nmcli connection reload
> nmcli connection up ens33
> ping 172.20.10.2			//此时ping不通
> ping wwwbaidu.com	//可以ping通
> 
> 
> 解决虚拟机ping不通计算机的问题
> 因为计算机的防火墙没有开启虚拟机入站规则，打开即可
> win10的宿主机：windows防火墙-高级设置-入站规则-虚拟机监控(回显请求-ICMPv4-In)。
> 
> 把这项给启用了就可以了。如果网络是在公用，则需要去属性-高级 勾选公用
> ```
>
> 

### 3、仅主机模式

> * VMnet1~19
> * 与网络是断开状态，不能联网
> * 只能windows和同一个仅主机模式下的虚拟机能通信

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/D0E24A9E06A246D398BCAA037AA70083/24E553EC22DA4234886C935B429F79F7/24537)

## 四、Linux网卡聚合

### 1、网卡聚合

> * 作用
>   * 避免网卡的单点故障
>   * 实现数据负载均衡
> * 工作模式
>   * loadbalance
>     * 负载均衡模式
>     * 所有的数据要通过两块网卡进入服务器，同一时间多个网卡同时工作
>   * activebackup
>     * 主备模式
>     * 所有数据优先走主网卡，同一时间只有一个网卡工作
>
> 

### 2、配置网卡聚合

#### 1、创建聚合网卡

* -type 聚合网卡类型
  * 指定聚合网卡类型
* ifname        网卡名
* con-name   网卡配置名

```bash
[root@localhost ~]# nmcli conn add type team ifname team1 con-name team1 config '{"runner":{"name":"loadbalance"}}'
```

#### 2、绑定物理接卡

* type team-slave
  * 为聚合网卡绑定物理网卡

```bash
[root@localhost ~]# nmcli conn add type team-slave ifname ens33 con-name team1-ens33 master team1
[root@localhost ~]# nmcli conn add type team-slave ifname ens37 con-name team1-ens37 master team1
```

#### 3、在NAT模式下配置聚合网卡IP

```bash
[root@localhost ~]# nmcli connection modify team1 ipv4.addresses "192.168.140.131/24"
[root@localhost ~]# nmcli connection modify team1 ipv4.gateway "192.168.140.2"
[root@localhost ~]# nmcli connection modify team1 ipv4.dns "223.5.5.5"
[root@localhost ~]# nmcli connection modify team1 +ipv4.dns "223.6.6.6"

[root@localhost ~]# nmcli conn modify team1 ipv4.method manual
```

#### 4、激活网卡

```bash
[root@localhost ~]# nmcli conn reload
[root@localhost ~]# nmcli conn up team1  
```

#### 5、验证通信

## 五、静态路由

### 1、路由器工作原理

> - 路由器转发路由依靠路由表
>
> - - 路由表记录的信息
>
>   - - 网段、接口
>
> - 根据数据的目的IP查找路由，有则继续转发，无则丢弃

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/61BCA989DE8D401F9C7317BFE0EE47ED/B664F979333B4F06B67E0165EFBA768D/24591)

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/61BCA989DE8D401F9C7317BFE0EE47ED/7B0C263B9BC84A6CA5209795BFA1FD84/24593)

### 2、配置静态路由

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/61BCA989DE8D401F9C7317BFE0EE47ED/683B82BC27A945AE8B2A8279A6E22FB7/24596)

**1、开启路由转发**

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# cat /proc/sys/net/ipv4/ip_forward   
```

**2、添加临时静态路由**

```bash
ip route  add  目的网段 via 下一跳地址  dev 路由器要出去的接口
```

​      

**3、永久静态路由**

```bash
vim /etc/sysconfig/network-scripts/route-ens37（出网卡的名字）
192.168.175.0/24 via 192.168.175.11 dev ens37
#记得重启让网卡生效
```



**4、删除静态路由**

 ```bash
# route del -net 网段 gw 下一跳地址    
 ```





## 六、内核参数/系统参数管理

### 1、查看所有系统参数

```bash
[root@hosta ~]# sysctl -a
[root@hosta ~]# sysctl -a | grep "ip_forward"
net.ipv4.ip_forward = 0
```

### 2、永久修改内核参数

```bash
[root@hosta ~]# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1

// 让系统重新读取/etc/sysct.conf文件，让修改立即生效
[root@hosta ~]# sysctl -p
net.ipv4.ip_forward = 1

[root@hosta ~]# sysctl -a | grep "ip_forward"
net.ipv4.ip_forward = 1
```

