---
title: "Python元组和集合"
slug: "tuples-and-sets"
sequence: 3
description: "学习Python中的元组（不可变序列）和集合（无序集合）的特性和使用方法"
status: "published"
---

# Python元组和集合

本节将介绍Python中的两种重要数据类型：元组（tuple）和集合（set）。它们各自有着独特的特性和应用场景。

## 元组（Tuple）

元组是不可变的序列类型，一旦创建就不能修改。这种特性使它特别适合存储不应该被改变的数据。

### 创建元组
```python
# 创建空元组
empty_tuple = ()
empty_tuple2 = tuple()

# 创建包含元素的元组
coordinates = (10, 20)
point3d = (1, 2, 3)

# 创建单元素元组（注意逗号）
single_item = (1,)      # 正确
not_tuple = (1)        # 错误：这是一个整数

# 不使用括号创建元组
coordinates = 10, 20
point3d = 1, 2, 3
```

### 访问元组元素
```python
coordinates = (10, 20, 30)

# 使用索引
print(coordinates[0])    # 输出: 10
print(coordinates[-1])   # 输出: 30

# 切片
print(coordinates[1:])   # 输出: (20, 30)
print(coordinates[:2])   # 输出: (10, 20)
```

### 元组解包
```python
# 基本解包
x, y, z = (1, 2, 3)
print(x, y, z)          # 输出: 1 2 3

# 使用*收集多余的值
first, *rest = (1, 2, 3, 4, 5)
print(first)            # 输出: 1
print(rest)             # 输出: [2, 3, 4, 5]

# 交换变量
a, b = 10, 20
a, b = b, a            # 使用元组解包交换值
```

### 元组的方法
```python
numbers = (1, 2, 2, 3, 4, 2)

# 计数
print(numbers.count(2))   # 输出: 3

# 查找索引
print(numbers.index(3))   # 输出: 3
```

## 集合（Set）

集合是无序的、不重复元素的集合。它支持数学中的集合操作，如并集、交集等。

### 创建集合
```python
# 创建空集合
empty_set = set()      # 注意：{}创建的是空字典

# 从列表创建集合
numbers = set([1, 2, 3, 2, 1])  # 重复元素会被自动去除
print(numbers)         # 输出: {1, 2, 3}

# 使用花括号创建
fruits = {"apple", "banana", "orange"}
```

### 集合操作
```python
# 添加和删除元素
fruits = {"apple", "banana"}

# 添加元素
fruits.add("orange")

# 删除元素
fruits.remove("banana")    # 元素不存在会报错
fruits.discard("mango")   # 元素不存在不会报错

# 随机移除并返回元素
item = fruits.pop()

# 清空集合
fruits.clear()
```

### 集合运算
```python
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}

# 并集
print(set1 | set2)         # 使用运算符
print(set1.union(set2))    # 使用方法

# 交集
print(set1 & set2)         # 使用运算符
print(set1.intersection(set2))  # 使用方法

# 差集
print(set1 - set2)         # 使用运算符
print(set1.difference(set2))   # 使用方法

# 对称差集（并集减去交集）
print(set1 ^ set2)         # 使用运算符
print(set1.symmetric_difference(set2))  # 使用方法
```

### 集合关系运算
```python
set1 = {1, 2, 3}
set2 = {1, 2, 3, 4}
set3 = {5, 6, 7}

# 子集检查
print(set1 <= set2)    # True：set1是set2的子集
print(set1.issubset(set2))

# 超集检查
print(set2 >= set1)    # True：set2是set1的超集
print(set2.issuperset(set1))

# 不相交检查
print(set1.isdisjoint(set3))  # True：set1和set3没有共同元素
```

## 实际应用场景

### 使用元组的场景
```python
# 函数返回多个值
def get_coordinates():
    return (10, 20)

x, y = get_coordinates()

# 字典键
locations = {
    (0, 0): "原点",
    (1, 0): "右侧",
    (0, 1): "上方"
}

# 数据不可变性保证
constants = (3.14, 2.718, 1.414)
```

### 使用集合的场景
```python
# 去重
numbers = [1, 2, 2, 3, 3, 4, 4, 5]
unique_numbers = list(set(numbers))

# 成员检查（比列表更快）
valid_users = {"alice", "bob", "charlie"}
user = "alice"
if user in valid_users:
    print("用户存在")

# 数据去重并保持顺序（Python 3.7+）
from dict.fromkeys import fromkeys
numbers = [1, 2, 2, 3, 3, 4, 4, 5]
unique_ordered = list(dict.fromkeys(numbers))
```

## 最佳实践

1. 元组使用建议：
   - 用于表示固定数量的相关值
   - 作为字典的键（列表不能作为键）
   - 返回多个值的函数结果

2. 集合使用建议：
   - 需要去重时优先使用集合
   - 需要频繁成员检查时使用集合
   - 需要执行集合运算时使用

3. 性能考虑：
   - 集合的成员检查比列表快
   - 元组比列表占用更少的内存

## 下一步

现在您已经掌握了Python中的元组和集合，接下来我们将学习如何在实际的LLM API数据处理中应用这些数据类型。
