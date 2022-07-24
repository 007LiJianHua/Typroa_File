[toc]

## 一、存储类型

### 1、DAS：直接附加存储

> * 存储设备通过数据线连接到主板
> * 优势：速度快
> * 劣势：不利于共享
> * 常见的磁盘接口：
>   * SCSI接口
>     * SCSI (Small Computer System Interface ) 小型计算机系统接口，它分为并行SCSI（内部DAS）和串行ISCSI（外部DAS）
>   * SATA接口
>     * SATA（baiSerial Advanced Technology Attachment，串行高级技术附件）是一种基于行业标du准的串行硬件驱动器接口，是由Intel、zhiIBM、Dell、APT、Maxtor和Seagate公司共同提出的硬盘接口规范。
>   * SAS接口
>     * 服务器上的常见接口
>   * M,2
>   * PCI-E

### 2、NAS：网络附加存储

> *  **<font color="red">基于文件共享的存储</font>**
> * **<font color="red">通过NFS、CIFS协议将存储空间进行共享</font>**
> * 实现方式：
>   * 专业存储设备
>   * NFS服务器

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/77243C2D04B741F3A3DBD7B35898976B/13923](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/77243C2D04B741F3A3DBD7B35898976B/13923)

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/048D1E642C8945CB97BA6F9EE528E97A/13904](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/048D1E642C8945CB97BA6F9EE528E97A/13904)

### 3、SAN：存储区域网络

> * **<font color="red">基于块设备的共享</font>**
> * 实现方式：
>   * **<font color="red">只能是专业的存储设备</font>**
> * SAN类型：
>   * FC SAN
>     * 光纤网络存储：通过光纤进行传播
>   * IP SAN
>     * 以太网网络存储 

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/A8240FF990A94444A0D2835BA58FC691/13914](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/A8240FF990A94444A0D2835BA58FC691/13914)

> * SAN在存储的时候只能是裸块设备，自己有一个独立的硬盘或者RAID磁盘阵列
> * Initiator(发起者)
>   * 前端应用发起者
> * Target(目标)
>   * 后端绑定的SAN存储名字
> * LUN
>   * 将某个逻辑卷、RAID、绑定到到某个 Target中 

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa7dce5d159ae1deb9a564369ed2b04/122](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa7dce5d159ae1deb9a564369ed2b04/122)

> * 当某一份数据进行存储时，数据先经过本地磁盘挂载的目录，存储成块，最后通过网络传输给后端SAN存储

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/6991771AE80B42D28598CD1DD67D8ADA/13920](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/6991771AE80B42D28598CD1DD67D8ADA/13920)

## 二、SAN存储配置

### 1、安装操作系统

> * 打开VMWare--文件--新建虚拟机
> * 典型--稍后安装操作系统--客户机操作系统选择（Linux）--版本选择（其他Linux2.6内核64位）--名字改为SAN存储64位--指定20G最大磁盘大小--选择将虚拟机磁盘存储为单个文件
>   * 20G磁盘用来存储操作系统
> * 点击自定义硬件--2G内存--4个CPU--使用openfiler镜像--点击添加--硬盘--选择推荐的SCSI--创建新的虚拟磁盘--指定磁盘大小为50G--将虚拟磁盘存储为单个文件--点击完成--点击确定--完成--开启虚拟机
>   * 50G磁盘用来共享数据
> * 开机之后openfiler是已经安装好操作系统，直接回车即可
> * 点击 NEXT  --  选择美式英语（U.S English）-- NEXT
> * 提示覆盖掉所有数据，重新安装，点击   Yes

> * 选择创建分区，只选择sda为操作系统分区，NEXT

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE73e0b5ee2fb7abb62a74591de754dc97/127](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE73e0b5ee2fb7abb62a74591de754dc97/127)

> * 点击New，先为启动分区/boot分区分配512M大小，选择sda

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE10fb4b09a8573afc3b342b14d1cdf938/136](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE10fb4b09a8573afc3b342b14d1cdf938/136)

> * 在选择交换分区，2G大小。sda分区

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE7c9cff619e986ad926117c6f1f1708fa/138](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE7c9cff619e986ad926117c6f1f1708fa/138)

> * 再为根分区分配剩余的所有大小，选择sda

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE9e4c5285f1233c2413e5c8ee08750a06/140](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE9e4c5285f1233c2413e5c8ee08750a06/140)

> * 分配完的分区信息如下，点击 NEXT

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEf04a539e84ef2a50d1f50666ec55f67d/142](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEf04a539e84ef2a50d1f50666ec55f67d/142)

> * 继续NEXT

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE310b4b0ba6f54127748cf2cefe507b56/144](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE310b4b0ba6f54127748cf2cefe507b56/144)

