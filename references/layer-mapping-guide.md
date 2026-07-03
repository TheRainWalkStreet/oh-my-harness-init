# 层级映射指南

用真实代码结构映射层级，不强行套模板。文档中优先使用项目已有命名。

## 通用步骤

1. 找入口：路由、Controller、Handler、页面、命令行入口、任务入口。
2. 找业务承载层：Service、Usecase、Action、Domain、Application。
3. 找数据和外部系统：Repository、DAO、Client、Adapter、Infra、Gateway。
4. 找共享代码：utils、common、shared、lib、packages。
5. 读取 import/use 关系，确认依赖方向。
6. 标记不清晰边界为 `待确认`。

## 前端项目

常见层级：

- 路由/页面：页面组合、数据入口、布局。
- 组件：展示组件、业务组件、表单组件。
- 状态：store、hooks、context、query cache。
- 服务：API client、SDK wrapper、数据访问。
- 工具：纯函数、格式化、类型、常量。

关注：

- Server/Client 边界。
- UI 与数据访问边界。
- 状态管理和副作用位置。
- 共享组件和业务组件边界。

## 后端 API

常见层级：

- 接口层：Controller、Router、Handler、View。
- 应用服务层：Service、Usecase、Command/Query。
- 领域层：Domain、Entity、Value Object、Policy。
- 持久化层：Repository、DAO、Mapper。
- 基础设施层：DB、Queue、Cache、External Client、Config。

关注：

- Controller/Handler 不承载核心业务规则。
- 领域层不依赖框架和外部系统。
- 外部系统调用集中封装。
- 数据转换边界清晰。

## 前后端一体项目

映射：

- 前端 UI/状态/API 客户端。
- 后端 Controller/Service/Domain/Repository。
- 模板或静态资源托管层。
- API 契约和共享类型。
- 构建产物和部署边界。

关注：

- 前端不能绕过 API 客户端散落请求。
- 后端不能把核心业务规则塞进模板层。
- 认证态、CSRF、CORS、环境变量暴露必须写清楚。

## Monorepo

映射：

- apps：可运行应用。
- packages/libs：共享包。
- tools：构建、生成、脚本。
- configs：共享配置。

关注：

- 跨包依赖方向。
- 公共包职责边界。
- 构建和测试入口。
- 发布或部署单元。

## 输出建议

在 `ARCHITECTURE.md` 中写：

- 当前层级表。
- 每层职责。
- 允许依赖。
- 禁止依赖。
- 例外和待确认点。
- 修改代码时应放置的位置。
