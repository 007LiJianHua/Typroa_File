[toc]

## 一、sys作用

> Python的sys模块提供访问由解释器使用或维护的变量的接口，并提供了一些函数用来和解释器进行交互，操控Python的运行时环境。

```python
>>> import sys
```

## 二、常用变量

### 1、sys.version

> 返回Python解释器版本号；用于某程序需要用指定版本号运行

```python
>>> import sys
>>> sys.version
3.8.5 (tags/v3.8.5:580fbb0, Jul 20 2020, 15:57:54) [MSC v.1924 64 bit (AMD64)]
```

### 2、sys.maxsize

> 表示操作系统承载的最大int值

```python
>>> import sys
>>> sys.maxsize
9223372036854775807
```

### 3、sys.maxunicode

> 给出最大Unicode代码点值的整数，即1114111（十六进制0x10FFFF）。

```python
>>> import sys
>>> sys.maxunicode
1114111
```

### 4、sys.path

当前脚本的path环境变量，如果没有python就找到

```python
>>> import sys
>>> sys.path
['', 'D:\\Python3.8\\python38.zip', 'D:\\Python3.8\\DLLs', 'D:\\Python3.8\\lib', 'D:\\Python3.8', 'D:\\Python3.8\\lib\\site-packages']
```

### 5、sys.platform

> 返回操作系统平台名称，在编写跨平台应用时很有用

| 系统    | 返回值   |
| ------- | -------- |
| Windows | ‘win32’  |
| Linux   | ‘linux’  |
| Mac     | ‘darwin’ |

```python
>>> import sys
>>> sys.platform
win32
```

### 6、sys.argv

> 将python脚本运行时的脚本名以及参数作为一个list，并输出。实现从程序外部像程序内容传递参数

```python
import sys
print(sys.argv)
123
E:\Python> python 2.py hello python
['2.py', 'hello', 'python']
```

### 7、sys.executable

> 一个字符串，给出Python解释器的可执行二进制文件的`绝对路径`。如果Python无法检索其可执行文件的真实路径，`sys.executable`则将为空字符串或`None`。

```python
>>> import sys
>>> sys.executable
'D:\\Python3.8\\python.exe'
```

### 8、sys.byteorder

> 本地字节顺序的指示符——在大端序（最高有效位优先）操作系统上值为 ‘big’ ，在小端序（最低有效位优先）操作系统上为 'little

’

```python
>>> import sys
>>> sys.byteorder
'little'
```

### 9、sys.version_info

> 包含版本号的五个组件的元组：major，minor， micro，releaselevel和serial。

```python
>>> import sys
>>> sys.version_info
sys.version_info(major=3, minor=7, micro=3, releaselevel='final', serial=0)
```

### 10、sys.api_version

> 此解释器的C API版本。

```python
>>> import sys
>>> sys.api_version
1013
```

### 11、sys.stdin/sys.stdout/sys.stderr

> 标准输入、输出、错误。

```python
>>> import sys
>>> sys.stdin
<_io.TextIOWrapper name='<stdin>' mode='r' encoding='utf-8'>
>>> sys.stdout
<_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
>>> sys.stderr
<_io.TextIOWrapper name='<stderr>' mode='w' encoding='utf-8'>
```

## 三、常用方法

### 1、sys.exit()

> 退出程序，正常退出时exit(0)

```python
import sys
print(sys.exit(0))
print(sys.exit(1))
```

### 2、sys.modules

> 返回系统导入的模块字段，key是模块名，value是模块

