[toc]

### 1、文档替换

假设我的集合中现在存了如下一段数据：

```java
{
    "_id" : ObjectId("59f005402844ff254a1b68f6"),
    "name" : "三国演义",
    "authorName" : "罗贯中",
    "authorGender" : "男",
    "authorAge" : 99.0
}
```

这是一本书，有书名和作者信息，但是作者是一个独立的实体，所以我想将之提取出来，变成下面这样：

```java
{
    "_id" : ObjectId("59f005402844ff254a1b68f6"),
    "name" : "三国演义",
    "author" : {
        "name" : "罗贯中",
        "gender" : "男",
        "age" : 99.0
    }
}
```

我可以采用如下操作：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmNMHUfOsSHGKz8yG8lFibXeJiavHLe21XWzFI4velm6JvJ43MITTUibksBAPjlhoclFicNzxU1WTHxqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

>  另外一个问题是更新时，MongoDB只会匹配第一个更新的文档，假设我的MongoDB中有如下数据：

```java
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f7"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f8"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f9"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68fa"), "x" : 2 }
```

我想把所有x为1的数据改为99，我们很容易想到如下命令：

```java
db.sang_collect.update({x:1},{x:99})
```

但我们发现执行结果却是这样：

```json
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f7"), "x" : 99 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f8"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f9"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68fa"), "x" : 2 }
```

即只有第一条匹配的结果被更新了，其他的都没有变化。这是MongoDB的更新规则，即只更新第一条匹配结果。如果我们想将所有x为1的更新为x为99，可以采用如下命令：

```json
db.sang_collect.update({x:1},{$set:{x:99}},false,true)
```

首先我们将要修改的数据赋值给set是一个`修改器`，我们将在下文详细讲解，然后后面多了两个参数，*第一个false表示如果不存在update记录*，是否将我们要更新的文档作为一个新文档插入，`true表示插入`，`false表示不插入`，默认为false，*第二个true表示是否更新全部查到的文档*，`false表示只更新第一条记录`，`true表示更新所有查到的文档`。

### 2、使用修改器

很多时候我们修改文档，只是要修改文章的某一部分，而不是全部，但是现在我面临这样一个问题，假设我有如下一个文档：

```json
{x:1,y:2,z:3}
```

我现在想把这个文档中x的值改为99，我可能使用如下操作：

```json
db.sang_collect.update({x:1},{x:99})
```

但是更新结果却变成了这样：

```json
{ "_id" : ObjectId("59f02dce95769f660c09955b"), "x" : 99 }
```

如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6Ndqjguq9KU2S3gNVGLktYoXj8icSzIzVeKNNdQYQibiaobb4sJjylwHFYeEg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

**MongoDB帮我把整个文档更新了！要解决这个问题，我们可以使用修改器。**

### 3、$set修改器

$set可以用来修改一个字段的值，如果这个字段不存在，则创建它。如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqSWMyT735whezM1JF0alyy0JOH3FcoEzXx9kqOEBE7ruNT5MMd9oU1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

```json
> db.sang_collecc.find()
{ "_id" : ObjectId("6305a119f8f7216c777bd1de"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c514f8f7216c777bd1df"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c8b7f8f7216c777bd1e9"), "x" : 1, "y" : 2, "z" : 3 }
> db.sang_collecc.update({x:1},{$set:{x:99}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.sang_collecc.find()
{ "_id" : ObjectId("6305a119f8f7216c777bd1de"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c514f8f7216c777bd1df"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c8b7f8f7216c777bd1e9"), "x" : 99, "y" : 2, "z" : 3 }
```

如果该字段不存在，则创建，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqHBxUjvMO6ULT9RlrNXyyz9g7oteEEbhhcjRHNUdG2icOvGzNYvFqWrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

```json
> db.sang_collecc.update({x:99},{$set:{i:99999}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.sang_collecc.find()
{ "_id" : ObjectId("6305a119f8f7216c777bd1de"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c514f8f7216c777bd1df"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c8b7f8f7216c777bd1e9"), "x" : 99, "y" : 2, "z" : 3, "i" : 99999 }
```

也可以利用$unset删除一个字段，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6Ndqa2ZIgWkoKyFtfEcicIKfRl2HB6Yy52kTefJvwp6KvRiciaQOSoW3ibRP5A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

```json
> db.sang_collecc.find()
{ "_id" : ObjectId("6305a119f8f7216c777bd1de"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c514f8f7216c777bd1df"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c8b7f8f7216c777bd1e9"), "x" : 99, "y" : 2, "z" : 3, "i" : 99999 }
> db.sang_collecc.update({i:99999},{$unset:{i:99999}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.sang_collecc.find()
{ "_id" : ObjectId("6305a119f8f7216c777bd1de"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c514f8f7216c777bd1df"), "name" : "三国演义", "author" : { "name" : "罗贯中", "age" : 99 } }
{ "_id" : ObjectId("6305c8b7f8f7216c777bd1e9"), "x" : 99, "y" : 2, "z" : 3 }
```

$set也可以用来修改内嵌文档，还以刚才的书为例，如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "罗贯中",
        "gender" : "男"
    }
}
```

想要修改作者的名字，操作如下：

```json
db.sang_collect.update({name:"三国演义"},{$set:{"author.name":"明代罗贯中"}})
```

修改结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男"
    }
}
```

