---
title: "测试与调试基础"
slug: "testing-basics"
sequence: 1
description: "深入理解Python测试与调试的核心概念和最佳实践"
is_published: true
estimated_minutes: 30
---

# 测试与调试基础

## 1. 单元测试

### 1.1 unittest框架

```python
import unittest
from typing import List, Optional

class Calculator:
    def add(self, x: float, y: float) -> float:
        return x + y
    
    def divide(self, x: float, y: float) -> Optional[float]:
        try:
            return x / y
        except ZeroDivisionError:
            return None

class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calc = Calculator()
    
    def test_add(self):
        self.assertEqual(self.calc.add(2, 3), 5)
        self.assertEqual(self.calc.add(-1, 1), 0)
        self.assertEqual(self.calc.add(0.1, 0.2), 0.3, places=7)
    
    def test_divide(self):
        self.assertEqual(self.calc.divide(6, 2), 3)
        self.assertIsNone(self.calc.divide(5, 0))
```

### 1.2 pytest框架

```python
import pytest

def test_calculator_add():
    calc = Calculator()
    assert calc.add(2, 3) == 5
    assert calc.add(-1, 1) == 0
    assert abs(calc.add(0.1, 0.2) - 0.3) < 1e-7

@pytest.mark.parametrize("x,y,expected", [
    (6, 2, 3),
    (5, 0, None),
    (-4, 2, -2)
])
def test_calculator_divide(x, y, expected):
    calc = Calculator()
    assert calc.divide(x, y) == expected
```

## 2. 集成测试

### 2.1 测试数据库操作

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

@pytest.fixture
def db_session():
    engine = create_engine('sqlite:///:memory:')
    Session = sessionmaker(bind=engine)
    session = Session()
    
    # 设置测试数据
    yield session
    
    # 清理
    session.close()

def test_user_creation(db_session):
    user = User(username='test_user')
    db_session.add(user)
    db_session.commit()
    
    saved_user = db_session.query(User).filter_by(username='test_user').first()
    assert saved_user is not None
    assert saved_user.username == 'test_user'
```

### 2.2 测试API接口

```python
from fastapi.testclient import TestClient
from your_app import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_create_item():
    response = client.post(
        "/items/",
        json={"name": "test item", "price": 10.5}
    )
    assert response.status_code == 201
    assert response.json()["name"] == "test item"
```

## 3. 调试技巧

### 3.1 使用pdb

```python
def complex_function(data: List[dict]) -> dict:
    result = {}
    for item in data:
        # 设置断点
        import pdb; pdb.set_trace()
        # 或使用Python 3.7+的breakpoint()
        result[item['id']] = process_item(item)
    return result
```

### 3.2 日志调试

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def process_data(data: dict) -> dict:
    logger.debug(f"Processing data: {data}")
    try:
        result = transform_data(data)
        logger.info(f"Data processed successfully: {result}")
        return result
    except Exception as e:
        logger.error(f"Error processing data: {str(e)}")
        raise
```

## 4. 性能分析

### 4.1 使用cProfile

```python
import cProfile
import pstats

def profile_function(func):
    profiler = cProfile.Profile()
    profiler.enable()
    result = func()
    profiler.disable()
    stats = pstats.Stats(profiler).sort_stats('cumulative')
    stats.print_stats()
    return result

# 使用示例
@profile_function
def main():
    # 你的代码
    pass
```

### 4.2 内存分析

```python
from memory_profiler import profile

@profile
def memory_intensive_function():
    data = []
    for i in range(1000000):
        data.append(i)
    return sum(data)
```

## 5. 测试最佳实践

1. 编写可测试的代码
   - 单一职责原则
   - 依赖注入
   - 避免全局状态

2. 测试覆盖率
   - 使用coverage.py
   - 关注关键业务逻辑
   - 平衡测试成本和收益

3. 持续集成
   - 自动化测试
   - CI/CD流程
   - 测试报告和监控

## 总结

良好的测试和调试实践是保证代码质量的关键。通过合理使用单元测试、集成测试、调试工具和性能分析，可以有效地发现和解决问题，提高代码的可靠性和可维护性。在实际开发中，需要根据项目特点选择合适的测试策略和工具。