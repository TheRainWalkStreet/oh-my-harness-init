# AGENTS.md 模板

约 100 行。是索引，不是百科全书。800 行的版本被废弃了，因为 Agent 根本找不到它需要的东西。

~~~markdown
# {项目名称} — Agent 导航地图

> {一行描述}

## 技术栈

| 维度         | 详情           |
|--------------|--------------|
| 语言         | {语言 + 版本}  |
| 框架         | {框架 + 版本}  |
| 数据库       | {类型}         |
| 包管理器     | {npm / pip / go mod / maven 等} |
| 测试框架     | {jest / pytest / go test / JUnit 等} |
| CI 平台      | {GitHub Actions / GitLab CI / Jenkins 等} |

## 架构层级

依赖关系**只能向下**流动（A → B 表示 A 可以导入 B）。禁止向上导入。

{从实际 import 模式发现的层级图，参见 references/layer-templates.md}

## 关键规范

- {规范 1 —— 简短，详情指向 docs/golden-principles/}
- {规范 2}
- {规范 3}

## 常用命令

```sh
{构建命令}       # 例：npm run build
{测试命令}       # 例：npm test
{Lint 命令}      # 例：npm run lint
{开发命令}       # 例：npm run dev
{GC 命令}        # 例：npm run gc
```

## 文档导航

```
ARCHITECTURE.md                       顶级领域地图（根目录）
docs/
├── architecture/LAYERS.md            层级规则、依赖图（权威来源）
├── golden-principles/                规范模式（DO/DON'T 示例）
├── SECURITY.md                       认证、密钥、威胁模型
├── guides/                           设置、测试、部署指南
├── exec-plans/                       功能实现执行计划
├── design-docs/                      架构决策记录（ADR）
└── references/                       外部库文档（LLM 友好格式）
```

> **注意：** 只列出实际存在的目录，不存在的可删除。

## 优先查阅位置

| 任务               | 从这里开始                          |
|--------------------|-------------------------------------|
| 架构概览           | ARCHITECTURE.md（根目录）           |
| 层级规则           | docs/architecture/LAYERS.md         |
| {常见任务 1}       | {目录/文件}                         |
| {常见任务 2}       | {目录/文件}                         |
| {常见任务 3}       | {目录/文件}                         |

## 约束（机器可读）

以下关键词为标准化语义标记，请保持英文以确保 Agent 可一致解析：

- MUST: {硬性规则 —— 附约束执行位置}
- MUST NOT: {禁令 —— 附 LAYERS.md 引用}
- PREFER: {软性偏好}
- VERIFY: {提交 PR 前的验证命令}

**示例（填写时替换为实际内容）：**

- MUST: 所有 import 使用 `@/` 路径别名，不使用 `../` 相对路径 [enforced: .eslintrc `no-restricted-imports`]
- MUST NOT: `services/` 层直接导入 `components/` 层 [enforced: tests/architecture/boundary.test.ts]
- PREFER: 使用 `Result<T, E>` 模式而非直接 throw [参见 docs/golden-principles/ERROR_HANDLING.md]
- PREFER: PR 保持短周期，偶发测试失败用跟进 PR 修复，不要无限等待 [高吞吐原则：纠错比阻塞更便宜]
- VERIFY: 提交前运行 `npm run lint && npm test` 确保无新违规
~~~
