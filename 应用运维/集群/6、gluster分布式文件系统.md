[toc]

## 一、分布式文件系统

> * 适用于海量数据，增加数据读写速度
> * 也可以用来构建分布式文件系统
>   * hadoop----HDFS
>   * gluster-----【云平台】
>   * ceph

### 1、gluster特性与流程

> * 开源软件
> * 容量达到PB级别、服务器的数量最多可以达到千台
>   * 1PB=1024TB、1TB=1024GB、
> * 提升数据==读写速度、高可用性==
> * 无元数据metadata的架构，采用弹性hash定位数据
>   * 一个磁盘的元数据与真实数据是存放到一起的，
>     * **真实数据-----数据块**
>     * **元数据（文件创建时间、属性、大小、权限）---inode节点**
>   * 然而其他的分布式架构，真实数据与元数据是分开存放的
>     * 就比如下边这个图：真实数据全都会存放到后端的存储设备上，而所有数据的元数据会单独存储在一个盘上。
> * 可以在廉价的PC上构建
> * 流程：
>   * 前端的大量数据在存储的时候，先到volumn，再通过弹性hash算法，将（真实数据和元数据）放到不同的后端磁盘上

### 2、gluster结构

> * brick：真实的存储空间，表现为磁盘挂载点
> * volumn：虚拟的存储空间，用于前端业务的挂载使用

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/70A6B0A17CA442A7BB2C2EB396A61100/14068](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/70A6B0A17CA442A7BB2C2EB396A61100/14068)

## 二、gluster集群环境部署

### 1、关闭防火墙、SELinux、配置时间同步、主机名解析

```bash
[root@node01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.152.10	node01.linux.com
192.168.152.11	node02.linux.com
192.168.152.12	node03.linux.com
192.168.152.13	node04.linux.com
[root@node01 ~]# for i in 11 12 13
> do
> rsync -av /etc/hosts root@192.168.152.11:/etc/hosts
> done
[root@node01 ~]# for i in 10 11 12 13; do  ssh root@192.168.152.$i date; done
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:08 CST 2022
```

### 2、配置免密

```bash
[root@node01 ~]# ssh-keygen -t rsa
[root@node01 ~]# mv /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
[root@node01 ~]# rsync /root/.ssh/ root@192.168.152.11:/root/.ssh/
[root@node01 ~]# rsync /root/.ssh/ root@192.168.152.12:/root/.ssh/
[root@node01 ~]# rsync /root/.ssh/ root@192.168.152.13:/root/.ssh/
[root@node01 ~]# rsync /root/.ssh/ root@192.168.152.14:/root/.ssh/
[root@node01 ~]# for i in 10 11 12 13; do  ssh root@192.168.152.$i date; done
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:07 CST 2022
Wed Mar 23 17:05:08 CST 2022
```

### 3、添加gluster软件仓库,安装gluster

```bash
[root@node01 ~]# cat /etc/yum.repos.d/gluster.repo 
[gluster]
name=gluster
baseurl=https://mirrors.aliyun.com/centos/7.9.2009/storage/x86_64/gluster-6/
enabled=1
gpgcheck=0
[root@node01 ~]# for i in 11 12 13 
> do
> rsync -av /etc/yum.repos.d/gluster.repo root@192.168.152.$i:/etc/yum.repos.d/
> done
[root@node01 ~]# for i in 10 11 12 13 
> do 
> ssh root@192.168.152.$i yum -y install glusterfs-server glusterfs-fuse glusterfs
> done

```

### 4、启动并添加gluster集群

