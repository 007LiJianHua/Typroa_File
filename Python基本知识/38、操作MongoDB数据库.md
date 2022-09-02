[toc]

## 一、安装pymongo库

**如果MongoDB不会的，博主为你们准备了基础教程：**[MongoDB数据库入门到精通看这一篇就够了](https://blog.csdn.net/yuan2019035055/article/details/123031732)

cmd 控制台中输入：

```sql
pip install pymongo
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/69513cda3d1f40878cf1f1bc140be3a4.png)

## 二、数据库操作

### 1、连接数据库

创建数据库连接需要使用 MongoClient 对象，并且指定连接的 IP 地址、端口号，创建的数据库名

```python
from pymongo import MongoClient


def mongodb_init01():
    """数据库连接方式1"""
    client = MongoClient(host='127.0.0.1', port=27017)
    print(client)


def mongodb_init02():
    """数据库连接方式2"""
    uri = "mongodb://{}:{}".format('127.0.0.1', 27017)
    client = MongoClient(uri)
    print(client)


if __name__ == '__main__':
    mongodb_init01()
    mongodb_init02()
```

输出结果：

```python
MongoClient(host=['127.0.0.1:27017'], document_class=dict, tz_aware=False, connect=True)
MongoClient(host=['127.0.0.1:27017'], document_class=dict, tz_aware=False, connect=True)/
```

### 2、数据库操作

* 数据库的相关操作有：
  * 查看所有数据库名称、
  * 判断数据库是否存在、
  * 使用现有数据库，
  * 创建不存在的数据库，
  * 删除数据库

```python
# 返回当前所有数据库名称
databases_list = client.list_database_names()

# 判断数据库是否存在
if 'test001' in database_names:
    print("数据库已存在")
else:
    print("数据库不存在")
    
# 使用现有数据库（数据库名：test_database）
test_database = client.test_database

# 创建不存在的数据库并使用
new_database = client.new_database

# 删除现有数据库
client.drop_database('new_database')
```

注意：

- database_names 在最新版本的 Python 中已废弃，Python3.7+ 之后的版本改为了 `list_database_names()`。
- 在 MongoDB 中，数据库只有在内容插入后才会创建。就是说，数据库创建后要创建集合(数据表)并插入一个文档(记录)，数据库才会真正创建。

### 3、[集合](https://so.csdn.net/so/search?q=集合&spm=1001.2101.3001.7020)操作

集合的相关操作有：查看所有集合名称、判断集合是否存在、使用现有集合，创建不存在的集合，删除集合

```python
# 查看当前数据库中的所有集合
collection_names = new_database.list_collection_names()

# 判断一个集合是否存在
if 'test001' in collection_names:
    print("集合已存在")
else:
    print("集合不存在")

# 获取指定数据库下的所有集合对象
collections = test_database.list_collections()
for collection in collections:
    print('获取集合:', collection)
      
# 使用一个已存在数集合（集合名称：test_collection）
test_collection = new_database.test_collection

# 创建一个新的集合
new_collection = new_database.new_collection

# 删除一个集合
new_database.drop('new_collection')
```

注意：

- collection_names 在最新版本的 Python 中已废弃，Python3.7+ 之后的版本改为了 `list_collection_names()`。
- 在 MongoDB 中，集合只有在内容插入后才会创建。就是说，集合创建后要插入一个数据，集合才会真正创建。

### 4、插入文档

[MongoDB](https://so.csdn.net/so/search?q=MongoDB&spm=1001.2101.3001.7020) 中的一个文档类似 SQL 表中的一条记录



集合中插入文档使用 `insert_one()` 方法，该方法的第一参数是字典 name => value 对

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

# 使用insert_one插入一条数据
x = new_collection.insert_one({"name": "张三", "age": 18})

print(x)  # 输出结果：<pymongo.results.InsertOneResult object at 0x000002851E6B0F80>
```

打开可视化工具查看结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9e0944d167d4db59aa02ec411f16e86.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 插入多个文档

集合中插入多个文档使用 `insert_many()` 方法，该方法的第一参数是字典列表。

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

mylist = [
    {"name": "张飞", "hometown": "蜀国", "age": 30, "sex": "男"},
    {"name": "关羽", "hometown": "蜀国", "age": 40, "sex": "男"},
    {"name": "刘备", "hometown": "蜀国", "age": 50, "sex": "男"},
    {"name": "曹操", "hometown": "魏国", "age": 45, "sex": "男"},
    {"name": "司马懿", "hometown": "魏国", "age": 45, "sex": "男"},
    {"name": "孙权", "hometown": "吴国", "age": 50, "sex": "男"},
    {"name": "貂蝉", "hometown": "未知", "age": 18, "sex": "女"},
    {"name": "西施", "hometown": "越国", "age": 18, "sex": "女"},
    {"name": "王昭君", "hometown": "西汉", "age": 18, "sex": "女"},
    {"name": "杨玉环", "hometown": "唐朝", "age": 18, "sex": "女"}
]

# 使用insert_many插入多条数据
x = new_collection.insert_many(mylist)
```

打开可视化工具查看结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/556f219375bd4b7e90bddcfd9e9d368a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 插入指定 _id 的多个文档

我们也可以自己指定 `_id`插入，以下实例我们在 new_collection1 集合中插入数据，_id 为我们指定的：

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection1 = new_database.new_collection1

mylist = [
    {"_id": 1, "name": "小红"},
    {"_id": 2, "name": "小黑"},
    {"_id": 3, "name": "小白"},
    {"_id": 4, "name": "小蓝"},
    {"_id": 5, "name": "小黄"}
]

# 使用insert_many插入多条数据
x = new_collection1.insert_many(mylist)

# 输出插入的所有文档对应的 _id 值
print(x.inserted_ids)  # 输出结果：[1, 2, 3, 4, 5]
```

打开可视化工具查看结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/89c92714b4094d10a70ff891e66f644e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 5、查看文档

MongoDB 中使用了 find() 和 find_one 方法来查询集合中的数据，它类似于 SQL 中的 SELECT 语句。

#### 查询一条数据

我们可以使用 `find_one()` 方法来查询集合中的一条数据

例如：查看 new_collection 集合一条数据

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

x = new_collection.find_one()

print(x)
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '张三', 'age': 18}
```

#### 查询集合中所有数据

`find()` 方法可以查询集合中的所有数据，类似 SQL 中的 SELECT * 操作。

例如：查看 new_collection 集合中所有数据

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

for x in new_collection.find():
    print(x)
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '张三', 'age': 18}
{'_id': ObjectId('6223436592276f5f7b639bed'), 'name': '张飞', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bee'), 'name': '关羽', 'hometown': '蜀国', 'age': 40, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bef'), 'name': '刘备', 'hometown': '蜀国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf0'), 'name': '曹操', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf1'), 'name': '司马懿', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf2'), 'name': '孙权', 'hometown': '吴国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf3'), 'name': '貂蝉', 'hometown': '未知', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf4'), 'name': '西施', 'hometown': '越国', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf5'), 'name': '王昭君', 'hometown': '西汉', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf6'), 'name': '杨玉环', 'hometown': '唐朝', 'age': 18, 'sex': '女'}
```

