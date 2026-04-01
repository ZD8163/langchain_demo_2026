# 🤖 智扫通机器人智能客服系统

> 基于 RAG + ReAct Agent 技术的扫地机器人 AI 智能客服，集知识检索、工具调用、动态提示词切换、个性化报告生成于一体。

---

## 📖 项目简介

本项目是一个面向扫地机器人/扫拖一体机器人领域的 **AI 智能客服系统**，综合运用 **RAG（检索增强生成）** 与 **ReAct Agent（智能体）** 技术，基于阿里云通义千问大模型构建，提供专业的产品问答、故障排查、选购建议及个性化使用报告等能力。

### 核心能力

- 💬 **专业问答**：基于5大知识库（100+ 问答、200条故障排查、200条保养建议、200条选购指南）精准回答用户问题
- 🔧 **故障排查**：结合天气/位置等环境信息，智能分析机器人故障原因
- 📊 **使用报告**：自动获取用户ID和月份，从外部数据库读取使用记录，生成 Markdown 格式的个性化月度报告
- 🌤️ **环境适配建议**：结合用户所在城市实时天气，给出机器人使用建议

---

## 🏗️ 项目架构

```
AI大模型RAG与智能体开发_Agent项目/
├── app.py                      # Streamlit Web 前端入口
├── main.py                     # PyCharm 占位文件
├── md5.text                    # 已入库文件 MD5 记录（去重用）
│
├── agent/                      # Agent 智能体核心模块
│   ├── react_agent.py          # ReAct Agent 主体
│   └── tools/
│       ├── agent_tools.py      # 7 个工具函数定义
│       └── middleware.py       # 中间件（监控 / 日志 / 动态提示词）
│
├── rag/                        # RAG 检索增强生成模块
│   ├── rag_service.py          # RAG 总结服务
│   └── vector_store.py         # 向量数据库管理（ChromaDB）
│
├── model/                      # 模型工厂模块
│   └── factory.py              # Chat 模型 + Embedding 模型工厂
│
├── utils/                      # 公共工具模块
│   ├── config_handler.py       # YAML 配置文件加载
│   ├── file_handler.py         # 文件操作（MD5、文档加载）
│   ├── logger_handler.py       # 双通道日志系统
│   ├── path_tool.py            # 跨平台绝对路径工具
│   └── prompt_loader.py        # 提示词加载器
│
├── config/                     # 配置文件
│   ├── rag.yml                 # 模型配置
│   ├── chroma.yml              # 向量库配置
│   ├── prompts.yml             # 提示词路径配置
│   └── agent.yml               # Agent 外部数据配置
│
├── prompts/                    # 提示词文件
│   ├── main_prompt.txt         # 主客服系统提示词
│   ├── rag_summarize.txt       # RAG 总结提示词
│   └── report_prompt.txt       # 报告生成专用提示词
│
├── data/                       # 知识库数据
│   ├── 扫地机器人100问2.txt
│   ├── 扫拖一体机器人100问.txt
│   ├── 故障排除.txt
│   ├── 维护保养.txt
│   ├── 选购指南.txt
│   └── external/
│       └── records.csv         # 用户使用记录（用户ID 1001-1010，2025年全年）
│
├── chroma_db/                  # ChromaDB 向量库持久化目录（运行时生成）
└── logs/                       # 日志目录（运行时生成）
```

---

## 🔧 技术栈

| 技术 / 框架 | 版本要求 | 用途 |
|---|---|---|
| Python | ≥ 3.10 | 运行环境 |
| Streamlit | 最新版 | Web UI 框架 |
| LangChain | 最新版 | Agent / Chain / 工具框架 |
| LangGraph | 最新版 | Agent 运行时（ReAct 流式执行） |
| ChromaDB | 最新版 | 向量数据库 |
| 通义千问 (qwen3-max) | — | 对话大模型 |
| DashScope text-embedding-v4 | — | 文本向量化模型 |
| PyYAML | 最新版 | 配置文件解析 |
| PyPDF | 最新版 | PDF 文档加载 |

---

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone <仓库地址>
cd AI大模型RAG与智能体开发_Agent项目
```

### 2. 创建虚拟环境

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
```

### 3. 安装依赖

```bash
pip install streamlit langchain langchain-core langchain-community langchain-chroma \
            langchain-text-splitters langgraph chromadb dashscope pyyaml pypdf python-dotenv
```

### 4. 配置 API Key

在项目根目录创建 `.env` 文件，填入阿里云 DashScope API Key：

```env
DASHSCOPE_API_KEY=your_api_key_here
```

