# 第十七章：中国大模型实战

本章将介绍如何使用中国主流的大语言模型，包括DeepSeek、文心一言、智谱AI等，并通过实际案例展示它们的应用。

## 1. DeepSeek-Chat

### 基础使用
```python
import os
from openai import OpenAI

# 设置API密钥和基础URL
client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1"
)

def chat_with_deepseek(prompt: str) -> str:
    """与DeepSeek模型对话"""
    try:
        response = client.chat.completions.create(
            model="deepseek-chat",
            messages=[
                {"role": "user", "content": prompt}
            ],
            temperature=0.7,
            max_tokens=1000
        )
        return response.choices[0].message.content
    except Exception as e:
        return f"Error: {str(e)}"

# 使用示例
prompt = "请解释什么是量子计算机？"
response = chat_with_deepseek(prompt)
print(response)
```

### 代码生成
```python
def generate_code_with_deepseek(prompt: str) -> str:
    """使用DeepSeek生成代码"""
    try:
        response = client.chat.completions.create(
            model="deepseek-coder",
            messages=[
                {"role": "user", "content": prompt}
            ],
            temperature=0.1,  # 降低随机性以获得更稳定的代码输出
            max_tokens=2000
        )
        return response.choices[0].message.content
    except Exception as e:
        return f"Error: {str(e)}"

# 使用示例
code_prompt = """
请用Python实现一个简单的贪吃蛇游戏，使用pygame库。
要求：
1. 基本的移动功能
2. 随机生成食物
3. 计分功能
"""
code = generate_code_with_deepseek(code_prompt)
print(code)
```

### 流式输出
```python
async def stream_deepseek_response(prompt: str):
    """流式获取DeepSeek响应"""
    try:
        stream = await client.chat.completions.create(
            model="deepseek-chat",
            messages=[
                {"role": "user", "content": prompt}
            ],
            stream=True
        )
        
        async for chunk in stream:
            if chunk.choices[0].delta.content is not None:
                print(chunk.choices[0].delta.content, end="", flush=True)
    except Exception as e:
        print(f"Error: {str(e)}")

# 使用示例
import asyncio
asyncio.run(stream_deepseek_response("请写一首关于人工智能的诗。"))
```

## 2. 文心一言

### API调用
```python
import requests
import json

class ERNIEBot:
    """文心一言API封装"""
    def __init__(self, api_key: str, secret_key: str):
        self.api_key = api_key
        self.secret_key = secret_key
        self.access_token = self._get_access_token()
    
    def _get_access_token(self) -> str:
        """获取访问令牌"""
        url = "https://aip.baidubce.com/oauth/2.0/token"
        params = {
            "grant_type": "client_credentials",
            "client_id": self.api_key,
            "client_secret": self.secret_key
        }
        response = requests.post(url, params=params)
        return response.json().get("access_token")
    
    def chat(self, prompt: str) -> str:
        """发送对话请求"""
        url = f"https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat/completions?access_token={self.access_token}"
        
        payload = {
            "messages": [
                {"role": "user", "content": prompt}
            ]
        }
        
        headers = {
            "Content-Type": "application/json"
        }
        
        response = requests.post(url, headers=headers, json=payload)
        return response.json().get("result", "")

# 使用示例
ernie = ERNIEBot(
    api_key="YOUR_API_KEY",
    secret_key="YOUR_SECRET_KEY"
)

response = ernie.chat("请介绍一下中国的人工智能发展现状。")
print(response)
```

### 知识库问答
```python
class ERNIEKnowledgeBase:
    """文心一言知识库问答"""
    def __init__(self, ernie_bot: ERNIEBot):
        self.ernie_bot = ernie_bot
        self.knowledge_base = {}
    
    def add_document(self, doc_id: str, content: str):
        """添加文档到知识库"""
        self.knowledge_base[doc_id] = content
    
    def query(self, question: str) -> str:
        """查询知识库"""
        # 构建包含知识库内容的提示
        context = "\n".join(self.knowledge_base.values())
        prompt = f"""
        基于以下背景信息回答问题：
        
        背景信息：
        {context}
        
        问题：{question}
        """
        
        return self.ernie_bot.chat(prompt)

# 使用示例
kb = ERNIEKnowledgeBase(ernie)
kb.add_document("doc1", "人工智能是计算机科学的一个分支...")
kb.add_document("doc2", "机器学习是人工智能的核心技术之一...")

answer = kb.query("什么是人工智能？")
print(answer)
```

