[toc]

### 1、find方法再探

find方法是很重要的一个查询方法，我们在前面也已经使用过多次了，一般情况下我们调用的是：

```json
find()
```

没有传入任何参数，这个等价于：

```json
find({})
```

都表示没有查询条件，查询所有的数据。如果有查询条件，我们传入查询条件即可，查询条件也是一个文档，如下表示`查询x为1`的文档：

```json
db.sang_collect.find({x:1})
```

如果查询条件文档中有多个字段，多个字段之间的关系是AND，如下表示`查询x为1并且y为99`的文档：

```json
db.sang_collect.find({x:1,y:99})
```

默认情况下，每次查询都会返回文档中所有的key/value对，我们也可以自定义返回的字段，如下表示`只返回x字段`，其他字段都不返回：

```json
db.sang_collect.find({},{x:1})
```

参数1表示返回某一个字段，0表示不返回某一个字段，当我们设置只返回x的时候，`_id`默认还是返回的，如果不想返回`_id`，我们可以设置`_id`为0，如下：

```json
db.sang_collect.find({},{x:1,_id:0})
```

此时返回的数据中就不包括`_id`字段了。

## 各种查询条件

### 比较运算符

这里的比较运算符都比较好理解，如下表：

| 符号   | 含义 |
| ------ | ---- |
| `$lt`  | <    |
| `$lte` | `<=` |
| `$gt`  | >    |
| `$gte` | `>=` |
| `$ne`  | `!=` |

比如我想查询所有成绩在[90,100]之间的学生：

原始数据如下：

```json
/* 2 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a666"),
    "name" : "zs",
    "score" : 100.0
}

/* 3 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a667"),
    "name" : "ls",
    "score" : 90.0
}

/* 4 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a668"),
    "name" : "ww",
    "score" : 70.0
}

/* 5 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a669"),
    "name" : "zl",
    "score" : 80.0
}
```

查询操作如下：

```json
db.sang_collect.find({score:{$lte:100,$gte:90}})
```

查询结果如下：

```json
/* 1 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a666"),
    "name" : "zs",
    "score" : 100.0
}

/* 2 */
{
    "_id" : ObjectId("59f0b17249fc5c9c2412a667"),
    "name" : "ls",
    "score" : 90.0
}
```

如果要查询分数不为90的学生，操作如下：

```json
db.sang_collect.find({score:{$ne:90}})
```

### nin

$in有点类似于SQL中的in关键字，表示查询某一个字段在某一个范围中的所有文档，比如我想查询**x为1或者2**的所有文档，如下：

```json
db.sang_collect.find({x:{$in:[1,2]}})
```

in恰好相反，表示查询某一个字段不在某一个范围内的所有文档，比如我想查询**x不为1或者2(不为1且不为2)**的所有文档，如下：

```json
db.sang_collect.find({x:{$nin:[1,2]}})
```

### $or

$or有点类似于SQL中的or关键字，表示多个查询条件之间是**或**的关系，比如我想查询x为1或者y为99的文档，如下：

```json
db.sang_collect.find({$or:[{x:1},{y:99}]})
```

### $type

$type可以用来根据数据类型查找数据，比如我想要查找x类型为数字的文档，如下：

```json
db.sang_collect.find({x:{$type:1}})
```

1表示数字，其他数据类型对应的数字参见下表。

| 类型                   | 对应数字 | 别名                | 说明 |
| ---------------------- | -------- | ------------------- | ---- |
| Double1                | 1        | double              |      |
| String                 | 2        | string              |      |
| Object                 | 3        | object              |      |
| Array                  | 4        | array               |      |
| Binary data            | 5        | binData             |      |
| Undefined              | 6        | undefined           | 弃用 |
| ObjectId               | 7        | objectId            |      |
| Boolean                | 8        | bool                |      |
| Date                   | 9        | date                |      |
| Null                   | 10       | null                |      |
| Regular Expression     | 11       | regex               |      |
| DBPointer              | 12       | dbPointer           |      |
| JavaScript             | 13       | javascript          |      |
| Symbol                 | 14       | symbol              |      |
| JavaScript(with scope) | 15       | javascriptWithScope |      |
| 32-bit integer         | 16       | int                 |      |
| Timestamp              | 17       | timestamp           |      |
| 64-bit integer         | 18       | long                |      |
| Min key                | -1       | minKey              |      |
| Max key                | 127      | maxKey              |      |

### $not

$not用来执行取反操作，比如我想要查询所有x的类型不为数字的文档，如下：

```json
db.sang_collect.find({x:{$not:{$type:1}}})
```

### $and

$and类似于SQL中的and，比如我想查询y大于98并且小于100的数据，如下：

```json
db.sang_collect.find({$and:[{y:{$gt:98}},{y:{$lt:100}}]})
```

上面的操作我们也可以使用下面简化的写法：

```json
db.sang_collect.find({y:{$lt:100,$gt:98}})
```