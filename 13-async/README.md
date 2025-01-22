# 第十四章：异步编程

本章将介绍Python的异步编程概念，包括协程基础、async/await语法、异步IO和并发编程。

## 1. 协程基础

### 协程概念
```python
import asyncio
from typing import List

async def hello(name: str):
    """简单的协程函数"""
    print(f"开始: {name}")
    await asyncio.sleep(1)  # 模拟IO操作
    print(f"结束: {name}")
    return f"Hello, {name}!"

# 运行单个协程
async def main():
    result = await hello("World")
    print(result)

# 运行多个协程
async def main_multiple():
    names = ["Alice", "Bob", "Charlie"]
    tasks = [hello(name) for name in names]
    results = await asyncio.gather(*tasks)
    print(results)

# 使用示例
asyncio.run(main())
asyncio.run(main_multiple())
```

### 任务和Future
```python
async def long_operation():
    """长时间运行的操作"""
    print("开始长时间操作")
    await asyncio.sleep(2)
    print("长时间操作完成")
    return "操作结果"

async def task_demo():
    """任务示例"""
    # 创建任务
    task = asyncio.create_task(long_operation())
    
    # 等待任务完成
    print("等待任务完成...")
    result = await task
    print(f"获得结果: {result}")

# 使用示例
asyncio.run(task_demo())
```

## 2. async/await语法

### 基本语法
```python
async def fetch_data(url: str) -> str:
    """模拟异步数据获取"""
    print(f"开始获取数据: {url}")
    await asyncio.sleep(1)  # 模拟网络延迟
    print(f"数据获取完成: {url}")
    return f"来自 {url} 的数据"

async def process_data(data: str) -> str:
    """模拟异步数据处理"""
    print(f"开始处理数据: {data}")
    await asyncio.sleep(0.5)  # 模拟处理时间
    print(f"数据处理完成: {data}")
    return f"处理后的 {data}"

async def main():
    # 串行执行
    data = await fetch_data("http://example.com")
    result = await process_data(data)
    print(result)
    
    # 并行执行
    urls = [
        "http://example.com/1",
        "http://example.com/2",
        "http://example.com/3"
    ]
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)
    print(results)

# 使用示例
asyncio.run(main())
```

### 异常处理
```python
async def risky_operation(success: bool):
    """可能失败的操作"""
    await asyncio.sleep(1)
    if not success:
        raise ValueError("操作失败")
    return "操作成功"

async def handle_errors():
    """异常处理示例"""
    try:
        # 尝试执行可能失败的操作
        result = await risky_operation(False)
        print(result)
    except ValueError as e:
        print(f"捕获到错误: {e}")
    
    # 使用gather处理多个操作的异常
    tasks = [
        risky_operation(True),
        risky_operation(False),
        risky_operation(True)
    ]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    for result in results:
        if isinstance(result, Exception):
            print(f"操作失败: {result}")
        else:
            print(f"操作成功: {result}")

# 使用示例
asyncio.run(handle_errors())
```

## 3. 异步IO

### 文件操作
```python
import aiofiles
import os

async def async_file_ops():
    """异步文件操作示例"""
    # 写文件
    async with aiofiles.open('test.txt', 'w') as f:
        await f.write('Hello, Async IO!')
    
    # 读文件
    async with aiofiles.open('test.txt', 'r') as f:
        content = await f.read()
        print(f"文件内容: {content}")
    
    # 删除文件
    os.remove('test.txt')

# 使用示例
asyncio.run(async_file_ops())
```

### 网络请求
```python
import aiohttp
from typing import List

async def fetch_url(session: aiohttp.ClientSession, url: str) -> str:
    """获取URL内容"""
    async with session.get(url) as response:
        return await response.text()

async def fetch_multiple_urls(urls: List[str]):
    """并发获取多个URL内容"""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

# 使用示例
async def main():
    urls = [
        'http://example.com',
        'http://example.org',
        'http://example.net'
    ]
    results = await fetch_multiple_urls(urls)
    for url, html in zip(urls, results):
        print(f"{url}: {len(html)} bytes")

asyncio.run(main())
```

## 4. 并发编程

