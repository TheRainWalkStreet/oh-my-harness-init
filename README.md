# Oh My Harness Init

> **harness-init 的中文版 Skill**，基于 OpenAI 的 harness 工程方法论，为 Cursor、Claude Code、Codex 等 AI 编程工具提供仓库初始化脚手架能力。

[![版本](https://img.shields.io/badge/版本-v1.2.0-green)]()
[![基于](https://img.shields.io/badge/基于-harness--init%401.1.0-blue)](https://github.com/Gizele1/harness-init)
[![语言](https://img.shields.io/badge/语言-中文-red)]()

---

## 这是什么

`oh-my-harness-init` 是 [harness-init](https://github.com/Gizele1/harness-init) 的中文优化版本，通过 8 个阶段将任意代码仓库改造为 **Agent 就绪（Agent-Ready）** 的工程环境：

| 阶段   | 做什么                                              |
|--------|-----------------------------------------------------|
| 阶段 0 | 识别 Greenfield/Brownfield，检测技术栈，映射架构层级 |
| 阶段 1 | 生成 AGENTS.md 导航地图（约 100 行索引）             |
| 阶段 2 | 建立 docs/ 知识体系（架构文档、黄金原则、安全文档）  |
| 阶段 3 | 创建架构边界测试（棘轮机制，只减不增）               |
| 阶段 4 | 配置 Linter import 限制规则（报错含修复指引）        |
| 阶段 5 | 生成 CI 流水线（lint + typecheck + test + build）    |
| 阶段 6 | 生成垃圾回收脚本和每周定时扫描（熵管理）             |
| 阶段 7 | 配置 Pre-commit hooks（可选，本地约束）              |

---

## 安装

> 仓库地址：`https://codeup.aliyun.com/658a544bb488fff322e7e6dd/llm-application/skills/oh-my-harness-init.git`

### 第一步：克隆 Skill 仓库到本地临时目录

```bash
git clone https://codeup.aliyun.com/658a544bb488fff322e7e6dd/llm-application/skills/oh-my-harness-init.git \
  /tmp/oh-my-harness-init
```

### 第二步：将 Skill 复制到目标项目

根据你使用的 AI 工具，选择对应的安装方式：

---

#### Cursor

Cursor 使用 `.cursor/rules/` 作为 AI 规则目录（Agent Requested 模式），将 Skill 复制到**目标项目**的该目录下：

```bash
# 在你的目标项目根目录执行

mkdir -p .cursor/rules/oh-my-harness-init/references

cp /tmp/oh-my-harness-init/SKILL.md \
   .cursor/rules/oh-my-harness-init/

cp /tmp/oh-my-harness-init/references/*.md \
   .cursor/rules/oh-my-harness-init/references/
```

> **为什么是 `.cursor/rules/` 而不是 `.cursor/skills/`？**
> `.cursor/skills/` 不是 Cursor 的官方目录（那是 Claude Code 的概念），Cursor 不会识别它。
> `.cursor/rules/` 是 Cursor Rules 的官方位置，Cursor Agent 会在判断任务相关时自动读取其中的文件。

---

#### Claude Code

```bash
# 在你的目标项目根目录执行

mkdir -p .claude/skills/oh-my-harness-init/references

cp /tmp/oh-my-harness-init/SKILL.md \
   .claude/skills/oh-my-harness-init/

cp /tmp/oh-my-harness-init/references/*.md \
   .claude/skills/oh-my-harness-init/references/
```

---

#### OpenAI Codex

```bash
# 在你的目标项目根目录执行

mkdir -p .agents/skills/oh-my-harness-init/references

cp /tmp/oh-my-harness-init/SKILL.md \
   .agents/skills/oh-my-harness-init/

cp /tmp/oh-my-harness-init/references/*.md \
   .agents/skills/oh-my-harness-init/references/
```

---

#### 手动（任意 AI 工具）

直接阅读 `/tmp/oh-my-harness-init/SKILL.md`，按照其中的 8 个阶段在任意 AI 编程工具中手动执行即可。

---

### 更新 Skill 到最新版本

```bash
# 拉取最新版本
cd /tmp/oh-my-harness-init && git pull

# 重新覆盖到目标项目（以 Cursor 为例，其他平台同理）
cp /tmp/oh-my-harness-init/SKILL.md \
   /path/to/your-project/.cursor/rules/oh-my-harness-init/

cp /tmp/oh-my-harness-init/references/*.md \
   /path/to/your-project/.cursor/rules/oh-my-harness-init/references/
```

## 使用方式

安装后，在 AI 工具对话中直接说：

```
oh-my-harness-init          # 交互式 —— 询问要设置什么
oh-my-harness-init full     # 完整设置，全部 8 个阶段
oh-my-harness-init 2        # 仅执行阶段 2
oh-my-harness-init 3-4      # 执行阶段 3 到 4
```

或者用自然语言：

- "使这个仓库 agent 就绪"
- "初始化项目规范"
- "给这个 Spring Boot 项目搭建工程脚手架"
- "添加架构边界约束"

---

## 初始化完成后：如何日常使用

> **重要：** 仓库初始化只是第一步。真正的价值在于初始化后每天的开发都在 harness 范式下进行。

安装并运行 oh-my-harness-init 后，仓库里的 `AGENTS.md`、`docs/`、Lint 规则和边界测试共同构成了"常驻上下文"——AI 工具在每次会话启动时自动读取，你不需要在每次对话中重复解释项目架构。

详细的日常工作流、有效提示词模式、处理 AI 违规的方法，以及每周维护清单，请阅读：

**→ [`WORKFLOW.md`](./WORKFLOW.md)**

---

## 文件结构

```
oh-my-harness-init/
├── SKILL.md                              # 主 Skill 文件（AI 读取执行）
├── README.md                             # 本文件（从这里开始）
├── WORKFLOW.md                           # 初始化后的日常开发工作流指南
├── CHANGELOG.md                          # 版本变更记录
└── references/                           # 按需加载的参考模板
    ├── agents-md-template.md             # AGENTS.md 模板（含约束示例）
    ├── layer-templates.md                # 架构层级模板（6 种技术栈）
    ├── context-strategy.md               # 静态/动态上下文策略
    ├── golden-principles-guide.md        # 黄金原则编写指南（含升级阶梯）
    ├── security-template.md              # SECURITY.md 模板
    ├── exec-plan-template.md             # 执行计划标准
    ├── boundary-test-template.md         # 边界测试骨架（TS/Python/Go）
    ├── stack-routing.md                  # 技术栈工具决策表（阶段 3-7）
    ├── ci-templates.md                   # CI 模板（GitHub/GitLab/Makefile）
    ├── gc-patterns.md                    # 垃圾回收模式
    ├── tool-routing.md                   # 各平台工具路由映射
    └── glossary.md                       # 术语对照表
```

---

## 与原版的区别

| 维度               | harness-init（原版）     | oh-my-harness-init v1.2.0（本版）            |
|--------------------|--------------------------|----------------------------------------------|
| 语言               | 英文                     | 中文                                         |
| 发现阶段           | 统一策略                 | Greenfield / Brownfield 明确分叉策略          |
| 工程哲学           | 6 条原则                 | 7 条（新增高吞吐 Merge 哲学）                 |
| 层级模板           | 4 种技术栈               | 6 种（新增 Java/Spring Boot）                 |
| 边界测试骨架       | TS + Python              | TS + Python + Go                              |
| 黄金原则指南       | 基础模板                 | 含执行方式章节 + 升级阶梯 + 后端候选文件      |
| 工具路由           | 5 种意图                 | 6 种意图（新增 Review/审查）                  |
| 错误示例           | 2 个 Bad 场景            | 4 个 Bad 场景                                 |
| 术语管理           | 无                       | 专用术语对照表（33 个术语）                   |
| 版本记录           | 无                       | CHANGELOG.md                                  |

---

## 许可

MIT — 基于 [harness-init](https://github.com/Gizele1/harness-init)（MIT）构建。
