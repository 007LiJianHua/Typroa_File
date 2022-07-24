# HTTP协议

[toc]

## 一、http协议

* 应用层协议
* 构建网站服务
  * **在客户端和服务器之间传递数据**
* 作用
  * 在客户端，web服务器传送数据
* Hyper Text Transfer Protocol   超文本传输协议
  * 客户端访问web服务器是，传递代码，由浏览器解析代码进行展示

## 二、web服务

### 1、类型

* 静态网站
  * 网页上所有的资源都是固定的
  * HTML语言，js、jquery
    * *.html
* 动态网站
  * 一段程序代码、根据传递的参数不同展示不同的结果
    * PHP语言：*.php文件
    * Java语言：*.jsp文件
* 目前大部分服务器都是动静结合，都能处理

### 2、cookie机制、session

* 作用
  * 识别客户端登身份
* 区别
  * cookie是将随机数存放到自己身上
  * session是将记录放到服务器

**![https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2D55E1987CE047208AEF5F26F480F896/2B81CC19FC9E4EF68313E8A3E26A934C/19875](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2D55E1987CE047208AEF5F26F480F896/2B81CC19FC9E4EF68313E8A3E26A934C/19875)**

![img](https://note.youdao.com/yws/public/resource/09cca8ab7a03af721408094d133dc002/xmlnote/2D55E1987CE047208AEF5F26F480F896/5FDB8769D23C4005B16B8A82A7FB6F49/19877)

## 三、HTTP协议特性

### 1、http/0.9

* 只支持传输纯文本数据

### 2、http/1.0

* 引用了MIME机制
  * 传输非文本数据（图片、视频、音频、动画）
* 缓存机制
  * 服务器缓存
  * 客户端缓存

### 3、http/1.1

* 长连接机制   keepalive    串行
  * 允许在一条连接上发送多次请求
    * 限制长连接的超时时间
    * 限制每个长连接的最大请求数    453532
* 管道机制
  * 允许一次性发送多个请求，并行，但是服务器返回的时候还是串行返回
* 加强了缓存的管理
  * 缓存静态数据
  * 缓存过期时间

### 4、http/2

* 以并行的方式发送请求、响应

## 四、HTTP状态码、请求方法

### 1、状态码

* 200
  * 成功响应
* 301、302、304
  * 成功响应
  * 重定向
* 4xx
  * 错误相应
    * 403			权限拒绝
    * 404           文件找不到
* 5xx
  * 错误相应
    * 服务端配置

### 2、请求方法

GET
用于获取内容、数据
POST
上传数据
DELETE