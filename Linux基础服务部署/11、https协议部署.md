# 基于https协议部署虚拟主机

[toc]

## 一、网站传输协议

* http
  * 80/tcp
  * 数据以明文的方式传输
* https
  * 443/tcp	
  * 数据以加密的方式传输

## 二、安全性保障

* 数据安全性-------------对称加密算法
  * 加密、解密、
* 数据完整性-------------hash算法
  * 校验算法、MD5/SHA   hash哈希算法
* 身份的真实性-----------签发证书
  * 客户端访问百度服务器时，确保访问的确实是百度，而不是恶意的钓鱼网站

## 三、数据安全性的保障

> 通过加密算法确保数据的安全
>
> 发送方发送数据前要加密、接收方解密

### 1、对称加密算法

* 加密数据时，使用的密钥、解密的密钥是一样的

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/F28D456AF4BE44C1A2A83F27BA1B2FCD/20147)

* 典型的对称加密算法，安全性从低到高排序，
  * DES	
  * 3DES
  * AES

**加密数据**

* opensll enc：加密命令
* -e：给数据加密
* -d：给数据解密
* -des：加密算法
* -in：给哪个数据进行加密
* -out：加密后的数据另存为

```bash
[root@localhost ~]# openssl enc -e -des -in /opt/file01 -out /opt/file01_s
enter des-cbc encryption password:
Verifying - enter des-cbc encryption password:

```

**解密数据**

```bash
#  openssl enc -d -des -in /opt/file01_s -out /opt/file01 
enter des-cbc decryption password:
```

### 2、非对称加密算法

加密解密数据时用的算法是不一样的

使用密钥对{公钥、私钥}

* 公钥加密
* 私钥解密

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/7CDEA5F5DFA54F5198525F0A55CACE1A/20150](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/7CDEA5F5DFA54F5198525F0A55CACE1A/20150)

* 典型的非对称加密算法
  * 公钥对和私钥对用什么数学算法生成
  * RSA：**常用，安全性高**，数学算法较复杂，但加密时间长
  * DSA：安全性低，但加密时间快

### **3、实际使用**

* **使用对称算法加密真实数据，使用非对称加密算法加密对称算法的密钥**

## 四、数据完整性保证

### 1、校验算法

* MD5
  * 只要数据内容本身没有改变，不管是第几个校验码都是一样的
  * 校验码本身就是数据的加密
* SHA
* 这两个算法统称为hash（哈希）算法

> * 发送数据时，使用校验算法对数据进行校验，生成校验码，同时将数据+校验码发送出去
>
> * 接收方接收数据，使用相同的校验算法再次进行校验，对比校验码

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/7BE9FF919B8B47628BF420DE38B890E4/20152](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/7BE9FF919B8B47628BF420DE38B890E4/20152)

## 五、身份的真实性

### 1、密钥对

* 数据加密（客户端与服务器之间）
  * 公钥加密、私钥解密
* 验证签名（CA证书颁发机构）
  * 私钥签名、公钥验证签名

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/73EB8CE994354781A52E546548365EB3/20148](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/73EB8CE994354781A52E546548365EB3/20148)

> 流程：
>
> * 电商服务器生成证书申请【.csr文件】， 同时将电商服务器的公钥放入证书申请；将证书申请发送给CA
> * CA审批信息, 通过后CA会使用自己的私钥进行签名；相当于签署证书【.crt】
> * 客户端访问电商服务器时，服务器会将证书信息发送给客户端 
> * 客户端通过CA的公钥验证证书，验证通过后可获取电商服务器的公钥
> * 客户端选取对称算法、生成对称算法密钥对，使用非对称加密公钥加密，发送给服务器、服务器解密 
> * 利用对称加密算法进行真实数据交互

## 六、配置基于https协议的虚拟主机

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/D904CBFE8E7347ED9D945AAA3A455D3B/20151](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/D904CBFE8E7347ED9D945AAA3A455D3B/20151)

### 1、配置私有CA

* 生成密钥对

  * 先创建CA服务器需要的数据库文件

  ```bash
  [root@ca ~]# touch /etc/pki/CA/index.txt
  [root@ca ~]# echo 01 > /etc/pki/CA/serial
  ```

  

  * 创建CA服务器需要的密钥

  ```bash
  [root@ca ~]# openssl genrsa -out /etc/pki/CA/private/cakey.pem 1024
  Generating RSA private key, 1024 bit long modulus
  ..++++++
  .....++++++
  e is 65537 (0x10001)
  ```

