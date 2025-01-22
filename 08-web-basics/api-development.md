---
title: "API开发实践"
slug: "api-development"
sequence: 2
description: "学习如何使用FastAPI开发LLM服务API，包括请求验证、错误处理和API文档"
is_published: true
estimated_minutes: 25
---

# API开发实践

本节将介绍如何使用FastAPI开发LLM服务API，包括请求验证、错误处理和API文档生成。

## 1. 请求验证

### 使用Pydantic模型

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from typing import Optional, List

app = FastAPI()

class GenerationRequest(BaseModel):
    prompt: str = Field(..., min_length=1, max_length=1000)
    max_tokens: int = Field(default=100, ge=1, le=1000)
    temperature: float = Field(default=0.7, ge=0, le=1)
    n: int = Field(default=1, ge=1, le=5)

class GenerationResponse(BaseModel):
    text: str
    tokens_used: int
    finish_reason: str

@app.post("/generate", response_model=GenerationResponse)
async def generate_text(request: GenerationRequest):
    try:
        # 模拟LLM生成
        response = GenerationResponse(
            text=f"Generated text for: {request.prompt}",
            tokens_used=len(request.prompt.split()),
            finish_reason="length"
        )
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 2. 错误处理

### 自定义异常处理

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

class LLMServiceError(Exception):
    def __init__(self, message: str, error_code: str):
        self.message = message
        self.error_code = error_code

@app.exception_handler(LLMServiceError)
async def llm_service_exception_handler(request, exc: LLMServiceError):
    return JSONResponse(
        status_code=400,
        content={
            "error": {
                "code": exc.error_code,
                "message": exc.message
            }
        }
    )

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "error": {
                "code": "VALIDATION_ERROR",
                "message": "请求参数验证失败",
                "details": exc.errors()
            }
        }
    )
```

## 3. API文档

### 配置Swagger UI

```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI(
    title="LLM Service API",
    description="A RESTful API service for LLM text generation",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title=app.title,
        version=app.version,
        description=app.description,
        routes=app.routes,
    )
    
    # 添加安全配置
    openapi_schema["components"]["securitySchemes"] = {
        "ApiKeyAuth": {
            "type": "apiKey",
            "in": "header",
            "name": "X-API-Key"
        }
    }
    
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

## 4. 实践示例

创建一个完整的LLM服务API：

```python
from fastapi import FastAPI, HTTPException, Depends, Header
from pydantic import BaseModel, Field
from typing import Optional, List
import time

app = FastAPI()

# 模拟API密钥验证
def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != "your-api-key":
        raise HTTPException(
            status_code=401,
            detail="Invalid API key"
        )
    return x_api_key

class ChatMessage(BaseModel):
    role: str = Field(..., regex='^(user|assistant|system)$')
    content: str = Field(..., min_length=1)

class ChatRequest(BaseModel):
    messages: List[ChatMessage]
    max_tokens: int = Field(default=100, ge=1, le=2000)
    temperature: float = Field(default=0.7, ge=0, le=1)

class ChatResponse(BaseModel):
    message: ChatMessage
    usage: dict
    created: int

@app.post("/chat", response_model=ChatResponse)
async def chat_completion(
    request: ChatRequest,
    api_key: str = Depends(verify_api_key)
):
    try:
        # 模拟聊天完成
        response = ChatResponse(
            message=ChatMessage(
                role="assistant",
                content=f"This is a response to: {request.messages[-1].content}"
            ),
            usage={
                "prompt_tokens": sum(len(m.content.split()) for m in request.messages),
                "completion_tokens": 20,
                "total_tokens": 20 + sum(len(m.content.split()) for m in request.messages)
            },
            created=int(time.time())
        )
        return response
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=str(e)
        )

# 启动命令：uvicorn main:app --reload
```