> * 为自己的机器分配IP地址，如果VMnet8网卡启动了DHCP，可以跳过，如果没有，自行编辑	

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE624e795ba4cc74a0d92bcd50d9681201/146](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE624e795ba4cc74a0d92bcd50d9681201/146)

> * 选择时区，亚洲/上海

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd058d81cd0c6355573691b8b75d588d9/150](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCEd058d81cd0c6355573691b8b75d588d9/150)

> * 输入Root账号密码

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE57c7020c4ef97e63bbd06468e9e6defe/156)

> * 等待系统安装完毕

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE8282020040223b72b62be57f98b10023/158)

> * 重启

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCEd3f5b3aba2d6a74fd66ad95127ddb23d/160)

> * 最后可以进入系统
>   * 账号root
>   * 密码(自己写的)

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE312bcc0d74c72dc33d94648edc5a767d/162)

### 2、SAN存储配置

#### 1）登录存储

> * 输入登陆界面的web GUI界面的网址
> * 默认用户名: openfiler,   密码: password

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/73F3B249321F48638FC2B4E3F24FCFD6/26434](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/73F3B249321F48638FC2B4E3F24FCFD6/26434)

#### 2）创建物理卷

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/4CBA971BB1174C30AC3160FAEB823421/26443](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/4CBA971BB1174C30AC3160FAEB823421/26443)

> * 选择为/dev/sdb创建物理卷

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE67561bd3c9cacbfbe3e94cb2e56161a2/170)

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE58e87e0c81ab72bdeb23817d34b3e79e/172)

#### 3）创建卷组

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE6d3eeeaabd6f5782b69e04c92f019806/174)

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE52afa48f17df4b6f5e7ab73dec5d405e/176)

#### 4）创建逻辑卷

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE3d5b525439c2d9bb6fe01b686f216c31/179)

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCEe6b3335d26ef5e44cf510b1593ca68f0/181)

#### 5）开启iSCSI服务

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCEecd16a91c9f2e12a5540f9c652083cb1/184)

#### 6）添加Target

> * **命名格式:** 
>
>   iqn.YYYY-MM.反域名:<自定义名称>

![img](https://note.youdao.com/yws/public/resource/b6dda696a3093105b462734e8c698252/xmlnote/WEBRESOURCE933ff5815586096f57aeacfd9d790d41/189)

#### 7）绑定LUN

### ![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/BB7EDD268C8B49F3BF66D3EFFFB9DAC1/26463](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/BB7EDD268C8B49F3BF66D3EFFFB9DAC1/26463)3、添加基于客户端地址的认证

#### 1)在存储端添加客户端地址

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/4F26AD6F6EF24EABBFEFE400FAAAAB3E/26469](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/4F26AD6F6EF24EABBFEFE400FAAAAB3E/26469)

#### 2）设置访问策略

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/767A521473B844D5B22D3D29FCD5EF4C/26472](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/767A521473B844D5B22D3D29FCD5EF4C/26472)

#### 3）添加基于用户名、密码的认证

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/CEE5C74E63964310BD4D8B3F563152E2/26476](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/D146D33C810B4E38A18165DB742A8505/CEE5C74E63964310BD4D8B3F563152E2/26476)

## 三、客户端连接存储

### 1、安装软件

```bash
[root@localhost ~]# yum install -y iscsi-initiator-utils
```

### 2、设置认证信息

```bash
[root@localhost ~]# vim /etc/iscsi/iscsid.conf 

# *************
# CHAP Settings
# *************
node.session.auth.authmethod = CHAP

node.session.auth.username = martin
node.session.auth.password = redhat
```

### 3、探索存储

```bash
[root@localhost ~]# iscsiadm -m discovery -t st -p 192.168.140.128:3260 
iscsiadm: No portals found
```

* 第一次探索的时候会报错，出现No portal found的提示；将存储端/etc/initiators.deny文件中的拒绝所有删除 

* 继续探索

```bash
[root@localhost ~]# iscsiadm -m discovery -t st -p 192.168.140.128:3260 
192.168.140.128:3260,1 iqn.2021-07.com.linux:huawei-disk
192.168.140.129:3260,1 iqn.2021-07.com.linux:huawei-disk
```

### 4、连接存储

```bash
[root@localhost ~]# iscsiadm -m node -T iqn.2021-07.com.linux:huawei-disk -p 192.168.140.128:3260 -l 
[root@localhost ~]# iscsiadm -m node -T iqn.2021-07.com.linux:huawei-disk -p 192.168.140.129:3260 -l
```

### 5、挂载存储时注意事项

* 写设备的UUID进行挂载
* 添加挂载参数 _netdev

```bash
UUID="66985a69-f9b7-4520-b947-bfc81ba7a53e"     /test1  ext4    defaults,_netdev        0 0
```

### 6、换成另一台设备

* 直接创建目录，挂载即可，可以看到之前创建的文件