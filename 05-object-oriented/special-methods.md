---
title: "特殊方法"
slug: "special-methods"
sequence: 3
description: "学习Python的特殊方法和魔术方法，包括运算符重载、上下文管理器和描述符"
status: "published"
---

# 特殊方法

本节将介绍Python的特殊方法（也称为魔术方法），这些方法能让我们自定义类的行为，使其更符合Python的惯用法。

## 基本特殊方法

### 字符串表示
```python
class APIResponse:
    """API响应类"""
    
    def __init__(self, status: int, data: dict):
        self.status = status
        self.data = data
    
    def __str__(self) -> str:
        """字符串表示，用于print()"""
        return f"Status: {self.status}, Data: {self.data}"
    
    def __repr__(self) -> str:
        """开发者字符串表示"""
        return f"APIResponse(status={self.status}, data={self.data})"

# 使用字符串表示
response = APIResponse(200, {"message": "Success"})
print(str(response))   # 用户友好的输出
print(repr(response))  # 开发者友好的输出
```

### 容器方法
```python
from typing import Any, List

class MessageHistory:
    """消息历史"""
    
    def __init__(self):
        self._messages: List[str] = []
    
    def __len__(self) -> int:
        """获取消息数量"""
        return len(self._messages)
    
    def __getitem__(self, index: int) -> str:
        """获取指定位置的消息"""
        return self._messages[index]
    
    def __setitem__(self, index: int, value: str):
        """设置指定位置的消息"""
        self._messages[index] = value
    
    def __iter__(self):
        """迭代器"""
        return iter(self._messages)
    
    def __contains__(self, item: str) -> bool:
        """检查消息是否存在"""
        return item in self._messages

# 使用容器方法
history = MessageHistory()
history._messages.extend(["Hello", "Hi", "How are you?"])

print(len(history))           # 输出: 3
print(history[0])            # 输出: Hello
history[1] = "Hey"           # 修改消息
print("Hello" in history)    # 输出: True

for message in history:      # 迭代消息
    print(message)
```

## 运算符重载

### 比较运算符
```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class APIRequest:
    """API请求"""
    timestamp: datetime
    priority: int
    
    def __eq__(self, other: 'APIRequest') -> bool:
        """相等比较"""
        if not isinstance(other, APIRequest):
            return NotImplemented
        return (self.timestamp == other.timestamp and
                self.priority == other.priority)
    
    def __lt__(self, other: 'APIRequest') -> bool:
        """小于比较"""
        if not isinstance(other, APIRequest):
            return NotImplemented
        return self.priority < other.priority

# 使用比较运算符
req1 = APIRequest(datetime.now(), 1)
req2 = APIRequest(datetime.now(), 2)

print(req1 < req2)   # 输出: True
print(req1 == req2)  # 输出: False
```

### 算术运算符
```python
class TokenBucket:
    """令牌桶"""
    
    def __init__(self, tokens: int):
        self.tokens = tokens
    
    def __add__(self, other: int) -> 'TokenBucket':
        """加法运算符"""
        return TokenBucket(self.tokens + other)
    
    def __sub__(self, other: int) -> 'TokenBucket':
        """减法运算符"""
        return TokenBucket(max(0, self.tokens - other))
    
    def __iadd__(self, other: int) -> 'TokenBucket':
        """加法赋值运算符"""
        self.tokens += other
        return self
    
    def __isub__(self, other: int) -> 'TokenBucket':
        """减法赋值运算符"""
        self.tokens = max(0, self.tokens - other)
        return self

# 使用算术运算符
bucket = TokenBucket(100)
bucket += 50          # 添加令牌
print(bucket.tokens)  # 输出: 150

bucket -= 30          # 消耗令牌
print(bucket.tokens)  # 输出: 120

new_bucket = bucket + 20  # 创建新的令牌桶
print(new_bucket.tokens)  # 输出: 140
```

## 上下文管理器

### 基本用法
```python
class APIClient:
    """API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.session = None
    
    def __enter__(self):
        """进入上下文"""
        import requests
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {self.api_key}"
        })
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出上下文"""
        if self.session:
            self.session.close()
            self.session = None
        return False  # 不处理异常

# 使用上下文管理器
with APIClient("your-api-key") as client:
    # 在上下文中使用客户端
    pass  # 实际的API调用
```

### 异步上下文管理器
```python
import aiohttp
from typing import Optional

class AsyncAPIClient:
    """异步API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def __aenter__(self):
        """进入异步上下文"""
        self.session = aiohttp.ClientSession(headers={
            "Authorization": f"Bearer {self.api_key}"
        })
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """退出异步上下文"""
        if self.session:
            await self.session.close()
            self.session = None
        return False

# 使用异步上下文管理器
async def main():
    async with AsyncAPIClient("your-api-key") as client:
        # 在异步上下文中使用客户端
        pass  # 实际的异步API调用
```

## 描述符

