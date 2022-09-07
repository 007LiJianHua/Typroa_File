[toc]

## 一、[爬虫](https://so.csdn.net/so/search?q=爬虫&spm=1001.2101.3001.7020)提取网页数据的流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/eeff6487353d453d908776e47d03a4e8.png)

## 二、[lxml](https://so.csdn.net/so/search?q=lxml&spm=1001.2101.3001.7020)库

> **lxml**是XML和HTML的解析器，其主要功能是解析和提取XML和HTML中的数据；，是一款高性能的python HTML、XML解析器，也可以利用[XPath](https://so.csdn.net/so/search?q=XPath&spm=1001.2101.3001.7020)语法，来定位特定的元素及节点信息

### 1. 下载安装

**1. window电脑点击`win键+ R`，输入：`cmd`**

![在这里插入图片描述](https://img-blog.csdnimg.cn/1b2ce81ab29e439e8954c293fb4387e4.png)

**2. 安装`lxml`，输入对应的pip命令**：`pip install lxml`，我已经安装过了出现版本就安装成功了

![在这里插入图片描述](https://img-blog.csdnimg.cn/3c42e77733ee48c2a4285c00787bc55e.png)

### 2. 解析HTML网页

> 主要使用的lxml库中的etree类

**案例1**：**解析HTML字符串**

* 使用etreee.HTML

```python
from lxml import etree

text = '''
<html><body>
    <div class="key">
        <div class="name">无羡</div>
        <div class="age">20</div>
        <div class="address">四川</div>
    </div>
</body></html>
'''
# 开始初始化
html = etree.HTML(text)  # 这里需要传入一个html形式的字符串
print(html)
print(type(html))
# 将字符串序列化为html字符串
result = etree.tostring(html).decode('utf-8')
print(result)
print(type(result))
```

输出结果：

```python
<Element html at 0x1f7fa7f2a80>
<class 'type'>
<html><body>
    <div class="key">
        <div class="name">&#26080;&#32673;</div>
        <div class="age">20</div>
        <div class="address">&#22235;&#24029;</div>
    </div>
</body></html>
<class 'str'>
```

**案例2**：**读取并解析HTML文件**

创建一个html文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20a0a5420fdd4a778fad86b7e11f8cb6.png)

```python
from lxml import etree

# 将html文件进行读取
html = etree.parse('1.html')

# 将html内容序列化
result = etree.tostring(html).decode('utf-8')
print(result)
print(type(result))
html = etree.HTML(result)  # 这里需要传入一个html形式的字符串
print(html)
print(type)
```

输出结果：

```python
<html><body>
    <div class="key">
        <div class="name">&#26080;&#32673;</div>
        <div class="age">20</div>
        <div class="address">&#22235;&#24029;</div>
    </div>
</body></html>
<class 'str'>
<Element html at 0x17dffaa4c40>
<class 'type'>
```

**lxml库掌握以上内容即可！**

## 三、Xpath介绍

> **XPath** (XML Path Language) 是一门在 XML 文档中查找信息的语言，可用来在 XML 文档中对元素和属性进行遍历。

### 1. 选取节点

**最常用的路径表达式：**

| 表达式     | 说明                                                       |
| ---------- | ---------------------------------------------------------- |
| `nodename` | 选取此节点的所有子节点。                                   |
| `/`        | 从根节点选取。                                             |
| `//`       | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。 |
| `.`        | 选取当前节点。                                             |
| `..`       | 选取当前节点的父节点。                                     |
| `@`        | 选取属性。                                                 |

案例：

| 表达式            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `bookstore`       | 选取 bookstore 元素的所有子节点。                            |
| `/bookstore`      | 选取根元素 bookstore。注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ |
| `bookstore/book`  | 选取属于 bookstore 的子元素的所有 book 元素。                |
| `//book`          | 选取所有 book 子元素，而不管它们在文档中的位置。             |
| `bookstore//book` | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| `//@lang`         | 选取名为 lang 的所有属性。                                   |

### 2. 谓语

> 谓语用来查找某个特定的节点或者包含某个指定的值的节点，被嵌在方括号中。

| 路径表达式                           | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `/bookstore/book[1]`                 | 选取属于 bookstore 子元素的第一个 book 元素。                |
| `/bookstore/book[last()]`            | 选取属于 bookstore 子元素的最后一个 book 元素。              |
| `/bookstore/book[last()-1]`          | 选取属于 bookstore 子元素的倒数第二个 book 元素。            |
| `/bookstore/book[position()<3]`      | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。    |
| `//title[@lang]`                     | 选取所有拥有名为 lang 的属性的 title 元素。                  |
| `//title[@lang=’eng’]`               | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。   |
| `/bookstore/book[price>35.00]`       | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。 |
| `/bookstore/book[price>35.00]/title` | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |

### 3. 选取未知节点

> XPath 通配符可用来选取未知的 XML 元素。

| 通配符   | 说明                 |
| -------- | -------------------- |
| `*`      | 匹配任何元素节点。   |
| `@*`     | 匹配任何属性节点。   |
| `node()` | 匹配任何类型的节点。 |

案例：

| 路径表达式            | 说明                                       |
| --------------------- | ------------------------------------------ |
| `/bookstore/*`        | 选取 bookstore 元素的所有子元素。          |
| `//*`                 | 选取文档中的所有元素。                     |
| `html/node()/meta/@*` | 选择html下面任意节点下的meta节点的所有属性 |
| `//title[@*]`         | 选取所有带有属性的 title 元素。            |

### 4. 选取若干路径

> 通过在路径表达式中使用`“|”`运算符，您可以选取若干个路径。

案例：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ec07949e9c34ab396361a08730514f2.png)

### 5. Chrome插件 XPath Helper安装使用

免费下载地址：[Chrome插件Xpath_helper.zip](https://download.csdn.net/download/yuan2019035055/85883020)

**一. 安装**：

- 1.、解压压缩包
- 2、打开chrome页面，点击：右上角三个点 > 更多工具 > 扩展程序
- 3、点击`加载已解压的扩展程序`，选择压缩包的路径

![在这里插入图片描述](https://img-blog.csdnimg.cn/d54ff73d950048459573536e7b1f7ec4.png)

- 3、将插件固定在左上角

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/008e7161c1e24eb79dc9d6c1596bd0c4.png)

- 4、重启谷歌浏览器

**二、使用：**

- 快捷键ctrl+shift+X 开启 xpath-helper插件。或者点击左上角图标
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/09dc1661e03d4dd2bbb49502b8b7db03.png)

- 定位爬取的内容：按住`x`键，移动鼠标在爬取内容上即可显示标签路径

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/00366406209747578e99d5ae9c72f436.png)

