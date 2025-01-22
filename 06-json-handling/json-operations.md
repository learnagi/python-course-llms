---
title: "JSON操作基础"
slug: "json-operations"
sequence: 1
description: "学习Python中的JSON数据处理基础，包括JSON数据类型、读写操作以及常见应用场景"
is_published: true
estimated_minutes: 20
---

# JSON操作基础

本节将介绍Python中处理JSON数据的基本操作，包括数据类型转换、读写操作以及在API开发中的应用。

## 1. JSON数据类型

### 基本数据类型

```python
# JSON支持的基本数据类型示例
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

### Python与JSON类型映射

```python
# Python类型与JSON类型的对应关系
python_to_json = {
    "dict": "object",
    "list/tuple": "array",
    "str": "string",
    "int/float": "number",
    "True/False": "true/false",
    "None": "null"
}
```

## 2. JSON操作

### 基本读写操作

```python
import json

# 写入JSON文件
def write_json_file(file_path: str, data: dict) -> bool:
    try:
        with open(file_path, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=4)
        return True
    except Exception as e:
        print(f"写入JSON文件时发生错误：{str(e)}")
        return False

# 读取JSON文件
def read_json_file(file_path: str) -> dict:
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            data = json.load(f)
        return data
    except FileNotFoundError:
        print(f"错误：文件 '{file_path}' 不存在")
        return {}
    except json.JSONDecodeError:
        print("错误：JSON格式不正确")
        return {}
    except Exception as e:
        print(f"读取JSON文件时发生错误：{str(e)}")
        return {}
```

### 字符串转换

```python
# JSON字符串与Python对象转换
def json_str_to_dict(json_str: str) -> dict:
    try:
        return json.loads(json_str)
    except json.JSONDecodeError:
        print("错误：JSON字符串格式不正确")
        return {}

def dict_to_json_str(data: dict) -> str:
    try:
        return json.dumps(data, ensure_ascii=False, indent=4)
    except Exception as e:
        print(f"转换为JSON字符串时发生错误：{str(e)}")
        return ""
```

## 3. 实践应用

### API响应处理

```python
# API响应JSON处理示例
def process_api_response(response_text: str) -> dict:
    """处理API返回的JSON响应"""
    try:
        # 解析JSON响应
        data = json.loads(response_text)
        
        # 提取所需字段
        result = {
            "status": data.get("status", ""),
            "message": data.get("message", ""),
            "data": data.get("data", {})
        }
        
        return result
    except json.JSONDecodeError:
        return {
            "status": "error",
            "message": "Invalid JSON response",
            "data": {}
        }
    except Exception as e:
        return {
            "status": "error",
            "message": str(e),
            "data": {}
        }

# 使用示例
if __name__ == "__main__":
    # 示例JSON响应
    sample_response = '''
    {
        "status": "success",
        "message": "Data retrieved successfully",
        "data": {
            "user": {
                "id": 1,
                "name": "Alice",
                "email": "alice@example.com"
            }
        }
    }
    '''
    
    result = process_api_response(sample_response)
    print(json.dumps(result, ensure_ascii=False, indent=4))
```