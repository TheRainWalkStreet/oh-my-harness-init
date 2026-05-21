# 日常开发工作流

> 本文档回答一个关键问题：**仓库初始化完成后，如何在日常开发中让 AI 工具持续遵循已建立的 harness 规范？**

---

## 核心原理：AGENTS.md 是会话的起点

初始化完成后，仓库里已经有：

```
AGENTS.md                   ← AI 每次会话自动读取的导航地图
docs/architecture/LAYERS.md ← 权威层级规则
docs/golden-principles/     ← 编码规范（DO/DON'T）
tests/architecture/         ← 边界测试（CI 自动运行）
.eslintrc / pyproject.toml  ← Linter 本地实时执行
```

**这些文件构成了"常驻上下文"**。你不需要在每次对话中重复解释项目架构——AI 工具会自动读取 AGENTS.md，从中获得项目地图、层级规则和关键约束。

---

## 各平台如何加载上下文

### Cursor

Cursor 在 Agent 模式下会自动读取仓库根目录的 `AGENTS.md`（如果存在）。此外，`.cursor/rules/oh-my-harness-init/SKILL.md` 以 **Agent Requested** 模式存在——当 Cursor 判断任务与 harness 初始化相关时，会主动加载它。

**推荐的日常使用方式：**

```
# 直接描述任务，AI 会基于 AGENTS.md 中的上下文执行
"给 UserService 增加一个获取用户列表的方法"

# 需要参考具体文档时，显式引用
"检查这个 PR 改动是否符合 @docs/architecture/LAYERS.md 的层级规则"

# 遇到架构相关问题时
"我想在 components 层中调用 db 层，可以吗？参考 @docs/architecture/LAYERS.md"
```

### Claude Code

Claude Code 在每次会话启动时**自动将 `AGENTS.md` 注入上下文**。技能文件（`.claude/skills/oh-my-harness-init/SKILL.md`）由触发词激活。

**推荐的日常使用方式：**

```bash
# 普通开发任务 —— AI 自动遵循 AGENTS.md 中的规范
claude "在 services 层新增一个 PaymentService"

# 需要进行 harness 相关操作时（触发 Skill）
claude "给现有仓库补充 CI 配置（harness-init 阶段 5）"

# 明确要求审查层级合规性
claude "审查最近的改动是否违反了架构层级规则"
```

### OpenAI Codex

Codex 同样自动读取 `AGENTS.md`，技能在 `.agents/skills/` 目录下按需加载。

```bash
# 普通任务
codex "重构 UserRepository，保持在 repository 层"

# 触发 harness skill
codex "harness-init 3-4"  # 仅执行边界测试和 Lint 配置

# 验证架构合规
codex "运行架构边界测试并报告结果"
```

---

## 日常开发循环

每次功能开发建议遵循以下循环：

```
1. 定义任务意图（人类）
      ↓
2. AI 自动读取 AGENTS.md，获取项目地图和约束
      ↓
3. AI 参考 docs/architecture/LAYERS.md 设计实现方案
      ↓
4. AI 编写代码，遵循 docs/golden-principles/ 中的规范
      ↓
5. 本地验证（AI 或人类触发）：
   - Lint：import 边界检查（实时报错）
   - 边界测试：npm test / pytest / go test（提交前）
      ↓
6. 开 PR → CI 全量检查（lint + typecheck + test + build）
      ↓
7. 合并 → 继续下一个任务
```

**每周一次：** GC 扫描自动运行，发现文档漂移和架构违规，以 Issue 形式报告。

---

## 有效的提示词模式

### 好的提示词

```
✅ "给 OrderService 增加取消订单的方法，注意它只能调用 repository 层"
   → 简洁，隐含层级约束提醒

✅ "这段代码的错误处理是否符合我们的规范？"
   → AI 会自动查阅 docs/golden-principles/ERROR_HANDLING.md

✅ "审查这个文件，找出所有违反 LAYERS.md 的 import"
   → 明确引导 AI 使用已有文档

✅ "用我们项目的测试模式为这个函数写单元测试"
   → AI 会参考 docs/golden-principles/TESTING.md
```

### 需要避免的提示词

