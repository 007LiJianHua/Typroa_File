[toc]

## 一、日志基础知识

### 1、日志是什么

> **日志**（logging）是一种可追踪（track）某些软件运行时所发生事件的方法。软件开发人员 可在他们的代码中调用日志记录相关的方法来表明发生了某些事件。通过一个描述性的消息来描述这个事件，该消息能够可选地包含可变数据。而
> 事件有重要性的概念，重要性被称为 严重性级别（Level）。

### 2、日志的作用

通过记录和分析日志可以了解一个系统 或软件程序运行情况是否正常，也可以在应用程序出现故障时快速定位问题。例如：开发者可通过在控制台上输出的各种日志进行程序调试；运维人员在接收到报警或各种问题反馈后，进行问题排查时通常都会先看各种日志，大部分问题都可在日志中找到答案。日志的作用可总结为3点：

- 1、程序调试
- 2、了解软件程序运行情况，是否正常
- 3、软件程序运行故障分析与问题定位

### 3、日志的级别

日志的级别有很多，我们一般只用四个，日志级别由低到高`DEBUG - INFO - WARN - ERROR`；

- **DEBUG**（调试）：开发调试日志。一般来说，在系统实际运行过程中，不会输出该级别的日志。因此，开发人员可以打印任何自己觉得有利于了解系统运行状态的东东。不过很多场景下，过多的DEBUG日志，并不是好事，建议是按照业务逻辑的走向打印。打个比方，打印日志就像读书时划重点，如果导出都是重点，也就失去了重点。
- **INFO**（通知）：INFO日志级别主要用于记录系统运行状态等关联信息。该日志级别，常用于反馈系统当前状态给最终用户。所以，在这里输出的信息，应该对最终用户具有实际意义，也就是最终用户要能够看得明白是什么意思才行。
- **WARN**（警告）：WARN日志常用来表示系统模块发生问题，但并不影响系统运行。 此时，进行一些修复性的工作，还能把系统恢复到正常的状态。
- **ERROR**（错误）：此信息输出后，主体系统核心模块正常工作，需要修复才能正常工作。 就是说可以进行一些修复性的工作，但无法确定系统会正常的工作下去，系统在以后的某个阶段，很可能会因为当前的这个问题，导致一个无法修复的错误（例如宕机），但也可能一直工作到停止也不出现严重问题。

### 4、日志的内容

一条日志信息对应的是 一个事件的发生，而一个事件通常需要包括以下内容：

- 1、发生时间
- 2、发生位置
- 3、严重程度，即 日志级别
- 4、内容

上述都是一条日志记录中可能包含的字段信息。还可能包括一些其他信息，如进程ID、进程名称、线程ID、线程名称等。

日志格式是用来定义一条日志记录中包含哪些字段的，而且日志格式通常是可以自定义的。

PS：输出一条日志时，日志内容、日志级别是需要开发人员明确指定的。对于其他字段信息，只需要是否显示在日志中即可。

### 5、怎么使用日志

> 在Python中，提供了一个用于记录日志，功能强大、使用简单的标准库模块`logging`。

## 二、logging基础介绍

> 在部署项目时，不可能直接将所有的信息都输出到控制台中，我们可以将这些信息记录到日志文件中，这样不仅方便我们查看程序运行时的情况，logging模块提供了在项目出现故障时根据运行时产生的日志快速定位问题出现位置的功能。

```python
import logging
```

### 1、logging库日志级别

问题思考：

- 开发人员在开发一个应用程序时需要什么日志信息？在程序正式上线后需要什么日志信息？
- 运维人员在部署开发环境时需要什么日志信息？在部署生产环境时需要什么日志信息？

**根据需求选择不同的日志级别**：

| 级别     | 级别数值        | 使用时机                                           |
| -------- | --------------- | -------------------------------------------------- |
| DEBUG    | 10 （级别最低） | 详细信息，常用于调试。                             |
| INFO     | 20（重要）      | 程序正常运行过程中产生的一些信息。                 |
| WARNING  | 30 （警告）     | 警告用户，虽然程序还在正常工作，但有可能发生错误。 |
| ERROR    | 40（错误）      | 由于更严重的问题，程序已经不能执行一些功能了。     |
| CRITICAL | 50 （严重）     | 严重错误，程序已经不能继续运行。                   |

