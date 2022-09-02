[toc]

## 一、安装第三方模块

> pymysql模块模块说明：是一个Python编写的MySQL驱动程序，让我们可以用Python语言操作MySQL数据库。

**1. window电脑点击`win键+ R`，输入：`cmd`**
![在这里插入图片描述](https://img-blog.csdnimg.cn/1b2ce81ab29e439e8954c293fb4387e4.png)

**2. 安装`pymysql`，输入对应的pip命令**：`pip install pymysql`，我已经安装过了出现版本就安装成功了

![在这里插入图片描述](https://img-blog.csdnimg.cn/a796c4d8a7d84c499ed3591c62ba1d25.png)
**3. 安装`pymssql`，输入对应的pip命令**：`pip install pymssql`，我已经安装过了出现版本就安装成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ccc88e66e1f49a5b21ddea793b28429.png)

## 二、pymysql模块使用说明

### 1. 操作流程流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c01706339404429b6b1172e427ed4c4.png)

### 2. 导入pymysql模块

```csharp
import pymysql
```

### 3. 创建连接对象

调用pymysqI模块中的`connect()`函数来创建连接对象,代码如下:

```csharp
conn = pymysql.connect(参数列表)
```

**参数说明**：

- `host`：连接的mysql主机，如果本机是’localhost’
- `port`：连接的mysql主机的端口，默认是3306
- `database`：数据库的名称
- `user`：连接的用户名
- `password`：连接的密码
- `charset`：通信采用的编码方式，推荐使用utf8

**连接对象操作说明**：

- `conn.close()`：关闭连接
- `conn.commit()`：提交数据
- `conn.rollback()`：撤销数据

### 4. 获取游标对象

获取游标对象的目标就是要执行sql语句，完成对数据库的增、删、改、查操作的代码如下:

```csharp
cur = conn.cursor()
```

**游标操作说明：**

- `execute(operation [parameters])`：使用游标执行SQL语句，返回受影响的行数，主要用于执行insert、update、delete、select等语句，也可以执行create、alter、drop等语句
- `cur.fetchone()`：获取查询结果集中的一条数据，返回一个元组，如`(1,张三)`
- `curfetchall()`：获取查询结果集中的所有数据，返回一个元组，如`((1,张三),(2,'李四"))`
- `cur.close()`：关闭游标，表示和数据库操作完成

### 5. 获取一条数据

> 使用`cur.fetchone()`：获取查询结果集中的一条数据，返回一个元组，如`(1,张三)`

```csharp
# 1. 导入pymysql模块
import pymysql

# 2. 创建连接对象
# host：连接的mysql主机，如果本机是’localhost’
# port：连接的mysql主机的端口，默认是3306
# database：数据库的名称
# user：连接的用户名
# password：连接的密码
# charset：通信采用的编码方式，推荐使用utf8
conn = pymysql.connect(host="localhost",
                       port=3306,
                       user="root",
                       password="mysql",
                       database="python41",
                       charset="utf8"
                       )

# 3.获取游标，目的就是要执行sql语句
cursor = conn.cursor()

# 准备sql，之前在mysql客户端如何编 写sql，在python程序里面还怎么编写
sql = "select * from students;"

# 4. 执行sql语句
cursor.execute(sql)

# 获取查询结果，返回的数据类型是一个元组：(1,张三)
row = cursor.fetchone()
print(row)

# 5.关闭游标
cursor.close()

# 6.关闭连接
conn.close()
```

### 6. 获取多条数据

> 使用`curfetchall()`：获取查询结果集中的所有数据，返回一个元组，如`((1,张三),(2,'李四"))`

```csharp
# 1. 导入pymysql模块
import pymysql

# 2. 创建连接对象
# host：连接的mysql主机，如果本机是’localhost’
# port：连接的mysql主机的端口，默认是3306
# database：数据库的名称
# user：连接的用户名
# password：连接的密码
# charset：通信采用的编码方式，推荐使用utf8
conn = pymysql.connect(host="localhost",
                       port=3306,
                       user="root",
                       password="mysql",
                       database="python41",
                       charset="utf8"
                       )

# 3.获取游标，目的就是要执行sql语句
cursor = conn.cursor()

# 准备sql，之前在mysql客户端如何编 写sql，在python程序里面还怎么编写
sql = "select * from students;"

# 4. 执行sql语句
cursor.execute(sql)

# 获取查询结果，返回的数据类型是一个元组：((1,'张三'),(2,'李四"))
rows = cursor.fetchall()
for row in rows:
    print(row)
    # (1, '张三')
    # (2, '李四")

# 5.关闭游标
cursor.close()

# 6.关闭连接
conn.close()
```

### 7. 对数据增删改操作

> 使用`conn.commit()`：提交数据；`conn.rollback()`：撤销数据对数据库进行增删改操作

```csharp
# 1. 导入pymysql模块
import pymysql

# 2. 创建连接对象
# host：连接的mysql主机，如果本机是’localhost’
# port：连接的mysql主机的端口，默认是3306
# database：数据库的名称
# user：连接的用户名
# password：连接的密码
# charset：通信采用的编码方式，推荐使用utf8
conn = pymysql.connect(host="localhost",
                       port=3306,
                       user="root",
                       password="mysql",
                       database="python41",
                       charset="utf8"
                       )

# 3.获取游标，目的就是要执行sql语句
cursor = conn.cursor()

# 增加操作
sql = "insert into classes(name) values('小明)"
# 修改操作
# sql = "update classes set name = '小红' where id = 2"
# 删除操作
# sql = "delete from classes where id=2"

try:
    # 4. 执行sql语句
    cursor.execute(sql)
    # 增删改都必须提交数据
    conn.commit()
except:
    # 如果报错就对修改的数据进行撤销，表示数据回滚
    conn.rollback()

# 5.关闭游标
cursor.close()

# 6.关闭连接
conn.close()
```

## 三、pymssql模块使用说明

**`pymssql`模块 与 `pymyql`模块的区别在于：**

- 1、导包：`import pymssql`
- 2、创建连接对象：`conn = pymssql.connect(参数列表)`

**其他语法与pymyql一模一样就不过多讲解了！！！**