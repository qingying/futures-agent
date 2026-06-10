# 期货问答智能体 - 技术方案

> 版本：v1.0
> 日期：2026-06-10
> 状态：初稿

---

## 一、项目概述

### 1.1 项目定位

构建一个面向期货交易领域的**问答型 AI 智能体**，具备以下核心能力：

- **专业知识问答**：回答期货基础知识、交易规则、合约规格、风控策略等问题
- **记忆能力**：记住用户的偏好、历史问答上下文、关注的品种
- **实时数据接入**：对接行情数据，回答实时/近实时的问题
- **分析能力**：技术面分析、基本面分析、持仓分析等

### 1.2 目标用户

- 期货交易新手：学习期货基础知识
- 交易员：获取品种分析、策略建议
- 投研人员：快速获取市场数据和合约信息

---

## 二、行业调研

### 2.1 期货/交易类 AI 智能体项目

| 项目 | Star | 简介 | 技术栈 |
|------|------|------|--------|
| [Lumibot](https://github.com/Lumiwealth/lumibot) | ~1.7k | AI 交易回测框架，支持期货/股票/期权 | Python |
| [Nof1 Tracker](https://github.com/terryso/nof1-tracker) | ~756 | AI Agent 交易信号追踪 + Binance 期货自动交易 | Python |
| [CloddsBot](https://github.com/alsk1992/CloddsBot) | ~342 | 开源 AI 交易智能体，覆盖 1000+ 市场 | Python |
| [OKX Agent Trade Kit](https://github.com/okx/agent-trade-kit) | ~326 | OKX 交易 MCP Server，163 个工具覆盖完整交易流程 | TypeScript |
| [TradingAgents_for_Futures](https://github.com/haoge10241024/TradingAgents_for_Futures) | ~76 | 商品期货 AI 分析系统，多智能体辩论 + 量化模型 | Python + Streamlit |
| [Futures Quant Agents](https://github.com/yllvar/futures-quant-agents) | ~16 | 实时市场数据 + 技术分析 + LLM 的 Agentic 交易平台 | Python |

### 2.2 关键技术能力调研

#### 2.2.1 记忆系统

行业主流方案分三层：

| 记忆类型 | 实现方式 | 代表项目 |
|----------|----------|----------|
| **短期记忆（对话上下文）** | 消息历史窗口 / sliding window | LangChain, OpenAI API |
| **长期记忆（用户画像/偏好）** | 向量数据库 (Chroma/Weaviate/Pinecone) | LangGraph Memory, Mem0 |
| **语义记忆（知识库）** | RAG + 向量检索 (FAISS/Milvus) | LlamaIndex RAG, LangChain RAG |

#### 2.2.2 技能（Skills/Tools）架构

行业内流行的交易相关 Skills：

| Skill 类别 | 具体能力 | 参考项目 |
|------------|----------|----------|
| **行情查询** | 实时价格、K 线数据、成交量 | OKX Trade Kit (70+ 指标) |
| **技术分析** | MACD、RSI、布林带等指标计算 | Lumibot, TradingAgents_for_Futures |
| **基本面分析** | 供需数据、库存、宏观经济 | TradingAgents_for_Futures (6 大评估维度) |
| **持仓分析** | CFTC 持仓报告、主力持仓变动 | CloddsBot |
| **风控评估** | 仓位管理、止损止盈、VaR 计算 | OKX Trade Kit (安全机制) |
| **策略回测** | 历史数据回测验证 | Lumibot |
| **情绪分析** | 新闻舆情、社交媒体情绪 | OKX Trade Kit (新闻+情绪分析) |
| **日历价差** | 跨期套利分析、价差监控 | TradingAgents_for_Futures |

#### 2.2.3 多智能体架构趋势

`TradingAgents_for_Futures` 项目采用**对抗式多智能体**架构值得关注：
- 多个分析师 Persona 进行辩论
- 首席分析师综合各方意见
- 风控评估 + 决策者审批流水线
- 这种模式可以显著降低单一模型幻觉带来的错误建议

### 2.3 竞品分析总结

| 维度 | 现有方案不足 | 本项目差异化 |
|------|-------------|-------------|
| 知识覆盖 | 大多侧重交易执行，缺乏知识问答 | **专注问答 + 知识库** |
| 中国市场 | 多数面向加密货币/美股 | **聚焦中国商品期货** |
| 记忆能力 | 简单对话历史 | **三层记忆系统（短期+长期+语义）** |
| 实时性 | 离线/延迟数据 | **接入 AkShare 实时行情** |
| 可解释性 | 黑盒建议 | **提供分析链路和引用来源** |

---

## 三、架构设计

### 3.1 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                      用户界面层                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │ Web Chat │  │  API     │  │  CLI     │               │
│  └──────────┘  └──────────┘  └──────────┘               │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                    智能体编排层                           │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐                     │
│  │  意图识别    │──│  路由分发    │                     │
│  └──────────────┘  └──────┬───────┘                     │
│                           │                             │
│  ┌──────────────┐  ┌──────▼───────┐  ┌──────────────┐   │
│  │  知识问答    │  │  行情查询    │  │  技术分析    │   │
│  │   Agent      │  │   Agent      │  │   Agent      │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                 │                 │           │
│  ┌──────▼─────────────────▼─────────────────▼───────┐   │
│  │              结果综合 & 安全校验                   │   │
│  └──────────────────┬───────────────────────────────┘   │
└─────────────────────┼──────────────────────────────────┘
                      │
┌─────────────────────▼──────────────────────────────────┐
│                      能力层                             │
│                                                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐          │
│  │ RAG 知识库  │ │ 记忆系统   │ │ 工具集     │          │
│  │            │ │            │ │            │          │
│  │ • 期货规则  │ │ • 短期记忆 │ │ • 行情API  │          │
│  │ • 合约规格  │ │ • 长期记忆 │ │ • 指标计算 │          │
│  │ • 历史问答  │ │ • 语义记忆 │ │ • 回测引擎 │          │
│  └────────────┘ └────────────┘ └────────────┘          │
└─────────────────────────────────────────────────────────┘
```

### 3.2 技术栈选型

| 层级 | 组件 | 选型 | 理由 |
|------|------|------|------|
| **LLM** | 大模型 | Claude Sonnet / GPT-4o | 推理能力强、工具调用成熟 |
| **Agent 框架** | 智能体编排 | LangGraph | 支持有状态多步推理、天然支持记忆 |
| **记忆** | 短期记忆 | LangGraph 内置 StateGraph | 自动管理对话上下文 |
| **记忆** | 长期记忆 | ChromaDB (轻量) / Qdrant (生产) | 开源、嵌入式部署 |
| **记忆** | 语义记忆 | LangChain RAG + FAISS | 成熟的 RAG 管线 |
| **数据** | 行情数据 | AkShare | 中国期货市场全覆盖、免费 |
| **数据** | 技术指标 | TA-Lib / pandas-ta | 行业标准指标库 |
| **Web** | 前端 | Next.js + Tailwind CSS | 快速构建、现代化 UI |
| **API** | 后端 | FastAPI (Python) | 与 Agent 框架同语言、异步支持 |
| **部署** | 容器化 | Docker Compose | 一键部署 |

---

## 四、记忆系统设计

### 4.1 三层记忆架构

```
┌──────────────────────────────────────────┐
│           记忆系统总览                     │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │ 第一层：短期记忆（会话级）            │ │
│  │ • 当前对话消息历史                    │ │
│  │ • 滑动窗口：最近 20 轮               │ │
│  │ • 存储在 LangGraph State 中          │ │
│  └─────────────────┬───────────────────┘ │
│                    │                     │
│  ┌─────────────────▼───────────────────┐ │
│  │ 第二层：长期记忆（用户级）            │ │
│  │ • 用户偏好（关注的品种、交易风格）    │ │
│  │ • 历史交互摘要                      │ │
│  │ • 存储在 ChromaDB / Qdrant          │ │
│  │ • 按用户 ID 隔离                    │ │
│  └─────────────────┬───────────────────┘ │
│                    │                     │
│  ┌─────────────────▼───────────────────┐ │
│  │ 第三层：语义记忆（知识级）            │ │
│  │ • 期货规则、合约规格、交易制度        │ │
│  │ • RAG 检索增强                      │ │
│  │ • 向量化存储 (FAISS)                │ │
│  │ • 带来源引用                        │ │
│  └─────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

### 4.2 各层详细设计

#### 4.2.1 短期记忆

```python
# LangGraph State 定义
class ConversationState(MessagesState):
    user_id: str
    focus_products: list[str]      # 用户关注的品种
    session_context: dict          # 当前会话上下文
```

- 管理方式：LangGraph 内置 `MessagesState`，自动维护消息历史
- 窗口策略：保留最近 20 轮对话，超出部分摘要压缩
- 关键信息提取：从对话中提取用户提到的品种、策略偏好等，同步到长期记忆

#### 4.2.2 长期记忆

```python
# 用户画像 schema
UserProfile = {
    "user_id": str,
    "focus_products": ["螺纹钢", "铁矿石", "原油"],  # 关注品种
    "trading_style": "趋势跟踪",                       # 交易风格
    "risk_level": "中等",                              # 风险偏好
    "interaction_summary": [                           # 交互摘要
        {"date": "2026-06-10", "topic": "螺纹钢技术分析", "key_points": [...]},
    ],
}
```

- 存储：ChromaDB（开发）/ Qdrant（生产）
- 检索策略：每次对话开始时，根据 `user_id` 加载用户画像
- 更新策略：对话结束时，提取关键信息异步更新

#### 4.2.3 语义记忆（RAG 知识库）

| 知识库分类 | 数据来源 | 更新频率 |
|------------|----------|----------|
| 期货基础知识 | 交易所官网、期货业协会 | 低频（季度） |
| 合约规格 | 上期所、大商所、郑商所、中金所 | 中频（合约变更时） |
| 交易规则 | 各交易所规章制度 | 低频 |
| 历史问答 | 用户交互数据（经筛选） | 持续 |
| 分析方法论 | 技术分析/基本面分析教材 | 低频 |
| **用户自建知识库** | **用户自主上传** | **随时** |

#### 4.2.4 用户自建知识库

允许用户通过 Web 界面自主上传、管理知识库文档，实现个性化知识注入。

**上传方式**：

- **文件上传**：拖拽或选择文件上传
- **文本粘贴**：直接粘贴文本内容
- **URL 抓取**：输入网页 URL，自动抓取并解析内容
- **批量导入**：支持 ZIP 批量上传

**支持的文档格式**：

| 格式类型 | 扩展名 | 解析策略 |
|----------|--------|----------|
| 纯文本 | `.txt`, `.md` | 直接读取 |
| 文档 | `.pdf`, `.docx` | 提取文本 + 保留段落结构 |
| 表格 | `.xlsx`, `.csv` | 结构化提取，按行/列分块 |
| 网页 | `.html`, URL | 提取正文，过滤导航/广告 |

**处理流程**：

```
上传 → 格式解析 → 文本清洗 → 智能分块 → 向量化 → 入库 → 状态展示
  │         │          │          │         │       │        │
  │         │          │          │         │       │        └─ 可见/可用/索引中
  │         │          │          │         │       └─ ChromaDB collection
  │         │          │          │         └─ embedding model (text-embedding-3-small)
  │         │          │          └─ 语义分块 (500-1000 tokens, 50 overlap)
  │         │          └─ 去除空白/特殊字符/乱码
  │         └─ PyPDF2 / python-docx / openpyxl
  └─ 校验: 大小 ≤50MB, 类型白名单
```

**知识库管理功能**：

- **知识库列表**：按分类/标签/时间筛选
- **文档管理**：查看、编辑、删除已上传文档
- **索引状态**：显示每篇文档的向量化进度
- **版本控制**：文档更新保留历史版本
- **搜索预览**：在知识库内搜索，预览检索结果
- **手动触发重索引**：当用户更换 embedding 模型时

**隔离策略**：

- **系统知识库**：管理员维护的基础知识，所有用户共享
- **用户私有知识库**：仅创建者本人可见，回答时优先检索
- **公开分享**：用户可选择将私有知识库公开给其他用户

---

## 五、核心功能模块

### 5.1 意图识别与路由

```
用户输入 → 意图分类器 → 路由到对应 Agent

意图类别：
├── knowledge      → 知识问答 Agent    （"什么是保证金？"）
├── quote          → 行情查询 Agent    （"螺纹钢现在什么价格？"）
├── analysis       → 技术分析 Agent    （"帮我分析一下铁矿石的走势"）
├── strategy       → 策略建议 Agent    （"螺纹钢有什么好的交易策略？"）
├── risk           → 风控评估 Agent    （"这个仓位风险大吗？"）
└── general        → 通用对话 Agent    （"你好"）
```

### 5.2 工具集（Tools）

| 工具名 | 功能 | 输入 | 输出 |
|--------|------|------|------|
| `get_quote` | 查询实时行情 | 品种代码 | 价格、涨跌幅、成交量等 |
| `get_kline` | 获取 K 线数据 | 品种代码、周期 | OHLCV 数据 |
| `calc_indicator` | 计算技术指标 | K 线数据、指标名 | 指标值 |
| `search_knowledge` | RAG 知识检索 | 问题文本 | 相关文档片段 + 引用 |
| `get_contract_info` | 查询合约信息 | 品种代码 | 合约规格、交割月等 |
| `backtest` | 策略回测 | 策略参数 | 回测报告 |
| `get_news` | 获取相关新闻 | 品种关键词 | 新闻列表 + 情绪分 |

### 5.3 数据源

| 数据类型 | 数据源 | 接口方式 |
|----------|--------|----------|
| 实时行情 | AkShare | REST API |
| 历史行情 | AkShare | REST API |
| 合约信息 | 交易所官网爬虫 | 定时爬取 |
| 新闻资讯 | AkShare + 新浪财经 | REST API |
| 持仓数据 | CFTC / 交易所 | REST API / 爬虫 |
| 知识文档 | 手动整理 + 文档解析 | 向量化入库 |
| 用户文档 | 用户上传（Web 界面） | 自动解析 + 向量化入库 |

### 5.4 知识库管理 API

| 接口 | 方法 | 描述 | 请求体 |
|------|------|------|--------|
| `/api/kb/collections` | GET | 获取知识库列表 | - |
| `/api/kb/collections` | POST | 创建知识库 | `{name, description, visibility}` |
| `/api/kb/collections/:id` | DELETE | 删除知识库 | - |
| `/api/kb/collections/:id/docs` | GET | 获取文档列表 | - |
| `/api/kb/collections/:id/docs` | POST | 上传文档 | `multipart/form-data` (文件) |
| `/api/kb/collections/:id/docs/:docId` | GET | 查看文档详情 | - |
| `/api/kb/collections/:id/docs/:docId` | PUT | 更新文档 | `{content}` (文本编辑) |
| `/api/kb/collections/:id/docs/:docId` | DELETE | 删除文档 | - |
| `/api/kb/collections/:id/docs/:docId/reindex` | POST | 手动重索引 | - |
| `/api/kb/search` | POST | 搜索知识库 | `{query, collection_ids}` |
| `/api/kb/url/fetch` | POST | URL 内容抓取 | `{url, collection_id}` |

---

## 六、项目实施计划

### 6.1 里程碑

| 阶段 | 时间 | 交付物 |
|------|------|--------|
| **P0 - 基础搭建** | Week 1-2 | 项目骨架、LLM 接入、基础问答 |
| **P1 - 记忆系统** | Week 3-4 | 三层记忆系统、用户画像 |
| **P2 - 数据接入** | Week 5-6 | AkShare 行情接入、技术指标 |
| **P3 - 知识库 + 用户上传** | Week 7-8 | RAG 知识库、文档上传/解析/向量化管线、KB 管理 UI |
| **P4 - Web 界面** | Week 9-10 | Chat UI、用户管理、知识库管理 |
| **P5 - 生产部署** | Week 11-12 | Docker 化、监控告警 |

### 6.2 项目结构

```
futures-agent/
├── README.md
├── docs/
│   └── technical-proposal.md      # 本文档
├── backend/
│   ├── src/
│   │   ├── agents/                 # Agent 实现
│   │   │   ├── router.py           # 意图识别与路由
│   │   │   ├── knowledge_agent.py  # 知识问答
│   │   │   ├── quote_agent.py      # 行情查询
│   │   │   ├── analysis_agent.py   # 技术分析
│   │   │   └── strategy_agent.py   # 策略建议
│   │   ├── memory/                 # 记忆系统
│   │   │   ├── short_term.py       # 短期记忆
│   │   │   ├── long_term.py        # 长期记忆
│   │   │   └── semantic.py         # 语义记忆 (RAG)
│   │   ├── rag/                    # RAG 知识库管理
│   │   │   ├── parser.py           # 文档解析器 (PDF/DOCX/XLSX/HTML)
│   │   │   ├── chunker.py          # 智能分块策略
│   │   │   ├── embedder.py         # 向量化引擎
│   │   │   ├── store.py            # 向量存储 (ChromaDB)
│   │   │   ├── retriever.py        # 检索器 (混合检索)
│   │   │   └── kb_manager.py       # 知识库 CRUD
│   │   ├── tools/                  # 工具集
│   │   │   ├── quote.py            # 行情工具
│   │   │   ├── indicator.py        # 指标计算
│   │   │   ├── backtest.py         # 回测工具
│   │   │   └── news.py             # 新闻工具
│   │   ├── data/                   # 数据层
│   │   │   ├── akshare_client.py   # AkShare 封装
│   │   │   └── crawler/            # 交易所爬虫
│   │   └── api/                    # FastAPI 路由
│   │       ├── chat.py             # 对话接口
│   │       ├── user.py             # 用户管理
│   │       └── kb.py               # 知识库管理 API
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── app/                        # Next.js 应用
│   │   ├── chat/                   # 对话页面
│   │   ├── kb/                     # 知识库管理页面
│   │   │   ├── page.tsx            # 知识库列表
│   │   │   ├── upload/             # 上传文档
│   │   │   └── [id]/               # 文档详情/编辑
│   │   └── settings/               # 用户设置
│   ├── components/
│   │   ├── kb/
│   │   │   ├── FileUploader.tsx    # 文件上传组件
│   │   │   ├── UrlFetcher.tsx      # URL 抓取组件
│   │   │   ├── TextEditor.tsx      # 文本粘贴编辑器
│   │   │   ├── KbTable.tsx         # 知识库列表表格
│   │   │   └── IndexProgress.tsx   # 索引进度指示器
│   │   └── chat/
│   │       └── ChatWindow.tsx      # 对话窗口
│   ├── package.json
│   └── Dockerfile
├── knowledge-base/                 # 系统知识库原始文档
│   ├── basics/                     # 基础知识
│   ├── contracts/                  # 合约规格
│   ├── rules/                      # 交易规则
│   └── methods/                    # 分析方法论
├── scripts/
│   ├── ingest_kb.py                # 系统知识库入库脚本
│   └── crawl_contracts.py          # 合约信息爬取
└── docker-compose.yml
```

---

## 七、风险与对策

| 风险 | 影响 | 对策 |
|------|------|------|
| LLM 幻觉导致错误建议 | 用户经济损失 | 所有建议附带免责声明 + 安全校验层 |
| 行情数据延迟 | 回答不准确 | 多数据源备份 + 数据新鲜度检查 |
| 记忆污染 | 用户画像偏差 | 定期审核 + 手动修正入口 |
| 知识库过期 | 规则变更未同步 | 设置文档过期提醒机制 |
| 用户上传低质/冲突内容 | 回答质量下降 | 文档质量评分 + 冲突检测 + 用户可关闭知识库 |
| 合规风险 | 违反期货咨询规定 | 仅提供信息，不构成投资建议，用户协议明确 |

---

## 八、下一步行动

1. **创建 GitHub 仓库** ✅（本文档所在仓库）
2. **搭建项目骨架**：初始化 Python 后端 + Next.js 前端
3. **接入 LLM**：配置 Claude API / OpenAI API
4. **实现基础问答**：Prompt 工程 + 简单 RAG
5. **逐步接入数据和记忆模块**

---

## 附录 A：参考项目

- [OKX Agent Trade Kit](https://github.com/okx/agent-trade-kit) — MCP Server 架构参考
- [TradingAgents_for_Futures](https://github.com/haoge10241024/TradingAgents_for_Futures) — 多智能体对抗分析参考
- [Lumibot](https://github.com/Lumiwealth/lumibot) — 回测框架参考
- [LangGraph](https://github.com/langchain-ai/langgraph) — Agent 编排框架
- [AkShare](https://github.com/akfamily/akshare) — 中国金融市场数据源

## 附录 B：术语表

| 术语 | 说明 |
|------|------|
| RAG | Retrieval-Augmented Generation，检索增强生成 |
| MCP | Model Context Protocol，模型上下文协议 |
| Agent | 智能体，具备感知、推理、行动能力的 AI 系统 |
| AkShare | 开源金融数据接口库，覆盖中国期货市场 |
| ChromaDB | 轻量级向量数据库，适合嵌入式部署 |
| FAISS | Facebook AI 相似性搜索库，高性能向量检索 |
