# 第十章：构建智能对话系统

本章将介绍如何构建一个完整的智能对话系统，包括上下文管理、多轮对话、状态追踪等功能。

## 1. 对话系统架构

### 基本组件
```python
from dataclasses import dataclass
from typing import List, Dict, Optional
import datetime

@dataclass
class Message:
    role: str
    content: str
    timestamp: datetime.datetime = datetime.datetime.now()

@dataclass
class Conversation:
    id: str
    messages: List[Message]
    metadata: Dict = None
```

### 对话管理器
```python
class DialogueManager:
    def __init__(self):
        self.conversations: Dict[str, Conversation] = {}
        self.max_history = 10
        self.max_tokens = 4000
    
    def create_conversation(self, conversation_id: str) -> Conversation:
        """创建新对话"""
        conversation = Conversation(
            id=conversation_id,
            messages=[],
            metadata={"created_at": datetime.datetime.now()}
        )
        self.conversations[conversation_id] = conversation
        return conversation
    
    def add_message(self, conversation_id: str, role: str, content: str) -> None:
        """添加消息到对话"""
        if conversation_id not in self.conversations:
            self.create_conversation(conversation_id)
        
        message = Message(role=role, content=content)
        self.conversations[conversation_id].messages.append(message)
        
        # 维护对话历史长度
        if len(self.conversations[conversation_id].messages) > self.max_history:
            self.conversations[conversation_id].messages.pop(0)
    
    def get_conversation_history(self, conversation_id: str) -> List[Dict]:
        """获取对话历史"""
        if conversation_id not in self.conversations:
            return []
        
        return [
            {"role": msg.role, "content": msg.content}
            for msg in self.conversations[conversation_id].messages
        ]
```

## 2. 上下文管理

### 上下文窗口
```python
class ContextWindow:
    def __init__(self, max_tokens: int = 4000):
        self.max_tokens = max_tokens
    
    def truncate_messages(self, messages: List[Dict]) -> List[Dict]:
        """截断消息以适应上下文窗口"""
        total_tokens = 0
        truncated_messages = []
        
        for message in reversed(messages):
            # 粗略估计token数量
            estimated_tokens = len(message["content"]) // 4
            
            if total_tokens + estimated_tokens > self.max_tokens:
                break
            
            total_tokens += estimated_tokens
            truncated_messages.insert(0, message)
        
        return truncated_messages
```

### 记忆管理
```python
class MemoryManager:
    def __init__(self):
        self.short_term_memory: List[Dict] = []
        self.long_term_memory: Dict[str, List[Dict]] = {}
    
    def add_to_short_term(self, message: Dict):
        """添加到短期记忆"""
        self.short_term_memory.append(message)
        if len(self.short_term_memory) > 5:  # 保持最近5条记录
            self.short_term_memory.pop(0)
    
    def add_to_long_term(self, key: str, message: Dict):
        """添加到长期记忆"""
        if key not in self.long_term_memory:
            self.long_term_memory[key] = []
        self.long_term_memory[key].append(message)
    
    def get_relevant_context(self, query: str) -> List[Dict]:
        """获取相关上下文"""
        # 这里可以实现更复杂的相关性搜索
        return self.short_term_memory
```

## 3. 状态追踪

### 对话状态管理器
```python
from enum import Enum

class DialogueState(Enum):
    INIT = "init"
    WAITING_USER_INPUT = "waiting_user_input"
    PROCESSING = "processing"
    ERROR = "error"
    COMPLETED = "completed"

class StateManager:
    def __init__(self):
        self.current_state = DialogueState.INIT
        self.state_history = []
        self.variables = {}
    
    def transition_to(self, new_state: DialogueState):
        """状态转换"""
        self.state_history.append((self.current_state, datetime.datetime.now()))
        self.current_state = new_state
    
    def set_variable(self, key: str, value: any):
        """设置对话变量"""
        self.variables[key] = value
    
    def get_variable(self, key: str) -> any:
        """获取对话变量"""
        return self.variables.get(key)
```

## 4. 对话策略

### 策略管理器
```python
class DialogueStrategy:
    def __init__(self):
        self.fallback_responses = [
            "抱歉，我没有理解您的意思，能请您重新表述一下吗？",
            "这个问题有点复杂，我们能换个方式讨论吗？",
            "让我们换个角度来看这个问题..."
        ]
        self.greeting_patterns = [
            "你好", "您好", "早上好", "下午好", "晚上好"
        ]
    
    def get_response_strategy(self, message: str) -> str:
        """确定响应策略"""
        if any(pattern in message for pattern in self.greeting_patterns):
            return "greeting"
        # 可以添加更多策略判断
        return "normal"
    
    def get_fallback_response(self) -> str:
        """获取备选响应"""
        import random
        return random.choice(self.fallback_responses)
```

## 5. 完整对话系统

