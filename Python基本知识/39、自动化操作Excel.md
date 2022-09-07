[toc]

## 一、Python操作Excel 7大库对比

> Excel是Windows环境下流行的、强大的电子表格应用。无论是在工作中还是学习中我们都几乎在不间断的使用Excel来 记录或者处理一些数据。 例如，可能有-个无聊的任务，需要从一个电子表格拷贝一些数据，粘贴到另一个电子表格中。或者可能需要从几千行中挑选几行，根据某种条件稍作修改。或者需要查看几百份部门预算电子表格，寻找其中的 指定内容。正是这种无聊无脑的电子表格任务，如果让人工来手动完成，则无论是时间升本还是精力成本都不是一件好事情，但是可以通过Python来完成。

7大库功能对比图如下（本文主要介绍：xlrd和xlwt）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/81b2f18cf0a9448eaa5a2fa11faaad69.png)

## 二、xlrd 读取excel操作

> **xlrd模块**用于读取Excel的数据，速度非常快。支持`.xls` 和 `.xlsx`两种文件格式的读取

**1. cmd控制台安装模块**：

```csharp
pip install xlrd
```

**2. 导入模块**：

```csharp
import xlrd
```

测试表如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf64cf858b4b470f9a1851d955ff1b75.png)

### 1. 打开文件

```python
xlrd.open_workbook(path)
wb = xlrd.open_workbook("./a.xls")
print(wb)
```

运行结果：

```csharp
<xlrd.book.Book object at 0x000001F905A33DC0>
```

>  xlrd.biffh.XLRDError: Excel xlsx file； not supported，两种解决方案
>
> ```bash
> ·调低xlrd版本
> xlrd过高，卸载旧版本重新安装1.2.0
> 1、打开terminal
> 2、卸载现在的版本 pip uninstall xlrd
> 3、安装低本班xlrd：pip install xlrd==1.2.0
> ```



### 2. 获取所有表名

> ```python
> wb.sheet_names()
> ```

```csharp
sheet_names_list = wb.sheet_names()
print(sheet_names_list)
```

运行结果：

```csharp
['Sheet1', 'Sheet2', 'Sheet3']
```

### 3. 指定sheet表

> ```python
> wb.sheet_by_index(索引)` or `wb.sheet_by_name("sheet表名")
> ```

```python
# 方式1：索引顺序获取
sheet_1 = wb.sheet_by_index(0)

# 方式2:名称获取
sheet_2 = wb.sheet_by_name("Sheet1")
```

### 4. 对sheet表的行操作

> - **1. 获取sheet表总函数**：`sheet.nrows`
> - **2. 返回由该行中所有的单元格对象组成的列表，列表内是键值对**：`sheet.row(1)`，如：`[列名1:'值',列名2:'值', ...]`
> - 或者`sheet.row_slice(1)`
> - **3. 返回指定行的所有单元格数值组成的列表**：`sheet.row_values(rowx, start_colx=0, end_colx=None)` ，如：`['值1','值2',...]`
> - **4. 返回指定行的有效长度**：`sheet.row_len(rowx)`

```python
# 1.获取sheet表总函数：sheet.nrows
print("总行数为：", sheet_1.nrows)
# 2.返回由该行中所有的单元格对象组成的列表，列表内是键值对: sheet.row(1)
print(sheet_1.row(1))
# 或者sheet.row_slice(1)
print(sheet_1.row_slice(1))
# 3.返回指定行的所有单元格数值组成的列表: sheet.row_values(rowx, start_colx=0, end_colx=None)
print(sheet_1.row_values(1))
# 4.返回指定行的有效长度：sheet.row_len(rowx)
print("返回指定行的有效长度：", sheet_1.row_len(1))

print('-' * 20)
# 5. 读取整个表
for i in range(sheet_1.nrows):
    print(sheet_1.row_values(i))
```

运行结果：

