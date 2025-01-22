---
title: "类和对象基础"
slug: "class-basics"
sequence: 1
description: "学习Python类和对象的基础知识，包括类的定义、属性、方法和访问控制"
status: "published"
---

# 类和对象基础

本节将介绍Python中类和对象的基础知识，这是面向对象编程的核心概念。

## 类的定义和实例化

### 基本语法
```python
class Person:
    """人员类"""
    
    def __init__(self, name: str, age: int):
        """构造函数"""
        self.name = name
        self.age = age
    
    def greet(self) -> str:
        """打招呼"""
        return f"Hello, I'm {self.name}"

# 创建对象
person = Person("Alice", 25)
print(person.greet())  # 输出: Hello, I'm Alice
```

### 类变量和实例变量
```python
class Counter:
    """计数器类"""
    
    # 类变量（所有实例共享）
    count = 0
    
    def __init__(self, name: str):
        # 实例变量（每个实例独有）
        self.name = name
        Counter.count += 1
    
    @classmethod
    def get_count(cls) -> int:
        """获取计数器数量"""
        return cls.count

# 使用类变量
counter1 = Counter("c1")
counter2 = Counter("c2")
print(Counter.get_count())  # 输出: 2
```

## 属性和方法

### 实例方法
```python
class TextProcessor:
    """文本处理器"""
    
    def __init__(self, text: str):
        self.text = text
    
    def word_count(self) -> int:
        """计算单词数"""
        return len(self.text.split())
    
    def char_count(self) -> int:
        """计算字符数"""
        return len(self.text)
    
    def to_uppercase(self) -> str:
        """转换为大写"""
        return self.text.upper()

# 使用实例方法
processor = TextProcessor("Hello World")
print(processor.word_count())    # 输出: 2
print(processor.char_count())    # 输出: 11
print(processor.to_uppercase())  # 输出: HELLO WORLD
```

### 类方法和静态方法
```python
class DateUtils:
    """日期工具类"""
    
    @classmethod
    def from_string(cls, date_str: str) -> 'DateUtils':
        """从字符串创建日期对象"""
        year, month, day = map(int, date_str.split('-'))
        return cls(year, month, day)
    
    @staticmethod
    def is_valid_date(year: int, month: int, day: int) -> bool:
        """验证日期是否有效"""
        try:
            import datetime
            datetime.datetime(year, month, day)
            return True
        except ValueError:
            return False
    
    def __init__(self, year: int, month: int, day: int):
        self.year = year
        self.month = month
        self.day = day

# 使用类方法和静态方法
date = DateUtils.from_string("2025-01-22")
print(DateUtils.is_valid_date(2025, 1, 22))  # 输出: True
```

## 属性装饰器

### @property装饰器
```python
class Circle:
    """圆形类"""
    
    def __init__(self, radius: float):
        self._radius = radius
    
    @property
    def radius(self) -> float:
        """半径属性"""
        return self._radius
    
    @radius.setter
    def radius(self, value: float):
        """设置半径"""
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value
    
    @property
    def area(self) -> float:
        """面积属性"""
        return 3.14159 * self._radius ** 2

# 使用属性
circle = Circle(5)
print(circle.radius)  # 输出: 5
circle.radius = 10    # 使用setter
print(circle.area)    # 输出: 314.159
```

## 访问控制

### 私有属性和方法
```python
class BankAccount:
    """银行账户"""
    
    def __init__(self, account_number: str, balance: float):
        self._account_number = account_number  # 受保护的属性
        self.__balance = balance               # 私有属性
    
    def deposit(self, amount: float):
        """存款"""
        if amount <= 0:
            raise ValueError("Amount must be positive")
        self.__balance += amount
    
    def withdraw(self, amount: float):
        """取款"""
        if amount <= 0:
            raise ValueError("Amount must be positive")
        if amount > self.__balance:
            raise ValueError("Insufficient funds")
        self.__balance -= amount
    
    def get_balance(self) -> float:
        """获取余额"""
        return self.__balance
    
    def __str__(self) -> str:
        """字符串表示"""
        return f"Account: {self._account_number}, Balance: {self.__balance}"

# 使用私有属性和方法
account = BankAccount("1234567890", 1000.0)
account.deposit(500)
print(account.get_balance())  # 输出: 1500.0
# print(account.__balance)    # 错误：无法直接访问私有属性
```