### 系统集成
```python
class DialogueSystem:
    def __init__(self):
        self.dialogue_manager = DialogueManager()
        self.context_window = ContextWindow()
        self.memory_manager = MemoryManager()
        self.state_manager = StateManager()
        self.strategy = DialogueStrategy()
    
    async def process_message(self, user_id: str, message: str) -> str:
        """处理用户消息"""
        try:
            # 更新状态
            self.state_manager.transition_to(DialogueState.PROCESSING)
            
            # 添加用户消息
            self.dialogue_manager.add_message(user_id, "user", message)
            
            # 获取对话历史
            history = self.dialogue_manager.get_conversation_history(user_id)
            
            # 应用上下文窗口
            truncated_history = self.context_window.truncate_messages(history)
            
            # 获取响应策略
            strategy = self.strategy.get_response_strategy(message)
            
            # 生成回复
            response = await self._generate_response(truncated_history, strategy)
            
            # 添加助手回复
            self.dialogue_manager.add_message(user_id, "assistant", response)
            
            # 更新记忆
            self.memory_manager.add_to_short_term({
                "user_message": message,
                "response": response
            })
            
            # 更新状态
            self.state_manager.transition_to(DialogueState.WAITING_USER_INPUT)
            
            return response
            
        except Exception as e:
            self.state_manager.transition_to(DialogueState.ERROR)
            return f"处理消息时出错: {str(e)}"
    
    async def _generate_response(self, history: List[Dict], strategy: str) -> str:
        """生成回复"""
        try:
            if strategy == "greeting":
                return "您好！我是AI助手，很高兴为您服务。"
            
            response = await openai.ChatCompletion.acreate(
                model="gpt-3.5-turbo",
                messages=history
            )
            
            return response.choices[0].message.content
            
        except Exception:
            return self.strategy.get_fallback_response()
```

## 6. 实际应用示例

### 客服机器人
```python
class CustomerServiceBot:
    def __init__(self):
        self.dialogue_system = DialogueSystem()
        self.product_info = {
            "商品查询": "您可以告诉我具体的商品名称或编号",
            "订单状态": "请提供您的订单号",
            "退换货": "请提供订单号和退换货原因"
        }
    
    async def handle_customer_query(self, user_id: str, query: str) -> str:
        """处理客户查询"""
        # 添加业务相关的上下文
        self.dialogue_system.state_manager.set_variable("service_type", "customer_service")
        
        # 处理查询
        response = await self.dialogue_system.process_message(user_id, query)
        
        # 添加产品信息（如果相关）
        for key in self.product_info:
            if key in query:
                response += f"\n\n{self.product_info[key]}"
        
        return response

# 使用示例
async def main():
    bot = CustomerServiceBot()
    response = await bot.handle_customer_query("user123", "我想查询一下订单状态")
    print(response)
```

### 教育助手
```python
class EducationAssistant:
    def __init__(self):
        self.dialogue_system = DialogueSystem()
        self.subjects = {
            "数学": ["代数", "几何", "微积分"],
            "物理": ["力学", "电磁学", "热力学"],
            "化学": ["有机化学", "无机化学", "物理化学"]
        }
    
    async def handle_student_query(self, student_id: str, query: str) -> str:
        """处理学生提问"""
        # 设置教育相关上下文
        self.dialogue_system.state_manager.set_variable("context", "education")
        
        # 添加学科信息
        subject_context = ""
        for subject, topics in self.subjects.items():
            if subject in query or any(topic in query for topic in topics):
                subject_context = f"在{subject}领域中，"
                break
        
        # 处理查询
        response = await self.dialogue_system.process_message(student_id, subject_context + query)
        
        return response

# 使用示例
async def main():
    assistant = EducationAssistant()
    response = await assistant.handle_student_query("student123", "能帮我解释一下微积分的基本概念吗？")
    print(response)
```

## 7. 测试与评估

### 对话质量评估
```python
class DialogueEvaluator:
    def __init__(self):
        self.metrics = {
            "response_time": [],
            "context_relevance": [],
            "user_satisfaction": []
        }
    
    def evaluate_response(self, context: str, response: str, response_time: float) -> Dict:
        """评估回复质量"""
        evaluation = {
            "response_time": response_time,
            "context_relevance": self._evaluate_relevance(context, response),
            "user_satisfaction": None  # 需要用户反馈
        }
        
        for key, value in evaluation.items():
            if value is not None:
                self.metrics[key].append(value)
        
        return evaluation
    
    def _evaluate_relevance(self, context: str, response: str) -> float:
        """评估上下文相关性"""
        # 这里可以实现更复杂的相关性评估算法
        common_words = set(context.split()) & set(response.split())
        total_words = set(context.split()) | set(response.split())
        return len(common_words) / len(total_words) if total_words else 0
    
    def get_metrics_summary(self) -> Dict:
        """获取评估指标总结"""
        return {
            key: {
                "mean": sum(values) / len(values) if values else 0,
                "count": len(values)
            }
            for key, values in self.metrics.items()
        }
```

## 8. 练习题

1. 实现一个简单的对话系统：
```python
async def exercise_1():
    """
    实现一个基本的对话系统，要求：
    1. 能够维护对话历史
    2. 能够处理基本的问候语
    3. 能够提供合适的fallback响应
    """
    dialogue_system = DialogueSystem()
    
    # 测试基本对话流程
    responses = []
    responses.append(await dialogue_system.process_message("user1", "你好"))
    responses.append(await dialogue_system.process_message("user1", "今天天气怎么样？"))
    responses.append(await dialogue_system.process_message("user1", "谢谢"))
    
    return responses
```

2. 实现特定领域的对话机器人：
```python
async def exercise_2():
    """
    实现一个特定领域的对话机器人，例如：
    1. 技术支持机器人
    2. 课程辅导机器人
    3. 心理咨询机器人
    """
    # 实现你的代码
    pass
```

## 小结

本章我们学习了：
- 对话系统的基本架构
- 上下文和记忆管理
- 状态追踪和对话策略
- 完整对话系统的实现
- 实际应用案例
- 测试与评估方法

这些知识将帮助你构建功能完善的对话系统。

## 下一步

在下一章中，我们将学习提示词工程与优化，探讨如何设计更好的提示词来提高模型输出质量。

## 额外资源
- [对话系统设计模式](https://rasa.com/docs/rasa/)
- [对话管理最佳实践](https://botpress.com/docs/building-chatbots/dialog-management/)
- [上下文管理策略](https://github.com/microsoft/botbuilder-python)