### 6. Xpath实战

新建一个`hello.html`文件：

```python
<!-- hello.html -->

<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
```

**1. 获取所有的 `<li>` 标签**：

```python
from lxml import etree

html = etree.parse('hello.html')
print(type(html))  # 显示etree.parse() 返回类型

result = html.xpath('//li')

print(result)  # 打印<li>标签的元素集合
print(len(result))
print(type(result))
print(type(result[0]))
```

输出结果：

```python
<type 'lxml.etree._ElementTree'>
[<Element li at 0x1014e0e18>, <Element li at 0x1014e0ef0>, <Element li at 0x1014e0f38>, <Element li at 0x1014e0f80>, <Element li at 0x1014e0fc8>]
5
<type 'list'>
<type 'lxml.etree._Element'>
```

**2. 继续获取`<li>` 标签的所有 `class`属性**：

```python
from lxml import etree

html = etree.parse('hello.html')
result = html.xpath('//li/@class')

print(result)
```

输出结果：

```python
['item-0', 'item-1', 'item-inactive', 'item-1', 'item-0']
```

**3. 继续获取`<li>`标签下hre 为 `link1.html` 的 `<a>` 标签**：

```python
from lxml import etree

html = etree.parse('hello.html')
result = html.xpath('//li/a[@href="link1.html"]')

print(result)
```

输出结果：

```python
[<Element a at 0x10ffaae18>]
1
```

**4. 获取`<li>` 标签下的所有 `<span>` 标签**：

```python
from lxml import etree

html = etree.parse('hello.html')

#result = html.xpath('//li/span')
#注意这么写是不对的：
#因为 / 是用来获取子元素的，而 <span> 并不是 <li> 的子元素，所以，要用双斜杠

result = html.xpath('//li//span')

print(result)
```

输出结果：

```python
[<Element span at 0x10d698e18>]
1
```

**5. 获取 `<li>` 标签下的`<a>`标签里的所有 `class`**：

```python
from lxml import etree

html = etree.parse('hello.html')
result = html.xpath('//li/a//@class')

print(result)
```

输出结果：

```python
['blod']
```

**6. 获取最后一个 `<li>` 的 `<a>` 的 `href`**：

```python
from lxml import etree

html = etree.parse('hello.html')

result = html.xpath('//li[last()]/a/@href')
# 谓语 [last()] 可以找到最后一个元素

print(result)
```

输出结果：

```python
['link5.html']
```

**7. 获取倒数第二个元素的内容**：

```python
from lxml import etree

html = etree.parse('hello.html')
result = html.xpath('//li[last()-1]/a')

# text 方法可以获取元素内容
print(result[0].text)
```

输出结果：

```python
fourth item
```

**8. 获取 `class` 值为 `bold` 的标签名**：

```python
from lxml import etree

html = etree.parse('hello.html')

result = html.xpath('//*[@class="bold"]')

# tag方法可以获取标签名
print(result[0].tag)
```

输出结果：

```python
span
```