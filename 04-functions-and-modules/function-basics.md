---
title: "函数基础"
slug: "function-basics"
sequence: 1
description: "学习Python函数的基本概念，包括函数的定义、调用、参数传递和返回值"
status: "published"
---

# 函数基础

函数是Python中最基本的代码复用单位，本节将介绍函数的基础知识。

## 函数的定义和调用

### 基本语法
```python
def function_name(parameter1, parameter2):
    """函数文档字符串：描述函数的功能"""
    # 函数体
    return result

# 调用函数
result = function_name(arg1, arg2)
```

### 简单示例
```python
def greet(name):
    """向指定的人打招呼"""
    return f"Hello, {name}!"

# 调用函数
message = greet("Alice")
print(message)  # 输出: Hello, Alice!
```

## 参数和返回值

### 位置参数
```python
def add(x, y):
    """计算两个数的和"""
    return x + y

result = add(3, 5)  # x=3, y=5
print(result)  # 输出: 8
```

### 关键字参数
```python
def create_user(name, age, city="Beijing"):
    """创建用户信息"""
    return {
        "name": name,
        "age": age,
        "city": city
    }

# 使用关键字参数
user1 = create_user(name="Alice", age=25)
user2 = create_user(name="Bob", age=30, city="Shanghai")
```

### 默认参数值
```python
def power(x, n=2):
    """计算x的n次方"""
    return x ** n

print(power(3))     # 输出: 9 (3^2)
print(power(3, 3))  # 输出: 27 (3^3)
```

### 可变参数
```python
def sum_numbers(*args):
    """计算任意数量参数的和"""
    return sum(args)

print(sum_numbers(1, 2, 3))       # 输出: 6
print(sum_numbers(1, 2, 3, 4, 5)) # 输出: 15
```

### 关键字可变参数
```python
def create_profile(**kwargs):
    """创建用户配置文件"""
    return kwargs

profile = create_profile(name="Alice", age=25, city="Beijing", hobby="reading")
print(profile)  # 输出: {'name': 'Alice', 'age': 25, 'city': 'Beijing', 'hobby': 'reading'}
```

## 函数文档

### 文档字符串
```python
def validate_api_key(api_key: str) -> bool:
    """
    验证API密钥是否有效。

    Args:
        api_key (str): 要验证的API密钥

    Returns:
        bool: 如果密钥有效返回True，否则返回False

    Raises:
        ValueError: 如果api_key为空
    """
    if not api_key:
        raise ValueError("API key cannot be empty")
    
    # 这里应该是实际的验证逻辑
    return len(api_key) >= 32
```

### 类型注解
```python
from typing import List, Dict, Optional

def process_response(
    response: Dict[str, any],
    max_tokens: Optional[int] = None
) -> List[str]:
    """
    处理API响应数据。

    Args:
        response: API响应的字典
        max_tokens: 最大标记数，可选

    Returns:
        处理后的文本列表
    """
    results = []
    choices = response.get("choices", [])
    
    for choice in choices:
        text = choice.get("text", "").strip()
        if text and (not max_tokens or len(text.split()) <= max_tokens):
            results.append(text)
    
    return results
```

## 作用域和命名空间

### 局部作用域
```python
def process_data():
    # 局部变量
    data = [1, 2, 3]
    result = sum(data)
    return result

# print(data)  # 错误：data在这里不可访问
```

### 全局作用域
```python
# 全局变量
API_KEY = "your-api-key"

def call_api():
    # 使用全局变量
    print(f"Using API key: {API_KEY}")

def update_api_key():
    # 修改全局变量需要声明
    global API_KEY
    API_KEY = "new-api-key"
```

### 闭包作用域
```python
def create_counter():
    count = 0  # 闭包变量
    
    def increment():
        nonlocal count  # 声明非局部变量
        count += 1
        return count
    
    return increment

# 创建计数器
counter = create_counter()
print(counter())  # 输出: 1
print(counter())  # 输出: 2
```

## 最佳实践

1. 函数命名：
   - 使用小写字母和下划线
   - 名称应该清晰表达功能
   - 动词开头（如get_、process_、update_）

2. 参数设计：
   - 位置参数在前，默认参数在后
   - 使用类型注解提高代码可读性
   - 合理使用可选参数

3. 返回值：
   - 保持返回值类型一致
   - 使用类型注解说明返回值
   - 考虑错误情况的返回值

4. 文档和注释：
   - 每个函数都写文档字符串
   - 说明参数和返回值
   - 注明可能的异常

## 实际应用示例

### API调用函数
```python
from typing import Dict, Optional
import requests
from time import sleep

def call_llm_api(
    prompt: str,
    model: str = "gpt-3.5-turbo",
    max_tokens: Optional[int] = None,
    temperature: float = 0.7,
    retry_count: int = 3
) -> Dict:
    """
    调用LLM API生成文本。

    Args:
        prompt: 输入提示
        model: 模型名称
        max_tokens: 最大生成标记数
        temperature: 温度参数
        retry_count: 重试次数

    Returns:
        API响应数据

    Raises:
        requests.RequestException: 当API调用失败时
    """
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    
    data = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
        "temperature": temperature
    }
    
    if max_tokens:
        data["max_tokens"] = max_tokens
    
    for attempt in range(retry_count):
        try:
            response = requests.post(
                "https://api.openai.com/v1/chat/completions",
                headers=headers,
                json=data
            )
            response.raise_for_status()
            return response.json()
        
        except requests.RequestException as e:
            if attempt == retry_count - 1:
                raise
            sleep(2 ** attempt)  # 指数退避
    
    return {}  # 不应该到达这里
```

## 下一步

现在您已经掌握了Python函数的基础知识，接下来我们将学习更高级的函数特性，如装饰器和生成器。
