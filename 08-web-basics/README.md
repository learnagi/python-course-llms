# 第八章：FastAPI Web应用开发基础

本章将介绍如何使用FastAPI框架创建简单的Web应用，重点是构建LLM服务的API接口。FastAPI是一个现代、快速、简单的Web框架，特别适合构建API服务。

## 1. FastAPI基础

### 安装和基本设置
```python
# 安装FastAPI和ASGI服务器
pip install fastapi uvicorn

# 基本FastAPI应用
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello, LLM World!"}

# 运行应用
# 命令行：uvicorn main:app --reload
```

### 基本路由和请求处理
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

# 定义请求模型
class Message(BaseModel):
    content: str

# GET请求
@app.get("/api/hello")
async def hello(name: str = "World"):
    return {"message": f"Hello, {name}!"}

# POST请求
@app.post("/api/chat")
async def chat(message: Message):
    try:
        # 这里可以调用OpenAI API
        response = "这是回复消息"
        return {"response": response}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 2. 创建LLM API服务

### 基本聊天API
```python
import openai
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import os
from typing import List, Optional

app = FastAPI()

# 配置OpenAI API密钥
openai.api_key = os.getenv("OPENAI_API_KEY")

class ChatMessage(BaseModel):
    content: str
    
class ChatResponse(BaseModel):
    response: str
    
@app.post("/api/chat", response_model=ChatResponse)
async def chat(message: ChatMessage):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": message.content}
            ]
        )
        
        return ChatResponse(
            response=response.choices[0].message.content
        )
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### 对话历史管理
```python
from datetime import datetime
from typing import List, Dict

class Conversation(BaseModel):
    role: str
    content: str
    timestamp: Optional[datetime] = None

class ConversationManager:
    def __init__(self):
        self.conversations: Dict[str, List[Dict]] = {}
        self.max_history = 10
    
    def add_message(self, session_id: str, role: str, content: str):
        if session_id not in self.conversations:
            self.conversations[session_id] = []
        
        message = Message(
            role=role,
            content=content,
            timestamp=datetime.now()
        )
        
        self.conversations[session_id].append(message)
        
        # 保持历史记录在限制范围内
        if len(self.conversations[session_id]) > self.max_history:
            self.conversations[session_id] = (
                self.conversations[session_id][-self.max_history:]
            )
    
    def get_messages(self, session_id: str) -> List[Dict]:
        messages = self.conversations.get(session_id, [])
        return [
            {"role": msg.role, "content": msg.content}
            for msg in messages
        ]

conversation_manager = ConversationManager()

class ChatRequest(BaseModel):
    message: str
    session_id: Optional[str] = "default"

@app.post("/api/chat/{session_id}")
async def chat_with_history(session_id: str, request: ChatRequest):
    try:
        # 添加用户消息
        conversation_manager.add_message(
            session_id,
            "user",
            request.message
        )
        
        # 获取对话历史
        messages = conversation_manager.get_messages(session_id)
        
        # 调用OpenAI API
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
        
        reply = response.choices[0].message.content
        
        # 添加助手回复
        conversation_manager.add_message(
            session_id,
            "assistant",
            reply
        )
        
        return {
            "response": reply,
            "session_id": session_id
        }
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 3. API文档和验证

### 自动生成的API文档
FastAPI自动为你的API生成交互式文档：
- Swagger UI: `/docs`
- ReDoc: `/redoc`

### 请求验证
```python
from pydantic import BaseModel, Field
from typing import Optional

class ChatRequest(BaseModel):
    message: str = Field(..., min_length=1, max_length=4096)
    temperature: Optional[float] = Field(0.7, ge=0, le=1)
    model: str = Field("gpt-3.5-turbo", pattern="^gpt-")
    
    class Config:
        schema_extra = {
            "example": {
                "message": "你好，请介绍一下Python。",
                "temperature": 0.7,
                "model": "gpt-3.5-turbo"
            }
        }

@app.post("/api/chat/validated")
async def validated_chat(request: ChatRequest):
    # Pydantic自动验证请求数据
    try:
        response = openai.ChatCompletion.create(
            model=request.model,
            messages=[{"role": "user", "content": request.message}],
            temperature=request.temperature
        )
        return {"response": response.choices[0].message.content}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 4. 中间件和依赖注入

### 认证中间件
```python
from fastapi import Depends, HTTPException, Security
from fastapi.security.api_key import APIKeyHeader, APIKey

API_KEY_NAME = "X-API-Key"
API_KEY = os.getenv("API_KEY")

api_key_header = APIKeyHeader(name=API_KEY_NAME, auto_error=True)

async def get_api_key(api_key: str = Security(api_key_header)):
    if api_key == API_KEY:
        return api_key
    raise HTTPException(
        status_code=403,
        detail="无效的API密钥"
    )

@app.post("/api/protected/chat")
async def protected_chat(
    message: ChatMessage,
    api_key: APIKey = Depends(get_api_key)
):
    # 处理请求
    pass
```

### 速率限制
```python
from fastapi import Request
from fastapi.middleware.base import BaseHTTPMiddleware
from collections import defaultdict
import time

