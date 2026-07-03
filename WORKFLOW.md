# 日常工作流

本文说明项目完成 harness 规范化后，团队和 AI Agent 如何持续使用并维护这些文档。

## 会话入口

每次进入项目，先读：

1. `AGENTS.md`
2. `ARCHITECTURE.md`
3. 与任务相关的 `docs/` 文件

`AGENTS.md` 只负责导航。不要把所有规则都塞回 `AGENTS.md`。

## 常见任务路由

| 任务 | 先读 |
| --- | --- |
| 理解系统结构 | `ARCHITECTURE.md` |
| 本地运行、测试、构建 | `docs/ENGINEERING.md` |
| 写代码或 review 代码 | `docs/standards/coding.md` |
| 组织提交或 PR 描述 | `docs/standards/commits.md` |
| 涉及认证、配置、数据、外部 API | `docs/SECURITY.md` |
| 判断维护优先级 | `docs/QUALITY_SCORE.md` |
| 做复杂多步任务 | `docs/exec-plans/` |
| 查询设计决策 | `docs/design-docs/` |

## 新功能开发

1. 先读 `AGENTS.md`，确认任务应进入哪个模块。
2. 读 `ARCHITECTURE.md`，确认层级边界和依赖方向。
3. 读 `docs/standards/coding.md`，确认语言、框架和项目工具规范。
4. 如果任务跨多个模块或会持续多轮，先在 `docs/exec-plans/active/` 创建执行计划。
5. 完成后更新相关文档，不让代码事实和文档漂移。

## 刷新 harness 知识库

开发一段时间后，不要重新初始化。使用 Refresh：

```text
使用 harness-normalize 刷新当前 harness 知识库
```

Refresh 会重新扫描项目事实，并增量更新：

- `AGENTS.md`
- `ARCHITECTURE.md`
- `docs/ENGINEERING.md`
- `docs/standards/coding.md`
- `docs/standards/commits.md`
- `docs/SECURITY.md`
- Standard 项目的 `docs/QUALITY_SCORE.md`、`docs/design-docs/`、`docs/exec-plans/`

刷新不是覆盖。仍然准确的内容应保留，过期内容用仓库事实修正，不确定内容标记为 `待确认`。

## Lite 升级 Standard

Lite 项目开始出现业务代码后，定期检查是否需要升级：

```text
使用 harness-normalize 检查是否需要从 Lite 升级到 Standard
```

建议升级的信号：

- 出现清晰业务模块或领域概念。
- 有稳定测试、lint、build 流程。
- 出现多人协作。
- 出现技术债或复杂改造需求。
- 有架构决策需要记录。
- 出现生产部署、外部 API、数据库或权限边界。

升级会保留已有 Lite 文档，并补充：

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

如果暂时不升级，也可以只刷新 Lite 文件。

## Bug 修复

1. 从复现路径定位相关模块。
2. 读 `ARCHITECTURE.md` 中该模块的职责。
3. 检查修复是否违反 `docs/standards/coding.md`。
4. 如果发现文档过期，顺手更新。
5. 最终说明中写清验证方式；没有运行验证时如实说明。

## 复杂任务与执行计划

复杂任务使用：

```text
docs/exec-plans/active/YYYY-MM-DD-任务简述.md
```

计划文件应持续记录：

- 背景。
- 目标和非目标。
- 步骤。
- 当前进度。
- 决策记录。
- 风险和待确认事项。
- 验证计划。

任务完成后移动到：

```text
docs/exec-plans/completed/YYYY-MM-DD-任务简述.md
```

归档前补齐：

- 最终结果。
- 实际变更。
- 验证情况。
- 遗留问题。
- 后续建议。

长期技术债进入 `docs/exec-plans/tech-debt.md`，不要混进单次执行计划。

## 设计决策

当任务改变架构、数据模型、接口契约、安全边界或长期技术方向时，更新：

- `docs/design-docs/index.md`
- `docs/design-docs/core-beliefs.md`
- 必要时新增设计文档

核心理念必须能指导 review。不要写泛泛口号。

## 开发规范维护

`docs/standards/coding.md` 不是外部规范搬运，而是项目自己的规范。

更新规则：

- 项目新增语言、框架或工具时更新。
- lint/format/typecheck 工具变更时更新。
- 同类代码问题反复出现时补充人工 review 标准。
- Java 项目可参考阿里巴巴 Java 开发手册；其它语言按自身生态适配。

## 提交规范维护

`docs/standards/commits.md` 默认采用 Conventional Commits，但项目已有规范优先。

维护规则：

- 如果引入 commitlint、changeset、语义化发布，更新规范。
- 如果团队修改分支命名或 PR 模板，更新规范。
- Agent 生成提交说明必须基于实际 diff，不编造测试或发布信息。

## 安全文档维护

`docs/SECURITY.md` 只记录安全边界和流程，不记录敏感事实。

禁止写入：

- 真实密钥、token、账号。
- 内部 IP、内网域名、生产地址。
- 真实用户数据。
- 未公开漏洞细节。

## 质量评分维护

`docs/QUALITY_SCORE.md` 服务于维护优先级，不是仪式文档。

建议在以下时机更新：

- 重大重构后。
- 测试覆盖或质量门禁明显变化后。
- 安全、可靠性或文档状态明显变化后。
- 进入下一个重要迭代前。

Lite 项目默认没有该文件；当项目事实足够时，可以升级到 Standard。

## 前后端一体项目

前后端一体项目默认不拆 `FRONTEND.md` / `BACKEND.md`。

日常维护时优先更新：

- `ARCHITECTURE.md`：前后端边界、API 契约、静态资源托管、构建产物流向。
- `docs/ENGINEERING.md`：前端命令、后端命令、统一入口、本地联调。
- `docs/standards/coding.md`：前端规范、后端规范、跨边界调用规则。
- `docs/SECURITY.md`：浏览器侧安全、服务端安全、CORS/CSRF、认证态、环境变量暴露。

只有复杂度达到独立治理程度时，才补充 `docs/FRONTEND.md` 或 `docs/BACKEND.md`。

## 处理 `待确认`

`待确认` 是有价值的信号，不是失败。

处理方式：

- 能从代码或配置确认的，确认后更新文档。
- 需要产品、运维或团队知识的，保留并在任务说明中提醒。
- 不要为了让文档看起来完整而编造答案。
