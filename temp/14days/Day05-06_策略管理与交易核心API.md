# Day 5-6: 策略管理与交易核心API

## 学习目标（Day 5）
- ✅ 设计和实现策略CRUD完整API
- ✅ 实现策略参数管理
- ✅ 设计策略版本控制
- ✅ 实现策略验证和激活
- ✅ 创建策略执行引擎API基础

## 学习目标（Day 6）
- ✅ 设计交易数据模型
- ✅ 实现订单管理API
- ✅ 实现持仓管理API
- ✅ 设计交易执行流程
- ✅ 实现风险控制接口

## Day 5: 策略管理系统

### 核心学习内容

1. **策略生命周期管理**
   ```
   Draft(草稿) → Validated(已验证) → Active(激活) → Backtesting(回测中) → Completed(已完成)
   ```

2. **策略版本控制**
   - 版本记录
   - 版本回滚
   - 版本对比

3. **策略参数化**
   - 参数定义
   - 参数验证
   - 参数组合测试

4. **策略验证**
   - 代码语法检查
   - 必需指标验证
   - 集成测试

### 实践任务（Day 5）

#### 任务1：扩展策略模型和Schema（1小时）

创建 `app/models/models.py` 追加：
```python
class StrategyVersion(Base):
    __tablename__ = "strategy_versions"

    id = Column(Integer, primary_key=True, index=True)
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    version_number = Column(Integer)
    code = Column(Text)
    description = Column(Text)
    created_at = Column(DateTime, server_default=func.now())
    created_by = Column(String)  # 变更者
    is_active = Column(Boolean, default=False)

    strategy = relationship("Strategy", foreign_keys=[strategy_id])

class StrategyBacktest(Base):
    __tablename__ = "strategy_backtests"

    id = Column(Integer, primary_key=True, index=True)
    strategy_id = Column(Integer, ForeignKey("strategies.id"))
    start_date = Column(DateTime)
    end_date = Column(DateTime)
    status = Column(String)  # pending, running, completed, failed
    progress = Column(Float, default=0.0)
    created_at = Column(DateTime, server_default=func.now())
```

创建 `app/schemas/strategy_extended.py`：
```python
from pydantic import BaseModel, Field
from typing import Optional, List, Dict
from datetime import datetime

class StrategyParameter(BaseModel):
    param_name: str
    param_value: str
    param_type: str
    description: Optional[str] = None

class StrategyValidationRequest(BaseModel):
    code: str
    parameters: List[StrategyParameter]

class StrategyValidationResult(BaseModel):
    is_valid: bool
    errors: List[str] = []
    warnings: List[str] = []
    required_indicators: List[str] = []

class StrategyVersion(BaseModel):
    id: int
    version_number: int
    code: str
    description: Optional[str] = None
    created_at: datetime
    created_by: str
    is_active: bool

class StrategyActivationRequest(BaseModel):
    parameters: List[StrategyParameter]

class StrategyExecutionConfig(BaseModel):
    strategy_id: int
    parameters: Dict[str, any]
    execution_type: str  # backtest, simulation, live
```

#### 任务2：实现策略验证服务（1.5小时）