```python
>>> import sys
>>> sys.modules
{'sys': <module 'sys' (built-in)>, 'builtins': <module 'builtins' (built-in)>, '_frozen_importlib': <module 'importlib._bootstrap' (frozen)>, '_imp': <module '_imp' (built-in)>, '_warnings': <module '_warnings' (built-in)>, '_frozen_importlib_external': <module 'importlib._bootstrap_external' (frozen)>, '_io': <module 'io' (built-in)>, 'marshal': <module 'marshal' (built-in)>, 'nt': <module 'nt' (built-in)>, '_thread': <module '_thread' (built-in)>, '_weakref': <module '_weakref' (built-in)>, 'winreg': <module 'winreg' (built-in)>, 'time': <module 'time' (built-in)>, 'zipimport': <module 'zipimport' (frozen)>, '_codecs': <module '_codecs' (built-in)>, 'codecs': <module 'codecs' from 'D:\\Python3.8\\lib\\codecs.py'>, 'encodings.aliases': <module 'encodings.aliases' from 'D:\\Python3.8\\lib\\encodings\\aliases.py'>, 'encodings': <module 'encodings' from 'D:\\Python3.8\\lib\\encodings\\__init__.py'>, 'encodings.utf_8': <module 'encodings.utf_8' from 'D:\\Python3.8\\lib\\encodings\\utf_8.py'>, '_codecs_cn': <module '_codecs_cn' (built-in)>, '_multibytecodec': <module '_multibytecodec' (built-in)>, 'encodings.gbk': <module 'encodings.gbk' from 'D:\\Python3.8\\lib\\encodings\\gbk.py'>, '_signal': <module '_signal' (built-in)>, '__main__': <module '__main__' (built-in)>, 'encodings.latin_1': <module 'encodings.latin_1' from 'D:\\Python3.8\\lib\\encodings\\latin_1.py'>, '_abc': <module '_abc' (built-in)>, 'abc': <module 'abc' from 'D:\\Python3.8\\lib\\abc.py'>, 'io': <module 'io' from 'D:\\Python3.8\\lib\\io.py'>, '_stat': <module '_stat' (built-in)>, 'stat': <module 'stat' from 'D:\\Python3.8\\lib\\stat.py'>, '_collections_abc': <module '_collections_abc' from 'D:\\Python3.8\\lib\\_collections_abc.py'>, 'genericpath': <module 'genericpath' from 'D:\\Python3.8\\lib\\genericpath.py'>, 'ntpath': <module 'ntpath' from 'D:\\Python3.8\\lib\\ntpath.py'>, 'os.path': <module 'ntpath' from 'D:\\Python3.8\\lib\\ntpath.py'>, 'os': <module 'os' from 'D:\\Python3.8\\lib\\os.py'>, '_sitebuiltins': <module '_sitebuiltins' from 'D:\\Python3.8\\lib\\_sitebuiltins.py'>, '_locale': <module '_locale' (built-in)>, '_bootlocale': <module '_bootlocale' from 'D:\\Python3.8\\lib\\_bootlocale.py'>, 'types': <module 'types' from 'D:\\Python3.8\\lib\\types.py'>, 'importlib._bootstrap': <module 'importlib._bootstrap' (frozen)>, 'importlib._bootstrap_external': <module 'importlib._bootstrap_external' (frozen)>, 'warnings': <module 'warnings' from 'D:\\Python3.8\\lib\\warnings.py'>, 'importlib': <module 'importlib' from 'D:\\Python3.8\\lib\\importlib\\__init__.py'>, 'importlib.machinery': <module 'importlib.machinery' from 'D:\\Python3.8\\lib\\importlib\\machinery.py'>, 'importlib.abc': <module 'importlib.abc' from 'D:\\Python3.8\\lib\\importlib\\abc.py'>, '_operator': <module '_operator' (built-in)>, 'operator': <module 'operator' from 'D:\\Python3.8\\lib\\operator.py'>, 'keyword': <module 'keyword' from 'D:\\Python3.8\\lib\\keyword.py'>, '_heapq': <module '_heapq' (built-in)>, 'heapq': <module 'heapq' from 'D:\\Python3.8\\lib\\heapq.py'>, 'itertools': <module 'itertools' (built-in)>, 'reprlib': <module 'reprlib' from 'D:\\Python3.8\\lib\\reprlib.py'>, '_collections': <module '_collections' (built-in)>, 'collections': <module 'collections' from 'D:\\Python3.8\\lib\\collections\\__init__.py'>, '_functools': <module '_functools' (built-in)>, 'functools': <module 'functools' from 'D:\\Python3.8\\lib\\functools.py'>, 'contextlib': <module 'contextlib' from 'D:\\Python3.8\\lib\\contextlib.py'>, 'importlib.util': <module 'importlib.util' from 'D:\\Python3.8\\lib\\importlib\\util.py'>, 'mpl_toolkits': <module 'mpl_toolkits' (namespace)>, 'site': <module 'site' from 'D:\\Python3.8\\lib\\site.py'>, 'atexit': <module 'atexit' (built-in)>}
```