默认情况是日志级别是warning，日志级别从上到下依次升高，即 `DEBUG < INFO < WARNING < ERROR < CRITICAL`。而日志的信息量是依次减少的。

开发应用程序 或部署开发环境时：应该使用DEBUG 或INFO级别的日志获取尽可能详细的日志信息来进行开发或部署调试；

应用上线 或部署生产环境时：应该使用`WARNING`或`CRITICAL`级别的日志来降低机器的I/O压力和提高获取错误日志信息的效率

PS：当为某个应用程序指定一个日志级别后，应用程序会记录所有日志级别大于或等于指定日志级别的日志信息，而不是仅仅记录指定级别的日志信息，只要级别大于或等于该指定日志级别的日志记录才会被输出，小于该级别的日志记录将会被丢弃。

### 2、logging工作流程

**Logger**：日志，暴露函数给应用程序，基于日志记录器和过滤器级别决定哪些日志有效。

**LogRecord** ：日志记录器，将日志传到相应的处理器处理。

**Handler** ：处理器, 将(日志记录器产生的)日志记录发送至合适的目的地。

**Filter** ：过滤器, 提供了更好的粒度控制,它可以决定输出哪些日志记录。

**Formatter**：格式化器, 指明了最终输出中日志记录的布局。
![在这里插入图片描述](https://img-blog.csdnimg.cn/485fa0a9082b434db4a3d1cf2c38681a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)
**流程描述**：

- 1、判断日志的等级是否大于 Logger 对象的等级，如果大于，则往下执行，否则，流程结束。
- 2、产生日志：第一步，判断是否有异常，如果有，则添加异常信息。 第二步，处理日志记录方法(如 debug，info 等)中的占位符，即一般的字符串格式化处理。
- 3、使用注册到 Logger 对象中的 Filters 进行过滤。如果有多个过滤器，则依次过滤；只要有一个过滤器返回假，则过滤结束，且该日志信息将丢弃，不再处理，而处理流程也至此结束。否则，处理流程往下执行。
- 4、在当前 Logger 对象中查找 Handlers，如果找不到任何 Handler，则往上到该 Logger 对象的父 Logger 中查找；如果找到一个或多个 Handler，则依次用 Handler 来处理日志信息。但在每个 Handler 处理日志信息过程中，会首先判断日志信息的等级是否大于该 Handler 的等级，如果大于，则往下执行(由 Logger 对象进入 Handler 对象中)，否则，处理流程结束。
- 5、执行 Handler 对象中的 filter 方法，该方法会依次执行注册到该 Handler 对象中的 Filter。如果有一个 Filter 判断该日志信息为假，则此后的所有 Filter 都不再执行，而直接将该日志信息丢弃，处理流程结束。
- 6、使用 Formatter 类格式化最终的输出结果。 注：Formatter 同上述第 2 步的字符串格式化不同，它会添加额外的信息，比如日志产生的时间，产生日志的源代码所在的源文件的路径等等。
- 7、真正地输出日志信息(到网络，文件，终端，邮件等)。至于输出到哪个目的地，由 Handler 的种类来决定。

### 3、日志输出格式

日志的输出格式可以认为设置，默认格式为下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d996daeb797549d7ad9a8b34264e54d9.png)

### 4、基本使用

logging 使用非常简单，使用 basicConfig() 方法就能满足基本的使用需要，如果方法没有传入参数，会根据默认的配置创建Logger 对象，默认的日志级别被设置为 WARNING，默认的日志输出格式如上图，该函数可选的参数如下表所示。

