# Godot 2D 平台游戏开发

> 适用场景：用 Godot 引擎开发 2D 平台跳跃/动作游戏 | 模式：Full Team

## 花名册

| 中文名 | 角色 | 职责 | 建议 skills |
|--------|------|------|------------|
| 红小谋 | 游戏设计师 + 制作人 | 玩法机制、关卡设计、UI/UX、进度跟踪 | — |
| 蓝小码 | Godot 程序员 | GDScript 脚本、玩家控制器、敌人 AI、物理、场景搭建 | terminal, file |
| 绿小画 | 2D 美术 + 音效 | 角色/场景精灵图、TileSet、动画、UI 素材、SFX/配乐集成 | — |
| 金小验 | 测试 | 功能测试、Bug 报告、平衡性验证 | terminal |

## 依赖关系

```
红小谋（GDD/关卡设计文档）
   ↓
蓝小码 ← → 绿小画（并行：程序 + 美术可同时开工，但需对齐接口）
   ↓
金小验（拿到可玩版本后测试）
```

## 典型验收标准

- **红小谋**：产出 Game Design Document（含核心机制、关卡结构、数值表），列出至少 5 个关键设计决策及其理由
- **蓝小码**：可运行的 Godot 项目，含玩家移动+跳跃+至少 1 种敌人 AI+2 个关卡；代码通过 gdlint
- **绿小画**：至少 1 个玩家角色精灵（含 idle/walk/jump 动画）、1 套 TileSet（含 3+ 地形块）、UI 主菜单+HUD 素材
- **金小验**：至少 10 条测试用例（含边界条件），所有 P0 级 bug 已修复或记录

## 预估耗时

| 角色 | 软超时 | 硬超时 |
|------|--------|--------|
| 红小谋 | 10 min | 20 min |
| 蓝小码 | 15 min | 30 min |
| 绿小画 | 15 min | 30 min |
| 金小验 | 10 min | 20 min |

## 技术栈参考

- 引擎：Godot 4.4
- 语言：GDScript
- 测试：GUT（Godot Unit Testing）
- Linter：gdlint
- 参考：Godot 官方 "Dodge the Creeps" 教程、Clear Code 15h 免费课程

## 来源

> 此模板基于以下实战/调研沉淀：
> - 2026-06-21：Phase 1 验证测试——模拟 "Godot 2D 平台游戏" 任务，调研 indie game dev 团队结构（Toño Game Consultants 行业指南 + Godot 官方文档）
