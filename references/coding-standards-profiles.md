# 代码规范适配规则

本文件提供生成 `docs/standards/coding.md` 的规范素材。按项目事实裁剪，不要全部照搬。

## 优先级

1. 项目已有工具和配置：ESLint、Prettier、Ruff、Black、mypy、Checkstyle、Spotless、golangci-lint、ktlint、SwiftLint、dart analyze。
2. 项目已有文档和历史约定。
3. 本 skill 的通用工程规范内核。
4. 语言专项规则。
5. 框架专项规则。

若规则冲突，以项目事实为准，并标记冲突为 `待确认`。

## 通用工程规范内核

- 清晰命名：名称说明意图，而不是说明实现细节。
- 小职责：函数、组件、类和模块保持单一职责。
- 明确边界：UI/接口层不承载核心业务规则。
- 可诊断错误：错误信息包含上下文，不泄露敏感数据。
- 显式校验：外部输入、环境变量、API 响应、数据库结果都视为不可信。
- 集中封装：外部 API、数据库、缓存、队列、文件系统等集中在适配层。
- 测试关键路径：覆盖业务规则、错误路径、权限边界和数据转换。

## Java / Spring Boot

- Java 项目可参考阿里巴巴 Java 开发手册，但只提炼适用规则。
- Controller 只处理协议、参数和响应，不写核心业务。
- Service 承载应用用例和事务边界。
- Repository/Mapper 只处理持久化。
- DTO、Entity、VO、Command/Query 职责分开。
- 异常处理集中，日志不打印敏感字段。

## TypeScript / JavaScript

- 使用现有 tsconfig 和 lint 规则作为权威。
- 公共类型放在清晰位置，避免跨层随意导出。
- 异步调用必须处理错误状态。
- API 响应在边界处解析或校验。
- 避免 `any` 扩散；无法避免时说明原因。

## React / Next.js / Vue

- 区分页面、组件、状态、服务、工具。
- 组件不直接散落请求逻辑，优先调用 API client 或服务层。
- 复杂状态集中管理，避免跨组件隐式共享。
- Next.js 项目写清 Server/Client 组件边界和数据加载位置。
- 表单、权限和错误状态必须有明确处理。

## Python / FastAPI / Django

- 使用现有 formatter/linter/type checker 规则。
- Router/View 处理协议和参数，不写核心业务。
- Service/Domain 承载业务规则。
- Schema/Serializer 处理输入输出边界。
- ORM 查询集中在 Repository/Manager/DAO 或明确的数据访问层。

## Go

- package 名称表达职责，避免 `utils` 泛化。
- error wrapping 保留上下文。
- context 传递遵循调用链，不滥存到结构体。
- 接口定义靠近使用方。
- `internal/`、`cmd/`、`pkg/` 的职责写清楚。

## Kotlin / Android

- 区分 UI、ViewModel、UseCase、Repository、Data Source。
- 生命周期、协程作用域和资源释放必须清楚。
- 权限、存储和网络状态显式处理。

## Swift / iOS

- 区分 View、ViewModel/State、Service、Repository。
- 主线程和异步任务边界明确。
- 权限、生命周期和错误状态显式处理。

## Dart / Flutter

- 区分 Widget、State、Service、Repository。
- 不在 Widget 中堆业务逻辑。
- 异步状态、错误状态、加载状态必须可见。

## 前后端一体

- 前端通过统一 API client 访问后端。
- 后端不把核心业务规则塞进模板或静态资源处理层。
- 共享类型、接口契约、鉴权状态和构建产物边界必须写清楚。
- 环境变量区分服务端可见和浏览器可见。