#### 查询指定字段的数据

我们可以使用 find() 方法来查询指定字段的数据，将要返回的字段对应值设置为 1。

例如：只查看 new_collection 集合中的 name 字段

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

for x in new_collection.find({}, {"_id": 0, "name": 1}):
    print(x)
```

输出结果：

```python
{'name': '张三'}
{'name': '张飞'}
{'name': '关羽'}
{'name': '刘备'}
{'name': '曹操'}
{'name': '司马懿'}
{'name': '孙权'}
{'name': '貂蝉'}
{'name': '西施'}
{'name': '王昭君'}
{'name': '杨玉环'}
```

注意：除了 `_id`，你不能在一个对象中同时指定 0 和 1，如果你设置了一个字段为 0，则其他都为 1，反之亦然。

#### 根据指定条件查询

我们可以在 `find()` 中设置参数来过滤数据。

例如：查看 new_collection 集合中 age 为 18 的数据

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"age": 18}

mydoc = new_collection.find(myquery)

for x in mydoc:
    print(x) 
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '张三', 'age': 18}
{'_id': ObjectId('6223436592276f5f7b639bf3'), 'name': '貂蝉', 'hometown': '未知', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf4'), 'name': '西施', 'hometown': '越国', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf5'), 'name': '王昭君', 'hometown': '西汉', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf6'), 'name': '杨玉环', 'hometown': '唐朝', 'age': 18, 'sex': '女'}
```

