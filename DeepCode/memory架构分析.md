
# 1️⃣ DeepCode 会把哪些内容写入 memory？

DeepCode 的 memory 保存的不是 RL 轨迹，而是**“对代码生成有用的上下文与历史”**，包括：

### ✅ 会写入 memory 的内容（4类）

1. **Agent 之间的对话 / 任务规划历史（关键摘要）**

   * 用于维持多轮任务规划与对话的一致性
   * 不会保留全部原文，而是 *摘要化*

2. **代码生成过程中的关键实现模式（implementation patterns）**

   * 例如生成的函数结构、常用代码片段、模块依赖
   * 在 README 中明确写明 memory 支持 *instant retrieval of implementation patterns*

3. **与项目结构相关的长期信息（长上下文）**

   * 项目中已创建的文件、模块方案、接口定义
   * 避免 agent 忘记之前写过什么文件

4. **CodeRAG 索引的语义嵌入（embeddings）**

   * DeepCode 的 RAG 系统会把部分内容写入向量存储
   * memory 本质上就是长期知识库 + 最近摘要

---

# 2️⃣ DeepCode 是怎么将内容写入 memory（做了哪些处理）？

根据 issue & README 描述，DeepCode 对 memory 写入过程包含 **3 个步骤**：

## ✦（1）内容提取（Extract）

从 Agent 任务中提取：

* 最新代码
* 最新规划
* 最新 agent 对话的关键意图句
* 有代表性的实现模式
  （不是所有消息都保存）

## ✦（2）摘要 / 压缩（Summarize / Concise Memory Optimization）

issue 中有明确日志：
**“concise memory optimization skipped; falling back to minimal history”**
说明 DeepCode会对 memory 做：

* **摘要（summarization）**
* **压缩（compression）**
* **去重（dedupe）**
* **截断 / token 限制处理**

失败时会回退到 minimal history。

## ✦（3）写入长期存储（Long-term Memory Store）

存储形式（综合 README 与工具接口）：

* 关键摘要文本
* 代码块（经裁剪处理）
* 嵌入向量（用于 CodeRAG）
* 层级（hierarchical）存储结构

---

# 3️⃣ DeepCode 什么时候会调用 memory？

根据模型架构与 tool 列表，有 3 个使用场景：

## ⏰（1）Agent 开始一个新步骤时（Planning）

例如：

* Code Planning Agent 需要知道之前已经讨论过哪些文件、模块
* Orchestrator 需要知道当前任务是否之前已完成过一部分

## ⏰（2）Code Generation Agent 生成代码时

需要从 memory 调用：

* 实现模式（patterns）
* 以前生成的相似函数
* 项目结构信息

## ⏰（3）在跨多轮会话时（Long Development Session）

这个是 memory 机制最重要的用途：

> “Maintain semantic coherence across extended development sessions.”

即：跨一长段开发过程维持一致性
（例如前 10 分钟创建了一个 API，让后续智能体不会忘记）

---

# 4️⃣ DeepCode 是怎么调用 memory？

### DeepCode 把 memory 作为一个 MCP 工具暴露

README 中列出了工具：

📌 **`read_code_mem`**

Agents 调用 memory 的方式一般是：

```
tool_call("read_code_mem", { query: "looking for file structure" })
```

memory 返回：

* 压缩后的上下文摘要
* 实现模式（patterns）
* 与 query 最相关的历史内容
* 内部向量检索（embedding search）结果

最终 memory 的调用特征是：

✔ 工具化（MCP tool）
✔ Agent 自主决定何时调用
✔ 查询式（基于 query 的 semantic retrieval）
✔ 只返回必要片段，不会全部 dump

---
