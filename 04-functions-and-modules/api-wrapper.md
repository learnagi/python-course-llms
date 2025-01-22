---
title: "API封装"
slug: "api-wrapper"
sequence: 4
description: "学习如何封装LLM API，创建易用的客户端库，包括错误处理、重试机制和速率限制"
status: "published"
---

# API封装

本节将介绍如何封装LLM API，创建一个健壮、易用的客户端库。我们将学习如何处理错误、实现重试机制和速率限制。

## 基础API客户端

### 基类设计
```python
from typing import Dict, List, Optional
import requests
from abc import ABC, abstractmethod

class BaseLLMClient(ABC):
    """LLM API客户端基类"""
    
    def __init__(self, api_key: str, base_url: str):
        """
        初始化客户端
        
        Args:
            api_key: API密钥
            base_url: API基础URL
        """
        self.api_key = api_key
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        })
    
    @abstractmethod
    def chat_completion(self, messages: List[Dict[str, str]], **kwargs) -> Dict:
        """
        聊天补全API
        
        Args:
            messages: 消息列表
            **kwargs: 其他参数
        
        Returns:
            API响应
        """
        pass
    
    @abstractmethod
    def embeddings(self, text: str) -> List[float]:
        """
        文本嵌入API
        
        Args:
            text: 输入文本
        
        Returns:
            嵌入向量
        """
        pass
```

### OpenAI客户端实现
```python
import json
from typing import Dict, List, Optional

class OpenAIClient(BaseLLMClient):
    """OpenAI API客户端"""
    
    def __init__(self, api_key: str):
        super().__init__(
            api_key=api_key,
            base_url="https://api.openai.com/v1"
        )
    
    def chat_completion(
        self,
        messages: List[Dict[str, str]],
        model: str = "gpt-3.5-turbo",
        temperature: float = 0.7,
        max_tokens: Optional[int] = None,
        **kwargs
    ) -> Dict:
        """
        调用chat completion API
        
        Args:
            messages: 消息列表
            model: 模型名称
            temperature: 温度参数
            max_tokens: 最大生成token数
            **kwargs: 其他参数
        
        Returns:
            API响应
        """
        endpoint = f"{self.base_url}/chat/completions"
        
        data = {
            "model": model,
            "messages": messages,
            "temperature": temperature,
            **kwargs
        }
        
        if max_tokens is not None:
            data["max_tokens"] = max_tokens
        
        response = self.session.post(endpoint, json=data)
        response.raise_for_status()
        
        return response.json()
    
    def embeddings(
        self,
        text: str,
        model: str = "text-embedding-ada-002"
    ) -> List[float]:
        """
        获取文本嵌入向量
        
        Args:
            text: 输入文本
            model: 模型名称
        
        Returns:
            嵌入向量
        """
        endpoint = f"{self.base_url}/embeddings"
        
        response = self.session.post(endpoint, json={
            "model": model,
            "input": text
        })
        response.raise_for_status()
        
        return response.json()["data"][0]["embedding"]
```

## 错误处理

### 自定义异常
```python
class LLMError(Exception):
    """LLM API错误基类"""
    pass

class AuthenticationError(LLMError):
    """认证错误"""
    pass

class RateLimitError(LLMError):
    """速率限制错误"""
    pass

class APIError(LLMError):
    """API调用错误"""
    def __init__(self, message: str, status_code: int, response: Dict):
        super().__init__(message)
        self.status_code = status_code
        self.response = response
```

### 错误处理装饰器
```python
from functools import wraps
import requests

def handle_api_errors(func):
    """处理API错误的装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except requests.exceptions.HTTPError as e:
            response = e.response
            status_code = response.status_code
            
            try:
                error_data = response.json()
            except json.JSONDecodeError:
                error_data = {"error": response.text}
            
            if status_code == 401:
                raise AuthenticationError("Invalid API key")
            elif status_code == 429:
                raise RateLimitError("Rate limit exceeded")
            else:
                raise APIError(
                    f"API request failed: {error_data.get('error', 'Unknown error')}",
                    status_code,
                    error_data
                )
        except requests.exceptions.ConnectionError:
            raise LLMError("Failed to connect to API server")
        except requests.exceptions.Timeout:
            raise LLMError("API request timed out")
        except Exception as e:
            raise LLMError(f"Unexpected error: {str(e)}")
    
    return wrapper
```

## 重试机制

### 重试装饰器
```python
import time
from typing import Type, Tuple

def retry_with_exponential_backoff(
    max_retries: int = 3,
    base_delay: float = 1,
    max_delay: float = 60,
    retryable_exceptions: Tuple[Type[Exception], ...] = (
        RateLimitError,
        requests.exceptions.ConnectionError,
        requests.exceptions.Timeout
    )
):
    """
    实现指数退避重试的装饰器
    
    Args:
        max_retries: 最大重试次数
        base_delay: 基础延迟时间（秒）
        max_delay: 最大延迟时间（秒）
        retryable_exceptions: 可重试的异常类型
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except retryable_exceptions as e:
                    if attempt == max_retries:
                        raise
                    
                    delay = min(
                        base_delay * (2 ** attempt),
                        max_delay
                    )
                    
                    print(f"Attempt {attempt + 1} failed: {str(e)}")
                    print(f"Retrying in {delay} seconds...")
                    
                    time.sleep(delay)
            
            return None  # 不应该到达这里
        return wrapper
    return decorator
```

