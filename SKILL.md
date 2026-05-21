---
name: oh-my-harness-init
description: 用于初始化新项目、使仓库具备 Agent 就绪能力，或添加架构层级边界 —— 生成 AGENTS.md、docs/ 知识体系、边界测试、Lint 规则、CI 流水线和垃圾回收脚本
triggers:
  - oh-my-harness-init
  - harness
  - harness-init
  - harness 工程
  - agent 就绪
  - 架构边界
  - 层级约束
  - 初始化 harness
  - 使仓库 agent 就绪
  - 设置架构
  - 初始化项目规范
  - 搭建工程脚手架
  - 规范代码结构
  - 添加架构约束
argument-hint: "[full|N|N-M]"
metadata:
  author: tangling
  version: "1.2.0"
  language: zh-CN
  based-on: "harness-init@1.1.0"
---

# Oh My Harness Init（中文版 v1.1）

<Purpose>
使用 OpenAI 的 harness 工程脚手架初始化仓库：包括 AGENTS.md 导航地图、docs/ 知识体系、架构层级约束、黄金原则、垃圾回收机制，以及静态/动态上下文策略。

这是 harness 工程的**仓库初始化子集**。运行时反馈循环、Agent 审查循环和可观测性基础设施搭建不在本范围内。如果仓库已有可观测性（日志、指标、追踪），本 Skill 会将其作为动态上下文读取，但不负责搭建。

来源：OpenAI《Harness engineering: leveraging Codex in an agent-first world》（2026-02-11）
</Purpose>

<Why_This_Exists>
AI Agent 只能处理它能看到的东西。如果没有结构化文档、机械约束和清晰的层级边界，Agent 会做出不一致的架构决策、引入 import 违规，并产生偏离规范的代码。本 Skill 通过预先设计好环境，确保每次 Agent 会话都从一张地图开始，而不是从一片空白开始。
</Why_This_Exists>

<Use_When>
- 从零开始创建新项目
- 首次将现有仓库改造为 Agent 就绪状态
- 向代码库添加架构层级边界
- 用户说"harness"、"初始化 harness"、"使这个仓库 agent 就绪"、"设置架构"、"规范代码结构"
- 在缺少 AGENTS.md + docs/ 的仓库中开展重大功能开发之前
- 仓库已有 CI，但想单独添加架构边界测试（可指定 `3-4` 阶段运行）
</Use_When>

<Do_Not_Use_When>
- 仓库已完整具备 AGENTS.md + docs/architecture/LAYERS.md + 边界测试 —— 直接使用现有 harness（若只缺少其中某项，可指定对应阶段补充，无需全量重跑）
- 用户需要按目录分层的 AGENTS.md —— 使用按目录初始化的工具
- 用户需要运行时可观测性 / Agent 审查循环 —— 超出本 Skill 范围
- 快速修复 Bug 或小型功能 —— 直接处理即可
- 用户想探索想法或头脑风暴 —— 本 Skill 用于结构化脚手架，不是创意探索
</Do_Not_Use_When>

<Principles>
1. **给 Agent 地图而非百科全书** —— AGENTS.md 约 100 行，渐进式披露
2. **Agent 看不到的等于不存在** —— 所有知识必须机器可读并存在于仓库中
3. **用机械手段强制架构** —— 靠 Lint 和测试，而不是 Markdown 说明
4. **每条报错都是 Agent 的上下文** —— 报错输出中必须包含修复指引
5. **朴素技术更优** —— 可组合、稳定、AI 训练充分的 API
6. **架构测试是棘轮** —— KNOWN_VIOLATIONS 只能缩小，不能增大（如棘轮只能单向转动，已修复的违规不得重现）
7. **高吞吐下纠错比等待更便宜** —— Agent 驱动的仓库应减少阻塞式门控：PR 保持短周期，测试偶发失败用跟进 PR 修复而非无限阻塞当前任务，等待的成本远高于后续修正的成本（来自 OpenAI 实践）
</Principles>

