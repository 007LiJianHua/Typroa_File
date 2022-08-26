[toc]

## 一、简介

  [Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020) 同 Apache 等其他 Web 服务器的配置记法不太相同，Nginx的配置文件使用语法的就是一门微型的编程语言。可以类似写程序一般编写配置文件，可操作性很大。既然是编程语言，一般也就少不了“变量”这种概念。
  所有的 Nginx变量在 Nginx 配置文件中引用时都须带上 `$ `前缀。在 Nginx 配置中，变量只能存放一种类型的值，有且也只存在一种类型，那就是[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)类型。nginx可以使用变量简化配置与提高配置的灵活性，所有的变量值都可以通过这种方式引用。

## 二、定义与使用

### 1.声明变量

  可以在sever,http,location等标签中使用set命令（非唯一）声明变量。
语法：`set $变量名 变量值`

注意:

> nginx 中的变量必须都以$开头
> nginx 的配置文件中所有使用的变量都必须是声明过的，**否则 nginx 会无 法启动并打印相关异常日志**

### 2.可见性

  在不同层级的标签中声明的变量性的可见性规则如下:

> * location标签中声明的变量中对这个location块可见
> * server标签中声明的变量对server块以及server块中的所有子块可见
> * http标签中声明的变量对http块以及http块中的所有子块可见

### 3.配置

```shell
[root@cheng ~]# cd /etc/nginx/conf.d/
[root@cheng conf.d]# vim echo.conf
server {
        listen 80;
        server_name     localhost;
        location /test {
                set $foo hello;
                echo "foo: $foo";
        }
}
[root@cheng conf.d]# nginx -s reload
[root@cheng conf.d]# curl localhost/test
foo: hello
```

### 4.大括号

  在“变量插值”的上下文中，还有一种特殊情况，即当引用的变量名之后紧跟着变量名的构成字符时（比如后跟字母、数字以及下划线），我们就需要使用特别的记法来消除歧义。

```shell
[root@cheng ~]# cd /etc/nginx/conf.d/
[root@cheng conf.d]# vim echo.conf
server {
        listen 80;
        server_name     localhost;
        location /test-brace {
                set $first "hello ";
                echo "${first}world";	
        }
}
[root@cheng conf.d]# nginx -s reload 
[root@cheng conf.d]# curl localhost/test-brace
hello world
```

### 5.作用域

  set 指令不仅有赋值的功能，它还有创建 `Nginx 变量`的作用，即当作为赋值对象的变量尚不存在时，它会自动创建该变量，但为空。如果我们不创建就直接使用它的值，则会报错。
  Nginx 变量一旦创建，其变量名的可见范围就是整个 Nginx 配置，甚至可以跨越不同虚拟主机的 server 配置块。Nginx 变量名的可见范围虽然是整个配置，但每个请求都有所有变量的独立副本，或者说都有各变量用来存放值的容器的独立副本，彼此互不干扰。

## 三、内置变量

