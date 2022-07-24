[toc]

## 一、文件系统存储

### 1、创建MDS服务

* ceph文件系统存储依赖于MDS，用来保存元数据信息

#### 1）在ceph-node01编辑配置文件，添加如下内容

* 这条指令是可以删除存储池

```bash
[root@ceph-node01 ceph]# tail -n 1 ceph.conf 
mon_allow_pool_delete = true
```

#### 2）同步配置文件

```bash
[root@ceph-node01 ceph]# ceph-deploy --overwrite-conf admin ceph-node01 ceph-node02 ceph-node03
```

#### 3）创建三个MDS

```bash
[root@ceph-node01 ceph]# ceph-deploy mds create ceph-node01 ceph-node02 ceph-node03
```

### 2、创建文件系统存储池

* 一个文件系统存储需要两个池RADOS池，一个用来存储真实的数据，一个用来存储元数据信息
  * 下面分别创建两个存储池，名字为cephfs_pool、 cephfs_metadata
  * 分别指定池的对应的PG数量为   128、64
* 存储池对应PG数量参考

- - 少于5个OSD则PG数为128
  - 5-10个OSD则PG数为512
  - 10-50个OSD则PG数为1024
  - 如果有更多的OSD需要自己理解计算

* PG计算公式

- - **Total PGs = ((Total_number_of_OSD \* 100) / max_replication_count) / pool_count**
  - 结算的结果往上取靠近2的N次方的值。比如总共OSD数量是160，复制份数3，pool数量也是3，那么按上述公式计算出的结果是1777.7。取跟它接近的2的N次方是2048，那么每个pool分配的PG数量就是2048

```bash
[root@ceph-node01 ceph]# ceph osd pool create cephfs_pool 128
pool 'cephfs_pool' created
[root@ceph-node01 ceph]# ceph osd pool create cephfs_metadata 64
pool 'cephfs_metadata' created
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   2 pools, 192 pgs
    objects: 0  objects, 0 B
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail
    pgs:     192 active+clean
```

### 3、查看存储池

#### 1）查看存储池

```bash
[root@ceph-node01 ceph]# ceph osd pool ls
cephfs_pool
cephfs_metadata
```

#### 2）查看详细信息

```bash
[root@ceph-node01 ceph]# ceph osd pool get cephfs_pool all
size: 3
min_size: 2
pg_num: 128
pgp_num: 128      # 用于管理pg自身的PG
crush_rule: replicated_rule
hashpspool: true
nodelete: false
nopgchange: false
nosizechange: false
write_fadvise_dontneed: false
noscrub: false
nodeep-scrub: false
use_gmt_hitset: 1
auid: 0
fast_read: 0
```

### 4、创建ceph文件系统

```bash
[root@ceph-node01 ceph]# ceph fs new cephfs cephfs_metadata cephfs_pool 
new fs with metadata pool 2 and data pool 1
#查看文件系统
[root@ceph-node01 ceph]# ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_pool ]
#查看mds状态
[root@ceph-node01 ceph]# ceph mds stat
cephfs-1/1/1 up  {0=ceph-node03=up:active}, 2 up:standby

#最后查看文件系统会发现多出了mds这一行信息
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    mds: cephfs-1/1/1 up  {0=ceph-node03=up:active}, 2 up:standby
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   2 pools, 192 pgs
    objects: 22  objects, 2.2 KiB
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail
    pgs:     192 active+clean
```

### 5、客户端挂载

* ceph集群默认启用了cephx认证，客户端必须通过认证

#### 1）在集群节点上查看客户端认证的key

```bash
[root@ceph-node01 ceph]# cat /etc/ceph/ceph.client.admin.keyring 
[client.admin]
	key = AQDP9oVh4TrYHBAA9vFaL0GKWA9D6mhzcK5JGw==
	caps mds = "allow *"
	caps mgr = "allow *"
	caps mon = "allow *"
	caps osd = "allow *"
```

#### 2）将key导出并发送

