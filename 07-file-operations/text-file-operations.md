---
title: "文本文件操作"
slug: "text-file-operations"
sequence: 1
description: "学习Python中的文本文件读写操作，包括基本的文件读写、上下文管理器使用等内容"
is_published: true
estimated_minutes: 15
---

# 文本文件操作

本节将介绍Python中处理文本文件的基本操作，包括文件的读取、写入以及使用上下文管理器进行文件操作。

## 1. 基本文件读写

### 读取文本文件

```python
# 基本文本文件读取
def read_text_file(file_path: str) -> str:
    try:
        with open(file_path, 'r', encoding='utf-8') as file:
            content = file.read()
        return content
    except FileNotFoundError:
        print(f"错误：文件 '{file_path}' 不存在")
        return ""
    except Exception as e:
        print(f"读取文件时发生错误：{str(e)}")
        return ""

# 按行读取文件
def read_lines(file_path: str) -> list[str]:
    try:
        with open(file_path, 'r', encoding='utf-8') as file:
            lines = file.readlines()
        return [line.strip() for line in lines]
    except FileNotFoundError:
        print(f"错误：文件 '{file_path}' 不存在")
        return []
    except Exception as e:
        print(f"读取文件时发生错误：{str(e)}")
        return []
```

### 写入文本文件

```python
# 写入文本文件
def write_text_file(file_path: str, content: str) -> bool:
    try:
        with open(file_path, 'w', encoding='utf-8') as file:
            file.write(content)
        return True
    except Exception as e:
        print(f"写入文件时发生错误：{str(e)}")
        return False

# 追加内容到文件
def append_to_file(file_path: str, content: str) -> bool:
    try:
        with open(file_path, 'a', encoding='utf-8') as file:
            file.write(content)
        return True
    except Exception as e:
        print(f"追加文件时发生错误：{str(e)}")
        return False
```

## 2. 使用上下文管理器

### with语句的使用

```python
# 使用with语句处理文件
def process_file(file_path: str) -> None:
    with open(file_path, 'r', encoding='utf-8') as file:
        content = file.read()
        # 处理文件内容
        print(f"文件内容长度：{len(content)}")
    # 文件会自动关闭

# 同时处理多个文件
def compare_files(file1_path: str, file2_path: str) -> bool:
    with open(file1_path, 'r', encoding='utf-8') as f1, \
         open(file2_path, 'r', encoding='utf-8') as f2:
        return f1.read() == f2.read()
```

## 3. 实践练习

创建一个简单的文本处理程序：

```python
from typing import List

def process_text_file(input_path: str, output_path: str) -> List[str]:
    """读取输入文件，处理内容后写入输出文件"""
    try:
        # 读取文件
        with open(input_path, 'r', encoding='utf-8') as infile:
            lines = infile.readlines()
        
        # 处理内容：去除空行，转换为大写
        processed_lines = [line.strip().upper() for line in lines if line.strip()]
        
        # 写入结果
        with open(output_path, 'w', encoding='utf-8') as outfile:
            outfile.write('\n'.join(processed_lines))
        
        return processed_lines
    
    except Exception as e:
        print(f"处理文件时发生错误：{str(e)}")
        return []

# 使用示例
if __name__ == "__main__":
    result = process_text_file('input.txt', 'output.txt')
    print(f"处理完成，共处理 {len(result)} 行")
```