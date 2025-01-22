---
title: "项目结构与组织"
slug: "project-structure"
sequence: 1
description: "学习Python项目的最佳实践、结构组织和工程化方法"
is_published: true
estimated_minutes: 30
---

# 项目结构与组织

## 1. 项目结构最佳实践

### 1.1 标准项目结构

```plaintext
my_project/
├── docs/                 # 文档目录
├── src/                  # 源代码目录
│   └── my_project/      # 主要代码包
│       ├── __init__.py
│       ├── core/        # 核心功能
│       ├── utils/       # 工具函数
│       └── config.py    # 配置文件
├── tests/               # 测试目录
├── requirements.txt     # 依赖管理
├── setup.py            # 安装配置
└── README.md           # 项目说明
```

### 1.2 依赖管理

```python
# requirements.txt
fastapi>=0.68.0,<0.69.0
pydantic>=1.8.0,<2.0.0
SQLAlchemy>=1.4.0,<1.5.0

# 开发依赖
# requirements-dev.txt
pytest>=6.0.0
black>=21.5b2
flake8>=3.9.0
```

## 2. 代码组织

### 2.1 模块划分

```python
# src/my_project/core/user.py
from typing import Optional
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    email: Optional[str] = None

    def validate_email(self) -> bool:
        if not self.email:
            return False
        return "@" in self.email
```

### 2.2 配置管理

```python
# src/my_project/config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My Project"
    database_url: str
    api_key: str
    debug: bool = False

    class Config:
        env_file = ".env"

settings = Settings()
```

## 3. 工程化实践

### 3.1 日志管理

```python
# src/my_project/utils/logger.py
import logging
from typing import Optional

def setup_logger(name: str, level: Optional[int] = None) -> logging.Logger:
    logger = logging.getLogger(name)
    if level:
        logger.setLevel(level)
    
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    handler = logging.StreamHandler()
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    
    return logger
```

### 3.2 错误处理

```python
# src/my_project/utils/errors.py
from typing import Dict, Any

class AppError(Exception):
    def __init__(self, message: str, code: int = 500, details: Dict[str, Any] = None):
        self.message = message
        self.code = code
        self.details = details or {}
        super().__init__(self.message)

class ValidationError(AppError):
    def __init__(self, message: str, details: Dict[str, Any] = None):
        super().__init__(message, code=400, details=details)
```

## 4. 开发工具配置

### 4.1 代码格式化

```toml
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py38']
include = '\.pyi?$'

[tool.isort]
profile = "black"
multi_line_output = 3
```

### 4.2 静态类型检查

```toml
# mypy.ini
[mypy]
python_version = 3.8
disallow_untyped_defs = True
disallow_incomplete_defs = True
check_untyped_defs = True
disallow_untyped_decorators = True
no_implicit_optional = True
warn_redundant_casts = True
```

## 5. CI/CD配置

### 5.1 GitHub Actions

```yaml
# .github/workflows/main.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      - name: Run tests
        run: pytest
      - name: Run type checks
        run: mypy src
```

## 6. 文档管理

### 6.1 API文档

```python
# src/my_project/api/users.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter()

class UserCreate(BaseModel):
    """用户创建模型

    Attributes:
        username: 用户名
        email: 电子邮件地址
        password: 密码
    """
    username: str
    email: str
    password: str

@router.post("/users/", response_model=User)
def create_user(user: UserCreate):
    """创建新用户

    Args:
        user: 用户创建数据

    Returns:
        User: 创建的用户信息

    Raises:
        HTTPException: 当用户名已存在时
    """
    if user_exists(user.username):
        raise HTTPException(status_code=400, detail="Username already exists")
    return create_user_in_db(user)
```

## 总结

良好的项目结构和组织是构建可维护、可扩展的Python应用程序的基础。通过采用标准的项目结构、合理的代码组织、完善的工程化实践和自动化工具，可以显著提高开发效率和代码质量。在实际开发中，需要根据项目规模和团队特点选择合适的实践方案。