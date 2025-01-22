# 继承和多态

本节将介绍Python中的继承和多态概念，这是面向对象编程的重要特性。

## 类的继承

继承允许我们创建一个新类，该类基于一个已存在的类。新类（子类）继承了基类（父类）的属性和方法。

```python
class LLMBot(ChatBot):
    """基于大语言模型的聊天机器人"""
    def __init__(self, name: str, model: str):
        super().__init__(name)
        self.model = model
    
    def say_hello(self) -> str:
        return f"你好，我是{self.name}，我使用{self.model}模型！"
    
    def chat(self, message: str) -> str:
        self.remember(message)
        return f"使用{self.model}处理：{message}"
```

## 多态

多态允许我们以统一的方式处理不同类的对象：

```python
def greet_bot(bot: ChatBot):
    print(bot.say_hello())

# 可以处理ChatBot及其子类
basic_bot = ChatBot("基础助手")
llm_bot = LLMBot("智能助手", "GPT-4")

greet_bot(basic_bot)  # 输出：你好，我是基础助手！
greet_bot(llm_bot)    # 输出：你好，我是智能助手，我使用GPT-4模型！
```

## 抽象类

Python通过`abc`模块提供抽象类支持：

```python
from abc import ABC, abstractmethod

class Bot(ABC):
    @abstractmethod
    def process_message(self, message: str) -> str:
        """处理输入消息"""
        pass

class RuleBot(Bot):
    def process_message(self, message: str) -> str:
        return f"使用规则处理：{message}"

class AIBot(Bot):
    def process_message(self, message: str) -> str:
        return f"使用AI处理：{message}"
```

## 多重继承

Python支持多重继承，但需要谨慎使用：

```python
class LoggerMixin:
    def log(self, message: str):
        print(f"[{self.__class__.__name__}] {message}")

class SmartBot(LLMBot, LoggerMixin):
    def chat(self, message: str) -> str:
        self.log(f"收到消息：{message}")
        response = super().chat(message)
        self.log(f"回复消息：{response}")
        return response
```