<Execution_Policy>
- 阶段 0（发现）是**必须的** —— 永远不要跳过，永远不要假设技术栈
- **参数解析：** `full` = 全部阶段。`N` = 仅执行阶段 N。`N-M` = 执行阶段 N 到 M。无参数 = 交互式（询问用户要设置什么）。无论何种参数，阶段 0 始终执行。
- 先读后写 —— 匹配现有代码风格和规范
- 文档重组使用 `git mv` 以保留历史记录
- 如果已有违规，新的 Lint 规则先以警告方式运行 —— 不要破坏构建
- 独立的阶段可以并行委托（参见下方阶段依赖图）
- 长时间操作（npm install、测试套件）在后台运行
- 如果仓库有现有工作，使用功能分支（`feat/harness-engineering`）
- **阶段检查点：** 每个阶段完成后，验证输出是否存在（文件已创建 + 适用的测试/Lint 通过）。记录已完成的阶段，以便中断后可以恢复。
- **失败处理：** 如果某个阶段失败，跳过它，报告失败原因，然后继续下一个独立阶段。不要因为单个阶段失败而中止整个流程。
- **验证证据：** "阶段完成" = 输出文件存在 **且** 相关测试/Lint 通过。仅文件存在不足以确认完成。
- **将学习持续升级为约束：** 同一问题重复出现时，按阶梯升级约束强度（Docs → Tests → Lint → 自动化）—— 参见 `references/golden-principles-guide.md` 中的完整升级阶梯说明。

**阶段依赖关系与并行执行图：**

```
阶段 0（必须首先完成，不可跳过）
           │
      ┌────┴────┐
   阶段 1    阶段 2      ← 可并行（均依赖阶段 0 的发现结果）
      └────┬────┘
           │
      ┌────┴────┐
   阶段 3    阶段 4      ← 可并行（均依赖阶段 2 的层级定义）
      └────┬────┘
           │
      ┌────┼────┐
   阶段 5  阶段 6  阶段 7  ← 三者均可并行（阶段 5/6/7 互不依赖）
```
</Execution_Policy>

<Steps>

**阶段 0 — 发现**（NEVER SKIP）

   **第一步：判断仓库类型（决定后续所有策略）**

   检查仓库是否有现有代码（`git log --oneline | wc -l` 是否 > 5，是否有 `src/` 或 `lib/` 目录）：

   > **Greenfield（新建/空仓库）** —— 从零开始，无历史包袱
   > - 询问用户在构建什么：产品类型、目标技术栈、团队偏好
   > - 基于用户回答 + `Read references/layer-templates.md` 推荐最适合的层级结构
   > - 所有文档从零生成，无需担心兼容现有代码
   > - Lint 规则直接以强制模式（而非警告模式）启用，因为没有历史违规
   > - CI 可以从一开始就设为强制门控

   > **Brownfield（改造现有仓库）** —— 适配为主，不破坏现有构建
   > - 扫描现有代码结构，发现而不是假设层级
   > - 先运行边界测试建立 KNOWN_VIOLATIONS 基线，再启用棘轮
   > - Lint 规则先以仅警告模式运行（避免立即破坏构建），逐步收紧
   > - 文档优先补充缺失内容，而不是重写现有文档
   > - 使用 `feat/harness-engineering` 功能分支隔离变更

   **第二步：技术栈检测**
   - 检测语言、框架、包管理器、构建工具、测试运行器、Linter
   - 映射目录结构（最大深度 3，排除 node_modules/.git）
   - 检查现有文档：AGENTS.md、CLAUDE.md、docs/、CI 工作流、测试、Lint 配置

   **第三步：架构层级识别**
   - 通过读取实际 import 模式识别架构层级 —— `Read references/layer-templates.md` 查看常见模型
   - **Greenfield**：与用户确认推荐层级是否符合意图
   - **Brownfield**：与用户确认扫描出的实际层级是否准确

   **第四步：注入动态上下文**
   - `Read references/context-strategy.md` 查看完整信号表
   - 收集：git status、LSP 诊断信息、CI 最近运行状态

   **第五步：确认三个关键决策点再继续**
   - ① 层级映射是否符合实际（Greenfield 是否认可推荐；Brownfield 是否认可发现）
   - ② 完整设置（full）还是指定阶段（N 或 N-M）
   - ③ Brownfield 专属：现有 CI/Lint 违规处理策略——警告优先还是立即修复

