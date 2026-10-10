# 📂 文档：受限资源下的 AI Agent 工程师实战路径

## 一、 资源评估与策略调整

### 1. 本地硬件能力边界 (32GB RAM)
*   **能做什么**：
    *   运行 **7B - 14B** 参数的量化模型 (Q4_KM/Q5_KM)。例如：`Llama-3.3-8B`, `Qwen-2.5-14B`, `Gemma-2-9B`。
    *   这些模型在 32GB RAM 下运行流畅，响应速度在 10-30 tokens/s，足以用于调试 Prompt 和测试逻辑。
    *   运行 LangGraph、MCP Server、SQLite、Streamlit 等 Python 应用毫无压力。
*   **不能做什么**：
    *   运行 70B+ 模型（需要 40GB+ VRAM/RAM）。
    *   进行模型微调 (Fine-tuning)。
    *   高并发多用户服务。

### 2. “白嫖”云资源策略 (2026 年版)
既然本地算力有限，我们需要利用云端的免费额度来弥补。

| 资源类型 | 推荐平台 (2026 现状) | 白嫖策略/额度 | 用途 |
| :--- | :--- | :--- | :--- |
| **LLM API (推理)** | **Groq** | 免费层每秒限制较高，支持 Llama 3.3, Mixtral 等。 | **主力调试**。速度极快，适合测试 Agent 逻辑，不消耗本地 RAM。 |
| | **Google Gemini API** | 免费层每月有一定额度 (Gemini 1.5 Pro/Flash)。 | **长上下文测试**。Flash 模型便宜且快，适合处理大量日志数据。 |
| | **OpenRouter** | 聚合平台，提供少量免费试用额度。 | 对比不同模型的效果。 |
| **GPU 算力 (临时)** | **Kaggle Kernels** | 每周 30 小时免费 T4/P100 GPU。 | 如果需要运行稍大的模型或做简单的向量嵌入加速。 |
| | **Google Colab (Free)** | 每日有限时长的 T4 GPU。 | 运行 Jupyter Notebook 实验，测试 MCP Server 连接远程 DB。 |
| | **Hugging Face Spaces** | 免费 CPU/轻量 GPU 托管。 | **部署你的 Streamlit 演示项目**，让面试官能在线访问。 |
| **代码托管** | **GitHub** | 无限私有/公有仓库。 | 存放代码、MCP Server 定义、Eval 数据集。 |

**核心原则**：
*   **开发阶段**：使用 **Groq API** (极速) 或 **本地 Ollama 8B/14B** (隐私/离线)。
*   **演示阶段**：将前端部署在 **Hugging Face Spaces**，后端逻辑调用免费 API。
*   **数据存储**：全部本地 SQLite，零成本。

---

## 二、 技术栈精简 (针对 32GB RAM)

1.  **本地推理引擎**: **Ollama** (必装，管理本地小模型)。
2.  **云端推理替代**: **LangChain Groq Chat Model** (作为主要调试后端)。
3.  **编排框架**: **LangGraph** (纯 Python，内存占用极低)。
4.  **协议层**: **MCP Python SDK** (轻量级)。
5.  **数据层**: **SQLite** + **ChromaDB** (本地向量库，32GB 足够处理百万级向量)。
6.  **UI/部署**: **Streamlit** (本地开发) -> **Hugging Face Spaces** (免费部署)。

---

## 三、 12 周“白嫖”实战学习计划

### 第一阶段：本地环境与云端 API 对接 (Week 1-4)
**目标**：在不占用大量本地资源的情况下，跑通 Agent 基本流程。

*   **Week 1: Ollama 本地基线**
    *   [ ] 安装 Ollama。
    *   [ ] 拉取 `qwen2.5:14b` (中文能力强，14B 在 32GB RAM 下可运行，约占用 9-10GB RAM)。
    *   [ ] **实操**：编写 Python 脚本，通过 `ollama.chat` 接口对话。监控内存占用，确保系统不卡顿。
*   **Week 2: 接入 Groq 免费 API**
    *   [ ] 注册 Groq Cloud，获取 API Key。
    *   [ ] 安装 `langchain-groq`。
    *   [ ] **实操**：将 Week 1 的脚本后端切换为 Groq (`llama-3.3-70b`)。体验毫秒级响应。
    *   [ ] **对比**：记录本地 Qwen-14B 与云端 Llama-3.3-70B 在相同 Prompt 下的表现差异。
*   **Week 3: MCP Server 本地开发**
    *   [ ] 学习 `fastmcp`。
    *   [ ] **实操**：编写一个 MCP Server，读取本地 CSV 文件 (模拟日志)。
    *   [ ] 使用 `mcp-inspector` 测试。注意：MCP Server 本身几乎不占内存。
*   **Week 4: 构建模拟数据集**
    *   [ ] 编写 Python 脚本生成 10,000 条模拟嵌入式日志。
    *   [ ] 存入 SQLite。
    *   [ ] **关键**：数据完全本地，无泄露风险，无成本。

### 第二阶段：LangGraph 编排与混合部署 (Week 5-8)
**目标**：构建复杂逻辑，利用云端算力解决本地瓶颈。