### 异步队列
```python
from asyncio import Queue
from typing import List

async def producer(queue: Queue, items: List[str]):
    """生产者"""
    for item in items:
        await queue.put(item)
        print(f"生产: {item}")
        await asyncio.sleep(1)
    
    # 添加结束标记
    await queue.put(None)

async def consumer(queue: Queue):
    """消费者"""
    while True:
        item = await queue.get()
        if item is None:
            break
        
        print(f"消费: {item}")
        await asyncio.sleep(2)
        queue.task_done()

async def main():
    queue = Queue()
    items = ["item1", "item2", "item3", "item4"]
    
    # 创建生产者和消费者任务
    producer_task = asyncio.create_task(producer(queue, items))
    consumer_task = asyncio.create_task(consumer(queue))
    
    # 等待所有任务完成
    await asyncio.gather(producer_task, consumer_task)

# 使用示例
asyncio.run(main())
```

### 信号量控制
```python
async def limited_concurrent_operation(semaphore: asyncio.Semaphore, task_id: int):
    """受限的并发操作"""
    async with semaphore:
        print(f"任务 {task_id} 开始")
        await asyncio.sleep(1)
        print(f"任务 {task_id} 完成")

async def main():
    # 限制最大并发数为3
    semaphore = asyncio.Semaphore(3)
    
    # 创建10个任务
    tasks = [
        limited_concurrent_operation(semaphore, i)
        for i in range(10)
    ]
    
    await asyncio.gather(*tasks)

# 使用示例
asyncio.run(main())
```

## 5. 实际应用示例

### 异步API客户端
```python
class AsyncAPIClient:
    """异步API客户端"""
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = None
    
    async def __aenter__(self):
        """异步上下文管理器入口"""
        self.session = aiohttp.ClientSession()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """异步上下文管理器出口"""
        if self.session:
            await self.session.close()
    
    async def get(self, endpoint: str) -> dict:
        """GET请求"""
        async with self.session.get(f"{self.base_url}{endpoint}") as response:
            return await response.json()
    
    async def post(self, endpoint: str, data: dict) -> dict:
        """POST请求"""
        async with self.session.post(f"{self.base_url}{endpoint}", json=data) as response:
            return await response.json()

# 使用示例
async def main():
    async with AsyncAPIClient("https://api.example.com") as client:
        # 获取用户数据
        user_data = await client.get("/users/1")
        print(user_data)
        
        # 创建新用户
        new_user = await client.post("/users", {
            "name": "John Doe",
            "email": "john@example.com"
        })
        print(new_user)

asyncio.run(main())
```

### 异步Web爬虫
```python
class AsyncWebCrawler:
    """异步网页爬虫"""
    def __init__(self, max_concurrent: int = 5):
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.session = None
        self.results = []
    
    async def crawl_page(self, url: str):
        """爬取单个页面"""
        async with self.semaphore:
            try:
                async with self.session.get(url) as response:
                    html = await response.text()
                    self.results.append({
                        "url": url,
                        "status": response.status,
                        "content_length": len(html)
                    })
            except Exception as e:
                print(f"爬取 {url} 时出错: {e}")
    
    async def crawl_multiple(self, urls: List[str]):
        """爬取多个页面"""
        async with aiohttp.ClientSession() as session:
            self.session = session
            tasks = [self.crawl_page(url) for url in urls]
            await asyncio.gather(*tasks)
            return self.results

# 使用示例
async def main():
    crawler = AsyncWebCrawler()
    urls = [
        "http://example.com",
        "http://example.org",
        "http://example.net"
    ]
    results = await crawler.crawl_multiple(urls)
    for result in results:
        print(result)

asyncio.run(main())
```

## 6. 练习题

1. 实现异步文件处理器：
```python
class AsyncFileProcessor:
    """异步文件处理器"""
    def __init__(self, input_dir: str, output_dir: str):
        self.input_dir = input_dir
        self.output_dir = output_dir
    
    async def process_file(self, filename: str):
        """处理单个文件"""
        pass
    
    async def process_all_files(self):
        """处理所有文件"""
        pass

# 完成上述类的实现
```

2. 实现异步任务调度器：
```python
class AsyncTaskScheduler:
    """异步任务调度器"""
    def __init__(self):
        self.tasks = []
    
    async def add_task(self, coroutine):
        """添加任务"""
        pass
    
    async def run_all(self):
        """运行所有任务"""
        pass

# 实现这些类
```

## 小结

本章我们学习了：
- 协程的基本概念
- async/await语法的使用
- 异步IO操作
- 并发编程技术
- 实际应用示例

这些异步编程的知识将帮助你构建高效的并发应用。

## 下一步

在下一章中，我们将学习测试与调试，探索如何确保代码质量和性能。

## 额外资源
- [Python官方文档 - asyncio](https://docs.python.org/zh-cn/3/library/asyncio.html)
- [aiohttp文档](https://docs.aiohttp.org/)
- [Python异步编程指南](https://realpython.com/async-io-python/)
