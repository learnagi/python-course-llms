---
title: "继承和多态"
slug: "inheritance"
sequence: 2
description: "学习Python中的继承和多态概念，包括类的继承、方法重写、抽象类和接口"
status: "published"
---

# 继承和多态

本节将介绍Python中的继承和多态概念，这是面向对象编程中实现代码重用和扩展的重要机制。

## 类的继承

### 基本继承
```python
class Animal:
    """动物基类"""
    
    def __init__(self, name: str):
        self.name = name
    
    def speak(self) -> str:
        """发出声音"""
        return "Some sound"

class Dog(Animal):
    """狗类"""
    
    def speak(self) -> str:
        """狗叫"""
        return "Woof!"

class Cat(Animal):
    """猫类"""
    
    def speak(self) -> str:
        """猫叫"""
        return "Meow!"

# 使用继承
dog = Dog("Buddy")
cat = Cat("Whiskers")

print(dog.name, dog.speak())  # 输出: Buddy Woof!
print(cat.name, cat.speak())  # 输出: Whiskers Meow!
```

### 调用父类方法
```python
class Vehicle:
    """车辆基类"""
    
    def __init__(self, brand: str, model: str):
        self.brand = brand
        self.model = model
    
    def get_info(self) -> str:
        """获取车辆信息"""
        return f"{self.brand} {self.model}"

class Car(Vehicle):
    """汽车类"""
    
    def __init__(self, brand: str, model: str, seats: int):
        # 调用父类的__init__方法
        super().__init__(brand, model)
        self.seats = seats
    
    def get_info(self) -> str:
        """获取汽车信息"""
        # 调用父类的get_info方法
        base_info = super().get_info()
        return f"{base_info} ({self.seats} seats)"

# 使用父类方法
car = Car("Toyota", "Camry", 5)
print(car.get_info())  # 输出: Toyota Camry (5 seats)
```

## 抽象类和接口

### 抽象基类
```python
from abc import ABC, abstractmethod
from typing import List, Dict

class LLMClient(ABC):
    """LLM客户端抽象基类"""
    
    @abstractmethod
    def chat_completion(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ) -> Dict:
        """聊天补全"""
        pass
    
    @abstractmethod
    def embeddings(
        self,
        text: str,
        **kwargs
    ) -> List[float]:
        """文本嵌入"""
        pass

class OpenAIClient(LLMClient):
    """OpenAI客户端"""
    
    def chat_completion(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ) -> Dict:
        # 实现OpenAI的聊天补全
        return {"choices": [{"message": {"content": "Hello!"}}]}
    
    def embeddings(
        self,
        text: str,
        **kwargs
    ) -> List[float]:
        # 实现OpenAI的文本嵌入
        return [0.1, 0.2, 0.3]

# 使用抽象类
client = OpenAIClient()
response = client.chat_completion([
    {"role": "user", "content": "Hi!"}
])
```

### 多重继承
```python
class Loggable:
    """可记录日志的混入类"""
    
    def log(self, message: str):
        """记录日志"""
        print(f"[LOG] {message}")

class Serializable:
    """可序列化的混入类"""
    
    def to_dict(self) -> Dict:
        """转换为字典"""
        return self.__dict__
    
    def from_dict(self, data: Dict):
        """从字典加载"""
        self.__dict__.update(data)

class APIResponse(Loggable, Serializable):
    """API响应类"""
    
    def __init__(self, status: int, data: Dict):
        self.status = status
        self.data = data
        self.log(f"Response created with status {status}")
    
    def __str__(self) -> str:
        return f"Status: {self.status}, Data: {self.data}"

# 使用多重继承
response = APIResponse(200, {"message": "Success"})
print(response.to_dict())  # 使用Serializable的方法
response.log("Processing complete")  # 使用Loggable的方法
```

## 多态

### 函数多态
```python
from typing import Union, List

class TextProcessor:
    """文本处理器"""
    
    def process(
        self,
        text: Union[str, List[str]]
    ) -> Union[str, List[str]]:
        """处理文本"""
        if isinstance(text, str):
            return text.upper()
        elif isinstance(text, list):
            return [t.upper() for t in text]
        else:
            raise TypeError("Unsupported type")

# 使用多态
processor = TextProcessor()
print(processor.process("hello"))        # 输出: HELLO
print(processor.process(["hi", "there"])) # 输出: ['HI', 'THERE']
```

