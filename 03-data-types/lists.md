---
title: "Python列表操作"
slug: "lists"
sequence: 1
description: "深入学习Python列表的创建、访问、修改和常用操作方法"
status: "published"
---

# Python列表操作

列表是Python中最常用的数据类型之一，它可以存储一系列的值，并且这些值可以是不同的数据类型。本节将详细介绍列表的使用方法。

## 列表的创建和访问

### 创建列表
```python
# 创建空列表
empty_list = []
empty_list2 = list()

# 创建包含元素的列表
fruits = ["apple", "banana", "orange"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]  # 不同类型的元素
```

### 访问列表元素
```python
fruits = ["apple", "banana", "orange", "grape", "mango"]

# 使用索引访问（索引从0开始）
print(fruits[0])       # 输出: "apple"
print(fruits[-1])      # 输出: "mango"（倒数第一个）
print(fruits[-2])      # 输出: "grape"（倒数第二个）

# 使用切片访问
print(fruits[1:3])     # 输出: ["banana", "orange"]
print(fruits[:3])      # 输出: ["apple", "banana", "orange"]
print(fruits[2:])      # 输出: ["orange", "grape", "mango"]
print(fruits[::2])     # 输出: ["apple", "orange", "mango"]（步长为2）
```

## 列表的修改

### 修改元素
```python
fruits = ["apple", "banana", "orange"]

# 修改单个元素
fruits[1] = "grape"
print(fruits)  # 输出: ["apple", "grape", "orange"]

# 使用切片修改多个元素
fruits[1:3] = ["pear", "mango"]
print(fruits)  # 输出: ["apple", "pear", "mango"]
```

### 添加元素
```python
fruits = ["apple", "banana"]

# 在末尾添加元素
fruits.append("orange")
print(fruits)  # 输出: ["apple", "banana", "orange"]

# 在指定位置插入元素
fruits.insert(1, "grape")
print(fruits)  # 输出: ["apple", "grape", "banana", "orange"]

# 扩展列表
more_fruits = ["mango", "pear"]
fruits.extend(more_fruits)
print(fruits)  # 输出: ["apple", "grape", "banana", "orange", "mango", "pear"]
```

### 删除元素
```python
fruits = ["apple", "banana", "orange", "grape", "mango"]

# 删除指定元素
fruits.remove("banana")
print(fruits)  # 输出: ["apple", "orange", "grape", "mango"]

# 删除指定位置的元素
del fruits[1]
print(fruits)  # 输出: ["apple", "grape", "mango"]

# 删除并返回最后一个元素
last_fruit = fruits.pop()
print(last_fruit)  # 输出: "mango"
print(fruits)      # 输出: ["apple", "grape"]

# 删除指定位置的元素并返回
second_fruit = fruits.pop(1)
print(second_fruit)  # 输出: "grape"
print(fruits)        # 输出: ["apple"]
```

## 列表的常用操作

### 查找和计数
```python
fruits = ["apple", "banana", "orange", "banana", "grape"]

# 查找元素位置
print(fruits.index("banana"))       # 输出第一个"banana"的位置：1
print(fruits.count("banana"))       # 计算"banana"出现的次数：2

# 检查元素是否存在
print("apple" in fruits)            # 输出: True
print("mango" in fruits)            # 输出: False
```

### 排序和反转
```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

# 排序
numbers.sort()                      # 直接修改列表
print(numbers)                      # 输出: [1, 1, 2, 3, 3, 4, 5, 5, 6, 9]

# 反转排序
numbers.sort(reverse=True)          # 降序排序
print(numbers)                      # 输出: [9, 6, 5, 5, 4, 3, 3, 2, 1, 1]

# 不修改原列表的排序
sorted_numbers = sorted(numbers)    # 返回新列表
print(sorted_numbers)

# 反转列表顺序
numbers.reverse()
print(numbers)
```

### 列表推导式
```python
# 基本列表推导式
squares = [x**2 for x in range(5)]
print(squares)  # 输出: [0, 1, 4, 9, 16]

# 带条件的列表推导式
even_squares = [x**2 for x in range(10) if x % 2 == 0]
print(even_squares)  # 输出: [0, 4, 16, 36, 64]

# 嵌套列表推导式
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]
print(flattened)  # 输出: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## 最佳实践

1. 列表命名：
   - 使用复数形式命名（如`fruits`而不是`fruit`）
   - 名称应该清晰表达列表内容

2. 性能考虑：
   - 对于大量数据，使用列表推导式比循环更高效
   - 需要频繁插入删除操作时，考虑使用其他数据结构

3. 内存管理：
   - 使用切片时注意创建新列表的开销
   - 不再需要的大列表记得清空（`clear()`或赋值为`[]`）

## 下一步

现在您已经掌握了Python列表的基本操作，接下来我们将学习字典的使用，它是另一个在处理LLM API数据时非常重要的数据类型。
