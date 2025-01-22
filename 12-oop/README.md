---
title: "面向对象编程"
slug: "oop"
sequence: 12
description: "学习Python的面向对象编程概念，包括类和对象、继承和多态、封装和抽象等核心概念"
status: "published"
---

# 面向对象编程

本章将介绍Python的面向对象编程概念，包括类和对象、继承和多态、封装和抽象等核心概念。

## 本章内容

1. [类和对象基础](./class-basics.md)
   - 类的定义和实例化
   - 属性和方法
   - 访问控制

2. [继承和多态](./inheritance.md)
   - 类的继承
   - 方法重写
   - 抽象类
   - 多重继承

3. [特殊方法](./special-methods.md)
   - 魔术方法
   - 运算符重载
   - 上下文管理器
   - 描述符
```
```python
class ChatBot:
    """聊天机器人类"""
    def __init__(self, name: str):
        self.name = name
        self.history = []
    
    def say_hello(self) -> str:
        return f"你好，我是{self.name}！"
    
    def remember(self, message: str):
        self.history.append(message)
