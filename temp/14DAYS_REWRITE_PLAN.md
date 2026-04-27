
# Qbot量化交易平台 - 14天FastAPI重写计划

## 📋 目录
- [一、项目目标](#一项目目标)
- [二、技术方案](#二技术方案)
- [三、14天详细计划](#三14天详细计划)
- [四、每日任务详解](#四每日任务详解)
- [五、代码规范](#五代码规范)
- [六、部署方案](#六部署方案)
- [七、学习资源](#七学习资源)

---

## 一、项目目标

### 1.1 总体目标
在14天内，使用现代化的FastAPI框架完全重写Qbot量化交易平台，重点优化：
- 清晰的三层架构（Controller/Service/Repository）
- 完善的错误处理和数据验证
- 更好的代码可维护性和可测试性
- 完整的API文档
- 上线级别的代码质量

### 1.2 核心成果物

| 时间周期 | 交付物 | 说明 |
|----------|--------|------|
| 第1-2天 | 项目初始化、架构搭建 | 项目骨架、工具配置 |
| 第3-5天 | 核心模块开发 | 用户、认证、数据库 |
| 第6-8天 | 业务功能模块 | 股票、策略、交易 |
| 第9-10天 | 集成与优化 | easytrader/easyquant集成 |
| 第11-12天 | 测试与文档 | 单元测试、API文档 |
| 第13-14天 | 部署与交付 | Docker、性能调优 |

### 1.3 非功能性指标

- ⚡ API响应时间：< 200ms (P95)
- 🔒 代码覆盖率：> 70%
- 📝 API文档完整度：100%
- 🐛 单体模式约束：< 5处代码坏味道

---

## 二、技术方案

### 2.1 后端技术栈（优化方案）

```
Core Framework:
├─ FastAPI 0.100+ (async/await)
├─ Pydantic 2.0+ (数据验证)
└─ SQLAlchemy 2.0+ (ORM)

Database & Storage:
├─ PostgreSQL 14+ (主数据库)
├─ Redis 7.0+ (缓存)
└─ Alembic (数据库迁移)

Async & Background:
├─ celery (异步任务)
├─ RabbitMQ (消息队列)
└─ APScheduler (定时任务)

Testing & Quality:
├─ pytest (单元测试)
├─ pytest-asyncio (异步测试)
├─ coverage (覆盖率)
└─ black + flake8 (代码格式)

Monitoring & Logging:
├─ structlog (结构化日志)
├─ Prometheus (指标收集)
└─ python-json-logger (JSON日志)

Utilities:
├─ python-multipart (文件上传)
├─ python-dotenv (环境变量)
├─ pytz (时区处理)
└─ redis (缓存客户端)

Quantitative:
├─ easytrader (交易接口)
├─ easyquotation (行情接口)
├─ TA-Lib (技术指标)
└─ pandas (数据分析)
```

### 2.2 项目结构设计

```
qbot-fastapi/
├── app/                           # 应用根目录
│   ├── __init__.py
│   ├── main.py                    # 应用入口
│   ├── config.py                  # 配置管理
│   ├── dependencies.py            # 依赖注入
│   │
│   ├── api/                       # API路由层 (Controller)
│   │   ├── v1/                    # API v1版本
│   │   │   ├── users.py           # 用户管理API
│   │   │   ├── auth.py            # 认证API
│   │   │   ├── stocks.py          # 股票管理API
│   │   │   ├── strategies.py      # 策略管理API
│   │   │   ├── trading.py         # 交易API
│   │   │   ├── backtest.py        # 回测API
│   │   │   └── dashboard.py       # 仪表板API
│   │   └── __init__.py
│   │
│   ├── schemas/                   # 数据模型 (DTO)
│   │   ├── user.py                # 用户相关模型
│   │   ├── auth.py                # 认证相关模型
│   │   ├── stock.py               # 股票相关模型
│   │   ├── strategy.py            # 策略相关模型
│   │   ├── trading.py             # 交易相关模型
│   │   ├── base.py                # 基础模型
│   │   └── __init__.py
│   │
│   ├── services/                  # 业务逻辑层 (Service)
│   │   ├── base.py                # 基础服务
│   │   ├── user_service.py        # 用户服务
│   │   ├── auth_service.py        # 认证服务
│   │   ├── stock_service.py       # 股票服务
│   │   ├── strategy_service.py    # 策略服务
│   │   ├── trading_service.py     # 交易服务
│   │   ├── backtest_service.py    # 回测服务
│   │   └── __init__.py
│   │
│   ├── crud/                      # 数据访问层 (Repository)
│   │   ├── base.py                # 基础CRUD操作
│   │   ├── user.py                # 用户CRUD
│   │   ├── stock.py               # 股票CRUD
│   │   ├── strategy.py            # 策略CRUD
│   │   ├── trading_record.py      # 交易记录CRUD
│   │   └── __init__.py
│   │
│   ├── models/                    # 数据库模型 (Entity)
│   │   ├── base.py                # 基础模型
│   │   ├── user.py                # User表
│   │   ├── stock.py               # Stock表
│   │   ├── strategy.py            # Strategy表
│   │   ├── trading_record.py      # TradingRecord表
│   │   ├── backtest_result.py     # BacktestResult表
│   │   └── __init__.py
│   │
│   ├── core/                      # 核心功能
│   │   ├── security.py            # 安全认证
│   │   ├── exceptions.py          # 自定义异常
│   │   ├── logger.py              # 日志配置
│   │   ├── cache.py               # 缓存管理
│   │   └── __init__.py
│   │
│   ├── db/                        # 数据库
│   │   ├── base.py                # 基础配置
│   │   ├── session.py             # 会话管理
│   │   ├── engine.py              # 引擎配置
│   │   └── __init__.py
│   │
│   ├── tasks/                     # 异步任务 (Celery)
│   │   ├── trading_tasks.py       # 交易任务
│   │   ├── backtest_tasks.py      # 回测任务
│   │   ├── data_tasks.py          # 数据采集任务
│   │   └── __init__.py
│   │
│   ├── integrations/              # 外部集成
│   │   ├── easytrader.py          # 交易接口
│   │   ├── easyquotation.py       # 行情接口
│   │   ├── easyquant.py           # 量化框架
│   │   └── __init__.py
│   │
│   ├── utils/                     # 工具函数
│   │   ├── common.py              # 通用工具
│   │   ├── validators.py          # 数据验证
│   │   ├── enums.py               # 枚举定义
│   │   └── __init__.py
│   │
│   └── middleware/                # 中间件
│       ├── auth.py                # 认证中间件
│       ├── error_handler.py       # 错误处理
│       ├── logger.py              # 日志中间件
│       └── __init__.py
│
├── tests/                         # 测试目录
│   ├── conftest.py                # pytest配置
│   ├── test_users.py              # 用户测试
│   ├── test_auth.py               # 认证测试
│   ├── test_stocks.py             # 股票测试
│   ├── test_strategies.py         # 策略测试
│   ├── test_trading.py            # 交易测试
│   └── fixtures.py                # 测试数据
│
├── migrations/                    # 数据库迁移 (Alembic)
│   ├── versions/
│   ├── env.py
│   ├── script.py.mako
│   └── alembic.ini
│
├── docker/                        # Docker配置
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── .dockerignore
│
├── docs/                          # 文档
│   ├── API.md                     # API文档
│   ├── DEPLOYMENT.md              # 部署指南
│   ├── DEVELOPMENT.md             # 开发指南
│   └── architecture.md            # 架构设计
│
├── .env.example                   # 环境变量示例
├── requirements.txt               # Python依赖
├── pyproject.toml                 # 项目配置
├── setup.py                       # 安装脚本
├── pytest.ini                     # pytest配置
├── .flake8                        # flake8配置
└── README.md                      # 项目README
```

### 2.3 核心设计模式

**三层架构**：
```
API层 (Controller)
    ↓ (数据验证、参数提取)
Service层 (业务逻辑)
    ↓ (数据访问)
Repository层 (数据访问)
    ↓ (ORM操作)
Database
```

**依赖注入**：
```python
# 使用Depends进行依赖注入
@app.get("/api/v1/users/{user_id}")
async def get_user(
    user_id: int,
    current_user: User = Depends(get_current_user),
    service: UserService = Depends(get_user_service),
):
    return await service.get_user(user_id)
```

**异常处理**：
```python
# 自定义异常
class QbotException(Exception):
    pass

class UserNotFoundError(QbotException):
    pass

class UnauthorizedError(QbotException):
    pass
```

---

## 三、14天详细计划

### 📅 时间安排总表

| 周期 | 日期 | 任务 | 交付 | 优先级 |
|------|------|------|------|--------|
| **第1周** |  |  |  |  |
| Day 1 | 周一 | 项目初始化、环境配置 | 项目骨架 | 🔴 HIGH |
| Day 2 | 周二 | 数据库设计、模型定义 | DB Schema | 🔴 HIGH |
| Day 3 | 周三 | 认证系统、JWT实现 | Auth模块 | 🔴 HIGH |
| Day 4 | 周四 | 用户系统、权限管理 | User模块 | 🔴 HIGH |
| Day 5 | 周五 | CRUD基础框架 | Base CRUD | 🔴 HIGH |
| **第2周** |  |  |  |  |
| Day 6 | 周一 | 股票模块、行情接口 | Stock模块 | 🟠 MEDIUM |
| Day 7 | 周二 | 策略模块、CRUD操作 | Strategy模块 | 🟠 MEDIUM |
| Day 8 | 周三 | 交易模块、easytrader集成 | Trading模块 | 🟠 MEDIUM |
| Day 9 | 周四 | 回测模块、异步任务 | Backtest模块 | 🟠 MEDIUM |
| Day 10 | 周五 | 仪表板API、统计功能 | Dashboard模块 | 🟡 LOW |
| **第3周** |  |  |  |  |
| Day 11 | 周一 | 单元测试、集成测试 | 测试套件 | 🟠 MEDIUM |
| Day 12 | 周二 | API文档、错误处理完善 | Swagger文档 | 🟠 MEDIUM |
| Day 13 | 周三 | 性能优化、缓存集成 | 优化报告 | 🟡 LOW |
| Day 14 | 周四 | Docker部署、最终检查 | 部署包 | 🔴 HIGH |

---

## 四、每日任务详解

### **第1天：项目初始化与环境配置** 🚀

**目标**：建立FastAPI项目骨架和开发环境

**任务清单**：
1. 创建项目目录结构
2. 初始化Git仓库
3. 配置Python虚拟环境（Python 3.9+）
4. 编写requirements.txt并安装依赖
5. 配置环境变量管理（.env）
6. 初始化FastAPI应用主文件

**代码样例**：
```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1 import router as api_v1_router
from app.core import logger, config

app = FastAPI(
    title="Qbot API",
    description="AI量化交易平台",
    version="2.0.0"
)

# CORS中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=config.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 路由
app.include_router(api_v1_router, prefix="/api/v1")

@app.get("/health")
async def health_check():
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**deliverables**：
- ✅ requirements.txt (核心依赖)
- ✅ app/main.py (应用入口)
- ✅ app/config.py (配置管理)
- ✅ .env.example (环境变量示例)
- ✅ 项目README

**验收标准**：
```bash
python -m uvicorn app.main:app --reload
# 访问 http://localhost:8000/docs → Swagger UI可访问
# 访问 http://localhost:8000/health → {"status": "ok"}
```

---

### **第2天：数据库设计与模型定义** 🗄️

**目标**：设计数据库schema，定义SQLAlchemy models

**任务清单**：
1. 设计数据库schema (ER图)
2. 配置数据库连接和会话
3. 定义SQLAlchemy Base Model
4. 创建所有数据模型（User/Stock/Strategy等）
5. 配置Alembic数据库迁移工具

**数据模型设计**：
```python
# app/models/user.py
from datetime import datetime
from sqlalchemy import Column, String, Integer, DateTime, Boolean
from app.db.base import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(255), unique=True, index=True)
    email = Column(String(255), unique=True, index=True)
    hashed_password = Column(String(255))
    full_name = Column(String(255))
    is_active = Column(Boolean, default=True)
    roles = Column(String(255), default="user")  # 权限角色
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# app/models/stock.py
class Stock(Base):
    __tablename__ = "stocks"

    id = Column(Integer, primary_key=True, index=True)
    code = Column(String(10), unique=True, index=True)
    name = Column(String(255))
    price = Column(Float)
    high = Column(Float)
    low = Column(Float)
    volume = Column(Integer)
    change_rate = Column(Float)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# app/models/strategy.py
class Strategy(Base):
    __tablename__ = "strategies"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    name = Column(String(255), index=True)
    description = Column(String(1024))
    code = Column(String(10000))  # 策略代码
    status = Column(String(50), default="draft")  # draft/active/inactive
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# app/models/trading_record.py
class TradingRecord(Base):
    __tablename__ = "trading_records"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    stock_code = Column(String(10))
    trade_type = Column(String(10))  # buy/sell
    price = Column(Float)
    amount = Column(Integer)
    commission = Column(Float)
    status = Column(String(50))  # pending/executed/cancelled
    created_at = Column(DateTime, default=datetime.utcnow)
    executed_at = Column(DateTime, nullable=True)
```

**deliverables**：
- ✅ app/db/ (数据库配置)
- ✅ app/models/ (所有ORM模型)
- ✅ migrations/versions/001_initial.py (初始迁移)
- ✅ ER图文档

**验收标准**：
```bash
alembic upgrade head
# 检查数据库是否创建成功
sqlite3 app/qbot.db ".tables"
```

---

### **第3天：认证系统实现** 🔐

**目标**：实现JWT认证、OAuth2、Token管理

**任务清单**：
1. 实现密码加密（bcrypt）
2. 实现JWT Token生成和验证
3. 实现OAuth2PasswordBearer
4. 实现Token刷新机制
5. 实现Token黑名单

**代码实现**：
```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def create_refresh_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(days=7)
    to_encode.update({"exp": expire, "type": "refresh"})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

async def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise JWTError("Invalid token")
        return payload
    except JWTError:
        raise JWTError("Invalid token")

# app/schemas/auth.py
from pydantic import BaseModel

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    expires_in: int

class LoginRequest(BaseModel):
    username: str
    password: str

class RefreshTokenRequest(BaseModel):
    refresh_token: str

# app/api/v1/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from app.services.auth_service import AuthService
from app.schemas.auth import LoginRequest, TokenResponse

router = APIRouter(prefix="/auth", tags=["认证"])

@router.post("/login", response_model=TokenResponse)
async def login(
    request: LoginRequest,
    auth_service: AuthService = Depends()
):
    user = await auth_service.authenticate(request.username, request.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid credentials"
        )

    tokens = await auth_service.create_tokens(user.id)
    return tokens

@router.post("/refresh", response_model=TokenResponse)
async def refresh_token(
    request: RefreshTokenRequest,
    auth_service: AuthService = Depends()
):
    tokens = await auth_service.refresh_tokens(request.refresh_token)
    return tokens
```

**deliverables**：
- ✅ app/core/security.py (密钥管理)
- ✅ app/services/auth_service.py (认证服务)
- ✅ app/api/v1/auth.py (认证API)
- ✅ 单元测试: tests/test_auth.py

**验收标准**：
```bash
pytest tests/test_auth.py -v
# 测试用例：
# - test_login_success
# - test_login_invalid_password
# - test_refresh_token
# - test_invalid_token
```

---

### **第4天：用户管理系统** 👥

**目标**：实现用户注册、信息管理、权限管理

**任务清单**：
1. 实现用户注册API
2. 实现用户信息查询API
3. 实现用户更新API
4. 实现用户删除API
5. 实现权限检查中间件

**代码实现**：
```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr
from datetime import datetime

class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: str

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    full_name: Optional[str] = None
    password: Optional[str] = None

class UserResponse(UserBase):
    id: int
    is_active: bool
    roles: str
    created_at: datetime

    class Config:
        from_attributes = True

# app/crud/user.py
from sqlalchemy.orm import Session
from app.models.user import User
from app.core.security import hash_password, verify_password

class UserCRUD:
    @staticmethod
    async def create_user(db: Session, username: str, email: str,
                         full_name: str, password: str) -> User:
        hashed_password = hash_password(password)
        user = User(
            username=username,
            email=email,
            full_name=full_name,
            hashed_password=hashed_password
        )
        db.add(user)
        db.commit()
        db.refresh(user)
        return user

    @staticmethod
    async def get_user_by_username(db: Session, username: str) -> Optional[User]:
        return db.query(User).filter(User.username == username).first()

    @staticmethod
    async def get_user_by_id(db: Session, user_id: int) -> Optional[User]:
        return db.query(User).filter(User.id == user_id).first()

    @staticmethod
    async def update_user(db: Session, user_id: int, **kwargs) -> User:
        user = await UserCRUD.get_user_by_id(db, user_id)
        if user:
            for key, value in kwargs.items():
                if key == "password":
                    value = hash_password(value)
                setattr(user, key, value)
            db.commit()
            db.refresh(user)
        return user

    @staticmethod
    async def delete_user(db: Session, user_id: int) -> bool:
        user = await UserCRUD.get_user_by_id(db, user_id)
        if user:
            db.delete(user)
            db.commit()
            return True
        return False

# app/services/user_service.py
from app.crud.user import UserCRUD
from app.schemas.user import UserCreate, UserResponse

class UserService:
    def __init__(self, db: Session):
        self.db = db

    async def register_user(self, user_data: UserCreate) -> UserResponse:
        # 检查用户是否已存在
        existing_user = await UserCRUD.get_user_by_username(self.db, user_data.username)
        if existing_user:
            raise ValueError("User already exists")

        user = await UserCRUD.create_user(
            self.db,
            username=user_data.username,
            email=user_data.email,
            full_name=user_data.full_name,
            password=user_data.password
        )
        return UserResponse.from_orm(user)

    async def get_user(self, user_id: int) -> UserResponse:
        user = await UserCRUD.get_user_by_id(self.db, user_id)
        if not user:
            raise ValueError("User not found")
        return UserResponse.from_orm(user)

    async def update_user(self, user_id: int, user_data: UserUpdate) -> UserResponse:
        update_data = user_data.dict(exclude_unset=True)
        user = await UserCRUD.update_user(self.db, user_id, **update_data)
        return UserResponse.from_orm(user)

# app/api/v1/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from app.services.user_service import UserService
from app.schemas.user import UserCreate, UserResponse, UserUpdate
from app.core.security import get_current_user

router = APIRouter(prefix="/users", tags=["用户"])

@router.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register(
    user_data: UserCreate,
    service: UserService = Depends()
):
    try:
        return await service.register_user(user_data)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.get("/me", response_model=UserResponse)
async def get_current_user_info(
    current_user: User = Depends(get_current_user)
):
    return UserResponse.from_orm(current_user)

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    service: UserService = Depends(),
    current_user: User = Depends(get_current_user)
):
    return await service.get_user(user_id)

@router.put("/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: int,
    user_data: UserUpdate,
    service: UserService = Depends(),
    current_user: User = Depends(get_current_user)
):
    if current_user.id != user_id:
        raise HTTPException(status_code=403, detail="Forbidden")
    return await service.update_user(user_id, user_data)

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: int,
    service: UserService = Depends(),
    current_user: User = Depends(get_current_user)
):
    if current_user.id != user_id and current_user.roles != "admin":
        raise HTTPException(status_code=403, detail="Forbidden")
    await service.delete_user(user_id)
```

**deliverables**：
- ✅ app/schemas/user.py (用户模型)
- ✅ app/crud/user.py (用户CRUD)
- ✅ app/services/user_service.py (用户服务)
- ✅ app/api/v1/users.py (用户API)
- ✅ tests/test_users.py (用户测试)

**验收标准**：
```bash
pytest tests/test_users.py -v
# 测试用例：
# - test_register_user
# - test_get_user
# - test_update_user
# - test_delete_user
```

---

### **第5天：CRUD基础框架** 🔧

**目标**：建立通用的CRUD操作框架，提高代码复用率

**任务清单**：
1. 设计GenericCRUD基类
2. 实现分页、排序、过滤
3. 实现软删除
4. 实现乐观并发控制

**代码实现**：
```python
# app/crud/base.py
from typing import Any, Dict, Generic, List, Optional, Type, TypeVar
from sqlalchemy.orm import Session
from sqlalchemy import desc, asc

ModelType = TypeVar("ModelType")
CreateSchemaType = TypeVar("CreateSchemaType")
UpdateSchemaType = TypeVar("UpdateSchemaType")

class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
    def __init__(self, model: Type[ModelType]):
        self.model = model

    def get(self, db: Session, id: Any) -> Optional[ModelType]:
        return db.query(self.model).filter(self.model.id == id).first()

    def get_multi(
        self,
        db: Session,
        skip: int = 0,
        limit: int = 100,
        order_by: Optional[str] = None,
        filters: Optional[Dict] = None
    ) -> List[ModelType]:
        query = db.query(self.model)

        # 应用过滤
        if filters:
            for key, value in filters.items():
                if hasattr(self.model, key):
                    query = query.filter(getattr(self.model, key) == value)

        # 应用排序
        if order_by:
            if order_by.startswith("-"):
                query = query.order_by(desc(getattr(self.model, order_by[1:])))
            else:
                query = query.order_by(asc(getattr(self.model, order_by)))

        return query.offset(skip).limit(limit).all()

    def create(self, db: Session, obj_in: CreateSchemaType) -> ModelType:
        obj_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_data)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    def update(
        self,
        db: Session,
        db_obj: ModelType,
        obj_in: UpdateSchemaType
    ) -> ModelType:
        obj_data = jsonable_encoder(obj_in)
        update_data = obj_data.dict(exclude_unset=True)

        for field in update_data:
            setattr(db_obj, field, update_data[field])

        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    def remove(self, db: Session, id: Any) -> ModelType:
        obj = db.query(self.model).get(id)
        if obj:
            db.delete(obj)
            db.commit()
        return obj

# 使用示例
# app/crud/stock.py
from app.crud.base import CRUDBase
from app.models.stock import Stock
from app.schemas.stock import StockCreate, StockUpdate

class StockCRUD(CRUDBase[Stock, StockCreate, StockUpdate]):
    pass

stock_crud = StockCRUD(Stock)
```

**deliverables**：
- ✅ app/crud/base.py (基础CRUD类)
- ✅ app/crud/stock.py (股票CRUD)
- ✅ app/crud/strategy.py (策略CRUD)
- ✅ app/crud/trading_record.py (交易记录CRUD)

**验收标准**：
```bash
# 测试CRUD操作
pytest tests/test_crud.py -v
```

---

### **第6天：股票模块开发** 📈

**目标**：实现股票数据管理和行情接口

**任务清单**：
1. 集成easyquotation行情接口
2. 实现股票查询API
3. 实现实时行情推送
4. 实现行情缓存策略

**代码实现**：
```python
# app/integrations/stock_quotation.py
import easyquotation
from functools import lru_cache
from datetime import datetime, timedelta

class StockQuotation:
    def __init__(self):
        self.quotation = easyquotation.use("qq")
        self.cache = {}
        self.cache_time = {}

    def get_real_quote(self, codes: List[str]) -> Dict:
        # 从缓存读取（5分钟过期）
        now = datetime.now()
        result = {}

        for code in codes:
            if code in self.cache:
                if now - self.cache_time[code] < timedelta(minutes=5):
                    result[code] = self.cache[code]
                    continue

            # 从API获取
            data = self.quotation.real([code])
            if code in data:
                self.cache[code] = data[code]
                self.cache_time[code] = now
                result[code] = data[code]

        return result

# app/schemas/stock.py
class StockPrice(BaseModel):
    code: str
    name: str
    price: float
    high: float
    low: float
    volume: int
    change_rate: float
    bid_price: float
    ask_price: float

class StockResponse(BaseModel):
    id: int
    code: str
    name: str
    price: float
    updated_at: datetime

    class Config:
        from_attributes = True

# app/services/stock_service.py
class StockService:
    def __init__(self, db: Session, quotation: StockQuotation):
        self.db = db
        self.quotation = quotation

    async def get_stock_quote(self, code: str) -> StockPrice:
        quotes = self.quotation.get_real_quote([code])
        if code not in quotes:
            raise ValueError(f"Stock {code} not found")
        return StockPrice(**quotes[code])

    async def get_multi_quotes(self, codes: List[str]) -> List[StockPrice]:
        quotes = self.quotation.get_real_quote(codes)
        return [StockPrice(**quotes[code]) for code in codes if code in quotes]

    async def watch_stock(self, user_id: int, stock_code: str):
        # 保存关注的股票
        watch_record = WatchStock(
            user_id=user_id,
            stock_code=stock_code,
            created_at=datetime.utcnow()
        )
        self.db.add(watch_record)
        self.db.commit()
        return watch_record

# app/api/v1/stocks.py
@router.get("/quote/{code}", response_model=StockPrice)
async def get_stock_quote(
    code: str,
    service: StockService = Depends()
):
    return await service.get_stock_quote(code)

@router.get("/quotes", response_model=List[StockPrice])
async def get_multi_quotes(
    codes: str,  # 逗号分隔的代码
    service: StockService = Depends()
):
    code_list = codes.split(",")
    return await service.get_multi_quotes(code_list)

@router.post("/watch/{code}")
async def watch_stock(
    code: str,
    current_user: User = Depends(get_current_user),
    service: StockService = Depends()
):
    return await service.watch_stock(current_user.id, code)
```

**deliverables**：
- ✅ app/integrations/stock_quotation.py (行情接口)
- ✅ app/schemas/stock.py (股票模型)
- ✅ app/services/stock_service.py (股票服务)
- ✅ app/api/v1/stocks.py (股票API)

**验收标准**：
```bash
pytest tests/test_stocks.py -v
curl http://localhost:8000/api/v1/stocks/quote/000001
# 返回实时行情数据
```

---

### **第7天：策略管理模块** 📊

**目标**：实现策略的CRUD和执行控制

**任务清单**：
1. 实现策略创建、编辑、删除
2. 实现策略查询和搜索
3. 实现策略版本管理
4. 实现策略代码保存和验证

**代码实现**：
```python
# app/schemas/strategy.py
class StrategyBase(BaseModel):
    name: str
    description: Optional[str] = None
    code: str

class StrategyCreate(StrategyBase):
    pass

class StrategyUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    code: Optional[str] = None

class StrategyResponse(StrategyBase):
    id: int
    user_id: int
    status: str
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True

# app/models/strategy.py
class StrategyVersion(Base):
    __tablename__ = "strategy_versions"

    id = Column(Integer, primary_key=True)
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    version = Column(Integer)
    code = Column(String(10000))
    created_at = Column(DateTime, default=datetime.utcnow)

# app/services/strategy_service.py
class StrategyService:
    def __init__(self, db: Session):
        self.db = db

    async def create_strategy(self, user_id: int, data: StrategyCreate) -> StrategyResponse:
        strategy = Strategy(
            user_id=user_id,
            name=data.name,
            description=data.description,
            code=data.code,
            status="draft"
        )
        self.db.add(strategy)
        self.db.commit()
        self.db.refresh(strategy)

        # 保存版本
        version = StrategyVersion(
            strategy_id=strategy.id,
            version=1,
            code=data.code
        )
        self.db.add(version)
        self.db.commit()

        return StrategyResponse.from_orm(strategy)

    async def get_strategy(self, strategy_id: int, user_id: int = None) -> StrategyResponse:
        query = self.db.query(Strategy).filter(Strategy.id == strategy_id)
        if user_id:
            query = query.filter(Strategy.user_id == user_id)

        strategy = query.first()
        if not strategy:
            raise ValueError("Strategy not found")

        return StrategyResponse.from_orm(strategy)

    async def list_strategies(self, user_id: int, skip: int = 0, limit: int = 100):
        strategies = self.db.query(Strategy)\
            .filter(Strategy.user_id == user_id)\
            .offset(skip)\
            .limit(limit)\
            .all()
        return [StrategyResponse.from_orm(s) for s in strategies]

    async def update_strategy(self, strategy_id: int, user_id: int,
                            data: StrategyUpdate) -> StrategyResponse:
        strategy = await self.get_strategy(strategy_id, user_id)

        update_data = data.dict(exclude_unset=True)
        for field, value in update_data.items():
            setattr(strategy, field, value)

        self.db.commit()
        self.db.refresh(strategy)

        return StrategyResponse.from_orm(strategy)

    async def delete_strategy(self, strategy_id: int, user_id: int):
        strategy = await self.get_strategy(strategy_id, user_id)
        self.db.delete(strategy)
        self.db.commit()

# app/api/v1/strategies.py
@router.post("/", response_model=StrategyResponse, status_code=status.HTTP_201_CREATED)
async def create_strategy(
    data: StrategyCreate,
    current_user: User = Depends(get_current_user),
    service: StrategyService = Depends()
):
    return await service.create_strategy(current_user.id, data)

@router.get("/", response_model=List[StrategyResponse])
async def list_strategies(
    skip: int = 0,
    limit: int = 100,
    current_user: User = Depends(get_current_user),
    service: StrategyService = Depends()
):
    return await service.list_strategies(current_user.id, skip, limit)

@router.get("/{strategy_id}", response_model=StrategyResponse)
async def get_strategy(
    strategy_id: int,
    current_user: User = Depends(get_current_user),
    service: StrategyService = Depends()
):
    return await service.get_strategy(strategy_id, current_user.id)

@router.put("/{strategy_id}", response_model=StrategyResponse)
async def update_strategy(
    strategy_id: int,
    data: StrategyUpdate,
    current_user: User = Depends(get_current_user),
    service: StrategyService = Depends()
):
    return await service.update_strategy(strategy_id, current_user.id, data)

@router.delete("/{strategy_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_strategy(
    strategy_id: int,
    current_user: User = Depends(get_current_user),
    service: StrategyService = Depends()
):
    await service.delete_strategy(strategy_id, current_user.id)
```

**deliverables**：
- ✅ app/models/strategy.py (策略模型)
- ✅ app/schemas/strategy.py (策略Schema)
- ✅ app/services/strategy_service.py (策略服务)
- ✅ app/api/v1/strategies.py (策略API)

---

### **第8天：交易模块集成** 💹

**目标**：集成easytrader实盘交易接口

**任务清单**：
1. 集成easytrader交易接口
2. 实现买入API
3. 实现卖出API
4. 实现持仓查询API
5. 实现交易记录API

**代码实现** (持续到第14天)

**第9-14天** 详见下一部分

---

### **第11天：单元测试与集成测试** ✅

**测试框架**：pytest + pytest-asyncio

**测试组织**：
```python
# tests/conftest.py - 测试配置和固件
import pytest
from sqlalchemy import create_engine
from sqlalchemy.pool import StaticPool
from app.db.base import Base
from app.main import app

@pytest.fixture
def test_db():
    # 使用内存SQLite进行测试
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    Base.metadata.create_all(bind=engine)
    yield engine
    Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client():
    return TestClient(app)

# tests/test_users.py
def test_register_user(client):
    response = client.post(
        "/api/v1/users/register",
        json={
            "username": "testuser",
            "email": "test@example.com",
            "full_name": "Test User",
            "password": "testpass123"
        }
    )
    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
```

---

### **第12天：API文档与错误处理** 📚

**API文档**：Swagger/OpenAPI

```python
# app/main.py
app = FastAPI(
    title="Qbot API",
    description="AI Quantitative Trading Platform",
    version="2.0.0",
    docs_url="/api/docs",
    openapi_url="/api/openapi.json"
)

# app/core/exceptions.py
class QbotException(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

@app.exception_handler(QbotException)
async def qbot_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={"code": exc.code, "message": exc.message}
    )
```

**访问地址**：
- Swagger UI: `http://localhost:8000/api/docs`
- ReDoc: `http://localhost:8000/api/redoc`

---

### **第13天：性能优化与缓存** ⚡

**优化方案**：
1. Redis缓存集成
2. 查询优化（数据库索引）
3. 连接池配置
4. 响应时间监控

```python
# app/core/cache.py
import redis
from typing import Any, Optional

class CacheManager:
    def __init__(self, redis_url: str = "redis://localhost"):
        self.redis = redis.from_url(redis_url)

    async def get(self, key: str) -> Optional[Any]:
        value = self.redis.get(key)
        return json.loads(value) if value else None

    async def set(self, key: str, value: Any, ttl: int = 3600):
        self.redis.setex(key, ttl, json.dumps(value))

    async def delete(self, key: str):
        self.redis.delete(key)

# 使用缓存
@router.get("/stocks/quote/{code}")
@cache(ttl=300)  # 5分钟缓存
async def get_stock_quote(code: str):
    ...
```

---

### **第14天：Docker部署与最终检查** 🐳

**Docker化**：
```dockerfile
# docker/Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: docker/Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/qbot
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=qbot
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

**启动命令**：
```bash
docker-compose up -d
# 访问 http://localhost:8000/api/docs
```

---

## 五、代码规范

### 5.1 编码规范

**目录规范**：
```
- 模块名小写：user, stock, strategy
- 文件名小写，使用下划线：user_service.py
- 类名PascalCase：UserService, StockCRUD
- 函数名snake_case：get_user_by_id()
- 常量大写：MAX_RETRIES, DEFAULT_TIMEOUT
```

**命名规范**：
```python
# 模型
class User(Base):  # 数据库表对应的模型
    pass

# Schema（对应Pydantic模型）
class UserResponse(BaseModel):  # 响应格式
    pass

class UserCreate(BaseModel):  # 创建请求
    pass

# 服务
class UserService:  # 业务逻辑
    pass

# CRUD
class UserCRUD(CRUDBase):  # 数据访问
    pass

# API
@router.get("/users")  # 获取多个
async def list_users():
    pass

@router.post("/users")  # 创建
async def create_user():
    pass

@router.get("/users/{id}")  # 获取单个
async def get_user(id: int):
    pass

@router.put("/users/{id}")  # 更新
async def update_user(id: int):
    pass

@router.delete("/users/{id}")  # 删除
async def delete_user(id: int):
    pass
```

### 5.2 注释规范

```python
def get_stock_price(code: str, date: datetime) -> float:
    """
    获取指定股票在指定日期的收盘价

    Args:
        code: 股票代码（如'000001'）
        date: 查询日期

    Returns:
        float: 收盘价

    Raises:
        ValueError: 如果股票代码不存在

    Examples:
        >>> price = get_stock_price('000001', datetime(2024, 1, 1))
        >>> print(price)
        10.5
    """
    ...
```

### 5.3 类型提示规范

```python
from typing import List, Optional, Dict, Any
from datetime import datetime

# 函数参数和返回值都要有类型提示
def get_users(
    skip: int = 0,
    limit: int = 100,
    filters: Optional[Dict[str, Any]] = None
) -> List[UserResponse]:
    pass

# Pydantic模型要有完整的字段类型
class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    created_at: datetime
    is_active: bool = True
```

### 5.4 异常处理规范

```python
# 定义自定义异常
class QbotException(Exception):
    pass

class UserNotFoundError(QbotException):
    pass

class UnauthorizedError(QbotException):
    pass

# 使用异常
async def get_user(user_id: int) -> User:
    user = await UserCRUD.get_user_by_id(db, user_id)
    if not user:
        raise UserNotFoundError(f"User {user_id} not found")
    return user
```

### 5.5 API设计规范

**RESTful原则**：
```
GET    /api/v1/users              → 获取用户列表
POST   /api/v1/users              → 创建用户
GET    /api/v1/users/{id}         → 获取单个用户
PUT    /api/v1/users/{id}         → 更新用户
DELETE /api/v1/users/{id}         → 删除用户

GET    /api/v1/stocks/quote/{code}    → 获取股票行情
POST   /api/v1/strategies             → 创建策略
GET    /api/v1/strategies/{id}        → 获取策略
PUT    /api/v1/strategies/{id}        → 更新策略
DELETE /api/v1/strategies/{id}        → 删除策略
```

**请求/响应格式**：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "username": "test"
  },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

---

## 六、部署方案

### 6.1 开发环境

```bash
# 1. 克隆项目
git clone https://github.com/yourusername/qbot-fastapi
cd qbot-fastapi

# 2. 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# 3. 安装依赖
pip install -r requirements.txt

# 4. 配置环境变量
cp .env.example .env
# 编辑.env文件

# 5. 初始化数据库
alembic upgrade head

# 6. 运行开发服务器
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 6.2 生产环境

```bash
# Docker构建和运行
docker build -t qbot-api:latest -f docker/Dockerfile .
docker-compose up -d

# 或使用Kubernetes
kubectl apply -f k8s/deployment.yaml
```

### 6.3 监控和日志

```python
# 日志配置
import structlog

structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

log = structlog.get_logger()
log.info("event", key="value")
```

---

## 七、学习资源

### 7.1 官方文档

- [FastAPI官方文档](https://fastapi.tiangolo.com/)
- [SQLAlchemy文档](https://docs.sqlalchemy.org/)
- [Pydantic文档](https://docs.pydantic.dev/)
- [pytest文档](https://docs.pytest.org/)

### 7.2 推荐书籍

1. **Building Data-Driven Applications with Danfo.js** - 数据处理
2. **Designing Machine Learning Systems** - ML系统设计
3. **Site Reliability Engineering** - 系统可靠性

### 7.3 在线课程

- FastAPI进阶教程（Udemy/YouTube）
- 量化交易基础（QuantInsti/Coursera）
- 系统设计面试（System Design Interview）

### 7.4 开源项目参考

- [tiangolo/fastapi](https://github.com/tiangolo/fastapi)
- [encode/starlette](https://github.com/encode/starlette)
- [sqlalchemy/sqlalchemy](https://github.com/sqlalchemy/sqlalchemy)

---

## 总结

本14天重写计划通过**渐进式**的方式逐步构建一个生产级别的Qbot量化交易平台。关键要点：

✅ **第1-5天**：基础设施（架构、认证、数据库）
✅ **第6-10天**：核心业务模块（股票、策略、交易）
✅ **第11-14天**：测试、优化、部署

**预期成果**：
- 完整的三层架构
- 100+个API端点
- 70%+代码覆盖率
- 上线级别的代码质量
- 完整的API文档

**学习收获**：
- 掌握FastAPI高级特性
- 理解量化交易平台架构
- 学会企业级代码组织方式
- 熟悉测试驱动开发（TDD）

加油! 🚀