### 4、$inc修改器

$inc用来增加已有键的值，如果该键不存在就新创建一个。比如我想给上文的罗贯中增加一个年龄为99，方式如下：

```json
db.sang_collect.update({name:"三国演义"},{$inc:{"author.age":99}})
```

执行结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 99.0
    }
}
```

加入我想给罗贯中增加1岁，执行如下命令：

```json
db.sang_collect.update({name:"三国演义"},{$inc:{"author.age":1}})
```

这是会在现有值上加1，结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    }
}
```

**注意$inc只能用来操作数字，不能用来操作null、布尔等。**

### 5、数组修改器

数组修改器有好几种，我们分别来看。
`$push`可以向已有数组末尾追加元素，要是不存在就创建一个数组，还是以我们的上面的book为例，假设book有一个字段为comments，是一个数组，表示对这个book的评论，我们可以使用如下命令添加一条评论：

```json
db.sang_collect.update({name:"三国演义"},{$push:{comments:"好书666"}})
```

此时不存在comments字段，系统会自动帮我们创建该字段，结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666"
    ]
}
```

此时我们可以追加评论，如下：

```json
db.sang_collect.update({name:"三国演义"},{$push:{comments:"好书666啦啦啦啦"}})
```

结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666", 
        "好书666啦啦啦啦"
    ]
}
```

如果想一次添加3条评论，可以结合`$each`一起来使用，如下：

```json
db.sang_collect.update({name:"三国演义"},{$push:{comments:{$each:["111","222","333"]}}})
```

结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666", 
        "好书666啦啦啦啦", 
        "111", 
        "222", 
        "333"
    ]
}
```

我们可以使用$slice来固定数组的长度，假设我固定数组的长度为5，如果数组中的元素不足5个，则全部保留，如果数组中的元素超过5个，则只会保留最新的5个，如下：

```json
db.sang_collect.update({name:"三国演义"},{$push:{comments:{$each:["444","555"],$slice:-5}}})
```

注意$slice的值为负数，运行结果如下：

```json
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "111", 
        "222", 
        "333", 
        "444", 
        "555"
    ]
}
```

我们还可以在清理之前使用`$sort`对数据先进行排序，然后再清理比如我有一个class文档，如下：

```json
{
    "_id" : ObjectId("59f07f3649fc5c9c2412a662"),
    "class" : "三年级二班"
}
```

现在向这个文档中插入student，每个student有姓名和成绩，然后按照成绩降序排列，只要前5条数据，如下：

```json
db.sang_collect.update({class:"三年级二班"},{$push:{students:{$each:[{name:"张一百",score:100},{name:"张九九",score:99},{name:"张九八",score:98}],$slice:5,$sort:{score:-1}}}})
```

$sort的取值为-1和1，-1表示降序，1表示升序。
上面的命令执行两次之后（即插入两次），结果如下：

```json
{
    "_id" : ObjectId("59f07f3649fc5c9c2412a662"),
    "class" : "三年级二班",
    "students" : [ 
        {
            "name" : "张一百",
            "score" : 100.0
        }, 
        {
            "name" : "张一百",
            "score" : 100.0
        }, 
        {
            "name" : "张九九",
            "score" : 99.0
        }, 
        {
            "name" : "张九九",
            "score" : 99.0
        }, 
        {
            "name" : "张九八",
            "score" : 98.0
        }
    ]
}
```

**sort不能只和each。** 

### 6、$addToSet

我们可以在插入的时候使用`$addToSet`，表示要插入的值如果存在则不插入，否则插入，如下：

```json
db.sang_collect.update({name:"三国演义"},{$addToSet:{comments:"好书"}})
```

上面的命令执行多次之后，发现只成功插入了一条数据。也可以将each结合起来使用，如下：

```json
db.sang_collect.update({name:"三国演义"},{$addToSet:{comments:{$each:["111","222","333"]}}})
```

### 7、$pop

$pop可以用来删除数组中的数据，如下：

```json
db.sang_collect.update({name:"三国演义"},{$pop:{comments:1}})
```

**1表示从comments数组的末尾删除一条数据，-1表示从comments数组的开头删除一条数据。**

### 8、$pull

使用$pull我们可以按条件删除数组中的某个元素，如下：

```json
db.sang_collect.update({name:"三国演义"},{$pull:{comments:"444"}})
```

表示删除数组中值为444的数据。

### 9、$

既然是数组，我们当然可以通过下标来访问，如下一行操作表示将下标为0的(第一个comments)comments修改为999：

```json
db.sang_collect.update({name:"三国演义"},{$set:{"comments.0":"999"}})
```

可是有的时候我并不知道我要修改的数据处于数组中的什么位置，这个时候可以使用$符号来解决：

```json
db.sang_collect.update({comments:"333"},{$set:{"comments.$":"333-1"}})
```

查询条件查出来333的下标，符号就能将之修改。

### 10、save

save是shell中的一个函数，接收一个参数，这个参数就是文档，如果文档中有`_id`参数save会执行更新操作，否则执行插入操作，使用save操作我们可以方便的完成一些更新操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6Ndqu1iaJ4bA2HGfmAicvYOg3elYtuKUrZXLyxXOIlGMnGISjlQTryyqicUbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

类似于如下命令则表示一个插入操作(因为没有`_id`)：

```json
db.sang_collect.save({x:111})
```