### 数据验证
```python
class Positive:
    """正数描述符"""
    
    def __init__(self):
        self.name = None
    
    def __set_name__(self, owner, name):
        """设置描述符名称"""
        self.name = name
    
    def __get__(self, instance, owner):
        """获取值"""
        if instance is None:
            return self
        return instance.__dict__[self.name]
    
    def __set__(self, instance, value):
        """设置值"""
        if value <= 0:
            raise ValueError(f"{self.name} must be positive")
        instance.__dict__[self.name] = value

class RateLimiter:
    """速率限制器"""
    
    requests_per_minute = Positive()  # 使用描述符
    burst_limit = Positive()
    
    def __init__(
        self,
        requests_per_minute: int,
        burst_limit: int
    ):
        self.requests_per_minute = requests_per_minute
        self.burst_limit = burst_limit

# 使用描述符
limiter = RateLimiter(60, 100)
try:
    limiter.requests_per_minute = -1  # 引发ValueError
except ValueError as e:
    print(e)  # 输出: requests_per_minute must be positive
```

### 延迟属性
```python
class LazyProperty:
    """延迟加载属性描述符"""
    
    def __init__(self, func):
        self.func = func
        self.name = None
    
    def __set_name__(self, owner, name):
        """设置描述符名称"""
        self.name = name
    
    def __get__(self, instance, owner):
        """获取值"""
        if instance is None:
            return self
        
        value = self.func(instance)
        instance.__dict__[self.name] = value  # 缓存结果
        return value

class APIClient:
    """API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    @LazyProperty
    def session(self):
        """延迟创建会话"""
        import requests
        session = requests.Session()
        session.headers.update({
            "Authorization": f"Bearer {self.api_key}"
        })
        return session

# 使用延迟属性
client = APIClient("your-api-key")
# session只在首次访问时创建
print(client.session)  # 创建并返回session
print(client.session)  # 直接返回缓存的session
```

## 实际应用示例

### 配置管理器
```python
from typing import Any, Dict, Optional
import json

class Config:
    """配置管理器"""
    
    def __init__(self, data: Dict[str, Any]):
        self._data = data
    
    def __getattr__(self, name: str) -> Any:
        """获取配置项"""
        if name not in self._data:
            raise AttributeError(f"No such config: {name}")
        return self._data[name]
    
    def __setattr__(self, name: str, value: Any):
        """设置配置项"""
        if name == "_data":
            super().__setattr__(name, value)
        else:
            self._data[name] = value
    
    def __getitem__(self, key: str) -> Any:
        """字典式访问"""
        return self._data[key]
    
    def __setitem__(self, key: str, value: Any):
        """字典式设置"""
        self._data[key] = value
    
    def __contains__(self, item: str) -> bool:
        """检查配置项是否存在"""
        return item in self._data
    
    def __str__(self) -> str:
        """字符串表示"""
        return json.dumps(self._data, indent=2)

# 使用配置管理器
config = Config({
    "api_key": "your-api-key",
    "base_url": "https://api.example.com",
    "timeout": 30
})

# 属性式访问
print(config.api_key)
config.timeout = 60

# 字典式访问
print(config["base_url"])
config["retry_limit"] = 3

# 检查配置
if "api_key" in config:
    print("API key is set")

# 打印配置
print(config)
```

### 资源管理器
```python
from typing import Optional
import threading
from contextlib import contextmanager

class ResourcePool:
    """资源池"""
    
    def __init__(self, size: int):
        self.size = size
        self._available = size
        self._lock = threading.Lock()
        self._condition = threading.Condition(self._lock)
    
    def __str__(self) -> str:
        """字符串表示"""
        return f"ResourcePool(size={self.size}, available={self._available})"
    
    @contextmanager
    def acquire(self, timeout: Optional[float] = None):
        """获取资源"""
        acquired = False
        try:
            acquired = self._acquire(timeout)
            if not acquired:
                raise TimeoutError("Failed to acquire resource")
            yield
        finally:
            if acquired:
                self._release()
    
    def _acquire(self, timeout: Optional[float] = None) -> bool:
        """内部获取资源方法"""
        with self._lock:
            if timeout is None:
                while self._available == 0:
                    self._condition.wait()
            else:
                if self._available == 0:
                    self._condition.wait(timeout)
                if self._available == 0:
                    return False
            self._available -= 1
            return True
    
    def _release(self):
        """释放资源"""
        with self._lock:
            self._available += 1
            self._condition.notify()

# 使用资源池
pool = ResourcePool(2)

def use_resource(name: str):
    """使用资源的函数"""
    try:
        with pool.acquire(timeout=5):
            print(f"{name} acquired resource")
            # 模拟使用资源
            import time
            time.sleep(1)
            print(f"{name} released resource")
    except TimeoutError:
        print(f"{name} failed to acquire resource")

# 创建多个线程使用资源
threads = []
for i in range(3):
    thread = threading.Thread(
        target=use_resource,
        args=(f"Thread-{i}",)
    )
    threads.append(thread)
    thread.start()

# 等待所有线程完成
for thread in threads:
    thread.join()
```

## 最佳实践

1. 特殊方法使用：
   - 遵循Python的惯用法
   - 实现必要的方法
   - 保持一致性

2. 运算符重载：
   - 保持直观的语义
   - 处理类型检查
   - 返回合适的类型

3. 上下文管理：
   - 正确管理资源
   - 处理异常情况
   - 支持异步操作

4. 描述符应用：
   - 实现数据验证
   - 支持延迟加载
   - 复用通用逻辑

## 下一步

现在您已经掌握了Python的特殊方法，接下来我们将学习常用的设计模式。