创建 `app/services/strategy_validation.py`：
```python
import ast
import re
from typing import List

class StrategyValidator:
    # 必需的指标
    REQUIRED_INDICATORS = {
        'sma', 'ema', 'rsi', 'macd', 'bollinger'
    }

    # 禁用的操作
    FORBIDDEN_OPERATIONS = {
        'os', 'subprocess', 'eval', '__import__',
        'exec', 'open', 'write'
    }

    @staticmethod
    def validate_strategy_code(code: str) -> tuple[bool, List[str], List[str]]:
        """
        验证策略代码的安全性和有效性
        返回: (是否有效, 错误列表, 警告列表)
        """
        errors = []
        warnings = []

        try:
            ast.parse(code)
        except SyntaxError as e:
            errors.append(f"Syntax error: {e}")
            return False, errors, warnings

        # 检查禁用操作
        for forbidden in StrategyValidator.FORBIDDEN_OPERATIONS:
            if forbidden in code:
                errors.append(f"Forbidden operation: {forbidden}")

        # 检查必需指标
        found_indicators = set()
        for indicator in StrategyValidator.REQUIRED_INDICATORS:
            if re.search(rf'\b{indicator}\b', code, re.IGNORECASE):
                found_indicators.add(indicator)

        missing_indicators = StrategyValidator.REQUIRED_INDICATORS - found_indicators
        if missing_indicators:
            warnings.append(f"Missing indicators: {missing_indicators}")

        # 检查代码长度
        if len(code) > 10000:
            warnings.append("Code is quite long, may affect performance")

        return len(errors) == 0, errors, warnings

class StrategyService:
    def __init__(self, db):
        self.db = db
        self.validator = StrategyValidator()

    def validate_strategy(self, strategy_code: str, parameters: List[dict]):
        """验证策略代码和参数"""
        is_valid, errors, warnings = self.validator.validate_strategy_code(strategy_code)

        # 验证参数
        param_errors = self._validate_parameters(parameters)
        errors.extend(param_errors)

        return {
            "is_valid": len(errors) == 0,
            "errors": errors,
            "warnings": warnings,
            "required_indicators": list(self.validator.REQUIRED_INDICATORS)
        }

    @staticmethod
    def _validate_parameters(parameters: List[dict]) -> List[str]:
        """验证策略参数"""
        errors = []

        for param in parameters:
            if not param.get('param_name'):
                errors.append("Parameter name is required")
            if param.get('param_type') not in ['int', 'float', 'string', 'bool']:
                errors.append(f"Invalid parameter type: {param.get('param_type')}")

        return errors

    def create_strategy_version(self, strategy_id: int, code: str, description: str):
        """创建策略版本"""
        from app.models.models import Strategy, StrategyVersion

        strategy = self.db.query(Strategy).filter(Strategy.id == strategy_id).first()
        if not strategy:
            raise Exception("Strategy not found")

        # 获取下一个版本号
        last_version = self.db.query(StrategyVersion)\
            .filter(StrategyVersion.strategy_id == strategy_id)\
            .order_by(StrategyVersion.version_number.desc())\
            .first()

        next_version = (last_version.version_number + 1) if last_version else 1

        new_version = StrategyVersion(
            strategy_id=strategy_id,
            version_number=next_version,
            code=code,
            description=description,
            created_by="system"
        )

        self.db.add(new_version)
        self.db.commit()

        return new_version
```

#### 任务3：策略管理API路由（1.5小时）

创建 `app/routes/strategies_extended.py`：
```python
from fastapi import APIRouter, Depends, Query, Path, HTTPException
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.strategy_extended import (
    StrategyValidationRequest,
    StrategyValidationResult,
    StrategyActivationRequest
)
from app.services.strategy_validation import StrategyService
from typing import List

router = APIRouter()

def get_strategy_service(db: Session = Depends(get_db)) -> StrategyService:
    return StrategyService(db)

@router.post("/validate", response_model=StrategyValidationResult)
async def validate_strategy(
    request: StrategyValidationRequest,
    service: StrategyService = Depends(get_strategy_service)
):
    """验证策略代码的有效性和安全性"""
    return service.validate_strategy(request.code, request.parameters)

@router.post("/{strategy_id}/versions")
async def create_strategy_version(
    strategy_id: int = Path(..., ge=1),
    code: str = Query(...),
    description: str = Query(""),
    service: StrategyService = Depends(get_strategy_service)
):
    """创建策略新版本"""
    try:
        version = service.create_strategy_version(strategy_id, code, description)
        return {"version_number": version.version_number, "created_at": version.created_at}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.post("/{strategy_id}/activate")
async def activate_strategy(
    strategy_id: int = Path(..., ge=1),
    request: StrategyActivationRequest = None,
    service: StrategyService = Depends(get_strategy_service)
):
    """激活策略"""
    # 验证参数
    validation_result = service.validate_strategy("", request.parameters)
    if not validation_result["is_valid"]:
        raise HTTPException(status_code=400, detail=validation_result["errors"])

    return {"activated": True, "strategy_id": strategy_id}

@router.get("/{strategy_id}/versions", response_model=List[dict])
async def list_strategy_versions(
    strategy_id: int = Path(..., ge=1),
    db: Session = Depends(get_db)
):
    """获取策略的所有版本"""
    from app.models.models import StrategyVersion

    versions = db.query(StrategyVersion)\
        .filter(StrategyVersion.strategy_id == strategy_id)\
        .order_by(StrategyVersion.version_number.desc())\
        .all()

    return [{"version_number": v.version_number, "created_at": v.created_at} for v in versions]

@router.post("/{strategy_id}/rollback/{version}")
async def rollback_strategy(
    strategy_id: int = Path(..., ge=1),
    version: int = Path(..., ge=1),
    db: Session = Depends(get_db)
):
    """回滚到指定版本"""
    from app.models.models import Strategy, StrategyVersion

    # 查找版本
    version_obj = db.query(StrategyVersion)\
        .filter(StrategyVersion.strategy_id == strategy_id,
                StrategyVersion.version_number == version)\
        .first()

    if not version_obj:
        raise HTTPException(status_code=404, detail="Version not found")

    # 更新策略
    strategy = db.query(Strategy).filter(Strategy.id == strategy_id).first()
    strategy.code = version_obj.code
    strategy.version = version
    db.commit()

    return {"rolled_back": True, "version": version}
```

---

## Day 6: 交易核心系统

