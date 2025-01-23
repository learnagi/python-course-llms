---
title: "OpenAI API使用基础"
slug: "openai-api-basics"
sequence: 1
description: "学习OpenAI API的基本使用方法，包括配置、文本生成、对话和图像生成等功能"
is_published: true
estimated_minutes: 20
---

# OpenAI API使用基础

本节将介绍OpenAI API的基本使用方法，包括环境配置、文本生成、对话功能和图像生成等内容。

## 1. 环境配置

### API密钥管理

在使用OpenAI API之前，需要进行适当的环境配置和API密钥管理。以下是推荐的最佳实践：

```python
import os
import openai
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 设置API密钥
openai.api_key = os.getenv("OPENAI_API_KEY")

# 可选：设置API基础URL（如果使用代理或自定义端点）
openai.api_base = os.getenv("OPENAI_API_BASE", "https://api.openai.com/v1")

# 可选：设置组织ID
if os.getenv("OPENAI_ORG_ID"):
    openai.organization = os.getenv("OPENAI_ORG_ID")
```

### 费用和限制管理

```python
class APIRateManager:
    def __init__(self):
        self.last_request_time = 0
        self.min_request_interval = 1  # 秒
        self.total_tokens = 0
        self.cost_per_1k_tokens = 0.002  # GPT-3.5-turbo的价格

    def wait_if_needed(self):
        current_time = time.time()
        time_since_last_request = current_time - self.last_request_time
        if time_since_last_request < self.min_request_interval:
            time.sleep(self.min_request_interval - time_since_last_request)
        self.last_request_time = time.time()

    def update_usage(self, response):
        if hasattr(response, 'usage'):
            self.total_tokens += response.usage.total_tokens

    def get_estimated_cost(self):
        return (self.total_tokens / 1000) * self.cost_per_1k_tokens
```

## 2. 文本生成

### 使用GPT模型

以下是一个完整的文本生成工具类，包含了错误处理、重试机制和参数优化：

```python
from typing import Optional, Dict, Any
import time
import logging

class TextGenerator:
    def __init__(self, model: str = "gpt-3.5-turbo"):
        self.model = model
        self.retry_count = 3
        self.retry_delay = 1
        self.logger = logging.getLogger(__name__)

    def generate(self, 
                 prompt: str, 
                 max_tokens: int = 100,
                 temperature: float = 0.7,
                 top_p: float = 1.0,
                 frequency_penalty: float = 0.0,
                 presence_penalty: float = 0.0,
                 stream: bool = False) -> Optional[str]:
        """使用GPT模型生成文本，支持高级参数配置和流式输出"""
        for attempt in range(self.retry_count):
            try:
                if stream:
                    return self._stream_generate(
                        prompt, max_tokens, temperature,
                        top_p, frequency_penalty, presence_penalty
                    )
                
                response = openai.ChatCompletion.create(
                    model=self.model,
                    messages=[{"role": "user", "content": prompt}],
                    max_tokens=max_tokens,
                    temperature=temperature,
                    top_p=top_p,
                    frequency_penalty=frequency_penalty,
                    presence_penalty=presence_penalty
                )
                return response.choices[0].message.content
            except Exception as e:
                self.logger.error(f"Attempt {attempt + 1} failed: {str(e)}")
                if attempt < self.retry_count - 1:
                    time.sleep(self.retry_delay * (attempt + 1))
                else:
                    raise

    def _stream_generate(self, prompt: str, **kwargs) -> Generator[str, None, None]:
        """流式生成文本"""
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            stream=True,
            **kwargs
        )
        for chunk in response:
            if chunk and chunk.choices and chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content

    def generate_with_functions(self, 
                              prompt: str,
                              functions: List[Dict[str, Any]],
                              **kwargs) -> Dict[str, Any]:
        """支持函数调用的生成"""
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            functions=functions,
            **kwargs
        )
        return response.choices[0].message
```

### 使用示例

```python
# 初始化生成器
generator = TextGenerator()

# 基础文本生成
prompt = "写一个简短的Python函数来计算斐波那契数列"
result = generator.generate(prompt)
print("基础生成结果：", result)

# 流式生成
for chunk in generator.generate(prompt, stream=True):
    print(chunk, end="", flush=True)

# 函数调用示例
functions = [
    {
        "name": "get_weather",
        "description": "获取指定城市的天气信息",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["city"]
        }
    }
]

result = generator.generate_with_functions(
    "查询北京的天气",
    functions=functions
)
print("函数调用结果：", result)
```

## 3. 聊天对话

### Chat Completion API

以下是一个专业的聊天系统实现，包含了上下文管理、流式响应和错误处理：

