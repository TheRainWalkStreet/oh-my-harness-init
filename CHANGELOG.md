# 变更日志

本文件记录 oh-my-harness-init 各版本的变更内容。

格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

---

## [1.2.0] — 2026-04-22

### 新增（来自 OpenAI 原文及社区实践）

- `SKILL.md <Principles>`：新增**原则 7 —— 高吞吐下纠错比等待更便宜**（来自 OpenAI 实践：减少阻塞门控，PR 短周期，跟进 PR 修复优于无限等待）
- `SKILL.md <Steps> 阶段 0`：完整重写，新增 **Greenfield/Brownfield 明确分叉路径**，两种仓库类型采用截然不同的策略
- `references/golden-principles-guide.md`：新增**"将学习持续升级为约束"升级阶梯章节**（Docs → Tests → Lint → 自动化，附决策矩阵和实践规则）
- `references/glossary.md`：新增 Greenfield、Brownfield 两个术语定义
- `references/agents-md-template.md`：约束部分新增 PREFER 示例（PR 短周期高吞吐原则）
- `SKILL.md <Execution_Policy>`：补充升级阶梯的引用指针
- `SKILL.md <Examples>`：两个 Good 示例更新，明确标注 Greenfield/Brownfield 路径标识

### 版本更新
- `SKILL.md` frontmatter：`version: "1.2.0"`

---

## [1.1.0] — 2026-04-22

### 新增
- `references/glossary.md`：24 个核心术语的中英文对照定义，统一各文件间的概念表述
- `references/layer-templates.md`：新增 Java/Spring Boot 层级模型（含 controller 禁止直连 repository 的警告）
- `references/boundary-test-template.md`：新增 Go 边界测试骨架（含 `go/parser` AST 解析）
- `references/golden-principles-guide.md`：新增"执行方式"模板章节、后端/Java 候选文件表、候选主题筛选策略
- `references/tool-routing.md`：新增 **Review（审查）** 意图；新增 Cursor 专项使用说明表
- `SKILL.md <Advanced>`：新增各阶段产出快速参考表
- `SKILL.md <Examples>`：新增 2 个 Bad 场景（安全文档误用、条件性文档全量创建）
- `README.md`：项目说明文件，包含各平台安装指引
- `CHANGELOG.md`：本文件

### 修复
- `stack-routing.md`：修正 Ruff Import 限制规则名（`banned-api` → `TID251`），附正确配置示例
- `stack-routing.md`：术语统一为"阶段 N"，消除与 SKILL.md 的跨文件漂移；Java/Kotlin 拆为独立两行
- `context-strategy.md`：`.omc/state/` 标注为平台专属；CI 状态信号按 GitHub/GitLab/Gitee/Jenkins 分平台列出
- `SKILL.md`：修正标题（恢复版本信息）、"第 6 阶段"遗留术语、Phase 0 子项格式（a-f → 列表符）
- `SKILL.md`：修正阶段并行图——阶段 5/6/7 三者互不依赖，可完全并行

### 优化
- `agents-md-template.md`：技术栈表格新增包管理器、测试框架、CI 平台；约束部分补充具体 MUST/MUST NOT 示例
- `gc-patterns.md`：知识新鲜度阈值明确为 30 天并附查询命令；"模式偏离"标注为人工审查项；补充 GC 结果处置表格
- `ci-templates.md`：SHA 哈希加"使用前核查"警告；Makefile 加 `@` 前缀说明；新增 Gitee 差异说明
- `exec-plan-template.md`：新增文件命名规范、预估工时字段、"完全自包含"边界说明
- `tool-routing.md`：模型名由硬编码改为通用描述占位，避免因模型迭代导致文档过时
- `SKILL.md <Do_Not_Use_When>`：说明部分已有组件时可指定阶段补充，而非要求全套才能"不适用"
- `SKILL.md <Final_Checklist>`：新增阶段 7 条目；黄金原则条目改为可命令验证的形式

---

## [1.0.0] — 2026-04-22

### 新增
- 基于 [harness-init@1.1.0](https://github.com/Gizele1/harness-init) 的完整中文版本
- `SKILL.md`：全文中文化，XML 标签保留英文（保证 LLM 解析鲁棒性），内容全中文
- 新增阶段依赖关系并行执行图
- 新增 Spring Boot 场景示例
- `references/` 下所有 11 个参考文件的中文版本
- Frontmatter 新增 `language: zh-CN`、`based-on` 版本溯源字段
- 参考文件列表按阶段 0-7 顺序排列，每条附阶段标注
- 各阶段步骤间视觉分隔（`---` 分隔线）
- `agents-md-template.md`：恢复 `MUST/MUST NOT/PREFER/VERIFY` 机器可读关键词
- `tool-routing.md`：标注 oh-my-claudecode 插件专属语法
