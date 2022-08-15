# LVM逻辑卷管理

[toc]

## 一、lvm逻辑卷介绍

> * linux独有的一种管理磁盘方式
> * 优势
>   * **支持在不影响数据的情况下，扩展容量，这是最大的优势**
>     * 扩展的设备必须是裸设备，不能有任何文件系统，最后由逻辑卷来创建文件系统
>   * 支持快照，便于备份数据
> * 在挂载逻辑卷时多了一层mapper目录：
>
> ```bash
> //这两个都是链接文件，指向/dev/dm-3,这个dm-3才是真的磁盘文件
> [root@localhost ~]# ls -l /dev/db/mysql 
> lrwxrwxrwx 1 root root 7 Dec 23 19:31 /dev/db/mysql -> ../dm-3
> [root@localhost ~]# ls -l /dev/mapper/db-mysql 
> lrwxrwxrwx 1 root root 7 Dec 23 19:31 /dev/mapper/db-mysql -> ../dm-3
> 
> ```
>
> * 为什么要缩减逻辑卷
>   * 实际业务用不了这么大的空间，所以先缩小

### 1、LVM实现流程

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE81d06f0a676ef01a14d91465a1e110c2/20)

### 2、LVM操作指令

1. Physical volume

* pv相关指令

```bash
[root@localhost ~]# pvscan 
  PV /dev/sda2    VG centos          lvm2 [<19.00 GiB / 0    free]
  PV /dev/sdb11   VG vg0             lvm2 [<4.00 GiB / 820.00 MiB free]
  PV /dev/sdc                        lvm2 [20.00 GiB]
  Total: 3 [42.99 GiB] / in use: 2 [22.99 GiB] / in no VG: 1 [20.00 GiB]
[root@localhost ~]# pvdisplay /dev/sdc
  "/dev/sdc" is a new physical volume of "20.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  VG Name               
  PV Size               20.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               St0oFd-Jb0z-Dck6-xQKS-DSbe-EGVI-JATDgi

```

* 创建pv

```bash
pvcreate 分区名字
[root@localhost ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.

```

2. Volume groups

* vgscan、vgdisplay	查看卷组

```bash
[root@localhost ~]# vgscan
  Reading volume groups from cache.
  Found volume group "centos" using metadata type lvm2
  Found volume group "vg0" using metadata type lvm2
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               a2UWNT-LMbO-40ox-0aEu-WNwy-5JF5-gjd5wU
```



* 创建卷组

```bash
[root@localhost ~]# vgcreate vg0 /dev/sdb
```

* 删除卷组

```bash
[root@localhost ~]# vgremove vg0 /dev/sdb
```

3. 查看逻辑卷

* lvscan

```bash
[root@localhost ~]# lvscan 
  ACTIVE            '/dev/centos/swap' [2.00 GiB] inherit
  ACTIVE            '/dev/centos/root' [<17.00 GiB] inherit
  ACTIVE            '/dev/vg0/lv0' [<3.20 GiB] inherit
```

* 创建逻辑卷

```bash
lvcreate -L 20G -n lv0 vg0
```

## 二、逻辑卷扩容

### 1、核心思想

* 扩展逻辑卷物理边界
* 扩展文件系统

### 2、扩展/dev/data/mysql 到20G

```bash
略
```

## 三、逻辑卷缩减

### 1、注意事项

* 不支持在线缩减，必须先卸载
* 先缩减文件系统，再缩减逻辑卷物理边界
* xfs不支持缩减
* 建议备份好数据

### 2、将/dev/db/mysql缩减到10G

* 备份数据
* 卸载逻辑卷
* 检测文件是否有损坏
* **将文件系统缩减到10G**
  * **resize2fs /dev/db/mysql 10G**
* 缩减逻辑卷的实际大小
* 验证
  * mount 重新挂载
  * 查看文件系统大小变化
  * 源文件是否还存在

### 3、xfs文件系统不支持缩减，但是可以实现

* 先备份文件

