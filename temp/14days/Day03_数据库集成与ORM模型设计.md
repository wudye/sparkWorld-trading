# Day 3: 数据库集成与ORM模型设计

## 学习目标
- ✅ 掌握SQLAlchemy ORM框架
- ✅ 设计Qbot的数据库模型
- ✅ 实现数据库连接池和会话管理
- ✅ 创建数据库初始化脚本
- ✅ 实现基础数据访问层（Repository）

## 核心学习内容

### 1. SQLAlchemy基础
- ORM概念：对象关系映射
- SQLAlchemy架构（Core vs ORM）
- 模型定义和关系映射
- 会话（Session）管理
- 查询API

### 2. Qbot数据库设计

核心实体：
```
Users（用户）
├── Strategies（策略）
│   ├── StrategyParameters（参数）
│   └── StrategyHistory（历史版本）
├── Backtests（回测任务）
│   ├── BacktestResults（回测结果）
│   └── BacktestMetrics（性能指标）
├── Trades（交易单）
│   ├── Orders（订单）
│   └── Positions（持仓）
└── Portfolio（投资组合）

Stocks（股票基础数据）
├── StockPrices（价格序列）
├── StockFactors（因子数据）
└── StockIndicators（技术指标）
```

### 3. 数据库连接管理
- 连接池配置
- 事务管理
- 异常处理
- 连接超时和重试

### 4. 数据迁移
- Alembic基础
- 迁移脚本管理
- 版本控制

## 关键技术点

1. **关系类型**
   - 一对多（One-to-Many）
   - 多对多（Many-to-Many）
   - 一对一（One-to-One）

2. **模型有效期**
   - created_at, updated_at 时间戳
   - 软删除设计
   - 数据版本管理

3. **查询优化**
   - 懒加载vs急加载
   - 联接优化
   - 索引设计

## 实践任务

### 任务1：安装依赖（30分钟）
```bash
uv add sqlalchemy psycopg2-binary alembic python-dotenv
```

创建 `app/core/database.py`：
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from sqlalchemy.pool import QueuePool
import os

DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://qbot:qbot@localhost:5432/qbot"
)

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
    echo=False
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 任务2：定义核心模型（2小时）

创建 `app/models/models.py`：
```python
from sqlalchemy import Column, Integer, String, Float, DateTime, Boolean, ForeignKey, Text, Enum
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.core.database import Base
from datetime import datetime
import enum

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True)
    email = Column(String, unique=True, index=True)
    password_hash = Column(String)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, onupdate=func.now())

    strategies = relationship("Strategy", back_populates="user")
    backtests = relationship("Backtest", back_populates="user")
    trades = relationship("Trade", back_populates="user")

class StrategyStatus(str, enum.Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    BACKTESTING = "backtesting"
    ERROR = "error"

class Strategy(Base):
    __tablename__ = "strategies"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), index=True)
    name = Column(String, index=True)
    description = Column(Text)
    code = Column(Text)  # 策略代码
    status = Column(String, default=StrategyStatus.INACTIVE.value)
    version = Column(Integer, default=1)
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, onupdate=func.now())

    user = relationship("User", back_populates="strategies")
    parameters = relationship("StrategyParameter", back_populates="strategy")
    backtests = relationship("Backtest", back_populates="strategy")

class StrategyParameter(Base):
    __tablename__ = "strategy_parameters"

    id = Column(Integer, primary_key=True, index=True)
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    param_name = Column(String)
    param_value = Column(String)
    param_type = Column(String)  # int, float, string

    strategy = relationship("Strategy", back_populates="parameters")

class Backtest(Base):
    __tablename__ = "backtests"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    name = Column(String)
    start_date = Column(DateTime)
    end_date = Column(DateTime)
    initial_capital = Column(Float)
    status = Column(String)  # running, completed, failed
    progress = Column(Float, default=0.0)
    created_at = Column(DateTime, server_default=func.now())

    user = relationship("User", back_populates="backtests")
    strategy = relationship("Strategy", back_populates="backtests")
    results = relationship("BacktestResult", back_populates="backtest")

class BacktestResult(Base):
    __tablename__ = "backtest_results"

    id = Column(Integer, primary_key=True, index=True)
    backtest_id = Column(Integer, ForeignKey("backtests.id"))
    total_return = Column(Float)
    annual_return = Column(Float)
    max_drawdown = Column(Float)
    sharpe_ratio = Column(Float)
    win_rate = Column(Float)
    trades_count = Column(Integer)

    backtest = relationship("Backtest", back_populates="results")

class Stock(Base):
    __tablename__ = "stocks"

    id = Column(Integer, primary_key=True, index=True)
    symbol = Column(String, unique=True, index=True)
    name = Column(String)
    market = Column(String)  # A-share, HK, US

    prices = relationship("StockPrice", back_populates="stock")

class StockPrice(Base):
    __tablename__ = "stock_prices"

    id = Column(Integer, primary_key=True, index=True)
    stock_id = Column(Integer, ForeignKey("stocks.id"))
    date = Column(DateTime, index=True)
    open = Column(Float)
    high = Column(Float)
    low = Column(Float)
    close = Column(Float)
    volume = Column(Integer)

    stock = relationship("Stock", back_populates="prices")

class Trade(Base):
    __tablename__ = "trades"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    symbol = Column(String)
    trade_type = Column(String)  # buy, sell
    quantity = Column(Integer)
    price = Column(Float)
    timestamp = Column(DateTime, server_default=func.now())
    status = Column(String)  # pending, completed, cancelled

    user = relationship("User", back_populates="trades")
```

