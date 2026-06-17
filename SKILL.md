---
name: agent-team
description: Use when 用户要求组建/派出一队 agent 协作执行一个可拆分的任务（关键词："agent team""智能体团队""派一队 agent""组个团队做""多个 agent 分工""并行派 agent 做"），需要指挥官统筹分工、监督进度与超时、并汇总结果时。本 skill 专用于 Hermes，以 kanban 任务板为骨架。
---

# Agent Team（智能体团队编排）

> 本 skill 专用于 **Hermes**，以原生 **`hermes kanban`** 任务板为骨架（持久化、任务可依赖、超时可控、按 profile 在隔离工作区执行）。kanban 板是**唯一事实源**——分工、状态、依赖、产出都落在板上，不要再用手写 Markdown 当真相。

## 概述

你（Hermes 指挥官 Agent）担任**指挥官**，不亲自做子任务，只负责：
**建板 → 给每个 worker 取人性化中文名（= assignee profile）并建带依赖/超时的任务 → dispatch 派出 → 监督事件流 → 汇总结果**。

核心映射（手搓什么都不用，kanban 全有）：

| 编排需求 | kanban 命令 |
|----------|-------------|
| 分工建卡 | `kanban create "<标题>" --assignee <中文名> --body "<职责+验收标准>" --max-runtime <秒>` |
| 并行队形（多 worker→校验→汇总） | `kanban swarm "<目标>" --worker 红小搜:调研 --worker 蓝小码:实现 --verifier 绿小验 --synthesizer 紫小文` |
| 依赖 | `kanban link <父id> <子id>`（子任务在父完成前自动 blocked），或 `create --parent <id>` |
| 超时 | 每卡 `--max-runtime <秒>`；`dispatch --failure-limit N` 连续超时/崩溃自动 block |
| 派出 | `kanban dispatch`（跑一次）/ `kanban daemon --interval 60`（持续） |
| 监督 | `kanban watch --kinds completed,blocked,gave_up,crashed,timed_out`、`list --status running`、`show <id>`、`tail <id>`、`stats` |
| 恢复 | `reclaim`（解卡死认领）/ `reassign` / `promote`·`unblock`（回 ready）/ `archive` |
| 产出 | worker `kanban complete <id> --result ... --summary ...`；指挥官 `list --status done` + `show <id>` 读取 |

## 何时使用

- 用户明确要求 "agent team / 智能体团队 / 派一队 agent / 多 agent 分工 / 并行派 agent 做"。
- 任务可拆成 **2 个以上相对独立** 的子任务/模块（能并行或弱依赖）。
- 典型场景：多文件代码修改、多模块功能开发、并行调研、复杂 bug 多点分析。

## 何时不要使用

- **单一简单任务**：直接做，不必组团。
- **强顺序依赖、无法拆分**：单线程线性执行更好。
- **纯咨询/对话/解释**：不涉及可分派的执行工作。
- **需要用户实时介入对话的任务**：worker 在隔离工作区跑，不与用户实时对话。
- **一次性、极轻量、不值得建板**：可不组团，或临时用会话内委派。

## 指挥官铁律

1. **指挥官不下场**：只编排、监督、汇总，不亲自写子任务的代码/内容。
2. **kanban 板是唯一事实源**：状态/依赖/产出都查板，别手工维护第二套真相。
3. **每个 worker 一个人性化中文名（颜色+小XX）= assignee profile**：见命名规范，花名册维护对照。
4. **建卡必须写全**：标题、`--assignee`、验收标准（写进 `--body`）、`--max-runtime`、依赖（`--parent`/`link`）。
5. **建卡 ≠ 派出**：`create`/`swarm` 只是建卡，必须 `dispatch` 或 `daemon` 才会真正跑；dispatch 只会跑 ready（无未决依赖）的卡。
6. **超时与卡死有兜底**：靠 `--max-runtime` + `dispatch --failure-limit` 自动 block，必要时 `reclaim`/`reassign`/`promote`，不放任。
7. **先汇报，后处理**：worker 超时/失败时先向用户汇报（已完成多少、原因），再决定续/换/放弃，**不自己补做**。

## 中文命名规范

目标：名字**既像真人名、又点明职能**，让团队有亲切的"上工"感。

**格式：颜色 + 小XX**，中文名同时就是 **assignee profile 名**。

| 颜色 | 示例 | 说明 |
|------|------|------|
| 红 | 红小搜、红小码 | 暖色系，活泼 |
| 蓝 | 蓝小搜、蓝小码 | 冷色系，稳重 |
| 绿 | 绿小验、绿小测 | 自然系，清新 |
| 黑 | 黑小审、黑小查 | 经典色，干练 |
| 白 | 白小文、白小档 | 简洁色，清爽 |
| 金 | 金小谋、金小设 | 贵气色，高端 |
| 紫 | 紫小文、紫小总 | 神秘色，独特 |

