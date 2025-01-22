# 第十三章：面向对象编程

本章将介绍Python的面向对象编程概念，包括类和对象、继承和多态、封装和抽象等核心概念。

## 1. 类和对象基础

### 类的定义和实例化
```python
class ChatBot:
    """聊天机器人类"""
    def __init__(self, name: str):
        self.name = name
        self.history = []
    
    def say_hello(self) -> str:
        return f"你好，我是{self.name}！"
    
    def remember(self, message: str):
        self.history.append(message)
    
    def get_history(self) -> list:
        return self.history

# 创建实例
bot = ChatBot("小助手")
print(bot.say_hello())  # 输出：你好，我是小助手！
```

### 属性和方法
```python
class Temperature:
    """温度类，展示属性和方法的使用"""
    def __init__(self, celsius: float):
        self._celsius = celsius  # 受保护的属性
    
    @property
    def celsius(self) -> float:
        """摄氏度属性"""
        return self._celsius
    
    @property
    def fahrenheit(self) -> float:
        """华氏度属性"""
        return (self._celsius * 9/5) + 32
    
    def set_temperature(self, celsius: float):
        """设置温度"""
        if -273.15 <= celsius <= 100:
            self._celsius = celsius
        else:
            raise ValueError("温度超出有效范围")

# 使用示例
temp = Temperature(25)
print(f"摄氏度：{temp.celsius}°C")
print(f"华氏度：{temp.fahrenheit}°F")
```

## 2. 继承和多态

### 基类和派生类
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    """动物基类"""
    def __init__(self, name: str):
        self.name = name
    
    @abstractmethod
    def make_sound(self) -> str:
        """发出声音（抽象方法）"""
        pass
    
    def introduce(self) -> str:
        return f"我是{self.name}"

class Dog(Animal):
    """狗类"""
    def make_sound(self) -> str:
        return "汪汪！"

class Cat(Animal):
    """猫类"""
    def make_sound(self) -> str:
        return "喵喵！"

# 使用示例
def animal_sound(animal: Animal):
    """展示多态性"""
    print(f"{animal.introduce()}，{animal.make_sound()}")

dog = Dog("小黑")
cat = Cat("咪咪")
animal_sound(dog)  # 输出：我是小黑，汪汪！
animal_sound(cat)  # 输出：我是咪咪，喵喵！
```

### 方法重写
```python
class LLMModel:
    """LLM模型基类"""
    def __init__(self, model_name: str):
        self.model_name = model_name
    
    def generate(self, prompt: str) -> str:
        return f"使用{self.model_name}生成回复"

class GPT3(LLMModel):
    """GPT-3模型"""
    def generate(self, prompt: str) -> str:
        # 重写父类方法
        return f"GPT-3生成：{prompt}的回复"

class GPT4(LLMModel):
    """GPT-4模型"""
    def generate(self, prompt: str) -> str:
        # 重写父类方法
        return f"GPT-4生成：{prompt}的高级回复"

# 使用示例
models = [GPT3("gpt-3"), GPT4("gpt-4")]
for model in models:
    print(model.generate("你好"))
```

## 3. 封装和抽象

### 访问控制
```python
class BankAccount:
    """银行账户类，展示封装"""
    def __init__(self, account_number: str, balance: float = 0):
        self.__account_number = account_number  # 私有属性
        self.__balance = balance  # 私有属性
    
    def deposit(self, amount: float):
        """存款"""
        if amount > 0:
            self.__balance += amount
            return True
        return False
    
    def withdraw(self, amount: float):
        """取款"""
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
    
    def get_balance(self) -> float:
        """获取余额"""
        return self.__balance
    
    def get_account_info(self) -> dict:
        """获取账户信息"""
        return {
            "account_number": self.__account_number[-4:],  # 只显示后4位
            "balance": self.__balance
        }

# 使用示例
account = BankAccount("1234567890", 1000)
account.deposit(500)
print(account.get_account_info())
```

### 抽象基类
```python
from abc import ABC, abstractmethod
from typing import List

class DataProcessor(ABC):
    """数据处理器抽象基类"""
    @abstractmethod
    def process(self, data: List) -> List:
        """处理数据"""
        pass
    
    @abstractmethod
    def validate(self, data: List) -> bool:
        """验证数据"""
        pass

class TextProcessor(DataProcessor):
    """文本处理器"""
    def process(self, data: List[str]) -> List[str]:
        return [text.strip().lower() for text in data]
    
    def validate(self, data: List) -> bool:
        return all(isinstance(item, str) for item in data)

