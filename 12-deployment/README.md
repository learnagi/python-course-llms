# 第十二章：部署与生产环境

本章将介绍如何将LLM应用部署到生产环境，包括环境配置、性能优化、监控和安全性等方面。

## 1. 部署准备

### 环境配置
```python
# config.py
import os
from dataclasses import dataclass
from typing import Optional
from dotenv import load_dotenv

@dataclass
class Config:
    """应用配置类"""
    # API密钥
    openai_api_key: str
    
    # 服务配置
    host: str = "0.0.0.0"
    port: int = 8000
    workers: int = 4
    
    # 数据库配置
    database_url: Optional[str] = None
    
    # 缓存配置
    redis_url: Optional[str] = None
    
    # 日志配置
    log_level: str = "INFO"
    log_file: Optional[str] = "app.log"
    
    @classmethod
    def from_env(cls):
        """从环境变量加载配置"""
        load_dotenv()
        
        return cls(
            openai_api_key=os.getenv("OPENAI_API_KEY"),
            host=os.getenv("HOST", "0.0.0.0"),
            port=int(os.getenv("PORT", "8000")),
            workers=int(os.getenv("WORKERS", "4")),
            database_url=os.getenv("DATABASE_URL"),
            redis_url=os.getenv("REDIS_URL"),
            log_level=os.getenv("LOG_LEVEL", "INFO"),
            log_file=os.getenv("LOG_FILE", "app.log")
        )
```

### 日志配置
```python
# logging_config.py
import logging
from logging.handlers import RotatingFileHandler

def setup_logging(config: Config):
    """配置日志系统"""
    # 创建日志格式
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    # 设置根日志器
    root_logger = logging.getLogger()
    root_logger.setLevel(config.log_level)
    
    # 添加控制台处理器
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(formatter)
    root_logger.addHandler(console_handler)
    
    # 添加文件处理器
    if config.log_file:
        file_handler = RotatingFileHandler(
            config.log_file,
            maxBytes=10485760,  # 10MB
            backupCount=5
        )
        file_handler.setFormatter(formatter)
        root_logger.addHandler(file_handler)
```

## 2. 性能优化

### 异步处理
```python
# async_utils.py
import asyncio
from typing import List, Any
from functools import partial

async def process_batch(batch: List[Any], func, max_concurrency: int = 5):
    """批量异步处理"""
    semaphore = asyncio.Semaphore(max_concurrency)
    
    async def process_item(item):
        async with semaphore:
            return await func(item)
    
    tasks = [process_item(item) for item in batch]
    return await asyncio.gather(*tasks)
```

### 缓存管理
```python
# cache.py
import redis
from functools import wraps
import json
import hashlib

class Cache:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
    
    def cached(self, ttl: int = 3600):
        """缓存装饰器"""
        def decorator(func):
            @wraps(func)
            async def wrapper(*args, **kwargs):
                # 生成缓存键
                key = self._generate_cache_key(func.__name__, args, kwargs)
                
                # 尝试从缓存获取
                cached_result = self.redis.get(key)
                if cached_result:
                    return json.loads(cached_result)
                
                # 执行函数
                result = await func(*args, **kwargs)
                
                # 存入缓存
                self.redis.setex(
                    key,
                    ttl,
                    json.dumps(result)
                )
                
                return result
            return wrapper
        return decorator
    
    def _generate_cache_key(self, func_name: str, args: tuple, kwargs: dict) -> str:
        """生成缓存键"""
        key_parts = [func_name]
        key_parts.extend(str(arg) for arg in args)
        key_parts.extend(f"{k}={v}" for k, v in sorted(kwargs.items()))
        
        key_string = "|".join(key_parts)
        return hashlib.md5(key_string.encode()).hexdigest()
```

## 3. 监控与告警

### 性能监控
```python
# monitoring.py
import time
from functools import wraps
from prometheus_client import Counter, Histogram

# 定义指标
REQUEST_COUNT = Counter(
    'request_total',
    'Total request count',
    ['endpoint']
)

RESPONSE_TIME = Histogram(
    'response_time_seconds',
    'Response time in seconds',
    ['endpoint']
)

def monitor(endpoint: str):
    """监控装饰器"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 记录请求数
            REQUEST_COUNT.labels(endpoint=endpoint).inc()
            
            # 记录响应时间
            start_time = time.time()
            try:
                result = await func(*args, **kwargs)
                return result
            finally:
                RESPONSE_TIME.labels(endpoint=endpoint).observe(
                    time.time() - start_time
                )
        return wrapper
    return decorator
```

### 健康检查
```python
# health.py
from fastapi import FastAPI, HTTPException
import psutil
import openai

async def check_openai_api():
    """检查OpenAI API连接"""
    try:
        await openai.ChatCompletion.acreate(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": "test"}],
            max_tokens=5
        )
        return True
    except:
        return False

async def check_database(db_url: str):
    """检查数据库连接"""
    # 实现数据库连接检查
    return True

def setup_health_checks(app: FastAPI):
    """设置健康检查路由"""
    @app.get("/health")
    async def health_check():
        # 系统资源检查
        cpu_usage = psutil.cpu_percent()
        memory_usage = psutil.virtual_memory().percent
        disk_usage = psutil.disk_usage('/').percent
        
        # API检查
        api_status = await check_openai_api()
        
        # 数据库检查
        db_status = await check_database(app.state.config.database_url)
        
        health_status = {
            "status": "healthy",
            "cpu_usage": cpu_usage,
            "memory_usage": memory_usage,
            "disk_usage": disk_usage,
            "api_status": api_status,
            "database_status": db_status
        }
        
        # 如果任何检查失败，返回不健康状态
        if not all([
            cpu_usage < 90,
            memory_usage < 90,
            disk_usage < 90,
            api_status,
            db_status
        ]):
            health_status["status"] = "unhealthy"
            raise HTTPException(status_code=503, detail=health_status)
        
        return health_status
```

