---
title: "Python运算符"
slug: "operators"
sequence: 3
description: "学习Python中的各种运算符，包括算术运算符、比较运算符和逻辑运算符"
status: "published"
---

# Python运算符

运算符是用来执行特定操作的符号。Python提供了多种类型的运算符，本节将详细介绍它们的使用方法。

## 算术运算符

算术运算符用于执行基本的数学运算：

```python
a = 10
b = 3

print(a + b)   # 加法：13
print(a - b)   # 减法：7
print(a * b)   # 乘法：30
print(a / b)   # 除法：3.333...
print(a // b)  # 整除：3
print(a % b)   # 取余：1
print(a ** b)  # 幂运算：1000
```

### 复合赋值运算符
```python
x = 5
x += 3   # 等同于 x = x + 3
x -= 2   # 等同于 x = x - 2
x *= 4   # 等同于 x = x * 4
x /= 2   # 等同于 x = x / 2
x //= 3  # 等同于 x = x // 3
x %= 3   # 等同于 x = x % 3
x **= 2  # 等同于 x = x ** 2
```

## 比较运算符

比较运算符用于比较两个值，返回布尔值（True或False）：

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

### 链式比较
Python支持链式比较操作：

```python
age = 25
print(18 <= age <= 65)  # 检查age是否在18到65之间
```

## 逻辑运算符

逻辑运算符用于组合条件：

```python
a = True
b = False

print(a and b)  # 与：False（两个都为True才返回True）
print(a or b)   # 或：True（有一个为True就返回True）
print(not a)    # 非：False（取反）
```

### 短路运算
Python的逻辑运算符使用短路运算：

```python
# and 的短路运算
result = False and print("不会执行")  # print不会执行

# or 的短路运算
result = True or print("不会执行")    # print不会执行
```

## 成员运算符

用于测试序列中是否包含某个值：

```python
numbers = [1, 2, 3, 4, 5]
print(3 in numbers)     # True
print(6 not in numbers) # True

text = "Hello"
print("H" in text)      # True
print("x" not in text)  # True
```

## 身份运算符

用于比较两个对象是否是同一个对象：

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a is c)      # True（a和c指向同一个对象）
print(a is b)      # False（a和b是不同对象）
print(a is not b)  # True
```

## 运算符优先级

Python运算符有优先级顺序，从高到低：

1. 幂运算 (`**`)
2. 乘、除、整除、取余 (`*`, `/`, `//`, `%`)
3. 加减 (`+`, `-`)
4. 比较运算符 (`==`, `!=`, `<`, `>`, `<=`, `>=`)
5. 逻辑运算符 (`not`, `and`, `or`)

```python
# 运算符优先级示例
result = 2 + 3 * 4    # 14（不是20）
result = (2 + 3) * 4  # 20（使用括号改变优先级）
```

## 最佳实践

1. 使用括号明确表示运算优先级，提高代码可读性
2. 注意除法运算的结果类型（`/`得到float，`//`得到int）
3. 谨慎使用身份运算符，大多数情况下应使用相等运算符
4. 利用短路运算特性优化代码性能

## 下一步

现在您已经掌握了Python的各种运算符，接下来我们将学习Python的输入和输出操作。
