# FastAPI RESTful API 项目

FastAPI + SQLAlchemy 2.0 + Alembic，分层架构，对接 React 前端。使用python，UV作为虚拟环境和包的管理

## 项目架构

```
app/
├── main.py                # FastAPI 应用入口
├── config.py              # 配置管理（pydantic-settings）
├── database.py            # 数据库配置（SQLAlchemy 2.0）
├── api/
│   ├── controller/        # 控制器层 - HTTP 请求处理
│   ├── service/           # 服务层 - 业务逻辑
│   ├── dao/               # DAO 层 - 数据访问
│   ├── models/            # Model 层 - ORM 模型
│   └── schemas/           # Schema 层 - Pydantic 验证模型
├── middleware/             # 中间件（JWT 认证、CORS）
└── common/                #工具方法
    └── util/              # 工具函数
migrations/                # Alembic 数据库迁移
tests/                     # 测试目录
scripts/                   # 脚本目录
```

**分层约束：**
- Controller 只接收/验证请求、调用 Service、返回 Result 响应，不含业务逻辑
- Service 之间可互相调用，DAO 之间不可
- Schemas（Pydantic）与 Models（SQLAlchemy）严格分离
- 新功能遵循：Schema → Model → DAO → Service → Controller → 注册路由

## FastAPI 路由与中间件

**路由注册：**
- 路由在 `app/main.py` 的 `create_app()` 中集中注册
- 使用 `APIRouter`，按资源分组，前缀 `/api/v1`
- 每个资源一个 controller 文件，如 `user_controller.py`
- 所有接口必须有中文 docstring（用于 Swagger 文档生成）

**路由风格：**
- RESTful：`GET /users`、`POST /users`、`GET /users/{user_id}`、`PUT /users/{user_id}`、`DELETE /users/{user_id}`
- URL 小写 + 连字符
- 路径参数 `{user_id}` 风格，查询参数 `?page=1&size=20`

**中间件顺序：** ExceptionMiddleware → CORS → JWT 认证 → 业务中间件

## 响应格式

成功响应：
```json
{ "data": { ... } }
```

分页响应：
```json
{ "data": [...], "total": 100, "page": 1, "size": 20 }
```

错误响应：
```json
{ "code": "NOT_FOUND", "message": "用户不存在" }
```

- `code`：业务错误码，大写下划线
- `message`：中文描述
- HTTP 状态码与业务错误码分离

## SQLAlchemy 2.0 与 DAO

**模型定义：**
- 放 `app/api/models/`，每个资源一个文件
- 继承 `app.database.Base`，用 `__tablename__` 显式指定表名
- 所有模型必须有中文 docstring
- 字段用类型注解 + `Column`，`created_at` / `updated_at` 用 `server_default=func.now()`

**Schema 定义：**
- 放 `app/api/schemas/`，每个资源一个文件
- Pydantic v2 语法，`BaseModel` + `Field`
- 命名：`{Name}Request`、`{Name}Response`、`{Name}Query`
- 响应 Schema 与 Model 分离，不暴露内部字段

**DAO 层：**
- 放 `app/api/dao/`，命名 `{Model}DAO`
- 使用 SQLAlchemy 2.0 `select()` 语法，不用 `Query` API
- 方法为 `@staticmethod`，接收 `db: Session` 作为第一个参数
- 返回 Model 实例，不返回 Session 或 Query 对象
- 分页封装：`paginate(query, page, size) -> (items, total)`

**事务：**
- 跨 DAO 操作在 Service 层管理 `db.commit()` / `db.rollback()`
- 禁止在 Controller 层管理事务

## API 设计

**请求：**
- 请求体用 Schema（Pydantic）验证，FastAPI 自动校验
- 路径参数 `user_id: int` 声明，查询参数 `Query()` 约束
- 分页参数统一 `page: int = Query(1)` / `size: int = Query(20)`

**状态码：**
- 200 成功 / 201 创建 / 204 删除（无响应体）
- 400 参数错误 / 401 未认证 / 403 无权限 / 404 不存在
- 409 冲突 / 422 验证失败（Pydantic 自动返回） / 500 内部错误
- 禁止用 200 + 业务错误码代替正确的 HTTP 状态码

## 测试

- `pytest` + `httpx`（FastAPI TestClient）
- 测试目录 `tests/`，按资源分文件
- Controller 测试：`TestClient` 模拟 HTTP 请求
- Service 测试：mock DAO
- DAO 测试：SQLite 内存数据库集成测试
- 运行：`pytest --cov=app tests/`

## 配置

- `pydantic-settings`，`app/config.py` 中 `Settings` 类
- 优先级：环境变量 > `.env` 文件 > 默认值
- 敏感信息只用环境变量
- 使用 `uv` 管理依赖

## 数据库迁移

- Alembic 管理迁移
- 创建：`alembic revision --autogenerate -m "描述"`
- 执行：`alembic upgrade head`
- 回滚：`alembic downgrade -1`
- 禁止修改已执行的迁移文件