## 3. 智谱AI (ChatGLM)

### API调用
```python
import zhipuai

class ChatGLM:
    """智谱AI API封装"""
    def __init__(self, api_key: str):
        zhipuai.api_key = api_key
    
    def chat(self, prompt: str) -> str:
        """发送对话请求"""
        try:
            response = zhipuai.model_api.invoke(
                model="chatglm_turbo",
                prompt=[
                    {"role": "user", "content": prompt}
                ]
            )
            return response.get("data", {}).get("text", "")
        except Exception as e:
            return f"Error: {str(e)}"
    
    def generate_embedding(self, text: str) -> list:
        """生成文本嵌入"""
        try:
            response = zhipuai.model_api.invoke(
                model="text_embedding",
                prompt=text
            )
            return response.get("data", {}).get("embedding", [])
        except Exception as e:
            return []

# 使用示例
chatglm = ChatGLM("YOUR_API_KEY")
response = chatglm.chat("请解释什么是深度学习？")
print(response)

# 生成文本嵌入
embedding = chatglm.generate_embedding("人工智能")
print(f"嵌入维度：{len(embedding)}")
```

### 语义搜索应用
```python
import numpy as np
from typing import List, Tuple

class SemanticSearch:
    """基于ChatGLM的语义搜索"""
    def __init__(self, chatglm: ChatGLM):
        self.chatglm = chatglm
        self.documents = []
        self.embeddings = []
    
    def add_document(self, doc: str):
        """添加文档"""
        self.documents.append(doc)
        embedding = self.chatglm.generate_embedding(doc)
        self.embeddings.append(embedding)
    
    def search(self, query: str, top_k: int = 3) -> List[Tuple[str, float]]:
        """搜索相似文档"""
        # 生成查询的嵌入向量
        query_embedding = self.chatglm.generate_embedding(query)
        
        # 计算相似度
        similarities = []
        for doc_embedding in self.embeddings:
            similarity = self._cosine_similarity(
                query_embedding,
                doc_embedding
            )
            similarities.append(similarity)
        
        # 获取最相似的文档
        top_indices = np.argsort(similarities)[-top_k:][::-1]
        return [
            (self.documents[i], similarities[i])
            for i in top_indices
        ]
    
    def _cosine_similarity(self, v1: List[float], v2: List[float]) -> float:
        """计算余弦相似度"""
        v1 = np.array(v1)
        v2 = np.array(v2)
        return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))

# 使用示例
search_engine = SemanticSearch(chatglm)

# 添加文档
documents = [
    "人工智能是计算机科学的一个分支，致力于开发能模拟人类智能的系统。",
    "机器学习是人工智能的一个子领域，专注于让计算机从数据中学习。",
    "深度学习是机器学习的一种方法，使用多层神经网络进行学习。"
]

for doc in documents:
    search_engine.add_document(doc)

# 搜索相似文档
results = search_engine.search("什么是机器学习？")
for doc, score in results:
    print(f"相似度: {score:.4f}")
    print(f"文档: {doc}\n")
```

## 4. 实战案例：智能客服系统

