[toc]

## 一、url重写介绍

### 1、语法

```bash
rewrite    uri地址    新的uri地址  [标志];
```

> 注意事项
>
> * server、location、if
> * rewrite 可以存在多条，依次进行处理
> * **旧uri地址支持正则表达式; 新uri支持反向引用**
> * 旧uri地址匹配客户端时，不包括请求中的参数 
> * 支持变量的使用

### 2、标志

* last
  * 终止本location块中的匹配，将新地址转交给下一个location
* break

- - 不会将新地址交给其他的location处理，只在本location中处理 

* redirect
  * 临时重定向，状态码302
* permanet
  * 永久重定向，状态码301

## 二、URL重写案例

##### 案例1：改写地址中的目录名称

```bash
rewrite ^/mp3 http://python.linux.com/music;
```

##### 案例2：域名跳转

```bash
rewrite ^/  https://www.baidu.com;
```

##### 案例3：实现https的自动跳转

```bash
   location / {
      root /data/python;
      index index.html;
      if ($host = www.linux.com) {
          rewrite ^/(.*)  https://www.linux.com/$1;
      }
   }
```

##  三、location的写法

### 1、写法

> * location用来匹配客户端的写法
>
> ```bash
> location [ = | ~ | ~* | ^~ ] uri {
>     ..........
> }
> ```

```bash
location  = / {
  # 精确匹配 / ，主机名后面不能带任何字符串
  [ configuration A ] 
}
location  / {
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  [ configuration B ] 
}
location /documents/ {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时 ，这一条才会采用这一条
  [ configuration C ] 
}
location ~ /documents/Abc {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration CC ] 
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ] 
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ] 
}
location /images/ {
  # 字符匹配到 /images/，继续往下，会发现 ^~ 存在
  [ configuration F ] 
}
location /images/abc {
  # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在
  # F与G的放置顺序是没有关系的
  [ configuration G ] 
}
location ~ /images/abc/ {
  # 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用
    [ configuration H ] 
}
location ~* /js/.*/\.js
```

### 2、优先级

> (location =) > (location 完整路径) > (location ^~ 路径) > (location ~* 正则顺序) > (location 部分起始路径) > (/)

* 定义错误页面

```bash
server {
        listen 192.168.152.128:80;
        server_name python.linux.com;
        error_log       /usr/local/nginx/logs/python_error.log  error;
        access_log      /usr/local/nginx/logs/python_access.log main;

        location / {
                root /python;
                index index.html;
        }
        error_page 404 /sorry.html;
        location = /sorry.html {
                root /python;
        }
}
```