---

**阶段 1 — AGENTS.md**（约 100 行，导航地图）

   - `Read references/agents-md-template.md` 查看模板
   - 从阶段 0 的发现结果填充 —— 不要凭空捏造，如实反映现有情况
   - 指向 docs/ 获取详情，不要在 AGENTS.md 中内联

---

**阶段 2 — docs/ 知识体系**

   > **〔必需〕** 每个 Agent 就绪仓库都必须有

   - 创建：仓库根目录的 `ARCHITECTURE.md`（顶级领域地图，约 30 行，指向 LAYERS.md）
   - 创建：`docs/architecture/LAYERS.md`（权威层级结构 + 修复指引）
   - 创建：`docs/golden-principles/` —— `Read references/golden-principles-guide.md` 了解写法
   - 创建：`docs/SECURITY.md` —— `Read references/security-template.md` 查看模板和排除规则

   > **〔推荐〕** 多人协作或生命周期超过 3 个月的项目

   - 创建：`docs/guides/`（设置、测试、部署 —— 仅创建相关内容）
   - 创建：`docs/exec-plans/` —— `Read references/exec-plan-template.md` 查看标准（active/ + completed/ 子目录）
   - 创建：`docs/design-docs/`，包含 `index.md`（ADR 索引）和 `core-beliefs.md`（不可违背的决策）
   - 创建：`docs/references/`（为 LLM 友好格式整理的外部库文档，例如 `{library}-llms.txt`）
   - 创建：`docs/DESIGN.md`、`docs/PLANS.md`、`docs/QUALITY_SCORE.md`

   > **〔条件性〕** 由阶段 0 发现结果决定是否创建

   - `docs/RELIABILITY.md` —— 针对有 SLA 的服务（服务级别协议、错误预算、韧性模式）
   - `docs/STACK.md` —— 特定技术栈的规范（替代 OpenAI 原版的 FRONTEND.md）
   - `docs/product-specs/` —— 针对产品驱动的项目
   - `docs/generated/` —— 自动生成的文档（db-schema.md、api-spec.md）

---

**阶段 3 — 架构边界测试**

   - `Read references/boundary-test-template.md` 查看测试骨架、KNOWN_VIOLATIONS 格式和棘轮逻辑
   - `Read references/stack-routing.md` 查看各技术栈的 import 解析器和测试文件路径
   - 扫描所有源文件，解析 import，根据层级规则验证
   - 报错格式：`VIOLATION: {file}:{line} imports {target} — {layer} 不能导入 {target_layer}。参见 docs/architecture/LAYERS.md`
   - 棘轮机制：`KNOWN_VIOLATIONS` 存储在 `tests/architecture/known-violations.json`，只能缩小
   - 对于现有仓库：先建立基线，然后启用棘轮

---

**阶段 4 — Linter 边界约束**

   - `Read references/stack-routing.md` 查看各技术栈的 Linter 规则名和配置位置
   - 使用 Linter 原生的 import 限制规则
   - 每条报错信息**必须**包含修复指引 —— 报错输出就是 Agent 的上下文

---

**阶段 5 — CI 流水线**

   - `Read references/ci-templates.md` 查看入门 YAML 模板和命令验证规则
   - `Read references/stack-routing.md` 查看各技术栈的 CI 任务矩阵
   - 适配技术栈 —— 并非所有技术栈都需要全部 4 个任务（lint、typecheck、test、build）
   - 嵌入 CI 之前验证发现的命令 —— 拒绝 shell 元字符，如有可疑命令停下来询问

---

