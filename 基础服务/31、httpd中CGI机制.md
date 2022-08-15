# CGI机制

[toc]

## 一、CGI机制介绍

> * Common Gateway Interface 通用网关接口
> * 作用
>   * 让web进程通过CGI机制调用其他的应用程序，解析动态网站 

### 1、流程图

![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/43D18286260647A9A152B27CC672CFBA/863766CA46024B588FE6709CE192D30D/20205](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/43D18286260647A9A152B27CC672CFBA/863766CA46024B588FE6709CE192D30D/20205)

## 二、具体应用

### 1、测试PHP页面

* 在一个虚拟主机下创建test1.php
* 浏览器访问是看不到phpinfo()界面的

```bash
[root@localhost ~]# cat /mp3/test1.php 
AAAAAAAAAAAAA
<?php
   phpinfo();
?>
```

### 2、安装PHP软件

```bash
# yum install -y php
# systemctl restart httpd 
```

* PHP实际上是作为httpd的功能模块存在着的

```bash
[root@localhost ~]# ls /etc/httpd/modules/
libphp5.so 
```

* 再次在浏览器上测试，

![https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa44a90b69599e96f6edf3f03c67ad5/82](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE2aa44a90b69599e96f6edf3f03c67ad5/82)