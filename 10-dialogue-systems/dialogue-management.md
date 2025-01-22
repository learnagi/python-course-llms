# 对话管理

本节将介绍对话系统中的对话管理模块，包括对话流程控制、状态追踪和错误处理。

## 对话流程控制

对话流程控制负责管理对话的进展和转换：

```python
class DialogueFlow:
    def __init__(self):
        self.current_state = "initial"
        self.transitions = {
            "initial": ["greeting", "query"],
            "greeting": ["query", "farewell"],
            "query": ["response", "clarification"],
            "clarification": ["query", "response"],
            "response": ["query", "farewell"],
            "farewell": ["end"]
        }
    
    def can_transition_to(self, next_state: str) -> bool:
        return next_state in self.transitions[self.current_state]
    
    def transition_to(self, next_state: str):
        if self.can_transition_to(next_state):
            self.current_state = next_state
            return True
        return False
```

## 状态追踪

状态追踪模块记录和管理对话的上下文状态：

```python
from dataclasses import dataclass
from typing import Dict, Any, Optional

@dataclass
class DialogueState:
    user_intent: str
    confidence: float
    entities: Dict[str, Any]
    turn_count: int
    last_action: Optional[str] = None

class StateTracker:
    def __init__(self):
        self.state = DialogueState(
            user_intent="",
            confidence=0.0,
            entities={},
            turn_count=0
        )
        self.history = []
    
    def update_state(self, 
                     intent: str, 
                     confidence: float, 
                     entities: Dict[str, Any]):
        self.state.turn_count += 1
        self.state.user_intent = intent
        self.state.confidence = confidence
        self.state.entities.update(entities)
        self.history.append(self.state)
    
    def get_current_state(self) -> DialogueState:
        return self.state
```

## 错误处理

错误处理模块负责处理对话中的异常情况：

```python
class DialogueErrorHandler:
    def __init__(self):
        self.error_responses = {
            "not_understood": "抱歉，我没有理解您的意思，能否换个方式表达？",
            "low_confidence": "我不太确定您的意思，请问您是想问{suggestion}吗？",
            "missing_info": "为了更好地帮助您，我还需要知道{missing_field}",
            "system_error": "系统遇到了一些问题，请稍后再试"
        }
    
    def handle_error(self, error_type: str, **kwargs) -> str:
        if error_type in self.error_responses:
            return self.error_responses[error_type].format(**kwargs)
        return self.error_responses["system_error"]
    
    def handle_low_confidence(self, 
                            confidence: float, 
                            suggestion: str) -> str:
        if confidence < 0.5:
            return self.handle_error(
                "low_confidence", 
                suggestion=suggestion
            )
        return ""
```

## 集成示例

将以上组件整合到对话管理器中：

```python
class DialogueManager:
    def __init__(self):
        self.flow = DialogueFlow()
        self.state_tracker = StateTracker()
        self.error_handler = DialogueErrorHandler()
    
    def process_turn(self, 
                     user_input: str, 
                     intent: str,
                     confidence: float,
                     entities: Dict[str, Any]) -> str:
        # 更新状态
        self.state_tracker.update_state(intent, confidence, entities)
        
        # 错误处理
        error_response = self.error_handler.handle_low_confidence(
            confidence,
            suggestion=intent
        )
        if error_response:
            return error_response
        
        # 状态转换
        next_state = self._determine_next_state(intent)
        if self.flow.transition_to(next_state):
            return self._generate_response(next_state)
        
        return self.error_handler.handle_error("system_error")
    
    def _determine_next_state(self, intent: str) -> str:
        # 根据意图确定下一个状态
        state_mapping = {
            "greet": "greeting",
            "ask": "query",
            "clarify": "clarification",
            "bye": "farewell"
        }
        return state_mapping.get(intent, "query")
    
    def _generate_response(self, state: str) -> str:
        # 根据状态生成回复
        # 实际应用中可能需要更复杂的响应生成逻辑
        pass
```