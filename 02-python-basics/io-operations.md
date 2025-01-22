---
title: "输入和输出操作"
slug: "io-operations"
sequence: 4
description: "学习Python中的基本输入输出操作，包括print函数、input函数和字符串格式化"
status: "published"
---

# 输入和输出操作

输入输出（I/O）操作是与用户交互的基础。本节将介绍Python中的基本输入输出方法。

## 输出操作

### print() 函数基础
`print()`函数是Python中最基本的输出方式：

```python
# 基本输出
print("Hello, World!")

# 输出多个值
print("姓名:", "张三", "年龄:", 25)

# 输出空行
print()
```

### print() 函数的参数

```python
# sep参数：指定分隔符
print("姓名", "张三", "年龄", 25, sep="|")  # 姓名|张三|年龄|25

# end参数：指定结束符
print("Hello", end=" ")
print("World")  # 输出：Hello World

# file参数：指定输出位置
with open("output.txt", "w") as f:
    print("保存到文件", file=f)
```

## 字符串格式化

### f-string（推荐）
Python 3.6+引入的格式化方法：

```python
name = "Alice"
age = 25
height = 1.68

# 基本用法
print(f"我叫{name}，今年{age}岁")

# 表达式
print(f"明年我{age + 1}岁")

# 格式化数字
print(f"身高：{height:.2f}米")
```

### str.format() 方法
传统的字符串格式化方法：

```python
# 基本用法
print("我叫{}，今年{}岁".format(name, age))

# 使用索引
print("{1}岁的{0}".format(name, age))

# 使用命名参数
print("{n}岁的{p}".format(p=name, n=age))

# 格式化数字
print("身高：{:.2f}米".format(height))
```

### % 操作符
早期的格式化方法（仍然可用）：

```python
# 基本用法
print("我叫%s，今年%d岁" % (name, age))

# 格式化数字
print("身高：%.2f米" % height)
```

## 输入操作

### input() 函数
`input()`函数用于从用户获取输入：

```python
# 基本输入
name = input("请输入你的名字：")
print(f"你好，{name}！")

# 获取数字输入（需要类型转换）
age = int(input("请输入你的年龄："))
height = float(input("请输入你的身高（米）："))

print(f"你今年{age}岁，身高{height}米")
```

### 输入验证
处理用户输入时应该进行验证：

```python
# 数字输入验证
while True:
    try:
        age = int(input("请输入年龄："))
        if 0 <= age <= 150:
            break
        print("请输入有效年龄（0-150）")
    except ValueError:
        print("请输入数字")

# 字符串输入验证
while True:
    name = input("请输入用户名（3-20个字符）：").strip()
    if 3 <= len(name) <= 20:
        break
    print("用户名长度必须在3-20个字符之间")
```

## 常见应用场景

### 用户交互界面
```python
print("==== 个人信息录入 ====")
name = input("姓名：")
age = int(input("年龄："))
height = float(input("身高（米）："))

print("\n==== 信息确认 ====")
print(f"姓名：{name}")
print(f"年龄：{age}岁")
print(f"身高：{height:.2f}米")
```

### 简单的计算器
```python
num1 = float(input("输入第一个数字："))
operator = input("输入运算符（+、-、*、/）：")
num2 = float(input("输入第二个数字："))

if operator == "+":
    result = num1 + num2
elif operator == "-":
    result = num1 - num2
elif operator == "*":
    result = num1 * num2
elif operator == "/":
    result = num1 / num2 if num2 != 0 else "错误：除数不能为0"
else:
    result = "错误：无效的运算符"

print(f"结果：{result}")
```

## 最佳实践

1. 使用f-string进行字符串格式化，代码更简洁易读
2. 始终验证用户输入，确保数据有效性
3. 提供清晰的输入提示和错误信息
4. 考虑使用异常处理来处理输入错误

## 下一步

现在您已经掌握了Python的基本输入输出操作，接下来我们将通过一些实践练习来巩固所学知识。
