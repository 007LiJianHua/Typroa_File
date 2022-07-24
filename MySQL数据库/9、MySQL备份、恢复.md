[toc]

# MySQL备份、恢复

## 一、备份类型

### 1、根据服务是否在线

> * 热备份
>
> * - 在备份的过程，可以继续执行数据的读写操作 
>   - 服务持续运行的状态 
>
> * 温备份
>
> * - 在备份的过程，只允许读操作、不允许写操作
>   - 锁表
>
> * 冷备份（**推荐**）
>
> * - 停止数据库服务、备份数据
>   - 影响业务运行
>   - **安全、可靠、稳定**
>
> * **禁止热插拔（可能会丢失数据）**

### 2、根据备份的数据量大小

* 只要是数据备份，就是这三种备份

> * 完全备份
>
> * - 每次备份，备份所有数据
>   - 假如有300G，每次备份都会产生300G数据
>
> * 增量备份
>
> * - 仅备份上一次完全备份后变化的数据
>   - 仅备份上一次增量备份后变化的数据 
>
> * 差异备份 
>
> * - 仅备份上一次完全备份后变化的数据
>
> **备份策略:**
>
> 完全 + 增量 ：**顺序不能乱**
>
> 完全 + 差异 

### 3、根据备份形成的结果

> - 逻辑备份
>
> - - 将数据库执行的写操作转换成SQL语句，保存到文件中
>
>   - .sql
>
>   - 备份工具
>
>   - - mysqldump 
>
> - 物理备份
>
> - - 直接拷贝数据文件
>
>   - 备份工具
>
>   - - cp、tar
>     - xtrabackup	开源 percona		https://www.percona.com/

### 4、要备份的东西

> * 数据------/var/lib/mysql/----丢失的数据在这里
> * 二进制日志-----独立的磁盘
> * 配置文件----/etc/my.cnf

## 二、mysqldump工具（逻辑备份）

### 1、常用选项

> * -u：用户名
> * -p：密码
> * --all-databases：备份所有数据库

```bash
[root@localhost ~]# mysqldump -uroot -predhat --all-databases > /tmp/a.sql
```

* --databases <数据库名><数据库名>

```bash
[root@localhost ~]# mysqldump -uroot -predhat --databases jiaowu > /tmp/b.sql
```

* 备份单个表，库名 表名

```bash
[root@localhost ~]# mysqldump -uroot -predhat jiaowu tutors > /tmp/c.sql
```

> * --lock-all-tables    锁表
> * --flush-logs     执行二进制日志滚动
> * --master-data={1|2}
>   * 以change master to 记录当前正在使用的二进制日志、最后一个事件的Position

### 2、注意

> **MySQL在恢复数据时，二进制日志一样会记录写操作，建议临时关闭**
>
> ```bash
> mysql> set sql_log_bin=0;
> 
> mysql> source 备份文件名称;
> ```
>

### 3、案例：模拟4天操作，完全+增量（rpm）

1、将需要备份的文件放到新的磁盘上

```bash
[root@localhost ~]# df -hT | grep backup
/dev/sdd                ext4       20G   45M   19G   1% /mysql/backup
```

2、第一天完全备份所有的数据库

```bash
[root@localhost ~]# mysqldump -uroot -predhat --lock-all-tables --master-data=2 --all-databases > /mysql/backup/data_$(date +%F_%T).sql
```

3、模拟写操作，第二天增量备份

```bash
mysql> delete from jiaowu.tutors where age > 60;
```

```bash
#关于开始的位置需要到第一天完全备份的change master to 的位置查看
[root@localhost ~]# mysqlbinlog --start-position=12644  /mysql/log/master.000003 > /mysql/backup/data_$(date +%F_%T).sql
```

4、模拟写操作，第三天增量备份

```bash
mysql> insert into jiaowu.tutors(Tname) values("user01"),("user02");
```

```bash
#这个起始位置也是同上
[root@localhost ~]# mysqlbinlog --start-position=12998 /mysql/log/master.000003 > /mysql/backup/data_$(date +%F_%T).sql
```

5、模拟第四天操作，不进行增量备份

```bash
mysql> insert into jiaowu.tutors(Tname) values("user03"),("user04");
```

6、模拟故障

```bash
[root@localhost ~]# rm -rf /var/lib	/mysyl/*
```