| 参数名称 | 参数描述                                                     |
| -------- | ------------------------------------------------------------ |
| filename | 日志输出到文件的文件名                                       |
| filemode | 文件模式，r[+]、w[+]、a[+]                                   |
| format   | 日志输出的格式                                               |
| datefat  | 日志附带日期时间的格式                                       |
| style    | 格式占位符，默认为 “%” 和 “{}”                               |
| level    | 设置日志输出级别                                             |
| stream   | 定义输出流，用来初始化 StreamHandler 对象，不能 filename 参数一起使用，否则会ValueError 异常 |
| handles  | 定义处理器，用来创建 Handler 对象，不能和 filename 、stream 参数一起使用，否则也会抛出 ValueError 异常 |

### 5、代码实战

#### 指定日志输出级别

- **默认日志级别输出**

  ```python
  import logging
  
  # 默认的日志输出级别为warning
  logging.debug('This is debug log')  # 调试级别
  logging.info('This is info log')  # 信息级别
  logging.warning('This is warning log')  # 警告级别
  logging.error('This is error log')  # 错误级别
  logging.critical('This is critical log')  # 严重错误级别
  ```

  输出结果如下：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/73b64310c72e4e29ae2cd00e11f91a5b.png)

- **使用`baseconfig()`来指定日志输出级别:**

  ```python
  import logging
  
  # 使用baseconfig()来指定日志输出级别
  logging.basicConfig(level=logging.DEBUG)
  
  logging.debug('This is debug log')  # 调试级别
  logging.info('This is info log')  # 信息级别
  logging.warning('This is warning log')  # 警告级别
  logging.error('This is error log')  # 错误级别
  logging.critical('This is critical log')  # 严重错误级别
  12345678910
  ```

  输出结果如下：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/1ee875c6f86247fe9b665fd6fbad79d8.png)

#### 输出日志到文件

```python
import logging

# 使用baseconfig()来指定日志输出级别，日志信息写入test.log文件中
logging.basicConfig(filename="test.log", level=logging.DEBUG)  

logging.debug('This is debug log')  # 调试级别
logging.info('This is info log')  # 信息级别
logging.warning('This is warning log')  # 警告级别
logging.error('This is error log')  # 错误级别
logging.critical('This is critical log')  # 严重错误级别
```

生成文件`test.log`，文件内容如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/88dff45af1f049eeab7dd7e10691146a.png)

**注意**：默认情况再执行程序会换行追加！如果想要每次执行程序，先清空文件，然后再输出日志到文件中，只需要新增一个属性`filemode="w"`文件模式（先清空在写入）

```python
logging.basicConfig(filename="test.log",filemode="w",level=logging.DEBUG)
```

#### 输出对象

```python
import logging

logging.basicConfig(level=logging.DEBUG)
name = "张三"
age = "10"
logging.debug("姓名:%s, 年龄:%s", name, age)
logging.debug("姓名:%s, 年龄:%s" % (name, age))
logging.debug(("姓名 {}，年龄 {}".format(name, age)))
logging.debug((f"姓名 {name}，年龄 {age}"))
```

输出结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/873dfe4e1f284be89b25efebf6f0ab38.png)

#### 添加重要信息

- `asctime`：当前时间
- `levelname`：输出日志级别
- `filename`：文件名称
- `lineno`：报错文件行号
- `message`：输出信息

```python
    import logging

    # 使用baseconfig()来指定日志输出级别，日志信息写入test.log文件中
    # asctime：日期，levelname：级别，message：报错信息，filename：文件名，lineno：报错行数；datefmt：格式化时间字符串；level：修改默认日志级别
    logging.basicConfig(filename="test.log", filemode="w",
                        format="%(asctime)s %(name)s:%(levelname)s:%(message)s|%(filename)s:%(lineno)s",
                        datefmt="%Y-%M-%d %H:%M:%S", level=logging.DEBUG)

    logging.debug('This is a debug message')
    logging.debug('This is debug log')  # 调试级别
    logging.info('This is info log')  # 信息级别
    logging.warning('This is warning log')  # 警告级别
    logging.error('This is error log')  # 错误级别
    logging.critical('This is critical log')  # 严重错误级别
```

生成文件`test.log`，文件内容如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b76c207e254447f59e93d70a3404f5d0.png)

