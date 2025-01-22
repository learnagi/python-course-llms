# 第十五章：测试与调试

本章将介绍Python的测试与调试技术，包括单元测试、集成测试、调试技巧和性能分析。

## 1. 单元测试

### 使用unittest
```python
import unittest
from typing import List

class Calculator:
    """简单计算器类"""
    @staticmethod
    def add(x: float, y: float) -> float:
        return x + y
    
    @staticmethod
    def divide(x: float, y: float) -> float:
        if y == 0:
            raise ValueError("除数不能为零")
        return x / y

class TestCalculator(unittest.TestCase):
    """计算器测试类"""
    def setUp(self):
        """测试前准备"""
        self.calc = Calculator()
    
    def test_add(self):
        """测试加法"""
        self.assertEqual(self.calc.add(1, 2), 3)
        self.assertEqual(self.calc.add(-1, 1), 0)
        self.assertEqual(self.calc.add(0, 0), 0)
    
    def test_divide(self):
        """测试除法"""
        self.assertEqual(self.calc.divide(6, 2), 3)
        self.assertEqual(self.calc.divide(5, 2), 2.5)
        
        # 测试除以零的情况
        with self.assertRaises(ValueError):
            self.calc.divide(1, 0)

if __name__ == '__main__':
    unittest.main()
```

### 使用pytest
```python
import pytest
from typing import List

def process_list(items: List[int]) -> List[int]:
    """处理列表"""
    return [x * 2 for x in items if x > 0]

@pytest.fixture
def sample_list():
    """测试数据fixture"""
    return [1, -2, 3, 0, 4]

def test_process_list(sample_list):
    """测试列表处理"""
    result = process_list(sample_list)
    assert len(result) == 3
    assert result == [2, 6, 8]

def test_empty_list():
    """测试空列表"""
    assert process_list([]) == []

def test_negative_numbers():
    """测试负数"""
    assert process_list([-1, -2, -3]) == []

@pytest.mark.parametrize("input_list,expected", [
    ([1, 2, 3], [2, 4, 6]),
    ([0, 1, 2], [2, 4]),
    ([-1, 0, 1], [2]),
])
def test_process_list_parametrize(input_list, expected):
    """参数化测试"""
    assert process_list(input_list) == expected
```

## 2. 集成测试

### API测试
```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
async def create_item(item: Item):
    return {"name": item.name, "price": item.price * 1.1}

# 测试代码
def test_create_item():
    """测试创建商品API"""
    client = TestClient(app)
    
    # 测试正常情况
    response = client.post(
        "/items/",
        json={"name": "test item", "price": 100.0}
    )
    assert response.status_code == 200
    assert response.json()["name"] == "test item"
    assert response.json()["price"] == 110.0
    
    # 测试错误情况
    response = client.post(
        "/items/",
        json={"name": "test item"}  # 缺少price字段
    )
    assert response.status_code == 422
```

### 数据库测试
```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String

# 数据库模型
Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    email = Column(String)

# 测试配置
@pytest.fixture
def db_session():
    """创建测试数据库会话"""
    engine = create_engine('sqlite:///:memory:')
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    
    yield session
    
    session.close()
    Base.metadata.drop_all(engine)

def test_create_user(db_session):
    """测试创建用户"""
    user = User(name="Test User", email="test@example.com")
    db_session.add(user)
    db_session.commit()
    
    # 验证用户已创建
    saved_user = db_session.query(User).first()
    assert saved_user.name == "Test User"
    assert saved_user.email == "test@example.com"
```

## 3. 调试技巧

### 使用pdb
```python
def complex_calculation(x: int, y: int) -> int:
    """复杂计算示例"""
    import pdb; pdb.set_trace()  # 设置断点
    
    result = 0
    for i in range(x):
        for j in range(y):
            result += i * j
    
    return result

# 使用示例
def main():
    result = complex_calculation(3, 4)
    print(f"结果: {result}")

if __name__ == "__main__":
    main()
```

### 使用logging
```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

logger = logging.getLogger(__name__)

def process_data(data: dict) -> dict:
    """处理数据"""
    logger.debug(f"开始处理数据: {data}")
    
    try:
        result = {
            "name": data["name"].upper(),
            "value": data["value"] * 2
        }
        logger.info(f"数据处理成功: {result}")
        return result
    
    except KeyError as e:
        logger.error(f"数据处理失败，缺少键: {e}")
        raise
    except Exception as e:
        logger.exception(f"数据处理时发生未知错误: {e}")
        raise

# 使用示例
try:
    result = process_data({"name": "test", "value": 10})
    print(result)
except Exception as e:
    print(f"错误: {e}")
```

## 4. 性能分析

