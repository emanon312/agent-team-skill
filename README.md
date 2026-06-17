# Agent Team Skill

Hermes Agent 智能体团队编排 Skill，以原生 `hermes kanban` 任务板为骨架。

## 功能

当用户需要多个 Agent 协作完成复杂任务时，指挥官用此 Skill：

1. **任务拆分**：把目标拆成带依赖的 kanban 任务（`kanban create` / `swarm`）
2. **分工协作**：每个 worker 一个人性化中文名（颜色+小XX）= assignee profile
3. **派出与监督**：`kanban dispatch` 派出，`kanban watch / list / show / stats` 监督事件流
4. **超时管理**：`--max-runtime` + `dispatch --failure-limit` 兜底，`reclaim`/`reassign` 恢复
5. **结果汇总**：worker `kanban complete` 落产出，指挥官读板汇报

## 为什么用 kanban

Hermes 自带 kanban 任务板（SQLite 持久化、任务可依赖、超时可控、按 profile 在隔离工作区执行），
提供了分工 / 依赖 / 超时 / 监督 / 恢复 / 汇总的全套原语——**无需手搓 Markdown 重造轮子**。
kanban 板是唯一事实源，需要时再从 `kanban list --json` / `stats` 导出一份人读汇总。

## 使用场景

### 适用场景
- 多文件代码修改（前端 + 后端 + 测试）
- 多模块功能开发（研究 + 设计 + 实现）
- 并行调研任务（多个信息源同时搜索）
- 复杂 bug 修复（多个日志/文件同时分析）

### 不适用场景
- 单一简单任务
- 强顺序依赖、无法拆分的任务
- 纯咨询/对话/解释
- 需要用户实时介入对话的任务

## 核心概念

### 指挥官
- 不亲自做子任务，只编排、监督、汇总
- kanban 板是唯一事实源

### Agent 命名
- 每个 worker 一个中文名，格式：颜色 + 小XX（红小搜、蓝小码、绿小验）
- 中文名 = assignee profile（`hermes profile create <名>`），颜色用于区分同职能重名

### 编排骨架
- `kanban create` / `swarm` 建卡分工
- `kanban link` / `--parent` 连依赖
- `kanban dispatch` / `daemon` 派出
- `kanban watch` / `list` / `show` / `tail` / `stats` 监督

## 安装

将 `SKILL.md` 复制到 Hermes skills 目录：

```bash
cp SKILL.md ~/.hermes/skills/productivity/agent-team/
```

## 作者

张大炮 (emanon312)

## License

MIT