```bash
[root@node01 ~]# for i in 10 11 12 13
> do
> ssh root@192.168.152.$i systemctl start glusterd
> done
[root@node01 ~]# for i in 10 11 12 13 
> do 
> ssh root@192.168.152.$i systemctl enable glusterd
> done
Created symlink from /etc/systemd/system/multi-user.target.wants/glusterd.service to /usr/lib/systemd/system/glusterd.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/glusterd.service to /usr/lib/systemd/system/glusterd.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/glusterd.service to /usr/lib/systemd/system/glusterd.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/glusterd.service to /usr/lib/systemd/system/glusterd.service.
[root@node01 ~]# gluster peer probe node02.linux.com
peer probe: success. 
[root@node01 ~]# gluster peer probe node03.linux.com
peer probe: success. 
[root@node01 ~]# gluster peer probe node04.linux.com
peer probe: success. 
[root@node01 ~]# gluster peer status
Number of Peers: 3

Hostname: node02.linux.com
Uuid: 31414f69-1bf8-42e7-b4f9-6dc3aa43414b
State: Peer in Cluster (Connected)

Hostname: node03.linux.com
Uuid: 9768a0b7-23f5-400a-88e5-8c2884f815a6
State: Peer in Cluster (Connected)

Hostname: node04.linux.com
Uuid: 11c62b1d-69cf-487a-9c59-0adb745a4d6e
State: Peer in Cluster (Connected)

```



## 三、volume卷类型

### 1、分布式卷概念（distributed volume）

> * 特征
>   * 以单个文件为单位，分散存储到不同的brick上
> * 使用场景
>   * 海量的小文件，以增加读写速度
> * 容量
>   * 所有brick之和，无brick限制



![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/C205319341954B3DA9AE7FD53A594A60/14106](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/C205319341954B3DA9AE7FD53A594A60/14106)

### 2、创建分布式卷

#### 1）格式化硬盘，挂载到本地目录

* 这个在node01.linux.com操作：

```bash
[root@node01 ~]# mkdir /data1
[root@node01 ~]# mkfs.xfs /dev/sdb
[root@node01 ~]# sed -ri  '$a \/dev/sdb /data1/ xfs defaults 0 0'  /etc/fstab 
[root@node01 ~]# mount -a
[root@node01 ~]# df -Th | grep sdb

```

* node02.linux.com

```bash
[root@node02 ~]# mkdir /data1
[root@node02 ~]# mkfs.xfs /dev/sdb
[root@node02 ~]# sed -ri  '$a \/dev/sdb /data1/ xfs defaults 0 0'  /etc/fstab 
[root@node02 ~]# mount -a
[root@node02 ~]# df -Th | grep sdb
```

#### 2）在随便一台机器上创建分布式卷

* br1会自动创建
* 这里选择node01.linux.com
* 选择了两个brick作为数据存储，分别在node01上和node02上

```bash
[root@node01 ~]# gluster volume create test1_distribute_volume \
> node01.linux.com:/data1/br1 \
> node02.linux.com:/data1/br1 
volume create: dis_volume: success: please start the volume to access data
[root@node01 ~]# gluster volume start dis_volume 
volume start: dis_volume: success
[root@node01 ~]# gluster volume info dis_volume 
 
Volume Name: dis_volume
Type: Distribute
Volume ID: 4da84de3-fb78-487c-9b91-ed4cb0249e4e
Status: Started
Snapshot Count: 0
Number of Bricks: 2
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data1/br1
Brick2: node02.linux.com:/data1/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
```

#### 3）客户端测试

* 作为测试的客户端也需要安装glusterfs，glusterfs-fuse，并且要识别各个主机名，所以也要拷贝hosts文件

```bash
[root@client ~]# rsync -av /etc/yum.repos.d/gluster.repo  root@192.168.152.15:/etc/yum.repos.d/
[root@client ~]# yum -y install glusterfs glusterfs-fuse
[root@client ~]# rsync -av /etc/hosts root@192.168.152.15:/etc/hosts
```

* 创建挂载点
* 因为创建的逻辑卷是为整个后端gluster集群创建的，所以每个gluster集群内的主机都有这个分布式卷，这里随便挂在了一个主机

```bash
[root@client ~]# mkdir /test1
[root@client ~]# sed -ri '$a \node04.linux.com:/test1_distribute_volume /test1/ glusterfs defaults,_netdev 0 0'  /etc/fstab 
[root@client ~]# mount -a
[root@client ~]# df -Th | grep node

```

* 测试

```bash
[root@client ~]# touch /test/{1..10}.sql
[root@client ~]# ls /test1/
10.sql  1.sql  2.sql  3.sql  4.sql  5.sql  6.sql  7.sql  8.sql  9.sql
```

* 然后会在node01和node02的/data1/br1 目录下会看到各自的文件

### 3、复制卷概念（replicate volume）

