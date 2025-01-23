# Async Programming Patterns in Python

## Introduction

Asynchronous programming is essential for building high-performance applications, especially when dealing with I/O-bound operations like API calls and file operations.

## Core Concepts

### 1. Coroutines

```python
import asyncio

async def example_coroutine():
    print("Starting")
    await asyncio.sleep(1)  # Simulating I/O operation
    print("Finished")

# Running a coroutine
asyncio.run(example_coroutine())
```

### 2. Tasks

```python
async def create_tasks():
    # Create multiple tasks
    task1 = asyncio.create_task(example_coroutine())
    task2 = asyncio.create_task(example_coroutine())
    
    # Wait for both tasks to complete
    await asyncio.gather(task1, task2)
```

## Common Patterns

### 1. Producer-Consumer Pattern

```python
async def producer(queue):
    for i in range(5):
        await queue.put(i)
        await asyncio.sleep(1)

async def consumer(queue):
    while True:
        item = await queue.get()
        print(f"Consumed {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    producer_task = asyncio.create_task(producer(queue))
    consumer_task = asyncio.create_task(consumer(queue))
    await producer_task
    await queue.join()
    consumer_task.cancel()
```

### 2. Async Context Managers

```python
class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(1)
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(1)

async def use_resource():
    async with AsyncResource() as resource:
        print("Using resource")
```

## Error Handling

### 1. Try-Except with Async

```python
async def handle_errors():
    try:
        await risky_operation()
    except Exception as e:
        print(f"Error occurred: {e}")
    finally:
        await cleanup()
```

### 2. Timeout Management

```python
async def with_timeout():
    try:
        async with asyncio.timeout(5):
            await long_operation()
    except asyncio.TimeoutError:
        print("Operation timed out")
```

## Performance Optimization

### 1. Batching Operations

```python
async def process_batch(items):
    async def process_item(item):
        await asyncio.sleep(0.1)  # Simulate processing
        return item * 2

    tasks = [process_item(item) for item in items]
    results = await asyncio.gather(*tasks)
    return results
```

### 2. Connection Pooling

```python
from aiohttp import ClientSession

async def fetch_urls(urls):
    async with ClientSession() as session:
        tasks = []
        for url in urls:
            task = asyncio.create_task(session.get(url))
            tasks.append(task)
        responses = await asyncio.gather(*tasks)
        return responses
```

## Best Practices

1. Always use `asyncio.create_task()` for concurrent operations
2. Implement proper error handling and timeouts
3. Avoid blocking operations in coroutines
4. Use connection pooling for network operations
5. Implement graceful shutdown mechanisms

## Testing Async Code

```python
import pytest

@pytest.mark.asyncio
async def test_async_operation():
    result = await async_operation()
    assert result == expected_value
```

## Common Pitfalls

1. Mixing sync and async code incorrectly
2. Not handling task cancellation properly
3. Blocking the event loop with CPU-intensive tasks
4. Not implementing proper error handling
5. Memory leaks from unmanaged resources

## Conclusion

Asynchronous programming in Python provides powerful tools for building efficient applications. Understanding these patterns and best practices is crucial for developing robust async applications.