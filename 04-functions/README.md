# 第四章：函数和模块

在本章中，我们将学习如何使用函数组织代码，以及如何使用模块复用代码。这些概念对于构建可维护的LLM应用至关重要。

## 1. 函数基础

### 函数定义和调用
```python
# 基本函数定义
def greet():
    print("Hello, World!")

# 调用函数
greet()

# 带参数的函数
def greet_user(name):
    print(f"Hello, {name}!")

greet_user("Alice")
```

### 返回值
```python
# 返回单个值
def calculate_square(number):
    return number * number

result = calculate_square(5)  # result = 25

# 返回多个值
def get_coordinates():
    return 10, 20

x, y = get_coordinates()  # x = 10, y = 20
```

### 参数类型
```python
# 必需参数
def divide(a, b):
    return a / b

# 默认参数
def power(base, exponent=2):
    return base ** exponent

print(power(3))      # 3² = 9
print(power(3, 3))   # 3³ = 27

# 关键字参数
def greet_person(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet_person(greeting="Hi", name="Bob")
```

## 2. 高级函数特性

### 可变参数
```python
# *args：接收任意数量的位置参数
def sum_all(*numbers):
    return sum(numbers)

print(sum_all(1, 2, 3, 4))  # 10

# **kwargs：接收任意数量的关键字参数
def print_info(**info):
    for key, value in info.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
```

### Lambda函数
```python
# 简单的匿名函数
square = lambda x: x * x
print(square(5))  # 25

# 在高阶函数中使用
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x * x, numbers))
```

## 3. 模块和包

### 创建模块
```python
# llm_utils.py
def format_prompt(template, **kwargs):
    return template.format(**kwargs)

def extract_response(api_response):
    return api_response["choices"][0]["message"]["content"]

# 在其他文件中使用
import llm_utils

prompt = llm_utils.format_prompt("Hello, {name}!", name="Alice")
```

### 模块导入方式
```python
# 导入整个模块
import math
print(math.pi)

# 导入特定函数
from math import sqrt
print(sqrt(16))

# 导入并重命名
import math as m
print(m.pi)

# 导入多个函数
from math import sin, cos, tan
```

## 4. 实际应用示例

### LLM对话管理器
```python
# conversation_manager.py
class ConversationManager:
    def __init__(self):
        self.history = []
    
    def add_message(self, role, content):
        self.history.append({
            "role": role,
            "content": content
        })
    
    def get_conversation(self):
        return self.history
    
    def clear_history(self):
        self.history = []

# 使用示例
manager = ConversationManager()
manager.add_message("user", "你好")
manager.add_message("assistant", "你好！有什么我可以帮你的吗？")
```

### API调用封装
```python
# api_client.py
import requests

def call_openai_api(prompt, api_key, model="gpt-3.5-turbo"):
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    
    data = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}]
    }
    
    response = requests.post(
        "https://api.openai.com/v1/chat/completions",
        headers=headers,
        json=data
    )
    
    return response.json()
```

## 5. 错误处理

### try-except 语句
```python
def safe_divide(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        return "错误：除数不能为零"
    except TypeError:
        return "错误：请确保输入的是数字"
    finally:
        print("运算完成")
```

## 练习题

1. 创建一个简单的LLM提示词管理器：
```python
# prompt_manager.py
class PromptManager:
    def __init__(self):
        self.templates = {}
    
    def add_template(self, name, template):
        self.templates[name] = template
    
    def get_prompt(self, name, **kwargs):
        if name not in self.templates:
            raise KeyError(f"模板 '{name}' 不存在")
        return self.templates[name].format(**kwargs)

# 使用示例
manager = PromptManager()
manager.add_template(
    "introduction",
    "你好，我是{name}。我是一名{occupation}。"
)

prompt = manager.get_prompt(
    "introduction",
    name="Alice",
    occupation="程序员"
)
print(prompt)
```

## 小结

本章我们学习了：
- 函数的定义和使用
- 不同类型的函数参数
- 模块的创建和导入
- 实际LLM应用中的函数封装
- 错误处理机制

这些概念将帮助你更好地组织代码，使LLM应用更加模块化和可维护。

## 下一步

在下一章中，我们将学习API调用基础，这将使我们能够实际与OpenAI等LLM服务进行交互。

## 额外资源
- [Python函数文档](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#defining-functions)
- [Python模块文档](https://docs.python.org/zh-cn/3/tutorial/modules.html)
