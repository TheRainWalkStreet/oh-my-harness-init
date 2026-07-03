# Refresh 与 Upgrade 策略

本文件用于已有 harness 知识库的后续维护。

## 运行模式

### Initialize

目标项目没有核心 harness 文件时使用。按结构档位首次生成知识库。

### Refresh

目标项目已有 Lite 或 Standard harness 文件时使用。Refresh 是增量更新，不是全量覆盖。

### Upgrade

目标项目当前是 Lite，且已经出现 Standard 信号时使用。Upgrade 只补充 Standard 缺失文件，并刷新已有 Lite 文档。

## Refresh 原则

- 重新扫描项目事实。
- 保留仍然准确的内容。
- 用事实修正过期内容。
- 无法确认的信息标记为 `待确认`。
- 不删除用户已有知识，除非用户明确要求。
- 不移动 `docs/exec-plans/active/` 和 `completed/` 中的计划文件，除非用户明确要求。
- 不把聊天记录当作事实来源。

## 文件更新策略

| 文件 | Refresh 行为 |
| --- | --- |
| `AGENTS.md` | 更新项目概览、命令、目录、任务路由、文档索引 |
| `ARCHITECTURE.md` | 更新模块地图、层级边界、关键数据流、待确认事项 |
| `docs/ENGINEERING.md` | 更新命令、工具、运行方式、本地联调方式，并检查测试/lint/format/build 覆盖范围 |
| `docs/standards/coding.md` | 更新语言、框架、工具链适配规则和命令覆盖规范 |
| `docs/standards/commits.md` | 更新提交工具、历史惯例、PR 规则 |
| `docs/SECURITY.md` | 更新认证、配置、外部 API、数据边界 |
| `docs/QUALITY_SCORE.md` | Standard/Extended 项目刷新质量判断 |
| `docs/design-docs/index.md` | 更新索引和待确认事项，不编造 ADR |
| `docs/design-docs/core-beliefs.md` | 只在真实工程理念变化时更新 |
| `docs/exec-plans/index.md` | 更新执行计划规则 |
| `docs/exec-plans/active/README.md` | 更新 active 目录规则 |
| `docs/exec-plans/completed/README.md` | 更新 completed 目录规则 |
| `docs/exec-plans/tech-debt.md` | 更新技术债列表，不混入单次执行计划 |

## Lite -> Standard Upgrade

升级条件来自 `references/structure-profiles.md`。

升级时新增：

```text
docs/
├── QUALITY_SCORE.md
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

同时刷新：

- `AGENTS.md`
- `ARCHITECTURE.md`
- `docs/ENGINEERING.md`
- `docs/standards/coding.md`
- `docs/standards/commits.md`
- `docs/SECURITY.md`

如果用户拒绝升级，只刷新 Lite 文件。

## 命令覆盖检查

Refresh 或 Upgrade 时，如果发现新增源码目录、业务模块或测试目录，必须重新检查：

- `docs/ENGINEERING.md` 中测试、lint、format、build 命令的覆盖范围是否仍准确。
- `AGENTS.md` 中常用命令是否仍准确。
- `docs/standards/coding.md` 是否需要补充“新增模块后更新命令覆盖”的规范。

如果命令覆盖不足，只记录风险并提醒用户。不要自动修改 `package.json`、Makefile、CI 或其它质量门禁配置，除非用户明确要求。

## Mixed 状态

Mixed 示例：

- 有 `docs/exec-plans/active/` 但没有 `docs/exec-plans/index.md`。
- 有 `docs/QUALITY_SCORE.md` 但没有 `ARCHITECTURE.md`。
- 有旧结构文档和当前结构冲突。

处理：

1. 报告当前文件状态。
2. 说明推荐整理为 Lite 还是 Standard。
3. 询问用户是否继续。

不要静默覆盖或删除。

## 变更前说明

写入前简要说明：

- 当前 harness 状态。
- 推荐运行模式：Initialize / Refresh / Upgrade。
- 将创建、更新或保留的文件。
- 需要用户确认的冲突或待确认事项。
