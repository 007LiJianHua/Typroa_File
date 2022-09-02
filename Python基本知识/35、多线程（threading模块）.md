[toc]

## 一、多任务介绍

现实生活中，有很多的场景中的事情是同时进行的，比如开车的时候 手和脚共同来驾驶汽车，再比如唱歌跳舞也是同时进行的；

试想，如果把唱歌和跳舞这2件事情分开依次完成的话，估计就没有那么好的效果了（想一下场景：先唱歌，然后在跳舞，O(∩_∩)O哈哈~）
程序中

如下程序，来模拟“唱歌跳舞”这件事情

```python
#coding=utf-8

from time import sleep

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    sing() #唱歌
    dance() #跳舞
```

运行结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/79845b4eef6c4261bff1d6976cbe7047.gif#pic_center)

**注意**：

- 很显然刚刚的程序并没有完成唱歌和跳舞同时进行的要求
- 如果想要实现“唱歌跳舞”同时进行，那么就需要一个新的方法，叫做：**多任务**

## 二、多任务的概念

什么叫“多任务”呢？简单地说，就是操作系统可以同时运行多个任务。打个比方，你**一边在用浏览器上网**，**一边在听MP3**，**一边在用Word赶作业**，这就是多任务，至少同时有3个任务正在运行。还有很多任务悄悄地在后台同时运行着，只是桌面上没有显示而已。

