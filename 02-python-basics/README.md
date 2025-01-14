# 第二章：Python基本语法入门

本章将介绍Python的基本语法，我们会通过简单的例子来学习这些概念。

## 1. Python代码基本结构

### 代码注释
```python
# 这是单行注释
print("Hello")  # 这也是注释

"""
这是多行注释
可以写很多行
"""
```

### 代码缩进
Python使用缩进来表示代码块：
```python
if True:
    print("这行代码被缩进了")
    print("这行也是")
print("这行没有缩进")
```

## 2. 变量和赋值

### 变量命名规则
- 只能包含字母、数字和下划线
- 不能以数字开头
- 区分大小写

```python
# 正确的变量名
user_name = "John"
age = 25
firstName = "Mike"

# 错误的变量名
# 2name = "John"  # 不能以数字开头
# my-name = "John"  # 不能使用连字符
```

### 变量赋值
```python
# 基本赋值
message = "Hello, LLM!"
number = 42

# 多个变量赋值
x, y, z = 1, 2, 3

# 值交换
a = 5
b = 10
a, b = b, a  # 现在 a = 10, b = 5
```

## 3. 基本数据类型

### 字符串（str）
```python
# 字符串创建
name = "Alice"
message = 'Hello'
long_text = """
这是多行
字符串
"""

# 字符串操作
print(name + " " + message)  # 连接字符串
print(name * 3)  # 重复字符串
print(name.upper())  # 转大写
print(len(name))  # 获取长度
```

### 数字（int和float）
```python
# 整数
age = 25
count = -10

# 浮点数
price = 19.99
temperature = -2.5

# 基本运算
sum = 5 + 3
difference = 10 - 4
product = 6 * 7
quotient = 15 / 3
```

### 布尔值（bool）
```python
is_student = True
has_license = False

# 比较运算
is_adult = age >= 18
is_valid = price < 20.00
```

## 4. 基本运算符

### 算术运算符
```python
a = 10
b = 3

print(a + b)  # 加法：13
print(a - b)  # 减法：7
print(a * b)  # 乘法：30
print(a / b)  # 除法：3.333...
print(a // b) # 整除：3
print(a % b)  # 取余：1
print(a ** b) # 幂运算：1000
```

### 比较运算符
```python
x = 5
y = 10

print(x == y)  # 等于：False
print(x != y)  # 不等于：True
print(x < y)   # 小于：True
print(x > y)   # 大于：False
print(x <= y)  # 小于等于：True
print(x >= y)  # 大于等于：False
```

### 逻辑运算符
```python
a = True
b = False

print(a and b)  # 与：False
print(a or b)   # 或：True
print(not a)    # 非：False
```

## 5. 输入和输出

### 基本输出
```python
print("Hello, World!")
print("My age is", 25)
print("Temperature:", 23.5, "degrees")
```

### 格式化输出
```python
name = "Alice"
age = 25

# 使用f-string（推荐）
print(f"My name is {name} and I am {age} years old")

# 使用format方法
print("My name is {} and I am {} years old".format(name, age))
```

### 用户输入
```python
name = input("请输入你的名字：")
print(f"你好，{name}！")

age = int(input("请输入你的年龄："))  # 将输入转换为整数
print(f"明年你将会{age + 1}岁")
```

## 练习题

1. 创建一个简单的计算器：
```python
# 创建文件 calculator.py
num1 = float(input("输入第一个数字："))
num2 = float(input("输入第二个数字："))

print(f"两数之和：{num1 + num2}")
print(f"两数之差：{num1 - num2}")
print(f"两数之积：{num1 * num2}")
print(f"两数之商：{num1 / num2}")
```

2. 温度转换程序：
```python
# 创建文件 temperature.py
celsius = float(input("输入摄氏温度："))
fahrenheit = (celsius * 9/5) + 32
print(f"{celsius}°C 等于 {fahrenheit}°F")
```

## 小结

本章我们学习了：
- Python代码的基本结构
- 变量的定义和使用
- 基本数据类型
- 基本运算符
- 输入输出操作

## 下一步

在下一章中，我们将深入学习更多的数据类型，包括列表、字典等，这些在处理LLM返回的数据时非常重要。

## 额外资源
- [Python官方教程](https://docs.python.org/zh-cn/3/tutorial/)
- [Python基础教程](https://www.runoob.com/python3/python3-tutorial.html)
