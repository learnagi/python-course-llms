# 第三章：数据类型和变量进阶

在本章中，我们将深入学习Python中的复杂数据类型，这些数据类型在处理LLM API返回的数据时非常重要。

## 1. 列表（List）

列表是Python中最常用的数据类型之一，用于存储一系列的值。

### 创建和访问列表
```python
# 创建列表
fruits = ["apple", "banana", "orange"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# 访问列表元素（索引从0开始）
print(fruits[0])      # 输出: "apple"
print(fruits[-1])     # 输出: "orange"（倒数第一个）
```

### 列表操作
```python
# 修改列表元素
fruits[1] = "grape"

# 添加元素
fruits.append("mango")        # 在末尾添加
fruits.insert(1, "pear")     # 在指定位置插入

# 删除元素
fruits.remove("apple")       # 删除指定元素
popped = fruits.pop()       # 删除并返回最后一个元素
del fruits[0]              # 删除指定位置的元素

# 列表切片
numbers = [1, 2, 3, 4, 5]
print(numbers[1:3])    # 输出: [2, 3]
print(numbers[:3])     # 输出: [1, 2, 3]
print(numbers[2:])     # 输出: [3, 4, 5]
```

### 列表方法
```python
# 常用列表方法
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
numbers.sort()              # 排序
numbers.reverse()           # 反转
print(len(numbers))        # 获取长度
print(numbers.count(5))    # 计算元素出现次数
```

## 2. 字典（Dictionary）

字典用于存储键值对，在处理API返回的JSON数据时特别有用。

### 创建和访问字典
```python
# 创建字典
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}

# 访问字典值
print(person["name"])           # 使用键访问值
print(person.get("age"))       # 使用get方法（更安全）
print(person.get("country", "China"))  # 设置默认值
```

### 字典操作
```python
# 修改和添加元素
person["age"] = 26             # 修改现有键的值
person["occupation"] = "Engineer"  # 添加新的键值对

# 删除元素
del person["city"]             # 删除指定键值对
occupation = person.pop("occupation")  # 删除并返回值
person.clear()                 # 清空字典

# 字典方法
keys = person.keys()           # 获取所有键
values = person.values()       # 获取所有值
items = person.items()         # 获取所有键值对
```

### 嵌套字典
```python
# 复杂的字典结构（常见于API响应）
response = {
    "status": "success",
    "data": {
        "user": {
            "name": "Alice",
            "settings": {
                "theme": "dark",
                "notifications": True
            }
        }
    }
}

# 访问嵌套值
theme = response["data"]["user"]["settings"]["theme"]
```

## 3. 元组（Tuple）

元组是不可变的列表，一旦创建就不能修改。

```python
# 创建元组
coordinates = (10, 20)
single_item = (1,)    # 单个元素的元组需要加逗号

# 访问元组
x = coordinates[0]
y = coordinates[1]

# 元组解包
x, y = coordinates    # 将元组的值赋给多个变量
```

## 4. 集合（Set）

集合是无序的，不重复的元素集合。

```python
# 创建集合
numbers = {1, 2, 3, 4, 5}
fruits = set(["apple", "banana", "orange"])

# 集合操作
numbers.add(6)        # 添加元素
numbers.remove(1)     # 删除元素
numbers.discard(10)   # 删除元素（如果存在）

# 集合运算
set1 = {1, 2, 3}
set2 = {3, 4, 5}
print(set1 | set2)    # 并集
print(set1 & set2)    # 交集
print(set1 - set2)    # 差集
```

## 5. 实际应用示例

### 处理LLM API响应
```python
# 模拟OpenAI API响应
api_response = {
    "id": "chatcmpl-123",
    "object": "chat.completion",
    "created": 1677858242,
    "model": "gpt-3.5-turbo-0613",
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "Hello! How can I help you today?"
            },
            "finish_reason": "stop",
            "index": 0
        }
    ]
}

# 提取消息内容
message = api_response["choices"][0]["message"]["content"]
print(message)

# 处理多轮对话历史
conversation = [
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好！有什么我可以帮你的吗？"},
    {"role": "user", "content": "请介绍一下Python"}
]

# 添加新的对话
conversation.append({
    "role": "assistant",
    "content": "Python是一种简单易学的编程语言..."
})
```

## 练习题

1. 创建一个简单的联系人管理系统：
```python
# 创建文件 contacts.py
contacts = {}

def add_contact():
    name = input("输入联系人姓名：")
    phone = input("输入联系人电话：")
    contacts[name] = phone
    print("联系人添加成功！")

def view_contacts():
    for name, phone in contacts.items():
        print(f"{name}: {phone}")

# 主循环
while True:
    print("\n1. 添加联系人")
    print("2. 查看所有联系人")
    print("3. 退出")
    
    choice = input("请选择操作（1-3）：")
    
    if choice == "1":
        add_contact()
    elif choice == "2":
        view_contacts()
    elif choice == "3":
        break
```

## 小结

本章我们学习了：
- 列表的创建和操作
- 字典的使用和嵌套结构
- 元组的基本概念
- 集合的操作和应用
- 如何处理复杂的数据结构

这些数据类型在处理LLM API返回的数据时都非常重要，特别是字典和列表的操作。

## 下一步

在下一章中，我们将学习函数和模块的使用，这将帮助我们更好地组织代码和重用功能。

## 额外资源
- [Python数据结构文档](https://docs.python.org/zh-cn/3/tutorial/datastructures.html)
- [Python字典使用指南](https://realpython.com/python-dicts/)