7、恢复数据库运行

```bash
[root@localhost ~]# systemctl start mysqld
```

8、恢复第一天的完全备份

```bash
[root@localhost ~]# mysql -uroot -predhat < /mysql/backup/data_2021-06-03_10\:54\:16.sql 
```

9、恢复第二、第三天增量

```bash
[root@localhost ~]# mysql -uroot -predhat < /mysql/backup/data_2021-06-03_10\:56\:52.sql 
```

```bash
mysql> source /mysql/backup/data_2021-06-03_11:00:38.sql;
```

10、通过二进制日志恢复第四天的数据，

```bash
[root@localhost ~]# mysqlbinlog --start-position=13277 /mysql/log/master.000003 | mysql -uroot -predhat
```

## 三、xtrabackup工具（物理备份）

### 1、安装xtrabackup

```bash
#安装软件包和依赖
```

### 2、完全备份

> innobackupex --user=用户名 --password=密码  备份目录

```bash
[root@localhost ~]# innobackupex --user=root --password=redhat --socket=/tmp/mysql.sock /mysql/backup/
```

### 3、恢复完全备份

1、准备完全备份，将日志中位同步的数据写入到数据文件中

```bash
innobackex --apply-log <完全备份目录>
```

2、恢复完全备份目录

```bash
innobackex --copy-back <完全备份目录>
```

3、恢复结束后要将文件的所属主和所属组改为mysql

```bash
[root@localhost ~]# chown -R mysql.mysql /mysql/data/
```

### 4、执行增量备份

innobackex --user=<用户ing> 

--password=<密码> 

--incremental <备份目录>

--incremental-basedir=<上一次备份目录>



* 注意：
  * incremental-basedir=
    * 第一次增量备份，完全备份目录
    * 第二次增量备份，指上一次增量备份目录

### 5、准备增量备份文件

```bash
innobackupex ‐‐apply‐log ‐‐redo‐only <完全备份目录> ‐‐incremental‐dir=<第二个增量备份目录>
```

### 6、案例：模拟4天备份  完全+增量

* 第一天完全备份

```bash
[root@localhost ~]# innobackupex --user=root --password=redhat --socket=/tmp/mysql.sock /mysql/backup/ 
```

* 模拟写操作，第二天的增量备份

```mysql
mysql> delete from jiaowu.tutors where age > 50;
```

```bash
[root@localhost ~]# innobackupex --user=root --password=redhat --socket=/tmp/mysql.sock --incremental /mysql/backup/ --incremental-basedir=/mysql/backup/2021-06-03_15-05-57/
```

* 模拟写操作，第三天的增量备份

```mysql
mysql> insert into tutors(Tname) values("userA"),("userB");
```

```bash
[root@localhost ~]# innobackupex --user=root --password=redhat --socket=/tmp/mysql.sock --incremental /mysql/backup/ --incremental-basedir=/mysql/backup/2021-06-03_15-08-11/
```

* 模拟写操作，不备份

```mysql
mysql> insert into tutors(Tname) values("userC"),("userD");
```

* 模拟故障

```bash
[root@localhost ~]#systemctl stop mysqld
[root@localhost ~]# rm -rf /var/lib/mysql/*
```

* 准备完全备份
  * 将未备份的写操作记录到日志文件中

```bash
[root@localhost ~]# innobackupex --apply-log --redo-only /mysql/backup/2021-06-03_15-05-57/
```

* 一次准备每个增量备份

```bash
[root@localhost ~]# innobackupex --apply-log --redo-only /mysql/backup/2021-06-03_15-05-57/ --incremental-dir=/mysql/backup/2021-06-03_15-08-11/
[root@localhost ~]# innobackupex --apply-log --redo-only /mysql/backup/2021-06-03_15-05-57/ --incremental-dir=/mysql/backup/2021-06-03_15-10-12/
```

* 恢复数据

```bash
[root@localhost ~]# innobackupex --copy-back /mysql/backup/2021-06-03_15-05-57/

[root@localhost ~]# chown -R mysql.mysql /mysql/data/
```

* 通过二进制文件进行即使点还原

```bash
[root@localhost ~]# mysqlbinlog --start-position=806 /mysql/log/master.000007 | mysql -uroot -predhat0
```