class NumberProcessor(DataProcessor):
    """数字处理器"""
    def process(self, data: List[float]) -> List[float]:
        return [round(num, 2) for num in data]
    
    def validate(self, data: List) -> bool:
        return all(isinstance(item, (int, float)) for item in data)

# 使用示例
text_processor = TextProcessor()
texts = ["Hello", "World", "Python"]
processed_texts = text_processor.process(texts)
print(processed_texts)
```

## 4. 魔术方法

### 常用魔术方法
```python
class Vector:
    """向量类，展示魔术方法的使用"""
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __str__(self) -> str:
        """字符串表示"""
        return f"Vector({self.x}, {self.y})"
    
    def __repr__(self) -> str:
        """开发者字符串表示"""
        return f"Vector(x={self.x}, y={self.y})"
    
    def __add__(self, other: 'Vector') -> 'Vector':
        """向量加法"""
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other: 'Vector') -> 'Vector':
        """向量减法"""
        return Vector(self.x - other.x, self.y - other.y)
    
    def __eq__(self, other: 'Vector') -> bool:
        """相等比较"""
        return self.x == other.x and self.y == other.y
    
    def __len__(self) -> int:
        """向量长度"""
        return int((self.x ** 2 + self.y ** 2) ** 0.5)

# 使用示例
v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # 输出：Vector(4, 6)
print(len(v1))  # 输出：2
```

### 上下文管理器
```python
class FileLogger:
    """文件日志记录器，展示上下文管理器"""
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
    
    def __enter__(self):
        """进入上下文"""
        self.file = open(self.filename, 'a')
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出上下文"""
        if self.file:
            self.file.close()
    
    def log(self, message: str):
        """记录日志"""
        if self.file:
            self.file.write(f"{message}\n")

# 使用示例
with FileLogger("app.log") as logger:
    logger.log("应用启动")
    logger.log("处理请求")
    logger.log("应用关闭")
```

## 5. 设计模式实例

### 单例模式
```python
class ConfigManager:
    """配置管理器（单例模式）"""
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self.config = {}
            self._initialized = True
    
    def set_config(self, key: str, value: any):
        """设置配置"""
        self.config[key] = value
    
    def get_config(self, key: str) -> any:
        """获取配置"""
        return self.config.get(key)

# 使用示例
config1 = ConfigManager()
config1.set_config("api_key", "abc123")

config2 = ConfigManager()
print(config2.get_config("api_key"))  # 输出：abc123
```

### 工厂模式
```python
class ModelFactory:
    """模型工厂类"""
    @staticmethod
    def create_model(model_type: str) -> LLMModel:
        """创建模型实例"""
        if model_type.lower() == "gpt3":
            return GPT3("gpt-3")
        elif model_type.lower() == "gpt4":
            return GPT4("gpt-4")
        else:
            raise ValueError(f"不支持的模型类型：{model_type}")

# 使用示例
factory = ModelFactory()
model = factory.create_model("gpt3")
print(model.generate("你好"))
```

## 6. 练习题

1. 实现一个聊天系统：
```python
class User:
    """用户类"""
    def __init__(self, name: str):
        self.name = name
        self.messages = []
    
    def send_message(self, content: str):
        """发送消息"""
        pass

class ChatRoom:
    """聊天室类"""
    def __init__(self):
        self.users = {}
    
    def add_user(self, user: User):
        """添加用户"""
        pass
    
    def broadcast(self, sender: User, message: str):
        """广播消息"""
        pass

# 完成上述类的实现
```

2. 实现一个文件处理系统：
```python
class FileHandler:
    """文件处理器基类"""
    @abstractmethod
    def read(self, filename: str) -> str:
        pass
    
    @abstractmethod
    def write(self, filename: str, content: str):
        pass

class TextFileHandler(FileHandler):
    """文本文件处理器"""
    pass

class JSONFileHandler(FileHandler):
    """JSON文件处理器"""
    pass

# 实现这些类
```

## 小结

本章我们学习了：
- 类和对象的基本概念
- 继承和多态的使用
- 封装和抽象的实现
- 魔术方法的应用
- 常见设计模式的实现

这些面向对象编程的概念将帮助你更好地组织和管理代码。

## 下一步

在下一章中，我们将学习异步编程，探索如何使用Python的async/await语法构建高效的异步应用。

## 额外资源
- [Python官方文档 - 类](https://docs.python.org/zh-cn/3/tutorial/classes.html)
- [Python设计模式](https://python-patterns.guide/)
- [Python魔术方法指南](https://rszalski.github.io/magicmethods/)
