---
title: "函数进阶"
slug: "advanced-functions"
sequence: 2
description: "深入学习Python的高级函数特性，包括装饰器、闭包、生成器和函数式编程"
status: "published"
---

# 函数进阶

本节将介绍Python中的高级函数特性，这些特性在构建复杂的LLM应用时非常有用。

## 装饰器

装饰器是Python中强大的代码复用机制，可以在不修改原函数的情况下增加新功能。

### 基本装饰器
```python
import time
from functools import wraps

def timer(func):
    """测量函数执行时间的装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.2f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"

# 使用装饰器
result = slow_function()  # 输出执行时间
```

### 带参数的装饰器
```python
def retry(max_attempts=3, delay=1):
    """带重试机制的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            attempts = 0
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    attempts += 1
                    if attempts == max_attempts:
                        raise
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def call_api():
    # 模拟API调用
    if random.random() < 0.5:
        raise ConnectionError("API call failed")
    return "Success"
```

### 多个装饰器
```python
def log_call(func):
    """记录函数调用的装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@timer
@log_call
def process_data():
    time.sleep(0.5)
    return "Data processed"
```

## 闭包

闭包是一个函数对象，它记住了在其定义环境中的值。

### 基本闭包
```python
def create_multiplier(factor):
    """创建一个乘法器"""
    def multiplier(x):
        return x * factor
    return multiplier

# 创建闭包
double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))  # 输出: 10
print(triple(5))  # 输出: 15
```

### 带状态的闭包
```python
def create_counter(initial=0):
    """创建一个计数器"""
    count = initial
    
    def increment(step=1):
        nonlocal count
        count += step
        return count
    
    def get_count():
        return count
    
    return increment, get_count

# 使用闭包
increment, get_count = create_counter(10)
print(increment())      # 输出: 11
print(increment(2))     # 输出: 13
print(get_count())      # 输出: 13
```

## 生成器

生成器是一种特殊的迭代器，可以按需生成值，节省内存。

### 生成器函数
```python
def fibonacci(n):
    """生成斐波那契数列的生成器"""
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# 使用生成器
for num in fibonacci(10):
    print(num, end=" ")  # 输出: 0 1 1 2 3 5 8 13 21 34
```

### 生成器表达式
```python
# 生成平方数
squares = (x**2 for x in range(10))

# 使用生成器表达式
print(list(squares))  # 输出: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### 无限生成器
```python
def token_counter(text, chunk_size=100):
    """分块计算文本的token数量"""
    start = 0
    text_length = len(text)
    
    while start < text_length:
        end = start + chunk_size
        chunk = text[start:end]
        token_count = len(chunk.split())
        yield token_count
        start = end

# 使用生成器
text = "This is a long text..." * 1000
for count in token_counter(text):
    print(f"Chunk token count: {count}")
```

## 函数式编程

Python支持函数式编程范式，提供了一些有用的内置函数。

### map函数
```python
# 将列表中的每个元素转换为大写
names = ["alice", "bob", "charlie"]
upper_names = list(map(str.upper, names))
print(upper_names)  # 输出: ['ALICE', 'BOB', 'CHARLIE']

# 使用lambda函数
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers))
print(squares)  # 输出: [1, 4, 9, 16, 25]
```

### filter函数
```python
# 过滤偶数
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # 输出: [2, 4, 6, 8, 10]

# 过滤非空字符串
strings = ["", "hello", None, "world", "  ", "python"]
valid_strings = list(filter(bool, strings))
print(valid_strings)  # 输出: ['hello', 'world', '  ', 'python']
```

### reduce函数
```python
from functools import reduce

# 计算列表元素的乘积
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 输出: 120

# 连接字符串
words = ["Hello", "World", "Python"]
sentence = reduce(lambda x, y: f"{x} {y}", words)
print(sentence)  # 输出: "Hello World Python"
```

## 实际应用示例

### API速率限制装饰器
```python
import time
from collections import deque
from functools import wraps

def rate_limit(calls=60, period=60):
    """
    实现API调用速率限制的装饰器。
    
    Args:
        calls: 允许的最大调用次数
        period: 时间周期（秒）
    """
    timestamps = deque()

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            
            # 移除过期的时间戳
            while timestamps and now - timestamps[0] > period:
                timestamps.popleft()
            
            # 检查是否超过速率限制
            if len(timestamps) >= calls:
                sleep_time = timestamps[0] + period - now
                if sleep_time > 0:
                    time.sleep(sleep_time)
            
            # 添加新的时间戳
            timestamps.append(now)
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls=2, period=10)
def call_api():
    print(f"API called at {time.strftime('%H:%M:%S')}")

# 测试速率限制
for _ in range(5):
    call_api()
```

### 带缓存的API调用
```python
from functools import lru_cache
import hashlib
import json

@lru_cache(maxsize=100)
def cached_api_call(prompt: str, **params):
    """
    带缓存的API调用函数。
    
    Args:
        prompt: 输入提示
        **params: API参数
    """
    # 创建缓存键
    cache_key = hashlib.md5(
        json.dumps({"prompt": prompt, "params": params}, sort_keys=True).encode()
    ).hexdigest()
    
    # 这里应该是实际的API调用
    return f"Response for {prompt}"

# 使用缓存的API调用
result1 = cached_api_call("Hello", temperature=0.7)
result2 = cached_api_call("Hello", temperature=0.7)  # 使用缓存
```

## 最佳实践

1. 装饰器使用：
   - 使用functools.wraps保持函数元数据
   - 合理使用装饰器参数
   - 注意装饰器的执行顺序

2. 闭包设计：
   - 明确变量作用域
   - 适当使用nonlocal关键字
   - 避免过度使用闭包

3. 生成器应用：
   - 处理大数据时优先使用生成器
   - 注意生成器的一次性使用特性
   - 合理设计生成器的终止条件

4. 函数式编程：
   - 适度使用函数式特性
   - 保持代码可读性
   - 注意性能影响

## 下一步

现在您已经掌握了Python的高级函数特性，接下来我们将学习如何组织和使用Python模块。