**落地（关键）**：中文名要作为 Hermes profile 存在，kanban 才能派给它执行。
```bash
hermes profile create 红小搜
hermes profile describe 红小搜 "调研岗：负责信息收集与资料调研，不写代码"   # 该描述供 kanban 编排器路由
```
- 同一团队内名字不重复；同职能用不同颜色区分重名（红小搜 / 蓝小搜）。
- 在板上或汇总里维护一张"花名册"对照表：中文名 ↔ profile ↔ 职能。
- 若不便为每个角色单独建 profile，可把中文名写进任务标题（如 `[红小搜] 调研X`），assignee 复用现有 profile。

## 工作流

### 0. 建板
```bash
hermes kanban init                      # 幂等，建 kanban.db
# 多项目隔离时：
hermes kanban boards create 我的项目 && hermes kanban boards switch 我的项目
```

### 1. 分工建卡
- **首选 swarm**（标准"并行 worker → 校验 → 汇总"队形，一条命令建好整张依赖图）：
```bash
hermes kanban swarm "做一份 X 速查卡" \
  --worker 红小搜:调研核心规则 --worker 蓝小例:写示例 \
  --verifier 绿小验 --synthesizer 紫小文
```
- **或手动建卡 + 连依赖**：
```bash
S=$(hermes kanban create "调研X" --assignee 红小搜 \
     --body "职责边界：只调研规则，不写代码。验收：列出≥3要点。" \
     --max-runtime 600 --json)        # 取回任务 id
hermes kanban create "整合成稿" --assignee 紫小文 --parent <S_id> --max-runtime 600
```
验收标准务必写进 `--body`，否则 worker 不知道做到什么算完。

### 2. 派出
```bash
hermes kanban dispatch                  # 跑一次，派出所有 ready 卡
# 或持续派出（长任务）：
hermes kanban daemon --interval 60 --failure-limit 2
```
有未决依赖的卡处于 blocked，父任务 done 后自动转 ready，下一轮 dispatch 才派出。

### 3. 监督（查询 + 事件流）
```bash
hermes kanban watch --kinds completed,blocked,gave_up,crashed,timed_out   # 事件流
hermes kanban list --status running     # 在跑哪些
hermes kanban show <id>                  # 单卡详情 + 评论 + 事件
hermes kanban tail <id>                  # 跟一张卡的事件
hermes kanban stats                      # 全局进度快照
```

### 4. 超时 / 卡死 / 失败处理

| 情况 | 处理 |
|------|------|
| 任务超 `--max-runtime` | kanban 记 `timed_out` 事件；连续超 `--failure-limit` 次自动 block |
| 认领卡死（claim 不释放） | `kanban reclaim <id> --reason "..."` 释放后重派 |
| 需要换人 | `kanban reassign <id> <新中文名>` |
| 被 block 但可继续 | 排障后 `kanban promote <id>` 或 `unblock <id>` 回 ready |
| 确定放弃 | `kanban archive <id>` |

**先汇报后处理**：超时/失败先告诉用户"X 已完成 Y%、原因 Z"，再决定续/换/放弃。

### 5. 汇总
- worker 完成时落产出：`kanban complete <id> --result "<结论>" --summary "<给下游的结构化交接>" --metadata '{"files":[...]}'`。
- 指挥官读取：`kanban list --status done` + `kanban show <id>`，向用户逐人汇报（做了什么、产出在哪、是否达标）。
- 需要给用户一份易读总览时，从 `kanban list --json` / `kanban stats` 生成一份 `TEAM-SUMMARY.md`——但**事实源永远是 kanban 板**，不要手工维护两套。

## 任务状态流转

`triage → todo → ready → running → review → done`；另有 `blocked`（依赖未决/超失败限）、`scheduled`（等时间）、`archived`（弃用）。

## 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|----------|
| 指挥官自己动手做子任务 | 角色错位、失去监督 | 只编排监督，不下场 |
| 手搓 Markdown 当事实源 | 与板不一致、双份维护 | kanban 为唯一事实源，文档只做导出镜像 |
| `create` 后不 `dispatch` | 任务永远停在 ready，不动 | 建卡后必须 dispatch 或开 daemon |
| 依赖靠手工排序、不用 `link`/`--parent` | 下游拿不到上游产出 | 用依赖边，blocked 会自动转 ready |
| 不设 `--max-runtime` | 卡死无兜底 | 每卡设超时 + dispatch `--failure-limit` |
| 验收标准没写进 `--body` | worker 不知道何为"完成" | `--body` 写清职责边界 + 可验证验收标准 |
| 超时/失败指挥官自己补做 | 失去团队意义、角色错位 | 先汇报，再 reclaim/reassign/换方式/问用户 |
| 中文名不是 profile / 无对照 | dispatch 派不出去、汇报混乱 | 中文名建为 profile（或写进标题），维护花名册 |
| 不取中文名、用编号 | 汇报枯燥、难追踪 | 每个 worker 必须有人性化中文花名（颜色+小XX） |

## 小抄（一页流）

```bash
hermes kanban init
hermes profile create 红小搜 && hermes profile describe 红小搜 "调研岗"
hermes kanban swarm "目标" --worker 红小搜:调研 --worker 蓝小码:实现 --verifier 绿小验 --synthesizer 紫小文
hermes kanban dispatch
hermes kanban watch --kinds completed,blocked,timed_out,crashed   # 盯事件
hermes kanban list --status done && hermes kanban show <id>       # 收产出
```