```bash
[root@localhost ~]# mkdir /tmp/nginx
[root@localhost ~]# cp -a /nginx/ /tmp/nginx/nginx_$(date +%Y_%m_%d)
```

* 取消挂载

```bash
[root@localhost ~]# umount /dev/vg0/lv0 
```



* 删除原有的逻辑卷

```bash
[root@localhost ~]# lvremove /dev/vg0/lv10 
Do you really want to remove active logical volume vg0/lv10? [y/n]: y
  Logical volume "lv10" successfully removed

```

* 新建同样名字、指定大小的逻辑卷

```bash
[root@localhost ~]# lvcreate -L 1G -n lv10 vg0 
WARNING: xfs signature detected on /dev/vg0/lv10 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/vg0/lv10.
  Logical volume "lv10" created.

```

* 再将xfs文件系统重新格式化

```bash
[root@localhost ~]# mkfs.xfs /dev/vg0/lv10 
meta-data=/dev/vg0/lv10          isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```

* 再将备份的数据拷贝回来

```bash
[root@localhost ~]# mv /tmp/nginx/nginx_2022_01_02/* /nginx/

```



* 这样xfs的文件系统就变成了指定大小

```bash
[root@localhost ~]# mount -a
[root@localhost ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  470M     0  470M    0% /dev
tmpfs                   tmpfs     487M  8.8M  478M    2% /run
tmpfs                   tmpfs     487M     0  487M    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        50G  5.4G   45G   11% /
tmpfs                   tmpfs      98M  4.0K   98M    1% /run/user/42
tmpfs                   tmpfs      98M   24K   98M    1% /run/user/0
/dev/sda1               xfs       2.0G  171M  1.9G    9% /boot
/dev/sdc1               ext4      190M  2.9M  173M    2% /db/mysql
/dev/sdc5               ext4      190M  1.6M  175M    1% /db/redis
/dev/sdc6               ext4      190M  1.6M  175M    1% /db/oracle
/dev/sdc7               xfs       197M   11M  187M    6% /db/mongo
/dev/mapper/vg0-lv10    xfs      1014M   33M  982M    4% /nginx

```

* 查看nginx服务可以正常使用

```bash
[root@localhost ~]# ps -ef | grep "nginx"
root       5375      1  0 23:41 ?        00:00:00 nginx: master process /nginx/sbin/nginx
nobody     5377   5375  0 23:41 ?        00:00:00 nginx: worker process
root       5405   2437  0 23:42 pts/0    00:00:00 grep -E --color nginx
[root@localhost ~]# netstat -tunlp | grep "nginx"
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5375/nginx: master  
[root@localhost ~]# 

```



## 四、逻辑卷快照

### 1、注意事项

* 建议将快照设为只读
* 创建快照是基于COW机制，Copy On Write  写时复制
  * 秒级复制是因为快照中只存了原文件的指针，用来指向源文件
  * 此时快照和源文件之间的指针已经建好，只有原逻辑卷中的文件发生变化时，触发COW机制，快照进行复制
  * 快照功能只是记录了文件系统的改变，它并不是对整个文件系统进行备份

### 2、创建快照

```linux
lvcreate -s -p r -L 快照容量 —n 快照名称  逻辑卷名称
```

* -s：快照 
* -p、r：只读
* -L：500M（指的是快照拍摄开始，到原数据开始的变化量）
  * 快照所能容忍的原逻辑卷中数据变化量
  * 如果原逻辑卷中的数据变化量超过指定的大小、快照会自动损毁

## 五、dd命令

* 导入导出命令（导出的空白字符）

  * 创建测试大文件

    * dd if=/dev/zero of =/opt/c.txt bs=1M count=200
      * if=：指定源文件
      * of=：目的文件
      * bs=：每次导出的数据大小

  * 测试硬盘读取速度

    * ```bash
      [root@localhost ~]# dd if=/dev/zero of=/data/mysql/c.txt bs=5M count=20
      20+0 records in
      20+0 records out
      104857600 bytes (105 MB) copied, 0.437974 s, 239 MB/s
      [root@localhost ~]# dd if=/dev/zero of=/db/ljh/c.txt bs=5M count=20
      dd: error writing ‘/db/ljh/c.txt’: No space left on device
      18+0 records in
      17+0 records out
      92917760 bytes (93 MB) copied, 0.41389 s, 224 MB/s
      [root@localhost ~]# 
      
      ```

    * 

