[toc]

## 一、playbook介绍

* 优势
  * 便于功能的重用
* 本质上就是.yml结尾的文件
* 遵循YAML语法编写

### 1、YAML语法

* 一个键对应一个值，冒号后边必须要有空格
  * name: nginx
* 一个键对应多个值时，分行写
  * 键：
    * \- 值
    * \- 值
    * \- 值
* 同级别的代码要有相同的缩进，建议是四个空格

### 2、playbook大体结构

```bash
- hosts: 主机或主机组 
  user: 用户名
  tasks:
      - name: 任务名称
	    模块名称: 
	      参数  
	      参数  
	      参数 
      - name: 任务名称
	    模块名称: 参数  参数  参数 

```



## 二、playbook简单应用

### 1、在被管理机创建openstack用户，指定用户shell为/sbin/nologin

#### 1）编写剧本

```bash
[root@ansible ~]# cat /opt/work/userCreate.yml
- hosts: appserver
  user: root
  tasks:
       - name: create openstack user
         user: name=openstack shell=/sbin/nologin state=present
```

#### 2）执行剧本

```bash
[root@ansible ~]# ansible-playbook /opt/work/userCreate.yml 
```

### 2、剧本在执行用户自定义的任务前，会自动执行一个名称为Gathering Facts的任务，即调用setup模块搜集被管理机的状态数据，可添加如下参数取消该行为

```bash
[root@ansible ~]# cat /opt/work/userCreate02.yml
- hosts: appserver
  user: root
  gather_facts: false
  tasks:
      - name: create hadoop user
        user: name=hadoop shell=/sbin/nologin state=present
```

## 三、playbook中变量的使用

### 1、调用变量的语法

```bash
{{ 变量名称 }}
```

### 2、在playbook中直接定义变量

* 这里测试有个错误？

```bash
[root@ansible ~]# cat /opt/work/test1.yml
- hosts: appserver
  user: root
  gather_facts: false
  vars:
     - username: "user01"
     - sh_name: "/sbin/nologin"
  tasks:
     - name: create user
       user: name={{ username }} shell={{ sh_name }} state=present
```

### 3、在主机清单文件中为主机组定义变量

```bash
[appserver:vars]
username="user02"
sh_name="/sbin/nologin"
```

```bash
[root@ansible ~]# cat /opt/work/test1.yml
- hosts: appserver
  user: root
  gather_facts: false
  tasks:
     - name: create user
       user: name={{ username }} shell={{ sh_name }} state=present
```

### 4、在主机清单文件中为单个主机组定义变量

```bash
[appserver]
192.168.140.11 username="user03" sh_name="/sbin/nologin"
192.168.140.12 username="user003" sh_name="/bin/bash"
192.168.140.13 ansible_ssh_user="root" ansible_ssh_pass="redhat" ansible_ssh_port=22 username="user0003" sh_name="/bin/sync"
```

### 5、在外部文件中定义变量

```bash
[root@ansible ~]# cat /opt/work/userInfo
username: "user04"
sh_name: "/bin/false"
```

```bash
[root@ansible ~]# cat /opt/work/test1.yml 
- hosts: appserver
  user: root
  gather_facts: false
  vars_files:
      - /opt/work/userInfo
  tasks:
     - name: create user
       user: name={{ username }} shell={{ sh_name }} state=present
```

#### 1）加密变量文件

```bash
[root@ansible ~]# ansible-vault encrypt /opt/work/userInfo 
New Vault password: 
Confirm New Vault password: 
Encryption successful

[root@ansible ~]# cat /opt/work/userInfo 
$ANSIBLE_VAULT;1.1;AES256
64343632653338376466326633373664333936646662386436333935623832333163316536366436
6631373063306364303332386533663764313633633836360a373364623431383063333233633130
63323066653237343863363564326165346436633165643763323366393038303534613734306538
3330623037656532370a623163363961353138376565366537613265353264336230353333343963
63373139306432353765336363306662366463396366386437383031303938323736626462333835
6335336664336631373830613866623930386336656438656263
```

#### 2）在执行剧本时，添加--ask-vault参数

```bash
[root@ansible ~]# ansible-playbook --ask-vault /opt/work/test1.yml 
Vault password: 
```

#### 3）解密文件

```bash
[root@ansible ~]# ansible-vault decrypt /opt/work/userInfo
Vault password: 
Decryption successful

[root@ansible ~]# cat /opt/work/userInfo
username: "user04"
sh_name: "/bin/false"
```

## 四、部署MySQL