* 特性
  * 以单个为单位，每个文件会被复制多份
  * 提升了数据的可靠性
* 浪费了1/2的存储空间
* replicas需要指定的复制数量与后端的brick数量一致

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/84CD975DBD484F4CA9C8E5919FFB6560/14135](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/84CD975DBD484F4CA9C8E5919FFB6560/14135)

### 4、创建复制卷

#### 1）格式化硬盘，挂载到本地目录

* node01.linux.com

```bash
[root@node01 ~]# mkdir /data2
[root@node01 ~]# mkfs.xfs /dev/sdb
[root@node01 ~]# sed -ri  '$a \/dev/sdc /data2/ xfs defaults 0 0'  /etc/fstab 
[root@node01 ~]# mount -a
[root@node01 ~]# df -Th | grep sdc
```

* node02.linux.com

```bash
[root@node02 ~]# mkdir /data2
[root@node02 ~]# mkfs.xfs /dev/sdb
[root@node02 ~]# sed -ri  '$a \/dev/sdc /data2/ xfs defaults 0 0'  /etc/fstab 
[root@node02 ~]# mount -a
[root@node02 ~]# df -Th | grep sdc
```

* node03.linux.com

```bash
[root@node03 ~]# mkdir /data2
[root@node03 ~]# mkfs.xfs /dev/sdb
[root@node03 ~]# sed -ri  '$a \/dev/sdc /data2/ xfs defaults 0 0'  /etc/fstab 
[root@node03 ~]# mount -a
[root@node03 ~]# df -Th | grep sdc
```

#### 2）在随便一台机器上创建复制卷

```bash
[root@node01 ~]# gluster volume create test02_repli_volume replica 3 \
> node01.linux.com:/data2/br1 \
> node02.linux.com:/data2/br1 \
> node03.linux.com:/data2/br1 
volume create: repli_volume: success: please start the volume to access data
[root@node01 ~]# gluster volume start test02_repli_volume 
volume start: test02_repli_volume: success
[root@node01 ~]# gluster volume info test02_repli_volume 
Volume Name: test02_repli_volume
Type: Replicate
Volume ID: 01ecf2cb-fab4-41fc-8d5e-b91a58153c4c
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data2/br1
Brick2: node02.linux.com:/data2/br1
Brick3: node03.linux.com:/data2/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off
```

#### 3）客户端测试

```bash
[root@client ~]# mkdir /test2
[root@client ~]# sed -ri '$a \node04.linux.com:/test2_distribute_volume /test1/ glusterfs defaults,_netdev 0 0'  /etc/fstab 
[root@client ~]# mount -a
[root@client ~]# df -Th | grep node
[root@client ~]# touch /test2/{1..3}.txt
[root@client ~]# ls /test2/
1.txt  2.txt  3.txt
```

* 回到这三个机器会发现，在/data/br1 下有相同的三个文件

### 5、分布复制卷概念（Distributed-replicate）

> * 特征
>   * 所有文件分散存储，不同的文件再进行复制
> * 使用场景
>   * 适用于小文件存储，提高可靠性
> * brick数量为replica参数的整数倍
> * 几倍就是说分为几组来分散数据

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/E77169A505E5416D8BC5E1C86D04E06C/14151](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/7F08661B88484497BD9385601ECFE175/E77169A505E5416D8BC5E1C86D04E06C/14151)

### 6、创建分布复制卷

* 规划使用4个主机
* node01.linux.com

```bash
[root@node01 ~]# mkdir /data3
[root@node01 ~]# mkfs.xfs /dev/sdd
[root@node01 ~]# sed -ri '$a \/dev/sdd /data3 xfs defaults 0 0' /etc/fstab 
[root@node01 ~]# mount -a
[root@node01 ~]# df -Th | grep "sdd"
/dev/sdd       xfs        20G   33M   20G   1% /data3
```

* node02.linux.com

```bash
[root@node02 ~]# mkdir /data3
[root@node02 ~]# mkfs.xfs /dev/sdd
[root@node02 ~]# sed -ri '$a \/dev/sdd /data3 xfs defaults 0 0' /etc/fstab 
[root@node02 ~]# mount -a
[root@node02 ~]# df -Th | grep "sdd"
/dev/sdd       xfs        20G   33M   20G   1% /data3
```

