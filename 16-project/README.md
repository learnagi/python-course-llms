# 第十六章：项目实践

本章将介绍Python项目的最佳实践，包括项目结构、代码规范、文档编写和开发流程。

## 1. 项目结构

### 标准项目结构
```
my_project/
├── README.md              # 项目说明文档
├── LICENSE               # 许可证文件
├── setup.py              # 安装脚本
├── requirements.txt      # 依赖列表
├── my_project/          # 主代码目录
│   ├── __init__.py
│   ├── main.py          # 主入口
│   ├── config.py        # 配置文件
│   ├── models/          # 数据模型
│   │   ├── __init__.py
│   │   └── user.py
│   ├── services/        # 业务逻辑
│   │   ├── __init__.py
│   │   └── auth.py
│   └── utils/           # 工具函数
│       ├── __init__.py
│       └── helpers.py
├── tests/               # 测试目录
│   ├── __init__.py
│   ├── conftest.py
│   └── test_models.py
└── docs/                # 文档目录
    ├── api.md
    └── usage.md
```

### 项目配置
```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="my_project",
    version="0.1.0",
    packages=find_packages(),
    install_requires=[
        "fastapi>=0.68.0",
        "sqlalchemy>=1.4.0",
        "pydantic>=1.8.0",
    ],
    author="Your Name",
    author_email="your.email@example.com",
    description="A short description of your project",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/username/project",
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.8",
)
```

## 2. 代码规范

### 代码风格
```python
# 遵循PEP 8规范的代码示例
from typing import List, Optional
from datetime import datetime

class User:
    """用户类
    
    这个类表示系统中的用户实体，包含用户的基本信息和操作。
    
    Attributes:
        username: 用户名
        email: 电子邮件
        created_at: 创建时间
    """
    
    def __init__(
        self,
        username: str,
        email: str,
        created_at: Optional[datetime] = None
    ):
        self.username = username
        self.email = email
        self.created_at = created_at or datetime.now()
    
    def get_info(self) -> dict:
        """获取用户信息
        
        Returns:
            包含用户基本信息的字典
        """
        return {
            "username": self.username,
            "email": self.email,
            "created_at": self.created_at.isoformat()
        }
```

### 类型注解
```python
from typing import Dict, List, Optional, Union, TypeVar, Generic

T = TypeVar('T')

class Repository(Generic[T]):
    """通用仓储类"""
    def __init__(self):
        self.items: Dict[str, T] = {}
    
    def add(self, key: str, item: T) -> None:
        """添加项目"""
        self.items[key] = item
    
    def get(self, key: str) -> Optional[T]:
        """获取项目"""
        return self.items.get(key)
    
    def list_all(self) -> List[T]:
        """列出所有项目"""
        return list(self.items.values())

# 使用示例
class Product:
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

product_repo: Repository[Product] = Repository()
product_repo.add("p1", Product("测试产品", 99.9))
```

## 3. 文档编写

### 代码文档
```python
def calculate_discount(
    price: float,
    discount_rate: float,
    max_discount: Optional[float] = None
) -> float:
    """计算折扣价格
    
    根据原价和折扣率计算折扣后的价格。如果指定了最大折扣金额，
    则折扣不会超过这个金额。
    
    Args:
        price: 原价
        discount_rate: 折扣率（0-1之间的小数）
        max_discount: 最大折扣金额（可选）
    
    Returns:
        折扣后的价格
    
    Raises:
        ValueError: 当折扣率不在0-1之间时
    
    Examples:
        >>> calculate_discount(100, 0.1)
        90.0
        >>> calculate_discount(100, 0.2, 15)
        85.0
    """
    if not 0 <= discount_rate <= 1:
        raise ValueError("折扣率必须在0-1之间")
    
    discount_amount = price * discount_rate
    if max_discount is not None:
        discount_amount = min(discount_amount, max_discount)
    
    return price - discount_amount
```

### API文档
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI(
    title="我的API",
    description="API接口文档",
    version="1.0.0"
)

class Item(BaseModel):
    """商品模型
    
    Attributes:
        name: 商品名称
        price: 商品价格
        description: 商品描述（可选）
    """
    name: str
    price: float
    description: Optional[str] = None

@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Item:
    """创建新商品
    
    Args:
        item: 商品信息
    
    Returns:
        创建的商品对象
    
    Raises:
        HTTPException: 当商品价格为负数时
    """
    if item.price < 0:
        raise HTTPException(status_code=400, detail="商品价格不能为负")
    return item
```

## 4. 开发流程

### 版本控制
```python
# .gitignore
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
.env
.venv
venv/
ENV/
```

### 依赖管理
```python
# requirements.txt
fastapi==0.68.0
uvicorn==0.15.0
sqlalchemy==1.4.23
pydantic==1.8.2
python-dotenv==0.19.0
pytest==6.2.5
black==21.7b0
isort==5.9.3
flake8==3.9.2
mypy==0.910