### 任务3：创建Repository层（1.5小时）

创建 `app/repository/base.py`：
```python
from sqlalchemy.orm import Session
from typing import TypeVar, Generic, List, Optional

T = TypeVar('T')

class BaseRepository(Generic[T]):
    def __init__(self, db: Session, model: T):
        self.db = db
        self.model = model

    def create(self, obj_in: dict) -> T:
        db_obj = self.model(**obj_in)
        self.db.add(db_obj)
        self.db.commit()
        self.db.refresh(db_obj)
        return db_obj

    def get(self, id: int) -> Optional[T]:
        return self.db.query(self.model).filter(self.model.id == id).first()

    def list(self, skip: int = 0, limit: int = 100) -> List[T]:
        return self.db.query(self.model).offset(skip).limit(limit).all()

    def update(self, id: int, obj_in: dict) -> Optional[T]:
        db_obj = self.get(id)
        if db_obj:
            for key, value in obj_in.items():
                setattr(db_obj, key, value)
            self.db.commit()
            self.db.refresh(db_obj)
        return db_obj

    def delete(self, id: int) -> bool:
        db_obj = self.get(id)
        if db_obj:
            self.db.delete(db_obj)
            self.db.commit()
            return True
        return False
```

创建 `app/repository/user.py`：
```python
from sqlalchemy.orm import Session
from app.models.models import User
from .base import BaseRepository

class UserRepository(BaseRepository[User]):
    def __init__(self, db: Session):
        super().__init__(db, User)

    def get_by_username(self, username: str) -> User:
        return self.db.query(User).filter(User.username == username).first()

    def get_by_email(self, email: str) -> User:
        return self.db.query(User).filter(User.email == email).first()
```

### 任务4：数据库初始化脚本（1小时）

创建 `app/core/init_db.py`：
```python
from sqlalchemy import inspect
from app.core.database import engine, Base
from app.models import models

def init_db():
    # 检查表是否存在
    inspector = inspect(engine)
    existing_tables = inspector.get_table_names()

    if not existing_tables:
        print("Creating database tables...")
        Base.metadata.create_all(bind=engine)
        print("Database tables created successfully!")
    else:
        print("Database tables already exist")

if __name__ == "__main__":
    init_db()
```

更新 `main.py`：
```python
from app.core.init_db import init_db

def create_app():
    app = FastAPI(...)

    @app.on_event("startup")
    async def startup():
        init_db()

    return app
```

## 检查清单（Day 3结束时）

- [ ] SQLAlchemy已安装和配置
- [ ] 数据库连接池已实现
- [ ] 所有核心模型已定义
- [ ] Repository基类已实现
- [ ] 至少一个具体Repository已实现
- [ ] 数据库初始化脚本已完成
- [ ] 能成功创建数据库表

## 环境配置示例（.env）

```
DATABASE_URL=postgresql://qbot:qbot@localhost:5432/qbot
SQLALCHEMY_ECHO=False
POOL_SIZE=10
MAX_OVERFLOW=20
```

## 学习资源

- SQLAlchemy ORM官方文档：https://docs.sqlalchemy.org/en/20/orm/
- FastAPI数据库集成：https://fastapi.tiangolo.com/tutorial/sql-databases/
- Alembic迁移：https://alembic.sqlalchemy.org/

## 常见问题

**Q: 为什么使用PostgreSQL而不是SQLite？**
A: PostgreSQL适合生产环境，支持连接池、高并发、强大的功能。SQLite只适合学习和测试。

**Q: 急加载和懒加载有什么区别？**
A: 急加载一次性加载所有关联数据，懒加载按需加载。懒加载更灵活但可能导致N+1查询问题。

**Q: 如何处理数据库迁移？**
A: Day 4会详细讲解Alembic迁移工具。

## 下一步预告

Day 4将学习：
- Pydantic Schemas设计
- 数据序列化和反序列化
- API请求/响应模型
- 基础CRUD API实现

