# AGENTS.md 模板

`AGENTS.md` 是 Agent 入口地图，目标 80-140 行。不要写成长篇说明书。

```markdown
# AGENTS.md

## 项目概览

- 项目类型：{项目类型}
- 技术栈：{主要语言/框架/运行时}
- 结构档位：{Lite|Standard|Extended}

## 先读什么

1. `ARCHITECTURE.md`：系统结构、层级边界、关键数据流。
2. `docs/ENGINEERING.md`：本地开发、测试、构建、协作流程。
3. `docs/standards/coding.md`：代码规范。
4. `docs/standards/commits.md`：提交规范。
5. `docs/SECURITY.md`：安全边界。
{Standard 或 Extended 时列出 QUALITY_SCORE/design-docs/exec-plans}

## 常用命令

- 安装依赖：`{真实命令或待确认}`
- 本地运行：`{真实命令或待确认}`
- 测试：`{真实命令或待确认}`
- lint：`{真实命令或待确认}`
- format：`{真实命令或待确认}`
- build：`{真实命令或待确认}`

## 关键目录

| 路径 | 职责 |
| --- | --- |
| `{路径}` | `{职责}` |

## 架构边界

- `{层级/模块}` 负责 `{职责}`。
- `{层级/模块}` 可以依赖 `{允许依赖}`。
- `{层级/模块}` 不应依赖 `{禁止依赖}`。
- 不清晰边界：`待确认`。

## 任务路由

- 改 UI/页面：先读 `{路径或文档}`。
- 改 API/后端：先读 `{路径或文档}`。
- 改数据访问：先读 `{路径或文档}`。
- 改配置/部署：先读 `{路径或文档}`。
- 复杂多步任务：在 `docs/exec-plans/active/` 建计划。

## Agent 约束

- 先读相关文档，再改代码。
- 不编造不存在的命令、目录、业务规则或外部系统。
- 不写入真实密钥、账号、内部地址、漏洞细节。
- 不把格式化、重构、功能、文档大面积混在一个变更里。
- 不修改自动化门禁，除非用户明确要求。

## 文档索引

- `ARCHITECTURE.md`
- `docs/ENGINEERING.md`
- `docs/standards/coding.md`
- `docs/standards/commits.md`
- `docs/SECURITY.md`
```

## 填写规则

- 命令必须来自真实配置。
- 目录必须真实存在。
- 文档链接必须真实存在。
- 没有事实的信息写 `待确认`。