## 三、logging高级应用

**logging模块采用了模块化设计，主要包含四种组件:**

| 类名       | 说明                                             |
| ---------- | ------------------------------------------------ |
| Loggers    | 记录器，提供应用程序代码能直接使用的`接口`。     |
| Handlers   | 处理器，将记录器产生的日志发送至目的地           |
| Filters    | 过滤器，提供更好的粒度控制，决定那些日志会被输出 |
| Formatters | 格式化器，设置日志内容的组成结构和消息字段       |

**logging模块的工作流程：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d3153a4233f4e1fa4f94d5bc812d76c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1、loggers记录器

- 1、提供应用程序的调用接口（logger是单例的）

  ```python
  logger  = logging.getLogger(__name__)
  ```

- 2、决定日志记录的级别

  ```python
  logger.setLevel()
  ```

- 3、将日志内容传递相关联的handlers中

  ```python
   logger.addHandler()和logger.removeHandler()
  ```

### 2、Handlers处理器

> 它们将日志分发到不同的目的地：文件、标准输出、邮件、或者通过socke、http等协议发送到任何地方。

- **SreamHandler**：标准输出stdout（如显示器）分发器。

  创建方法：`sh = logging.StreamHandler(stream=None)`

- **FileHandle**：将日志保存到磁盘文件的处理器

  创建方法:`fh=logging.FileHandler(filename,mode=“a”,encoding=Node,delay=False)`

  setFormatter()：设置当前handler对象使用的消息格式。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/0c3f68bdbf7c4c74908c2f305a8baefe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_13,color_FFFFFF,t_70,g_se,x_16)

### 3、Formatters 格式

> Formatter对象用来最终设置日志信息的顺序、结构和内容。

**构造方法**：`ft = logging.Formatter.__init__(fmt=None,datefmt=None,style='%')`

**datefmt默认格式为**：`%Y-%m-%d %H:%M:%S`；style参数默认为百分符`%`,这表示`%(<dictionary key>)s`格式的字符串

| 属性        | 格式            | 描述                                               |
| ----------- | --------------- | -------------------------------------------------- |
| asctime     | %(asctime)s     | 日志产生时间，默认格式为 `2021-11-23 10:20:30,123` |
| created     | %(created)f     | time.time()生成的日志创建时间戳                    |
| filename    | %(filename)s    | 生成日志的程序名                                   |
| funcName    | %(funcName)s    | 调用日志的函数名                                   |
| levelname   | %(levelname)s   | 日志级别(DEGBU、INFO、WARNING、ERROR、CRITICAL)    |
| levelno     | %(levelno)s     | 日志级别对应的数值                                 |
| lineno      | %(lineno)d      | 日志所针对的代码行号（报错代码所在行数）           |
| module      | %(module)s      | 生成日志的模块名                                   |
| msecs       | %(msecs)d       | 日志生成时间的毫秒部分                             |
| message     | %(message)s     | 具体的日志信息                                     |
| name        | %(name)s        | 日志调用者                                         |
| pathname    | %(pathname)s    | 生成日志的完整路径                                 |
| processName | %(processName)s | 进程名（如果可用）                                 |
| thread      | %(thread)d      | 生成日志的线程id（如果可用）                       |
| process     | %(process)d     | 生成日志的进程id                                   |
| threadName  | %(threadName)s  | 线程名（如果可用）                                 |

## 四、高级用法实战

### 1、记录器

- **默认情况：**

  ```python
  import logging
  
  logger = logging.getLogger()
  print(logger)
  print(type(logger))
  ```

  输出结果如下：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/030deb0ab924459fb2e8abc564c2eba2.png)

- **修改为指定级别**

  ```python
  import logging
  
  logger = logging.getLogger("applog")  # 修改的显示名称
  logger.setLevel(logging.DEBUG)  # 修改日志级别
  print(logger)
  print(type(logger))
  ```

  输出结果如下：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/02446aa12d784c5685dc39cba013bcee.png)

### 2、处理器