class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, calls: int, period: int):
        super().__init__(app)
        self.calls = calls
        self.period = period
        self.requests = defaultdict(list)
    
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        now = time.time()
        
        # 清理过期的请求记录
        self.requests[client_ip] = [
            req_time for req_time in self.requests[client_ip]
            if now - req_time < self.period
        ]
        
        # 检查请求频率
        if len(self.requests[client_ip]) >= self.calls:
            raise HTTPException(
                status_code=429,
                detail="请求过于频繁，请稍后再试"
            )
        
        self.requests[client_ip].append(now)
        return await call_next(request)

# 添加中间件
app.add_middleware(
    RateLimitMiddleware,
    calls=10,  # 最大请求次数
    period=60  # 时间周期（秒）
)
```

## 5. 实际应用示例

### 完整的聊天API服务
```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import List, Optional, Dict
import openai
import os
from datetime import datetime

app = FastAPI(
    title="LLM Chat API",
    description="基于FastAPI的LLM聊天服务",
    version="1.0.0"
)

# 配置CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 数据模型
class Message(BaseModel):
    role: str
    content: str
    timestamp: Optional[datetime] = None

class ChatRequest(BaseModel):
    message: str = Field(..., min_length=1)
    session_id: Optional[str] = "default"
    temperature: Optional[float] = 0.7

class ChatResponse(BaseModel):
    response: str
    session_id: str
    timestamp: datetime

# 聊天管理器
class ChatManager:
    def __init__(self):
        self.conversations: Dict[str, List[Message]] = {}
        self.max_history = 10
    
    def add_message(self, session_id: str, role: str, content: str):
        if session_id not in self.conversations:
            self.conversations[session_id] = []
        
        message = Message(
            role=role,
            content=content,
            timestamp=datetime.now()
        )
        
        self.conversations[session_id].append(message)
        
        # 保持历史记录在限制范围内
        if len(self.conversations[session_id]) > self.max_history:
            self.conversations[session_id] = (
                self.conversations[session_id][-self.max_history:]
            )
    
    def get_messages(self, session_id: str) -> List[Dict]:
        messages = self.conversations.get(session_id, [])
        return [
            {"role": msg.role, "content": msg.content}
            for msg in messages
        ]

chat_manager = ChatManager()

# API端点
@app.post("/api/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    try:
        # 添加用户消息
        chat_manager.add_message(
            request.session_id,
            "user",
            request.message
        )
        
        # 获取对话历史
        messages = chat_manager.get_messages(request.session_id)
        
        # 调用OpenAI API
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=messages,
            temperature=request.temperature
        )
        
        reply = response.choices[0].message.content
        
        # 添加助手回复
        chat_manager.add_message(
            request.session_id,
            "assistant",
            reply
        )
        
        return ChatResponse(
            response=reply,
            session_id=request.session_id,
            timestamp=datetime.now()
        )
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# 获取对话历史
@app.get("/api/chat/{session_id}/history")
async def get_chat_history(session_id: str):
    messages = chat_manager.get_messages(session_id)
    return {"history": messages}

# 清除对话历史
@app.delete("/api/chat/{session_id}/history")
async def clear_chat_history(session_id: str):
    if session_id in chat_manager.conversations:
        chat_manager.conversations[session_id] = []
    return {"message": "历史记录已清除"}
```

## 6. 练习题

1. 创建一个文档问答API：
```python
from fastapi import FastAPI, UploadFile, File, HTTPException
from pydantic import BaseModel
import os
import openai
from typing import Optional

app = FastAPI()

class Question(BaseModel):
    doc_id: str
    question: str

class DocumentQA:
    def __init__(self):
        self.documents = {}
    
    async def add_document(self, file: UploadFile) -> str:
        content = await file.read()
        doc_id = file.filename
        self.documents[doc_id] = content.decode()
        return doc_id
    
    def answer_question(self, doc_id: str, question: str) -> str:
        if doc_id not in self.documents:
            raise ValueError("文档不存在")
        
        content = self.documents[doc_id]
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": f"基于以下文档回答问题：\n\n{content}"},
                {"role": "user", "content": question}
            ]
        )
        
        return response.choices[0].message.content

qa_system = DocumentQA()

@app.post("/upload")
async def upload_document(file: UploadFile = File(...)):
    try:
        doc_id = await qa_system.add_document(file)
        return {"doc_id": doc_id}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/ask")
async def ask_question(question: Question):
    try:
        answer = qa_system.answer_question(
            question.doc_id,
            question.question
        )
        return {"answer": answer}
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 小结

本章我们学习了：
- FastAPI框架基础
- 创建LLM API服务
- API文档和验证
- 中间件和依赖注入
- 实际应用开发

FastAPI的特点：
1. 快速：性能非常高
2. 简单：代码直观，易于理解
3. 自动API文档
4. 类型提示和验证
5. 异步支持

这些知识将帮助你快速构建可靠的LLM API服务。

## 下一步

在下一章中，我们将深入学习OpenAI API的使用，包括更多高级特性和最佳实践。

## 额外资源
- [FastAPI官方文档](https://fastapi.tiangolo.com/)
- [Pydantic文档](https://pydantic-docs.helpmanual.io/)
- [Uvicorn文档](https://www.uvicorn.org/)
