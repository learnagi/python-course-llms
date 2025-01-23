---
title: "设计模式实践"
slug: "design-patterns"
sequence: 3
description: "学习Python中常用的设计模式，包括单例模式、工厂模式、观察者模式等"
is_published: true
estimated_minutes: 20
---

# 设计模式实践

## 1. 创建型模式

### 单例模式
```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # 初始化代码
        pass

# 使用示例
s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # True
```

### 工厂模式
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class AnimalFactory:
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        if animal_type.lower() == "dog":
            return Dog()
        elif animal_type.lower() == "cat":
            return Cat()
        raise ValueError(f"Unknown animal type: {animal_type}")

# 使用示例
factory = AnimalFactory()
dog = factory.create_animal("dog")
cat = factory.create_animal("cat")
```

## 2. 结构型模式

### 装饰器模式
```python
from functools import wraps
from typing import Callable

def retry(max_attempts: int = 3):
    def decorator(func: Callable):
        @wraps(func)
        def wrapper(*args, **kwargs):
            attempts = 0
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    attempts += 1
                    if attempts == max_attempts:
                        raise e
        return wrapper
    return decorator

# 使用示例
@retry(max_attempts=3)
def api_call():
    # 可能失败的API调用
    pass
```

### 适配器模式
```python
class OldSystem:
    def specific_request(self) -> str:
        return "Old system request"

class NewSystem:
    def unified_request(self) -> str:
        pass

class Adapter(NewSystem):
    def __init__(self, old_system: OldSystem):
        self.old_system = old_system
    
    def unified_request(self) -> str:
        return self.old_system.specific_request()

# 使用示例
old = OldSystem()
adapter = Adapter(old)
result = adapter.unified_request()
```

## 3. 行为型模式

### 观察者模式
```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, message: str):
        pass

class Subject:
    def __init__(self):
        self._observers: List[Observer] = []
        self._state = None
    
    def attach(self, observer: Observer):
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update(self._state)
    
    @property
    def state(self):
        return self._state
    
    @state.setter
    def state(self, value):
        self._state = value
        self.notify()

# 使用示例
class ConcreteObserver(Observer):
    def update(self, message: str):
        print(f"Received update: {message}")

subject = Subject()
observer = ConcreteObserver()
subject.attach(observer)
subject.state = "New State"
```

## 4. 最佳实践

1. **选择合适的模式**
   - 根据实际需求选择
   - 避免过度设计
   - 考虑维护成本

2. **遵循SOLID原则**
   - 单一职责
   - 开闭原则
   - 依赖倒置
   - 接口隔离
   - 里氏替换

3. **注意性能影响**
   - 合理使用抽象
   - 避免过度封装
   - 考虑内存开销