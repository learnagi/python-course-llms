---
title: "变量和数据类型"
slug: "variables-and-types"
sequence: 2
description: "学习Python中的变量定义、命名规则和基本数据类型，包括字符串、数字和布尔值"
status: "published"
---

# 变量和数据类型

变量是存储数据的容器，而数据类型定义了变量可以存储的数据种类。本节将详细介绍Python中的变量使用和基本数据类型。

## 变量

### 变量命名规则
- 只能包含字母、数字和下划线
- 不能以数字开头
- 区分大小写
- 不能使用Python关键字

```python
# 正确的变量名
user_name = "John"
age = 25
firstName = "Mike"
_private = "Hidden"

# 错误的变量名
# 2name = "John"    # 不能以数字开头
# my-name = "John"  # 不能使用连字符
# class = "Python"  # 不能使用关键字
```

### 变量赋值
```python
# 基本赋值
message = "Hello, Python!"
number = 42

# 多个变量赋值
x, y, z = 1, 2, 3

# 值交换
a = 5
b = 10
a, b = b, a  # 现在 a = 10, b = 5
```

## 基本数据类型

### 字符串（str）
字符串是一系列字符的序列：

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
print(name * 3)             # 重复字符串
print(name.upper())         # 转大写
print(len(name))           # 获取长度
print(name[0])             # 获取第一个字符
```

### 数字
Python支持多种数字类型：

#### 整数（int）
```python
age = 25
count = -10
big_number = 1_000_000  # 使用下划线提高可读性
```

#### 浮点数（float）
```python
price = 19.99
temperature = -2.5
scientific = 1.23e-4  # 科学计数法
```

### 布尔值（bool）
布尔值表示真（True）或假（False）：

```python
is_student = True
has_license = False

# 比较运算产生布尔值
is_adult = age >= 18
is_valid = price < 20.00
```

### 类型转换
Python提供了多种类型转换函数：

```python
# 字符串转数字
age_str = "25"
age_num = int(age_str)      # 转换为整数
price_str = "19.99"
price_num = float(price_str) # 转换为浮点数

# 数字转字符串
count = 42
count_str = str(count)      # 转换为字符串

# 转换为布尔值
empty_string = bool("")     # False
zero = bool(0)              # False
non_zero = bool(42)         # True
```

## 类型检查

可以使用`type()`函数检查变量的类型：

```python
name = "Alice"
age = 25
price = 19.99
is_student = True

print(type(name))      # <class 'str'>
print(type(age))       # <class 'int'>
print(type(price))     # <class 'float'>
print(type(is_student)) # <class 'bool'>
```

## 最佳实践

1. 使用有意义的变量名，能清楚表达变量的用途
2. 保持一致的命名风格（推荐使用小写字母和下划线）
3. 适当使用类型转换，确保数据类型的正确性
4. 注意数值计算中的精度问题，特别是浮点数

## 下一步

现在您已经掌握了Python的变量和基本数据类型，接下来我们将学习Python中的各种运算符。
