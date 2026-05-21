# CI 模板

阶段 5 的入门模板。根据 `references/stack-routing.md` 阶段 5 表格适配命令。

## 命令验证

在将发现的命令替换到 CI YAML 的 `run:` 字段（以及任何嵌入的脚本字符串）之前，先验证它们：
- **允许：** 已知的构建/测试/Lint 工具（`npm`、`npx`、`eslint`、`prettier`、`jest`、`vitest`、`tsc`、`ruff`、`pytest`、`mypy`、`go`、`golangci-lint`、`cargo`、`clippy`、`gradle`）
- **允许链接：** 已知安全命令之间用 `&&` 连接没问题（例如 `cd subdir && npm test`）
- **拒绝：** `|`（管道）、`;`、`$()`、`` ` ``、`>>`、`curl`、`wget`、`eval`、`exec` —— 这些表示潜在的注入风险
- 如果发现的命令看起来可疑或不符合预期模式，**停下来询问**

## GitHub Actions（.github/workflows/ci.yml）

~~~yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: {setup-action}          # 必须 SHA 固定，参见下方"Action 固定"说明
      - run: {安装命令}
      - run: {Lint 命令}

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: {setup-action}
      - run: {安装命令}
      - run: {类型检查命令}

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: {setup-action}
      - run: {安装命令}
      - run: {测试命令}

  build:
    runs-on: ubuntu-latest
    needs: [lint, typecheck, test]
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: {setup-action}
      - run: {安装命令}
      - run: {构建命令}
~~~

**说明：** 如果技术栈没有单独的类型检查步骤（Go、Rust），移除 `typecheck` 任务。如果不是打包/部署的制品，移除 `build` 任务。

### Action 固定

所有 `uses:` 引用**必须**用 SHA 固定，并附上标签注释以便审计。永远不要使用裸标签引用（`@v4`）—— 标签是可变的，容易受到供应链攻击。

> **⚠️ SHA 有效期提醒：** 以下 SHA 值为模板示例，**使用时请自行核查最新版本**。可通过以下方式获取最新 SHA：
> ```bash
> # 查询指定 tag 对应的 SHA（以 setup-node v4 为例）
> gh api /repos/actions/setup-node/commits/refs/tags/v4 --jq '.sha'
> ```

常用 setup action（请使用时核查最新 SHA）：
- `actions/setup-node@{SHA} # v4.x.x`
- `actions/setup-python@{SHA} # v5.x.x`
- `actions/setup-go@{SHA} # v5.x.x`
- `actions/setup-java@{SHA} # v4.x.x`

## GitLab CI（.gitlab-ci.yml）

~~~yaml
default:
  image: {运行时镜像}  # 例如 node:20、python:3.12、golang:1.22

stages: [lint, typecheck, test, build]

lint:
  stage: lint
  script:
    - {安装命令}
    - {Lint 命令}

typecheck:
  stage: typecheck
  script:
    - {安装命令}
    - {类型检查命令}

test:
  stage: test
  script:
    - {安装命令}
    - {测试命令}

build:
  stage: build
  script:
    - {安装命令}
    - {构建命令}
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
~~~

**说明：** 如果技术栈没有单独的类型检查步骤（Go、Rust），移除 `typecheck` stage。

> **Gitee（码云）说明：** Gitee Actions 语法与 GitHub Actions 基本一致，但存在以下差异：
> - 自托管 Runner 配置不同，镜像名可能有所区别
> - 部分第三方 Action 在 Gitee 上不可用，建议优先使用官方基础 Action 或 Makefile 命令替代
> - 目前无需 SHA 固定（Gitee Actions 不存在供应链攻击风险）

## Makefile 降级方案（无 CI 平台时）

如果仓库不使用 GitHub/GitLab，提供 Makefile 让 `make ci` 在本地运行所有检查：

~~~makefile
.PHONY: lint typecheck test build gc ci

# 加 @ 前缀可以抑制命令回显（不打印命令本身，只打印输出）
# 不加 @ 前缀则命令本身也会打印，便于调试时查看执行了哪条命令
# 建议 CI 模式下加 @，本地调试时可去掉

lint:
	@{Lint 命令}

typecheck:
	@{类型检查命令}

test:
	@{测试命令}

build:
	@{构建命令}

gc:
	@{GC 命令}

ci: lint typecheck test build
~~~

## GC 工作流（.github/workflows/gc.yml）

~~~yaml
name: 垃圾回收
on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一 UTC 9:00
  workflow_dispatch:       # 允许手动触发

permissions:
  contents: read
  issues: write

jobs:
  gc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: {setup-action}
      - run: {安装命令}
      - run: {GC 命令}
      - name: 失败时创建 Issue
        if: failure()
        uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7.0.1
        with:
          script: |
            const title = `GC 扫描发现问题 — ${new Date().toISOString().slice(0, 10)}`;
            const { data: existing } = await github.rest.issues.listForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: 'garbage-collection',
              state: 'open',
            });
            if (existing.some(i => i.title === title)) {
              return; // 今天已经报告过了
            }
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title,
              labels: ['garbage-collection'],
              body: '每周 GC 扫描检测到熵增。请在本地运行 Makefile 或 package.json 中的 GC 命令查看详情。'
            });
~~~

**说明：** GC 工作流仅报告 —— 永远不会自动修复。`workflow_dispatch` 触发器允许手动运行。请在仓库中创建 `garbage-collection` 标签。
