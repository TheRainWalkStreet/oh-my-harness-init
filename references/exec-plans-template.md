# exec-plans 模板

用于 Standard/Extended 的 `docs/exec-plans/`。

## index.md

````markdown
# 执行计划

## 什么时候需要执行计划

- 多步骤重构。
- 跨模块功能。
- 数据迁移。
- 安全或可靠性调整。
- 需要中断后恢复上下文的任务。

## 文件命名

使用 `YYYY-MM-DD-任务简述.md`。

## 生命周期

1. 新计划放入 `docs/exec-plans/active/`。
2. 执行中持续更新进度、决策、风险、待确认事项。
3. 完成后移动到 `docs/exec-plans/completed/`。
4. 长期技术债进入 `docs/exec-plans/tech-debt.md`。
````

## active/README.md

````markdown
# Active Exec Plans

这里存放正在执行的计划。

## 命名

使用 `YYYY-MM-DD-任务简述.md`。

## 计划文件必须包含

- 背景。
- 目标。
- 非目标。
- 步骤。
- 当前进度。
- 决策记录。
- 风险和待确认事项。
- 验证计划。
````

## completed/README.md

````markdown
# Completed Exec Plans

这里存放已完成计划。

完成归档前，计划文件必须补齐：

- 最终结果。
- 实际变更。
- 验证情况。
- 遗留问题。
- 后续建议。
````

## tech-debt.md

````markdown
# 技术债

| 编号 | 问题 | 影响 | 建议 | 优先级 | 状态 |
| --- | --- | --- | --- | --- | --- |
| `{ID}` | `{问题}` | `{影响}` | `{建议}` | `{P0-P3}` | `{open/doing/done}` |
````