![在这里插入图片描述](https://img-blog.csdnimg.cn/abdfe536da574ee0897b1e0c2b0d6fd3.png)

现在，多核**CPU**已经非常普及了，但是，即使过去的单核CPU，也可以执行多任务。由于CPU执行代码都是顺序执行的，那么，单核CPU是怎么执行多任务的呢？

答案就是**操作系统轮流让各个任务交替执行**，任务1执行0.01秒，切换到任务2，任务2执行0.01秒，再切换到任务3，执行0.01秒……这样反复执行下去。表面上看，每个任务都是交替执行的，但是，由于CPU的执行速度实在是太快了，我们感觉就像所有任务都在同时执行一样。

真正的并行执行多任务只能在多核CPU上实现，但是，由于任务数量远远多于CPU的核心数量，所以，操作系统也会自动把很多任务轮流调度到每个核心上执行。

**注意**：

- 并发：指的是任务数多余cpu核数，通过操作系统的各种任务调度算法，实现用多个任务“一起”执行（实际上总有一些任务不在执行，因为切换任务的速度相当快，看上去一起执行而已）
  - 两个或者多个事件，在同一个时间间隔内发生
    - 例如在1s内
      - 0~15ms：任务一
      - 15~30ms：任务二
      - 30~45ms：任务三
      - 45~60ms：任务四
    - 1s的时间间隔内，宏观上4个任务同时执行，微观上，四个任务分时间，交替运行。                                                                                                                                                                                                                                                                                   
- 并行：指的是任务数小于等于cpu核数，即任务真的是一起执行的

## 三、threading 模块介绍

> Python的thread模块是比较底层的模块，python的`threading模块`是对thread做了一些包装的，可以更加方便的被使用

threading模块常用方法如下：

| 方法名                       | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `threading.active_count()`   | 返回当前处于active状态的Thread对象                           |
| `threading.current_thread()` | 返回当前Thread对象                                           |
| `threading.get_ident()`      | 返回当前线程的线程标识符。线程标识符是一个非负整数，并无特殊含义，只是用来标识线程，该整数可能会被循环利用。Python3.3及以后版本支持该方法 |
| `threading.enumerate()`      | 返回当前处于active状态的所有Thread对象列表                   |
| `threading.main_thread()`    | 返回主线程对象，即启动Python解释器的线程对象。Python3.4及以后版本支持该方法 |
| `threading.stack_size()`     | 返回创建线程时使用的栈的大小，如果指定size参数，则用来指定后续创建的线程使用的栈大小，size必须是0(表示使用系统默认值)或大于32K的正整数 |

### 1. Thread类使用说明

> threading模块提供了Thread、Lock、RLock、Condition、Event、Timer和Semaphore等类来支持[多线程](https://so.csdn.net/so/search?q=多线程&spm=1001.2101.3001.7020)，Thread是其中最重要也是最基本的一个类，可以通过该类创建线程并控制线程的运行。

**使用Thread创建线程的方法**：

- 1、为构造函数传递一个可调用对象
- 2、继承Thread类并在子类中重写_`_init__()`和`run()`方法

**语法格式**：`threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)`

**参数说明**：

- `group`：通常默认即可，作为日后扩展 ThreadGroup 类实现而保留。
- `target`：用于 run() 方法调用的可调用对象，默认为 None。
- `name`：线程名称，默认是 Thread-N 格式构成的唯一名称，其中 N 是十进制数。
- `args`：用于调用目标函数的参数元组，默认为 ()。
- `kwargs`：用于调用目标函数的关键字参数字典，默认为 {}。
- `daemon`：设置线程是否为守护模式，默认为 None。

**线程对象 threading.Thread 的方法和属性**：

| 方法名                                                       | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `start()`                                                    | 启动线程。                                                   |
| `run()`                                                      | 线程代码，用来实现线程的功能与业务逻辑，可以在子类中重写该方法来自定义线程的行为 |
| `init(self, group=None, target=None, name=None, args=(), kwargs=None, daemon=None)` | 构造函数                                                     |
| `is_alive()`                                                 | 判断线程是否存活                                             |
| `getName()`                                                  | 返回线程名                                                   |
| `setName()`                                                  | 设置线程名                                                   |
| `isDaemon()`                                                 | 判断线程是否为守护线程                                       |
| `setDaemon()`                                                | 设置线程是否为守护线程                                       |
| `name`                                                       | 用来读取或设置线程的名字                                     |
| `ident`                                                      | 线程标识，用非0数字或None(线程未被启动)                      |
| `daemon`                                                     | 表示线程是否为守护线程，默认为False                          |
| `join(timeout=None)`                                         | 当 timeout 为 None 时，会等待至线程结束；当 timeout 不为 None 时，会等待至 timeout 时间结束，单位为秒。 |

### 2. 实例化 threading.Thread（重点） 

**1）单线程执行**

```python
import time


def saySorry():
    print("亲爱的，我错了，我能吃饭了吗？")
    time.sleep(1)


if __name__ == "__main__":
    for i in range(5):
        saySorry()
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fe342c3c7088423391244a36a5c25239.gif#pic_center)

**2）使用threading模块**:

```python
#coding=utf-8
import threading
import time

def saySorry():
    print("亲爱的，我错了，我能吃饭了吗？")
    time.sleep(1)

if __name__ == "__main__":
    for i in range(5):
        t = threading.Thread(target=saySorry)
        t.start() #启动线程，即让线程开始执行
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/31532e1fb95d49cfbd6a17e2d92647db.gif#pic_center)

**说明**：

- 可以明显看出使用了多线程并发的操作，花费时间要短很多
- 当调用`start()`时，才会真正的创建线程，并且开始执行

**3）主线程会等待所有的子线程结束后才结束**

```python
#coding=utf-8
import threading
from time import sleep,ctime

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    print('---开始---:%s'%ctime())

    t1 = threading.Thread(target=sing)
    t2 = threading.Thread(target=dance)

    t1.start()
    t2.start()

    #sleep(5) # 屏蔽此行代码，试试看，程序是否会立马结束？
    print('---结束---:%s'%ctime())
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e7895cd238b44d83a591ec2050e65bf1.gif#pic_center)

**4）查看线程数量**

```python
import threading
from time import sleep,ctime

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    print('---开始---:%s'%ctime())

    t1 = threading.Thread(target=sing)
    t2 = threading.Thread(target=dance)

    t1.start()
    t2.start()

    while True:
        length = len(threading.enumerate())
        print('当前运行的线程数为：%d'%length)
        if length<=1:
            break

        sleep(0.5)
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2aa4c22f60c44b2e80a546d8241de48d.gif#pic_center)

### 3. 继承 threading.Thread

**1）线程执行代码的封装**

> 通过上一小节，能够看出，通过使用threading模块能完成多任务的程序开发，为了让每个线程的封装性更完美，所以使用threading模块时，往往会定义一个新的子类class，只要继承`threading.Thread`就可以了，然后重写`run`方法

示例如下：

```python
#coding=utf-8
import threading
import time

class MyThread(threading.Thread):
    def run(self):
        for i in range(3):
            time.sleep(1)
            msg = "I'm "+self.name+' @ '+str(i) #name属性中保存的是当前线程的名字
            print(msg)


if __name__ == '__main__':
    t = MyThread()
    t.start()
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4337ba4b55874e7487ab2984f4c02ac6.gif#pic_center)

**说明**：

> python的`threading.Thread`类有一个`run`方法，用于定义线程的功能函数，可以在自己的线程类中覆盖该方法。而创建自己的线程实例后，通过Thread类的start方法，可以启动该线程，交给python虚拟机进行调度，当该线程获得执行的机会时，就会调用run方法执行线程。

**2）线程的执行顺序**

```python
    #coding=utf-8
    import threading
    import time

    class MyThread(threading.Thread):
        def run(self):
            for i in range(3):
                time.sleep(1)
                msg = "I'm "+self.name+' @ '+str(i)
                print(msg)
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()
```

执行结果：(运行的结果可能不一样，但是大体是一致的)

```python
I'm Thread-1 @ 0
I'm Thread-2 @ 0
I'm Thread-5 @ 0
I'm Thread-3 @ 0
I'm Thread-4 @ 0
I'm Thread-3 @ 1
I'm Thread-4 @ 1
I'm Thread-5 @ 1
I'm Thread-1 @ 1
I'm Thread-2 @ 1
I'm Thread-4 @ 2
I'm Thread-5 @ 2
I'm Thread-2 @ 2
I'm Thread-1 @ 2
I'm Thread-3 @ 2
```

**说明**：

> 从代码和执行结果我们可以看出，多线程程序的执行顺序是不确定的。当执行到sleep语句时，线程将被阻塞（Blocked），到sleep结束后，线程进入就绪（Runnable）状态，等待调度。而线程调度将自行选择一个线程执行。上面的代码中只能保证每个线程都运行完整个run函数，但是线程的启动顺序、run函数中每次循环的执行顺序都不能确定。

**3）总结**

- 1、每个线程默认有一个名字，尽管上面的例子中没有指定线程对象的name，但是python会自动为线程指定一个名字。
- 2、当线程的run()方法结束时该线程完成。
- 3、无法控制线程调度程序，但可以通过别的方式来影响线程调度的方式。

### 4. 多线程 - 共享全局变量（重点）

```python
from threading import Thread
import time

g_num = 100

def work1():
    global g_num
    for i in range(3):
        g_num += 1

    print("----in work1, g_num is %d---"%g_num)


def work2():
    global g_num
    print("----in work2, g_num is %d---"%g_num)


print("---线程创建之前g_num is %d---"%g_num)

t1 = Thread(target=work1)
t1.start()

#延时一会，保证t1线程中的事情做完
time.sleep(1)

t2 = Thread(target=work2)
t2.start()
```

运行结果:

```python
---线程创建之前g_num is 100---
----in work1, g_num is 103---
----in work2, g_num is 103---
```

**列表当做实参传递到线程中**：

```python
from threading import Thread
import time

def work1(nums):
    nums.append(44)
    print("----in work1---",nums)


def work2(nums):
    #延时一会，保证t1线程中的事情做完
    time.sleep(1)
    print("----in work2---",nums)

g_nums = [11,22,33]

t1 = Thread(target=work1, args=(g_nums,))
t1.start()

t2 = Thread(target=work2, args=(g_nums,))
t2.start()=
```

运行结果:

```python
----in work1--- [11, 22, 33, 44]
----in work2--- [11, 22, 33, 44]
```

**总结**：

- **在一个进程内的所有线程共享全局变量，很方便在多个线程间共享数据**
- 缺点就是，线程是对全局变量随意遂改**可能造成多线程之间对全局变量的混乱**（即线程非安全）

### 5. 多线程-共享全局变量问题

**多线程开发可能遇到的问题**：假设两个线程 t1 和 t2 都要对全局变量 g_num(默认是0) 进行加1运算，t1 和 t2 都各对g_num 加 10 次，g_num的最终的结果应该为20。

但是由于是多线程同时操作，有可能出现下面情况：

- 1、在g_num=0时，t1取得g_num=0。此时系统把t1调度为”sleeping”状态，把t2转换为”running”状态，t2也获得g_num=0
- 2、然后t2对得到的值进行加1并赋给g_num，使得g_num=1
- 3、然后系统又把t2调度为”sleeping”，把t1转为”running”。线程t1又把它之前得到的0加1后赋值给g_num。
- 4、这样导致虽然t1和t2都对g_num加1，但结果仍然是g_num=1

**测试1：**

```python
import threading
import time

g_num = 0

def work1(num):
    global g_num
    for i in range(num):
        g_num += 1
    print("----in work1, g_num is %d---"%g_num)


def work2(num):
    global g_num
    for i in range(num):
        g_num += 1
    print("----in work2, g_num is %d---"%g_num)


print("---线程创建之前g_num is %d---"%g_num)

t1 = threading.Thread(target=work1, args=(100,))
t1.start()

t2 = threading.Thread(target=work2, args=(100,))
t2.start()

while len(threading.enumerate()) != 1:
    time.sleep(1)

print("2个线程对同一个全局变量操作之后的最终结果是:%s" % g_num)
```

运行结果：

```python
---线程创建之前g_num is 0---
----in work1, g_num is 100---
----in work2, g_num is 200---
2个线程对同一个全局变量操作之后的最终结果是:200
```

**测试2：**

```python
import threading
import time

g_num = 0

def work1(num):
    global g_num
    for i in range(num):
        g_num += 1
    print("----in work1, g_num is %d---"%g_num)


def work2(num):
    global g_num
    for i in range(num):
        g_num += 1
    print("----in work2, g_num is %d---"%g_num)


print("---线程创建之前g_num is %d---"%g_num)

t1 = threading.Thread(target=work1, args=(1000000,))
t1.start()

t2 = threading.Thread(target=work2, args=(1000000,))
t2.start()

while len(threading.enumerate()) != 1:
    time.sleep(1)

print("2个线程对同一个全局变量操作之后的最终结果是:%s" % g_num)
```

运行结果：

```python
---线程创建之前g_num is 0---
----in work1, g_num is 1088005---
----in work2, g_num is 1286202---
2个线程对同一个全局变量操作之后的最终结果是:1286202
```

**结论**：如果多个线程同时对同一个全局变量操作，会出现**资源竞争**问题，从而数据结果会不正确

### 6. 线程同步概念

同步就是协同步调，按预定的先后次序进行运行。如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。

使用 Thread 对象的 `Lock` 和 `Rlock` 可以实现简单的线程同步，这两个对象都有 `acquire` 方法和 `release` 方法，对于那些需要每次只允许一个线程操作的数据，可以将其操作放到 acquire 和 release 方法之间。

**对于上一小节提出的那个计算错误的问题，可以通过线程同步来进行解决思路，如下:**

- 1、系统调用t1，然后获取到g_num的值为0，此时上一把锁，即不允许其他线程操作g_num
- 2、t1对g_num的值进行+1
- 3、t1解锁，此时g_num的值为1，其他的线程就可以使用g_num了，而且是g_num的值不是0而是1
- 4、同理其他线程在对g_num进行修改时，都要先上锁，处理完后再解锁，在上锁的整个过程中不允许其他线程访问，就保证了数据的正确性

### 7. 互斥锁（重点）

> 当多个线程几乎同时修改某一个共享数据的时候，需要进行同步控制
>
> 线程同步能够保证多个线程安全访问竞争资源，最简单的同步机制是引入互斥锁。
>
> 互斥锁为资源引入一个状态：锁定/非锁定
>
> 某个线程要更改共享数据时，先将其锁定，此时资源的状态为“锁定”，其他线程不能更改；直到该线程释放资源，将资源的状态变成“非锁定”，其他的线程才能再次锁定该资源。互斥锁保证了每次只有一个线程进行写入操作，从而保证了多线程情况下数据的正确性。
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/92988b37bf9d4d42be73c4aba5dee038.png)

threading模块中定义了Lock类，可以方便的处理锁定：

```python
# 创建锁
mutex = threading.Lock()

# 锁定
mutex.acquire()

# 释放
mutex.release()
```

**注意**：

- 如果这个锁之前是没有上锁的，那么acquire不会堵塞
- 如果在调用acquire对这个锁上锁之前 它已经被 其他线程上了锁，那么此时acquire会堵塞，直到这个锁被解锁为止

**使用互斥锁完成2个线程对同一个全局变量各加100万次的操作：**

```python
import threading
import time

g_num = 0

def test1(num):
    global g_num
    for i in range(num):
        mutex.acquire()  # 上锁
        g_num += 1
        mutex.release()  # 解锁

    print("---test1---g_num=%d"%g_num)

def test2(num):
    global g_num
    for i in range(num):
        mutex.acquire()  # 上锁
        g_num += 1
        mutex.release()  # 解锁

    print("---test2---g_num=%d"%g_num)

# 创建一个互斥锁
# 默认是未上锁的状态
mutex = threading.Lock()

# 创建2个线程，让他们各自对g_num加1000000次
p1 = threading.Thread(target=test1, args=(1000000,))
p1.start()

p2 = threading.Thread(target=test2, args=(1000000,))
p2.start()

# 等待计算完成
while len(threading.enumerate()) != 1:
    time.sleep(1)

print("2个线程对同一个全局变量操作之后的最终结果是:%s" % g_num)
```

运行结果：

```python
---test1---g_num=1909909
---test2---g_num=2000000
2个线程对同一个全局变量操作之后的最终结果是:2000000
```

可以看到最后的结果，加入互斥锁后，其结果与预期相符。

**上锁解锁过程**：

- 当一个线程调用锁的acquire()方法获得锁时，锁就进入“locked”状态。
- 每次只有一个线程可以获得锁。如果此时另一个线程试图获得这个锁，该线程就会变为“`blocked`”状态，称为“阻塞”，直到拥有锁的线程调用锁的`release()`方法释放锁之后，锁进入“`unlocked`”状态。
- 线程调度程序从处于同步阻塞状态的线程中选择一个来获得锁，并使得该线程进入运行（running）状态。

**总结**

- **锁的好处**：确保了某段关键代码只能由一个线程从头到尾完整地执行

- **锁的坏处**：

  阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了
  由于可以存在多个锁，不同的线程持有不同的锁，并试图**互相获取对方持有的锁时**，可能会造成死锁

### 8. 死锁

> 在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方的资源，就会造成死锁。

尽管死锁很少发生，但一旦发生就会造成应用的停止响应。下面看一个死锁的例子：

```python
import threading
import time

class MyThread1(threading.Thread):
    def run(self):
        # 对mutexA上锁
        mutexA.acquire()

        # mutexA上锁后，延时1秒，等待另外那个线程 把mutexB上锁
        print(self.name+'----do1---up----')
        time.sleep(1)

        # 此时会堵塞，因为这个mutexB已经被另外的线程抢先上锁了
        mutexB.acquire()
        print(self.name+'----do1---down----')
        mutexB.release()

        # 对mutexA解锁
        mutexA.release()

class MyThread2(threading.Thread):
    def run(self):
        # 对mutexB上锁
        mutexB.acquire()

        # mutexB上锁后，延时1秒，等待另外那个线程 把mutexA上锁
        print(self.name+'----do2---up----')
        time.sleep(1)

        # 此时会堵塞，因为这个mutexA已经被另外的线程抢先上锁了
        mutexA.acquire()
        print(self.name+'----do2---down----')
        mutexA.release()

        # 对mutexB解锁
        mutexB.release()

mutexA = threading.Lock()
mutexB = threading.Lock()

if __name__ == '__main__':
    t1 = MyThread1()
    t2 = MyThread2()
    t1.start()
    t2.start()
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac990d7b960e421d9f2c2b24d6e1b8f8.gif#pic_center)
此时已经进入到了死锁状态，可以使用`ctrl + c`退出

**如何避免死锁？**

- 程序设计时要尽量避免（银行家算法）
- 添加超时时间等

**附录-银行家算法**

**[背景知识]**

一个银行家如何将一定数目的资金安全地借给若干个客户，使这些客户既能借到钱完成要干的事，同时银行家又能收回全部资金而不至于破产，这就是银行家问题。这个问题同操作系统中资源分配问题十分相似：银行家就像一个操作系统，客户就像运行的进程，银行家的资金就是系统的资源。

**[问题的描述]**

一个银行家拥有一定数量的资金，有若干个客户要贷款。每个客户须在一开始就声明他所需贷款的总额。若该客户贷款总额不超过银行家的资金总数，银行家可以接收客户的要求。客户贷款是以每次一个资金单位（如1万RMB等）的方式进行的，客户在借满所需的全部单位款额之前可能会等待，但银行家须保证这种等待是有限的，可完成的。

例如：有三个客户C1，C2，C3，向银行家借款，该银行家的资金总额为10个资金单位，其中C1客户要借9各资金单位，C2客户要借3个资金单位，C3客户要借8个资金单位，总计20个资金单位。某一时刻的状态如图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7f076aa80e046589483283b0400d4ff.png)

对于a图的状态，按照安全序列的要求，我们选的第一个客户应满足该客户所需的贷款小于等于银行家当前所剩余的钱款，可以看出只有C2客户能被满足：C2客户需1个资金单位，小银行家手中的2个资金单位，于是银行家把1个资金单位借给C2客户，使之完成工作并归还所借的3个资金单位的钱，进入b图。同理，银行家把4个资金单位借给C3客户，使其完成工作，在c图中，只剩一个客户C1，它需7个资金单位，这时银行家有8个资金单位，所以C1也能顺利借到钱并完成工作。最后（见图d）银行家收回全部10个资金单位，保证不赔本。那麽客户序列{C1，C2，C3}就是个安全序列，按照这个序列贷款，银行家才是安全的。否则的话，若在图b状态时，银行家把手中的4个资金单位借给了C1，则出现不安全状态：这时C1，C3均不能完成工作，而银行家手中又没有钱了，系统陷入僵持局面，银行家也不能收回投资。

综上所述，银行家算法是从当前状态出发，逐个按安全序列检查各客户谁能完成其工作，然后假定其完成工作且归还全部贷款，再进而检查下一个能完成工作的客户，…。如果所有客户都能完成工作，则找到一个安全序列，银行家才是安全的。