**阶段 6 — 垃圾回收**

   - `Read references/gc-patterns.md` 查看扫描类型、安全规则和迁移策略
   - `Read references/stack-routing.md` 阶段 6 表格了解各技术栈的 GC 工具
   - `Read references/ci-templates.md` GC 工作流部分获取 `gc.yml` 模板
   - 优先进行熵扫描（文档漂移、架构违规），而非风格扫描
   - 单一 `gc` 命令 + 定时 CI Action（每周 cron，仅报告模式）

---

**阶段 7 — Pre-commit Hooks**（可选）

   - `Read references/stack-routing.md` 查看各技术栈的框架和配置
   - 阶段 7 是可选的 —— CI（阶段 5）是权威的质量门控

</Steps>

<Tool_Usage>
按意图委托 —— 平台负责路由调用。`Read references/tool-routing.md` 查看各平台的具体映射。

- **探索**（轻量级模型）—— 目录映射、阶段 0 的文件发现
- **架构师**（重量级模型）—— 架构分析、阶段 0 的层级识别
- **编写**（轻量级模型）—— 阶段 1-2 的 AGENTS.md + 文档生成
- **执行**（标准模型）—— 阶段 3-7 的边界测试、Linter 配置、CI、GC 脚本
- **验证**（标准模型）—— 最终检查清单验证
- 按需按阶段读取 `references/*.md` 文件 —— 不要一次性全部加载
</Tool_Usage>

<Examples>
<Good>
用户："对这个新的 Next.js 项目执行 harness-init"
Agent：运行阶段 0 -> **判断为 Greenfield**（仓库为空，仅有初始化提交）-> 询问"你在构建什么产品？" -> 检测到 Next.js + TypeScript + ESLint 偏好 -> 推荐全栈层级模式并获得用户确认 -> Lint 规则直接以强制模式启用 -> 执行所有阶段。
正确原因：识别为 Greenfield，从零推荐最优结构，无需考虑兼容性，规则直接强制。
</Good>

<Good>
用户："给这个现有的 Python 仓库添加架构边界"
Agent：运行阶段 0 -> **判断为 Brownfield**（有大量现有代码）-> 扫描实际 import 模式，发现 200+ 个现有违规 -> 与用户确认层级映射 -> 建立 KNOWN_VIOLATIONS 基线 -> 将 Lint 规则设为仅警告 -> 创建棘轮测试 -> 询问用户关于收敛时间线。
正确原因：识别为 Brownfield 后采用"适配而非覆盖"策略，不破坏现有构建，使用棘轮机制逐步收敛。
</Good>

<Good>
用户："harness-init 3-4"
Agent：运行阶段 0（始终执行）-> 检测到 Go + golangci-lint -> 读取 stack-routing.md -> 创建使用 `go/parser` + depguard 配置的边界测试 -> 跳过阶段 1-2、5-7。
正确原因：遵守阶段参数，仍然运行发现阶段，使用决策表选择工具。
</Good>

<Good>
用户："初始化这个 Spring Boot 项目的架构规范"
Agent：运行阶段 0 -> 检测到 Java + Maven + Spring Boot -> 识别 controller/service/repository/domain 层级 -> 读取 layer-templates.md 后端 API 模型 -> 使用 ArchUnit 创建边界测试 -> 生成 Maven 插件驱动的 CI（checkstyle + maven test + maven package）。
正确原因：适配 Java 生态，选用 ArchUnit 而不是 JS 测试框架。
</Good>

<Good>
用户："对这个 turborepo monorepo 执行 harness-init"
Agent：运行阶段 0 -> 检测到 packages/ 结构 -> 读取 layer-templates.md 的 monorepo 模型 -> 映射跨包依赖 -> 创建每个包的边界测试 + 共享 CI 矩阵。
正确原因：适配 monorepo 结构，而不是强套单应用模式。
</Good>

<Bad>
用户："harness-init"
Agent：立即用 React/TypeScript 模板创建 AGENTS.md，没有先读取仓库。
错误原因：跳过了阶段 0 发现。假设了技术栈而不是检测。
</Bad>

