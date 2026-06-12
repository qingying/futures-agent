# CLAUDE.md - 项目规范

本文件为 Claude Code 提供项目上下文，指导 AI 在此项目中的行为。

---

## 一、项目概述

**期货问答智能体（Futures Agent）** - 面向期货交易领域的 AI 问答工具。

核心能力：
- 专业知识问答（期货基础、交易规则、合约规格）
- 实时行情查询（接入 AkShare）
- 技术分析（MACD、RSI、布林带等指标）
- 三层记忆系统（短期对话 + 长期用户画像 + 语义知识库 RAG）
- 用户自建知识库（文档上传、解析、向量化）

---

## 二、技术栈

| 层级 | 技术 | 版本要求 |
|------|------|----------|
| **LLM** | Claude Sonnet / GPT-4o | 最新稳定版 |
| **Agent 框架** | LangGraph | >= 0.2 |
| **记忆系统** | ChromaDB（开发）/ Qdrant（生产） | ChromaDB >= 0.4 |
| **向量检索** | FAISS / LangChain RAG | - |
| **数据源** | AkShare | 最新稳定版 |
| **技术指标** | TA-Lib / pandas-ta | - |
| **后端** | FastAPI (Python) | Python >= 3.10, FastAPI >= 0.100 |
| **前端** | Next.js + Tailwind CSS | Next.js >= 14 |
| **部署** | Docker Compose | - |

---

## 三、项目结构

```
futures-agent/
├── backend/                # Python 后端
│   ├── src/
│   │   ├── agents/         # Agent 实现（router, knowledge, quote, analysis, strategy）
│   │   ├── memory/         # 记忆系统（short_term, long_term, semantic）
│   │   ├── rag/            # RAG 知识库（parser, chunker, embedder, store, retriever, kb_manager）
│   │   ├── tools/          # 工具集（quote, indicator, backtest, news）
│   │   ├── data/           # 数据层（akshare_client, crawler）
│   │   └── api/            # FastAPI 路由（chat, user, kb）
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/               # Next.js 前端
│   ├── app/                # 页面（chat, kb, settings）
│   ├── components/         # 组件（kb, chat）
│   ├── package.json
│   └── Dockerfile
├── knowledge-base/         # 系统知识库原始文档
├── docs/                   # 项目文档
│   ├── technical-proposal.md
│   └── prd.md
├── scripts/                # 工具脚本
└── docker-compose.yml
```

---

## 四、开发规范

### 4.1 语言要求

- **后端代码**：Python 3.10+，使用类型注解（type hints）
- **前端代码**：TypeScript，严格模式
- **注释语言**：中文为主，代码注释和文档字符串使用中文
- **变量命名**：英文，snake_case（Python）/ camelCase（TypeScript）

### 4.2 代码风格

**Python**:
- 遵循 PEP 8，使用 `ruff` 或 `black` 格式化
- 函数和类必须添加 docstring（中文）
- 使用 `dataclass` 或 `pydantic.BaseModel` 定义数据结构
- 异步代码使用 `async/await`

**TypeScript/React**:
- 使用函数组件 + Hooks
- 组件 Props 使用 `interface` 定义
- 使用 Tailwind CSS 进行样式开发，避免自定义 CSS

### 4.3 模块设计原则

- **单一职责**：每个 Agent/Tool 只负责一类任务
- **可测试性**：核心逻辑与外部依赖解耦，便于单元测试
- **错误处理**：所有外部调用（API、数据库）必须有异常处理和重试机制
- **日志记录**：关键操作记录日志，使用 `structlog` 或标准 `logging`

### 4.4 Git 规范

- 分支命名：`feature/xxx`、`fix/xxx`、`docs/xxx`
- Commit message 格式：`type: description`
  - type: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
  - 示例：`feat: 添加知识库上传功能`
- 提交前必须通过 lint 检查

---

## 五、API 设计规范

### 5.1 RESTful API

- 路径使用小写 + 连字符：`/api/kb/collections`
- HTTP 方法语义：
  - `GET`：查询
  - `POST`：创建
  - `PUT`：更新（全量）
  - `PATCH`：更新（部分）
  - `DELETE`：删除

