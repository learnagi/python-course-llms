---
title: "异步编程基础"
slug: "async-basics"
sequence: 1
description: "深入理解Python异步编程的核心概念和实践应用"
is_published: true
estimated_minutes: 30
---

# 异步编程基础

## 1. 协程基础

### 1.1 协程概念

协程（Coroutine）是Python中实现异步编程的核心机制。它们是可以暂停执行并稍后恢复的特殊函数。

```python
import asyncio
from typing import List

async def example_coroutine():
    print("开始执行")
    await asyncio.sleep(1)  # 模拟IO操作
    print("执行完成")
    return "结果"
```

### 1.2 async/await语法

- `async def`: 定义协程函数
- `await`: 等待一个协程完成
- `asyncio.run()`: 运行协程

## 2. 任务和并发

### 2.1 创建任务

```python
async def main():
    # 创建任务
    task1 = asyncio.create_task(example_coroutine())
    task2 = asyncio.create_task(example_coroutine())
    
    # 等待任务完成
    results = await asyncio.gather(task1, task2)
    return results
```

### 2.2 并发执行

```python
async def process_items(items: List[str]):
    async def process_item(item: str):
        await asyncio.sleep(1)  # 模拟处理时间
        return f"Processed {item}"
    
    tasks = [process_item(item) for item in items]
    return await asyncio.gather(*tasks)
```

## 3. 异步上下文管理器

### 3.1 基本用法

```python
class AsyncResource:
    async def __aenter__(self):
        print("获取资源")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("释放资源")

async def use_resource():
    async with AsyncResource() as resource:
        # 使用资源
        pass
```

### 3.2 异步迭代器

```python
class AsyncIterExample:
    def __init__(self, items):
        self.items = items
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if not self.items:
            raise StopAsyncIteration
        item = self.items.pop(0)
        await asyncio.sleep(0.1)
        return item
```

## 4. 实际应用示例

### 4.1 异步HTTP请求

```python
import aiohttp

async def fetch_url(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def fetch_multiple_urls(urls: List[str]):
    tasks = [fetch_url(url) for url in urls]
    return await asyncio.gather(*tasks)
```

### 4.2 异步数据库操作

```python
from databases import Database

async def async_db_example():
    database = Database('postgresql://user:pass@localhost/db')
    await database.connect()
    
    try:
        query = "SELECT * FROM users"
        rows = await database.fetch_all(query)
        return rows
    finally:
        await database.disconnect()
```

## 5. 最佳实践

1. 避免在协程中使用阻塞操作
2. 合理使用asyncio.gather()和asyncio.create_task()
3. 正确处理异常和清理资源
4. 使用异步上下文管理器管理资源
5. 注意协程的取消和超时处理

## 6. 性能优化

```python
async def optimized_example():
    # 使用信号量限制并发数
    semaphore = asyncio.Semaphore(10)
    
    async def limited_operation():
        async with semaphore:
            await asyncio.sleep(1)
    
    # 创建多个任务但限制并发数
    tasks = [limited_operation() for _ in range(100)]
    await asyncio.gather(*tasks)
```

## 总结

异步编程是Python中处理IO密集型任务的有效方式。通过合理使用协程、任务和异步上下文管理器，可以显著提高应用程序的性能和响应能力。在实际应用中，需要注意正确处理异常、资源管理和并发控制。