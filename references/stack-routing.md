# 技术栈路由 —— 阶段 3-7 工具决策表

使用这些表格为检测到的技术栈选择正确的工具。阶段 0 发现确定技术栈；这些表格确定执行方式。

## 阶段 3：边界测试 —— Import 解析器与模式

| 技术栈          | Import 模式                          | 解析方式                            | 测试文件                                    |
|-----------------|--------------------------------------|-------------------------------------|---------------------------------------------|
| JS/TS           | `import ... from '...'`              | 正则表达式或 AST（ts-morph、babel） | `tests/architecture/boundary.test.ts`       |
| Python          | `import ...` / `from ... import`     | AST（标准库 `ast` 模块）            | `tests/architecture/test_boundary.py`       |
| Go              | `import "..."`                       | `go/parser` 标准库或正则            | `tests/architecture/boundary_test.go`       |
| Rust            | `use ...` / `mod ...`                | 正则表达式或 `syn` crate（较重）    | `tests/architecture/boundary_test.rs`       |
| Java            | `import ...`                         | ArchUnit（推荐）或正则              | `tests/architecture/BoundaryTest.java`      |
| Kotlin          | `import ...`                         | ArchUnit（推荐）或正则              | `tests/architecture/BoundaryTest.kt`        |

**报错格式（所有技术栈统一）：**
`VIOLATION: {文件}:{行号} imports {目标} — {层级} 不能导入 {目标层级}。参见 docs/architecture/LAYERS.md`

## 阶段 4：Linter Import 限制规则

| 技术栈              | Linter         | 规则                                                | 配置位置                              |
|---------------------|----------------|-----------------------------------------------------|---------------------------------------|
| JS/TS（ESLint）     | eslint         | `no-restricted-imports` / `import/no-restricted-paths` | `.eslintrc` 或 `eslint.config.js`  |
| Python（Ruff）      | ruff           | `TID251`（`flake8-tidy-imports` 插件）              | `pyproject.toml [tool.ruff.lint]`    |
| Python（Flake8）    | flake8         | `flake8-import-restrictions`                        | `.flake8` 或 `setup.cfg`             |
| Go                  | golangci-lint  | `depguard`                                          | `.golangci.yml`                       |
| Rust                | clippy         | `pub(crate)` 可见性 + workspace deps                | `Cargo.toml` + 模块结构              |
| Java/Kotlin         | ArchUnit       | `ArchRuleDefinition.noClasses()`                    | 测试文件（ArchUnit 是基于测试的）     |

> **Ruff 配置示例（TID251）：**
> ```toml
> [tool.ruff.lint]
> select = ["TID"]
>
> [tool.ruff.lint.flake8-tidy-imports.banned-module-level-imports]
> "src.services" = "services 层禁止被 components 层导入。参见 docs/architecture/LAYERS.md"
> ```

**关键规则：** 每条 Linter 报错**必须**包含修复指引。报错输出就是 Agent 的上下文。

## 阶段 5：CI 任务矩阵

| 技术栈          | Lint                    | 类型检查             | 测试                        | 构建                              |
|-----------------|-------------------------|----------------------|-----------------------------|-----------------------------------|
| JS/TS           | `eslint .`              | `tsc --noEmit`       | `jest` / `vitest`           | `next build` / `tsc`              |
| Python          | `ruff check .`          | `mypy .`（如有类型） | `pytest`                    | `python -m build`（如打包发布）   |
| Go              | `golangci-lint run`     | （包含在构建中）      | `go test ./...`             | `go build ./...`                  |
| Rust            | `cargo clippy`          | （包含在构建中）      | `cargo test`                | `cargo build --release`           |
| Java/Kotlin     | `checkstyle` / `ktlint` | （编译型语言）        | `./gradlew test`            | `./gradlew build`                 |

**并非所有技术栈都需要全部 4 个任务。** Go 和 Rust 将类型检查与构建合并。如果 Python 不发布包，可以跳过构建。参考 `references/ci-templates.md` 获取入门 YAML。

## 阶段 6：垃圾回收工具

| 技术栈      | Import 扫描器                      | 文档漂移检查                                    | GC 执行命令                                  | 配置                       |
|-------------|------------------------------------|-------------------------------------------------|----------------------------------------------|----------------------------|
| JS/TS       | `ts-morph` 或正则匹配 import       | 通过 `git log` 比较 `docs/` 和 `src/` 时间戳   | `npm run gc` 或 `npx tsx scripts/gc/run.ts`  | `package.json` scripts     |
| Python      | 标准库 `ast` 模块                  | 通过 `git log` 比较 `docs/` 和 `src/` 时间戳   | `python scripts/gc/run_all.py` 或 `make gc`  | `pyproject.toml` 或 Makefile |
| Go          | `go/parser` 标准库                 | 通过 `git log` 比较 `docs/` 和源码时间戳        | `go run scripts/gc/main.go` 或 `make gc`     | Makefile                   |
| Rust        | 正则匹配 `use`/`mod` 语句          | 通过 `git log` 比较 `docs/` 和 `src/` 时间戳   | `cargo run --bin gc` 或 `make gc`            | `Cargo.toml` [[bin]]       |
| Java/Kotlin | 正则匹配 import 语句               | 通过 `git log` 比较 `docs/` 和 `src/` 时间戳   | `./gradlew gc` 或 `make gc`                  | `build.gradle` 或 Makefile |

**最小可行 GC：** 架构违规扫描 + 文档-代码漂移检查。其他扫描（文件大小、TODO 数量、未使用 import）是可选增强。

## 阶段 7：Pre-commit 框架

| 技术栈      | 框架                          | 配置文件                    | 主要 Hook                                                              |
|-------------|-------------------------------|-----------------------------|------------------------------------------------------------------------|
| JS/TS       | husky + lint-staged           | `.husky/`、`package.json`   | `lint-staged: { "*.ts": ["eslint --fix", "prettier --write"] }`       |
| Python      | pre-commit                    | `.pre-commit-config.yaml`   | `ruff`、`mypy`、`black`                                                |
| Go          | golangci-lint（无需框架）      | `.golangci.yml`             | 作为 `pre-commit` git hook 或 Makefile target 运行                    |
| Rust        | cargo-husky 或自定义          | `.cargo-husky/`             | `cargo fmt --check`、`cargo clippy`                                    |
| Java/Kotlin | Gradle spotless 或 pre-commit | `build.gradle`              | `spotlessApply`、`ktlintFormat`                                        |

**阶段 7 是可选的。** 只有在团队需要本地约束时才添加。CI（阶段 5）是权威的质量门控。
