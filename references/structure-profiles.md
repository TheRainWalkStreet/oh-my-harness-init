# 结构档位

根据发现结果选择 Lite、Standard 或 Extended。不要固定生成一套目录。

## 当前 harness 状态

- Uninitialized：缺少核心 harness 文件，使用 Initialize 模式。
- Lite：已有 Lite 核心文件，使用 Refresh；如出现升级信号，建议 Upgrade 到 Standard。
- Standard：已有 Standard 核心文件，使用 Refresh。
- Mixed：文件不完整、结构不一致或新旧结构混杂；先报告并询问用户整理方向。

## Lite

适用：

- 框架刚搭好，能运行，但业务代码很少。
- 小型库、工具、脚本服务、单人维护项目。
- 缺少足够事实做质量评分、设计决策和执行计划体系。

输出：

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

不要生成 `docs/QUALITY_SCORE.md`、`docs/design-docs/`、`docs/exec-plans/`。

## Lite -> Standard

当 Lite 项目出现多个信号时，建议升级 Standard：

- 清晰业务模块或领域概念。
- 稳定测试、lint、build 流程。
- 多人协作迹象。
- 技术债或复杂改造需求。
- 架构决策需要记录。
- 多模块依赖边界。
- 生产部署、外部 API、数据库或权限边界。

升级只补充 Standard 缺失文件，不重写已有 Lite 文档。

## Standard

适用：

- 多数真实业务项目。
- 正在开发或已开发完成。
- 有业务模块、测试/构建脚本、多人维护迹象或长期迭代迹象。

输出：

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

## Extended

在 Standard 基础上条件生成：

- `docs/generated/api-spec.md`：存在 OpenAPI、GraphQL、protobuf、IDL 等接口事实。
- `docs/generated/db-schema.md`：存在 schema、migration、ORM metadata 等数据库事实。
- `docs/references/{library-or-platform}-llms.txt`：项目强依赖外部平台或关键库，且 Agent 经常需要该知识。
- `docs/product-specs/index.md`：存在明确产品模块、用户流程或需求文档。
- `docs/RELIABILITY.md`：线上服务有 SLA、告警、降级、恢复要求。
- `docs/FRONTEND.md`：复杂前端交互、设计系统、SSR/CSR、浏览器兼容需要独立治理。
- `docs/BACKEND.md`：复杂 API、权限、任务队列、数据一致性、服务边界需要独立治理。
- `docs/MOBILE.md`：移动端项目。
- `docs/OPERATIONS.md`：已有部署、回滚、环境、值班或运维流程。

Standard 项目出现 Extended 信号时，只提示可扩展，不自动生成没有事实支撑的空壳文档。

## 前后端一体项目

识别：

- 同仓存在前端构建文件和后端框架入口。
- 后端托管模板、静态资源或前端构建产物。
- 同一部署流程处理前端和后端。
- 目录如 `frontend/` + `backend/`、`client/` + `api/`、`web/` + `server/`，但没有 workspace/package 边界。

默认使用 Standard，不拆 `FRONTEND.md` / `BACKEND.md`。在以下文档内增加专项章节：

- `ARCHITECTURE.md`：前后端边界、API 契约、静态资源托管、构建产物流向、部署关系。
- `docs/ENGINEERING.md`：前端命令、后端命令、统一入口、本地联调。
- `docs/standards/coding.md`：前端规范、后端规范、跨边界调用规则。
- `docs/SECURITY.md`：浏览器侧安全、服务端安全、CORS/CSRF、认证态、环境变量暴露。