## 实际应用示例

### API配置管理
```python
class APIConfig:
    """API配置管理器"""
    
    def __init__(self, api_key: str, base_url: str):
        self._api_key = api_key
        self._base_url = base_url.rstrip('/')
        self._rate_limit = 60  # 默认速率限制
    
    @property
    def api_key(self) -> str:
        """API密钥"""
        return self._api_key
    
    @api_key.setter
    def api_key(self, value: str):
        """设置API密钥"""
        if not value or len(value) < 32:
            raise ValueError("Invalid API key")
        self._api_key = value
    
    @property
    def base_url(self) -> str:
        """基础URL"""
        return self._base_url
    
    @property
    def rate_limit(self) -> int:
        """速率限制"""
        return self._rate_limit
    
    @rate_limit.setter
    def rate_limit(self, value: int):
        """设置速率限制"""
        if value <= 0:
            raise ValueError("Rate limit must be positive")
        self._rate_limit = value
    
    def get_headers(self) -> dict:
        """获取HTTP头信息"""
        return {
            "Authorization": f"Bearer {self._api_key}",
            "Content-Type": "application/json"
        }
    
    def get_endpoint(self, path: str) -> str:
        """获取完整的API端点URL"""
        return f"{self._base_url}/{path.lstrip('/')}"

# 使用配置管理器
config = APIConfig(
    api_key="your-api-key",
    base_url="https://api.openai.com/v1"
)

# 设置新的速率限制
config.rate_limit = 100

# 获取API调用所需的信息
headers = config.get_headers()
endpoint = config.get_endpoint("/chat/completions")
```

### 请求跟踪器
```python
from typing import List, Dict
from datetime import datetime, timedelta

class RequestTracker:
    """API请求跟踪器"""
    
    def __init__(self, window_size: int = 60):
        self.__requests: List[datetime] = []
        self.__window_size = window_size
        self.__total_tokens = 0
        self.__total_cost = 0.0
    
    def add_request(self, tokens: int, cost: float):
        """记录新请求"""
        now = datetime.now()
        self.__cleanup(now)
        
        self.__requests.append(now)
        self.__total_tokens += tokens
        self.__total_cost += cost
    
    def __cleanup(self, current_time: datetime):
        """清理过期的请求记录"""
        cutoff = current_time - timedelta(seconds=self.__window_size)
        self.__requests = [
            req for req in self.__requests
            if req >= cutoff
        ]
    
    @property
    def request_count(self) -> int:
        """获取当前时间窗口内的请求数"""
        self.__cleanup(datetime.now())
        return len(self.__requests)
    
    @property
    def total_tokens(self) -> int:
        """获取总token数"""
        return self.__total_tokens
    
    @property
    def total_cost(self) -> float:
        """获取总成本"""
        return self.__total_cost
    
    def get_statistics(self) -> Dict:
        """获取统计信息"""
        return {
            "requests": self.request_count,
            "tokens": self.__total_tokens,
            "cost": self.__total_cost
        }

# 使用请求跟踪器
tracker = RequestTracker(window_size=3600)  # 1小时窗口

# 记录请求
tracker.add_request(tokens=100, cost=0.002)
tracker.add_request(tokens=150, cost=0.003)

# 获取统计信息
stats = tracker.get_statistics()
print(f"Requests in last hour: {stats['requests']}")
print(f"Total tokens: {stats['tokens']}")
print(f"Total cost: ${stats['cost']:.3f}")
```

## 最佳实践

1. 类的设计：
   - 遵循单一职责原则
   - 保持类的接口简单
   - 使用适当的访问控制

2. 属性管理：
   - 使用property保护属性访问
   - 合理使用私有属性
   - 提供必要的访问方法

3. 方法设计：
   - 方法名应清晰表达功能
   - 适当使用类方法和静态方法
   - 保持方法的原子性

4. 文档和注释：
   - 为类添加文档字符串
   - 说明参数和返回值
   - 注明可能的异常

## 下一步

现在您已经掌握了Python类和对象的基础知识，接下来我们将学习继承和多态的概念。
