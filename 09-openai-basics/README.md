# 第九章：OpenAI API深入使用

本章将深入介绍OpenAI API的使用方法，包括不同模型的特点、参数调优、最佳实践等。

## 1. OpenAI API基础

### 环境设置
```python
import openai
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 设置API密钥
openai.api_key = os.getenv("OPENAI_API_KEY")
```

### 基本API调用
```python
def chat_completion(prompt, model="gpt-3.5-turbo"):
    """基本的聊天完成调用"""
    try:
        response = openai.ChatCompletion.create(
            model=model,
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message.content
    except Exception as e:
        print(f"API调用错误: {e}")
        return None
```

## 2. 不同模型的使用

### GPT-3.5-Turbo
```python
def chat_with_gpt35(prompt, temperature=0.7):
    """使用GPT-3.5-Turbo进行对话"""
    return openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "user", "content": prompt}
        ],
        temperature=temperature
    )

# 使用示例
response = chat_with_gpt35("解释Python的列表推导式")
```

### GPT-4
```python
def chat_with_gpt4(prompt, temperature=0.7):
    """使用GPT-4进行对话（需要访问权限）"""
    return openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "user", "content": prompt}
        ],
        temperature=temperature
    )
```

## 3. 参数优化

### 温度和采样
```python
def generate_with_params(prompt, temperature=0.7, top_p=1.0):
    """使用不同的生成参数"""
    return openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        temperature=temperature,  # 控制随机性 (0-2)
        top_p=top_p,            # 核采样 (0-1)
        n=1,                    # 生成数量
        stream=False,           # 是否流式响应
        presence_penalty=0,     # 重复惩罚
        frequency_penalty=0     # 频率惩罚
    )
```

### 角色设定
```python
def chat_with_system_prompt(prompt, system_prompt):
    """使用系统提示设定角色"""
    return openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ]
    )

# 使用示例
system_prompt = "你是一个Python专家，专注于帮助初学者理解代码。"
response = chat_with_system_prompt("解释装饰器", system_prompt)
```

## 4. 流式响应处理

### 基本流式响应
```python
def stream_chat(prompt):
    """处理流式响应"""
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    for chunk in response:
        if chunk and chunk.choices and chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content

# 使用示例
for text in stream_chat("讲个故事"):
    print(text, end="", flush=True)
```

### 异步流式处理
```python
async def async_stream_chat(prompt):
    """异步处理流式响应"""
    response = await openai.ChatCompletion.acreate(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    async for chunk in response:
        if chunk and chunk.choices and chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content

# 使用示例
async def main():
    async for text in async_stream_chat("讲个故事"):
        print(text, end="", flush=True)
```

## 5. 错误处理和重试

### 错误处理
```python
class OpenAIError(Exception):
    """自定义OpenAI错误"""
    pass

def safe_chat_completion(prompt, max_retries=3):
    """带错误处理的API调用"""
    for attempt in range(max_retries):
        try:
            response = openai.ChatCompletion.create(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": prompt}]
            )
            return response.choices[0].message.content
        
        except openai.error.RateLimitError:
            wait_time = (attempt + 1) * 2
            print(f"达到速率限制，等待{wait_time}秒...")
            time.sleep(wait_time)
        
        except openai.error.APIError as e:
            print(f"API错误: {e}")
            if attempt == max_retries - 1:
                raise OpenAIError(f"达到最大重试次数: {e}")
        
        except Exception as e:
            print(f"未知错误: {e}")
            raise OpenAIError(f"未知错误: {e}")
```

## 6. 高级应用

### 对话管理器
```python
class ConversationManager:
    def __init__(self, model="gpt-3.5-turbo", max_tokens=4096):
        self.model = model
        self.max_tokens = max_tokens
        self.conversations = {}
    
    def add_message(self, session_id, role, content):
        """添加消息到对话历史"""
        if session_id not in self.conversations:
            self.conversations[session_id] = []
        
        self.conversations[session_id].append({
            "role": role,
            "content": content
        })
        
        # 简单的上下文长度管理
        while self._estimate_tokens(self.conversations[session_id]) > self.max_tokens:
            self.conversations[session_id].pop(0)
    
    def _estimate_tokens(self, messages):
        """粗略估计token数量"""
        return sum(len(msg["content"]) / 4 for msg in messages)
    
    async def get_response(self, session_id, message):
        """获取回复"""
        self.add_message(session_id, "user", message)
        
        try:
            response = await openai.ChatCompletion.acreate(
                model=self.model,
                messages=self.conversations[session_id]
            )
            
            reply = response.choices[0].message.content
            self.add_message(session_id, "assistant", reply)
            
            return reply
        
        except Exception as e:
            print(f"获取回复失败: {e}")
            return None
```

