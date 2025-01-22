---
title: "FastAPI基础"
slug: "fastapi-basics"
sequence: 1
description: "学习FastAPI框架的基本概念和使用方法，包括路由、请求处理和响应处理"
is_published: true
estimated_minutes: 20
---

# FastAPI基础

本节将介绍FastAPI框架的基本概念和使用方法，包括如何创建API端点、处理请求和响应。

## 1. 安装和基本设置

### 环境准备

```python
# 安装FastAPI和ASGI服务器
pip install fastapi uvicorn

# 安装类型提示支持
pip install pydantic
```

### 创建基本应用

```python
from fastapi import FastAPI
from pydantic import BaseModel

# 创建FastAPI应用实例
app = FastAPI(
    title="LLM API Service",
    description="A simple API service for LLM applications",
    version="1.0.0"
)

# 基本路由
@app.get("/")
async def root():
    return {"message": "Hello, LLM World!"}

# 启动服务器
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 2. 路由和请求处理

### 路径参数

```python
from typing import Optional

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}
```

### 请求体

```python
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float

@app.post("/items/")
async def create_item(item: Item):
    return item
```

## 3. 响应处理

### 状态码和响应模型

```python
from fastapi import HTTPException, status
from typing import List

class ResponseModel(BaseModel):
    message: str
    data: Optional[dict] = None

@app.get("/items/{item_id}", response_model=ResponseModel)
async def get_item(item_id: int):
    if item_id == 0:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found"
        )
    return ResponseModel(
        message="Success",
        data={"item_id": item_id}
    )
```

## 4. 实践示例

创建一个简单的LLM API服务：

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class TextRequest(BaseModel):
    text: str
    max_length: Optional[int] = 100

class TextResponse(BaseModel):
    original_text: str
    processed_text: str
    length: int

@app.post("/process", response_model=TextResponse)
async def process_text(request: TextRequest):
    try:
        # 模拟文本处理
        processed = request.text.upper()
        if len(processed) > request.max_length:
            processed = processed[:request.max_length] + "..."
        
        return TextResponse(
            original_text=request.text,
            processed_text=processed,
            length=len(processed)
        )
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )

# 启动命令：uvicorn main:app --reload
```