### 3、sys.modules.keys()

> 返回所有已导入的模块名列表

```python
>>> import sys
>>> sys.modules.keys()
dict_keys(['sys', 'builtins', '_frozen_importlib', '_imp', '_warnings', '_frozen_importlib_external', '_io', 'marshal', 'nt', '_thread', '_weakref', 'winreg', 'time', 'zipimport', '_codecs', 'codecs', 'encodings.aliases', 'encodings', 'encodings.utf_8', '_codecs_cn', '_multibytecodec', 'encodings.gbk', '_signal', '__main__', 'encodings.latin_1', '_abc', 'abc', 'io', '_stat', 'stat', '_collections_abc', 'genericpath', 'ntpath', 'os.path', 'os', '_sitebuiltins', '_locale', '_bootlocale', 'types', 'importlib._bootstrap', 'importlib._bootstrap_external', 'warnings', 'importlib', 'importlib.machinery', 'importlib.abc', '_operator', 'operator', 'keyword', '_heapq', 'heapq', 'itertools', 'reprlib', '_collections', 'collections', '_functools', 'functools', 'contextlib', 'importlib.util', 'mpl_toolkits', 'site', 'atexit'])
```

### 4、sys.getdefaultencoding()

> 返回Unicode实现使用的当前默认字符串编码的名称。

```python
>>> import sys
>>> sys.getdefaultencoding()
'utf-8'
```

### 5、sys.getfilesystemencoding()

> 返回用于在Unicode文件名和字节文件名之间进行转换的编码名称

```python
>>> import sys
>>> sys.getfilesystemencoding()
utf-8
```

### 6、sys.getrecursionlimit()

> 返回最大递归次数

```python
>>> import sys
>>> sys.getrecursionlimit()  # 查看当前解释器的最大递归深度
1000
```

### 7、sys.setrecursionlimit(num)

> 设置最大递归次数

```python
>>> import sys
>>> sys.setrecursionlimit(1100)  # 将解释器的最大递归深度设置为1100
>>> sys.getrecursionlimit()  # 再次查看当前解释器的最大递归深度
1100
```

### 8、sys.getsizeof()

> 获取对象占用的内存大小（用字节表示）

```python
>>> import sys
>>> for obj in [int(), float(), list(), tuple(), set(), dict(), object]:
...     print(str(obj.__class__).ljust(20), sys.getsizeof(obj))
...
<class 'int'>        24
<class 'float'>      24
<class 'list'>       56
<class 'tuple'>      40
<class 'set'>        216
<class 'dict'>       232
<class 'type'>       416
```

### 9、sys.getrefcount(obj)

> 返回obj的引用计数。返回的计数通常比预期的高一个，因为它包含（临时）引用作为参数。

```python
>>> import sys
>>> a = [1,2,3]
>>> b = a
>>> c = b
>>> sys.getrefcount(a)
4
```

### 10、sys.exc_info()

获取当前正在处理的异常类，exc_type、exc_value、exc_traceback当前处理的异常详细信息

```python
>>> import sys
>>> sys.exc_info()
(None, None, None)
```

### 11、sys.getwindowsversion()

> 获取Windows的版本，Windows系统中有效

```python
>>> import sys
>>> sys.getwindowsversion()
sys.getwindowsversion(major=10, minor=0, build=19041, platform=2, service_pack='')
```

### 12、sys.stdin.readline()

> 从标准输入读一行，会读取末尾的换行符

### 13、sys.stdout.write()

> 向标准输出写入内容

```python
>>> import sys
>>> sys.stdout.write("hello world")
hello world11
```