## 六、逻辑卷实例

* 逻辑卷	/dev/db/mysql，容量为10G，文件系统ext4，挂载/data/mysql，先添加5G的逻辑卷，再删除3G的逻辑卷，最后再添加10G到逻辑卷mysql中

1. 创建物理卷

```bash
[root@localhost ~]# pvcreate /dev/sdc
[root@localhost ~]# pvdisplay /dev/sdc

```

2. 创建卷组

```bash
[root@localhost ~]# vgcreate db /dev/sdc
[root@localhost ~]# vgscan
```

3. 创建逻辑卷

```bash
[root@localhost ~]# lvcreate -L 10G -n mysql db
[root@localhost ~]# lvscan
```

4. 格式化逻辑卷

```bash
[root@localhost ~]# mkfs -t ext4 /dev/db/mysql
[root@localhost ~]# blkid 
```

5. 实现开机自动挂载逻辑卷

```bash
[root@localhost ~]# vim /etc/fstab 
dev/db/mysql   /data/mysql     ext4    defaults        0       0
```

6. 创建挂载目录并查看挂载情况

```bash
[root@localhost ~]# mount -a
[root@localhost ~]# df -Th
/dev/mapper/db-mysql    ext4      9.8G   37M  9.2G   1% /data/mysql
```

7. 写入文件

```bash
[root@localhost ~]# touch /data/mysql/{1..10}.sql
[root@localhost ~]# ls /data/mysql/
10.sql  2.sql  4.sql  6.sql  8.sql  lost+found
1.sql   3.sql  5.sql  7.sql  9.sql
```

8. 添加5G到逻辑卷mysql中

```bash
[root@localhost ~]# lvextend -L +5G /dev/db/mysql 
[root@localhost ~]# lvscan
[root@localhost ~]# df -Th			//文件系统还是10G，要先更新文件系统
[root@localhost ~]# resize2fs /dev/db/mysql 
[root@localhost ~]# df -Th
```

9. 减小3G的逻辑卷大小

```bash
[root@localhost ~]# umount /data/mysql/			//减小逻辑卷大小需要先卸载文件系统
[root@localhost ~]# resize2fs /dev/db/mysql 12G
resize2fs 1.42.9 (28-Dec-2013)
Please run 'e2fsck -f /dev/db/mysql' first.
			//先将文件系统的大小减到12G，

[root@localhost ~]# e2fsck -f /dev/db/mysql
[root@localhost ~]# resize2fs /dev/db/mysql 12G
[root@localhost ~]# lvresize -L -3G /dev/db/mysql 
[root@localhost ~]# lvdisplay /dev/db/mysql
[root@localhost ~]# mount -a 
[root@localhost ~]# df -Th
/dev/mapper/db-mysql    ext4       12G   41M   12G   1% /data/mysql
```

10. 为逻辑卷增加10G

```bash
//因为卷组空间不够，所以先为卷组增加一个20G的硬盘
[root@localhost ~]# lsblk 
sdd               8:48   0   20G  0 disk 		//可以看到新加的20G硬盘
[root@localhost ~]# pvcreate /dev/sdd
[root@localhost ~]# vgextend db /dev/db
[root@localhost ~]# vgdisplay /dev/db
[root@localhost ~]# lvextend -L +10G /dev/db/mysql
[root@localhost ~]# lvdisplay  /dev/db/mysql
[root@localhost ~]# df -Th			//此时文件系统大小还没有更新
[root@localhost ~]# resize2fs /dev/db/mysql 
[root@localhost ~]# df -Th
/dev/mapper/db-mysql    ext4       22G   44M   21G   1% /data/mysql
扩展成功
```