### 1、为不同的主机定义变量，

```bash
[root@ansible ~]# grep -A 3 "appserver" /etc/ansible/hosts 

[appserver]
192.168.140.11 server_id=11
192.168.140.12 server_id=12
192.168.140.13 ansible_ssh_user="root" ansible_ssh_pass="redhat" ansible_ssh_port=22 server_id=13
```

### 2、准备MySQL配置文件模板

```bash
[root@ansible ~]# sed -n '1,3p' /opt/work/my.cnf 
[mysqld]
server_id={{ server_id }}
log_bin=master
........
```

### 3、编写剧本

```bash
[root@ansible ~]# cat /opt/work/installMySQL.yml 
- hosts: appserver
  user: root
  gather_facts: false
  tasks:
      - name: install MySQL
        yum: name=mariadb-server state=present

      - name: push my.cnf
        template: src=/opt/work/my.cnf dest=/etc/my.cnf

      - name: start MySQL daemon
        service: name=mariadb state=started enabled=yes
```

### 4、执行剧本

```bash
[root@ansible ~]# ansible-playbook /opt/work/installMySQL.yml
```

### 5、验证

```bash
[root@ansible ~]# ansible appserver -m shell -a 'netstat -antp | grep mysql'
192.168.140.12 | CHANGED | rc=0 >>
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      22303/mysqld        
192.168.140.11 | CHANGED | rc=0 >>
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      22320/mysqld        
192.168.140.13 | CHANGED | rc=0 >>
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      21716/mysqld        
```

## 五、条件判断的实现

* when：条件

```bash
- hosts: appserver
  user: root
  tasks:
      - name: create userA
        user: name=userA state=present
        when: ansible_default_ipv4["address"]  == "192.168.140.11"

      - name: create userB
        user: name=userB state=present
        when: ansible_default_ipv4["address"] == "192.168.140.12"

      - name: create userC
        user: name=userC state=present
        when: ansible_default_ipv4["address"] == "192.168.140.13"
```

## 六、循环的实现

### 1、loop关键字

```bash
- hosts: appserver
  user: root
  tasks: 
      - name: create userAA/BB/CC
        user: name={{ item }} state=present
        loop:
           - "userAA"
           - "userBB"
           - "userCC"
```

### 2、通过字典为item变量赋值

```bash
- hosts: appserver
  user: root
  gather_facts: false
  tasks:
      - name: create user11/22/33
        user: name={{ item["username"] }} shell={{ item["sh_name"] }} state=present
        loop:
            - { "username":"user11", "sh_name":"/sbin/nologin" }
            - { "username":"user22", "sh_name":"/bin/bash" }
            - { "username":"user33", "sh_name":"/bin/sync" }
```

## 七、Jinja模板应用

### 1、Jinja模板

* 支持在配置文件中调用变量 {{ 变量名称 }}
* 增加配置文件灵活性
* 建议配置文件以j2结尾
* JInja模板要使用templete模块推送，触发变量替换

```bash
[appserver:vars]
mysql_port=3307
```

```bash
[root@ansible ~]# sed -n '1,2p' /opt/work/my.cnf.j2
[mysqld]
port={{ mysql_port }}
```

```bash
[root@ansible ~]# cat /opt/work/test5.yml
- hosts: appserver
  user: root
  tasks:
     - name: copy my.cnf
       template: src=/opt/work/my.cnf.j2 dest=/etc/my.cnf
```

**使用setup模块的状态数据定义配置文件**

```bash
[mysqld]
bind-address={{ ansible_all_ipv4_addresses[0] }}
port={{ mysql_port }}
```

### 2、handles组件

* 与task同级别组件，默认情况，handers下定义的任务是不会自动执行的，只有在满足了一定的条件，由特定的条件触发其执行
* 应用场景
  * 检测配置文件，自动重启服务加载配置文件

```bash
- hosts: appserver
  user: root
  tasks:
     - name: copy my.cnf
       template: src=/opt/work/my.cnf.j2 dest=/etc/my.cnf
       notify: restart MySQL daemon
  handlers:
     - name: restart MySQL daemon
       service: name=mariadb state=restarted
```

## 八、角色 role

### 1、角色介绍

* 本质上就是个目录
  * /etc/ansible/roles
* 一个需求对应一个角色

#### 1）创建角色

```bash
[root@ansible ~]# cd /etc/ansible/roles/
[root@ansible roles]# ansible-galaxy init nginx 
- Role nginx was created successfully
```

#### 2）目录结构说明

