# claude-rules

生产级 Claude Code 规则模板。覆盖前端、Go 后端、Python 后端和 UI 设计，复制即用，按需定制。

## 模板一览

| 路径 | 适用场景 | 核心内容 |
|------|---------|---------|
| [`CLAUDE.md`](CLAUDE.md) | 所有项目通用 | 开发者规则：指令优先级、编码风格、Git 策略、验证命令、跨平台适配 |
| [`frontend/CLAUDE.md`](frontend/CLAUDE.md) | React 前端 | React + Vite + Ant Design + TanStack Query + Zustand 架构约束与 API 对接规范 |
| [`frontend/AGENTS.md`](frontend/AGENTS.md) | 前端 UI 设计 | 配色框架、展示型/后台页面设计规则、动画体系、文案风格、组件库选择指南 |
| [`backend-go/CLAUDE.md`](backend-go/CLAUDE.md) | Go 后端 | Gin + GORM 分层架构、RESTful API 设计、错误响应格式、GORM 约定 |
| [`backend-fastapi/CLAUDE.md`](backend-fastapi/CLAUDE.md) | Python 后端 | FastAPI + SQLAlchemy 2.0 + Alembic 分层架构、DAO 模式、Pydantic Schema 规范 |

## 使用方式

1. 将对应模板复制到项目根目录或子目录
2. **修改 Git 用户信息**：将 `CLAUDE.md` 中 Git 策略部分的 `user.email` 和 `user.name` 替换为你自己的：
   ```
   user.email "your@email.com"，user.name "your-name"
   ```
3. 根据项目实际情况修改（数据库名、端口号、业务规则等）
4. Claude Code 会在会话启动时自动读取 `CLAUDE.md`

```bash
# 示例：Go 后端项目
cp backend-go/CLAUDE.md your-project/CLAUDE.md

# 示例：全栈项目（根目录放通用规则，子目录放技术栈规则）
cp CLAUDE.md your-project/CLAUDE.md
cp frontend/CLAUDE.md your-project/frontend/CLAUDE.md
cp backend-go/CLAUDE.md your-project/backend/CLAUDE.md
```

## 通用规则要点

根 `CLAUDE.md` 覆盖以下维度，适用于任何语言和框架：

- **指令优先级**：系统限制 > 用户指令 > 项目规则 > 通用规则
- **研究先行**：编码前理解既有代码，复用已有模式
- **手术式修改**：每行改动追溯到需求，不顺手重构
- **验证命令发现**：从 `package.json` / `pyproject.toml` / `go.mod` 等读取，不猜测
- **跨平台适配**：自动识别 Windows/Linux，选择对应命令
- **Git 策略**：约定式提交，禁止 push/merge/rebase，commit 前确认

## 前后端对接约定

所有模板共享统一的 API 响应格式，确保前后端无缝对接：

**成功响应：**
```json
{ "data": { ... } }
```

**分页响应：**
```json
{ "data": [...], "total": 100, "page": 1, "size": 20 }
```

**错误响应：**
```json
{ "code": "NOT_FOUND", "message": "用户不存在" }
```

## 许可

MIT
