---
title: "设计模式"
slug: "design-patterns"
sequence: 4
description: "学习Python中常用的设计模式，包括单例模式、工厂模式、观察者模式和策略模式"
status: "published"
---

# 设计模式

本节将介绍Python中常用的设计模式，这些模式能帮助我们更好地组织和管理代码。我们将重点关注在LLM应用开发中特别有用的设计模式。

## 单例模式

### 基本实现
```python
class Singleton:
    """基本单例模式"""
    
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# 使用单例
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # 输出: True
```

### 线程安全单例
```python
from threading import Lock

class ThreadSafeSingleton:
    """线程安全的单例模式"""
    
    _instance = None
    _lock = Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # 双重检查锁定
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        """
        确保即使多次调用__init__也是安全的
        """
        pass

# 配置管理器示例
class ConfigManager(ThreadSafeSingleton):
    """配置管理器"""
    
    def __init__(self):
        super().__init__()
        self._config = {}
    
    def set(self, key: str, value: str):
        """设置配置"""
        self._config[key] = value
    
    def get(self, key: str, default: str = None) -> str:
        """获取配置"""
        return self._config.get(key, default)

# 使用配置管理器
config = ConfigManager()
config.set("api_key", "your-api-key")

# 在其他地方获取同一个实例
same_config = ConfigManager()
print(same_config.get("api_key"))  # 输出: your-api-key
```

## 工厂模式

### 简单工厂
```python
from abc import ABC, abstractmethod
from typing import Dict, List

class LLMClient(ABC):
    """LLM客户端接口"""
    
    @abstractmethod
    def generate(self, prompt: str) -> str:
        """生成文本"""
        pass

class OpenAIClient(LLMClient):
    """OpenAI客户端"""
    
    def generate(self, prompt: str) -> str:
        return f"OpenAI: {prompt}"

class AnthropicClient(LLMClient):
    """Anthropic客户端"""
    
    def generate(self, prompt: str) -> str:
        return f"Anthropic: {prompt}"

class LLMFactory:
    """LLM客户端工厂"""
    
    @staticmethod
    def create(provider: str) -> LLMClient:
        """创建LLM客户端"""
        if provider == "openai":
            return OpenAIClient()
        elif provider == "anthropic":
            return AnthropicClient()
        else:
            raise ValueError(f"Unknown provider: {provider}")

# 使用工厂
client = LLMFactory.create("openai")
response = client.generate("Hello!")
```

### 工厂方法
```python
class LLMClientFactory(ABC):
    """LLM客户端工厂接口"""
    
    @abstractmethod
    def create_client(self) -> LLMClient:
        """创建客户端"""
        pass
    
    def generate_text(self, prompt: str) -> str:
        """生成文本的模板方法"""
        client = self.create_client()
        return client.generate(prompt)

class OpenAIFactory(LLMClientFactory):
    """OpenAI客户端工厂"""
    
    def create_client(self) -> LLMClient:
        return OpenAIClient()

class AnthropicFactory(LLMClientFactory):
    """Anthropic客户端工厂"""
    
    def create_client(self) -> LLMClient:
        return AnthropicClient()

# 使用工厂方法
factory = OpenAIFactory()
response = factory.generate_text("Hello!")
```

## 观察者模式

### 基本实现
```python
from abc import ABC, abstractmethod
from typing import List, Any

class Observer(ABC):
    """观察者接口"""
    
    @abstractmethod
    def update(self, subject: 'Subject'):
        """更新方法"""
        pass

class Subject:
    """被观察者"""
    
    def __init__(self):
        self._observers: List[Observer] = []
        self._state = None
    
    def attach(self, observer: Observer):
        """添加观察者"""
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        """移除观察者"""
        self._observers.remove(observer)
    
    def notify(self):
        """通知所有观察者"""
        for observer in self._observers:
            observer.update(self)
    
    @property
    def state(self) -> Any:
        """获取状态"""
        return self._state
    
    @state.setter
    def state(self, value: Any):
        """设置状态"""
        self._state = value
        self.notify()

# API调用监控示例
class APIMonitor(Subject):
    """API监控器"""
    
    def __init__(self):
        super().__init__()
        self._calls = 0
        self._errors = 0
    
    def record_call(self):
        """记录API调用"""
        self._calls += 1
        self.state = {"calls": self._calls, "errors": self._errors}
    
    def record_error(self):
        """记录错误"""
        self._errors += 1
        self.state = {"calls": self._calls, "errors": self._errors}

class AlertObserver(Observer):
    """告警观察者"""
    
    def __init__(self, error_threshold: int):
        self.error_threshold = error_threshold
    
    def update(self, subject: Subject):
        state = subject.state
        if state["errors"] >= self.error_threshold:
            print(f"Alert: Error count ({state['errors']}) "
                  f"exceeded threshold!")

class LogObserver(Observer):
    """日志观察者"""
    
    def update(self, subject: Subject):
        state = subject.state
        print(f"Log: API calls: {state['calls']}, "
              f"Errors: {state['errors']}")

# 使用观察者模式
monitor = APIMonitor()
monitor.attach(AlertObserver(error_threshold=3))
monitor.attach(LogObserver())

# 模拟API调用
monitor.record_call()
monitor.record_call()
monitor.record_error()
monitor.record_error()
monitor.record_error()  # 触发告警
```

