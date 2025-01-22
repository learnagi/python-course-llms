---
title: "LLM数据处理"
slug: "llm-data-handling"
sequence: 4
description: "学习如何在处理LLM API响应时运用Python的各种数据类型，包括实际的API调用示例和数据处理方法"
status: "published"
---

# LLM数据处理

本节将介绍如何在处理大语言模型（LLM）API数据时运用Python的各种数据类型。我们将以实际的API调用和数据处理为例，学习如何有效地处理API响应。

## OpenAI API响应结构

### 基本响应结构
```python
# 典型的OpenAI API响应
response = {
    "id": "chatcmpl-123",
    "object": "chat.completion",
    "created": 1677858242,
    "model": "gpt-3.5-turbo-0613",
    "usage": {
        "prompt_tokens": 56,
        "completion_tokens": 31,
        "total_tokens": 87
    },
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "Hello! How can I help you today?"
            },
            "finish_reason": "stop",
            "index": 0
        }
    ]
}
```

## 数据提取和处理

### 安全地提取数据
```python
def extract_message_content(response):
    """安全地从API响应中提取消息内容"""
    try:
        return response["choices"][0]["message"]["content"]
    except (KeyError, IndexError):
        return None

def extract_usage_info(response):
    """提取使用量信息"""
    usage = response.get("usage", {})
    return {
        "prompt_tokens": usage.get("prompt_tokens", 0),
        "completion_tokens": usage.get("completion_tokens", 0),
        "total_tokens": usage.get("total_tokens", 0)
    }
```

### 处理多轮对话
```python
class Conversation:
    def __init__(self):
        self.messages = []
    
    def add_message(self, role, content):
        """添加新消息到对话历史"""
        message = {
            "role": role,
            "content": content
        }
        self.messages.append(message)
    
    def get_messages(self):
        """获取所有消息"""
        return self.messages
    
    def clear_history(self):
        """清空对话历史"""
        self.messages = []

# 使用示例
conversation = Conversation()
conversation.add_message("user", "你好！")
conversation.add_message("assistant", "你好！有什么我可以帮你的吗？")
conversation.add_message("user", "请介绍一下Python。")
```

## 处理流式响应

### 流式响应处理
```python
def process_stream_response(response_stream):
    """处理流式API响应"""
    collected_chunks = []
    collected_messages = []
    
    # 处理每个数据块
    for chunk in response_stream:
        collected_chunks.append(chunk)
        chunk_message = chunk["choices"][0]["delta"].get("content", "")
        collected_messages.append(chunk_message)
    
    # 组合完整消息
    full_reply_content = "".join(collected_messages)
    return full_reply_content

# 使用示例
async def stream_chat_response(messages):
    try:
        response = await client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages,
            stream=True
        )
        return process_stream_response(response)
    except Exception as e:
        print(f"Error: {e}")
        return None
```

## 数据存储和管理

### 对话历史管理
```python
class ChatHistory:
    def __init__(self):
        self.conversations = {}
    
    def add_conversation(self, user_id):
        """为新用户创建对话"""
        if user_id not in self.conversations:
            self.conversations[user_id] = []
    
    def add_message(self, user_id, role, content):
        """添加消息到用户的对话历史"""
        if user_id not in self.conversations:
            self.add_conversation(user_id)
        
        message = {
            "role": role,
            "content": content,
            "timestamp": time.time()
        }
        self.conversations[user_id].append(message)
    
    def get_conversation(self, user_id, limit=None):
        """获取用户的对话历史"""
        if user_id not in self.conversations:
            return []
        
        history = self.conversations[user_id]
        if limit:
            return history[-limit:]
        return history
    
    def clear_conversation(self, user_id):
        """清空用户的对话历史"""
        if user_id in self.conversations:
            self.conversations[user_id] = []
```

### 结果缓存
```python
class ResponseCache:
    def __init__(self, max_size=1000):
        self.cache = {}
        self.max_size = max_size
        self.timestamps = {}
    
    def add_response(self, prompt, response):
        """缓存API响应"""
        if len(self.cache) >= self.max_size:
            # 删除最旧的条目
            oldest_prompt = min(self.timestamps, key=self.timestamps.get)
            del self.cache[oldest_prompt]
            del self.timestamps[oldest_prompt]
        
        self.cache[prompt] = response
        self.timestamps[prompt] = time.time()
    
    def get_response(self, prompt):
        """获取缓存的响应"""
        return self.cache.get(prompt)
    
    def clear_cache(self):
        """清空缓存"""
        self.cache.clear()
        self.timestamps.clear()
```

## 错误处理和验证

### 响应验证
```python
def validate_api_response(response):
    """验证API响应的完整性"""
    required_fields = {
        "id": str,
        "object": str,
        "created": int,
        "model": str,
        "choices": list
    }
    
    try:
        # 检查必需字段
        for field, field_type in required_fields.items():
            if field not in response:
                raise ValueError(f"Missing required field: {field}")
            if not isinstance(response[field], field_type):
                raise TypeError(f"Invalid type for field {field}")
        
        # 检查choices数组
        if not response["choices"]:
            raise ValueError("Empty choices array")
        
        # 验证消息格式
        message = response["choices"][0].get("message", {})
        if not isinstance(message, dict):
            raise TypeError("Invalid message format")
        
        if "role" not in message or "content" not in message:
            raise ValueError("Missing role or content in message")
        
        return True
    
    except Exception as e:
        print(f"Validation error: {e}")
        return False
```

## 最佳实践

1. 数据提取：
   - 总是使用get()方法安全地访问字典
   - 为所有可能的错误情况提供默认值
   - 使用异常处理捕获意外情况

2. 数据存储：
   - 根据需求选择适当的数据结构
   - 实现清理机制避免内存泄漏
   - 考虑持久化存储重要数据

3. 性能优化：
   - 使用缓存减少API调用
   - 适当清理不再需要的数据
   - 使用异步处理提高响应速度

4. 错误处理：
   - 全面验证API响应
   - 优雅处理各种错误情况
   - 提供有意义的错误信息

## 下一步

现在您已经了解了如何处理LLM API数据，接下来我们将通过一些实践练习来巩固这些知识。
