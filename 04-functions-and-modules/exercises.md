---
title: "实战练习"
slug: "exercises"
sequence: 5
description: "通过实践项目巩固函数和模块的知识，包括多模型API封装和聊天机器人的实现"
status: "published"
---

# 实战练习

本节将通过两个实践项目来巩固前面学习的函数和模块知识。这些项目将综合运用装饰器、错误处理、API封装等技术。

## 项目一：多模型API封装

创建一个支持多个LLM服务商API的统一接口库。

### 项目结构
```
llm_toolkit/
│
├── __init__.py
├── base.py
├── clients/
│   ├── __init__.py
│   ├── openai.py
│   ├── anthropic.py
│   └── deepseek.py
│
├── utils/
│   ├── __init__.py
│   ├── rate_limit.py
│   └── retry.py
│
└── exceptions.py
```

### 实现代码

```python
# base.py
from abc import ABC, abstractmethod
from typing import Dict, List, Optional

class BaseLLMClient(ABC):
    """LLM客户端基类"""
    
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

# clients/openai.py
from ..base import BaseLLMClient
from ..utils.rate_limit import RateLimiter
from ..utils.retry import retry_with_backoff

class OpenAIClient(BaseLLMClient):
    """OpenAI API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.rate_limiter = RateLimiter(60, 60)  # 60次/分钟
    
    @retry_with_backoff()
    def chat_completion(self, messages: List[Dict[str, str]], **kwargs):
        with self.rate_limiter:
            # 实现OpenAI chat completion调用
            pass
    
    @retry_with_backoff()
    def embeddings(self, text: str, **kwargs):
        with self.rate_limiter:
            # 实现OpenAI embeddings调用
            pass

# clients/anthropic.py
class AnthropicClient(BaseLLMClient):
    """Anthropic API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.rate_limiter = RateLimiter(50, 60)  # 50次/分钟
    
    @retry_with_backoff()
    def chat_completion(self, messages: List[Dict[str, str]], **kwargs):
        with self.rate_limiter:
            # 实现Anthropic chat completion调用
            pass
    
    @retry_with_backoff()
    def embeddings(self, text: str, **kwargs):
        with self.rate_limiter:
            # 实现Anthropic embeddings调用
            pass

# utils/rate_limit.py
import time
from threading import Lock
from contextlib import contextmanager

class RateLimiter:
    """速率限制器"""
    
    def __init__(self, max_requests: int, time_window: int):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []
        self.lock = Lock()
    
    def _cleanup(self):
        """清理过期的请求记录"""
        now = time.time()
        self.requests = [
            req_time for req_time in self.requests
            if now - req_time <= self.time_window
        ]
    
    @contextmanager
    def __enter__(self):
        with self.lock:
            self._cleanup()
            
            if len(self.requests) >= self.max_requests:
                sleep_time = self.requests[0] + self.time_window - time.time()
                if sleep_time > 0:
                    time.sleep(sleep_time)
            
            self.requests.append(time.time())
            yield
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        pass

# utils/retry.py
import time
from functools import wraps

def retry_with_backoff(
    max_retries: int = 3,
    base_delay: float = 1,
    max_delay: float = 60
):
    """重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries:
                        raise
                    
                    delay = min(
                        base_delay * (2 ** attempt),
                        max_delay
                    )
                    time.sleep(delay)
            return None
        return wrapper
    return decorator
```

### 使用示例
```python
from llm_toolkit.clients import OpenAIClient, AnthropicClient

# 创建客户端
openai_client = OpenAIClient("openai-api-key")
anthropic_client = AnthropicClient("anthropic-api-key")

# 使用OpenAI
response = openai_client.chat_completion([
    {"role": "user", "content": "Hello!"}
])

# 使用Anthropic
response = anthropic_client.chat_completion([
    {"role": "user", "content": "Hello!"}
])
```

## 项目二：聊天机器人

创建一个支持多轮对话的聊天机器人，包括对话历史管理和多模型支持。

### 项目结构
```
chatbot/
│
├── __init__.py
├── bot.py
├── history.py
├── models/
│   ├── __init__.py
│   └── llm.py
│
└── utils/
    ├── __init__.py
    └── prompt.py
```

### 实现代码

