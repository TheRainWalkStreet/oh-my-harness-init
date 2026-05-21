# 术语对照表

本文件统一定义 **Oh My Harness Init** Skill 中使用的所有翻译词汇，确保 Agent 在不同文件间对同一概念的理解保持一致。

| 英文原词                         | 中文译词       | 解释                                                                                          |
|----------------------------------|----------------|-----------------------------------------------------------------------------------------------|
| Ratchet                          | 棘轮机制       | 一种"只减不增"的约束：KNOWN_VIOLATIONS 数量一旦建立基线，只能通过修复减少，不能新增。如同棘轮只允许单向转动。 |
| Golden Principles                | 黄金原则       | 存放在 `docs/golden-principles/` 的规范文档，每个文件覆盖一个主题，含 DO/DON'T 示例和执行方式说明，30-60 行。 |
| Entropy Scan                     | 熵扫描         | 检测项目"腐化程度"的扫描类型，关注文档漂移、架构违规等高价值问题，而非风格格式（风格是 Linter 的职责）。 |
| ExecPlan                         | 执行计划       | 存放在 `docs/exec-plans/` 的复杂功能实现计划文档，自包含、可重启，随进展持续更新。            |
| Boundary Test                    | 边界测试       | 自动验证代码层级 import 规则的测试，存放在 `tests/architecture/`，是架构约束的机械化执行手段。  |
| Known Violations                 | 已知违规基线   | `tests/architecture/known-violations.json` 文件，记录建立棘轮时存在的历史违规，棘轮约束的起点。 |
| Import Boundary                  | 导入边界       | 由层级规则定义的合法 import 范围：只能从允许的层级导入，其他均为违规。由边界测试和 Linter 共同执行。 |
| Doc-Code Drift                   | 文档-代码漂移  | `docs/` 中的描述与实际代码行为不一致的状态，通常通过比较 git log 时间戳发现。                 |
| Architecture Layers              | 架构层级       | 项目代码按职责划分的结构层次，依赖关系只能单向向下流动。                                       |
| Layer Rule                       | 层级规则       | 定义每个层级允许 import 哪些其他层级的规则集合，存储在边界测试和 Linter 配置中。               |
| Provider                         | Provider       | （保留英文）处理横切关注点（认证、遥测等）的唯一合法机制，禁止跨域直接导入。                  |
| Harness Engineering              | Harness 工程   | OpenAI 提出的 Agent 优先工程方法论，通过前置环境设计让 Agent 每次会话从有结构的起点开始。      |
| Agent-Ready                      | Agent 就绪     | 仓库具备让 AI Agent 高效理解和操作所需的全套基础设施（AGENTS.md、docs/、约束工具链）的状态。   |
| Orientation Map                  | 导航地图       | AGENTS.md 的核心定位：约 100 行的索引文件，帮助 Agent 快速定位项目中的关键信息入口。           |
| Progressive Disclosure           | 渐进式披露     | 信息架构原则：AGENTS.md 只提供入口和指针，详情存在 docs/ 中，避免在入口文件中内联所有内容。    |
| Static Context                   | 静态上下文     | 存储在仓库中、会话间不变的信息（AGENTS.md、LAYERS.md、golden-principles 等）。                |
| Dynamic Context                  | 动态上下文     | 每次会话开始时需要重新探测的实时信息（git status、CI 状态、LSP 诊断等）。                     |
| Architecture Decision Record (ADR) | 架构决策记录 | 存放在 `docs/design-docs/` 的决策文档，记录为什么做出某个架构选择，格式为 `{NNNN-标题}.md`。  |
| Garbage Collection (GC)          | 垃圾回收       | 定期运行的自动扫描脚本，检测文档漂移、架构违规等熵增现象。注意：与内存管理的 GC 无关。         |
| Tech Debt Tracker                | 技术债追踪器   | `docs/exec-plans/tech-debt-tracker.md`，记录已知但尚未修复的技术债，按优先级排列。            |
| Core Beliefs                     | 核心信念       | `docs/design-docs/core-beliefs.md`，记录项目不可违背的架构决策，Agent 不得绕过。              |
| Greenfield                        | 绿地仓库       | 新建或近乎空白的仓库。harness 初始化时可从零推荐最优结构，规则直接以强制模式启用，无历史包袱。    |
| Brownfield                        | 棕地仓库       | 有现有代码的仓库。harness 初始化时以"适配而非覆盖"为原则：扫描现有结构、建立违规基线、规则先以警告模式运行。 |
| Feature Branch                    | 功能分支       | 用于隔离 harness 工程改动的 git 分支，推荐命名 `feat/harness-engineering`，避免直接在主分支操作。 |
| Warn-only Mode                   | 仅警告模式     | 对现有仓库添加 Lint/测试规则时的初始模式：新违规只警告不阻断构建，建立基线后再切换为强制模式。  |
| Quality Score                    | 质量评分       | `docs/QUALITY_SCORE.md` 中按领域记录的代码质量等级，供团队和 Agent 了解各模块的当前健康状态。  |