### 接口多态
```python
class DataSource(ABC):
    """数据源接口"""
    
    @abstractmethod
    def read(self) -> str:
        """读取数据"""
        pass

class FileSource(DataSource):
    """文件数据源"""
    
    def __init__(self, filename: str):
        self.filename = filename
    
    def read(self) -> str:
        with open(self.filename, 'r') as f:
            return f.read()

class APISource(DataSource):
    """API数据源"""
    
    def __init__(self, url: str):
        self.url = url
    
    def read(self) -> str:
        # 模拟API调用
        return f"Data from {self.url}"

def process_data(source: DataSource) -> str:
    """处理数据"""
    data = source.read()
    return data.upper()

# 使用接口多态
file_source = FileSource("data.txt")
api_source = APISource("https://api.example.com/data")

print(process_data(file_source))  # 处理文件数据
print(process_data(api_source))   # 处理API数据
```

## 实际应用示例

### LLM模型抽象
```python
class BaseModel(ABC):
    """LLM模型基类"""
    
    @abstractmethod
    def generate(
        self,
        prompt: str,
        **kwargs
    ) -> str:
        """生成文本"""
        pass
    
    @abstractmethod
    def get_embedding(
        self,
        text: str,
        **kwargs
    ) -> List[float]:
        """获取文本嵌入"""
        pass

class GPT3Model(BaseModel):
    """GPT-3模型"""
    
    def __init__(self, api_key: str):
        self.client = OpenAIClient(api_key)
    
    def generate(
        self,
        prompt: str,
        **kwargs
    ) -> str:
        response = self.client.chat_completion([
            {"role": "user", "content": prompt}
        ], **kwargs)
        return response["choices"][0]["message"]["content"]
    
    def get_embedding(
        self,
        text: str,
        **kwargs
    ) -> List[float]:
        return self.client.embeddings(text, **kwargs)

class ClaudeModel(BaseModel):
    """Claude模型"""
    
    def __init__(self, api_key: str):
        self.client = AnthropicClient(api_key)
    
    def generate(
        self,
        prompt: str,
        **kwargs
    ) -> str:
        response = self.client.complete(prompt, **kwargs)
        return response["completion"]
    
    def get_embedding(
        self,
        text: str,
        **kwargs
    ) -> List[float]:
        return self.client.embed(text, **kwargs)

# 使用模型抽象
def process_text(
    model: BaseModel,
    text: str
) -> tuple[str, List[float]]:
    """处理文本"""
    response = model.generate(text)
    embedding = model.get_embedding(text)
    return response, embedding

# 可以使用不同的模型
gpt3 = GPT3Model("openai-key")
claude = ClaudeModel("anthropic-key")

result_gpt3 = process_text(gpt3, "Hello!")
result_claude = process_text(claude, "Hello!")
```

### 插件系统
```python
class Plugin(ABC):
    """插件基类"""
    
    @abstractmethod
    def name(self) -> str:
        """插件名称"""
        pass
    
    @abstractmethod
    def process(
        self,
        text: str
    ) -> str:
        """处理文本"""
        pass

class TranslationPlugin(Plugin):
    """翻译插件"""
    
    def name(self) -> str:
        return "translator"
    
    def process(self, text: str) -> str:
        # 实现翻译逻辑
        return f"Translated: {text}"

class SummaryPlugin(Plugin):
    """摘要插件"""
    
    def name(self) -> str:
        return "summarizer"
    
    def process(self, text: str) -> str:
        # 实现摘要逻辑
        return f"Summary: {text}"

class PluginManager:
    """插件管理器"""
    
    def __init__(self):
        self.plugins: Dict[str, Plugin] = {}
    
    def register(self, plugin: Plugin):
        """注册插件"""
        self.plugins[plugin.name()] = plugin
    
    def process(
        self,
        text: str,
        plugin_name: str
    ) -> str:
        """使用指定插件处理文本"""
        if plugin_name not in self.plugins:
            raise ValueError(f"Plugin {plugin_name} not found")
        
        return self.plugins[plugin_name].process(text)

# 使用插件系统
manager = PluginManager()
manager.register(TranslationPlugin())
manager.register(SummaryPlugin())

text = "Hello, world!"
translated = manager.process(text, "translator")
summarized = manager.process(text, "summarizer")
```

## 最佳实践

1. 继承使用：
   - 遵循"是一个"关系
   - 避免过深的继承层次
   - 优先使用组合而非继承

2. 抽象类设计：
   - 定义清晰的接口
   - 只抽象必要的方法
   - 提供适当的默认实现

3. 多态应用：
   - 使用依赖注入
   - 面向接口编程
   - 遵循里氏替换原则

4. 代码复用：
   - 使用混入类
   - 合理设计类层次
   - 避免代码重复

## 下一步

现在您已经掌握了继承和多态的概念，接下来我们将学习Python的特殊方法和魔术方法。
