[toc]

​	

集群案例

![https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/0F2DEC1B30E54D3AAAE30F3F9D94D677/C4D4145919D54411B7689003E9B20CE3/13430](https://note.youdao.com/yws/public/resource/2dd243e3793f68efa7840971eb8d1a90/xmlnote/0F2DEC1B30E54D3AAAE30F3F9D94D677/C4D4145919D54411B7689003E9B20CE3/13430)

> keepalived配合LVS实现的作用
>
> * 实现调度器的高可用
> * 自动生成LVS负载均衡集群
> * 对后端real-server进行健康状态检查

## 一、需求：

> 后端的web配置LAMP+discuz，通过NFS挂载共享给两个web服务器
>
> 前端部署主备LVS+keepalive

## 二、开始部署

### 1、添加新磁盘，挂载NFS，将Discuz文件共享出去

```bash
[root@nfs_server ~]# df -hT | grep "sdb"
/dev/sdb                ext4      4.8G   20M  4.6G   1% /webdata	
[root@nfs_server ~]# yum install -y nfs-utils rpcbind 
[root@nfs_server ~]# vim /etc/exports
/webdata	192.168.140.0/24(rw,no_root_squash)
[root@nfs_server ~]# chmod o+w /webdata/
[root@nfs_server ~]# systemctl start nfs-server
[root@nfs_server ~]# systemctl enable nfs-server
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
[root@nfs_server ~]# 
[root@nfs_server ~]# showmount -e localhost
Export list for localhost:
/webdata 192.168.140.13,192.168.140.12
[root@nfs_server]# cd /webdata/
[root@nfs_server /web/data/]# unzip -xf Discuz_7.2_FULL_SC_UTF8.zip 
[root@nfs nfs_server]# chmod -R a+w config.inc.php attachments/ forumdata/ uc_client/ 
```

### 2、在web01上安装LAMP平台

```bash
[root@web01 ~]# yum -y install httpd php mariadb-server php-mysql gd php-gd php-devel nfs-utils
[root@web01 ~]# mkdir /web01
[root@web01 ~]# vim /etc/fstab 
[root@web01 ~]# mount -a
df [root@web01 ~]# df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
192.168.152.14:/nfs_server nfs4       20G   53M   20G   1% /web01
[root@web01 ~]# systemctl start httpd
[root@web01 ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@web01 ~]# systemctl start mariadb
[root@web01 ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@web01 ~]# rm -rf /etc/httpd/conf.d/welcome.conf 
[root@web01 ~]# vim /etc/httpd/conf.d/discuz.conf
#因为使用虚拟IP，所以让主机监控在所有网段上的80端口
<VirtualHost 0.0.0.0:80>
        ServerName      discuz.linux.com
        DocumentRoot    /web01
        ErrorLog        /var/log/httpd/discuz_error.log
        CustomLog       /var/log/httpd/discuz_access.log        combined
</VirtualHost>

<Directory "/web01">
        Require all granted
</Directory>
[root@web01 ~]# systemctl restart httpd
#开始为discuz用户授权
[root@web01 ~]# mysql -uroot
MariaDB [(none)]> create database discuz charset utf8;

MariaDB [(none)]> grant all on discuz.* to "discuz"@"localhost" identified by "redhat";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

> 浏览器输入http://192.168.152.12/upload/install 开始安装

### 3、在web02上安装LAMP平台

> 操作与上边一样，不再赘述
>
> 浏览器输入http://192.168.152.13/upload/isntall  再次安装

### 4、在lvs1上安装ipvsadm和keepalived

```bash
[root@lvs01 ~]# yum -y install keepalived ipvsadm
[root@lvs01 ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id master_lvs					# 定义调度器的id
}

vrrp_instance web_service {				# web_service虚拟组名
    state MASTER						# 主备状态
    interface ens33						# VIP出现的网卡
    virtual_router_id 66					# 组ID
    priority 100							# 优先级  1---255， 数字越大，优先级越高
    advert_int 1							# 心跳的时间间隔
    authentication {
        auth_type PASS
        auth_pass redhat					# 认证密码
    }
    virtual_ipaddress {
        192.168.152.100					# VIP
    }
}

virtual_server 192.168.152.100 80 {			
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 300
    protocol TCP

    real_server 192.168.152.12 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            connect_port 80
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.152.13 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            connect_port 80
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
[root@lvs01 ~]# systemctl start keepalived.service

```

### 5、在备调度器上安装ipvsadm+keepalived

```bash
[root@master_lvs ~]# rsync -av /etc/keepalived/keepalived.conf root@192.168.152.11:/etc/keepalived/
[root@slave_lvs ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id slave_lvs				# 调度器ID
}			

vrrp_instance web_service {
    state BACKUP					# 状态
    interface ens33
    virtual_router_id 66
    priority 50						# 优先级
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.152.100
    }
}

virtual_server 192.168.152.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 300
    protocol TCP

    real_server 192.168.152.12 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            connect_port 80
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.152.13 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            connect_port 80
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
[root@lvs02 ~]# systemctl start keepalived.service

```

### 6、查看主调度的VIP、负载均衡规则

```bash
[root@slave_lvs ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.140.100:80 rr persistent 300
  -> 192.168.140.12:80            Route   1      0          0         
  -> 192.168.140.13:80            Route   1      0          1   
  
 虚拟IP同一时间只能存在一个调度器上
```

### 7、测试通过VIP正常访问服务

http://192.168.152.100/upload/

### 8、关闭主调度器上的keepalived服务，验证VIP会转移到备调度；客户端通过VIP仍然可正常访问服务