#### 高级查询

查询的条件语句中，我们还可以使用修饰符

例如：查看 new_collection 集合中 age 大于 18 的数据

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"age": {"$gt": 18}}

mydoc = new_collection.find(myquery)

for x in mydoc:
    print(x)
```

输出结果：

```python
{'_id': ObjectId('6223436592276f5f7b639bed'), 'name': '张飞', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bee'), 'name': '关羽', 'hometown': '蜀国', 'age': 40, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bef'), 'name': '刘备', 'hometown': '蜀国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf0'), 'name': '曹操', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf1'), 'name': '司马懿', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf2'), 'name': '孙权', 'hometown': '吴国', 'age': 50, 'sex': '男'}
```

#### 使用[正则表达式](https://so.csdn.net/so/search?q=正则表达式&spm=1001.2101.3001.7020)查询

我们还可以使用正则表达式作为修饰符。正则表达式修饰符只用于搜索字符串的字段。

案例：查看name 以`张`开头的数据，正则表达式修饰符条件为 {“$regex”: “^张”}

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"name": {"$regex": "^张"}}

mydoc = new_collection.find(myquery)

for x in mydoc:
    print(x)
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '张三', 'age': 18}
{'_id': ObjectId('6223436592276f5f7b639bed'), 'name': '张飞', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
```

#### 返回指定条数记录

如果我们要对查询结果设置指定条数的记录可以使用 limit() 方法，该方法只接受一个数字参数。

案例：以下实例返回 3 条文档记录：

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myresult = new_collection.find().limit(3)

# 输出结果
for x in myresult:
    print(x)
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '张三', 'age': 18}
{'_id': ObjectId('6223436592276f5f7b639bed'), 'name': '张飞', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bee'), 'name': '关羽', 'hometown': '蜀国', 'age': 40, 'sex': '男'}
```

### 6、修改文档

#### 修改一条

我们可以在 MongoDB 中使用 update_one() 方法修改文档中的记录。该方法第一个参数为查询的条件，第二个参数为要修改的字段。

如果查找到的匹配数据多于一条，则只会修改第一条。

案例：把张三的名字修改为 张三三

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"name": "张三"}
newvalues = {"$set": {"name": "张三三"}}

new_collection.update_one(myquery, newvalues)
```

打开可视化工具查看结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cdfc0c03e078472e8ca4e3d2b7f2cf44.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 修改全部

update_one() 方法只能修匹配到的第一条记录，如果要修改所有匹配到的记录，可以使用 update_many()。
案例：把 集合中姓张的修改为李好啊

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"name": {"$regex": "^张"}}
newvalues = {"$set": {"name": "李好啊"}}

new_collection.update_many(myquery, newvalues)
```

[data-channel:将数据通道转换为流zip![img](https://csdnimg.cn/release/blogv2/dist/components/img/star.png)0星超过10%的资源3KB![img](https://csdnimg.cn/release/blogv2/dist/components/img/arrowDownWhite.png)下载](https://download.csdn.net/download/weixin_42110038/19871459)

打开可视化工具查看结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/78bd1dd29ce64b59a45c95cda9dd0fe5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 7、删除数据

### 输出一个文档

我们可以使用 delete_one() 方法来删除一个文档，该方法第一个参数为查询对象，指定要删除哪些数据。

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"name": "关羽"}

new_collection.delete_one(myquery)

# 删除后输出
for x in new_collection.find():
    print(x)
```

输出结果：

