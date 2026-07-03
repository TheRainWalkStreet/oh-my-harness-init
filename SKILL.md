---
name: harness-normalize
description: 规范化、刷新或升级已有项目的 harness 知识库。Use when the user asks to make an existing repository agent-ready, harness-ready, initialize project knowledge, refresh an existing harness knowledge base, upgrade Lite to Standard, generate or update AGENTS.md, ARCHITECTURE.md, docs/ENGINEERING.md, docs/SECURITY.md, docs/QUALITY_SCORE.md, docs/standards/coding.md, docs/standards/commits.md, docs/design-docs, or docs/exec-plans for a project that already has a runnable framework, source tree, build config, or package setup. Do not use for empty project creation, feature implementation, bug fixing, CI generation, lint rule generation, boundary tests, garbage collection scripts, or pre-commit setup.
---

# Harness Normalize

## 目标

将已有项目规范化为适合 AI Agent 长期协作的 harness 知识库。输出中文文档，默认提交到目标项目 Git 中长期维护。

本 skill 只处理已有项目：项目必须已经有源码目录、构建配置、包管理配置、框架骨架或可运行入口之一。若目标目录接近空仓库、还没有技术栈或框架骨架，停止并说明“不支持空项目初始化”。

## 基本原则

- 先发现事实，再写文档；不要凭模板假设技术栈、命令、目录或业务边界。
- `AGENTS.md` 是入口地图，不是百科全书；详细内容放入 `ARCHITECTURE.md` 和 `docs/`。
- 文档少而准；没有事实触发就不创建 Extended 文档。
- 生成内容使用中文；不确定的信息标记为 `待确认`。
- 不生成 `.gitkeep`；每个生成文件都必须有长期维护价值。
- 不修改 lint、CI、边界测试、GC 脚本、pre-commit 等自动化质量门禁；如果项目已有这些能力，只记录使用方式。
- 不复制外部规范全文；开发规范必须按当前项目语言、框架和工具链裁剪。

## 执行流程

### 1. 仓库发现

读取 `references/discovery-checklist.md`，扫描目标项目事实：

- Git 状态、未提交变更、源码目录、构建配置、包管理文件。
- 技术栈、框架、运行入口、测试/lint/format/build/dev 命令。
- 现有文档：`README.md`、`AGENTS.md`、`CLAUDE.md`、`docs/`、架构文档。
- 主要 import/use 关系、模块边界、前后端一体痕迹、monorepo 痕迹。
- 敏感配置暴露迹象：密钥、token、账号、密码、内部地址、真实业务样本。只记录风险类别，不复制具体值。

如果技术栈、目录结构或项目类型无法判断，先询问用户。不要强行套用结构。

### 2. 识别 harness 状态与运行模式

根据 `references/discovery-checklist.md` 判断当前 harness 状态：

- Uninitialized：缺少核心 harness 文件。
- Lite：已有 Lite 核心文件，但没有 Standard 文件。
- Standard：已有 Standard 核心文件。
- Mixed：文件不完整、结构互相冲突或新旧结构混杂。

选择运行模式：

- Initialize：Uninitialized 项目按首次规范化流程生成。
- Refresh：Lite/Standard 项目已有 harness 知识库时，增量刷新已有文档。
- Upgrade：Lite 项目已经出现 Standard 信号时，建议从 Lite 升级到 Standard。

如果是 Mixed 状态，先报告发现的问题和建议修复路径，询问用户整理为 Lite 还是 Standard。不要静默覆盖。

读取 `references/refresh-strategy.md` 处理 Refresh 和 Upgrade。刷新不是覆盖，升级不是重建。

### 3. 选择结构档位

读取 `references/structure-profiles.md`，选择 Lite、Standard 或 Extended。

- Lite：框架已搭好但业务代码很少，或小型库/工具/脚本服务。
- Standard：多数真实业务项目、多人维护项目、长期迭代项目。
- Extended：monorepo、多应用、强合规、强运行时要求、已有生产用户的项目。

前后端一体项目默认使用 Standard，在标准文档中生成专项章节；只有复杂到需要独立治理时，才条件生成 `docs/FRONTEND.md` 或 `docs/BACKEND.md`。

写入前简要告知用户：检测到的技术栈、判断为已有项目的证据、推荐结构档位、将创建或更新的文件清单。若用户明确要求全自动，可直接执行并在最终回复说明判断依据。

### 4. 映射架构层级

读取 `references/layer-mapping-guide.md`。基于真实目录、入口、import/use 关系映射项目自己的层级名称。