```bash
[root@ceph-node01 ceph]# ceph-authtool -p /etc/ceph/ceph.client.admin.keyring > /root/admin.key
[root@ceph-node01 ceph]# cat /root/admin.key
AQDP9oVh4TrYHBAA9vFaL0GKWA9D6mhzcK5JGw==
[root@ceph-node01 ceph]# rsync -av /root/admin.key root@192.168.183.13:/root/
```



#### 3）客户端挂载

```bash
[root@client ~]# yum -y install ceph-fuse
[root@client ~]# mkdir /test1
[root@client ~]# mount -t ceph node1:6789:/ /test1-o name=admin,secretfile=/root/admin.key
```

* 在挂载点测试写入数据，可在web管理界面可看到数据使用率

  \# dd if=/dev/zero of=/test1/file01.txt bs=1M count=5000

### 6、删除文件系统存储

#### 1）客户端挂载

#### 2）停到所有集群的MDS

```bash
systemctl stop ceph-mds.target
```

#### 3）删除文件系统

```bash
[root@ceph-node01 ceph]# ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_pool ]

[root@ceph-node01 ceph]# ceph fs rm cephfs --yes-i-really-mean-it
```

#### 4）删除存储池

```bash
[root@ceph-node01 ceph]# ceph osd pool delete cephfs_metadata cephfs_metadata --yes-i-really-really-mean-it
pool 'cephfs_metadata' removed
[root@ceph-node01 ceph]# ceph osd pool delete cephfs_pool cephfs_pool --yes-i-really-really-mean-it
pool 'cephfs_pool' removed
```

> 删除存储池时，如果出现下面错误，确认在配置文件中添加了mon_allow_pool_delete = true配置项， 需要将集群节点的ceph-mon.target重启
>
> Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool

```bash
[root@ceph-node01 ceph]# systemctl restart ceph-mon.target
```

## 二、块存储

### 1、将ceph配置文件同步到客户端

```bash
[root@ceph-node01 ceph]# ceph-deploy admin client.linux.com
```

<font color="red">**以下在客户端操作**</font>

### 2、创建存储池并优化

```bash
[root@client ~]# ceph osd pool create rbd_pool 128
pool 'rbd_pool' created
[root@client ~]# rbd pool init rbd_pool
```

### 3、创建卷

```bash
[root@client ~]# rbd create volume1 --pool rbd_pool --size 3000
#查看卷
[root@client ~]# rbd ls rbd_pool
volume1
[root@client ~]# rbd info volume1 -p rbd_pool
rbd image 'volume1':
	size 2.9 GiB in 750 objects
	order 22 (4 MiB objects)
	id: 5e686b8b4567
	block_name_prefix: rbd_data.5e686b8b4567
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
	op_features: 
	flags: 
	create_timestamp: Sat Nov  6 12:21:38 2021
```

### 4、将卷映射为块设备

```bash
[root@client ~]# rbd map rbd_pool/volume1
rbd: sysfs write failed
RBD image feature set mismatch. You can disable features unsupported by the kernel with "rbd feature disable rbd_pool/volume1 object-map fast-diff deep-flatten".
In some cases useful info is found in syslog - try "dmesg | tail".
rbd: map failed: (6) No such device or address
```

* 出现报错是因为“特性冲突”，
  * 由于操作系统底层原因，导致映射 失败， 可在系统层将对应的特性关掉

```bash
 rbd feature disable rbd_pool/volume1 object-map fast-diff deep-flatten
```

* 重新映射

```bash
[root@client ~]# rbd map rbd_pool/volume1
/dev/rbd0
[root@client ~]# rbd showmapped
id  pool     image    snap  device    
0  rbd_pool  volume1    -   /dev/rbd0 
```

### 5、格式化挂载使用

```bash
[root@client ~]# mkfs.xfs /dev/rbd0
[root@client ~]# mkdir /test2
[root@client ~]# mount /dev/rbd0 /test2
```

### 6、块设备扩容