### 使用cProfile
```python
import cProfile
import pstats
from typing import List

def fibonacci(n: int) -> int:
    """计算斐波那契数"""
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

def process_numbers(numbers: List[int]) -> List[int]:
    """处理数字列表"""
    return [fibonacci(n) for n in numbers]

# 性能分析
def profile_code():
    """性能分析示例"""
    numbers = list(range(20))
    
    # 创建分析器
    profiler = cProfile.Profile()
    
    # 运行代码
    profiler.enable()
    result = process_numbers(numbers)
    profiler.disable()
    
    # 分析结果
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats()

if __name__ == "__main__":
    profile_code()
```

### 使用line_profiler
```python
@profile
def optimize_me(n: int) -> List[int]:
    """需要优化的函数"""
    result = []
    for i in range(n):
        # 一些耗时操作
        temp = []
        for j in range(i):
            temp.append(j ** 2)
        result.extend(temp)
    return result

# 使用kernprof运行:
# kernprof -l -v script.py
```

## 5. 性能优化

### 代码优化
```python
from typing import List, Dict
from collections import defaultdict
import time

class PerformanceExample:
    """性能优化示例"""
    def __init__(self):
        self.cache = {}
    
    def slow_function(self, data: List[int]) -> Dict[int, int]:
        """未优化的函数"""
        result = {}
        for x in data:
            if x not in result:
                result[x] = 0
            result[x] += 1
        return result
    
    def fast_function(self, data: List[int]) -> Dict[int, int]:
        """优化后的函数"""
        return defaultdict(int, {x: data.count(x) for x in set(data)})
    
    def cached_function(self, n: int) -> int:
        """使用缓存的函数"""
        if n in self.cache:
            return self.cache[n]
        
        # 模拟耗时计算
        time.sleep(0.1)
        result = n * n
        self.cache[n] = result
        return result

# 性能比较
def compare_performance():
    """比较性能"""
    pe = PerformanceExample()
    data = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
    
    # 测试未优化的函数
    start = time.time()
    result1 = pe.slow_function(data)
    slow_time = time.time() - start
    
    # 测试优化后的函数
    start = time.time()
    result2 = pe.fast_function(data)
    fast_time = time.time() - start
    
    print(f"慢函数耗时: {slow_time:.4f}秒")
    print(f"快函数耗时: {fast_time:.4f}秒")
    print(f"性能提升: {slow_time/fast_time:.2f}倍")

if __name__ == "__main__":
    compare_performance()
```

## 6. 测试最佳实践

### 测试结构
```python
# tests/
# ├── __init__.py
# ├── conftest.py
# ├── test_models.py
# └── test_views.py

# conftest.py
import pytest
from typing import Generator
from sqlalchemy.orm import Session

@pytest.fixture(scope="session")
def db_session() -> Generator[Session, None, None]:
    """数据库会话fixture"""
    # 设置测试数据库
    engine = create_engine('sqlite:///:memory:')
    Base.metadata.create_all(engine)
    session = sessionmaker(bind=engine)()
    
    yield session
    
    # 清理
    session.close()
    Base.metadata.drop_all(engine)

# test_models.py
def test_user_model(db_session):
    """测试用户模型"""
    user = User(name="Test", email="test@example.com")
    db_session.add(user)
    db_session.commit()
    
    assert user.id is not None
    assert user.name == "Test"

# test_views.py
def test_user_view(client, db_session):
    """测试用户视图"""
    response = client.post(
        "/users/",
        json={"name": "Test", "email": "test@example.com"}
    )
    assert response.status_code == 200
```

### 测试覆盖率
```python
# 使用coverage.py
# 安装：pip install coverage

# .coveragerc
[run]
source = your_package
omit = tests/*

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError
    if __name__ == .__main__.:
    pass

# 运行测试并生成覆盖率报告
# coverage run -m pytest
# coverage report
# coverage html
```

## 7. 练习题

1. 实现测试套件：
```python
class DataProcessor:
    """数据处理器"""
    def process_data(self, data: List[int]) -> List[int]:
        """处理数据"""
        pass

class TestDataProcessor(unittest.TestCase):
    """数据处理器测试"""
    def setUp(self):
        """设置"""
        pass
    
    def test_process_data(self):
        """测试数据处理"""
        pass

# 完成上述类的实现
```

2. 实现性能分析器：
```python
class PerformanceAnalyzer:
    """性能分析器"""
    def analyze_function(self, func, *args, **kwargs):
        """分析函数性能"""
        pass
    
    def generate_report(self):
        """生成报告"""
        pass

# 实现这些类
```

## 小结

本章我们学习了：
- 单元测试的编写
- 集成测试的实现
- 调试技巧和工具
- 性能分析方法
- 代码优化技术
- 测试最佳实践

这些测试和调试技能将帮助你提高代码质量和性能。

## 下一步

在下一章中，我们将学习项目实践，探索如何组织和管理实际的Python项目。

## 额外资源
- [Python单元测试文档](https://docs.python.org/zh-cn/3/library/unittest.html)
- [pytest文档](https://docs.pytest.org/)
- [Python调试技巧](https://realpython.com/python-debugging-pdb/)
- [性能优化指南](https://wiki.python.org/moin/PythonSpeed/PerformanceTips)
