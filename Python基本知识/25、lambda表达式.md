[toc]

## 一、lambda函数介绍

有时在使用函数时不需要给函数分配一个名称，该函数就是 “匿名函数”。在Python中使用lambda[表达式表示**匿名函数**，声明 lambda表达式的语法如下:

```python
lambda 参数列表: lambda体
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8cc0d132d36349bcb2a67f891f02384e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_14,color_FFFFFF,t_70,g_se,x_16)

lambda是关键字声明，在lambda表达式中，“参数列表”与函数中的参数列表是一样的， 但不需要用小括号括起来，冒号后面是lambda体，lambda 表达式的主要代码在lambda体处编写，类似于函数体。

`提示`：lambda体不能是一个代码块，不能包含多条语句，只能包含一条语句，该语句会计算一个结果返回给 lambda表达式，但与函数不同的是，不需要使用return语句返回。而且当使用函数作为参数的时候，lambda表达式非常有用，可以让代码简单，简洁。

## 二、lambda函数与 def 函数的区别

- 1、lambda可以立刻传递(无需变量) ，自动返回结果;
- 2、lambda在内部只能包含一行代码 ;
- 3、lambda是一个为编写简单的函数而设计的，而 def 用来处理更大的任务;
- 4、lambda可以定义一个匿名函数，而 def 定义的函数必须有一个名字。

**lambda函数的优势**：

- 1、对于单行函数,使用 lambda表达式可以省去定义函数的过程,让代码更加简洁;
- 2、对于不需要多次复用的函数，使用 lambda表达式可以在用完之后立即释放,提高程序执行的性能。

## 三、lambda函数案例

### 3.1 定义加法函数

```python
# def函数写法
def add(a, b):
    return a + b


print(add(10, 20))
print("----------这是一个分割线----------")

# lambda函数写法
add_lambda = lambda a, b: a + b
print(add_lambda(10, 20))
```

输出结果：

```python
30
----------这是一个分割线----------
30
```

### 3.2 使用if判断奇偶性

```python
# def函数写法
def get_odd_even(x):
    if x % 2 == 0:
        return "偶数"
    else:
        return "奇数"


print(get_odd_even(10))
print("----------这是一个分割线----------")

# lambda函数写法
get_odd_even1 = lambda x: "偶数" if x % 2 == 0 else "奇数"
print(get_odd_even1(10))
print(get_odd_even1(9))
```

输出结果：

```python
偶数
----------这是一个分割线----------
偶数
奇数
```

### 3.3 无参数表达式

```python
# def函数写法
def test1():
    return "Python YYDS！！！"


print(test1())
print("----------这是一个分割线----------")

# lambda函数写法
test2 = lambda: "Python YYDS！！！"
print(test2())
```

输出结果：

```python
Python YYDS！！！
----------这是一个分割线----------
Python YYDS！！！
```

### 3.4 列表排序

```python
# 根据元组中第一个数排序
a = [(2, "小黑"), (5, "小白"), (1, "张三"), (4, "李四"), (3, "王五")]
a.sort(key=lambda x: x[0])

print(a)
12345
```

输出结果：

```python
[(1, '张三'), (2, '小黑'), (3, '王五'), (4, '李四'), (5, '小白')]
1
```

上面例子中，根据列表第一个元素对原本打乱的数据进行了排序，`元组、字典同理都可以这样使用！`

## 3.5 map方法混搭（常用）

遍历序列，对序列中每个元素进行操作，最终获得新的序列
![在这里插入图片描述](https://img-blog.csdnimg.cn/3da4edc886b54db69712a813b0f1133d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
# def函数写法
def add(num):
    return num ** 2


x = map(add, [1, 2, 3, 4, 5])
print(list(x))
print("----------这是一个分割线----------")

# lambda函数写法
y = map(lambda num: num ** 2, [1, 2, 3, 4, 5])
print(list(y))
123456789101112
```

输出结果：

```python
[1, 4, 9, 16, 25]
----------这是一个分割线----------
[1, 4, 9, 16, 25]
123
```

上面例子中，map函数第一个参数是一个lambda表达式，输入一个参数，返回元素的平方。第二个就是需要作用的对象，此处是一个列表。Python3中map返回一个map对象，我们需要人工转为list，得到的结果就是[1, 4, 9, 16, 25]

## 3.6 filter方法混搭（常用）

对序列中的元素进行筛选，最终获得符合条件的序列
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5186e2db71b4e8399bcc892b81b0baa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
x = filter(lambda num: num % 2 == 0, range(10))
print(list(x))

# [0, 2, 4, 6, 8]`
1234
```

上面例子中，定义lambda表达式，筛选为偶数的元素，结果为[0, 2, 4, 6, 8]。

## 3.7 [reduce](https://so.csdn.net/so/search?q=reduce&spm=1001.2101.3001.7020)方法混搭（常用）

对序列内所有元素进行累计操作；Python3中删掉了全局的reduce函数，需要从functools引入
![在这里插入图片描述](https://img-blog.csdnimg.cn/a666754e54184a12bc474c513a892a97.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
from functools import reduce

list1 = [1, 2, 3, 4, 5]
list2 = reduce(lambda x, y: x + y, list1)
print(list2)  # 15
12345
```

reduce 中使用的 lambda表达式需要两个参数，reduce函数共三个参数，第一个是就是lambda表达式，第二个是要累计的序列，第三个是初始值，我们没给初始值，那么开始操作的两个元素就是序列的前两个。否则将使用我们给出的初始值和序列第一个元素操作，然后结果再与第三个元素操作，以此类推。上个例子结果是15

# 四、lambda函数总结

1、lambda只是一个表达式，函数体比def简单很多

2、lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去

3、lambda函数拥有自己的名字空间，且不能访问自有参数列表之外或全局名字空间里的参数

4、 简单单行代码或者一次性的函数可以用lambda函数来书写，可以让代码更简洁。

5、 对于复杂函数或者函数体体量大的函数，最好不要用lambda函数，会增加代码的阅读难度，使代码晦涩难懂。

6、 在非多次调用的函数的情况下，lambda表达式即用既得，提高性能