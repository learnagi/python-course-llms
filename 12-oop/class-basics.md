# 类和对象基础

本节将介绍Python中类和对象的基础知识，这是面向对象编程的核心概念。

## 类的定义和实例化

在Python中，我们使用`class`关键字来定义类。下面是一个简单的聊天机器人类的例子：

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
```

让我们来分析这个类的结构：

1. 类名：`ChatBot`使用大驼峰命名法
2. 文档字符串：类的说明文档
3. 构造方法：`__init__`用于初始化对象
4. 实例属性：`name`和`history`
5. 实例方法：`say_hello`和`remember`

### 创建和使用对象

```python
# 创建ChatBot实例
bot = ChatBot("小助手")

# 调用实例方法
print(bot.say_hello())  # 输出：你好，我是小助手！

# 使用实例属性
bot.remember("你好啊")
print(bot.history)  # 输出：['你好啊']
```

## 访问控制

Python使用命名约定来实现属性和方法的访问控制：

- 公有成员：直接命名
- 私有成员：使用双下划线前缀`__`
- 保护成员：使用单下划线前缀`_`

```python
class Account:
    def __init__(self, owner: str, balance: float):
        self.owner = owner          # 公有属性
        self._type = "savings"      # 保护属性
        self.__balance = balance    # 私有属性
    
    def get_balance(self) -> float:
        return self.__balance
```