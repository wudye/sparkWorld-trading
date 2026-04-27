# Day 4: API数据模型设计与CRUD操作

## 学习目标
- ✅ 掌握Pydantic Schema设计
- ✅ 实现请求/响应数据模型
- ✅ 创建通用CRUD API模式
- ✅ 学习数据验证和错误处理
- ✅ 集成数据库操作到API路由

## 核心学习内容

### 1. Pydantic Schema设计
- 与SQLAlchemy模型的区别
- Schema的嵌套和关系
- 字段验证（Validators）
- 配置类（Config）

### 2. CRUD操作模式
- Create（创建）
- Read（读取）
- Update（更新）
- Delete（删除）
- List with Pagination（列表分页）

### 3. 数据验证
- 字段级验证（Field validators）
- 模型级验证（Model validators）
- 自定义验证器
- 错误响应格式

### 4. 关系处理
- 嵌套模型
- 循环引用处理
- 深度控制

## 关键技术点

1. **模式分离**
   ```python
   # 基础模式（用于列表）
   UserBasicSchema

   # 创建模式（POST请求）
   UserCreateSchema

   # 更新模式（PUT/PATCH请求）
   UserUpdateSchema

   # 响应模式（API返回）
   UserSchema
   ```

2. **验证示例**
   ```python
   from pydantic import BaseModel, Field, validator

   class UserCreateSchema(BaseModel):
       username: str = Field(..., min_length=3, max_length=50)
       email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$")

       @validator('username')
       def validate_username(cls, v):
           if not v.isalnum():
               raise ValueError('Username must be alphanumeric')
           return v
   ```

3. **分页模式**
   ```python
   class PaginationSchema(BaseModel):
       page: int = Field(1, gt=0)
       limit: int = Field(10, gt=0, le=100)

   class PaginatedResponse(BaseModel):
       data: List[T]
       total: int
       page: int
       limit: int
   ```

## 实践任务

### 任务1：设计Pydantic Schemas（1.5小时）

创建 `app/schemas/user.py`：
```python
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from datetime import datetime

class UserBase(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    username: Optional[str] = None
    email: Optional[EmailStr] = None

class User(UserBase):
    id: int
    is_active: bool
    created_at: datetime
    updated_at: Optional[datetime] = None

    class Config:
        from_attributes = True  # 支持从ORM对象创建
```

创建 `app/schemas/strategy.py`：
```python
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

class StrategyParameterBase(BaseModel):
    param_name: str
    param_value: str
    param_type: str  # int, float, string

class StrategyBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = None
    code: str

class StrategyCreate(StrategyBase):
    parameters: Optional[List[StrategyParameterBase]] = []

class StrategyUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    code: Optional[str] = None
    status: Optional[str] = None

class Strategy(StrategyBase):
    id: int
    user_id: int
    status: str
    version: int
    created_at: datetime

    class Config:
        from_attributes = True
```

创建 `app/schemas/__init__.py`：
```python
from .user import User, UserCreate, UserUpdate
from .strategy import Strategy, StrategyCreate, StrategyUpdate
from .common import PaginationParams, PaginatedResponse

__all__ = [...]
```

创建 `app/schemas/common.py`：
```python
from pydantic import BaseModel, Field
from typing import TypeVar, Generic, List, Optional

T = TypeVar('T')

class PaginationParams(BaseModel):
    page: int = Field(1, ge=1)
    limit: int = Field(10, ge=1, le=100)

class PaginatedResponse(BaseModel, Generic[T]):
    data: List[T]
    total: int
    page: int
    limit: int
    pages: int

    def __init__(self, data: List[T], total: int, page: int, limit: int):
        pages = (total + limit - 1) // limit
        super().__init__(data=data, total=total, page=page, limit=limit, pages=pages)
```

### 任务2：创建Service层（1.5小时）

创建 `app/services/user_service.py`：
```python
from sqlalchemy.orm import Session
from app.models.models import User
from app.repository.user import UserRepository
from app.schemas.user import UserCreate, UserUpdate
from app.exceptions import QbotException
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class UserService:
    def __init__(self, db: Session):
        self.repository = UserRepository(db)

    def create_user(self, user_in: UserCreate) -> User:
        # 检查用户是否已存在
        existing = self.repository.get_by_username(user_in.username)
        if existing:
            raise QbotException(400, "Username already exists")

        # 密码哈希
        hashed_password = pwd_context.hash(user_in.password)

        db_obj = {
            "username": user_in.username,
            "email": user_in.email,
            "password_hash": hashed_password
        }
        return self.repository.create(db_obj)

    def get_user(self, user_id: int) -> User:
        user = self.repository.get(user_id)
        if not user:
            raise QbotException(404, "User not found")
        return user

    def list_users(self, skip: int = 0, limit: int = 10):
        return self.repository.list(skip, limit)

    def update_user(self, user_id: int, user_in: UserUpdate) -> User:
        user = self.get_user(user_id)
        update_data = user_in.dict(exclude_unset=True)
        return self.repository.update(user_id, update_data)

    def delete_user(self, user_id: int) -> bool:
        return self.repository.delete(user_id)
```