* 生成自签证书

```bash
[root@ca ~]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
```

### 2、在Web-server上申请证书

* 在web-server创建密钥

> 安全套接字（Secure [Socket](https://so.csdn.net/so/search?q=Socket&spm=1001.2101.3001.7020) Layer，SSL）协议是Web浏览器与Web服务器之间安全交换信息的协议，提供两个基本的安全服务：鉴别与保密。

```bash
[root@web_server ~]# mkdir /etc/httpd/ssl
[root@web_server ~]# openssl genrsa -out /etc/httpd/ssl/www.linux.com.key 2048
```

* 创建证书申请
  * 注意：申请证书的国家地区省份需要与CA对应一致，否则不能颁发证书

```bash
[root@localhost ~]# openssl req -new -key /etc/httpd/ssl/www.linux.com.key -out /etc/httpd/ssl/www.linux.com.csr 
```

* 将证书申请发送给CA

```bash
[root@web_server ~]# rsync -av /usr/local/nginx/ssl/www.linux.com.csr root@192.168.140.11:/tmp/
```

### 3、CA签署证书

```bash
[root@ca ~]# openssl ca -in /tmp/www.linux.com.csr -out /etc/pki/tls/certs/www.linux.com.crt -days 3650 
```

* 查看生成的证书

```bash
[root@ca ~]# ls /etc/pki/tls/certs/
ca-bundle.crt  ca-bundle.trust.crt  make-dummy-cert  Makefile  renew-dummy-cert  www.linux.com.crt
```

* 查看签发证书数量

```bash
[root@ca ~]# cat /etc/pki/CA/serial
02
```

* 查看签发证书数据库信息

```bash
[root@ca ~]# cat /etc/pki/CA/index.txt
V	310619060654Z		01	unknown	/C=cn/ST=bj/O=bj/OU=bj/CN=www.linux.com/emailAddress=bj@qq.com
```

### 4、将证书发送给web-server

```bash
[root@ca ~]# rsync -av /etc/pki/tls/certs/www.linux.com.crt root@192.168.140.10:/etc/httpd/ssl
```

### 5、在web-server安装mod_ssl模块

```bash
[root@localhost ~]# yum install -y mod_ssl 
```

### 6、创建网页目录，测试文件

```bash
[root@localhost ~]# mkdir /linux
[root@localhost ~]# vim /linux/index.html
<h1> www.linux.com </h1>
```

### 7、配置ssl虚拟主机

* **在配置多个加密虚拟主机时，只能保留一个全局配置，剩下直接修改虚拟主机部分**

```bash
[root@localhost ~]# vim /etc/httpd/conf.d/ssl.conf 
....................
<VirtualHost _default_:443>

# General setup for the virtual host, inherited from global configuration
DocumentRoot "/linux"
ServerName www.linux.com:443
SSLCertificateFile /etc/httpd/ssl/www.linux.com.crt
SSLCertificateKeyFile /etc/httpd/ssl/www.linux.com.key

<Directory "/linux">
    Require all granted
</Directory>
[root@localhost ~]# httpd -t
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
Syntax OK
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# netstat -antp | grep http
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      7144/httpd          
tcp6       0      0 :::443                  :::*                    LISTEN      7144/httpd          
```



### 8、测试访问

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/2305CA329DAD4CEE8EA47C5290B15501/20149](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2BBAC98EC91D403687C29E7F07F24AC6/2305CA329DAD4CEE8EA47C5290B15501/20149)

## 七、配置https自动跳转

> * 浏览器默认访问服务器用http协议
> * 跳转到加密主机需要修改配置文件
> * 具体修改哪个配置文件就要看单独写主机名的时候，默认跳转到了哪个网页，跳转到哪个主机，就修改哪个主机的配置文件

### 1、增加三行配置

* 开启重定向功能

```bash
RewriteEngine On
```

* %{HTTP_HOST}   
  * httpd中的变量,表示的是客户端访问服务器时候写的是什么主机名
* [NC]		
  * 忽略大小写；

```bash
RewriteCond %{HTTP_HOST} www.linux.com [NC]
```

*  ^/	
  * 写的主机名最右边是以/开头的，进行跳转
* [L]	
  * 立即

```bash
RewriteRule ^/   https://www.linux.com [L]
```