# requirements-dev.txt
-r requirements.txt
pytest-cov==2.12.1
black==21.7b0
isort==5.9.3
flake8==3.9.2
mypy==0.910
```

## 5. 最佳实践示例

### 配置管理
```python
# config.py
from pydantic import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    """应用配置
    
    使用pydantic管理配置，支持从环境变量加载
    """
    # 数据库配置
    database_url: str
    database_pool_size: int = 5
    
    # API配置
    api_key: str
    api_prefix: str = "/api/v1"
    
    # 缓存配置
    redis_url: Optional[str] = None
    cache_ttl: int = 3600
    
    class Config:
        env_file = ".env"
        case_sensitive = False

settings = Settings()
```

### 依赖注入
```python
# dependencies.py
from fastapi import Depends, HTTPException
from sqlalchemy.orm import Session
from typing import Generator

def get_db() -> Generator[Session, None, None]:
    """数据库会话依赖"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """获取当前用户依赖"""
    user = authenticate_token(token, db)
    if not user:
        raise HTTPException(
            status_code=401,
            detail="Invalid authentication credentials"
        )
    return user
```

### 错误处理
```python
# exceptions.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

class AppException(Exception):
    """应用异常基类"""
    def __init__(self, message: str, code: int = 400):
        self.message = message
        self.code = code

@app.exception_handler(AppException)
async def app_exception_handler(
    request: Request,
    exc: AppException
) -> JSONResponse:
    """处理应用异常"""
    return JSONResponse(
        status_code=exc.code,
        content={"message": exc.message}
    )

@app.exception_handler(Exception)
async def global_exception_handler(
    request: Request,
    exc: Exception
) -> JSONResponse:
    """处理全局异常"""
    return JSONResponse(
        status_code=500,
        content={"message": "Internal server error"}
    )
```

## 6. 项目示例

### LLM应用项目
```python
# llm_app/
# ├── README.md
# ├── requirements.txt
# ├── setup.py
# ├── llm_app/
# │   ├── __init__.py
# │   ├── main.py
# │   ├── config.py
# │   ├── models/
# │   │   ├── __init__.py
# │   │   ├── conversation.py
# │   │   └── user.py
# │   ├── services/
# │   │   ├── __init__.py
# │   │   ├── llm.py
# │   │   └── chat.py
# │   └── utils/
# │       ├── __init__.py
# │       └── helpers.py
# ├── tests/
# │   ├── __init__.py
# │   ├── conftest.py
# │   └── test_services.py
# └── docs/
#     ├── api.md
#     └── usage.md

# main.py
from fastapi import FastAPI, Depends
from .config import settings
from .services.llm import LLMService
from .services.chat import ChatService

app = FastAPI(title=settings.app_name)

@app.post("/chat")
async def chat(
    message: str,
    chat_service: ChatService = Depends()
) -> dict:
    """处理聊天请求"""
    response = await chat_service.process_message(message)
    return {"response": response}

# services/llm.py
class LLMService:
    """LLM服务"""
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    async def generate_response(
        self,
        prompt: str,
        max_tokens: int = 100
    ) -> str:
        """生成回复"""
        # 实现LLM调用逻辑
        pass

# services/chat.py
class ChatService:
    """聊天服务"""
    def __init__(self, llm_service: LLMService):
        self.llm_service = llm_service
    
    async def process_message(self, message: str) -> str:
        """处理消息"""
        # 实现消息处理逻辑
        pass
```

## 7. 练习题

1. 创建项目模板：
```python
def create_project_template(project_name: str):
    """创建项目模板"""
    # 实现项目模板创建逻辑
    pass

# 完成上述函数的实现
```

2. 实现配置管理：
```python
class ProjectConfig:
    """项目配置管理"""
    def load_config(self, env: str):
        """加载配置"""
        pass
    
    def get_setting(self, key: str):
        """获取配置项"""
        pass

# 实现这些类
```

## 小结

本章我们学习了：
- 项目结构组织
- 代码规范和风格
- 文档编写方法
- 开发流程管理
- 最佳实践示例
- 实际项目示例

这些知识将帮助你更好地组织和管理Python项目。

## 课程总结

恭喜你完成了整个课程！现在你已经掌握了：
- Python编程基础
- Web开发技能
- LLM应用开发
- 测试和调试技术
- 项目管理最佳实践

继续练习和实践，将这些知识应用到实际项目中！

## 额外资源
- [Python项目结构指南](https://docs.python-guide.org/writing/structure/)
- [Python代码风格指南](https://www.python.org/dev/peps/pep-0008/)
- [FastAPI最佳实践](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [Python项目模板](https://github.com/cookiecutter/cookiecutter)
