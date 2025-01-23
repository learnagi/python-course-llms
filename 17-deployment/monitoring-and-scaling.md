---
title: "监控与扩展"
slug: "monitoring-and-scaling"
sequence: 2
description: "学习LLM应用的监控策略、性能分析和扩展模式"
is_published: true
estimated_minutes: 15
---

# 监控与扩展

## 1. 监控策略

### 关键指标
- 请求延迟
- 吞吐量
- 错误率
- 资源使用率
- API限额使用情况
- 缓存命中率

### Prometheus + Grafana配置
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'llm_app'
    static_configs:
      - targets: ['localhost:8000']
```

### 监控面板示例
```python
from prometheus_client import Counter, Histogram, Gauge

# 请求计数器
REQUEST_COUNTER = Counter(
    'llm_requests_total',
    'Total number of LLM API requests',
    ['endpoint', 'status']
)

# 响应时间
RESPONSE_TIME = Histogram(
    'llm_response_time',
    'Response time in seconds',
    ['endpoint'],
    buckets=(0.1, 0.5, 1.0, 2.0, 5.0)
)

# 当前活跃请求
ACTIVE_REQUESTS = Gauge(
    'llm_active_requests',
    'Number of active requests'
)
```

## 2. 性能分析

### 性能分析工具
```python
import cProfile
import pstats

def profile_request(func):
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        try:
            return profiler.runcall(func, *args, **kwargs)
        finally:
            stats = pstats.Stats(profiler)
            stats.sort_stats('cumulative')
            stats.print_stats()
    return wrapper

@profile_request
async def process_llm_request(prompt: str):
    # 处理请求
    pass
```

### 性能日志
```python
import time
from contextlib import contextmanager

@contextmanager
def timing_log(name: str):
    start = time.time()
    try:
        yield
    finally:
        duration = time.time() - start
        logger.info(f"{name} took {duration:.2f} seconds")

async def handle_request(prompt: str):
    with timing_log("LLM API call"):
        response = await llm_client.generate(prompt)
    return response
```

## 3. 扩展模式

### 水平扩展
```python
# docker-compose.yml
version: '3'

services:
  app:
    build: .
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - REDIS_URL=redis://redis:6379

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### 队列处理
```python
from celery import Celery
from typing import Optional

app = Celery('llm_tasks', broker='redis://localhost:6379/0')

@app.task(bind=True, max_retries=3)
def process_llm_request(self, prompt: str) -> Optional[str]:
    try:
        return llm_client.generate(prompt)
    except Exception as exc:
        self.retry(exc=exc, countdown=2 ** self.request.retries)
```

## 4. 负载测试

### Locust测试脚本
```python
from locust import HttpUser, task, between

class LLMUser(HttpUser):
    wait_time = between(1, 5)

    @task
    def generate_text(self):
        self.client.post(
            "/generate",
            json={"prompt": "测试提示词"},
            headers={"X-API-Key": "test-key"}
        )
```

### 压力测试配置
```python
from locust import HttpUser, task, between
from typing import Dict, Any

class LoadTest(HttpUser):
    wait_time = between(1, 3)
    
    def on_start(self):
        self.headers = {"X-API-Key": "test-key"}
        self.test_prompts = [
            "简单提示词",
            "中等长度的提示词示例",
            "较长的提示词带有上下文信息的示例"
        ]
    
    @task(3)
    def test_short_prompt(self):
        self._send_request(self.test_prompts[0])
    
    @task(2)
    def test_medium_prompt(self):
        self._send_request(self.test_prompts[1])
    
    @task(1)
    def test_long_prompt(self):
        self._send_request(self.test_prompts[2])
    
    def _send_request(self, prompt: str) -> Dict[str, Any]:
        return self.client.post(
            "/generate",
            json={"prompt": prompt},
            headers=self.headers
        )
```

## 5. 故障恢复

### 熔断器模式
```python
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=60)
def call_llm_api(prompt: str) -> str:
    try:
        return llm_client.generate(prompt)
    except Exception as e:
        logger.error(f"LLM API call failed: {e}")
        raise
```

### 降级策略
```python
from typing import Optional

class LLMService:
    def __init__(self):
        self.fallback_responses = {
            "error": "服务暂时不可用，请稍后重试",
            "timeout": "请求处理时间过长，请简化您的提示词"
        }
    
    async def generate(
        self,
        prompt: str,
        timeout: float = 10.0
    ) -> str:
        try:
            return await asyncio.wait_for(
                self._call_api(prompt),
                timeout=timeout
            )
        except asyncio.TimeoutError:
            return self.fallback_responses["timeout"]
        except Exception:
            return self.fallback_responses["error"]
```

## 6. 最佳实践清单

1. **监控设置**
   - [ ] 设置基础监控指标
   - [ ] 配置告警规则
   - [ ] 实现健康检查
   - [ ] 设置日志聚合

2. **性能优化**
   - [ ] 实现缓存策略
   - [ ] 优化响应时间
   - [ ] 减少资源消耗
   - [ ] 实现请求限流

3. **扩展配置**
   - [ ] 配置自动扩展
   - [ ] 实现负载均衡
   - [ ] 设置资源限制
   - [ ] 优化数据存储

4. **故障处理**
   - [ ] 实现熔断机制
   - [ ] 设置重试策略
   - [ ] 准备降级方案
   - [ ] 制定恢复流程