* node03.linux.com

```bash
[root@node03 ~]# mkdir /data3
[root@node03 ~]# mkfs.xfs /dev/sdd
[root@node03 ~]# sed -ri '$a \/dev/sdd /data3 xfs defaults 0 0' /etc/fstab 
[root@node03 ~]# mount -a
[root@node03 ~]# df -Th | grep "sdd"
/dev/sdd       xfs        20G   33M   20G   1% /data3
```

* node04.linux.com

```bash
[root@node04 ~]# mkdir /data3
[root@node04 ~]# mkfs.xfs /dev/sdd
[root@node04 ~]# sed -ri '$a \/dev/sdd /data3 xfs defaults 0 0' /etc/fstab 
[root@node04 ~]# mount -a
[root@node04 ~]# df -Th | grep "sdd"
/dev/sdd       xfs        20G   33M   20G   1% /data3
```

* 回到node01上创建分布复制卷

```bash
[root@node01 ~]# gluster volume create test3_dr_volume replica 2  node01.linux.com:/data3/br1 node02.linux.com:/data3/br1 node03.linux.com:/data3/br1 node04.linux.com:/data3/br1 
#这里会提示是因为，官方建议复制卷的机器最少3个，这个机器数量有限，就是用2个了
Replica 2 volumes are prone to split-brain. Use Arbiter or Replica 3 to avoid this. See: http://docs.gluster.org/en/latest/Administrator%20Guide/Split%20brain%20and%20ways%20to%20deal%20with%20it/.
Do you still want to continue?
 (y/n) y
volume create: test3_dr_volume: success: please start the volume to access data
[root@node01 ~]# gluster volume start test3_dr_volume 
volume start: test3_dr_volume: success
[root@node01 ~]# gluster volume info test3_dr_volume 
 
Volume Name: test3_dr_volume
Type: Distributed-Replicate
Volume ID: 3db53329-39f5-419f-9f1c-e119e8184ec7
Status: Started
Snapshot Count: 0
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data3/br1
Brick2: node02.linux.com:/data3/br1
Brick3: node03.linux.com:/data3/br1
Brick4: node04.linux.com:/data3/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

```

* 客户端测试

```bash
[root@client ~]# mkdir /data3
[root@client ~]# vim /etc/fstab 
[root@client ~]# mount -a
[root@client ~]# df -Th
Filesystem                                Type            Size  Used Avail Use% Mounted on
node04.linux.com:/test3_dr_volume         fuse.glusterfs   40G  475M   40G   2% /data3
[root@client ~]# touch /data3/{1..5}.sql
[root@client ~]# ls /data3/
1.sql  2.sql  3.sql  4.sql  5.sql
```

* 这个时候在剩下的四个机器上可以看到两两一组的数据

### 7、分散卷概念（Disperse）

> * 特征
>   * 单个文件会被查分成多个小部分存储，并且同时存储数据的校验码
> * 使用场景
>   * 存储大文件，提升速度
> * disperse
>   * 用于指定brick的数量
> * redundany
>   * 用于指定保存校验码的brick数量，不指定的话，gluster集群会自动选定brick数量用于保存校验码

### 8、创建分散卷

> * 在每一台机器上把/dev/sde 磁盘格式化，挂载到/data4目录，这里不再赘述

* 创建分散卷

```bash
[root@node01 ~]# gluster volume create test4_disperse_volume disperse 4 \
> node01.linux.com:/data4/br1 \
> node02.linux.com:/data4/br1 \
> node03.linux.com:/data4/br1 \
> node04.linux.com:/data4/br1
#这里会提示是因为：没有指定校验码的brick数量，机器自动创建
There isn't an optimal redundancy value for this configuration. Do you want to create the volume with redundancy 1 ? (y/n) y
volume create: test4_disperse_volume: success: please start the volume to access data
[root@node01 ~]# gluster volume start test4_disperse_volume 
volume start: test4_disperse_volume: success
[root@node01 ~]# gluster volume info test4_disperse_volume 
 
Volume Name: test4_disperse_volume
Type: Disperse
Volume ID: 6694fab2-623a-40ff-b6f6-140781f93588
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x (3 + 1) = 4
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data4/br1
Brick2: node02.linux.com:/data4/br1
Brick3: node03.linux.com:/data4/br1
Brick4: node04.linux.com:/data4/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

```