```
❌ "写一个 UserService，从数据库读取用户，然后调用前端 API 渲染"
   → 混淆了层级职责，容易引发架构违规
   → 更好的方式：拆分成"在 services 层写 UserService"和"在 pages 层调用它"

❌ "帮我快速实现这个功能，不用管代码质量"
   → 明确绕过了黄金原则，日后会产生技术债
   → 更好的方式：任何实现都应遵循现有规范，速度不是跳过规范的理由

❌ 在每次提示词里手动粘贴架构规则
   → 这正是 AGENTS.md 存在的意义，不要重复它
   → AI 已经通过 AGENTS.md 知道这些规则了
```

---

## 当 AI 违反 harness 规范时

即使有了 AGENTS.md 和 Lint 规则，AI 偶尔仍会产生不符合规范的代码。按以下方式处理：

### 情况 1：边界测试或 Lint 报错

```
VIOLATION: src/components/UserCard.tsx:5 imports src/services/userService
— components 不能导入 services。参见 docs/architecture/LAYERS.md
```

**处理方式：**
1. 将报错信息直接粘贴给 AI，让它自己修复
2. AI 的报错信息已包含修复指引，通常能自行解决
3. 如果反复出现，将这个违规加入 KNOWN_VIOLATIONS 作为临时豁免，记录移除时间线

### 情况 2：黄金原则被忽视（代码可以运行但不符合规范）

```bash
# 明确告知 AI 哪条规范被违反
"这里的错误处理方式不符合我们的 ERROR_HANDLING.md，
请重写，使用 Result 类型而不是直接 throw"
```

### 情况 3：同一类问题反复出现

**触发升级阶梯**（参见 `references/golden-principles-guide.md`）：

```
第 1 次 → 在 docs/golden-principles/ 中补充或澄清规则
第 2 次 → 添加对应的边界测试 / 单元测试
第 3 次 → 添加 Lint 规则，让工具机械阻断
持续发生 → 编入 GC 脚本，定期自动扫描修复
```

> **核心思路：** AI 的错误是环境信号，不是 AI 的问题。每次错误都是一个改进 harness 的机会。

---

## 复杂任务：使用执行计划

对于多小时的复杂功能（如重大重构、新功能模块），使用执行计划（ExecPlan）来保持上下文连续：

```bash
# 让 AI 先写执行计划，再开始实现
"在 docs/exec-plans/active/ 下为'用户认证重构'创建一个执行计划，
命名为 2026-04-22-auth-refactor.md，
然后按计划逐步实现"
```

执行计划会记录：进度、意外发现、决策日志——即使会话中断，下次也能从计划文件恢复上下文。

---

## 每周维护清单

harness 不是"设置一次就永久有效"的，需要定期维护：

```markdown
每周一次（通常由 GC Action 自动触发，也可手动运行）：

- [ ] 运行 GC 扫描：`npm run gc` / `make gc`
      → 检查文档漂移、架构违规、知识新鲜度
- [ ] 查看 GC 报告生成的 Issue（标签：garbage-collection）
- [ ] 处理报告中的问题（按升级阶梯决定处理方式）

每月一次：
- [ ] 检查 QUALITY_SCORE.md 是否需要更新
- [ ] 回顾 KNOWN_VIOLATIONS —— 是否有可以修复并移除的条目
- [ ] 检查 docs/exec-plans/active/ —— 是否有已完成但未归档的计划
- [ ] 回顾 golden-principles/ —— 是否有需要升级为 Lint 规则的文档规则
```

---

## 快速参考

| 我想做什么 | 怎么告诉 AI |
|------------|-------------|
| 新增功能 | 直接描述功能，AI 自动遵循 AGENTS.md 中的规范 |
| 审查架构合规性 | "检查最近改动是否违反 `@docs/architecture/LAYERS.md`" |
| 修复层级违规 | 直接粘贴 VIOLATION 报错信息，让 AI 自行修复 |
| 补充规范文档 | "在 golden-principles/ 下为 [主题] 写一个规范文档" |
| 复杂多步任务 | "先在 exec-plans/active/ 创建执行计划，再开始实现" |
| 运行全量检查 | "运行 lint、边界测试和 GC 扫描，报告结果" |
| 单独补充某个阶段 | "执行 oh-my-harness-init [N]" 或 "执行 harness-init 阶段 [N]" |
