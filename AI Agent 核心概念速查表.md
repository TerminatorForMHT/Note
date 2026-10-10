# 📄 AI Agent 核心概念速查表

**更新日期**：2026-10-10
**适用对象**：具备代码与硬件验证经验，需快速构建 AI 知识体系的工程师。

## 1. 基础组件层 (The Hardware of AI)

| AI 概念 | 硬件/系统映射 | 核心定义 (2026视角) | 关键知识点 |
| :--- | :--- | :--- | :--- |
| **LLM (Large Language Model)** | **CPU / Coprocessor** | 概率性推理引擎。它不“知道”真理，只预测下一个 Token。 | **Context Window**: 类似 RAM 大小，决定一次能处理多少数据。<br>**Temperature**: 类似噪声注入，控制输出的随机性/创造性。 |
| **Token** | **Byte / Word** | LLM 处理的最小文本单位。1 Token ≈ 0.75 个英文单词或 1 个汉字。 | **Cost**: API 按 Token 计费。<br>**Limit**: 超出 Context Window 会导致“遗忘”。 |
| **Embedding (向量嵌入)** | **Hash / Feature Vector** | 将文本转换为高维向量数组。语义相似的文本，向量距离近。 | **Vector DB**: 存储向量的数据库，用于快速检索相似内容。<br>**Dimension**: 向量维度 (如 768, 1536)，类似总线位宽。 |
| **Prompt** | **Instruction Set / Microcode** | 发送给 LLM 的指令文本。包含角色、任务、约束和示例。 | **System Prompt**: 固件级配置，设定全局行为。<br>**Few-Shot**: 提供示例 (Test Vectors)，引导 LLM 模仿格式。 |

## 2. 通信与接口层 (The Bus & Protocol)

| AI 概念 | 硬件/系统映射 | 核心定义 (2026视角) | 关键知识点 |
| :--- | :--- | :--- | :--- |
| **MCP (Model Context Protocol)** | **USB-C / PCIe Bus** | **行业标准协议**。统一了 AI 应用与数据源/工具之间的连接方式。 | **Server**: 暴露数据/工具的驱动。<br>**Client**: 嵌入在 Agent 中的控制器。<br>**Transport**: STDIO (本地进程间) 或 SSE (网络)。 |
| **Tool / Skill** | **GPIO / Peripheral Register** | LLM 可以调用的外部函数。LLM 本身不能联网或查库，必须通过 Tool。 | **JSON Schema**: 工具的“数据手册”，严格定义参数类型。<br>**Description**: 工具的“功能描述”，LLM 靠它决定何时调用。 |
| **Resource** | **Memory-Mapped I/O** | 只读的数据源 (如文件、数据库行)。Agent 可以“读取”但不能“修改”。 | **URI**: 资源的地址标识。<br>**Lazy Loading**: 按需加载内容，节省 Context。 |

## 3. 编排与控制层 (The FSM & Control Logic)

| AI 概念 | 硬件/系统映射 | 核心定义 (2026视角) | 关键知识点 |
| :--- | :--- | :--- | :--- |
| **Agent Orchestration** | **Finite State Machine (FSM)** | 管理 Agent 思考、行动、观察循环的控制逻辑。 | **LangGraph**: 基于图的编排框架。<br>**Node**: 状态节点 (执行单元)。<br>**Edge**: 转移条件 (逻辑判断)。 |
| **State** | **Register File / SRAM** | 全局共享的数据结构，记录当前任务的所有上下文。 | **Persistence**: 将 State 存入磁盘 (SQLite)，实现断点续传。<br>**Immutable**: 每次更新生成新版本，便于回溯。 |
| **ReAct Pattern** | **Fetch-Decode-Execute Cycle** | **Reasoning + Acting**。LLM 先思考，再调用工具，再观察结果，再思考。 | **Loop**: 典型的 while 循环，直到满足终止条件。<br>**Max Iterations**: 防止死循环的“看门狗”。 |
| **Multi-Agent** | **SoC (System on Chip)** | 多个专用 Agent 协作完成复杂任务。 | **Supervisor**: 主控核，负责任务分发。<br>**Worker**: 从核，负责具体执行 (如搜索、编码)。 |

## 4. 记忆与存储层 (The Memory Hierarchy)

| AI 概念 | 硬件/系统映射 | 核心定义 (2026视角) | 关键知识点 |
| :--- | :--- | :--- | :--- |
| **Short-term Memory** | **L1/L2 Cache / RAM** | 当前对话的 Context Window。速度快，但容量有限，断电即失。 | **Sliding Window**: 只保留最近的 N 轮对话。<br>**Summarization**: 将旧对话压缩成摘要，腾出空间。 |
| **Long-term Memory** | **Flash / HDD / Vector DB** | 存储在向量数据库中的历史知识。容量大，持久化，但检索慢。 | **RAG (Retrieval-Augmented Generation)**: 先从 DB 检索相关片段，再喂给 LLM。<br>**Indexing**: 建立向量索引，加速检索。 |

## 5. 质量保障层 (The Verification & Validation)

| AI 概念 | 硬件/系统映射 | 核心定义 (2026视角) | 关键知识点 |
| :--- | :--- | :--- | :--- |
| **Hallucination** | **Bit Flip / Soft Error** | LLM 编造事实。由于概率本质，无法完全消除，只能抑制。 | **Grounding**: 强制 LLM 基于提供的上下文回答。<br>**Citation**: 要求 LLM 引用来源。 |
| **Eval (Evaluation)** | **Testbench / UVM** | 自动化测试框架，量化 Agent 的性能。 | **Metrics**: 准确率、忠实度、相关性。<br>**Golden Dataset**: 标准测试用例集。 |
| **Observability** | **Logic Analyzer / Trace** | 追踪 Agent 的每一步执行过程。 | **Trace**: 记录每个 Node 的输入输出、耗时、Token 消耗。<br>**Debugging**: 通过 Trace 定位是 Prompt 问题还是 Tool 问题。 |
| **Guardrails** | **ECC / Safety Interlock** | 输入/输出的过滤层，防止恶意注入或不合规内容。 | **Input Validation**: 检查用户输入是否包含攻击指令。<br>**Output Filtering**: 屏蔽敏感信息。 |