## 4. 安全性配置

### API认证
```python
# auth.py
from fastapi import Security, HTTPException
from fastapi.security.api_key import APIKeyHeader
from starlette.status import HTTP_403_FORBIDDEN
import secrets

api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

class APIKeyAuth:
    def __init__(self, api_keys: set):
        self.api_keys = api_keys
    
    async def __call__(self, api_key: str = Security(api_key_header)):
        if not api_key or api_key not in self.api_keys:
            raise HTTPException(
                status_code=HTTP_403_FORBIDDEN,
                detail="Could not validate API key"
            )
        return api_key

def generate_api_key() -> str:
    """生成安全的API密钥"""
    return secrets.token_urlsafe(32)
```

### 速率限制
```python
# rate_limit.py
import time
from fastapi import HTTPException
from redis import Redis

class RateLimiter:
    def __init__(self, redis_client: Redis):
        self.redis = redis_client
    
    async def check_rate_limit(
        self,
        key: str,
        max_requests: int,
        time_window: int
    ) -> bool:
        """检查速率限制"""
        current = int(time.time())
        pipeline = self.redis.pipeline()
        
        # 添加当前请求
        pipeline.zadd(key, {str(current): current})
        
        # 移除过期的请求
        pipeline.zremrangebyscore(key, 0, current - time_window)
        
        # 获取当前时间窗口内的请求数
        pipeline.zcard(key)
        
        # 设置键的过期时间
        pipeline.expire(key, time_window)
        
        # 执行命令
        _, _, request_count, _ = pipeline.execute()
        
        if request_count > max_requests:
            raise HTTPException(
                status_code=429,
                detail="Too many requests"
            )
        
        return True
```

## 5. 容器化部署

### Dockerfile
```dockerfile
# Dockerfile
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY requirements.txt .

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 设置环境变量
ENV PORT=8000
ENV HOST=0.0.0.0

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:6
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 6. 生产环境配置

### Nginx配置
```nginx
# nginx.conf
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static {
        alias /path/to/your/static/files;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

### Supervisor配置
```ini
# supervisor.conf
[program:llm-app]
command=/path/to/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
directory=/path/to/your/app
user=www-data
autostart=true
autorestart=true
stderr_logfile=/var/log/llm-app.err.log
stdout_logfile=/var/log/llm-app.out.log
```

## 7. 部署脚本

### 部署脚本
```python
# deploy.py
import os
import subprocess
import argparse

def deploy(environment: str):
    """部署应用"""
    # 加载环境配置
    config = load_config(environment)
    
    # 运行数据库迁移
    run_migrations(config)
    
    # 构建Docker镜像
    build_docker_image()
    
    # 启动服务
    start_services(config)
    
    # 健康检查
    check_deployment_health()

def load_config(environment: str) -> dict:
    """加载环境配置"""
    config_file = f"config/{environment}.yml"
    if not os.path.exists(config_file):
        raise ValueError(f"配置文件不存在: {config_file}")
    
    with open(config_file) as f:
        return yaml.safe_load(f)

def run_migrations(config: dict):
    """运行数据库迁移"""
    subprocess.run(["alembic", "upgrade", "head"], check=True)

def build_docker_image():
    """构建Docker镜像"""
    subprocess.run(
        ["docker-compose", "build"],
        check=True
    )

def start_services(config: dict):
    """启动服务"""
    subprocess.run(
        ["docker-compose", "up", "-d"],
        check=True
    )

def check_deployment_health():
    """检查部署健康状态"""
    # 实现健康检查逻辑
    pass

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "environment",
        choices=["development", "staging", "production"],
        help="部署环境"
    )
    args = parser.parse_args()
    
    deploy(args.environment)
```

## 8. 练习题

1. 实现一个基本的部署配置：
```python
async def exercise_1():
    """
    实现一个基本的部署配置，要求：
    1. 配置文件管理
    2. 环境变量处理
    3. 日志配置
    """
    config = Config.from_env()
    setup_logging(config)
    return config
```

2. 实现容器化部署：
```python
async def exercise_2():
    """
    实现容器化部署，要求：
    1. 编写Dockerfile
    2. 配置docker-compose.yml
    3. 实现部署脚本
    """
    # 实现你的代码
    pass
```

## 小结

本章我们学习了：
- 部署准备工作
- 性能优化技术
- 监控与告警系统
- 安全性配置
- 容器化部署
- 生产环境配置
- 部署脚本编写

这些知识将帮助你将LLM应用安全、高效地部署到生产环境。

## 下一步

在接下来的章节中，我们将回顾一些Python基础语法，以确保你掌握了所有必要的编程基础。

## 额外资源
- [FastAPI部署指南](https://fastapi.tiangolo.com/deployment/)
- [Docker文档](https://docs.docker.com/)
- [Nginx配置指南](https://nginx.org/en/docs/)
- [Supervisor文档](http://supervisord.org/)
