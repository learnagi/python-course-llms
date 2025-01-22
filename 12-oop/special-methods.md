# 特殊方法

本节将介绍Python中的特殊方法（魔术方法），这些方法可以让我们自定义类的行为。

## 魔术方法

魔术方法是Python中以双下划线开始和结束的特殊方法，它们在特定情况下被自动调用。

```python
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius
    
    def __str__(self) -> str:
        return f"{self.celsius}°C"
    
    def __repr__(self) -> str:
        return f"Temperature({self.celsius})"
    
    def __eq__(self, other) -> bool:
        if not isinstance(other, Temperature):
            return NotImplemented
        return self.celsius == other.celsius
```

## 运算符重载

我们可以通过实现特定的魔术方法来自定义运算符的行为：

```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)
    
    def __mul__(self, scalar: float):
        return Vector(self.x * scalar, self.y * scalar)
    
    def __str__(self) -> str:
        return f"Vector({self.x}, {self.y})"
```

## 上下文管理器

通过实现`__enter__`和`__exit__`方法，我们可以创建支持`with`语句的类：

```python
class Timer:
    def __enter__(self):
        self.start = time.time()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end = time.time()
        self.duration = self.end - self.start
        print(f"执行时间：{self.duration:.2f}秒")
```

## 描述符

描述符允许我们自定义属性的访问行为：

```python
class Positive:
    def __init__(self):
        self._name = None
    
    def __set_name__(self, owner, name):
        self._name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]
    
    def __set__(self, instance, value):
        if value <= 0:
            raise ValueError(f"{self._name}必须为正数")
        instance.__dict__[self._name] = value

class Product:
    price = Positive()
    quantity = Positive()
    
    def __init__(self, name: str, price: float, quantity: int):
        self.name = name
        self.price = price
        self.quantity = quantity
```