> 获取地址：[https://dashscope.console.aliyun.com/](https://dashscope.console.aliyun.com/)

### 5. 初始化向量知识库

首次运行前，需将 `data/` 目录下的知识文档加载至向量库：

```bash
python rag/vector_store.py
```

运行完成后，`chroma_db/` 目录将生成持久化的向量数据，`md5.text` 将记录已处理文件的 MD5（防止重复加载）。

### 6. 启动 Web 应用

```bash
streamlit run app.py
```

启动成功后，浏览器自动打开 `http://localhost:8501`，即可使用智能客服。

---

## 💡 功能演示

### 普通问答

```
用户：扫地机器人建图不完整怎么办？
      我家今天适合使用扫拖功能吗？
      小户型适合买哪款扫地机器人？
```

### 使用报告生成

```
用户：帮我生成上个月的使用报告
```

Agent 将自动按以下流程执行：
1. 调用 `get_user_id` 获取用户 ID
2. 调用 `get_current_month` 获取当前月份
3. 调用 `fill_context_for_report` 触发提示词切换（客服模式 → 报告模式）
4. 调用 `fetch_external_data` 获取使用记录
5. 调用 `rag_summarize` 补充保养建议
6. 输出 Markdown 格式的完整月度报告

---

## 🛠️ 可用工具一览

| 工具名 | 入参 | 功能描述 |
|---|---|---|
| `rag_summarize` | `query: str` | 从向量知识库检索专业资料并总结回答 |
| `get_weather` | `city: str` | 获取指定城市天气信息（模拟数据） |
| `get_user_location` | 无 | 获取用户所在城市 |
| `get_user_id` | 无 | 获取用户唯一 ID |
| `get_current_month` | 无 | 获取当前月份（YYYY-MM 格式） |
| `fetch_external_data` | `user_id`, `month` | 从 CSV 读取指定用户指定月份的使用记录 |
| `fill_context_for_report` | 无 | 触发报告模式，切换系统提示词 |

---

## ⚙️ 配置说明

### `config/rag.yml` — 模型配置

```yaml
chat_model_name: qwen3-max           # 通义千问对话模型
embedding_model_name: text-embedding-v4  # 向量化模型
```

### `config/chroma.yml` — 向量库配置

```yaml
collection_name: agent               # 向量集合名称
persist_directory: chroma_db         # 持久化目录
k: 3                                 # 检索返回 Top-K 文档数
data_path: data                      # 知识库数据目录
chunk_size: 200                      # 文本分块大小（字符数）
chunk_overlap: 20                    # 分块重叠长度
```

---

## 🧠 核心设计亮点

### 1. RAG + Agent 深度融合

RAG 作为 Agent 的一个工具被调用，而非独立系统，实现知识增强与智能推理的无缝结合。

### 2. 动态提示词切换（中间件机制）

通过 LangGraph 中间件 + 运行时上下文（`context["report"]`），同一个 Agent 在不同业务场景下自动切换系统提示词：

```
普通问答  →  main_prompt.txt（客服角色）
报告生成  →  report_prompt.txt（报告写手角色）
```

### 3. 工具调用强约束设计

`fill_context_for_report` 作为报告流程的"门控工具"，在提示词中明确约束调用顺序，防止模型跳过关键步骤。

### 4. MD5 文件去重机制

知识库加载时通过 MD5 哈希比对，避免文档重复向量化，保证数据一致性。

### 5. 完整的日志体系

双通道日志（控制台 INFO + 文件 DEBUG），工具调用的入参、出参、异常全链路可追溯，日志文件按日期自动命名。

---

## 📁 知识库数据说明

| 文件 | 条目数 | 覆盖内容 |
|---|---|---|
| `扫地机器人100问2.txt` | 100 条 | 基础使用、清洁效果、导航建图、WiFi 连接等 |
| `扫拖一体机器人100问.txt` | 100 条 | 扫拖融合功能、拖地系统、水箱管理、拖布保养等 |
| `故障排除.txt` | 200 条 | 故障现象 → 检测方法 → 修复方案 |
| `维护保养.txt` | 200 条 | 通用基础 / 扫地专属 / 拖地专属 / 耗材 / 长期存放 |
| `选购指南.txt` | 200 条 | 吸力、导航、续航、水箱、拖布、品牌等维度 |

外部数据：`data/external/records.csv` 包含用户 ID 1001-1010 的 2025 年全年使用记录（约 120 条），用于个性化报告生成。

---

## 📝 系统运行流程

```
用户输入
    │
    ▼
app.py (Streamlit UI)
    │ execute_stream(query)
    ▼
ReactAgent
    ├── [中间件] log_before_model    每次调模型前记录日志
    ├── [中间件] report_prompt_switch 根据 context["report"] 动态切换提示词
    │       ├── False → main_prompt.txt（客服模式）
    │       └── True  → report_prompt.txt（报告模式）
    ├── LLM 推理（qwen3-max，ReAct 思考循环）
    └── 工具调用（由 monitor_tool 中间件监控）
            ├── rag_summarize → VectorStore → ChromaDB → RAG Chain
            ├── get_weather / get_user_location
            ├── get_user_id / get_current_month
            ├── fetch_external_data → records.csv
            └── fill_context_for_report → 触发提示词切换
    │
    ▼
流式输出最终回答（打字机效果）
```

---

## 📄 许可证

本项目仅供学习交流使用。

---

> 💡 **提示**：首次运行前请确保已执行知识库初始化步骤，否则 RAG 检索将无法返回有效内容。
