---
title: "Python代码基本结构"
slug: "code-structure"
sequence: 1
description: "学习Python代码的基本结构，包括注释、缩进和代码块的概念"
status: "published"
---

# Python代码基本结构

Python的代码结构清晰简洁，本节将介绍Python代码的基本组成部分。

## 代码注释

注释是程序中的说明性文字，不会被执行。Python支持两种类型的注释：

### 单行注释
使用`#`符号添加单行注释：
```python
# 这是单行注释
print("Hello")  # 这也是注释，可以跟在代码后面
```

### 多行注释
使用三个引号（`"""` 或 `'''`）添加多行注释：
```python
"""
这是多行注释
可以写很多行
Python会忽略这些内容
"""
```

## 代码缩进

Python使用缩进来表示代码块，这是Python最显著的特征之一：

### 基本规则
- 同一个代码块中的语句必须有相同的缩进
- 通常使用4个空格作为一个缩进级别
- 不要混用空格和Tab

### 示例
```python
if True:
    print("这行代码被缩进了")  # 缩进4个空格
    print("这行也是")         # 同样的缩进
print("这行没有缩进")         # 新的代码块
```

## 代码块

代码块是构成Python程序的基本单位：

### 常见的代码块结构
```python
# if语句块
if condition:
    # 这里的代码是一个块
    statement1
    statement2

# 函数定义块
def function_name():
    # 这里的代码是一个块
    statement1
    statement2

# 循环块
for item in items:
    # 这里的代码是一个块
    statement1
    statement2
```

## 行的连接

Python通常一行写一条语句，但有时需要将长语句分成多行：

### 使用括号
```python
total = (item_one
         + item_two
         + item_three)
```

### 使用反斜杠
```python
total = item_one + \
        item_two + \
        item_three
```

## 最佳实践

1. 保持一致的缩进风格（推荐使用4个空格）
2. 适当使用空行分隔代码块，提高可读性
3. 注释要简明扼要，说明代码的"为什么"而不是"是什么"
4. 避免过长的代码行（PEP 8建议每行不超过79个字符）

## 下一步

现在您已经了解了Python代码的基本结构，接下来我们将学习Python中的变量和数据类型。
