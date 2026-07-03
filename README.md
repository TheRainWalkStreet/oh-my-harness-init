# Harness Normalize

`harness-normalize` 是一个中文 Codex/Cursor/Claude Code skill，用于把**已有项目**规范化为适合 AI Agent 长期协作的 harness 知识库。

它不负责创建新项目、不生成业务代码、不改造 CI/Lint/Pre-commit。它只做一件事：基于项目真实代码和配置，生成或整理一套精炼、可维护、可提交到 Git 的项目知识结构。

## 适用场景

使用它处理：

- 已经搭好开发框架、能运行但还没写业务代码的项目。
- 正在开发的业务项目。
- 已经开发完成、需要补齐 Agent 协作上下文的项目。
- 前后端在同一工作空间中的一体化项目。
- 需要中文 `AGENTS.md`、架构说明、工程规范、开发规范、提交规范、安全边界和执行计划体系的项目。
- 已有 Lite harness 知识库、需要刷新或升级到 Standard 的项目。

不使用它处理：

- 空仓库或尚未确定技术栈的项目。
- 功能开发、Bug 修复、代码重构。
- CI、lint、架构边界测试、垃圾回收脚本、pre-commit hooks 生成。
- 大而全的文档站点生成。

## 输出结构

skill 会根据项目事实选择 Lite、Standard 或 Extended。

### Lite

适合框架刚搭好、业务代码很少的小型项目。

```text
AGENTS.md
ARCHITECTURE.md
docs/
├── ENGINEERING.md
├── SECURITY.md
└── standards/
    ├── coding.md
    └── commits.md
```

### Standard

适合大多数企业级已有项目。

```text
AGENTS.md
ARCHITECTURE.md
docs/
├── ENGINEERING.md
├── SECURITY.md
├── QUALITY_SCORE.md
├── standards/
│   ├── coding.md
│   └── commits.md
├── design-docs/
│   ├── index.md
│   └── core-beliefs.md
└── exec-plans/
    ├── index.md
    ├── active/
    │   └── README.md
    ├── completed/
    │   └── README.md
    └── tech-debt.md
```

### Extended

只在项目事实触发时扩展，例如：

- `docs/generated/api-spec.md`
- `docs/generated/db-schema.md`
- `docs/references/{library-or-platform}-llms.txt`
- `docs/product-specs/index.md`
- `docs/RELIABILITY.md`
- `docs/FRONTEND.md`
- `docs/BACKEND.md`
- `docs/MOBILE.md`
- `docs/OPERATIONS.md`

没有事实触发就不创建空壳文档。

## 核心规则

- 先扫描项目事实，再生成文档。
- 所有输出默认使用中文。
- 所有生成文件默认提交到 Git 长期维护。
- 不生成 `.gitkeep`。
- 不确定内容写 `待确认`。
- 项目已有工具和规范优先。
- `AGENTS.md` 是入口地图，不是百科全书。
- 开发规范使用“通用工程规范内核 + 语言/框架适配层”，Java 项目可参考阿里巴巴 Java 开发手册，但不把它作为所有语言的默认规范。
- 提交规范默认使用 Conventional Commits；项目已有规范优先。

## 安装

先克隆 skill 仓库：

```bash
git clone https://github.com/TheRainWalkStreet/harness-normalize.git /tmp/harness-normalize
cd /tmp/harness-normalize
git checkout v1.3.0
```

### Cursor

```bash
mkdir -p .cursor/rules/harness-normalize/references
cp /tmp/harness-normalize/SKILL.md .cursor/rules/harness-normalize/
cp /tmp/harness-normalize/references/*.md .cursor/rules/harness-normalize/references/
```

### Claude Code

```bash
mkdir -p .claude/skills/harness-normalize/references
cp /tmp/harness-normalize/SKILL.md .claude/skills/harness-normalize/
cp /tmp/harness-normalize/references/*.md .claude/skills/harness-normalize/references/
```

### OpenAI Codex

```bash
mkdir -p .agents/skills/harness-normalize/references
cp /tmp/harness-normalize/SKILL.md .agents/skills/harness-normalize/
cp /tmp/harness-normalize/references/*.md .agents/skills/harness-normalize/references/
```

## 使用方式

在已有项目中对 AI 工具说：

```text
使用 harness-normalize 规范化这个已有项目
为这个项目生成 harness 知识库
给这个项目补齐 AGENTS.md、架构说明和开发规范
让这个前后端一体项目适合 Agent 长期维护
```

skill 会先扫描项目，再说明当前 harness 状态、推荐运行模式、结构档位和将创建/更新的文件。

## 刷新与升级

`harness-normalize` 支持三种运行模式：

- Initialize：项目还没有 harness 知识库时，首次生成。
- Refresh：项目已有 Lite 或 Standard 知识库时，增量刷新。
- Upgrade：Lite 项目已经进入真实开发阶段时，建议升级到 Standard。

Refresh 不是全量重跑：

- 保留仍然准确的内容。
- 用仓库事实修正过期内容。
- 无法确认的信息标记为 `待确认`。
- 不静默覆盖用户已有知识。

Lite 项目出现以下信号时，建议升级 Standard：

- 已出现清晰业务模块或领域概念。
- 已有稳定测试、lint、build 流程。
- 出现多人协作迹象。
- 出现技术债或复杂改造需求。
- 有架构决策需要记录。
- 出现生产部署、外部 API、数据库或权限边界。

可以这样触发：

```text
使用 harness-normalize 刷新当前 harness 知识库
使用 harness-normalize 检查是否需要从 Lite 升级到 Standard
这个 Lite 项目已经开始写业务了，升级 harness 知识库
```

## Skill 文件结构

```text
harness-normalize/
├── SKILL.md
├── README.md
├── WORKFLOW.md
└── references/
    ├── harness-principles.md
    ├── discovery-checklist.md
    ├── structure-profiles.md
    ├── refresh-strategy.md
    ├── layer-mapping-guide.md
    ├── agents-md-template.md
    ├── architecture-template.md
    ├── engineering-template.md
    ├── coding-standards-template.md
    ├── coding-standards-profiles.md
    ├── commit-standards-template.md
    ├── security-template.md
    ├── quality-score-template.md
    ├── design-docs-template.md
    ├── exec-plans-template.md
    └── glossary.md
```

## 日常维护

初始化后的日常使用方式见 [`WORKFLOW.md`](./WORKFLOW.md)。
