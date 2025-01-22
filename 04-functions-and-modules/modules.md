---
title: "模块使用"
slug: "modules"
sequence: 3
description: "学习Python模块的创建、导入和使用，以及如何组织代码到包中"
status: "published"
---

# 模块使用

本节将介绍Python模块的使用，包括如何创建和导入模块，如何组织代码到包中，以及一些常用的内置模块。

## 模块基础

### 创建模块
```python
# math_utils.py
"""数学工具模块"""

def add(x, y):
    """加法函数"""
    return x + y

def multiply(x, y):
    """乘法函数"""
    return x * y

PI = 3.14159

class Circle:
    """圆类"""
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        """计算面积"""
        return PI * self.radius ** 2
```

### 导入模块
```python
# 导入整个模块
import math_utils

# 使用模块中的函数
result = math_utils.add(3, 5)

# 导入特定的函数或变量
from math_utils import add, PI

# 使用导入的函数
result = add(3, 5)

# 使用别名
import math_utils as mu
result = mu.multiply(3, 4)

# 导入所有内容（不推荐）
from math_utils import *
```

## 包的创建和组织

### 包结构
```
my_llm_package/
│
├── __init__.py
├── api/
│   ├── __init__.py
│   ├── client.py
│   └── models.py
│
├── utils/
│   ├── __init__.py
│   ├── text.py
│   └── tokens.py
│
└── config/
    ├── __init__.py
    └── settings.py
```

### 包的初始化
```python
# my_llm_package/__init__.py
"""
LLM工具包
"""

from .api.client import APIClient
from .utils.text import preprocess_text
from .config.settings import load_config

__version__ = "1.0.0"
```

### 子模块
```python
# my_llm_package/api/client.py
"""API客户端模块"""

class APIClient:
    def __init__(self, api_key):
        self.api_key = api_key
    
    def call_api(self, prompt):
        # API调用实现
        pass

# my_llm_package/utils/text.py
"""文本处理工具"""

def preprocess_text(text):
    """预处理文本"""
    return text.strip().lower()

# my_llm_package/config/settings.py
"""配置管理"""

def load_config(config_file):
    """加载配置文件"""
    pass
```

## 模块导入机制

### 导入搜索路径
```python
import sys

# 查看模块搜索路径
for path in sys.path:
    print(path)

# 添加搜索路径
sys.path.append("/path/to/my/modules")
```

### 相对导入
```python
# my_llm_package/api/models.py
from ..utils.text import preprocess_text
from ..config.settings import load_config

# 显式相对导入
from . import client
```

## 常用内置模块

### os模块
```python
import os

# 文件和目录操作
current_dir = os.getcwd()
os.makedirs("new_directory", exist_ok=True)
os.path.join("path", "to", "file")

# 环境变量
api_key = os.getenv("API_KEY")
os.environ["DEBUG"] = "true"
```

### json模块
```python
import json

# 写入JSON
data = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}
with open("data.json", "w") as f:
    json.dump(data, f, indent=4)

# 读取JSON
with open("data.json", "r") as f:
    loaded_data = json.load(f)

# 字符串转换
json_str = json.dumps(data)
parsed_data = json.loads(json_str)
```

### datetime模块
```python
from datetime import datetime, timedelta

# 获取当前时间
now = datetime.now()
utc_now = datetime.utcnow()

# 时间运算
tomorrow = now + timedelta(days=1)
one_hour_later = now + timedelta(hours=1)

# 格式化时间
formatted = now.strftime("%Y-%m-%d %H:%M:%S")
parsed = datetime.strptime("2023-01-01", "%Y-%m-%d")
```

### logging模块
```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

# 创建logger
logger = logging.getLogger(__name__)

# 使用日志
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

## 实际应用示例

### LLM API包结构
```python
# llm_toolkit/api/base.py
class BaseClient:
    """基础API客户端"""
    def __init__(self, api_key):
        self.api_key = api_key
    
    def validate_key(self):
        raise NotImplementedError

# llm_toolkit/api/openai_client.py
from .base import BaseClient

class OpenAIClient(BaseClient):
    """OpenAI API客户端"""
    def __init__(self, api_key):
        super().__init__(api_key)
        self.base_url = "https://api.openai.com/v1"
    
    def chat_completion(self, messages):
        # 实现chat completion
        pass

# llm_toolkit/utils/rate_limiter.py
class RateLimiter:
    """API速率限制器"""
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []
    
    def can_proceed(self):
        # 实现速率限制逻辑
        pass
```

### 使用示例
```python
# 使用包
from llm_toolkit.api.openai_client import OpenAIClient
from llm_toolkit.utils.rate_limiter import RateLimiter

# 创建客户端
client = OpenAIClient(api_key="your-api-key")
limiter = RateLimiter(max_requests=60, time_window=60)

# 调用API
if limiter.can_proceed():
    response = client.chat_completion([
        {"role": "user", "content": "Hello!"}
    ])
```

## 最佳实践

1. 模块组织：
   - 相关功能放在同一模块
   - 使用有意义的模块名
   - 保持模块功能单一

2. 导入规范：
   - 导入放在文件开头
   - 避免使用`from module import *`
   - 按标准库、第三方库、本地模块顺序导入

3. 包设计：
   - 使用清晰的目录结构
   - 提供必要的文档
   - 实现版本控制

4. 代码重用：
   - 提取共用功能到工具模块
   - 使用基类实现共同特性
   - 避免代码重复

## 下一步

现在您已经了解了Python模块的使用方法，接下来我们将学习如何封装LLM API，创建易用的客户端库。