```python
from typing import List, Dict, Generator, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    role: str
    content: str
    timestamp: datetime = datetime.now()

class ChatSystem:
    def __init__(self, 
                 model: str = "gpt-3.5-turbo",
                 max_context_messages: int = 10,
                 system_prompt: str = None):
        self.model = model
        self.max_context_messages = max_context_messages
        self.messages: List[Message] = []
        if system_prompt:
            self.add_message("system", system_prompt)

    def add_message(self, role: str, content: str) -> None:
        """添加消息到对话历史"""
        self.messages.append(Message(role=role, content=content))
        # 保持上下文长度在限制范围内
        if len(self.messages) > self.max_context_messages:
            # 保留system消息
            system_messages = [m for m in self.messages if m.role == "system"]
            other_messages = [m for m in self.messages if m.role != "system"]
            other_messages = other_messages[-(self.max_context_messages - len(system_messages)):]
            self.messages = system_messages + other_messages

    def get_chat_response(self, 
                         user_message: str,
                         stream: bool = False,
                         **kwargs) -> Union[str, Generator[str, None, None]]:
        """获取聊天响应，支持流式输出"""
        self.add_message("user", user_message)
        messages = self.get_formatted_messages()

        try:
            if stream:
                return self._stream_response(messages, **kwargs)

            response = openai.ChatCompletion.create(
                model=self.model,
                messages=messages,
                **kwargs
            )
            assistant_message = response.choices[0].message.content
            self.add_message("assistant", assistant_message)
            return assistant_message

        except Exception as e:
            self.messages.pop()  # 移除失败的消息
            raise

    def _stream_response(self, messages: List[Dict[str, str]], **kwargs) -> Generator[str, None, None]:
        """流式获取响应"""
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=messages,
            stream=True,
            **kwargs
        )
        
        full_response = ""
        for chunk in response:
            if chunk and chunk.choices and chunk.choices[0].delta.content:
                content = chunk.choices[0].delta.content
                full_response += content
                yield content

        self.add_message("assistant", full_response)
```

### 使用示例

```python
# 初始化聊天系统
chat = ChatSystem(
    system_prompt="你是一个专业的Python开发专家，擅长解决性能优化问题。"
)

# 基础对话
response = chat.get_chat_response("如何实现一个装饰器来测量函数执行时间？")
print("助手回答：", response)

# 流式对话
print("助手正在回答：", end="")
for chunk in chat.get_chat_response(
    "请解释Python的上下文管理器原理",
    stream=True
):
    print(chunk, end="", flush=True)
print()
```

## 4. 图像生成

### DALL-E API使用

```python
from typing import Optional, List
from PIL import Image
import requests
from io import BytesIO

class ImageGenerator:
    def __init__(self):
        self.default_size = "1024x1024"
        self.supported_sizes = ["256x256", "512x512", "1024x1024"]

    def generate_image(
        self,
        prompt: str,
        size: str = None,
        n: int = 1,
        response_format: str = "url"
    ) -> List[str]:
        """生成图像"""
        try:
            response = openai.Image.create(
                prompt=prompt,
                n=n,
                size=size or self.default_size,
                response_format=response_format
            )
            return [item.url for item in response.data]
        except Exception as e:
            logging.error(f"图像生成失败: {str(e)}")
            raise

    def edit_image(
        self,
        image_path: str,
        mask_path: str,
        prompt: str,
        size: str = None,
        n: int = 1
    ) -> List[str]:
        """编辑图像"""
        try:
            with open(image_path, "rb") as image_file, \
                 open(mask_path, "rb") as mask_file:
                response = openai.Image.create_edit(
                    image=image_file,
                    mask=mask_file,
                    prompt=prompt,
                    n=n,
                    size=size or self.default_size
                )
            return [item.url for item in response.data]
        except Exception as e:
            logging.error(f"图像编辑失败: {str(e)}")
            raise

    def create_variation(
        self,
        image_path: str,
        n: int = 1,
        size: str = None
    ) -> List[str]:
        """创建图像变体"""
        try:
            with open(image_path, "rb") as image_file:
                response = openai.Image.create_variation(
                    image=image_file,
                    n=n,
                    size=size or self.default_size
                )
            return [item.url for item in response.data]
        except Exception as e:
            logging.error(f"创建变体失败: {str(e)}")
            raise

    @staticmethod
    def save_image(url: str, output_path: str) -> None:
        """保存图像到本地"""
        try:
            response = requests.get(url)
            response.raise_for_status()
            image = Image.open(BytesIO(response.content))
            image.save(output_path)
        except Exception as e:
            logging.error(f"图像保存失败: {str(e)}")
            raise
```

### 使用示例