```python
# history.py
from typing import List, Dict
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    """对话消息"""
    role: str
    content: str
    timestamp: datetime = datetime.now()

class ChatHistory:
    """对话历史管理器"""
    
    def __init__(self, max_messages: int = 100):
        self.messages: List[Message] = []
        self.max_messages = max_messages
    
    def add_message(self, role: str, content: str):
        """添加新消息"""
        if len(self.messages) >= self.max_messages:
            self.messages.pop(0)
        
        self.messages.append(Message(role, content))
    
    def get_messages(self, limit: Optional[int] = None) -> List[Dict]:
        """获取最近的消息"""
        messages = self.messages[-limit:] if limit else self.messages
        return [
            {"role": msg.role, "content": msg.content}
            for msg in messages
        ]
    
    def clear(self):
        """清空历史"""
        self.messages.clear()

# models/llm.py
from abc import ABC, abstractmethod
from typing import List, Dict

class LLMModel(ABC):
    """LLM模型接口"""
    
    @abstractmethod
    def generate_response(
        self,
        messages: List[Dict[str, str]]
    ) -> str:
        """生成响应"""
        pass

class OpenAIModel(LLMModel):
    """OpenAI模型"""
    
    def __init__(self, client, model="gpt-3.5-turbo"):
        self.client = client
        self.model = model
    
    def generate_response(self, messages):
        response = self.client.chat_completion(
            messages,
            model=self.model
        )
        return response["choices"][0]["message"]["content"]

class AnthropicModel(LLMModel):
    """Anthropic模型"""
    
    def __init__(self, client, model="claude-2"):
        self.client = client
        self.model = model
    
    def generate_response(self, messages):
        response = self.client.chat_completion(
            messages,
            model=self.model
        )
        return response["completion"]

# utils/prompt.py
def create_system_prompt(persona: str) -> str:
    """创建系统提示"""
    return f"""You are {persona}. 
    Please provide helpful and informative responses.
    Keep your answers concise and relevant."""

# bot.py
class Chatbot:
    """聊天机器人"""
    
    def __init__(
        self,
        model: LLMModel,
        persona: str = "a helpful assistant",
        max_history: int = 10
    ):
        self.model = model
        self.history = ChatHistory(max_history)
        self.system_prompt = create_system_prompt(persona)
    
    def chat(self, message: str) -> str:
        """进行对话"""
        # 添加用户消息
        self.history.add_message("user", message)
        
        # 准备完整的对话历史
        messages = [
            {"role": "system", "content": self.system_prompt}
        ] + self.history.get_messages()
        
        try:
            # 生成响应
            response = self.model.generate_response(messages)
            
            # 添加助手消息
            self.history.add_message("assistant", response)
            
            return response
        
        except Exception as e:
            error_message = f"Error generating response: {str(e)}"
            self.history.add_message("system", error_message)
            return error_message
    
    def clear_history(self):
        """清空对话历史"""
        self.history.clear()
```

### 使用示例
```python
from chatbot import Chatbot
from chatbot.models.llm import OpenAIModel
from llm_toolkit.clients import OpenAIClient

# 创建OpenAI客户端和模型
client = OpenAIClient("your-api-key")
model = OpenAIModel(client)

# 创建聊天机器人
bot = Chatbot(
    model=model,
    persona="a Python programming expert",
    max_history=10
)

# 进行对话
while True:
    user_input = input("You: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    
    response = bot.chat(user_input)
    print(f"Bot: {response}")
```

## 扩展练习

1. 多模型API封装：
   - 添加更多API提供商的支持
   - 实现模型自动切换
   - 添加流式响应支持
   - 实现响应缓存

2. 聊天机器人：
   - 添加对话保存和加载功能
   - 实现多用户支持
   - 添加中间件系统
   - 实现会话管理

## 最佳实践

1. 代码组织：
   - 使用清晰的项目结构
   - 分离接口和实现
   - 使用适当的设计模式

2. 错误处理：
   - 实现完整的错误处理
   - 提供有意义的错误消息
   - 记录错误日志

3. 性能优化：
   - 实现响应缓存
   - 优化资源使用
   - 处理并发请求

4. 可维护性：
   - 编写单元测试
   - 添加类型注解
   - 保持代码文档更新

## 下一步

完成这些练习后，您应该已经掌握了如何使用Python的函数和模块来构建实际的应用程序。接下来，我们将学习面向对象编程的概念和应用。
