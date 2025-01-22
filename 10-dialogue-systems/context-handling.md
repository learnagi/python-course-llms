# 上下文处理

本节将介绍对话系统中的上下文处理机制，包括对话历史管理、上下文窗口和记忆机制。

## 对话历史管理

对话历史管理是维持对话连贯性的关键：

```python
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class Message:
    role: str
    content: str
    timestamp: float

class DialogueHistory:
    def __init__(self, max_history: int = 100):
        self.messages: List[Message] = []
        self.max_history = max_history
    
    def add_message(self, role: str, content: str):
        message = Message(
            role=role,
            content=content,
            timestamp=time.time()
        )
        self.messages.append(message)
        self._trim_history()
    
    def get_recent_messages(self, n: int) -> List[Message]:
        return self.messages[-n:]
    
    def _trim_history(self):
        if len(self.messages) > self.max_history:
            self.messages = self.messages[-self.max_history:]
```

## 上下文窗口

上下文窗口机制用于控制对话上下文的大小：

```python
class ContextWindow:
    def __init__(self, max_tokens: int = 2000):
        self.max_tokens = max_tokens
        self.current_tokens = 0
        self.messages: List[Message] = []
    
    def add_message(self, message: Message) -> bool:
        tokens = self._count_tokens(message.content)
        if self.current_tokens + tokens > self.max_tokens:
            self._trim_context(tokens)
        
        self.messages.append(message)
        self.current_tokens += tokens
        return True
    
    def _count_tokens(self, text: str) -> int:
        # 简单估算token数量
        return len(text.split())
    
    def _trim_context(self, needed_tokens: int):
        while (self.messages and 
               self.current_tokens + needed_tokens > self.max_tokens):
            removed = self.messages.pop(0)
            self.current_tokens -= self._count_tokens(removed.content)
```

## 记忆机制

记忆机制用于存储和检索重要的对话信息：

```python
class DialogueMemory:
    def __init__(self):
        self.short_term: Dict[str, Any] = {}
        self.long_term: List[Dict[str, Any]] = []
    
    def remember_fact(self, key: str, value: Any):
        self.short_term[key] = value
        
        # 重要信息存入长期记忆
        if self._is_important(key, value):
            self.long_term.append({
                'key': key,
                'value': value,
                'timestamp': time.time()
            })
    
    def recall_fact(self, key: str) -> Any:
        # 优先从短期记忆中查找
        if key in self.short_term:
            return self.short_term[key]
        
        # 在长期记忆中查找最新的记录
        for memory in reversed(self.long_term):
            if memory['key'] == key:
                return memory['value']
        return None
    
    def _is_important(self, key: str, value: Any) -> bool:
        # 判断信息是否重要的逻辑
        important_keys = {'name', 'preference', 'goal'}
        return key in important_keys
```

## 集成应用

将上述组件整合到对话系统中：

```python
class ContextManager:
    def __init__(self):
        self.history = DialogueHistory()
        self.window = ContextWindow()
        self.memory = DialogueMemory()
    
    def process_turn(self, user_input: str) -> Dict[str, Any]:
        # 记录用户输入
        self.history.add_message("user", user_input)
        
        # 提取重要信息
        entities = self._extract_entities(user_input)
        for key, value in entities.items():
            self.memory.remember_fact(key, value)
        
        # 构建上下文窗口
        context = {
            'recent_history': self.history.get_recent_messages(5),
            'relevant_facts': self._get_relevant_facts(user_input)
        }
        
        return context
    
    def _extract_entities(self, text: str) -> Dict[str, Any]:
        # 实现实体提取逻辑
        pass
    
    def _get_relevant_facts(self, query: str) -> Dict[str, Any]:
        # 实现相关信息检索逻辑
        pass
```