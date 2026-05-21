# 各平台工具路由

Oh My Harness Init 按**意图**委托工作 —— 平台负责路由调用。将每种意图映射到你所在平台的委托机制。

> **说明：** 下表中 "Claude Code + OMC" 列使用的 `subagent_type="oh-my-claudecode:*"` 语法是 [oh-my-claudecode](https://github.com/oh-my-claudecode) 插件的**专属**调用格式，仅在安装了该插件的 Claude Code 环境中有效。如果你使用的是原生 Claude Code、Cursor 或 Codex，请参考对应列。

## 意图 → 平台映射

| 意图          | 模型层级   | Claude Code + OMC（插件专属）                                              | Claude Code（原生）               | Codex                                   | Cursor                     |
|---------------|------------|---------------------------------------------------------------------------|-----------------------------------|-----------------------------------------|----------------------------|
| **探索**      | 轻量级     | `Agent(subagent_type="oh-my-claudecode:explore", model="<轻量模型>")`     | `Agent(subagent_type="Explore")`  | `codex exec "explore ..."` 或内联文件搜索 | 参见下方 Cursor 使用说明   |
| **架构师**    | 重量级     | `Agent(subagent_type="oh-my-claudecode:architect", model="<重量模型>")`   | `Agent(subagent_type="Plan")`     | `codex exec "analyze ..."` 或内联       | 参见下方 Cursor 使用说明   |
| **编写**      | 轻量级     | `Agent(subagent_type="oh-my-claudecode:writer", model="<轻量模型>")`      | `Agent(model="<轻量模型>")`       | 内联（直接编写）                         | 参见下方 Cursor 使用说明   |
| **执行**      | 标准       | `Agent(subagent_type="oh-my-claudecode:executor", model="<标准模型>")`    | `Agent(model="<标准模型>")`       | `codex exec "implement ..."` 或内联     | 参见下方 Cursor 使用说明   |
| **审查**      | 标准       | `Agent(subagent_type="oh-my-claudecode:reviewer", model="<标准模型>")`    | `Agent(model="<标准模型>")`       | `codex exec "review ..."` 或内联       | 参见下方 Cursor 使用说明   |
| **验证**      | 标准       | `Agent(subagent_type="oh-my-claudecode:verifier", model="<标准模型>")`    | `Agent(model="<标准模型>")`       | `codex exec "verify ..."` 或内联       | 参见下方 Cursor 使用说明   |

> **模型名说明：** 表中使用 `<轻量模型>`、`<标准模型>`、`<重量模型>` 占位。**具体模型名称请参考所用平台的最新文档**，以实际可用模型为准。模型迭代速度快，硬编码名称会很快过时。

## 模型层级

| 层级     | 用途                                 | 代表性用途场景                     |
|----------|--------------------------------------|------------------------------------|
| 轻量级   | 快速、低成本任务（文件列举、文档生成） | 探索目录、生成 AGENTS.md           |
| 标准     | 实现、审查与验证                     | 生成边界测试、审查 PR diff         |
| 重量级   | 架构决策、深度分析                   | 识别层级结构、设计 LAYERS.md       |

## Cursor 使用说明

Cursor 没有独立的子 Agent 机制，所有意图均在主对话中内联执行。以下是各意图在 Cursor 中的对应做法：

| 意图       | Cursor 中的做法                                                            |
|------------|----------------------------------------------------------------------------|
| **探索**   | 使用 `@文件名` 或 `@目录名` 引入具体上下文；使用 `@codebase` 做语义搜索   |
| **架构师** | 在对话中直接描述分析任务，引用 `@docs/architecture/LAYERS.md`              |
| **编写**   | 直接在对话中请求生成文件内容                                               |
| **执行**   | 直接在对话中请求执行具体实现任务                                           |
| **审查**   | 使用 `@git diff` 或粘贴 diff 内容，请求对照 `@docs/architecture/LAYERS.md` 审查 |
| **验证**   | 引用 `@tests/` 或请求运行检查清单                                          |

## 降级方案

如果你的平台不支持委托（无 Agent 工具、无子 Agent）：
- 在主对话中内联执行所有意图
- 将模型层级作为判断哪些任务值得投入更多推理的参考
- 本 Skill 不依赖委托也能工作 —— 只是会按顺序执行，而不是并行
