[toc]

### 1.1 with语句的作用

> with语句可以自动管理上下文资源，不论什么原因跳出with块，**都能确保文件正确的关闭**，以此来达到释放资源的目的
> 简单来说就是不用手动写`close()`关闭，自动释放资源

**常规操作文件代码**：

```python
# 1. 打开 - 文件名需要注意大小写
file = open("README.txt")

# 2. 读取
text = file.read()
print(text)

# 3. 关闭
file.close()
```

- 注意：必须得有`file.close()`关闭文件

**with语法的代码**：

```python
with open("README.txt") as f:   
    data= f.read()
    print(data)
```

- 可以看出 with语句 更加简洁方便，也不用担心没写关闭造成资源浪费

### 1.2 with语句工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/64395400b95841cdad4e6b472d998bd2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)
**定义原则**：是with所求值的对象必须有一个`__enter__()`方法，一个`__exit__()`方法。

**执行步骤**：

- 1.紧跟with后面的语句被求值后，返回对象的`__enter__()`方法被调用，这个方法的返回值将被赋值给as后面的变量。
- 2.当with后面的代码块全部被执行完之后，将调用前面返回对象的`__exit__()`方法。

**定义一个类具体说明with工作如何工作**：

- 实现了特殊方法`__enter__()`,`__exit()`称为该类对象遵守了上下文管理器协议
- 该类对象的实例对象，称为上下文管理器

```python
class MyContentMgr(object):
    def __enter__(self):
        print("enter方法被调用执行了")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit方法被调用执行了")

    def show(self):
        print("show方法被调用执行了")


with MyContentMgr() as file:
    file.show()
```

运行代码，输出结果如下：

```python
enter方法被调用执行了
show方法被调用执行了
exit方法被调用执行了
```

- 可以发现在`show`方法结束后自动调用了对象的`__exit__()`方法。

### 1.3 with语句自动处理异常

**with真正强大之处是它可以处理异常**。可能你已经注意到 MyContentMgr 类的`__exit__`方法有三个参数`exc_type, exc_val, exc_tb`。 这些参数在异常处理中相当有用。我们来改一下代码，看看具体如何工作的。

**修改代码使show方法报错**：

```python
class MyContentMgr(object):
    def __enter__(self):
        print("enter方法被调用执行了")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit方法被调用执行了")

    def show(self):
        print("show方法被调用执行了", 1 / 0)  # 这加上一个报错操作


with MyContentMgr() as file:
    file.show()
```

运行代码，输出结果如下：

```python
enter方法被调用执行了
exit方法被调用执行了
Traceback (most recent call last):
  File "E:/Python_demo/1.py", line 14, in <module>
    file.show()
  File "E:/Python_demo/1.py", line 10, in show
    print("show方法被调用执行了", 1 / 0)  # 这加上一个报错操作
ZeroDivisionError: division by zero
```

- 可以看出即使show方法报错依然不影响`__exit__`方法自动调用

因此，Python的with语句是提供一个有效的机制，让代码更简练，同时在异常产生时，清理工作更简单。