#### 1）扩容

```bash
[root@client ~]# rbd resize --size 5000 rbd_pool/volume1 
Resizing image: 100% complete...done.
[root@client ~]# xfs_growfs -d /test2/
```

#### 2）缩减

* 不支持在线缩减，缩减后重新格式化
* 写在磁盘

```bash
[root@client ~]# umount /test2 
[root@client ~]# rbd resize --size 2000 rbd_pool/volume1 --allow-shrink 
Resizing image: 100% complete...done.
[root@client ~]# mkfs.xfs -f /dev/rb
[root@client ~]# mount /dev/rbd0 /test2/
[root@client ~]# df -hT
/dev/rbd0               xfs       2.0G   33M  2.0G   2% /test2
```

### 7、删除存储块

```bash
[root@client ~]# umount /mnt
[root@client ~]# rbd unmap /dev/rbd0
[root@client ~]# ceph osd pool delete rbd_pool rbd_pool --yes-i-really-really-mean-it
pool 'rbd_pool' removed
```

## 三、对象存储

* 对象存储依赖于rgw服务

### 1、在ceph-node01创建rgw服务

```bash
[root@ceph-node01 ceph]# ceph-deploy rgw create ceph-node01
```

### 2、查看服务

```bash
[root@ceph-node01 ceph]# ceph -s
  cluster:
    id:     1ca6c582-8598-4300-93c4-4da0f2b16d32
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node01,ceph-node02,ceph-node03
    mgr: ceph-node01(active), standbys: ceph-node02, ceph-node03
    osd: 3 osds: 3 up, 3 in
    rgw: 1 daemon active
 
  data:
    pools:   4 pools, 32 pgs
    objects: 187  objects, 1.1 KiB
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail
    pgs:     32 active+clean
 
  io:
    client:   35 KiB/s rd, 0 B/s wr, 52 op/s rd, 34 op/s wr
 [root@ceph-node01 ceph]# netstat -tunlp | grep 7480
tcp        0      0 0.0.0.0:7480            0.0.0.0:*               LISTEN      12104/radosgw   
```

### 3、在客户端测试连接对象网关

#### 1）创建测试用户

```bash
[root@client ~]# radosgw-admin user create --uid="testuser" --display-name="first user" 
```

#### 2）获取用户的key

```bash
[root@client ~]# radosgw-admin user create --uid="testuser" --display-name="first user" | grep -E "access_key|secret_key"
            "access_key": "FPUSYUBXCS18FU4J7Q8M",
            "secret_key": "NwM0GvYkVSzFAvvotZ7fnX5IHsxjrczNJaA3rTvX"
```

#### 3）安装s3cmd工具

```bash
[root@client ~]# yum install -y s3cmd
```

#### 4）编辑配置文件

```bash
[root@client ~]# cat /root/.s3cfg 
[default]
access_key = FPUSYUBXCS18FU4J7Q8M
secret_key = NwM0GvYkVSzFAvvotZ7fnX5IHsxjrczNJaA3rTvX
host_base = 192.168.183.10:7480
host_bucket = 192.168.183.10:7480/%(bucket)
cloudfront_host = 192.168.183.10:7480
use_https = False
```

#### 5）创建bucket桶

```bash
[root@client ~]# s3cmd mb s3://test_bucket
Bucket 's3://te_bucket/' created
[root@client ~]# s3cmd ls
2021-11-06 04:36  s3://te_bucket
```

#### 6）测试上传文件

```bash
[root@client ~]# s3cmd put /etc/fstab s3://te_bucket
upload: '/etc/fstab' -> 's3://te_bucket/fstab'  [1 of 1]
 465 of 465   100% in    1s   249.67 B/s  done
```

#### 7）测试下载文件

```bash
[root@client ~]# s3cmd get s3://te_bucket/fstab
download: 's3://te_bucket/fstab' -> './fstab'  [1 of 1]
 465 of 465   100% in    0s     9.27 KB/s  done
```

