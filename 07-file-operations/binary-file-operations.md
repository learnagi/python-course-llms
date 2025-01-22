---
title: "二进制文件操作"
slug: "binary-file-operations"
sequence: 2
description: "学习Python中的二进制文件操作基础，包括图片文件的读写处理"
is_published: true
estimated_minutes: 15
---

# 二进制文件操作

本节将介绍Python中处理二进制文件的基本操作，包括图片、音频等文件的读写。

## 1. 基本二进制文件操作
### 读取二进制文件

```python
# 读取二进制文件
def read_binary_file(file_path: str) -> bytes:
    try:
        with open(file_path, 'rb') as file:
            content = file.read()
        return content
    except FileNotFoundError:
        print(f"错误：文件 '{file_path}' 不存在")
        return b""
    except Exception as e:
        print(f"读取文件时发生错误：{str(e)}")
        return b""

# 写入二进制文件
def write_binary_file(file_path: str, content: bytes) -> bool:
    try:
        with open(file_path, 'wb') as file:
            file.write(content)
        return True
    except Exception as e:
        print(f"写入文件时发生错误：{str(e)}")
        return False
```

## 2. 图片文件处理

### 使用Pillow库处理图片

```python
from PIL import Image
from typing import Tuple, Optional

# 读取图片文件
def read_image(file_path: str) -> Optional[Image.Image]:
    try:
        return Image.open(file_path)
    except Exception as e:
        print(f"读取图片时发生错误：{str(e)}")
        return None

# 调整图片大小
def resize_image(image: Image.Image, size: Tuple[int, int]) -> Optional[Image.Image]:
    try:
        return image.resize(size)
    except Exception as e:
        print(f"调整图片大小时发生错误：{str(e)}")
        return None

# 保存图片文件
def save_image(image: Image.Image, file_path: str) -> bool:
    try:
        image.save(file_path)
        return True
    except Exception as e:
        print(f"保存图片时发生错误：{str(e)}")
        return False
```

## 3. 实践练习

创建一个简单的图片处理程序：

```python
from PIL import Image
from typing import Optional, Tuple

def process_image(input_path: str, output_path: str, size: Tuple[int, int]) -> bool:
    """处理图片：读取、调整大小并保存"""
    try:
        # 读取图片
        with Image.open(input_path) as img:
            # 调整大小
            resized_img = img.resize(size)
            # 保存结果
            resized_img.save(output_path)
        return True
    except Exception as e:
        print(f"处理图片时发生错误：{str(e)}")
        return False

# 使用示例
if __name__ == "__main__":
    # 处理图片
    success = process_image('input.jpg', 'output.jpg', (800, 600))
    if success:
        print("图片处理完成")
    else:
        print("图片处理失败")
```