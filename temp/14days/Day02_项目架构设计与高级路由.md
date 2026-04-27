# Day 2: 项目架构设计与高级路由

## 学习目标
- ✅ 掌握FastAPI路由组织和API分组
- ✅ 理解中间件和异常处理机制
- ✅ 设计Qbot后端的完整架构
- ✅ 实现全局错误处理和日志系统
- ✅ API版本控制策略

## 核心学习内容

### 1. 路由组织和分组
- APIRouter的使用
- 路由前缀和标签
- 路由依赖注入
- 路由组织最佳实践

### 2. 中间件（Middleware）
- 什么是中间件
- 常见中间件类型
- 自定义中间件开发
- 中间件执行顺序

### 3. 异常处理
- HTTPException基础
- 自定义异常类
- 全局异常处理器
- 错误响应格式标准化

### 4. 依赖注入系统
- Depends的深入理解
- 构建可复用的依赖
- 依赖提供者（Provider）
- 依赖链的管理

### 5. API版本控制
- URL路径版本化（/api/v1/, /api/v2/）
- 头部版本化
- 查询参数版本化
- 版本管理最佳实践

## Qbot后端架构设计

```
应用层（Routes）
    ↓
业务逻辑层（Services）
    ↓
数据访问层（Repository）
    ↓
模型层（Models）
    ↓
数据库层（Database）
```

### 核心模块划分

- **User模块**：用户管理、认证
- **Stock模块**：股票数据获取、查询
- **Strategy模块**：策略CRUD、管理
- **Backtest模块**：回测任务、结果
- **Trade模块**：交易订单、执行
- **Data模块**：行情数据、因子计算

## 关键技术点

1. **路由结构**
   ```
   /api/v1/
   ├── /users       - 用户相关
   ├── /stocks      - 股票数据
   ├── /strategies  - 策略管理
   ├── /backtest    - 回测系统
   ├── /trade       - 交易管理
   └── /data        - 数据服务
   ```

2. **错误处理标准**
   ```json
   {
     "code": 400,
     "message": "错误描述",
     "data": null,
     "timestamp": "2024-04-27T10:00:00Z"
   }
   ```

3. **依赖注入模式**
   - 请求级依赖
   - 路由级依赖
   - 应用级依赖

## 实践任务

### 任务1：创建路由层结构（1.5小时）

创建 `app/routes/__init__.py`：
```python
from fastapi import APIRouter
from .stocks import router as stocks_router
from .strategies import router as strategies_router
from .backtest import router as backtest_router
from .trade import router as trade_router

def create_routes():
    api_router = APIRouter(prefix="/api/v1")

    api_router.include_router(stocks_router, prefix="/stocks", tags=["Stocks"])
    api_router.include_router(strategies_router, prefix="/strategies", tags=["Strategies"])
    api_router.include_router(backtest_router, prefix="/backtest", tags=["Backtest"])
    api_router.include_router(trade_router, prefix="/trade", tags=["Trade"])

    return api_router
```

创建 `app/routes/stocks.py`（示例）：
```python
from fastapi import APIRouter, Query
from typing import Optional

router = APIRouter()

@router.get("/")
async def list_stocks(
    symbol: Optional[str] = Query(None),
    limit: int = Query(10, ge=1, le=100)
):
    return {"stocks": [], "total": 0}

@router.get("/{symbol}")
async def get_stock(symbol: str):
    return {"symbol": symbol, "data": {}}

@router.post("/")
async def create_stock(data: dict):
    return {"created": True, "id": 1}
```

创建其他路由文件（strategies.py, backtest.py, trade.py）为空框架

### 任务2：中间件和异常处理（1.5小时）

创建 `app/middleware.py`：
```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
import time
import logging

logger = logging.getLogger(__name__)

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time

        logger.info(
            f"{request.method} {request.url.path} "
            f"Status: {response.status_code} "
            f"Duration: {process_time:.3f}s"
        )
        return response
```

创建 `app/exceptions.py`：
```python
class QbotException(Exception):
    def __init__(self, code: int, message: str, data=None):
        self.code = code
        self.message = message
        self.data = data

class StockNotFoundError(QbotException):
    def __init__(self, symbol: str):
        super().__init__(404, f"Stock {symbol} not found")

class InvalidStrategyError(QbotException):
    def __init__(self, reason: str):
        super().__init__(400, f"Invalid strategy: {reason}")
```

### 任务3：更新主应用文件（1小时）

更新 `app/main.py`：
```python
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from .middleware import LoggingMiddleware
from .exceptions import QbotException
from .routes import create_routes
from app.models.base import ErrorResponse

def create_app():
    app = FastAPI(
        title="Qbot Trading API",
        version="0.1.0"
    )

    # 添加中间件
    app.add_middleware(LoggingMiddleware)

    # 异常处理
    @app.exception_handler(QbotException)
    async def qbot_exception_handler(request, exc):
        return {
            "code": exc.code,
            "message": exc.message,
            "data": exc.data
        }

    # 包含路由
    app.include_router(create_routes())

    return app

app = create_app()
```

### 任务4：创建基础模型（1小时）

创建 `app/models/base.py`：
```python
from pydantic import BaseModel
from typing import Optional, Generic, TypeVar
from datetime import datetime

T = TypeVar('T')

class ResponseModel(BaseModel, Generic[T]):
    code: int = 200
    message: str = "Success"
    data: Optional[T] = None
    timestamp: datetime = None

class PaginationModel(BaseModel):
    page: int = 1
    limit: int = 10
    total: int = 0

class StockData(BaseModel):
    symbol: str
    name: str
    price: float
    change: float

class StrategyData(BaseModel):
    id: int
    name: str
    description: str
    status: str  # active, inactive, backtesting
```

## 检查清单（Day 2结束时）

- [ ] 路由组织结构已建立
- [ ] APIRouter已应用于模块分组
- [ ] 中间件系统已实现
- [ ] 异常处理框架已完成
- [ ] 基础模型已定义
- [ ] API文档中显示正确的分组和标签
- [ ] 理解依赖注入系统

## 代码规范

1. **命名规范**
   - 路由文件：`{module}.py`
   - 类名：PascalCase
   - 函数名：snake_case
   - 常量名：UPPER_SNAKE_CASE

2. **文档要求**
   - 每个路由需要docstring
   - 参数需要描述
   - 返回值需要示例

示例：
```python
@router.get("/{symbol}")
async def get_stock_by_symbol(symbol: str):
    """
    获取指定代码的股票数据

    - **symbol**: 股票代码，例如 600000

    返回股票的基本信息和最新价格
    """
    pass
```

## 学习资源

- FastAPI路由官方文档：https://fastapi.tiangolo.com/tutorial/bigger-applications/
- 中间件信息：https://fastapi.tiangolo.com/tutorial/middleware/
- 异常处理：https://fastapi.tiangolo.com/tutorial/handling-errors/

## 常见问题

**Q: 为什么要分离Service和Repository层？**
A: 关注点分离，Repository处理数据访问，Service处理业务逻辑，便于测试和维护。

**Q: 中间件执行顺序是什么？**
A: 后添加的中间件先执行（栈结构），响应时顺序相反。

**Q: 如何处理跨域（CORS）？**
A: Day 3会详细讲解，现在知道FastAPI内置支持即可。

## 下一步预告

Day 3将学习：
- 数据库集成（SQLAlchemy）
- 模型设计（ORM）
- 数据验证和序列化
- 数据库初始化和迁移

