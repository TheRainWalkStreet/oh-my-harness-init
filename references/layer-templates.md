# 架构层级模板

常见技术栈的参考模板。请适配**实际**目录结构 —— 通过 import 模式发现，不要强行套用。

**符号说明：**
- `A → B` 表示 **A 层可以导入 B 层**（依赖方向向下）
- `→ 无 app import` 表示该层不能导入任何本项目的其他层
- 依赖关系**只能向下单向流动**，禁止向上导入

---

## Web 前端（React / Vue / Svelte）

```
types/        → 无 app import（纯类型定义）
utils/        → 无 app import（纯函数）
lib/          → types/（客户端、配置）
services/     → lib/、types/（业务逻辑）
hooks/states/ → lib/、services/、types/（状态管理）
components/   → hooks/、lib/、types/（UI 组件）
pages/routes/ → components/、hooks/、lib/、types/（入口点）
```

---

## 后端 API（Express / FastAPI / Rails）

```
types/models/ → 无 app import（数据定义）
config/       → types/
db/repo/      → config/、types/（数据访问层）
services/     → db/、config/、types/（业务逻辑）
middleware/   → services/、config/、types/（请求处理）
routes/       → services/、middleware/、types/（HTTP 处理器）
```

---

## Java / Spring Boot

适用于标准 Spring Boot 分层架构（controller → service → repository → domain）。

```
domain/entity/ → 无 app import（领域实体、值对象）
config/        → domain/（Spring 配置，仅依赖实体）
repository/    → domain/（JPA/MyBatis 数据访问）
service/       → repository/、domain/（业务逻辑）
controller/    → service/、domain/（HTTP 层，不直接访问 repository）
```

> **注意：** `controller` 层**禁止**直接导入 `repository`，必须通过 `service` 层。这是 Spring Boot 分层架构最常见的违规点，应在 ArchUnit 边界测试中明确约束。

---

## 全栈（Next.js / Nuxt / SvelteKit）

```
types/        → 无 app import
lib/          → types/（共享工具）
db/           → lib/、types/（数据库）
services/     → db/、lib/、types/（业务逻辑）
components/   → lib/、types/（UI 基础组件）
features/     → components/、services/、lib/、types/（功能模块）
app/pages/    → features/、components/、lib/、types/（路由）
```

---

## Monorepo（Turborepo / Nx）

```
packages/types/   → 无内部 import
packages/config/  → types/
packages/db/      → config/、types/
packages/api/     → db/、config/、types/
packages/ui/      → types/
packages/web/     → ui/、api/、types/
```

---

## OpenAI 原版模型

来自 harness 工程文章的标准模型：

```
Types → Config → Repo → Service → Runtime → UI
```

每层只能从其左侧的层导入。

**Provider** 处理横切关注点（认证、连接器、遥测、特性开关）。Provider 是注入横切依赖的**唯一**机制 —— 禁止跨域直接导入。Provider 封装外部服务或共享能力，并通过干净的接口暴露出来，使任何层都可以消费，同时不违反依赖方向。