创建 `app/services/strategy_service.py`：
```python
from sqlalchemy.orm import Session
from app.models.models import Strategy
from app.schemas.strategy import StrategyCreate, StrategyUpdate
from app.repository.base import BaseRepository
from app.exceptions import QbotException

class StrategyService:
    def __init__(self, db: Session):
        self.db = db
        self.repository = BaseRepository(db, Strategy)

    def create_strategy(self, user_id: int, strategy_in: StrategyCreate) -> Strategy:
        db_obj = {
            "user_id": user_id,
            "name": strategy_in.name,
            "description": strategy_in.description,
            "code": strategy_in.code,
            "status": "inactive"
        }
        return self.repository.create(db_obj)

    def get_strategy(self, strategy_id: int) -> Strategy:
        strategy = self.repository.get(strategy_id)
        if not strategy:
            raise QbotException(404, "Strategy not found")
        return strategy

    def list_user_strategies(self, user_id: int, skip: int = 0, limit: int = 10):
        return self.db.query(Strategy)\
            .filter(Strategy.user_id == user_id)\
            .offset(skip)\
            .limit(limit)\
            .all()

    def update_strategy(self, strategy_id: int, strategy_in: StrategyUpdate) -> Strategy:
        strategy = self.get_strategy(strategy_id)
        update_data = strategy_in.dict(exclude_unset=True)
        return self.repository.update(strategy_id, update_data)

    def delete_strategy(self, strategy_id: int) -> bool:
        return self.repository.delete(strategy_id)
```

### 任务3：实现CRUD API路由（1.5小时）

创建 `app/routes/users.py`：
```python
from fastapi import APIRouter, Depends, Query
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.user import User, UserCreate, UserUpdate
from app.services.user_service import UserService
from typing import List

router = APIRouter()

def get_user_service(db: Session = Depends(get_db)) -> UserService:
    return UserService(db)

@router.post("/", response_model=User)
async def create_user(
    user_in: UserCreate,
    service: UserService = Depends(get_user_service)
):
    """创建新用户"""
    return service.create_user(user_in)

@router.get("/{user_id}", response_model=User)
async def get_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
):
    """获取用户详情"""
    return service.get_user(user_id)

@router.get("/", response_model=List[User])
async def list_users(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
    service: UserService = Depends(get_user_service)
):
    """获取用户列表"""
    return service.list_users(skip, limit)

@router.put("/{user_id}", response_model=User)
async def update_user(
    user_id: int,
    user_in: UserUpdate,
    service: UserService = Depends(get_user_service)
):
    """更新用户信息"""
    return service.update_user(user_id, user_in)

@router.delete("/{user_id}")
async def delete_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
):
    """删除用户"""
    success = service.delete_user(user_id)
    return {"deleted": success}
```

创建 `app/routes/strategies.py`：
```python
from fastapi import APIRouter, Depends, Query, Path
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.strategy import Strategy, StrategyCreate, StrategyUpdate
from app.services.strategy_service import StrategyService
from typing import List

router = APIRouter()

def get_strategy_service(db: Session = Depends(get_db)) -> StrategyService:
    return StrategyService(db)

@router.post("/", response_model=Strategy)
async def create_strategy(
    strategy_in: StrategyCreate,
    user_id: int = Query(..., ge=1),
    service: StrategyService = Depends(get_strategy_service)
):
    """创建新策略"""
    return service.create_strategy(user_id, strategy_in)

@router.get("/{strategy_id}", response_model=Strategy)
async def get_strategy(
    strategy_id: int = Path(..., ge=1),
    service: StrategyService = Depends(get_strategy_service)
):
    """获取策略详情"""
    return service.get_strategy(strategy_id)

@router.get("/user/{user_id}", response_model=List[Strategy])
async def list_strategies(
    user_id: int = Path(..., ge=1),
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
    service: StrategyService = Depends(get_strategy_service)
):
    """获取用户的所有策略"""
    return service.list_user_strategies(user_id, skip, limit)

@router.put("/{strategy_id}", response_model=Strategy)
async def update_strategy(
    strategy_id: int = Path(..., ge=1),
    strategy_in: StrategyUpdate = None,
    service: StrategyService = Depends(get_strategy_service)
):
    """更新策略"""
    return service.update_strategy(strategy_id, strategy_in)

@router.delete("/{strategy_id}")
async def delete_strategy(
    strategy_id: int = Path(..., ge=1),
    service: StrategyService = Depends(get_strategy_service)
):
    """删除策略"""
    success = service.delete_strategy(strategy_id)
    return {"deleted": success}
```

## 检查清单（Day 4结束时）

- [ ] Pydantic Schemas已为主要实体定义
- [ ] Service层已实现
- [ ] CRUD API路由已完成
- [ ] 数据验证工作正常
- [ ] API文档显示正确的请求/响应模型
- [ ] 能成功通过Swagger UI测试

## 测试示例

使用Swagger UI测试：
1. POST /users 创建用户
2. GET /users/{user_id} 获取用户
3. PUT /users/{user_id} 更新用户
4. DELETE /users/{user_id} 删除用户

## 常见问题

**Q: `from_attributes = True` 是什么意思？**
A: 允许Pydantic模型从SQLAlchemy ORM对象创建，即使属性不是直接的dict。

**Q: 为什么要分离Service和Repository？**
A: Repository处理数据访问，Service处理业务逻辑，便于单元测试。

**Q: 如何处理数据库事务？**
A: FastAPI会在请求结束时自动提交或回滚，依赖于是否有异常。

## 下一步预告

Day 5将学习：
- API版本控制和兼容性
- 身份验证和授权（JWT）
- 请求限流和缓存
- 高级查询（过滤、排序、搜索）

