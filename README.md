# Agent Team Skill

Hermes / Claude Code Agent 智能体团队编排 Skill。

> 本仓库包含两个版本：
> - **根目录 SKILL.md** — Hermes 版（异步后台派出 + 事件驱动监督）
> - **[claude-code/SKILL.md](claude-code/SKILL.md)** — Claude Code 版（Agent 工具同步等待 + 无需轮询）
>
> 两者共享 `templates/` 目录下的模板文件。

## 功能

当用户需要多个 Agent 协作完成复杂任务时，使用此 Skill 进行：

1. **任务拆分**：将大任务拆分为独立子任务
2. **分工协作**：为每个 Agent 分配角色和职责
3. **进度监督**：跟踪各 Agent 的执行状态和产出
4. **超时管理**：处理超时和阻塞情况
5. **结果汇总**：收集并整合所有 Agent 的输出

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
- 需要用户实时交互的任务

## 核心概念

### 指挥官
- 不亲自做子任务
- 只负责编排、监督、记录、汇总
- 维护四份活文档

### Agent 命名
- 每个 Agent 都有中文名字
- 格式：颜色 + 小XX（红小搜、蓝小码、绿小验）
- 颜色用于区分同职能的多个 Agent，并让团队有亲切的"上工"感

### 四份文档
1. `RESEARCH.md` - 领域调研文档
2. `TEAM-PLAN.md` - 任务分工表
3. `TEAM-PROGRESS.md` - 进度跟踪表
4. `TEAM-RESULTS.md` - 结果汇总表

## 安装

### Hermes 版

```bash
cp SKILL.md ~/.hermes/skills/productivity/agent-team/
```

### Claude Code 版

```bash
# 创建 skill 目录
mkdir -p ~/.claude/skills/agent-team/templates

# 下载 SKILL.md
curl -o ~/.claude/skills/agent-team/SKILL.md \
  https://raw.githubusercontent.com/emanon312/agent-team-skill/master/claude-code/SKILL.md

# 下载模板（可选，用于快速参考）
curl -o ~/.claude/skills/agent-team/templates/TEMPLATE-FORMAT.md \
  https://raw.githubusercontent.com/emanon312/agent-team-skill/master/templates/TEMPLATE-FORMAT.md
curl -o ~/.claude/skills/agent-team/templates/game-2d-godot.md \
  https://raw.githubusercontent.com/emanon312/agent-team-skill/master/templates/game-2d-godot.md
curl -o ~/.claude/skills/agent-team/templates/quick-research.md \
  https://raw.githubusercontent.com/emanon312/agent-team-skill/master/templates/quick-research.md
curl -o ~/.claude/skills/agent-team/templates/single-review.md \
  https://raw.githubusercontent.com/emanon312/agent-team-skill/master/templates/single-review.md
```

## 作者

张大炮 (emanon312)

## License

MIT