### 核心学习内容

1. **订单管理**
   - 订单类型（市价、限价、条件单）
   - 订单状态（待成交、已成交、已取消、已拒绝）
   - 订单生命周期

2. **持仓管理**
   - 持仓记录
   - 成本计算
   - 浮动盈亏
   - 已实现盈亏

3. **交易执行**
   - 订单提交
   - 订单成交
   - 订单撤销

4. **风险控制**
   - 止损设置
   - 止盈设置
   - 仓位管理
   - 资金管理

### 实践任务（Day 6）

#### 任务1：交易模型扩展（1小时）

创建 `app/models/trade_models.py`：
```python
from sqlalchemy import Column, Integer, String, Float, DateTime, Boolean, ForeignKey, Enum
from app.core.database import Base
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
import enum

class OrderType(str, enum.Enum):
    MARKET = "market"
    LIMIT = "limit"
    CONDITIONAL = "conditional"

class OrderStatus(str, enum.Enum):
    PENDING = "pending"
    PARTIAL = "partial"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
    REJECTED = "rejected"

class Order(Base):
    __tablename__ = "orders"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    strategy_id = Column(Integer, ForeignKey("strategies.id"), nullable=True)
    symbol = Column(String, index=True)
    order_type = Column(String)  # market, limit, conditional
    side = Column(String)  # buy, sell
    quantity = Column(Float)
    filled_quantity = Column(Float, default=0)
    price = Column(Float)
    current_price = Column(Float)
    status = Column(String)
    created_at = Column(DateTime, server_default=func.now())
    filled_at = Column(DateTime, nullable=True)

    user = relationship("User")
    strategy = relationship("Strategy")

class Position(Base):
    __tablename__ = "positions"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    symbol = Column(String, index=True)
    quantity = Column(Float)
    average_cost = Column(Float)
    current_price = Column(Float)
    market_value = Column(Float)
    cost_basis = Column(Float)  # 总成本
    unrealized_pnl = Column(Float)  # 未实现盈亏
    realized_pnl = Column(Float, default=0)  # 已实现盈亏
    updated_at = Column(DateTime, server_default=func.now(), onupdate=func.now())

    user = relationship("User")

class RiskRule(Base):
    __tablename__ = "risk_rules"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    symbol = Column(String)
    stop_loss = Column(Float, nullable=True)  # 止损价格
    take_profit = Column(Float, nullable=True)  # 止盈价格
    max_position = Column(Float, nullable=True)  # 最大持仓比例
    max_daily_loss = Column(Float, nullable=True)  # 最大日亏损
```

#### 任务2：交易服务实现（1.5小时）

创建 `app/services/trade_service.py`：
```python
from sqlalchemy.orm import Session
from app.models.trade_models import Order, Position, RiskRule, OrderStatus, OrderType
from typing import List, Optional
from decimal import Decimal

class TradeService:
    def __init__(self, db: Session):
        self.db = db

    def create_order(self, user_id: int, symbol: str, side: str,
                     quantity: float, price: float, order_type: str) -> Order:
        """创建订单"""
        # 检查风险规则
        risk_rule = self.db.query(RiskRule)\
            .filter(RiskRule.user_id == user_id, RiskRule.symbol == symbol)\
            .first()

        order = Order(
            user_id=user_id,
            symbol=symbol,
            side=side,
            quantity=quantity,
            price=price,
            order_type=order_type,
            status=OrderStatus.PENDING.value,
            current_price=price
        )

        self.db.add(order)
        self.db.commit()
        self.db.refresh(order)

        return order

    def execute_order(self, order_id: int, filled_price: float, filled_quantity: float):
        """执行订单"""
        order = self.db.query(Order).filter(Order.id == order_id).first()
        if not order:
            raise Exception("Order not found")

        order.filled_quantity = filled_quantity
        order.current_price = filled_price

        if filled_quantity >= order.quantity:
            order.status = OrderStatus.COMPLETED.value
        else:
            order.status = OrderStatus.PARTIAL.value

        # 更新持仓
        self._update_position(order)

        self.db.commit()

    def _update_position(self, order: Order):
        """更新持仓"""
        position = self.db.query(Position)\
            .filter(Position.user_id == order.user_id, Position.symbol == order.symbol)\
            .first()

        if not position:
            position = Position(
                user_id=order.user_id,
                symbol=order.symbol,
                quantity=0,
                average_cost=0,
                cost_basis=0
            )
            self.db.add(position)

        if order.side == "buy":
            # 买入：更新平均成本
            total_cost = position.cost_basis + (order.filled_quantity * order.current_price)
            total_qty = position.quantity + order.filled_quantity
            position.average_cost = total_cost / total_qty if total_qty > 0 else 0
            position.quantity = total_qty
            position.cost_basis = total_cost
        else:
            # 卖出：计算已实现盈亏
            realized_gain = (order.current_price - position.average_cost) * order.filled_quantity
            position.realized_pnl += realized_gain
            position.quantity -= order.filled_quantity
            position.cost_basis -= order.filled_quantity * position.average_cost

        position.market_value = position.quantity * order.current_price
        position.unrealized_pnl = position.market_value - position.cost_basis

        self.db.commit()

    def get_position(self, user_id: int, symbol: str) -> Optional[Position]:
        """获取持仓"""
        return self.db.query(Position)\
            .filter(Position.user_id == user_id, Position.symbol == symbol)\
            .first()

    def list_positions(self, user_id: int) -> List[Position]:
        """获取所有持仓"""
        return self.db.query(Position)\
            .filter(Position.user_id == user_id, Position.quantity > 0)\
            .all()

    def list_orders(self, user_id: int, status: Optional[str] = None) -> List[Order]:
        """获取订单列表"""
        query = self.db.query(Order).filter(Order.user_id == user_id)
        if status:
            query = query.filter(Order.status == status)
        return query.all()

    def cancel_order(self, order_id: int) -> bool:
        """取消订单"""
        order = self.db.query(Order).filter(Order.id == order_id).first()
        if not order:
            return False

        if order.status in [OrderStatus.COMPLETED.value, OrderStatus.CANCELLED.value]:
            return False

        order.status = OrderStatus.CANCELLED.value
        self.db.commit()
        return True
```

