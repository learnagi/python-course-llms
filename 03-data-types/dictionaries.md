---
title: "Python字典使用"
slug: "dictionaries"
sequence: 2
description: "深入学习Python字典的创建、访问、嵌套结构和常用操作方法，特别是在处理API响应时的应用"
status: "published"
---

# Python字典使用

字典是Python中的一种键值对数据结构，它在处理JSON格式的API响应时特别有用。本节将详细介绍字典的使用方法。

## 字典的创建和访问

### 创建字典
```python
# 创建空字典
empty_dict = {}
empty_dict2 = dict()

# 创建包含键值对的字典
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}

# 使用dict()函数创建
person2 = dict(name="Bob", age=30, city="Shanghai")
```

### 访问字典值
```python
person = {"name": "Alice", "age": 25, "city": "Beijing"}

# 使用键访问值
print(person["name"])           # 输出: "Alice"

# 使用get()方法（更安全）
print(person.get("age"))       # 输出: 25
print(person.get("country"))   # 输出: None
print(person.get("country", "China"))  # 输出: "China"（设置默认值）
```

## 字典的修改

### 添加和修改键值对
```python
person = {"name": "Alice", "age": 25}

# 添加新键值对
person["city"] = "Beijing"
person["email"] = "alice@example.com"

# 修改现有键值对
person["age"] = 26
person.update({"age": 27, "phone": "123456"})
```

### 删除键值对
```python
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing",
    "email": "alice@example.com"
}

# 删除指定键值对
del person["email"]

# 删除并返回值
age = person.pop("age")
print(age)  # 输出: 25

# 删除并返回最后插入的键值对（Python 3.7+保证字典有序）
last_item = person.popitem()
print(last_item)  # 输出: ("city", "Beijing")

# 清空字典
person.clear()
```

## 字典的常用操作

### 遍历字典
```python
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}

# 遍历键
for key in person:
    print(key, person[key])

# 遍历键值对
for key, value in person.items():
    print(f"{key}: {value}")

# 遍历值
for value in person.values():
    print(value)

# 遍历键
for key in person.keys():
    print(key)
```

### 字典推导式
```python
# 创建平方数字典
squares = {x: x**2 for x in range(5)}
print(squares)  # 输出: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# 条件字典推导式
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # 输出: {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```

## 嵌套字典

### 创建嵌套字典
```python
# 复杂的字典结构（常见于API响应）
user_data = {
    "id": "user123",
    "profile": {
        "name": {
            "first": "Alice",
            "last": "Smith"
        },
        "location": {
            "city": "Beijing",
            "country": "China"
        },
        "preferences": {
            "theme": "dark",
            "notifications": {
                "email": True,
                "sms": False
            }
        }
    }
}
```

### 访问嵌套值
```python
# 使用多级键访问
print(user_data["profile"]["name"]["first"])  # 输出: "Alice"

# 使用get()方法安全访问
email_enabled = user_data.get("profile", {}).get("preferences", {}).get("notifications", {}).get("email")
print(email_enabled)  # 输出: True
```

## 实际应用：处理API响应

### OpenAI API响应示例
```python
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

# 安全地提取数据
def get_message_content(response):
    try:
        return response["choices"][0]["message"]["content"]
    except (KeyError, IndexError):
        return None
```

## 最佳实践

1. 键的命名：
   - 使用有意义的键名
   - 保持命名风格一致（全小写或下划线分隔）

2. 安全访问：
   - 优先使用get()方法而不是直接索引
   - 处理嵌套字典时使用链式get()调用

3. 性能考虑：
   - 避免频繁修改大型字典
   - 使用字典推导式代替循环创建字典

4. 数据验证：
   - 在处理API响应时总是验证数据结构
   - 使用try-except处理可能的KeyError

## 下一步

现在您已经掌握了Python字典的使用方法，接下来我们将学习元组和集合这两种数据类型，它们在特定场景下有着独特的优势。
