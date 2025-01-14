# 第六章：JSON数据处理

本章将深入讲解JSON数据的处理，这是在使用LLM API时必不可少的技能。我们将学习如何解析、创建和操作JSON数据。

## 1. JSON基础

### JSON数据类型
```python
# JSON支持的数据类型示例
json_data = {
    "string": "Hello, World",  # 字符串
    "number": 42,             # 数字
    "float": 3.14,           # 浮点数
    "boolean": True,         # 布尔值
    "null": None,           # 空值
    "array": [1, 2, 3],     # 数组
    "object": {              # 对象
        "key": "value"
    }
}
```

### JSON与Python数据类型对应关系
```python
# Python类型 -> JSON类型
python_to_json = {
    "dict": "object",
    "list": "array",
    "str": "string",
    "int/float": "number",
    "True/False": "true/false",
    "None": "null"
}
```

## 2. JSON处理基础

### 读写JSON文件
```python
import json

# 写入JSON文件
data = {
    "name": "Alice",
    "age": 25,
    "skills": ["Python", "API开发"]
}

with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=4)

# 读取JSON文件
with open("data.json", "r", encoding="utf-8") as f:
    loaded_data = json.load(f)
```

### JSON字符串转换
```python
# 字典转JSON字符串
json_string = json.dumps(data, ensure_ascii=False, indent=4)

# JSON字符串转字典
data_dict = json.loads(json_string)
```

## 3. 处理复杂JSON数据

### 嵌套JSON处理
```python
# 复杂的嵌套JSON示例
complex_data = {
    "user": {
        "personal_info": {
            "name": "Alice",
            "contact": {
                "email": "alice@example.com",
                "phone": {
                    "home": "123-456-7890",
                    "work": "098-765-4321"
                }
            }
        },
        "preferences": {
            "theme": "dark",
            "notifications": {
                "email": True,
                "sms": False
            }
        }
    }
}

# 安全地访问嵌套数据
def get_nested_value(data, keys, default=None):
    """安全地获取嵌套字典中的值"""
    current = data
    for key in keys:
        if isinstance(current, dict):
            current = current.get(key, default)
        else:
            return default
    return current

# 使用示例
email = get_nested_value(
    complex_data,
    ["user", "personal_info", "contact", "email"]
)
print(email)  # alice@example.com
```

### JSON数组处理
```python
# 处理JSON数组
users = [
    {"id": 1, "name": "Alice", "active": True},
    {"id": 2, "name": "Bob", "active": False},
    {"id": 3, "name": "Charlie", "active": True}
]

# 过滤数组
active_users = [user for user in users if user["active"]]

# 提取特定字段
user_names = [user["name"] for user in users]

# 转换数据结构
user_dict = {user["id"]: user["name"] for user in users}
```

## 4. LLM API响应处理

### OpenAI API响应处理
```python
def process_chat_response(response):
    """处理OpenAI聊天API的响应"""
    try:
        # 提取主要内容
        message = response["choices"][0]["message"]["content"]
        
        # 提取元数据
        metadata = {
            "id": response["id"],
            "model": response["model"],
            "usage": {
                "prompt_tokens": response["usage"]["prompt_tokens"],
                "completion_tokens": response["usage"]["completion_tokens"],
                "total_tokens": response["usage"]["total_tokens"]
            }
        }
        
        return {
            "content": message,
            "metadata": metadata
        }
    except KeyError as e:
        print(f"响应格式错误: {e}")
        return None
```

### 处理流式响应
```python
def process_stream_response(chunk):
    """处理OpenAI流式响应的单个数据块"""
    try:
        if chunk and "choices" in chunk:
            delta = chunk["choices"][0].get("delta", {})
            return delta.get("content", "")
    except Exception as e:
        print(f"处理响应块失败: {e}")
        return ""

# 使用示例
def collect_stream_response(response_stream):
    """收集完整的流式响应"""
    full_response = ""
    for chunk in response_stream:
        content = process_stream_response(chunk)
        full_response += content
    return full_response
```

## 5. JSON验证和模式

### 基本JSON验证
```python
def validate_json(data):
    """验证JSON数据的基本有效性"""
    try:
        # 尝试序列化和反序列化
        json_string = json.dumps(data)
        json.loads(json_string)
        return True
    except (TypeError, json.JSONDecodeError):
        return False
```

