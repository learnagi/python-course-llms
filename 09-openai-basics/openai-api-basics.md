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

```python
import os
import openai
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 设置API密钥
openai.api_key = os.getenv("OPENAI_API_KEY")
```

## 2. 文本生成

### 使用GPT模型

```python
from typing import Optional

def generate_text(prompt: str, max_tokens: int = 100) -> Optional[str]:
    """使用GPT模型生成文本"""
    try:
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=prompt,
            max_tokens=max_tokens,
            temperature=0.7
        )
        return response.choices[0].text.strip()
    except Exception as e:
        print(f"生成文本时发生错误：{str(e)}")
        return None

# 使用示例
prompt = "写一个简短的Python函数来计算斐波那契数列"
result = generate_text(prompt)
print(result)
```

## 3. 聊天对话

### Chat Completion API

```python
from typing import List, Dict

def chat_with_gpt(messages: List[Dict[str, str]]) -> Optional[str]:
    """使用Chat API进行对话"""
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
        return response.choices[0].message.content
    except Exception as e:
        print(f"对话时发生错误：{str(e)}")
        return None

# 对话示例
messages = [
    {"role": "system", "content": "你是一个Python编程助手。"},
    {"role": "user", "content": "如何在Python中读取JSON文件？"}
]

response = chat_with_gpt(messages)
print(response)
```

### 对话上下文管理

```python
class ChatSession:
    def __init__(self, system_message: str):
        self.messages = [{"role": "system", "content": system_message}]
    
    def add_message(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})
    
    def get_response(self) -> Optional[str]:
        try:
            response = openai.ChatCompletion.create(
                model="gpt-3.5-turbo",
                messages=self.messages
            )
            message = response.choices[0].message.content
            self.add_message("assistant", message)
            return message
        except Exception as e:
            print(f"获取响应时发生错误：{str(e)}")
            return None

# 使用示例
chat = ChatSession("你是一个Python编程助手。")
chat.add_message("user", "如何使用Python处理Excel文件？")
print(chat.get_response())
```

## 4. 图像生成

### DALL-E API

```python
from typing import Optional
import base64

def generate_image(prompt: str, size: str = "512x512") -> Optional[str]:
    """使用DALL-E生成图像"""
    try:
        response = openai.Image.create(
            prompt=prompt,
            n=1,
            size=size
        )
        return response.data[0].url
    except Exception as e:
        print(f"生成图像时发生错误：{str(e)}")
        return None

def edit_image(image_path: str, mask_path: str, prompt: str) -> Optional[str]:
    """编辑现有图像"""
    try:
        with open(image_path, "rb") as image_file:
            with open(mask_path, "rb") as mask_file:
                response = openai.Image.create_edit(
                    image=image_file,
                    mask=mask_file,
                    prompt=prompt,
                    n=1,
                    size="512x512"
                )
                return response.data[0].url
    except Exception as e:
        print(f"编辑图像时发生错误：{str(e)}")
        return None

# 使用示例
prompt = "一只可爱的卡通猫咪"
image_url = generate_image(prompt)
print(f"生成的图像URL：{image_url}")
```

## 5. 错误处理和最佳实践

### API调用封装

```python
from typing import Optional, Dict, Any
import time

class OpenAIAPI:
    def __init__(self, api_key: str):
        self.api_key = api_key
        openai.api_key = api_key
        self.retry_count = 3
        self.retry_delay = 1
    
    def _handle_api_error(self, e: Exception) -> None:
        """处理API错误"""
        if hasattr(e, 'response'):
            status = e.response.status
            if status == 429:  # Rate limit
                time.sleep(self.retry_delay)
            elif status == 500:  # Server error
                time.sleep(self.retry_delay * 2)
    
    def chat_completion(self, messages: List[Dict[str, str]]) -> Optional[str]:
        """带重试机制的聊天完成"""
        for attempt in range(self.retry_count):
            try:
                response = openai.ChatCompletion.create(
                    model="gpt-3.5-turbo",
                    messages=messages
                )
                return response.choices[0].message.content
            except Exception as e:
                self._handle_api_error(e)
                if attempt == self.retry_count - 1:
                    print(f"达到最大重试次数：{str(e)}")
                    return None

# 使用示例
api = OpenAIAPI(os.getenv("OPENAI_API_KEY"))
messages = [
    {"role": "user", "content": "Python中如何实现多线程？"}
]
response = api.chat_completion(messages)
print(response)
```