*   **Week 5: LangGraph 基础状态机**
    *   [ ] 使用 **Groq API** 作为 LLM 节点，降低本地负载。
    *   [ ] **实操**：构建“日志分类器”。输入日志 -> LLM 分类 -> 输出 JSON。
*   **Week 6: 引入 MCP Tool 调用**
    *   [ ] **实操**：Agent 调用 Week 3 写的 MCP Server，查询 SQLite 中的特定错误码统计。
    *   [ ] 实现“推理 -> 查库 -> 总结”闭环。
*   **Week 7: 向量检索 (RAG) 本地化**
    *   [ ] 安装 `chromadb` (本地模式)。
    *   [ ] **实操**：将 10,000 条日志的 Embedding 存入 Chroma。
    *   [ ] **注意**：Embedding 模型使用 `all-MiniLM-L6-v2` (极小，CPU 即可运行)，不占用大显存。
    *   [ ] Agent 先查向量库，再调用 LLM 总结。
*   **Week 8: 持久化与中断**
    *   [ ] 使用 `SqliteSaver` 保存状态。
    *   [ ] **实操**：模拟“人工确认”环节。当检测到 Critical 错误时，暂停并等待 CLI 输入。

### 第三阶段：评估、优化与免费部署 (Week 9-12)
**目标**：打造可在线演示的作品集。

*   **Week 9: 自动化评估 (Eval)**
    *   [ ] 使用 `deepeval`。
    *   [ ] **实操**：在本地运行评估脚本，调用 Groq API 进行批量测试。
    *   [ ] 生成 HTML 报告。
*   **Week 10: Streamlit 本地开发**
    *   [ ] 编写 UI，展示 Agent 思考过程。
    *   [ ] **优化**：添加缓存机制，避免重复调用 API 浪费免费额度。
*   **Week 11: 部署到 Hugging Face Spaces**
    *   [ ] 注册 Hugging Face。
    *   [ ] 创建 Space，选择 **Streamlit** 模板。
    *   [ ] **关键技巧**：
        *   代码上传到 GitHub。
        *   在 HF Space 中配置 `requirements.txt`。
        *   **API Key 管理**：在 HF Space 的 "Settings" -> "Variables and Secrets" 中设置 `GROQ_API_KEY`。
        *   **注意**：HF 免费层是 CPU，运行速度慢。建议只部署 UI 和轻量逻辑，重计算依然调用 Groq/Gemini API。
*   **Week 12: 最终整合与文档**
    *   [ ] 完善 README，包含架构图、白嫖资源列表、性能对比数据。
    *   [ ] 录制演示视频 (因为 HF 免费层可能休眠，视频更可靠)。

---

## 四、 避坑指南 (32GB RAM 特供)

1.  **严禁同时运行多个大模型**：
    *   Ollama 运行时，关闭浏览器中不必要的标签页 (Chrome 吃内存)。
    *   不要同时运行 Docker 容器，除非必要。
2.  **Embedding 模型选小的**：
    *   不要用 `text-embedding-ada-002` (API 收费) 或 `bge-large` (本地太大)。
    *   **推荐**：`sentence-transformers/all-MiniLM-L6-v2`。只有 80MB，CPU 跑得飞快，效果对于日志分类足够好。
3.  **向量库选型**：
    *   不要用 Milvus/Qdrant (需要独立服务，吃内存)。
    *   **推荐**：`ChromaDB` (Persistent Client 模式) 或 `FAISS` (纯内存/文件)。它们可以直接嵌入 Python 进程，零额外开销。
4.  **API 额度管理**：
    *   Groq 免费层有 RPM (每分钟请求数) 限制。
    *   **策略**：在代码中加入 `time.sleep()` 或重试机制，避免触发限流。
    *   **备用**：注册 Google Gemini API 作为备选，当 Groq 限流时自动切换。

---

## 五、 最终作品集架构 (面试展示用)

**项目名称**：**LocalLogAnalyst (离线优先的智能日志诊断助手)**

*   **前端**: Streamlit (部署在 Hugging Face Spaces，免费)。
*   **后端逻辑**:
    *   **Orchestrator**: LangGraph (运行在 HF Space 的 CPU 上，轻量)。
    *   **LLM Brain**: 调用 **Groq API** (Llama-3.3-70b，免费，极速)。
    *   **Memory**: ChromaDB (本地文件模式，随代码上传到 HF)。
    *   **Tools**: 自定义 Python 函数 (模拟 MCP Tool，解析 SQLite)。
*   **数据**: 内置 10k 条模拟嵌入式日志 (CSV/SQLite)。
*   **亮点**:
    *   **零成本部署**。
    *   **混合架构**：本地轻量编排 + 云端强力推理。
    *   **安全合规**：无敏感数据上传，所有测试数据均为合成数据。

---

## 六、 立即行动清单 (Today)

1.  **清理内存**：关闭不必要的软件，确保 32GB RAM 可用空间 > 20GB。
2.  **安装 Ollama**：`curl -fsSL https://ollama.com/install.sh | sh` (Mac/Linux) 或下载 Windows 版。
3.  **拉取小模型**：`ollama pull qwen2.5:14b` (观察下载速度和内存占用)。
4.  **注册 Groq**：[groq.com](https://groq.com/)，获取 API Key。
5.  **创建 GitHub Repo**：`local-agent-lab`，初始化 README。