```python
总行数为： 4
[text:'物品1', empty:'', number:100.0, text:'EUR 0.1', text:'EUR 0.1']
[text:'物品1', empty:'', number:100.0, text:'EUR 0.1', text:'EUR 0.1']
['物品1', '', 100.0, 'EUR 0.1', 'EUR 0.1']
返回指定行的有效长度： 5
--------------------
['item', 'b', 'PCS', 'UNIT', 'TOTAL']
['物品1', '', 100.0, 'EUR 0.1', 'EUR 0.1']
['物品2', '', 100.0, 'EUR 0.1', 'EUR 0.1']
['物品3', '', 200.0, 'EUR 0.2', 'EUR 0.2']
```

### 5. 对sheet表的列操作

> - **1. 返回指定sheet表的有效列数**：`sheet.ncols`
> - **2. 返回由该列中所有的单元格对象组成的列表，列表内是键值对**：`sheet.col(1)`，如：`[列名1:'值',列名2:'值', ...]`
> - 或者`sheet.col_slice(1)`
> - **3. 返回指定列的所有单元格数值组成的列表：sheet.col_values(colx, start_colx=0, end_colx=None)**，如：`['列名','值1','值2',...]`

```python
# 1. 返回指定sheet表的有效列数：sheet.ncols
print("总列数为：", sheet_1.ncols)
# 2. 返回由该列中所有的单元格对象组成的列表，列表内是键值对：sheet.col(1)
print(sheet_1.col(1))
# 或者sheet.col_slice(1)
print(sheet_1.col_slice(1))
# 3. 返回指定列的所有单元格数值组成的列表：sheet.col_values(colx, start_colx=0, end_colx=None)
print(sheet_1.col_values(0))  # 带列名
print(sheet_1.col_values(0, 1))  # 不带列名
print('-' * 20)

# 4. 读取全部表
for i in range(sheet_1.ncols):
    print(sheet_1.col_values(i))
```

运行结果：

```python
总列数为： 5
[text:'b', empty:'', empty:'', empty:'']
[text:'b', empty:'', empty:'', empty:'']
['item', '物品1', '物品2', '物品3']
['物品1', '物品2', '物品3']
--------------------
['item', '物品1', '物品2', '物品3']
['b', '', '', '']
['PCS', 100.0, 100.0, 200.0]
['UNIT', 'EUR 0.1', 'EUR 0.1', 'EUR 0.2']
['TOTAL', 'EUR 0.1', 'EUR 0.1', 'EUR 0.2']
1234567891011
```

## 三、xlwt 写入Excel表操作

> **xlwt 模块**可以将数据写入到 Excel 工作簿中。目前已支持写入`.xls` 和 `.xlsx`两种文件格式

**1. cmd控制台安装模块**：

```csharp
pip install xlwt
```

**2. 导入模块**：

```csharp
import xlwt
```

### 1. 写入单个数据

> - **1. 创建Excel表对象**：`xlwt.Workbook(encoding='utf8')`
> - **2. 新建sheet表**：`worksheet = workbook.add_sheet('Sheet1')`
> - **3. 写入数据到指定单元格**：`worksheet.write(0, 0, "python")`。第一个参数是第 i 行，第二个是第 j 列，第三个是要写的参数（字符串或数字）
> - **4. 保存文件**：`workbook.save('test.xlsx')`。目前已支持写入`.xls` 和 `.xlsx`两种文件格式

测试代码：

```python
import xlwt

# 1. 创建Excel表对象
workbook = xlwt.Workbook(encoding='utf8')
# 2. 新建sheet表
worksheet = workbook.add_sheet('Sheet1')
# 3. 写入数据到指定单元格
worksheet.write(0, 0, "python")
# 4. 保存文件分两种格式
workbook.save('test.xls')
workbook.save('test.xlsx')
```

运行结果如下：生成两个新文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/96e208dc858447cfa5bab3ad2c794dc7.png)

### 2. 写入多个数据