### 使用JSON Schema
```python
from jsonschema import validate, ValidationError

# 定义schema
user_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "number"},
        "email": {"type": "string", "format": "email"},
        "interests": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["name", "email"]
}

def validate_user_data(data):
    """验证用户数据是否符合schema"""
    try:
        validate(instance=data, schema=user_schema)
        return True
    except ValidationError as e:
        print(f"验证失败: {e.message}")
        return False

# 使用示例
user_data = {
    "name": "Alice",
    "age": 25,
    "email": "alice@example.com",
    "interests": ["Python", "AI"]
}

is_valid = validate_user_data(user_data)
```

## 6. 实际应用示例

### 聊天历史管理器
```python
class ChatHistoryManager:
    def __init__(self, file_path="chat_history.json"):
        self.file_path = file_path
        self.history = self.load_history()
    
    def load_history(self):
        """加载聊天历史"""
        try:
            with open(self.file_path, "r", encoding="utf-8") as f:
                return json.load(f)
        except FileNotFoundError:
            return []
    
    def save_history(self):
        """保存聊天历史"""
        with open(self.file_path, "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)
    
    def add_message(self, role, content):
        """添加新消息"""
        message = {
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat()
        }
        self.history.append(message)
        self.save_history()
    
    def get_recent_messages(self, limit=10):
        """获取最近的消息"""
        return self.history[-limit:]
```

### 配置管理器
```python
class ConfigManager:
    def __init__(self, config_file="config.json"):
        self.config_file = config_file
        self.config = self.load_config()
    
    def load_config(self):
        """加载配置"""
        try:
            with open(self.config_file, "r", encoding="utf-8") as f:
                return json.load(f)
        except FileNotFoundError:
            return self.create_default_config()
    
    def create_default_config(self):
        """创建默认配置"""
        default_config = {
            "api_settings": {
                "model": "gpt-3.5-turbo",
                "temperature": 0.7,
                "max_tokens": 1000
            },
            "app_settings": {
                "save_history": True,
                "max_history": 100,
                "language": "zh-CN"
            }
        }
        
        with open(self.config_file, "w", encoding="utf-8") as f:
            json.dump(default_config, f, ensure_ascii=False, indent=4)
        
        return default_config
    
    def update_config(self, key_path, value):
        """更新配置项"""
        current = self.config
        keys = key_path.split(".")
        
        for key in keys[:-1]:
            current = current.setdefault(key, {})
        
        current[keys[-1]] = value
        
        with open(self.config_file, "w", encoding="utf-8") as f:
            json.dump(self.config, f, ensure_ascii=False, indent=4)
```

## 练习题

1. 创建一个简单的JSON数据分析器：
```python
def analyze_json_structure(data):
    """分析JSON数据的结构"""
    def get_type(value):
        if isinstance(value, dict):
            return "object"
        elif isinstance(value, list):
            return "array"
        else:
            return type(value).__name__
    
    def analyze_recursive(data, path=""):
        structure = {}
        
        if isinstance(data, dict):
            for key, value in data.items():
                current_path = f"{path}.{key}" if path else key
                structure[current_path] = {
                    "type": get_type(value),
                    "nested": analyze_recursive(value, current_path) if isinstance(value, (dict, list)) else None
                }
        elif isinstance(data, list) and data:
            structure["items"] = {
                "type": get_type(data[0]),
                "count": len(data),
                "sample": data[0] if not isinstance(data[0], (dict, list)) else analyze_recursive(data[0])
            }
        
        return structure
    
    return analyze_recursive(data)

# 使用示例
test_data = {
    "user": {
        "name": "Alice",
        "scores": [85, 92, 78],
        "contact": {
            "email": "alice@example.com"
        }
    }
}

structure = analyze_json_structure(test_data)
print(json.dumps(structure, indent=2))
```

## 小结

本章我们学习了：
- JSON数据类型和基本操作
- 复杂JSON数据的处理方法
- LLM API响应的处理
- JSON验证和模式
- 实际应用中的JSON处理

这些知识对于处理LLM API的响应数据和构建可靠的应用程序至关重要。

## 下一步

在下一章中，我们将学习文件操作基础，这将帮助我们更好地管理配置文件、日志和其他数据文件。

## 额外资源
- [Python JSON官方文档](https://docs.python.org/zh-cn/3/library/json.html)
- [JSON Schema文档](https://json-schema.org/)
- [Python jsonschema库](https://python-jsonschema.readthedocs.io/)
