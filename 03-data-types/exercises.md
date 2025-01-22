---
title: "实战练习"
slug: "exercises"
sequence: 5
description: "通过实践项目巩固Python数据类型的知识，包括联系人管理系统和对话历史管理器的实现"
status: "published"
---

# 实战练习

本节将通过两个实践项目来巩固前面学习的Python数据类型知识。这些项目将综合运用列表、字典、元组和集合等数据类型。

## 项目一：联系人管理系统

创建一个简单的联系人管理系统，支持添加、查看、搜索和删除联系人信息。

```python
# contact_manager.py

class ContactManager:
    def __init__(self):
        self.contacts = {}
    
    def add_contact(self):
        """添加新联系人"""
        name = input("请输入联系人姓名：").strip()
        if not name:
            print("姓名不能为空！")
            return
        
        # 收集联系人信息
        contact = {
            "phone": input("请输入电话号码：").strip(),
            "email": input("请输入电子邮箱：").strip(),
            "address": input("请输入地址：").strip()
        }
        
        # 保存联系人
        self.contacts[name] = contact
        print(f"\n成功添加联系人：{name}")
    
    def view_contacts(self):
        """查看所有联系人"""
        if not self.contacts:
            print("\n通讯录为空！")
            return
        
        print("\n=== 联系人列表 ===")
        for name, info in self.contacts.items():
            print(f"\n姓名：{name}")
            print(f"电话：{info['phone']}")
            print(f"邮箱：{info['email']}")
            print(f"地址：{info['address']}")
            print("=" * 20)
    
    def search_contact(self):
        """搜索联系人"""
        keyword = input("请输入搜索关键词：").strip().lower()
        
        # 存储搜索结果
        results = []
        for name, info in self.contacts.items():
            # 在所有字段中搜索关键词
            if (keyword in name.lower() or
                keyword in info['phone'] or
                keyword in info['email'].lower() or
                keyword in info['address'].lower()):
                results.append((name, info))
        
        # 显示搜索结果
        if results:
            print(f"\n找到 {len(results)} 个匹配结果：")
            for name, info in results:
                print(f"\n姓名：{name}")
                print(f"电话：{info['phone']}")
                print(f"邮箱：{info['email']}")
                print(f"地址：{info['address']}")
                print("=" * 20)
        else:
            print("\n未找到匹配的联系人。")
    
    def delete_contact(self):
        """删除联系人"""
        name = input("请输入要删除的联系人姓名：").strip()
        
        if name in self.contacts:
            del self.contacts[name]
            print(f"\n已删除联系人：{name}")
        else:
            print("\n未找到该联系人！")
    
    def run(self):
        """运行联系人管理系统"""
        actions = {
            "1": ("添加联系人", self.add_contact),
            "2": ("查看所有联系人", self.view_contacts),
            "3": ("搜索联系人", self.search_contact),
            "4": ("删除联系人", self.delete_contact),
            "5": ("退出", None)
        }
        
        while True:
            print("\n=== 联系人管理系统 ===")
            for key, (name, _) in actions.items():
                print(f"{key}. {name}")
            
            choice = input("\n请选择操作（1-5）：").strip()
            
            if choice in actions:
                if choice == "5":
                    print("\n感谢使用！再见！")
                    break
                actions[choice][1]()
            else:
                print("\n无效的选择！请重试。")

if __name__ == "__main__":
    manager = ContactManager()
    manager.run()
```

## 项目二：对话历史管理器

创建一个对话历史管理器，用于管理与LLM的对话记录。