测试代码：

```python
import xlwt

data_list = [('小白', '20', '男'), ('小黑', '21', '男'), ('小红', '20', '女')]
# 1. 创建Excel表对象
workbook = xlwt.Workbook(encoding='utf8')
# 2. 新建sheet表
worksheet = workbook.add_sheet('Sheet1')
# 3. 自定义列名
col1 = ('姓名', '年龄', '性别')
# 4. 将列属性元组col写进sheet表单中第一行
for i in range(0, len(col1)):
    worksheet.write(0, i, col1[i])
# 5. 将数据写进sheet表单中
for i in range(0, len(data_list)):
    data = data_list[i]
    for j in range(0, len(col1)):
        worksheet.write(i + 1, j, data[j])
# 6. 保存文件分两种格式
workbook.save('test.xls')
```

运行结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e918258ad9244c7ea0882a60957676ea.png)

### 3. 设置列宽

> cols_num是列的数目，可以通过修改12这个值，修改列的宽度

```python
for c in range(len(col1)):
    worksheet.col(c).width = 256 * 24
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c52373b1a1b4499902623bc1c3464e8.png)

### 4. 设置行高

> 设置行高，修改1600为别的值，可以修改行的高度

测试代码：

```python
# 5. 将数据写进sheet表单中
for i in range(0, len(data_list)):
    data = data_list[i]
    for j in range(0, len(col1)):
        worksheet.write(i + 1, j, data[j])
        # 6. 设置行高
        worksheet.row(i + 1).height_mismatch = True
        worksheet.row(i + 1).height = 1600  # 设置行高
```

运行结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6d236c483904fab8b9c515d4d71a97a.png)

### 5. 设置单元格风格

常用设置单元格的属性：

| 样式               | 说明     |
| ------------------ | -------- |
| `xlwt.Font()`      | 字体设置 |
| `xlwt.Pattern()`   | 背景设置 |
| `xlwt.Borders()`   | 边框设置 |
| `xlwt.Alignment()` | 对准设置 |

测试代码：

```python
def body_style():
    # 一、创建一个样式对象，初始化样式 style
    style = xlwt.XFStyle()  # Create Style对象

    # 二、字体风格设置
    font = xlwt.Font()  # Create Font对象
    font.name = "SimSun"  # 设置字体类型，宋体
    font.colour_index = 4  # 设置字体颜色
    font.height = 20 * 12  # 字体大小，12为字号，20为衡量单位
    font.bont = True  # 设置字体加粗
    font.underline = True  # 下划线
    font.italic = True  # 斜体字

    # 二、背景设置
    pattern = xlwt.Pattern()
    pattern.pattern = xlwt.Pattern.SOLID_PATTERN  # May be: NO_PATTERN, SOLID_PATTERN, or 0x00 through 0x12
    pattern.pattern_fore_colour = 4  # 给背景颜色赋值

    # 三、边框设置
    borders = xlwt.Borders()  # 创建边框对象，    # .DASHED：虚线；.NO_LINE：没有
    # 上下左右都添加边框
    borders.left = 1
    borders.right = 1
    borders.top = 1
    borders.bottom = 1
    # 设置边框颜色
    borders.left_colour = 2
    borders.right_colour = 2
    borders.top_colour = 2
    borders.bottom_colour = 2

    # 四、位置设置
    alignment = xlwt.Alignment()
    alignment.horz = 1  # 设置水平位置，0是左对齐，1是居中，2是右对齐
    alignment.wrap = 1  # 设置自动换行

    # 五、设置好之后，全部都加到style上
    style.alignment = alignment
    style.font = font
    style.borders = borders
    return style

1234567891011121314151617181920212223242526272829303132333435363738394041
```

应用样式：

```python
sheet.write(row, column, i)  # 不带格式
sheet.write(row, column, i, style)  # 有格式
```

运行结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f2c41dff86447f2b7e85457dce09e44.png)