```python
# 初始化图像生成器
image_generator = ImageGenerator()

# 生成图像
urls = image_generator.generate_image(
    prompt="一只可爱的卡通猫咪，坐在电脑前编程",
    n=1,
    size="512x512"
)

# 保存图像
for i, url in enumerate(urls):
    image_generator.save_image(url, f"cat_programming_{i}.png")

# 编
```

## 5. 错误处理和最佳实践

### API调用封装

以下是一个通用的API调用封装类，实现了完整的错误处理、重试机制和速率限制：

```python
from typing import Optional, Dict, Any, Callable
import time
import logging
from functools import wraps

class APIRateLimiter:
    def __init__(self, calls: int, period: float):
        self.calls = calls  # 允许的调用次数
        self.period = period  # 时间周期（秒）
        self.timestamps = []

    def __call__(self, func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            # 清理过期的时间戳
            self.timestamps = [ts for ts in self.timestamps if ts > now - self.period]
            
            if len(self.timestamps) >= self.calls:
                sleep_time = self.timestamps[0] - (now - self.period)
                if sleep_time > 0:
                    time.sleep(sleep_time)
            
            self.timestamps.append(now)
            return func(*args, **kwargs)
        return wrapper

class OpenAIClient:
    def __init__(self, 
                 api_key: str,
                 org_id: Optional[str] = None,
                 max_retries: int = 3,
                 timeout: float = 30.0):
        self.api_key = api_key
        self.org_id = org_id
        self.max_retries = max_retries
        self.timeout = timeout
        self.logger = logging.getLogger(__name__)
        self._setup()

    def _setup(self) -> None:
        """初始化OpenAI客户端配置"""
        openai.api_key = self.api_key
        if self.org_id:
            openai.organization = self.org_id

    @APIRateLimiter(calls=50, period=60)  # 每分钟最多50次调用
    def safe_request(self, func: Callable, *args, **kwargs) -> Any:
        """安全的API调用封装"""
        last_error = None
        for attempt in range(self.max_retries):
            try:
                return func(*args, **kwargs, timeout=self.timeout)
            except openai.error.RateLimitError as e:
                wait_time = (attempt + 1) * 2
                self.logger.warning(f"Rate limit hit, waiting {wait_time} seconds...")
                time.sleep(wait_time)
                last_error = e
            except openai.error.APIError as e:
                if e.http_status >= 500:
                    wait_time = (attempt + 1) * 1
                    self.logger.warning(f"API error, waiting {wait_time} seconds...")
                    time.sleep(wait_time)
                    last_error = e
                else:
                    raise
            except Exception as e:
                self.logger.error(f"Unexpected error: {str(e)}")
                raise

        raise last_error or Exception("Max retries exceeded")

    def chat_completion(self, messages: List[Dict[str, str]], **kwargs) -> Dict[str, Any]:
        """安全的聊天完成请求"""
        return self.safe_request(
            openai.ChatCompletion.create,
            messages=messages,
            **kwargs
        )

    def image_generation(self, prompt: str, **kwargs) -> List[Dict[str, str]]:
        """安全的图像生成请求"""
        return self.safe_request(
            openai.Image.create,
            prompt=prompt,
            **kwargs
        )
```

### 使用示例

```python
# 初始化客户端
client = OpenAIClient(
    api_key=os.getenv("OPENAI_API_KEY"),
    org_id=os.getenv("OPENAI_ORG_ID"),
    max_retries=3
)

# 聊天完成
try:
    response = client.chat_completion([
        {"role": "user", "content": "你好，请介绍一下自己"}
    ])
    print(response.choices[0].message.content)
except Exception as e:
    print(f"Error: {str(e)}")

# 图像生成
try:
    images = client.image_generation(
        prompt="一只优雅的猫咪",
        n=1,
        size="512x512"
    )
    print(f"生成的图片URL: {images[0].url}")
except Exception as e:
    print(f"Error: {str(e)}")
```

### 最佳实践建议

1. 错误处理
   - 始终使用try-except捕获可能的API错误
   - 对不同类型的错误实现不同的处理策略
   - 实现合理的重试机制

2. 速率限制
   - 实现令牌桶或滑动窗口算法进行限流
   - 在并发环境下使用分布式限流
   - 监控API使用情况和成本

3. 性能优化
   - 使用异步API调用提高并发性能
   - 实现请求缓存减少重复调用
   - 优化模型参数减少token使用

4. 安全性
   - 使用环境变量管理API密钥
   - 实现访问控制和鉴权
   - 加密敏感数据

5. 监控和日志
   - 实现详细的日志记录
   - 监控API调用状态和性能
   - 设置告警机制