### 系统架构
```python
from typing import List, Dict, Optional
import json
import datetime

class Message:
    """消息类"""
    def __init__(self, role: str, content: str):
        self.role = role
        self.content = content
        self.timestamp = datetime.datetime.now()
    
    def to_dict(self) -> dict:
        return {
            "role": self.role,
            "content": self.content,
            "timestamp": self.timestamp.isoformat()
        }

class Conversation:
    """对话类"""
    def __init__(self):
        self.messages: List[Message] = []
    
    def add_message(self, role: str, content: str):
        self.messages.append(Message(role, content))
    
    def get_context(self, max_messages: int = 5) -> str:
        """获取最近的对话上下文"""
        recent_messages = self.messages[-max_messages:]
        return "\n".join([
            f"{msg.role}: {msg.content}"
            for msg in recent_messages
        ])

class CustomerService:
    """智能客服系统"""
    def __init__(self):
        # 初始化不同的模型实例
        self.deepseek = OpenAI(
            api_key=os.getenv("DEEPSEEK_API_KEY"),
            base_url="https://api.deepseek.com/v1"
        )
        self.ernie = ERNIEBot(
            api_key=os.getenv("ERNIE_API_KEY"),
            secret_key=os.getenv("ERNIE_SECRET_KEY")
        )
        self.chatglm = ChatGLM(os.getenv("CHATGLM_API_KEY"))
        
        # 存储用户会话
        self.conversations: Dict[str, Conversation] = {}
        
        # 加载知识库
        self.knowledge_base = self._load_knowledge_base()
    
    def _load_knowledge_base(self) -> dict:
        """加载知识库"""
        try:
            with open("knowledge_base.json", "r", encoding="utf-8") as f:
                return json.load(f)
        except FileNotFoundError:
            return {}
    
    def get_or_create_conversation(
        self,
        user_id: str
    ) -> Conversation:
        """获取或创建会话"""
        if user_id not in self.conversations:
            self.conversations[user_id] = Conversation()
        return self.conversations[user_id]
    
    async def handle_message(
        self,
        user_id: str,
        message: str,
        model: str = "deepseek"
    ) -> str:
        """处理用户消息"""
        conversation = self.get_or_create_conversation(user_id)
        conversation.add_message("user", message)
        
        # 构建提示
        context = conversation.get_context()
        knowledge = self._find_relevant_knowledge(message)
        
        prompt = f"""
        作为客服助手，请基于以下信息回答用户的问题：
        
        对话历史：
        {context}
        
        相关知识：
        {knowledge}
        
        请用专业、友好的语气回答。
        """
        
        # 根据选择的模型生成回复
        response = await self._generate_response(prompt, model)
        conversation.add_message("assistant", response)
        
        return response
    
    def _find_relevant_knowledge(self, query: str) -> str:
        """查找相关知识"""
        # 实现简单的关键词匹配
        relevant_info = []
        for key, value in self.knowledge_base.items():
            if any(word in query for word in key.split()):
                relevant_info.append(value)
        return "\n".join(relevant_info)
    
    async def _generate_response(
        self,
        prompt: str,
        model: str
    ) -> str:
        """生成回复"""
        try:
            if model == "deepseek":
                response = await self._call_deepseek(prompt)
            elif model == "ernie":
                response = self.ernie.chat(prompt)
            elif model == "chatglm":
                response = self.chatglm.chat(prompt)
            else:
                raise ValueError(f"不支持的模型: {model}")
            
            return response
        except Exception as e:
            return f"抱歉，系统出现错误: {str(e)}"
    
    async def _call_deepseek(self, prompt: str) -> str:
        """调用DeepSeek API"""
        response = await self.deepseek.chat.completions.create(
            model="deepseek-chat",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message.content

# 使用示例
async def main():
    # 创建客服系统实例
    customer_service = CustomerService()
    
    # 模拟用户对话
    user_id = "user123"
    
    # 测试不同模型
    models = ["deepseek", "ernie", "chatglm"]
    question = "你们的退货政策是怎样的？"
    
    for model in models:
        print(f"\n使用 {model} 模型:")
        response = await customer_service.handle_message(
            user_id,
            question,
            model
        )
        print(f"用户: {question}")
        print(f"客服: {response}")

# 运行示例
import asyncio
asyncio.run(main())
```

## 5. 练习题

1. 实现多模型对比评估：
```python
class ModelEvaluator:
    """模型评估器"""
    def evaluate_models(self, test_cases: List[dict]):
        """评估不同模型的表现"""
        pass
    
    def generate_report(self):
        """生成评估报告"""
        pass

# 完成上述类的实现
```

2. 实现文本分类应用：
```python
class TextClassifier:
    """使用中国大模型实现文本分类"""
    def train(self, texts: List[str], labels: List[str]):
        """训练分类器"""
        pass
    
    def predict(self, text: str) -> str:
        """预测文本类别"""
        pass

# 实现这些类
```

## 小结

本章我们学习了：
- DeepSeek的基本使用和代码生成
- 文心一言的API调用和知识库问答
- 智谱AI的文本处理和语义搜索
- 实际的智能客服系统实现

这些内容将帮助你更好地使用中国的大语言模型，开发实际应用。

## 下一步

继续探索更多的应用场景，如：
- 多模态处理
- 垂直领域优化
- 模型微调
- 部署优化

## 额外资源
- [DeepSeek官方文档](https://platform.deepseek.com/)
- [文心一言开放平台](https://cloud.baidu.com/doc/WENXINWORKSHOP/index.html)
- [智谱AI开发者中心](https://open.bigmodel.cn/)