不要强行使用通用层级名。若项目叫 `handlers`、`usecases`、`adapters`、`modules`、`features`，文档中优先使用这些真实名称，再解释它们的职责。

### 5. 读取模板并生成或更新文档

按需要读取以下 reference。不要一次性加载无关文件。

| 输出 | 何时读取 |
| --- | --- |
| 设计原则不清晰或需要解释 harness 范式 | `references/harness-principles.md` |
| `AGENTS.md` | `references/agents-md-template.md` |
| `ARCHITECTURE.md` | `references/architecture-template.md` |
| `docs/ENGINEERING.md` | `references/engineering-template.md` |
| `docs/standards/coding.md` | `references/coding-standards-template.md` 和 `references/coding-standards-profiles.md` |
| `docs/standards/commits.md` | `references/commit-standards-template.md` |
| `docs/SECURITY.md` | `references/security-template.md` |
| `docs/QUALITY_SCORE.md` | `references/quality-score-template.md`，仅 Standard/Extended 默认生成 |
| `docs/design-docs/index.md`、`docs/design-docs/core-beliefs.md` | `references/design-docs-template.md`，仅 Standard/Extended 默认生成 |
| `docs/exec-plans/*` | `references/exec-plans-template.md`，仅 Standard/Extended 默认生成 |
| 刷新已有 harness 或 Lite 升级 Standard | `references/refresh-strategy.md` |
| 术语统一 | `references/glossary.md`，仅在命名或概念模糊时读取 |

推荐写入顺序：

1. `ARCHITECTURE.md`
2. `docs/ENGINEERING.md`
3. `docs/standards/coding.md`
4. `docs/standards/commits.md`
5. `docs/SECURITY.md`
6. `docs/QUALITY_SCORE.md`（Standard/Extended）
7. `docs/design-docs/index.md`（Standard/Extended）
8. `docs/design-docs/core-beliefs.md`（Standard/Extended）
9. `docs/exec-plans/index.md`（Standard/Extended）
10. `docs/exec-plans/active/README.md`（Standard/Extended）
11. `docs/exec-plans/completed/README.md`（Standard/Extended）
12. `docs/exec-plans/tech-debt.md`（Standard/Extended）
13. `AGENTS.md`

Initialize 模式先生成事实文档，再生成 `AGENTS.md`，让入口文件只承担导航职责。

Refresh 模式按文件增量更新，保留仍然准确的内容；对过期信息用仓库事实修正，对无法确认的信息标记 `待确认`。

Upgrade 模式只补充 Standard 缺失文件并刷新 Lite 已有文件，不重写已有 Lite 知识。

## 默认输出结构

### Lite

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

在 Standard 基础上，仅按项目事实条件生成：

- `docs/generated/api-spec.md`
- `docs/generated/db-schema.md`
- `docs/references/{library-or-platform}-llms.txt`
- `docs/product-specs/index.md`
- `docs/RELIABILITY.md`
- `docs/FRONTEND.md`
- `docs/BACKEND.md`
- `docs/MOBILE.md`
- `docs/OPERATIONS.md`

## 更新已有文档

- 如果目标项目已有同名文件，先阅读并保留仍然准确的内容。
- 若现有文档和新结构冲突明显，先说明冲突并询问用户。
- 不要删除用户已有知识，除非用户明确要求或内容已明显错误且有项目事实支撑。
- 文档中列出的命令、目录、环境和工具必须来自真实配置。
- 已有 harness 文件时，默认进入 Refresh/Upgrade 判断，而不是重新初始化。
- Lite 项目出现 Standard 信号时，先建议升级并说明将新增哪些文件；用户拒绝升级时只刷新 Lite 文件。

## 完成前自检

- `AGENTS.md` 中引用的文件真实存在。
- 文档中列出的命令来自项目配置或现有脚本。
- 文档中列出的目录真实存在。
- 技术栈判断有证据支撑。
- 架构层级来自真实代码结构。
- 没有写入真实密钥、内部地址、账号、漏洞细节或其它敏感信息。
- 已对生成文档做敏感字面值自检；如目标项目存在明文敏感配置，只在安全文档中描述类别和治理要求。
- 没有生成 `.gitkeep` 或空壳文档。
- 没有修改 lint、CI、边界测试、GC、pre-commit 等自动化配置。

## 停止条件

- 目标目录不是已有项目。
- 技术栈或项目类型无法可靠判断，且用户没有提供补充信息。
- 现有文档结构与目标 harness 结构冲突，继续写入会覆盖重要知识。
- 用户要求生成自动化质量门禁、初始化新项目或实现业务功能；这些任务不属于 v1.3.0 范围。