- 普通处理器定义

  ```python
  consoleHandler = logging.StreamHandler()
  consoleHandler.setLevel(logging.DEBUG)
  ```

- 写入文件处理器定义

  ```python
  # 写入文件，没有给handler指定日志级别，将使用logger的级别
  fileHandler = logging.FileHandler(filename="test.log")
  fileHandler.setLevel(logging.INFO)
  ```

- 设置formatter格式

  ```python
  # 给处理器设置formatter格式
  formatter = logging.Formatter("%(asctime)s %(name)s:%(levelname)s:%(message)s|%(filename)s:%(lineno)s")
  ```

- 处理器设置格式

  ```python
  # 给处理器设置格式
  consoleHandler.setFormatter(formatter)
  fileHandler.setFormatter(formatter)
  ```

- 记录器要设置处理器

  ```python
  logger.addHandler(consoleHandler)
  logger.addHandler(fileHandler)
  ```

完整代码：

```python
import logging

# 记录器
logger = logging.getLogger("applog")  # 修改的显示名称
logger.setLevel(logging.DEBUG)  # 修改日志级别

# 处理器
consoleHandler = logging.StreamHandler()
consoleHandler.setLevel(logging.DEBUG)

# 写入文件，没有给handler指定日志级别，将使用logger的级别
fileHandler = logging.FileHandler(filename="test.log")
fileHandler.setLevel(logging.INFO)

# 给处理器设置formatter格式; 8s：表示s前面的占8位；-8s：左对齐
formatter = logging.Formatter("%(asctime)s %(name)s:%(levelname)8s:%(message)s|%(filename)s:%(lineno)s")

# 给处理器设置格式
consoleHandler.setFormatter(formatter)
fileHandler.setFormatter(formatter)

# 记录器要设置处理器
logger.addHandler(consoleHandler)
logger.addHandler(fileHandler)

# 打印日志的代码
logger.debug('This is debug log')  # 调试级别
logger.info('This is info log')  # 信息级别
logger.warning('This is warning log')  # 警告级别
logger.error('This is error log')  # 错误级别
logger.critical('This is critical log')  # 严重错误级别
```

**输出结果：**

- 控制台
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/ba0f79693aab4fb78b3011c491ef8e13.png)
- `test.log`文件中
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/fbe8495582fd4eca83fb0b1fbd484b36.png)

### 3、过滤器

```python
#定义过滤器；过滤开头名称为cn.cccb的文件
fit = logging.Filter("cn.cccb")
#关联过滤器
logger.addFilter(fit)
```

**输出结果**：

- 控制台无数据
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/9661a0de7e9b4af89db0fc63e53a3654.png)
- `test.log`文件中无数据
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/b78c09e432c64aabbf4c66f83528dd84.png)

**想要正常显示，需要修改记录器的自定义名称**：

```python
logger = logging.getLogger("cn.cccb.log")
```

**输出结果如下**：

- 控制台
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/142d1460768a426f90f71d8b5f8f3f9a.png)
- `test.log`文件
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/a94895a0cb284ac0989b409ef9105866.png)

**当然我们在设置过滤器的时候，我们也可以指定给我们的某个handler处理器指定过滤器来过滤，我们修改代码完整如下：**

```python
import logging

# 记录器
logger = logging.getLogger("apploglog")  # 修改的显示名称
logger.setLevel(logging.DEBUG)  # 修改日志级别

# 处理器
consoleHandler = logging.StreamHandler()
consoleHandler.setLevel(logging.DEBUG)

# 写入文件，没有给handler指定日志级别，将使用logger的级别
fileHandler = logging.FileHandler(filename="test.log")
fileHandler.setLevel(logging.INFO)

# 给处理器设置formatter格式; 8s：表示s前面的占8位
formatter = logging.Formatter("%(asctime)s %(name)s:%(levelname)8s:%(message)s|%(filename)s:%(lineno)s")

# 给处理器设置格式
consoleHandler.setFormatter(formatter)
fileHandler.setFormatter(formatter)

# 记录器要设置处理器
logger.addHandler(consoleHandler)
logger.addHandler(fileHandler)

# 定义过滤器；过滤开头名称为cn.cccb的文件
fit = logging.Filter("cn.cccb")

# 关联过滤器
# logger.addFilter(fit)
fileHandler.addFilter(fit)

# 打印日志的代码
logger.debug('This is debug log')  # 调试级别
logger.info('This is info log')  # 信息级别
logger.warning('This is warning log')  # 警告级别
logger.error('This is error log')  # 错误级别
logger.critical('This is critical log')  # 严重错误级别
1234567891011121314151617181920212223242526272829303132333435363738
```

