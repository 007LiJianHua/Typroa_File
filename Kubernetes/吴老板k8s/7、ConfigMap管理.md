[toc]

## 一、作用

> ConfigMap也是kubernetes集群中的一种资源对象
>
> 用于向集群内的Pod传递环境变量，或者传递配置文件 

## 二、创建使用ConfigMap传递环境变量

### 1、在yaml中定义数据，并通过deployment部署两个pod

```bash
apiVersion: v1			#版本
kind: ConfigMap			#类型
metadata:
  name: mysql-data		#名字
data:
  data01: happy			#在config中定义的键值
  mysql_pwd: redhat
---
apiVersion: apps/v1		#通过deployment部署两个mysqlpod
kind: Deployment
metadata:
        name: test-mysql
spec:
        replicas: 2
        selector:
                matchLabels:
                        name: test-mysql
        template:
                metadata:
                        labels:
                                name: test-mysql
                spec:
                        containers:
                        - name: dbserver
                          image: mysql:5.7		#这里往上就是部署了两个mysql的POD
                          env:					#定义环境变量
                             - name: MYSQL_DATABASE		#环境变量名子
                               value: jishu				#环境变量值
                             - name: test1				#环境变量名字
                               valueFrom:				#环境变量值的来源
                                  configMapKeyRef:		#使用configmap
                                        name: mysql-data#configmap的名字
                                        key: data01		#使用configmap中的哪个键
                             - name: MYSQL_ROOT_PASSWORD#环境变量名字
                               valueFrom:				#环境变量值的来源
                                  configMapKeyRef:		#使用configmap
                                        name: mysql-data#configmap的名字
                                        key: mysql_pwd	#使用configmap中的哪个键
                                                                                 
```

### 2、创建POD

```bash
[root@k8s-master01 demo_configMap]# kubectl create -f test_configMap.yaml 
```

```bash
[root@k8s-master01 demo_configMap]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
test-mysql-65fb5966f8-fb7g9   1/1     Running   0          17m
test-mysql-65fb5966f8-rbjf8   1/1     Running   0          17m
```

### 3、查看传递的环境变量

```bash
[root@k8s-master01 demo_configMap]# kubectl exec -it test-mysql-65fb5966f8-fb7g9 bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-mysql-65fb5966f8-fb7g9:/# echo $test1
happy
root@test-mysql-65fb5966f8-fb7g9:/# mysql -uroot -predhat
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jishu              |
| mysql              |
| performance_schema |
| sys                |

```

## 三、使用configmap传递配置文件

### 1、编写configmap

```bash
apiVersion: v1
kind: ConfigMap
metadata:
   name: test-cfg-03
data:
   cache_host: memcached01
   cache_port: "11211"
   my.cnf: |
      [mysqld]
      server_id=10
      log_bin=master
---
apiVersion: apps/v1
kind: Deployment
metadata:
        name: test-mysql
spec:
        replicas: 2
        selector:
                matchLabels:
                        name: test-mysql
        template:
                metadata:
                        labels:
                                name: test-mysql
                spec:
                        containers:
                        - name: dbserver
                          image: mysql:5.7
                          env:
                          - name: MYSQL_ROOT_PASSWORD
                            value: redhat
                          volumeMounts:
                          - name: mysql-config-volume
                            mountPath: "/etc/mysql"
                        volumes:
                        - name: mysql-config-volume
                          projected:
                             sources:
                             - configMap:
                                 name: test-cfg-03
```

### 2、创建

```bash
[root@k8s-master01 demo_configMap]# kubectl create -f test2-configmap.yaml
```

### 3、查看

```bash
[root@k8s-master01 demo_configMap]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
test-mysql-745bfc957b-26fqp   1/1     Running   0          5m19s
test-mysql-745bfc957b-shnq2   1/1     Running   0          5m19s
```

### 4、测试

```bash
[root@k8s-master01 demo_configMap]# kubectl exec -it test-mysql-745bfc957b-26fqp bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@test-mysql-745bfc957b-26fqp:/# 
root@test-mysql-745bfc957b-26fqp:/# ls
bin  boot  dev	docker-entrypoint-initdb.d  entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@test-mysql-745bfc957b-26fqp:/# mysql -uroot -predhat
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.37-log MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| master.000003 |      154 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