<Bad>
用户："使这个仓库 agent 就绪"（仓库有 500 个 Lint 违规）
Agent：添加严格的 Lint 规则，立即在所有 500 个违规上触发 CI 失败。
错误原因：没有建立基线。破坏了构建。应该先以警告模式运行，然后再启用棘轮。
</Bad>

<Bad>
用户："帮我完善这个项目的安全文档"
Agent：直接在 docs/SECURITY.md 中填入具体的环境变量名、数据库 IP 和已知漏洞详情。
错误原因：违反 security-template.md 的排除规则。SECURITY.md 只能包含通用描述，不能包含具体凭证信息。应先 `Read references/security-template.md` 查看排除规则再动笔。
</Bad>

<Bad>
用户："生成这个项目的完整文档体系"
Agent：不区分 Required/Recommended/Conditional，把 docs/ 下所有可能的文档全部创建（RELIABILITY.md、STACK.md、product-specs/ 等）。
错误原因：违反条件性原则。RELIABILITY.md 只对有 SLA 的服务适用，product-specs/ 只对产品驱动项目适用。应先由阶段 0 的发现结果决定是否创建条件性文档，而不是全量生成。
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- 如果技术栈检测模糊（多个包管理器、框架不明确），**停下来询问**
- 如果没有清晰的目录结构（无 src/ 或 lib/ 的扁平仓库），**停下来询问**
- 如果现有 AGENTS.md 或 docs/ 与 harness 结构冲突，**停下来询问**
- 如果 Linter/测试运行器无法安装（权限问题、版本不兼容），**停下来报告**
- 如果 `gh` CLI、LSP 或会话状态不可用，**优雅降级** —— 跳过那些动态上下文信号，记录跳过了什么
- **永远不要强行**套用不适合实际代码库的层级结构
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] 阶段 0 发现完成（技术栈已检测，层级已识别，用户已确认）`[阶段 0]`
- [ ] AGENTS.md 存在于仓库根目录（约 100 行，是索引而非百科全书）`[阶段 1]`
- [ ] docs/architecture/LAYERS.md 存在，包含层级图 + 修复指引 `[阶段 2]`
- [ ] 黄金原则文档 ≥ 2 个（`ls docs/golden-principles/*.md | wc -l` 结果 ≥ 2），每个含 DO/DON'T 示例和执行方式说明 `[阶段 2]`
- [ ] 架构边界测试存在并通过（如是现有仓库，包含 KNOWN_VIOLATIONS）`[阶段 3]`
- [ ] Linter 规则强制 import 边界，报错信息中包含修复指引 `[阶段 4]`
- [ ] CI 流水线运行 lint + test（至少）`[阶段 5]`
- [ ] GC 执行命令存在（`npm run gc` / `make gc` / 等价命令）`[阶段 6]`
- [ ] Pre-commit hooks 已配置（可选，仅在团队需要本地约束时）`[阶段 7]`
- [ ] 所有新文件已提交到功能分支（`git status` 显示无未提交变更）`[全局]`
- [ ] 测试套件、Linter 和 GC 脚本已在本地验证可成功运行 `[全局]`
</Final_Checklist>

<Advanced>
## 目标文件结构