### 函数调用
```python
def chat_with_function(prompt, functions):
    """使用函数调用功能"""
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        functions=functions,
        function_call="auto"
    )
    
    return response

# 使用示例
functions = [
    {
        "name": "get_weather",
        "description": "获取指定城市的天气信息",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "城市名称"
                }
            },
            "required": ["city"]
        }
    }
]

response = chat_with_function("北京今天天气怎么样？", functions)
```

## 7. 实际应用示例

### 智能文档分析器
```python
class DocumentAnalyzer:
    def __init__(self, model="gpt-3.5-turbo"):
        self.model = model
    
    def analyze_document(self, text):
        """分析文档内容"""
        prompts = [
            "提供文档的主要观点总结",
            "识别文档中的关键概念",
            "提出需要澄清或补充的问题"
        ]
        
        results = {}
        for prompt in prompts:
            try:
                response = openai.ChatCompletion.create(
                    model=self.model,
                    messages=[
                        {"role": "system", "content": "你是一个专业的文档分析专家。"},
                        {"role": "user", "content": f"{prompt}\n\n文档内容：\n{text}"}
                    ]
                )
                results[prompt] = response.choices[0].message.content
            except Exception as e:
                results[prompt] = f"分析失败: {e}"
        
        return results

# 使用示例
analyzer = DocumentAnalyzer()
analysis = analyzer.analyze_document("你的文档内容...")
```

### 代码助手
```python
class CodeAssistant:
    def __init__(self, model="gpt-3.5-turbo"):
        self.model = model
    
    def explain_code(self, code):
        """解释代码"""
        prompt = f"请解释以下代码的功能和工作原理：\n\n```python\n{code}\n```"
        return self._get_response(prompt)
    
    def suggest_improvements(self, code):
        """建议改进"""
        prompt = f"请分析以下代码，并提供改进建议：\n\n```python\n{code}\n```"
        return self._get_response(prompt)
    
    def add_comments(self, code):
        """添加注释"""
        prompt = f"请为以下代码添加详细的中文注释：\n\n```python\n{code}\n```"
        return self._get_response(prompt)
    
    def _get_response(self, prompt):
        try:
            response = openai.ChatCompletion.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": "你是一个Python专家，专注于代码分析和改进。"},
                    {"role": "user", "content": prompt}
                ]
            )
            return response.choices[0].message.content
        except Exception as e:
            return f"处理失败: {e}"

# 使用示例
assistant = CodeAssistant()
code = """
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
"""
explanation = assistant.explain_code(code)
improvements = assistant.suggest_improvements(code)
commented_code = assistant.add_comments(code)
```

## 8. 练习题

1. 创建一个多模型对比工具：
```python
class ModelComparator:
    def __init__(self):
        self.models = ["gpt-3.5-turbo", "gpt-4"]  # 需要GPT-4访问权限
    
    async def compare_responses(self, prompt):
        """比较不同模型的响应"""
        results = {}
        
        for model in self.models:
            try:
                response = await openai.ChatCompletion.acreate(
                    model=model,
                    messages=[{"role": "user", "content": prompt}]
                )
                results[model] = {
                    "response": response.choices[0].message.content,
                    "tokens": response.usage.total_tokens,
                    "time": datetime.now().isoformat()
                }
            except Exception as e:
                results[model] = {
                    "error": str(e)
                }
        
        return results

# 使用示例
async def main():
    comparator = ModelComparator()
    results = await comparator.compare_responses("解释量子计算的基本原理")
    print(json.dumps(results, indent=2, ensure_ascii=False))
```

## 小结

本章我们学习了：
- OpenAI API的基本使用
- 不同模型的特点和选择
- 参数优化和调整
- 流式响应处理
- 错误处理和重试机制
- 高级应用开发

这些知识将帮助你更好地使用OpenAI API，构建更可靠、更高效的LLM应用。

## 下一步

在下一章中，我们将学习如何构建完整的对话系统，包括上下文管理、多轮对话等功能。

## 额外资源
- [OpenAI API文档](https://platform.openai.com/docs/api-reference)
- [OpenAI Cookbook](https://github.com/openai/openai-cookbook)
- [OpenAI Python库](https://github.com/openai/openai-python)