```python
{'_id': ObjectId('6223413c927c6c0e82b9a4b0'), 'name': '李好啊', 'age': 18}
{'_id': ObjectId('6223436592276f5f7b639bed'), 'name': '李好啊', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bef'), 'name': '刘备', 'hometown': '蜀国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf0'), 'name': '曹操', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf1'), 'name': '司马懿', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf2'), 'name': '孙权', 'hometown': '吴国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf3'), 'name': '貂蝉', 'hometown': '未知', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf4'), 'name': '西施', 'hometown': '越国', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf5'), 'name': '王昭君', 'hometown': '西汉', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf6'), 'name': '杨玉环', 'hometown': '唐朝', 'age': 18, 'sex': '女'}
12345678910
```

### 删除多个文档

我们可以使用 delete_many() 方法来删除多个文档，该方法第一个参数为查询对象，指定要删除哪些数据。

删除所有 name 字段中以 李 开头的文档：

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

myquery = {"name": {"$regex": "^李"}}

x = new_collection.delete_many(myquery)

print(x.deleted_count, "个文档已删除")

# 删除后输出
for x in new_collection.find():
    print(x)
12345678910111213141516171819
```

输出结果：

```python
2 个文档已删除
{'_id': ObjectId('6223436592276f5f7b639bef'), 'name': '刘备', 'hometown': '蜀国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf0'), 'name': '曹操', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf1'), 'name': '司马懿', 'hometown': '魏国', 'age': 45, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf2'), 'name': '孙权', 'hometown': '吴国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223436592276f5f7b639bf3'), 'name': '貂蝉', 'hometown': '未知', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf4'), 'name': '西施', 'hometown': '越国', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf5'), 'name': '王昭君', 'hometown': '西汉', 'age': 18, 'sex': '女'}
{'_id': ObjectId('6223436592276f5f7b639bf6'), 'name': '杨玉环', 'hometown': '唐朝', 'age': 18, 'sex': '女'}
123456789
```

### 删除集合中的所有文档

delete_many() 方法如果传入的是一个空的查询对象，则会删除集合中的所有文档：

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

x = new_collection.delete_many({})

print(x.deleted_count, "个文档已删除")

# 删除后输出
for x in new_collection.find():
    print(x)
1234567891011121314151617
```

输出结果：

```python
8 个文档已删除
1
```

### 删除集合

我们可以使用 drop() 方法来删除一个集合。

以下实例删除了 customers 集合：

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

new_collection.drop()
1234567891011
```

打开可视化工具查看结果：已经没有 new_collection 了![在这里插入图片描述](https://img-blog.csdnimg.cn/a585ee3e68564d9c848093ec0d9b6988.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_10,color_FFFFFF,t_70,g_se,x_16)

## 8、排序

sort() 方法可以指定升序或降序排序。

sort() 方法第一个参数为要排序的字段，第二个字段指定排序规则，1 为升序，-1 为降序，默认为升序。

### 升序

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

mylist = [
    {"name": "张飞", "hometown": "蜀国", "age": 30, "sex": "男"},
    {"name": "关羽", "hometown": "蜀国", "age": 40, "sex": "男"},
    {"name": "刘备", "hometown": "蜀国", "age": 50, "sex": "男"}
]

# 使用insert_many插入多条数据
x = new_collection.insert_many(mylist)

mydoc = new_collection.find().sort("name")
for x in mydoc:
    print(x)
12345678910111213141516171819202122
```

### 降序

```python
from pymongo import MongoClient

client = MongoClient(host='127.0.0.1', port=27017, tz_aware=True)

# 创建新的数据库
new_database = client.new_database

# 创建一个新的集合
new_collection = new_database.new_collection

mydoc = new_collection.find().sort("age", -1)
for x in mydoc:
    print(x)
12345678910111213
```

输出结果：

```python
{'_id': ObjectId('6223589a5fbb76bdbb85050c'), 'name': '刘备', 'hometown': '蜀国', 'age': 50, 'sex': '男'}
{'_id': ObjectId('6223589a5fbb76bdbb85050b'), 'name': '关羽', 'hometown': '蜀国', 'age': 40, 'sex': '男'}
{'_id': ObjectId('6223589a5fbb76bdbb85050a'), 'name': '张飞', 'hometown': '蜀国', 'age': 30, 'sex': '男'}
```