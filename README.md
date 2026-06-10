# Futures Agent - 期货问答智能体

面向期货交易领域的智能问答 Agent，具备专业期货知识问答、实时行情查询、技术分析和三层记忆系统。

## 特性

- **专业知识问答**：期货基础知识、交易规则、合约规格
- **三层记忆系统**：短期对话上下文 + 长期用户画像 + 语义知识库 (RAG)
- **实时行情**：接入 AkShare，覆盖中国四大期货交易所
- **技术分析**：MACD、RSI、布林带等 70+ 技术指标
- **多 Agent 编排**：意图识别路由到专业知识 Agent

## 快速开始

> 项目正在开发中，详见 [技术方案文档](docs/technical-proposal.md)

```bash
# 克隆项目
git clone https://github.com/qingying/futures-agent.git
cd futures-agent

# 安装依赖 (待实现)
make install

# 启动开发服务器 (待实现)
make dev
```

## 技术栈

- **Agent 框架**：LangGraph
- **LLM**：Claude Sonnet / GPT-4o
- **记忆系统**：ChromaDB + LangChain RAG
- **数据源**：AkShare（中国期货市场）
- **后端**：FastAPI (Python)
- **前端**：Next.js + Tailwind CSS
- **部署**：Docker Compose

## 项目结构

```
├── backend/           # Python 后端
├── frontend/          # Next.js 前端
├── docs/              # 技术文档
├── knowledge-base/    # RAG 知识库
└── scripts/           # 工具脚本
```

## 开发状态

- [x] 技术方案
- [ ] 项目骨架
- [ ] LLM 接入
- [ ] 记忆系统
- [ ] 数据接入
- [ ] Web 界面

## License

MIT