```bash
[root@ansible ~]# tree /etc/ansible/roles/nginx/
/etc/ansible/roles/nginx/
├── defaults		#默认参数
│   └── main.yml
├── files			#普通文件、软件安装包 
├── handlers		#触发的操作
│   └── main.yml
├── meta			#元数据
│   └── main.yml
├── README.md
├── tasks			#常规任务、操作
│   └── main.yml
├── templates		#jinja模块
├── tests			#剧本相关的测试代码
│   ├── inventory
│   └── test.yml
└── vars			#定义变量
    └── main.yml
```

- **同一个角色中，相互引用数据时，不需要添加任何目录，直接调用即可**

## 九、案例

### 1、ansible部署zabbix-agent配合自动注册

* main.yml文件

```bash
[root@zabbix-server roles]# cat zabbix/tasks/main.yml
---
# tasks file for zabbix
    - name: get zabbix-agent repo
      yum_repository:
        name: zabbix
        description: zabbix
        file: zabbix
        baseurl: https://mirrors.aliyun.com/zabbix/zabbix/4.4/rhel/7/x86_64/
        gpgcheck: no
        enabled: yes 

    - name: install zabbix-agent
      yum:
        name: zabbix-agent
        state: present

    - name: push zabbix-agent config file
      template:
        src: zabbix_agentd.conf
        dest: /etc/zabbix/zabbix_agentd.conf
      notify: restart zabbix-agent 

    - name: start zabbix-agent
      service:
        name: zabbix-agent
        state: started

```

```bash
[root@zabbix-server roles]# cat /opt/install_zabbix.conf.yaml 
- hosts: dbserver
  user: root
  roles:
    - zabbix

```

### 2、ansible部署nginx

```bash
[root@zabbix-server roles]# cat nginx/tasks/main.yml 
---
# tasks file for nginx
- name: get env
  yum: 
    name={{ item }}
    state=present
  loop:
    - gcc
    - openssl
    - zlib-devel
    - pcre-devel

- name: push nginx.tar.gz
  copy:
    src: /root/nginx-1.21.6.tar.gz
    dest: /tmp/

- name:  tar -xf  nginx
  shell: tar -xf /tmp/nginx-1.21.6.tar.gz -C /tmp && chmod a+x /etc/rc.d/rc.local

- name: change dir nginx && install nginx
  shell: ./configure --prefix=/usr/local/nginx && make && make install && /usr/local/nginx/sbin/nginx
  args:
    chdir: /tmp/nginx-1.21.6


- name: enabled nginx
  lineinfile:
    path: /etc/rc.d/rc.local
    line: /usr/local/nginx/sbin/nginx

- name: clean /tmp/nginx
  shell: rm -rf /tmp/ngin*
```

```bash
[root@zabbix-server roles]# cat /opt/install_nginx.yaml 
- hosts: dbserver
  user: root
  roles:
    - nginx

```

### 3、ansible部署wordpress（LAMP）

```bash
[root@zabbix-server roles]# cat wp_lnmp/tasks/main.yml 
---
# tasks file for wp_lnmp
- name: install httpd php mysql
  yum:
    name={{ item }}
    state=present
  loop:
    - httpd
    - mariadb-server
    - php
    - php-devel
    - php-mysql
    - gd
    - php-gd

- name: change httpd.conf
  shell: sed -ri '164 s|DirectoryIndex index.html|DirectoryIndex index.php index.html|' /etc/httpd/conf/httpd.conf && rm -rf /etc/httpd/conf.d/welcome.conf

- name: change mysql
  shell: mysql -uroot -e "CREATE DATABASE wp01 CHARSET utf8" && mysql -uroot -e "GRANT ALL ON wp.* TO 'wpuser'@'localhost' IDENTIFIED BY 'redhat'" && mysql -uroot -e "FLUSH PRIVILEGES"
  
- name: start httpd
  service:
    name: httpd
    state: started

- name: start mariadb-server
  service:
    name: mariadb
    state: started

- name: push wordpress
  copy:
    src: wordpress-4.6.10.tar.gz
    dest: /root/

- name: tar -xf wordpress
  shell: tar -xf /root/wordpress-4.6.10.tar.gz -C ./ && cp -r /root/wordpress/* /var/www/html/
  
- name: restart http
  shell: systemctl restart httpd mariadb 
```

```bash
[root@zabbix-server roles]# cat /opt/install_wp_.yaml 
- hosts: dbserver
  user: root
  roles:
     - wp_lnmp

```