```
项目根目录/
├── AGENTS.md                          # 约 100 行，导航地图              [必需]
├── ARCHITECTURE.md                    # 顶级领域地图                    [必需]
├── docs/
│   ├── architecture/
│   │   └── LAYERS.md                  # 层级结构 + 约束说明             [必需]
│   ├── golden-principles/             # DO/DON'T 模式，每个 30-60 行   [必需]
│   ├── SECURITY.md                    # 认证、密钥、威胁模型             [必需]
│   ├── guides/                        # 设置、测试、部署指南             [推荐]
│   ├── exec-plans/                    # 执行计划生命周期                 [推荐]
│   │   ├── active/
│   │   ├── completed/
│   │   └── tech-debt-tracker.md
│   ├── design-docs/                   # 架构决策记录 ADR                [推荐]
│   │   ├── index.md
│   │   ├── core-beliefs.md
│   │   └── {NNNN-标题}.md
│   ├── references/                    # LLM 友好格式的外部文档           [推荐]
│   │   └── {库名}-llms.txt
│   ├── DESIGN.md                      # 设计哲学                       [推荐]
│   ├── PLANS.md                       # 执行计划概览                    [推荐]
│   ├── QUALITY_SCORE.md               # 各领域质量评分                  [推荐]
│   ├── RELIABILITY.md                 # SLA、错误预算（仅服务类项目）    [条件性]
│   ├── STACK.md                       # 技术栈规范                     [条件性]
│   ├── product-specs/                 # 产品规格                       [条件性]
│   └── generated/                     # 自动生成的文档                  [条件性]
│       └── {db-schema,api-spec}.md
├── scripts/gc/                        # 垃圾回收脚本
├── tests/architecture/
│   └── boundary.test.*                # 机械化层级约束
└── .github/workflows/
    ├── ci.yml                         # lint + typecheck + test + build
    └── gc.yml                         # 每周熵扫描
```

## 各阶段产出快速参考

| 阶段   | 产出文件/制品                                             | 验证方式                                  |
|--------|-----------------------------------------------------------|-------------------------------------------|
| 阶段 0 | 无文件（内存中的技术栈档案）                              | 向用户复述检测到的技术栈和层级映射        |
| 阶段 1 | `AGENTS.md`                                               | 文件存在，约 100 行，包含所有必需章节     |
| 阶段 2 | `ARCHITECTURE.md`、`docs/architecture/LAYERS.md`、`docs/golden-principles/*.md`、`docs/SECURITY.md` | 必需文件全部存在                          |
| 阶段 3 | `tests/architecture/boundary.test.*`、`tests/architecture/known-violations.json` | 边界测试通过（含已知违规基线）            |
| 阶段 4 | `.eslintrc` / `pyproject.toml` / `.golangci.yml` 等       | `{lint 命令}` 通过，报错含修复指引        |
| 阶段 5 | `.github/workflows/ci.yml`（或等价 CI 配置）              | CI 流水线在仓库中触发并通过               |
| 阶段 6 | `scripts/gc/`、`.github/workflows/gc.yml`                 | `{gc 命令}` 可执行，GC Action 可手动触发  |
| 阶段 7 | `.husky/` / `.pre-commit-config.yaml` / 等                | 执行 `git commit` 时 hook 正常触发        |

## 参考文件（按阶段顺序）

按需按阶段读取，不要一次性全部加载：

- `references/context-strategy.md` —— 静态与动态上下文表格 `[阶段 0]`
- `references/layer-templates.md` —— 6 种层级模型（含 Java/Spring Boot）`[阶段 0]`
- `references/agents-md-template.md` —— AGENTS.md 模板（含约束示例）`[阶段 1]`
- `references/golden-principles-guide.md` —— 黄金原则编写指南 `[阶段 2]`
- `references/security-template.md` —— SECURITY.md 模板及排除规则 `[阶段 2]`
- `references/exec-plan-template.md` —— 执行计划（docs/exec-plans/）标准 `[阶段 2]`
- `references/boundary-test-template.md` —— 测试骨架（TS/Python/Go）、KNOWN_VIOLATIONS、棘轮逻辑 `[阶段 3]`
- `references/stack-routing.md` —— 技术栈 → 阶段 3-7 工具决策表 `[阶段 3-7]`
- `references/ci-templates.md` —— GitHub Actions、GitLab、Gitee、Makefile 的入门 CI YAML `[阶段 5-6]`
- `references/gc-patterns.md` —— GC 扫描类型 + 知识新鲜度检查 + 迁移策略 `[阶段 6]`
- `references/tool-routing.md` —— 各平台工具委托映射（含 Cursor 专项说明）`[通用]`
- `references/glossary.md` —— 术语对照表（中英文概念统一定义）`[通用]`
</Advanced>