* 客户端测试

```bash
[root@client ~]# mkdir /data4
[root@client ~]# vim /etc/fstab 
[root@client ~]# mount -a
[root@client ~]# df -Th
Filesystem                                Type            Size  Used Avail Use% Mounted on
node04.linux.com:/test3_dr_volume         fuse.glusterfs   40G  475M   40G   2% /data3
node04.linux.com:/test4_disperse_volume   fuse.glusterfs   60G  712M   60G   2% /data4
[root@client ~]# dd if=/dev/zero of=/data4/file1 bs=10M count=5
5+0 records in
5+0 records out
52428800 bytes (52 MB) copied, 0.513233 s, 102 MB/s
[root@client ~]# dd if=/dev/zero of=/data4/file2 bs=10M count=5
5+0 records in
5+0 records out
52428800 bytes (52 MB) copied, 0.562917 s, 93.1 MB/s
[root@client ~]# dd if=/dev/zero of=/data4/file3 bs=10M count=5
5+0 records in
5+0 records out
52428800 bytes (52 MB) copied, 1.5084 s, 34.8 MB/s
[root@client ~]# dd if=/dev/zero of=/data4/file4 bs=10M count=5
5+0 records in
5+0 records out
52428800 bytes (52 MB) copied, 1.15887 s, 45.2 MB/s
[root@client ~]# ll -h /data4/
total 201M
-rw-r--r-- 1 root root 50M Mar 24 17:00 file1
-rw-r--r-- 1 root root 50M Mar 24 17:00 file2
-rw-r--r-- 1 root root 50M Mar 24 17:00 file3
-rw-r--r-- 1 root root 50M Mar 24 17:00 file4
```

* 这个时候在其余的机器上产看，会发现这每一个文件都分散存储到了四个机器上，每个文件的大小都是17M，至于校验码则无法查看，以数据的形式存在

## 四、扩展卷容量 Expanding Volume

### 1、扩展卷容量

* 在扩展分布复制卷时，添加的brick数量是replica参数的整数倍

### 2、案例：扩展分布式卷

* 先创建出一个brick，这里选择空闲的node03的/dev/sdb 挂载目录是/data1
* node03.linux.com

```bash
[root@node03 ~]# mkdir /data1 
[root@node03 ~]# mkfs.xfs /dev/sdb
meta-data=/dev/sdb               isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@node03 ~]# sed -rii '$a \/dev/sdb /data1 xfs defaults 0 0' /etc/fstab 
[root@node03 ~]# mount -a
[root@node03 ~]# df -Th | grep "sdb"
/dev/sdb                xfs        20G   33M   20G   1% /data1

```

* node01.linux.com

```bash
[root@node01 ~]# gluster volume add-brick test1_distribute_volume node03.linux.com:/data1/br1 
volume add-brick: success
[root@node01 ~]# gluster volume info test1_distribute_volume 
 
Volume Name: test1_distribute_volume
Type: Distribute
Volume ID: aff5ccd5-db20-48ec-83e4-120e1dbb0d3f
Status: Started
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data1/br1
Brick2: node02.linux.com:/data1/br1
Brick3: node03.linux.com:/data1/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
#在添加完毕后虽然加入到了分布式卷中，但实际分配数据的时候还是不会往新的brick上存储数据，所以需要重新将数据分配一下，建议这个操作不要再系统繁忙的时候操作，否则会加重对系统的负载   
[root@node01 ~]# gluster volume rebalance test1_distribute_volume start 
volume rebalance: test1_distribute_volume: success: Rebalance on test1_distribute_volume has been started successfully. Use rebalance status command to check status of the rebalance process.
ID: 8ae52832-ab46-4366-8eb5-770a5a1606a5
[root@node01 ~]# gluster volume rebalance test1_distribute_volume status
                                    Node Rebalanced-files          size       scanned      failures       skipped               status  run time in h:m:s
                               ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                        node02.linux.com                2        0Bytes             7             0             0            completed        0:00:00
                        node03.linux.com                0        0Bytes             3             0             0            completed        0:00:00
                               localhost                1        0Bytes             3             0             0            completed        0:00:00
volume rebalance: test1_distribute_volume: success

```

