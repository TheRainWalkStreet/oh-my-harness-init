# 发现清单

在写任何文档前完成发现。目标是用仓库事实建立项目画像。

## 基础状态

- 当前路径和仓库根目录。
- `git status --short --branch`。
- 是否存在未提交改动；不要覆盖用户改动。
- 顶层目录和深度 2-3 的源码目录。
- 现有文档：`README.md`、`AGENTS.md`、`CLAUDE.md`、`docs/`、架构文档、规范文档。

## Harness 状态

检查已有 harness 文件：

- `AGENTS.md`
- `ARCHITECTURE.md`
- `docs/ENGINEERING.md`
- `docs/SECURITY.md`
- `docs/standards/coding.md`
- `docs/standards/commits.md`
- `docs/QUALITY_SCORE.md`
- `docs/design-docs/`
- `docs/exec-plans/`

据此判断：

- Uninitialized：缺少核心 harness 文件。
- Lite：存在 Lite 核心文件，且没有 Standard 文件。
- Standard：存在 Standard 核心文件。
- Mixed：文件不完整、结构不一致或新旧结构混杂。

Mixed 状态需要先报告并询问用户整理方向，不要静默覆盖。

## 技术栈信号

检查常见配置文件：

- JavaScript/TypeScript：`package.json`、`pnpm-lock.yaml`、`yarn.lock`、`package-lock.json`、`tsconfig.json`、`next.config.*`、`vite.config.*`。
- Java/Kotlin：`pom.xml`、`build.gradle`、`settings.gradle`、`gradle.properties`。
- Python：`pyproject.toml`、`requirements.txt`、`poetry.lock`、`manage.py`。
- Go：`go.mod`、`go.sum`、`cmd/`、`internal/`。
- Mobile：`pubspec.yaml`、`android/`、`ios/`、`*.xcodeproj`、`Package.swift`。
- .NET/PHP/Ruby：`*.csproj`、`composer.json`、`Gemfile`。

## 命令信号

只记录真实存在的命令，并记录覆盖范围：

- 安装依赖。
- 本地启动。
- 测试。
- lint。
- format。
- build。
- 数据库迁移或代码生成。

若命令存在于多个入口，记录权威入口和备用入口。

对测试、lint、format、build 命令额外检查：

- 覆盖哪些目录或文件。
- 是否覆盖全部源码。
- 是否只覆盖单个入口文件。
- 新增源码目录或业务模块后是否需要更新命令。
- 覆盖范围不清楚时标记为 `待确认`。

不要因为存在命令就假设质量门禁完整。v1.3.0 不自动改造命令，只记录覆盖风险并提醒用户。

## 安全与配置信号

检查但不复制敏感值：

- `README.md`、`.env*`、`docker-compose.yml`、`Dockerfile`、配置源码、部署脚本中的 key、token、password、secret。
- 内部 IP、内网域名、生产账号、真实业务样本、真实用户数据。
- 日志目录、测试样例、静态测试页面中是否包含敏感内容。
- CORS、认证授权、会话、缓存、外部 API、文件上传或图片分析相关边界。

如果发现明文敏感配置，只在生成文档中写“存在明文敏感配置迹象”和配置类别，不粘贴具体值。

## 架构信号

- 入口文件和路由入口。
- 主要模块、包、应用或服务目录。
- import/use 关系。
- Controller/Handler/View、Service/Usecase、Domain/Model、Repository/DAO、Infra/Adapter 等职责痕迹。
- 前后端一体痕迹：模板目录、静态资源目录、API 路由、前端构建产物、统一部署脚本。
- monorepo 痕迹：workspace、apps/packages、多个可独立发布单元。

## Lite 升级信号

Lite 项目出现以下多个信号时，建议升级 Standard：

- 已出现清晰业务模块或领域概念。
- 已有稳定测试、lint、build 流程。
- 有多人协作迹象。
- 出现技术债或复杂改造需求。
- 有架构决策需要记录。
- 出现多个模块之间的依赖边界。
- 出现生产部署、外部 API、数据库或权限边界。

## 停止并询问

遇到以下情况先问用户：

- 同时存在多个互相冲突的包管理器或构建系统。
- 源码目录和运行入口无法判断。
- 现有文档与代码事实明显冲突。
- 写入新结构会覆盖重要已有文档。
- 目标项目接近空仓库或还没有框架骨架。
