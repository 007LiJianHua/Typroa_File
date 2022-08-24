[toc]

MongoDB中每条记录称作一个文档，这个文档和我们平时用的JSON有点像，但也不完全一样。JSON是一种轻量级的数据交换格式。简洁和清晰的层次结构使得JSON成为理想的数据交换语言，JSON易于阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率，但是JSON也有它的局限性，比如它只有null、布尔、数字、字符串、数组和对象这几种数据类型，没有日期类型，只有一种数字类型，无法区分浮点数和整数，也没法表示正则表达式或者函数。由于这些局限性，`BSON`闪亮登场啦，BSON是一种类JSON的二进制形式的存储格式，简称`Binary JSON`，它和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型，MongoDB使用BSON做为文档数据存储和网络传输格式。

### 1、数字

shell默认使用64位浮点型数值，如下：

```java
db.sang_collec.insert({x:3.1415926})
db.sang_collec.insert({x:3})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqhqHQLIJ90gQvoLLD8B9pzSQX1L2RjoibOVsjzJN2u0ROL8IGgGZia1YQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

对于整型值，我们可以使用NumberInt或者NumberLong表示，如下：

```java
db.sang_collec.insert({x:NumberInt(10)})
db.sang_collec.insert({x:NumberLong(12)})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6Ndq55ibzxssdWKiaWWO8h8cVt2HWQAjrc20LuqWdmEu29ghCicnFgtVo9kbg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 2、字符串

字符串也可以直接存储，如下：

```java
db.sang_collec.insert({x:"hello MongoDB!"})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6Ndqo4FYKNfHfLUQPKluFWd5c9AUeQxsSjgibQyLMqbsYros3ykFlqjcUTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 3、正则表达式

正则表达式主要用在查询里边，查询时我们可以使用正则表达式，语法和JavaScript中正则表达式的语法相同，比如查询所有`key为x`，`value以hello`开始的文档且不区分大小写：

```java
db.sang_collec.find({x:/^(hello)(.[a-zA-Z0-9])+/i})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqLmS1texXwu8WzrefiaA9rM1nzowKZicMSXyHFj6zbFy49qJfkibeAUkgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 4、数组

数组一样也是被支持的，如下：

```java
db.sang_collec.insert({x:[1,2,3,4,new Date()]})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqicYj4k8exCAicHicnTma1SZWSTj8V1Krxs3pT2voCXrko9vJ3TVoFTr6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

数组中的数据类型可以是多种多样的。

### 5、日期

MongoDB支持Date类型的数据，可以直接new一个Date对象，如下：

```java
db.sang_collec.insert({x:new Date()})
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqZKxUPMcuxvE3yf7s0JSUxljRq2RibV2MumxicEvUiaV0uGR6YHfthxVLw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### 6、内嵌文档

一个文档也可以作为另一个文档的value，这个其实很好理解，如下：

```java
db.sang_collect.insert({name:"三国演义",author:{name:"罗贯中",age:99}});
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqBwqvY920G2Kejdjz9HU4lNL1SQf5dWt9JJ1YBa2Htn4KkT4qst3xOQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

书有一个属性是作者，作者又有name，年龄等属性。

### 7、ObjectId

我们在前面提到过，我们每次插入一条数据系统都会自动帮我们插入一个`_id`键，这个**键的值不可以重复**，它可以是任何类型的，我们也可以手动的插入，默认情况下它的数据类型是ObjectId，由于MongoDB在设计之初就是用作分布式数据库，所以使用ObjectId可以避免不同数据库中`_id`的重复（如果使用自增的方式在分布式系统中就会出现重复的`_id`的值），这个特点有点类似于Git中的版本号和Svn中版本号的区别。

`ObjectId使用12字节的存储空间`，每个字节可以存储两个十六进制数字，所以一共可以存储24个十六进制数字组成的字符串，在这24个字符串中，前8位表示时间戳，接下来6位是一个机器码，接下来4位表示进程id，最后6位表示计数器。

### 8、二进制

MongoDB中也可以存储二进制数据，不过这种情况并不多，二进制数据的存储不能在shell中操作，我们在后面的代码中会介绍这种存储方式。

### 9、代码

文档中也可以包括JavaScript代码，如下：

```java
db.sang_collect.insert({x:function f1(a,b){return a+b;}});
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYm4hs6deF49BFiapdhKX6NdqhictbmxXBfJx6czLoiaM6Zpk0QaicvwXRWe0hRQnqZmcgY6RMgu8sxQQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)