### 5.2 响应格式

```json
{
  "code": 0,           // 0 表示成功，非 0 为错误码
  "message": "success",
  "data": {}           // 业务数据
}
```

### 5.3 错误处理

- 使用统一的异常处理中间件
- 返回明确的错误信息和错误码
- 敏感错误（如数据库错误）不暴露细节给用户

---

## 六、知识库规范

### 6.1 文档处理流程

```
上传 → 格式解析 → 文本清洗 → 智能分块 → 向量化 → 入库
```

### 6.2 分块策略

- 语义分块：按段落/章节边界
- 块大小：500-1000 tokens
- 重叠：50 tokens（保持上下文连贯）

### 6.3 知识库隔离

- **系统知识库**：管理员维护，所有用户共享
- **用户私有知识库**：仅创建者可见
- **公开分享**：用户可选择公开

---

## 七、安全与合规

### 7.1 数据安全

- 用户数据按 `user_id` 严格隔离
- API Key / Secret 不硬编码，使用环境变量
- 敏感操作记录审计日志

### 7.2 合规要求

- **免责声明**：所有涉及行情分析、策略建议的回答必须附带免责声明
- **不构成投资建议**：明确告知用户 AI 仅提供信息，不构成投资建议
- **数据准确性**：标注数据来源和时效性

---

## 八、测试规范

### 8.1 单元测试

- 核心业务逻辑覆盖率 >= 80%
- 使用 `pytest`（Python）/ `Jest`（TypeScript）

### 8.2 集成测试

- API 接口测试：覆盖正常流程和异常场景
- Agent 调用链测试：验证意图识别 → 路由 → 工具调用完整链路

### 8.3 端到端测试

- 关键用户流程：提问 → 回答 → 知识库检索
- 使用 Playwright 或 Cypress

---

## 九、部署规范

### 9.1 环境划分

- `dev`：本地开发环境
- `staging`：预发布测试环境
- `prod`：生产环境

### 9.2 Docker 部署

- 使用 `docker-compose.yml` 编排服务
- 镜像标签使用 Git commit hash 或语义化版本
- 生产环境不使用 `latest` 标签

### 9.3 环境变量

必须配置的环境变量：
- `LLM_API_KEY`：LLM 服务 API Key
- `DATABASE_URL`：数据库连接
- `CHROMADB_HOST`：ChromaDB 地址
- `AKSHARE_*`：AkShare 相关配置

---

## 十、Claude Code 行为准则

在此项目中，Claude Code 应：

1. **优先阅读文档**：修改代码前先阅读 `docs/technical-proposal.md` 和 `docs/prd.md` 了解上下文
2. **遵循现有架构**：不随意引入新的框架或依赖，如需引入需说明理由
3. **保持代码风格一致**：参考现有代码的命名、注释、结构风格
4. **关注安全性**：涉及用户数据、API Key 时格外谨慎
5. **添加免责声明**：涉及行情分析、策略建议时，在回答中提醒用户"不构成投资建议"
6. **测试优先**：修改核心逻辑后主动运行相关测试
7. **中文优先**：代码注释、commit message、文档使用中文

---

## 十一、常见问题

### Q: 如何选择 LLM 模型？
A: 默认使用 Claude Sonnet，如需更强推理能力切换到 Claude Opus，简单任务可用 Haiku 降低成本。

### Q: 知识库文档格式限制？
A: 支持 `.txt`, `.md`, `.pdf`, `.docx`, `.xlsx`, `.csv`, `.html`，单文件不超过 50MB。

### Q: 如何处理实时行情数据延迟？
A: AkShare 数据有 15 分钟延迟，回答时需标注数据时效性，关键决策建议用户参考交易软件。

---

## 十二、参考资源

- [技术方案文档](docs/technical-proposal.md)
- [产品需求文档](docs/prd.md)
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [AkShare 文档](https://akshare.akfamily.xyz/)
- [FastAPI 文档](https://fastapi.tiangolo.com/)