```python
# chat_history_manager.py

import time
from typing import List, Dict, Optional

class Message:
    def __init__(self, role: str, content: str):
        self.role = role
        self.content = content
        self.timestamp = time.time()
    
    def to_dict(self) -> dict:
        """转换为字典格式"""
        return {
            "role": self.role,
            "content": self.content,
            "timestamp": self.timestamp
        }
    
    @classmethod
    def from_dict(cls, data: dict) -> 'Message':
        """从字典创建消息对象"""
        msg = cls(data["role"], data["content"])
        msg.timestamp = data.get("timestamp", time.time())
        return msg

class Conversation:
    def __init__(self, title: str):
        self.title = title
        self.messages: List[Message] = []
        self.created_at = time.time()
        self.updated_at = time.time()
    
    def add_message(self, role: str, content: str):
        """添加新消息"""
        message = Message(role, content)
        self.messages.append(message)
        self.updated_at = time.time()
    
    def get_messages(self, limit: Optional[int] = None) -> List[Dict]:
        """获取消息历史"""
        messages = self.messages[-limit:] if limit else self.messages
        return [msg.to_dict() for msg in messages]
    
    def clear_history(self):
        """清空对话历史"""
        self.messages.clear()
        self.updated_at = time.time()

class ChatHistoryManager:
    def __init__(self):
        self.conversations: Dict[str, Conversation] = {}
    
    def create_conversation(self, title: str) -> str:
        """创建新对话"""
        # 使用时间戳作为对话ID
        conversation_id = str(int(time.time()))
        self.conversations[conversation_id] = Conversation(title)
        return conversation_id
    
    def add_message(self, conversation_id: str, role: str, content: str):
        """添加消息到指定对话"""
        if conversation_id not in self.conversations:
            raise ValueError("对话不存在")
        
        self.conversations[conversation_id].add_message(role, content)
    
    def get_conversation(self, conversation_id: str) -> Optional[Conversation]:
        """获取指定对话"""
        return self.conversations.get(conversation_id)
    
    def list_conversations(self) -> List[Dict]:
        """列出所有对话"""
        return [
            {
                "id": conv_id,
                "title": conv.title,
                "message_count": len(conv.messages),
                "created_at": conv.created_at,
                "updated_at": conv.updated_at
            }
            for conv_id, conv in self.conversations.items()
        ]
    
    def delete_conversation(self, conversation_id: str):
        """删除指定对话"""
        if conversation_id in self.conversations:
            del self.conversations[conversation_id]

def main():
    """主函数：演示对话历史管理器的使用"""
    manager = ChatHistoryManager()
    
    # 创建示例对话
    conv_id = manager.create_conversation("Python学习讨论")
    
    # 添加一些消息
    manager.add_message(conv_id, "user", "请介绍Python的数据类型。")
    manager.add_message(conv_id, "assistant", 
                       "Python有几种基本的数据类型：数字（包括整数、浮点数）、"
                       "字符串、列表、元组、字典和集合。每种类型都有其特定的用途。")
    manager.add_message(conv_id, "user", "能详细说说列表吗？")
    
    # 获取并显示对话内容
    conversation = manager.get_conversation(conv_id)
    if conversation:
        print(f"\n=== {conversation.title} ===")
        for msg in conversation.get_messages():
            print(f"\n{msg['role'].title()}:")
            print(msg['content'])
            print(f"时间: {time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(msg['timestamp']))}")
    
    # 显示所有对话列表
    print("\n=== 所有对话 ===")
    for conv in manager.list_conversations():
        print(f"\nID: {conv['id']}")
        print(f"标题: {conv['title']}")
        print(f"消息数: {conv['message_count']}")
        print(f"创建时间: {time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(conv['created_at']))}")

if __name__ == "__main__":
    main()
```

## 运行和测试

1. 联系人管理系统：
```bash
python contact_manager.py
```

2. 对话历史管理器：
```bash
python chat_history_manager.py
```

## 扩展练习

1. 联系人管理系统扩展：
   - 添加数据持久化（保存到文件）
   - 添加联系人分组功能
   - 添加导入/导出功能
   - 添加生日提醒功能

2. 对话历史管理器扩展：
   - 添加对话标签功能
   - 实现对话导出功能
   - 添加对话搜索功能
   - 实现对话统计分析

## 最佳实践

1. 代码组织：
   - 使用类封装相关功能
   - 分离业务逻辑和用户界面
   - 添加适当的注释和文档字符串

2. 错误处理：
   - 验证用户输入
   - 使用异常处理捕获错误
   - 提供清晰的错误信息

3. 用户体验：
   - 提供清晰的菜单和提示
   - 确认重要操作
   - 保持界面简洁直观

## 下一步

完成这些练习后，您应该已经掌握了Python各种数据类型的使用方法。接下来，我们将进入下一章，学习Python中的函数和模块。
