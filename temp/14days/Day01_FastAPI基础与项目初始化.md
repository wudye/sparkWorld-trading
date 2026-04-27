# Day 1: FastAPI基础与项目初始化

## 学习目标
- ✅ 安装和配置uv包管理器
- ✅ 理解FastAPI核心概念和特性
- ✅ 创建Qbot FastAPI后端项目结构
- ✅ 运行第一个FastAPI应用并理解请求/响应流程

## 核心学习内容

### 1. uv包管理器基础
- uv vs pip：性能对比、安装管理、依赖管理
- 安装uv工具
- 创建虚拟环境：`uv venv`
- 项目初始化：`uv init`

### 2. FastAPI核心概念
- FastAPI是什么（基于Starlette + Pydantic）
- 为什么选择FastAPI（性能、自动文档、类型检查）
- 与Django、Flask的对比
- 异步编程基础（async/await）

### 3. 项目结构设计（Qbot后端）
```
qbot-backend/
├── pyproject.toml          # uv项目配置
├── uv.lock                 # 依赖锁定文件
├── .env                    # 环境变量
├── main.py                 # 应用入口
├── app/
│   ├── __init__.py
│   ├── main.py            # FastAPI应用实例
│   ├── config.py          # 配置管理
│   ├── models/            # Pydantic数据模型
│   ├── routes/            # API路由
│   ├── services/          # 业务逻辑层
│   ├── repository/        # 数据访问层
│   └── utils/             # 工具函数
├── tests/                 # 测试目录
└── README.md
```

### 4. FastAPI第一个应用
- 创建应用实例
- 定义路由（@app.get、@app.post等）
- 请求体和响应体
- 路径参数、查询参数
- 状态码和错误处理

## 关键技术点

1. **依赖注入**
   - FastAPI的Depends机制
   - 构建清晰的依赖链

2. **Pydantic模型**
   - 数据验证
   - 自动文档生成
   - 类型检查

3. **异步编程**
   - async/await语法
   - 异步数据库操作准备

4. **API文档**
   - Swagger UI（/docs）
   - ReDoc（/redoc）
   - OpenAPI标准

## 实践任务

### 任务1：环境搭建（1小时）
```bash
# 创建项目目录
mkdir qbot-backend
cd qbot-backend

# 初始化项目
uv init
uv add fastapi uvicorn python-dotenv
```

### 任务2：创建项目结构（1小时）
创建上述目录结构，创建`__init__.py`文件

### 任务3：编写第一个FastAPI应用（1.5小时）
创建 `app/main.py`：
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI(
    title="Qbot Trading API",
    description="AI-driven Quantitative Trading Platform",
    version="0.1.0"
)

@app.get("/")
async def root():
    return {"message": "Welcome to Qbot API"}

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

@app.get("/api/v1/stocks/{symbol}")
async def get_stock(symbol: str, limit: int = 10):
    return {
        "symbol": symbol,
        "limit": limit,
        "data": []
    }
```

创建 `main.py`（项目入口）：
```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=8000,
        reload=True
    )
```

### 任务4：测试和验证（1小时）
```bash
uv run python main.py

# 在浏览器访问：
# http://localhost:8000/
# http://localhost:8000/docs (Swagger UI)
# http://localhost:8000/api/v1/stocks/600000
```

## 代码示例需求

请完成以下文件的实现：

1. **app/config.py** - 配置管理
   - 数据库连接配置
   - API配置
   - 日志配置

2. **app/models/base.py** - 基础数据模型
   - Response模型
   - Error模型
   - PagenationModel

3. **app/utils/logger.py** - 日志工具
   - 结构化日志
   - 日志级别配置

## 学习资源链接

- FastAPI官方文档：https://fastapi.tiangolo.com/
- uv仓库：https://github.com/astral-sh/uv
- Pydantic官方文档：https://docs.pydantic.dev/

## 检查清单（Day 1结束时）

- [ ] uv环境已配置
- [ ] 第一个FastAPI应用能正常运行
- [ ] 能访问Swagger UI文档
- [ ] 理解FastAPI的基本路由和参数处理
- [ ] 项目目录结构已创建
- [ ] pyproject.toml已正确配置

## 常见问题

**Q: 为什么选择uv而不是pip？**
A: uv是用Rust编写的，速度快10-100倍，内存占用少，依赖解析更准确。

**Q: async/await必须学习吗？**
A: 是的，FastAPI的性能优势来自异步处理，这是核心概念。

**Q: 第一天需要学习数据库吗？**
A: 暂时不需要，Day 3才涉及数据库集成。今天关注API基础。

## 下一步预告

Day 2将学习：
- 更复杂的路由设计
- 中间件和全局异常处理
- 请求/响应的深度理解
- API版本控制策略