## 策略模式

### 基本实现
```python
from abc import ABC, abstractmethod
from typing import List, Dict

class RetryStrategy(ABC):
    """重试策略接口"""
    
    @abstractmethod
    def should_retry(
        self,
        attempt: int,
        error: Exception
    ) -> bool:
        """判断是否应该重试"""
        pass
    
    @abstractmethod
    def get_delay(self, attempt: int) -> float:
        """获取重试延迟时间"""
        pass

class ExponentialBackoff(RetryStrategy):
    """指数退避策略"""
    
    def __init__(
        self,
        max_attempts: int,
        base_delay: float
    ):
        self.max_attempts = max_attempts
        self.base_delay = base_delay
    
    def should_retry(
        self,
        attempt: int,
        error: Exception
    ) -> bool:
        return (attempt < self.max_attempts and
                isinstance(error, (TimeoutError, ConnectionError)))
    
    def get_delay(self, attempt: int) -> float:
        return self.base_delay * (2 ** attempt)

class FixedRetry(RetryStrategy):
    """固定间隔重试策略"""
    
    def __init__(
        self,
        max_attempts: int,
        delay: float
    ):
        self.max_attempts = max_attempts
        self.delay = delay
    
    def should_retry(
        self,
        attempt: int,
        error: Exception
    ) -> bool:
        return attempt < self.max_attempts
    
    def get_delay(self, attempt: int) -> float:
        return self.delay

# API客户端示例
class APIClient:
    """API客户端"""
    
    def __init__(
        self,
        retry_strategy: RetryStrategy
    ):
        self.retry_strategy = retry_strategy
    
    def call_api(self) -> str:
        """调用API"""
        attempt = 0
        while True:
            try:
                # 模拟API调用
                if attempt < 2:
                    raise TimeoutError("API timeout")
                return "Success"
            except Exception as e:
                if not self.retry_strategy.should_retry(
                    attempt, e
                ):
                    raise
                
                import time
                delay = self.retry_strategy.get_delay(attempt)
                time.sleep(delay)
                attempt += 1

# 使用策略模式
client = APIClient(
    ExponentialBackoff(
        max_attempts=3,
        base_delay=1
    )
)

try:
    result = client.call_api()
    print(result)
except Exception as e:
    print(f"Failed after retries: {e}")
```

## 实际应用示例

