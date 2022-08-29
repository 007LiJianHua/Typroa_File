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

### 2、各种查询条件

#### 1、比较运算符

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

#### 2、条件运算符

##### $in

$in有点类似于SQL中的in关键字，表示查询某一个字段在某一个范围中的所有文档，比如我想查询**x为1或者2**的所有文档，如下：

```json
db.sang_collect.find({x:{$in:[1,2]}})
```

in恰好相反，表示查询某一个字段不在某一个范围内的所有文档，比如我想查询**x不为1或者2(不为1且不为2)**的所有文档，如下：

```json
db.sang_collect.find({x:{$nin:[1,2]}})
```

##### $or

$or有点类似于SQL中的or关键字，表示多个查询条件之间是**或**的关系，比如我想查询x为1或者y为99的文档，如下：

```json
db.sang_collect.find({$or:[{x:1},{y:99}]})
```

##### $type

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

##### $not

$not用来执行取反操作，比如我想要查询所有x的类型不为数字的文档，如下：

```json
db.sang_collect.find({x:{$not:{$type:1}}})
```

##### $and

$and类似于SQL中的and，比如我想查询y大于98并且小于100的数据，如下：

```json
db.sang_collect.find({$and:[{y:{$gt:98}},{y:{$lt:100}}]})
```

上面的操作我们也可以使用下面简化的写法：

```json
db.sang_collect.find({y:{$lt:100,$gt:98}})
```

#### 3、null

null的查询稍微有点不同，假如我想查询z为null的数据，如下：

```c
db.sang_collect.find({z:null})
```

这样不仅会查出z为null的文档，也会查出所有没有z字段的文档，如果只想查询z为null的字段，那就再多加一个条件，判断一下z这个字段存在不，如下：

```c++
db.sang_collect.find({z:{$in:[null],$exists:true}})
```

### 3、正则表达式查询

使用正则表达式查询我们在前面也已经介绍过了，这里的正则表达式语法和JavaScript中的正则表达式语法一致，比如`查询所有key为x，value以hello开始的文档且不区分大小写`：

```c
db.sang_collec.find({x:/^(hello)(.[a-zA-Z0-9])+/i})
```

### 4、数组查询

假设我有一个数据集如下：

```c
{
    "_id" : ObjectId("59f1ad41e26b36b25bc605ae"),
    "books" : [ 
        "三国演义", 
        "红楼梦", 
        "水浒传"
    ]
}
```

查询books中含有三国演义的文档，如下：

```c
db.sang_collect.find({books:"三国演义"})
```

如果要查询既有三国演义又有红楼梦的文档，可以使用$all，如下：

```c
db.sang_collect.find({books:{$all:["三国演义","红楼梦"]}})
```

当然我们也可以使用精确匹配，比如查询books为`"三国演义","红楼梦", "水浒传"`的数据，如下：

```c
db.sang_collect.find({books:["三国演义","红楼梦", "水浒传"]})
```

不过这种就会一对一的精确匹配。

也可以按照下标匹配，比如我想查询数组中下标为2的项的为`"水浒传"`的文档，如下：

```c
db.sang_collect.find({"books.2":"水浒传"})
```

也可以按照数组长度来查询，比如我想查询数组长度为3的文档：

```c
db.sang_collect.find({books:{$size:3}})
```

如果想查询数组中的前两条数据，可以使用$slice，如下：

```c
db.sang_collect.find({},{books:{$slice:2}})
```

注意这里要写在find的第二个参数的位置。`2表示数组中前两个元素`，`-2表示从后往前数两个元素`。也可以截取数组中间的元素，比如查询数组的第二个到第四个元素：

```c
db.sang_collect.find({},{books:{$slice:[1,3]}})
```

数组中的与的问题也值得说一下，假设我有如下数据：

```c
{
    "_id" : ObjectId("59f208bc7b00f982986c669c"),
    "x" : [ 
        5.0, 
        25.0
    ]
}
```

我想将数组中value取值在(10,20)之间的文档获取到，如下操作：

```c
db.sang_collect.find({x:{$lt:20,$gt:10}})
```

此时上面这个文档虽然不满足条件却依然被查找出来了，因为`5<20`，而`25>10`，要解决这个问题，我们可以使用$elemMatch，如下：

```c
db.sang_collect.find({x:{$elemMatch:{$lt:20,$gt:10}}})
```

`$elemMatch`要求MongoDB同时使用查询条件中的两个语句与一个数组元素进行比较。

### 5、嵌套文档查询

嵌套文档有两种查询方式，比如我的数据如下：

```c
{
    "_id" : ObjectId("59f20c9b7b00f982986c669f"),
    "x" : 1.0,
    "y" : {
        "z" : 2.0,
        "k" : 3.0
    }
}
```

想要查询上面这个文档，我的查询语句如下：

```c
db.sang_collect.find({y:{z:2,k:3}})
```

但是这种写法要求严格匹配，顺序都不能变，假如写成了`db.sang_collect.find({y:{k:3,z:2}})`，就匹配不到了，因此这种方法不够灵活，我们一般推荐的是下面这种写法：

```c
db.sang_collect.find({"y.z":2,"y.k":3})
```

这种写法可以任意颠倒顺序。

### 6、游标

游标这个概念在很多地方都有，Java中JDBC里的ResultSet，Android中的Cursor等等都是，MongoDB中也有类似的概念。当我们调用find方法时，就可以返回一个游标，如下：

```c
var cursor = db.sang_collect.find();
```

游标中有hasNext()方法，也有next()方法，这两个方法结合可以用来遍历结果，如下：

```c
while(cursor.hasNext()){
    print(cursor.next())
}
```

#### next()

方法可以获取查询到的每一个文档，如下：

```c
{
    "_id" : ObjectId("59f299579babb96c21ddc9e8"),
    "x" : 0.0,
    "y" : 1000.0
}

/* 2 */
{
    "_id" : ObjectId("59f299579babb96c21ddc9e9"),
    "x" : 1.0,
    "y" : 999.0
}
```

如果我只想获取文档中的某一个字段，可以按如下方式：

```c
while(cursor.hasNext()){
    print(cursor.next().y)
}
```

cursor也实现了JavaScript中的迭代器接口，所以我们也可以直接调用forEach方法来遍历：

```c
cursor.forEach(function(x){
    print(x)
    })
```

当我们调用`find方法`获取cursor时，shell并不会立即查询数据库，而是在真正使用数据时才会去加载，这有点类似于数据库框架中的懒加载，shell在每次查询的时候会获取前100条结果或者前4MB数据(两者之间取最小)，然后我们调用hasNext和next时shell就不用再去连接数据库了，直接一条一条的返回查询到的数据，这100条或者4MB数据全部被返回之后，shell才会再次发起请求向MongoDB要数据。

#### limit

limit是cursor中的方法，用来限制返回结果的数量，比如我只想获取查询的前三条结果，方式如下：

```c
var cursor = db.sang_collect.find().limit(3)
```

#### skip

skip也是cursor中的方法，用来表示跳过的记录数，比如我想获取第2到第5条记录，如下：

```c
var cursor = db.sang_collect.find().skip(2).limit(4)
```

跳过前两条(0和1)然后获取后面4条数据，skip和limit结合有点类似于MySQL中的limit，可以用来做分页，不过这种分页方式效率过低。

#### sort

sort用来实现排序功能，比如按x排序，如下：

```c
var cursor = db.sang_collect.find().sort({x:-1})
```

1表示升序，-1表示降序。