* 在重新对数据分配后，就可以看到数据了

```bash
[root@node03 ~]# ls /data1/br1/
[root@node03 ~]# ls /data1/br1/
5.txt  7.txt  9.txt
```

## 五、缩减卷容量 Shriking Volume

> * 缩减分布式卷时，缩减的brick数量是replica参数的整数倍

### 1、缩减分布式卷

```bash
[root@node01 ~]# gluster volume remove-brick test1_distribute_volume node03.linux.com:/data1/br1 start
Running remove-brick with cluster.force-migration enabled can result in data corruption. It is safer to disable this option so that files that receive writes during migration are not migrated.
Files that are not migrated can then be manually copied after the remove-brick commit operation.
Do you want to continue with your current cluster.force-migration settings? (y/n) y
volume remove-brick start: success
ID: ce36938a-9762-4be4-a16f-4a74910baac3
[root@node01 ~]# gluster volume remove-brick test1_distribute_volume node03.linux.com:/data1/br1 status
                                    Node Rebalanced-files          size       scanned      failures       skipped               status  run time in h:m:s
                               ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                        node03.linux.com                3        0Bytes             3             0             0            completed        0:00:00
                        
#这个时候还没有缩减成功，只是将node03.linux.com:/data3/br1 的数据重新分配到node01和node02
[root@node01 ~]# gluster volume info test1_distribute_volume 
 
Volume Name: test1_distribute_volume
Type: Distribute
Volume ID: aff5ccd5-db20-48ec-83e4-120e1dbb0d3f
Status: Started
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data1/br1
Brick2: node02.linux.com:/data1/br1
Brick3: node03.linux.com:/data1/br1
Options Reconfigured:
performance.client-io-threads: on
transport.address-family: inet
nfs.disable: on
#正式将node3踢出test1_distribute_volume卷
[root@node01 ~]# gluster volume remove-brick test1_distribute_volume node03.linux.com:/data1/br1 commit
volume remove-brick commit: success
Check the removed bricks to ensure all files are migrated.
If files with data are found on the brick path, copy them via a gluster mount point before re-purposing the removed brick. 
[root@node01 ~]# gluster volume info test1_distribute_volume 
 
Volume Name: test1_distribute_volume
Type: Distribute
Volume ID: aff5ccd5-db20-48ec-83e4-120e1dbb0d3f
Status: Started
Snapshot Count: 0
Number of Bricks: 2
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data1/br1
Brick2: node02.linux.com:/data1/br1
Options Reconfigured:
performance.client-io-threads: on
transport.address-family: inet
nfs.disable: on
```

## 六、替换故障卷 Replace faulty brick

### 1、替换分布卷的brick

> * 添加新的brick【不需要手动rebalance】
> * 删除旧的brick

### 2、替换复制卷的brick

```bash
[root@node01 ~]# gluster volume replace-brick test2_replicate_volume node03.linux.com:/data2/br1 node04.linux.com:/data2/br1 commit force
volume replace-brick: success: replace-brick commit force operation successful
[root@node01 ~]# gluster volume info test2_replicate_volume 
 
Volume Name: test2_replicate_volume
Type: Replicate
Volume ID: 8ce5f8a1-6ac0-4d89-af43-47d5b9469ab3
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: node01.linux.com:/data2/br1
Brick2: node02.linux.com:/data2/br1
Brick3: node04.linux.com:/data2/br1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

```

## 七、设置卷的参数

> ```bash
> gluster volume set <VOLNAME> <OPT-NAME> <OPT-VALUE>
> ```

### 1、常用参数

* 基于IP地址认证
  * auth.allow
  * auth.reject
* performance.write-behind-window-size
  * 设置写缓冲区buffer大小，默认1M
* performance.io-thread-count
  * 设置卷的IO线程数量1--64
* performance.cache-size
  * 设置卷的缓存大小
* performance.cache-max-file-size
  * 设置缓存的最大文件大小
* performance.cache-min-file-size
  * 设置缓存的最小文件大小
* performance.cache-refresh-timeout
  * 设置缓冲区的刷新时间间隔，单位秒