#### 任务3：交易API路由（1.5小时）

创建 `app/routes/trade.py`：
```python
from fastapi import APIRouter, Depends, Query, Path
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.services.trade_service import TradeService
from pydantic import BaseModel
from typing import List, Optional

router = APIRouter()

class OrderRequest(BaseModel):
    symbol: str
    side: str  # buy, sell
    quantity: float
    price: float
    order_type: str = "market"

class PositionResponse(BaseModel):
    id: int
    symbol: str
    quantity: float
    average_cost: float
    current_price: float
    unrealized_pnl: float
    realized_pnl: float

def get_trade_service(db: Session = Depends(get_db)) -> TradeService:
    return TradeService(db)

@router.post("/orders")
async def create_order(
    user_id: int = Query(..., ge=1),
    request: OrderRequest = None,
    service: TradeService = Depends(get_trade_service)
):
    """创建订单"""
    order = service.create_order(
        user_id, request.symbol, request.side,
        request.quantity, request.price, request.order_type
    )
    return {"order_id": order.id, "status": order.status}

@router.get("/orders")
async def list_orders(
    user_id: int = Query(..., ge=1),
    status: Optional[str] = Query(None),
    service: TradeService = Depends(get_trade_service)
):
    """获取订单列表"""
    orders = service.list_orders(user_id, status)
    return {"orders": [{"id": o.id, "symbol": o.symbol, "status": o.status} for o in orders]}

@router.post("/orders/{order_id}/cancel")
async def cancel_order(
    order_id: int = Path(..., ge=1),
    service: TradeService = Depends(get_trade_service)
):
    """取消订单"""
    success = service.cancel_order(order_id)
    return {"cancelled": success}

@router.get("/positions")
async def list_positions(
    user_id: int = Query(..., ge=1),
    service: TradeService = Depends(get_trade_service)
):
    """获取所有持仓"""
    positions = service.list_positions(user_id)
    return {
        "positions": [
            {
                "symbol": p.symbol,
                "quantity": p.quantity,
                "average_cost": p.average_cost,
                "market_value": p.market_value,
                "unrealized_pnl": p.unrealized_pnl,
                "realized_pnl": p.realized_pnl
            }
            for p in positions
        ]
    }

@router.get("/positions/{symbol}")
async def get_position(
    user_id: int = Query(..., ge=1),
    symbol: str = Path(...),
    service: TradeService = Depends(get_trade_service)
):
    """获取单个持仓"""
    position = service.get_position(user_id, symbol)
    if not position:
        return {"error": "Position not found"}

    return {
        "symbol": position.symbol,
        "quantity": position.quantity,
        "average_cost": position.average_cost,
        "market_value": position.market_value,
        "unrealized_pnl": position.unrealized_pnl,
        "realized_pnl": position.realized_pnl
    }
```

## 检查清单（Day 5-6结束时）

- [ ] 策略版本控制系统已实现
- [ ] 策略验证功能已完成
- [ ] 策略激活和回滚功能已实现
- [ ] 订单管理系统已完成
- [ ] 持仓管理系统已完成
- [ ] 风险控制接口已定义
- [ ] 所有API文档完整

## 下一步预告

Day 7-8将学习：
- 回测系统架构设计
- 性能指标计算
- 回测结果存储和查询
- 结果可视化数据接口