| 变量名              | 定义                                                         |
| ------------------- | ------------------------------------------------------------ |
| $arg_PARAMETER      | GET请求中变量名PARAMETER参数的值                             |
| $args               | **这个变量等于GET请求中的参数。例如，foo=123&bar=blahblah;这个变量只可以被修改** |
| $binary_remote_addr | 二进制码形式的客户端地址。                                   |
| $body_bytes_sent    | 传送页面的字节数                                             |
| $content_length     | 请求头中的Content-length字段。                               |
| $content_type       | 请求头中的Content-Type字段。                                 |
| $cookie_COOKIE      | cookie COOKIE的值。                                          |
| $document_root      | 当前请求在root指令中指定的值。                               |
| $document_uri       | 与$uri相同。                                                 |
| $host               | 请求中的主机头(Host)字段，如果请求中的主机头不可用或者空，则为处理请求的server名称(处理请求的server的server_name指令的值)。值为小写，不包含端口。 |
| $hostname           | 机器名使用 gethostname系统调用的值                           |
| $http_HEADER        | HTTP请求头中的内容，HEADER为HTTP请求中的内容转为小写，-变为_(破折号变为下划线)，例如：$http_user_agent(Uaer-Agent的值); |
| $sent_http_HEADER   | HTTP响应头中的内容，HEADER为HTTP响应中的内容转为小写，-变为_(破折号变为下划线)，例如： $sent_http_cache_control, $sent_http_content_type…; |
| $is_args            | 如果$args设置，值为"?"，否则为""。                           |
| $limit_rate         | 这个变量可以限制连接速率。                                   |
| $nginx_version      | 当前运行的nginx版本号。                                      |
| $query_string       | 与$args相同。                                                |
| $remote_addr        | 客户端的IP地址。                                             |
| $remote_port        | 客户端的端口。                                               |
| $remote_user        | 已经经过Auth Basic Module验证的用户名。                      |
| $request_filename   | **当前连接请求的文件路径，由root或alias指令与URI请求生成。** |
| $request_body       | 这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义。 |
| $request_body_file  | 客户端请求主体信息的临时文件名。                             |
| $request_completion | 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空。 |
| $request_method     | 这个变量是客户端请求的动作，通常为GET或POST。包括0.8.20及之前的版本中，这个变量总为main request中的动作，如果当前请求是一个子请求，并不使用这个当前请求的动作。 |
| $request_uri        | 这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI。 |
| $scheme             | 所用的协议，比如http或者是https，比如rewrite ^(.+)$ $scheme://example.com$1 redirect; |
| $server_addr        | 服务器地址，在完成一次系统调用后可以确定这个值，如果要绕开系统调用，则必须在listen中指定地址并且使用bind参数。 |
| $server_name        | 服务器名称。                                                 |
| $server_port        | 请求到达服务器的端口号。                                     |
| $server_protocol    | 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。                   |
| $uri                | 请求中的当前URI(不带请求参数，参数位于args，不同于浏览器传递的args)，不同于浏览器传递的request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。不包括协议和主机名，例如/foo/bar.html |

## 四、uri 和 request_uri

   由 ngx_http_core 模块提供的内建变量 uri，可以用来获取当前请求的 URI（不含请求参数）， 而 request_uri 则用来获取请求最原始的 URI（包含请求参数）。

示例（无echo模块需先加载）

```shell
[root@cheng ~]# cd /etc/nginx/conf.d/
[root@cheng conf.d]# vim echo.conf
server {
        listen 80;
        server_name     localhost;
        location /test-uri {
                echo "uri = $uri";
                echo "request_uri = $request_uri";
        }
}
[root@cheng conf.d]# nginx -s reload 
[root@cheng conf.d]# curl localhost/test-uri
uri = /test-uri
request_uri = /test-uri
[root@cheng conf.d]# curl "localhost/test-uri?a=3&b=4"
uri = /test-uri
request_uri = /test-uri?a=3&b=4
[root@cheng conf.d]# curl "localhost/test-uri/hello%20world?a=3&b=4"
uri = /test-uri/hello world
request_uri = /test-uri/hello%20world?a=3&b=4
```

## 五、$arg_XXX

  另一个特别常用的内建变量其实并不是单独一个变量，而是有无限多变种的一群变量，即名字以 arg_ 开头的所有变量，我们估且称之为 arg_XXX 变量群。

```shell
[root@cheng ~]# cd /etc/nginx/conf.d/
[root@cheng conf.d]# vim echo.conf
server {
        listen 80;
        server_name     localhost;
        location /test-arg {
        echo "name: $arg_name";
        echo "class: $arg_class";
        }
}
[root@cheng conf.d]# nginx -s reload
[root@cheng conf.d]# curl localhost/test-arg
name: 
class: 
[root@cheng conf.d]# curl "localhost/test-arg?name=Tom&class=3"
name: Tom
class: 3
[root@cheng conf.d]# curl "localhost/test-arg?name=hello%20world&class=9"
name: hello%20world
class: 9
[root@cheng conf.d]# curl "localhost/test-arg?NAME=Marry"		//$arg不区分大小写
name: Marry
class: 
```