**输出结果**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fe10c42e38c94a90bd2b69931d8e6401.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2cec64a5c1a4dcfa771e5c7d6e9e033.png)
以上的一些方式，在我们日常使用的简单过程中，我们这样定义就可以了，但是如果我们在大型的项目过程中，如果去定义的话，那么我们就不能这样去做，我们总不能修改个日志等级的话，我们都需要去修改下代码吧？

于是我们采用了和java中日志记录相关的方式，我们将这些配置记录在我们的配置文件中，这样就很方便我们去进行修改了。

### 4、配置文件定义

新建一个`logging.conf`文件，新增如下内容（复制代码进去）：

```python
[loggers]
keys=root,applog

[handlers]
keys=fileHandler,consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_applog]
level=DEBUG
handlers=fileHandler,consoleHandler
qualname=applog
propagate=0


[handler_consoleHandler]
class=StreamHandler
args=(sys.stdout,)
level=DEBUG
formatter = simpleFormatter

[handler_fileHandler]
class=handlers.TimedRotatingFileHandler
args=("applog.log","midnight",1,0)
level=DEBUG
formatter = simpleFormatter

[formatter_simpleFormatter]
format=%(asctime)s|%(levelname)8s|%(name)s:%(lineno)s|%(message)s|%(processName)s
datafm=%Y-%m-%d %H:%M:%S
```

**代码解读**：

- `[loggers]、[handlers]、[formatters]`：定义记录器器、处理器和格式化
- `[logger_root]`：为root的日志记录器进行属性设置
- `[logger_applog]`：为applog的日志记录器进行属性设置；`handlers`：绑定的处理器对象；`qualname=applog` ：表示代码读取配置文件的时候，必须使用`getLogger(”applog“)`的方式才能获取到这个对象
- `[handler_consoleHandler]`：`class=StreamHandler`表示定义的`StreaHandler`对象；`args=(sys.stdout,)` ：标准输出流（会输出到控制台）
- `[handler_fileHandler]`：`class=handlers.TimedRotatingFileHandler`：大型成产模式中最常用，以时间滚动来生成文件；`args=("applog.log","midnight",1,0)`：规定文件名称，每天凌晨12点后备份一份数据，1代表的是1秒钟之后开始备份，0表示备份几份
- `[formatter_simpleFormatter]`：`format`：输出内容格式；`datafm`：输出时间格式

### 5、代码异常记录日志

```python
import logging.config

# 使用配置文件的方式来处理日志
logging.config.fileConfig('logging.conf')

rootLogger = logging.getLogger()
rootLogger.debug("This is root Logger, debug")

# 记录器
logger = logging.getLogger("applog")  # 修改的显示名称
# 打印日志的代码
logger.debug('This is debug log，bug')  # 调试级别

# 设置报错将str转换成int
```

输出结果（此时我们直接转换会出错，但是在我们的日志文件并不打印）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3c521dbdfde4816972321f32b05b06a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/798599924797471f925ab6c23c63e1af.png)

**使用`try...catch`捕捉异常处理就可以输入到日志文件中：**

```python
a = "abc"
try:
    int(a)
except Exception as e:
    logger.exception(e)
```

输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1bf4519550f84ed398c00446e87af504.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/15808994309a48aa8a19a73c0c343219.png)

我们发现不仅是在控制台上，在文件中也进行了具体的输出。此时我们全部讲完了logging标准库。