## 速率限制

### 令牌桶算法
```python
import time
from threading import Lock

class TokenBucket:
    """令牌桶速率限制器"""
    
    def __init__(
        self,
        tokens_per_second: float,
        max_tokens: int
    ):
        """
        初始化令牌桶
        
        Args:
            tokens_per_second: 每秒补充的令牌数
            max_tokens: 桶的最大容量
        """
        self.tokens_per_second = tokens_per_second
        self.max_tokens = max_tokens
        self.tokens = max_tokens
        self.last_update = time.time()
        self.lock = Lock()
    
    def _add_tokens(self):
        """补充令牌"""
        now = time.time()
        time_passed = now - self.last_update
        new_tokens = time_passed * self.tokens_per_second
        
        self.tokens = min(
            self.tokens + new_tokens,
            self.max_tokens
        )
        self.last_update = now
    
    def acquire(self, tokens: int = 1, timeout: Optional[float] = None) -> bool:
        """
        获取令牌
        
        Args:
            tokens: 需要的令牌数
            timeout: 超时时间（秒）
        
        Returns:
            是否获取成功
        """
        start_time = time.time()
        
        while True:
            with self.lock:
                self._add_tokens()
                
                if self.tokens >= tokens:
                    self.tokens -= tokens
                    return True
            
            if timeout is not None:
                if time.time() - start_time >= timeout:
                    return False
            
            time.sleep(0.1)
```

### 使用速率限制
```python
class RateLimitedClient(OpenAIClient):
    """带速率限制的API客户端"""
    
    def __init__(
        self,
        api_key: str,
        tokens_per_minute: int = 60
    ):
        super().__init__(api_key)
        self.rate_limiter = TokenBucket(
            tokens_per_second=tokens_per_minute / 60,
            max_tokens=tokens_per_minute
        )
    
    @retry_with_exponential_backoff()
    @handle_api_errors
    def chat_completion(self, messages: List[Dict[str, str]], **kwargs) -> Dict:
        """带速率限制的chat completion调用"""
        if not self.rate_limiter.acquire(timeout=60):
            raise RateLimitError("Failed to acquire rate limit token")
        
        return super().chat_completion(messages, **kwargs)
```

## 完整示例

### 客户端使用
```python
# 创建客户端
client = RateLimitedClient(
    api_key="your-api-key",
    tokens_per_minute=60
)

try:
    # 调用chat completion API
    response = client.chat_completion(
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Tell me about Python."}
        ],
        temperature=0.7,
        max_tokens=100
    )
    
    # 处理响应
    message = response["choices"][0]["message"]["content"]
    print(f"Assistant: {message}")

except AuthenticationError:
    print("Invalid API key")
except RateLimitError:
    print("Rate limit exceeded")
except APIError as e:
    print(f"API error: {e}")
except LLMError as e:
    print(f"General error: {e}")
```

### 异步客户端
```python
import asyncio
import aiohttp
from typing import Dict, List, Optional

class AsyncOpenAIClient(BaseLLMClient):
    """异步OpenAI API客户端"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.openai.com/v1"
        self.session = None
    
    async def __aenter__(self):
        """创建异步会话"""
        self.session = aiohttp.ClientSession(headers={
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        })
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """关闭异步会话"""
        if self.session:
            await self.session.close()
    
    @retry_with_exponential_backoff()
    @handle_api_errors
    async def chat_completion(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ) -> Dict:
        """异步chat completion调用"""
        endpoint = f"{self.base_url}/chat/completions"
        
        async with self.session.post(endpoint, json={
            "messages": messages,
            **kwargs
        }) as response:
            response.raise_for_status()
            return await response.json()

# 使用异步客户端
async def main():
    async with AsyncOpenAIClient("your-api-key") as client:
        response = await client.chat_completion([
            {"role": "user", "content": "Hello!"}
        ])
        print(response)

asyncio.run(main())
```

## 最佳实践

1. 错误处理：
   - 定义清晰的异常层次
   - 提供详细的错误信息
   - 适当的日志记录

2. 重试策略：
   - 使用指数退避
   - 只重试可恢复的错误
   - 设置最大重试次数

3. 速率限制：
   - 实现平滑的限制
   - 考虑并发情况
   - 提供超时机制

4. 代码组织：
   - 使用装饰器分离关注点
   - 提供同步和异步接口
   - 保持代码可测试性

## 下一步

现在您已经学会了如何封装LLM API，接下来我们将通过实战练习来应用这些知识。
