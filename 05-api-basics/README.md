# 第五章：API调用基础

本章将介绍如何使用Python调用API，这是与LLM服务交互的基础。我们将从基本的HTTP请求开始，逐步过渡到实际的LLM API调用。

## 1. HTTP请求基础

### 安装requests库
```bash
pip install requests
```

### 基本GET请求
```python
import requests

# 发送GET请求
response = requests.get("https://api.example.com/data")

# 检查响应状态
print(response.status_code)  # 200表示成功

# 获取响应内容
print(response.text)  # 文本形式
print(response.json())  # JSON形式（如果响应是JSON）
```

### POST请求
```python
# 发送POST请求
data = {
    "name": "Alice",
    "age": 25
}

response = requests.post(
    "https://api.example.com/users",
    json=data  # 自动转换为JSON
)
```

## 2. API认证

### API密钥认证
```python
# 在header中使用API密钥
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}

response = requests.get(
    "https://api.example.com/data",
    headers=headers
)
```

### 环境变量管理
```python
# .env文件
OPENAI_API_KEY=your-api-key-here

# Python代码
from dotenv import load_dotenv
import os

load_dotenv()  # 加载.env文件
api_key = os.getenv("OPENAI_API_KEY")
```

## 3. 错误处理

### 基本错误处理
```python
def make_api_call(url, headers=None):
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()  # 抛出HTTP错误
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"API调用失败: {e}")
        return None
```

### 重试机制
```python
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

# 配置重试策略
retry_strategy = Retry(
    total=3,  # 最大重试次数
    backoff_factor=1,  # 重试间隔
    status_forcelist=[500, 502, 503, 504]  # 需要重试的HTTP状态码
)

# 创建会话
session = requests.Session()
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)
```

## 4. OpenAI API调用

### 基本聊天完成
```python
import openai
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

def chat_with_gpt(prompt):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message.content
    except Exception as e:
        print(f"API调用失败: {e}")
        return None

# 使用示例
response = chat_with_gpt("你好，请介绍一下Python。")
print(response)
```

### 流式响应
```python
def stream_chat_with_gpt(prompt):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt}
            ],
            stream=True  # 启用流式响应
        )
        
        for chunk in response:
            if chunk and chunk.choices and chunk.choices[0].delta.content:
                print(chunk.choices[0].delta.content, end="")
    except Exception as e:
        print(f"API调用失败: {e}")

# 使用示例
stream_chat_with_gpt("讲个故事")
```

## 5. API响应处理

### JSON处理
```python
import json

# 解析JSON
def parse_response(response_text):
    try:
        data = json.loads(response_text)
        return data
    except json.JSONDecodeError:
        print("JSON解析失败")
        return None

# 创建JSON
data = {
    "name": "Alice",
    "age": 25,
    "skills": ["Python", "API开发"]
}
json_string = json.dumps(data, ensure_ascii=False)
```

### 响应数据提取
```python
def extract_completion(response):
    """从OpenAI API响应中提取生成的文本"""
    if response and "choices" in response:
        return response["choices"][0]["message"]["content"]
    return None

def extract_embedding(response):
    """从OpenAI API响应中提取嵌入向量"""
    if response and "data" in response:
        return response["data"][0]["embedding"]
    return None
```

## 6. 实际应用示例

### 简单的翻译器
```python
def translate_text(text, target_language):
    prompt = f"将以下文本翻译成{target_language}：\n{text}"
    
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message.content
    except Exception as e:
        print(f"翻译失败: {e}")
        return None

# 使用示例
english_text = "Hello, how are you?"
chinese = translate_text(english_text, "中文")
print(chinese)
```

### 文本分析器
```python
def analyze_sentiment(text):
    prompt = f"""
    分析以下文本的情感倾向，返回以下JSON格式：
    {{
        "sentiment": "positive/negative/neutral",
        "confidence": 0-1之间的数值,
        "explanation": "简短解释"
    }}
    
    文本：{text}
    """
    
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return json.loads(response.choices[0].message.content)
    except Exception as e:
        print(f"分析失败: {e}")
        return None

# 使用示例
text = "这个产品太棒了，我非常喜欢！"
analysis = analyze_sentiment(text)
print(analysis)
```

## 练习题

1. 创建一个简单的天气查询API客户端：
```python
import requests
from dotenv import load_dotenv
import os

load_dotenv()
WEATHER_API_KEY = os.getenv("WEATHER_API_KEY")

def get_weather(city):
    url = f"http://api.weatherapi.com/v1/current.json"
    params = {
        "key": WEATHER_API_KEY,
        "q": city
    }
    
    try:
        response = requests.get(url, params=params)
        response.raise_for_status()
        data = response.json()
        
        return {
            "temperature": data["current"]["temp_c"],
            "condition": data["current"]["condition"]["text"],
            "humidity": data["current"]["humidity"]
        }
    except Exception as e:
        print(f"获取天气信息失败: {e}")
        return None

# 使用示例
weather = get_weather("Beijing")
if weather:
    print(f"温度: {weather['temperature']}°C")
    print(f"天气: {weather['condition']}")
    print(f"湿度: {weather['humidity']}%")
```

## 小结

本章我们学习了：
- HTTP请求的基础知识
- API认证和安全性
- 错误处理和重试机制
- OpenAI API的基本使用
- JSON数据处理
- 实际API应用开发

这些知识将帮助你更好地理解和使用各种API，特别是LLM相关的API服务。

## 下一步

在下一章中，我们将深入学习JSON数据处理，这对于处理API响应数据至关重要。

## 额外资源
- [Requests库文档](https://docs.python-requests.org/)
- [OpenAI API文档](https://platform.openai.com/docs/api-reference)
- [Python JSON处理](https://docs.python.org/zh-cn/3/library/json.html)
