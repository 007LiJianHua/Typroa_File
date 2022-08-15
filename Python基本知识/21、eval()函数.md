[toc]

## 简介eval

> eval() 函数十分强大 —— **将字符串** 当成 **有效的表达式** 来求值 并 **返回计算结果**

**案例**：

```python
# 基本的数学计算
print(eval("1 + 1")) # 2

# 字符串重复
print(eval("'*' * 10")) # **********

# 将字符串转换成列表
print(type(eval("[1, 2, 3, 4, 5]"))) # <class 'list'>

# 将字符串转换成字典
print(type(eval("{'name': 'xiaoming', 'age': 18}"))) # <class 'dict'>
```

**案例 - 计算器**

**需求**

- 提示用户输入一个 **加减乘除混合运算**

- 返回计算结果

  ```python
  input_str = input("请输入一个算术题：")
  
  print(eval(input_str))
  ```

**不要滥用 eval**：

- 在开发时千万不要使用 eval 直接转换 input 的结果

  ```python
  __import__('os').system('ls')
  ```

  等价代码

  ```python
  import os
  
  os.system("终端命令")
  ```

- 执行成功，返回 0

- 执行失败，返回错误信息