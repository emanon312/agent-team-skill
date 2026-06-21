# Agent Team Skill 优化方案

> 借鉴 Metaskill（xvirobotics/metaskill），结合实跑验证的 Hermes 能力边界。

---

## 一、Metaskill 可取之处

经过对 Metaskill 源码和 README 的分析，以下能力值得借鉴：

| Metaskill 能力 | 核心思路 | 对 Agent Team 的启发 |
|---|---|---|
| 4 阶段研究驱动 | 建团队前先 web search 调研领域最佳实践 | **指挥官先做功课，再分工** |
| 意图自动路由 | 一句话识别"组团/单人/skill"三种需求 | **降低触发门槛，一条命令搞定** |
| 一键脚手架 | 说"iOS app"就吐出完整目录+配置 | **自动生成 TEAM-PLAN.md + kanban 板** |
| 领域定制 | 不同技术栈配不同角色、工具、规范 | **拆任务不靠猜，靠调研** |
| 预置示例 | examples/ 目录放现成可复制的团队配置 | **提供常用场景的参考花名册** |
| MCP 自动集成 | 根据技术栈选 MCP server 写 .mcp.json | **根据角色自动建议 toolsets/skills** |

---

## 二、优化方案（三个迭代）

### Phase 1：研究驱动分工（低风险，立即可做）

**现状问题：** 当前 skill 告诉指挥官"拆分任务 → 写 TEAM-PLAN.md"，但没告诉指挥官**怎么拆**。遇到不熟悉的领域（比如"做个 iOS AR 应用"），指挥官只能瞎猜角色。

**方案：** 在"拆分任务"前插入一个 **RESEARCH 步骤**：

```
0. 理解任务 → web_search 该领域典型团队结构 + 最佳实践
1. 根据调研结果拆分任务 + 确定角色
2. 写 TEAM-PLAN.md
3. 派出
...
```

**具体规则：**
- 收到任务后，先用 web_search 做 2-3 轮调研：
  - 搜索 "`{领域}` 开发团队结构 / tech stack / best practices"
  - 搜索 "`{领域}` 常见角色分工"
  - 搜索 "`{领域}` linter / testing framework"
- 调研结果写入 `.agent-team/RESEARCH.md`
- 根据调研结果确定角色（不是用死板的"调研→实现→测试"模板）
- 再走现有流程

**产出：** `.agent-team/RESEARCH.md`（任务完成后保留，供复盘参考）

**改动范围：** SKILL.md 加一个 RESEARCH 步骤 + 一份调研模板，不动现有流程骨架。

---

### Phase 2：意图路由 + 轻量模式（中风险）

**现状问题：** 当前 skill 只有"全职团队"一条路径。用户说"帮我调研一下 X 技术"，也被迫走完整的三文档 + 指挥官 + 花名册流程，太重。

**方案：** 根据用户意图自动选择模式：

| 用户说什么 | 走哪条路 | 特点 |
|---|---|---|
| "派一队 agent 做 X" / "组个团队" | **Full Team 模式** | 研究 → 分工 → kanban/swarm → 监督 → 汇总 |
| "帮我调研 X" / "查一下 Y" | **Quick Research 模式** | 2-3 个搜索 agent 并行，直接汇总，不建板 |
| "帮我 review 这段代码" | **Single Agent 模式** | 派一个审查 agent，完成即汇报 |

**Quick Research 模式示例：**
```
用户："帮我调研一下 WebAssembly 在 2026 年的生态"

指挥官不建板，直接：
1. 派 红小搜 → 搜索 WASM 运行时/工具链
2. 派 蓝小搜 → 搜索 WASM 应用案例/公司采用情况
3. 派 绿小搜 → 搜索 WASM 性能对比/基准测试
4. 三路并行，完成后由指挥官汇总成一份调研报告
```

**改动范围：** SKILL.md 增加意图识别规则 + Quick Research / Single Agent 流程。

---

### Phase 3：预置领域模板（后期，积累驱动）

**思路：** 用户对领域模板的需求是"懒人想复制"。不用一上来就写一堆，而是**每实战一次，沉淀一个模板**。

**模板结构：**
```
.agent-team/templates/
├── fullstack-web.md       # 红小前 + 蓝小后 + 绿小验 + 紫小文
├── ios-app.md             # 红小iOS + 蓝小UI + 绿小测 + 紫小审
├── data-pipeline.md       # 红小数 + 蓝小管 + 绿小验 + 紫小文
├── research-report.md     # 红小搜 + 蓝小搜 + 紫小文（Quick Research 模式）
└── code-review.md         # 红小审（Single Agent 模式）
```

每份模板包含：推荐角色列表、各角色 skills 建议、典型验收标准、预估耗时。

**改动范围：** 新建 `templates/` 目录 + SKILL.md 尾部引用模板索引。

---

## 三、与 PR #4（kanban 重写）的关系

PR #4 的 kanban 骨架重写是更底层的基础设施升级。上述优化方案**不依赖 PR #4**——它们改的是"指挥官怎么思考、怎么拆任务"，而不是"用什么工具管任务"。

建议的合并路线：
1. **Phase 1（研究驱动）**——立即做，改现有 master
2. **PR #4（kanban 骨架）**——Phase 1 稳定后，rebase PR #4 到新 master
3. **Phase 2（意图路由）**——kanban 版稳定后加
4. **Phase 3（模板积累）**——用一次加一个，自然生长

---

## 四、不改的东西

以下 Metaskill 的特点**不适合 Agent Team Skill**，不借鉴：

| Metaskill 做法 | 为什么不借鉴 |
|---|---|
| 生成 `.claude/agents/*.md` 静态角色卡 | Hermes 的 agent 是动态派出的，不存在"预定义 agent 文件"这个概念 |
| 生成 `.mcp.json` MCP 配置 | Hermes 的 MCP 在 config.yaml 里统一管，不需要 skill 碰 |
| 生成 `CLAUDE.md` 路由表 | Hermes 没有 CLAUDE.md 这个概念；路由靠指挥官脑子 + skill 规则 |
| 要求安装 Claude Code CLI | 不相关 |
| Claude 专有 slash command `/metaskill` | Hermes 用 skill 的 description 字段做关键词触发，不用 slash command |

---

## 五、优先级建议

| 阶段 | 内容 | 工作量 | 风险 | 建议时间 |
|---|---|---|---|---|
| Phase 1 | 研究驱动分工 | ~30 行加到 SKILL.md | 低 | 现在 |
| PR #4 | kanban 骨架重写 | 已完成，需修 deprecation | 中 | Phase 1 后 |
| Phase 2 | 意图路由 + 轻量模式 | ~60 行 | 中 | kanban 稳定后 |
| Phase 3 | 模板积累 | 每次实战 +20 行 | 低 | 随用随加 |
