[toc]

## 一、pip介绍

### 1、pip是什么？

> 什么是 pip ？pip 是 Python 中的标准库管理器。它允许你**安装**和**管理**不属于 Python标准库 的其它软件包。

### 2、怎么操作pip？

1）window电脑点击`win键+ R`，输入：`cmd`
![在这里插入图片描述](https://img-blog.csdnimg.cn/1b2ce81ab29e439e8954c293fb4387e4.png)

2）输入对应的pip命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ca14a699a1a4451ac3ad682d84eddc3.png)

## 二、pip常用命令

### 1、查看pip所在路径

```python
where pip
```

### 2、查看pip版本

```python
pip -V
pip --version
```

### 3、pip升级命令

```python
python -m pip install --upgrade pip
```

如果pip 默认源的网络连接较差，临时使用本镜像站来升级：

```python
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

### 4、安装包

**第一种**：

```xml
pip install package_name  #这里的package_name是要安装的第三方库名
1
```

**第二种**（推荐使用，下载更快）：使用国内镜像源

```xml
pip install package_name -i http://mirrors.aliyun.com/pypi/simple/ 
1
```

**国内镜像源**

- 阿里云：http://mirrors.aliyun.com/pypi/simple/ （推荐使用）
- 清华大学：https://pypi.tuna.tsinghua.edu.cn/simple
- 豆瓣：http://pypi.douban.com/simple/

**第三种**（指定包版本号）：

```python
pip install package_name=版本号 
```

**第四种**（本地文件安装）：

- 下载包：https://pypi.org/

- 搜索包名
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/4def2cf134e547e198db78f964aac23f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5aSn5pWw5o2uX-Wwj-iigQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

- 找到对应版本
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/70082f185e684515aa670872c4b75b3c.png)

- 点击下载
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/9fa2eca0c2604952999be4dd393f296f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5aSn5pWw5o2uX-Wwj-iigQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

- 下载对应whl文件（我python版本是3.8，电脑window64位）
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/c22fa4d26c774df1ba36810cd8ac5bbc.png)

- 执行安装命令

  ```python
  pip install C:\Users\Administrator\Downloads\pymssql-2.2.2-cp38-cp38-win_amd64.whl #后面是包所在路径和包名
  ```

- 安装成功：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/d654a28789e64842bedd30a001a9e482.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5aSn5pWw5o2uX-Wwj-iigQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 5、全局设置镜像源地址

> 每次安装都加国内镜像地址太麻烦了，可以设置固定配置

```python
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 6、卸载包

```python
pip uninstall package_name #这里的package_name是要卸载的第三方库名
```

### 7、搜索包

```python
pip search package_name
```

### 8、查看所有安装的包

```python
pip list
```

### 9、查看安装包的详细信息

```python
pip show package_name
```

### 10、更新指定的包

```python
pip install --upgrade package_name
```

### 11、查看需要更新的包

```python
pip list --outdated
```

### 12、查看帮助

```python
pip -h
pip --help

Usage:  
 pip<command>[options]
 
Commands:
 install                    安装包.
 uninstall                  卸载包.
 freeze                     按着一定格式输出已安装包列表
 list                       列出已安装包.
 show                       显示包详细信息.
 search                     搜索包，类似yum里的search.
 wheel                      Buildwheelsfromyourrequirements.
 zip                        不推荐.Zipindividualpackages.
 unzip                      不推荐.Unzipindividualpackages.
 bundle                     不推荐.Createpybundles.
 help                       当前帮助.
 
GeneralOptions:
 -h,--help                 显示帮助.
 -v,--verbose              更多的输出，最多可以使用3次
 -V,--version              现实版本信息然后退出.
 -q,--quiet                最少的输出.
 --log-file<path>          覆盖的方式记录verbose错误日志，默认文件：/root/.pip/pip.log
 --log<path>               不覆盖记录verbose输出的日志.
 --proxy<proxy>            Specifyaproxyintheform[user:passwd@]proxy.server:port.
 --timeout<sec>            连接超时时间(默认15秒).
 --exists-action<action>   Defaultactionwhenapathalreadyexists:(s)witch,(i)gnore,(w)ipe,(b)ackup.
 --cert<path>              证书.
123456789101112131415161718192021222324252627282930
```

## 

## 三、报错问题

权限问题报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb5aa47f9dcc46f3a80c44b5af90d406.png)
加上 `--trusted-host mirrors.aliyun.com`后缀

```xml
pip install pymssql -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/addb0eb152b14dd3b3b4c14417cb9a12.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5aSn5pWw5o2uX-Wwj-iigQ==,size_20,color_FFFFFF,t_70,g_se,x_16)