[toc]

## 一、random库介绍

> random模块实现了各种分布的`伪随机数`生成器。为什么称为伪随机数：即人类使用算法等方式，以一个基准（也被叫做种子,最常用的就是时间戳）来构造一系列数字，这些数字的特性符合人们所理解的随机数。但因为是通过算法得到的，所以一旦算法和种子都确定，那么产生的随机数序列也是确定的，所以叫伪随机数。

```python
import random
```

## 二、常用函数

### 1、random.seed(a)

> 设置初始化随机种子，可输出相同随机数序列；a取整数或浮点数，不设置时默认以系统时间为种子

```python
import random

print("没有设定种子时")
for i in range(5):
    ret = random.randint(1, 10)
    print(ret, end=" ")
print()

print("设定种子时")
random.seed(1)
for i in range(5):
    ret = random.randint(1, 10)
    print(ret, end=" ")
12345678910111213
```

输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/98b2adadf1624cc289bdf0db67ae2ee8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)
如上图可以看出：**没有显式设定种子时，每次输出的随机数都是不一样的；显式设定种子时，每次输出的随机数都是一样的**

## random.random()

> 用于生成一个 0.0 到 1.0 的随机浮点数

```python
>>> import random
>>> random.random()
0.9279749775408933
>>> random.random()
0.12720379394341363
>>> random.random()
0.9391670189485866
1234567
```

## random.uniform(a,b)

> 生成一个`[a,b]`之间的随机小数；a, b 取整数 或 浮点数

```python
>>> import random
>>> random.uniform(10.0, 20.0)
10.839441969258752
>>> random.uniform(10.0, 20.0)
12.233491150445115
>>> random.uniform(10, 20)
11.290566243261305
1234567
```

## random.randint(a,b)

> 生成一个`[a,b]`之间的随机整数

```python
>>> import random
>>> random.randint(10,100)
100
>>> random.randint(10,100)
83
>>> random.randint(10,100)
66
1234567
```

## random.randrange(start,stop,[step])

> 生成一个`[start,stop)`之间以step为步数的随机整数；start,stop,step取整数，step不设时默认值为1

随机生成1-100的整数：

```python
>>> import random
>>> random.randrange(1,100)
54
>>> random.randrange(1,100)
21
>>> random.randrange(1,100)
71
1234567
```

随机生成1-100的奇数：

```python
>>> import random
>>> random.randrange(1,100,2)
37
>>> random.randrange(1,100,2)
63
>>> random.randrange(1,100,2)
29
1234567
```

随机生成1-100的偶数：

```python
>>> import random
>>> random.randrange(2,100,2)
62
>>> random.randrange(2,100,2)
6
>>> random.randrange(2,100,2)
46
1234567
```

## random.getrandbits(k)

> 生成一个`占内存k位`以内的随机整数；k取长度的整数值

```python
>>> import random
>>> random.getrandbits(10)
29
>>> random.getrandbits(10)
540
>>> random.getrandbits(10)
227
1234567
```

## random.choice(seq)

> 从序列类型seq中随机返回一个元素；seq取序列类型：如字符串，列表，元组

```python
>>> import random
>>> list = ['a', 'b', 'c', 'd', 'f', 'g']
>>> random.choice(list)
'b'
>>> random.choice(list)
'f'
>>> random.choice(list)
'g'
12345678
```

## random.shuffle(seq)

> 将序列类型中元素随机排序，返回打乱后序列，seq被改变（`改变原列表`），shuffle为洗牌之意； seq取序列类型：如字符串，列表，元组

```python
>>> import random
>>> list = ['a', 'b', 'c', 'd', 'f', 'g']
>>> random.shuffle(list)
>>> list
['c', 'a', 'f', 'd', 'g', 'b']
>>> random.shuffle(list)
>>> list
['f', 'a', 'b', 'c', 'g', 'd']
>>> random.shuffle(list)
>>> list
['a', 'd', 'g', 'c', 'b', 'f']
1234567891011
```

## random.sample(pop,k)

> 从pop中选取k个元素，以列表类型返回（`不改变原列表`）；pop取序列类型，k取整数：代表选取个数

```python
>>> import random
>>> list = ['a', 'b', 'c', 'd', 'f', 'g']
>>> random.sample(list, 4)
['b', 'f', 'c', 'a']
>>> random.sample(list, 4)
['g', 'f', 'b', 'd']
>>> random.sample(list, 4)
['g', 'f', 'c', 'b']
12345678
```

# 三、不常用函数

## random.getstate()

> 捕获并返回生成器当前内部状态的对象，可以把它传递给setstate()函数来把生成器内部状态恢复到调用getstate()函数之前的状态。相当于备份。

## random.setstate(state)

> state应为getstate()函数的结果，用来把生成器当前的内部状态恢复为state。

## random.betavariate(alpha, beta)

> Beta分布：参数的条件是 alpha > 0 和 beta > 0， 返回值的范围介于 0 和 1 之间。

## random.expovariate(lambd)

> 指数分布

## random.gammavariate(alpha, beta)

> Gamma分布：参数的条件是 alpha > 0 和 beta > 0

## random.gauss(mu, sigma)

> 高斯分布：mu是平均值，sigma是标准差。

## random.normalvariate(mu, sigma)

> 正态分布：mu是平均值，sigma是标准差。

## random.paretovariate(alpha)

> 帕累托分布：alpha是形状参数。

## random.weibullvariate(alpha,beta)

> 威布尔分布：alpha是比例参数，beta是形状参数。

# 四、真实案例

## 随机密码字符串

> 字符串包括数字和字母， 可以指定密码的位数

```python
import random
import string


def get_random_string(length):
    # 随机生成字母和数字的位数
    num_count = random.randint(1, length - 1)
    letter_count = length - num_count

    # 随机抽样生成数字序列
    num_list = [random.choice(string.digits) for _ in range(num_count)]

    # 随机抽样生成字母序列
    letter_list = [random.choice(string.ascii_letters) for _ in range(letter_count)]

    # 合并字母和数字
    all_list = num_list + letter_list

    # 乱序
    random.shuffle(all_list)

    result = "".join([i for i in all_list])
    return result


# 生成10位的密码
password1 = get_random_string(10)
print(password1)
# 生成15位的密码
password2 = get_random_string(15)
print(password2)
# 生成20位的密码
password3 = get_random_string(20)
print(password3)
12345678910111213141516171819202122232425262728293031323334
```

输出结果：

```python
41eD76F3e1
915087432k8443z
002L5292840A07284755
123
```

## 计算圆周率

**1）圆周率的近似计算公式：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/daefcb40c41148ef8fb87b08b79cadce.png)

```python
pi = 0
N = 100
for k in range(N):
    pi += 1 / pow(16, k) * (4 / (8 * k + 1) - 2 / (8 * k + 4) - 1 / (8 * k + 5) - 1 / (8 * k + 6))
print("圆周率值是：%s" % pi)
12345
```

输出结果：

```python
圆周率值是: 3.141592653589793
1
```

**2）蒙特卡洛算法：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/5444171e84694850b09187ecc0eb3f96.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP6KKBSVRTdXBlcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
import random

DARTS = 1000 * 1000 * 10
hits = 0.0

for i in range(1, DARTS + 1):
    x, y = random.random(), random.random()
    dist = pow(x ** 2 + y ** 2, 0.5)
    if dist <= 1.0:
        hits = hits + 1
pi = 4 * (hits / DARTS)
print("圆周率值是：%s" % pi)
123456789101112
```

输出结果：

```python
圆周率值是：3.14205
```