### 模型选择器
```python
from abc import ABC, abstractmethod
from typing import Dict, List, Optional
import random

class ModelStrategy(ABC):
    """模型选择策略"""
    
    @abstractmethod
    def select_model(
        self,
        prompt: str,
        context: Dict
    ) -> str:
        """选择模型"""
        pass

class CostBasedStrategy(ModelStrategy):
    """基于成本的选择策略"""
    
    def __init__(self, budget: float):
        self.budget = budget
        self.costs = {
            "gpt-4": 0.03,
            "gpt-3.5-turbo": 0.002,
            "claude-2": 0.01
        }
    
    def select_model(
        self,
        prompt: str,
        context: Dict
    ) -> str:
        # 根据预算选择模型
        if context.get("importance") == "high":
            if self.budget >= self.costs["gpt-4"]:
                return "gpt-4"
        return "gpt-3.5-turbo"

class PerformanceBasedStrategy(ModelStrategy):
    """基于性能的选择策略"""
    
    def __init__(self):
        self.performance_scores = {
            "gpt-4": 0.95,
            "gpt-3.5-turbo": 0.85,
            "claude-2": 0.90
        }
    
    def select_model(
        self,
        prompt: str,
        context: Dict
    ) -> str:
        # 根据任务类型选择模型
        task_type = context.get("task_type", "general")
        if task_type == "coding":
            return "gpt-4"
        elif task_type == "writing":
            return "claude-2"
        return "gpt-3.5-turbo"

class LoadBalancingStrategy(ModelStrategy):
    """负载均衡策略"""
    
    def __init__(self):
        self.models = ["gpt-4", "gpt-3.5-turbo", "claude-2"]
        self.current_index = 0
    
    def select_model(
        self,
        prompt: str,
        context: Dict
    ) -> str:
        # 轮询选择模型
        model = self.models[self.current_index]
        self.current_index = (
            self.current_index + 1
        ) % len(self.models)
        return model

class ModelSelector:
    """模型选择器"""
    
    def __init__(self, strategy: ModelStrategy):
        self.strategy = strategy
    
    def set_strategy(self, strategy: ModelStrategy):
        """设置策略"""
        self.strategy = strategy
    
    def select_model(
        self,
        prompt: str,
        context: Dict = None
    ) -> str:
        """选择模型"""
        if context is None:
            context = {}
        return self.strategy.select_model(prompt, context)

# 使用模型选择器
selector = ModelSelector(
    CostBasedStrategy(budget=0.1)
)

# 高重要性任务
model = selector.select_model(
    "Complex analysis",
    {"importance": "high"}
)
print(f"Selected model: {model}")

# 切换到性能策略
selector.set_strategy(PerformanceBasedStrategy())

# 编码任务
model = selector.select_model(
    "Write a function",
    {"task_type": "coding"}
)
print(f"Selected model: {model}")
```

### 插件系统
```python
from abc import ABC, abstractmethod
from typing import Dict, List, Any
import json

class Plugin(ABC):
    """插件接口"""
    
    @abstractmethod
    def process(
        self,
        input_data: Any
    ) -> Any:
        """处理数据"""
        pass

class PluginManager:
    """插件管理器"""
    
    def __init__(self):
        self._plugins: Dict[str, Plugin] = {}
    
    def register(
        self,
        name: str,
        plugin: Plugin
    ):
        """注册插件"""
        self._plugins[name] = plugin
    
    def unregister(self, name: str):
        """注销插件"""
        self._plugins.pop(name, None)
    
    def get_plugin(
        self,
        name: str
    ) -> Optional[Plugin]:
        """获取插件"""
        return self._plugins.get(name)
    
    def process(
        self,
        name: str,
        input_data: Any
    ) -> Any:
        """使用插件处理数据"""
        plugin = self.get_plugin(name)
        if plugin is None:
            raise ValueError(f"Plugin {name} not found")
        return plugin.process(input_data)

# 实现具体插件
class TranslationPlugin(Plugin):
    """翻译插件"""
    
    def process(self, input_data: str) -> str:
        # 实现翻译逻辑
        return f"Translated: {input_data}"

class SummaryPlugin(Plugin):
    """摘要插件"""
    
    def process(self, input_data: str) -> str:
        # 实现摘要逻辑
        return f"Summary: {input_data}"

class FormatPlugin(Plugin):
    """格式化插件"""
    
    def process(self, input_data: Dict) -> str:
        # 实现格式化逻辑
        return json.dumps(input_data, indent=2)

# 使用插件系统
manager = PluginManager()

# 注册插件
manager.register("translate", TranslationPlugin())
manager.register("summarize", SummaryPlugin())
manager.register("format", FormatPlugin())

# 使用插件
text = "Hello, World!"
translated = manager.process("translate", text)
summarized = manager.process("summarize", text)

data = {"name": "John", "age": 30}
formatted = manager.process("format", data)

print(translated)
print(summarized)
print(formatted)
```

## 最佳实践

1. 设计模式选择：
   - 根据实际需求选择
   - 避免过度设计
   - 保持代码简单

2. 模式实现：
   - 遵循SOLID原则
   - 保持接口清晰
   - 考虑扩展性

3. 性能考虑：
   - 权衡抽象和性能
   - 避免不必要的复杂性
   - 适当使用缓存

4. 代码维护：
   - 添加充分的文档
   - 编写单元测试
   - 定期重构代码

## 下一步

现在您已经掌握了常用的设计模式，接下来我们将通过实战练习来应用这些知识。
