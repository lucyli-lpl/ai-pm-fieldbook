# 项目复盘 / Retrospectives

方法论从项目里长出来，复盘是它的养分。这个目录记录真实项目的复盘，并和 `methodology/` 双向打通：

- 复盘声明它触碰了哪些模块（`affected_modules`）、推动了哪个版本（`produced_version`）
- 方法论 CHANGELOG 条目反向指回复盘（`retro: <目录名>`）
- 站点 [lucyli-lpl.github.io/retros](https://lucyli-lpl.github.io/retros/) 渲染复盘，并汇总所有踩坑到 [坑库](https://lucyli-lpl.github.io/pitfalls/)

> ⚠️ 本仓库公开。复盘内容需脱敏：不写公司名、客户名、内部数据、可识别的人。

## 目录结构

```
retros/
├── README.md                      ← 本文件
├── _inbox/                        ← 碎片区：随手记，站点不渲染
│   └── 2026-09.md                 ← 按月一个文件
├── _template/                     ← 复盘模板：复制整个目录开始写
└── 2026-09-<project-slug>/        ← 一次复盘一个目录，站点按目录渲染
    ├── retro.md                   ← frontmatter + 一段摘要
    ├── 01-context.md              ← 背景：做什么、约束是什么
    ├── 02-process.md              ← 过程：关键决策与转折
    ├── 03-pitfalls.md             ← 踩坑：固定格式，站点跨项目汇总
    └── 04-lessons.md              ← 沉淀：改了方法论哪里、该改没改的
```

以 `_` 开头的目录站点跳过。章节文件按 `NN-` 前缀排序，数量不限；文件名含 `pitfall` 或 `坑` 的章节会被解析为踩坑卡片。

## 写作流程

1. **平时**：往 `_inbox/YYYY-MM.md` 扔碎片。一行一条，带日期，不用整理。
2. **节点到了**（项目上线 / 月底 / 一次重大返工）：复制 `_template/` 为 `YYYY-MM-<project-slug>/`，基于碎片写。可以让 Claude 先把碎片归拢成初稿。
3. **推送**：`git push` 后站点约 90 秒更新。
4. **持续更新**：复盘不必一次写完。`status: ongoing` 期间每次补充在 `updates:` 加一行；封版时改 `status: closed`。
5. **反哺方法论**：如果这次复盘改了方法论，在 `methodology/CHANGELOG.yaml` 新条目里写 `trigger: project` 和 `retro: <目录名>`，站点的演化矩阵和模块页会自动连到这篇复盘。

## retro.md frontmatter

```yaml
title: 服务台助手项目复盘          # 必填，站点标题
project: servicedesk-agent        # 项目短名，同一项目多次复盘时用于归组
period: 2026-07 ~ 2026-09         # 项目时间段，自由文本
status: ongoing                   # ongoing | closed
started: 2026-09-16               # 复盘开始日期
affected_modules: [1, 3]          # 触碰的方法论模块编号
produced_version: "1.6"           # 推动的方法论版本，没有就删掉这行
tags: [enterprise, rag]           # 自由标签
updates:                          # 每次补充加一行，站点显示更新轨迹
  - 2026-09-16: 初稿
  - 2026-09-30: 补上线两周数据
```

## 踩坑格式（03-pitfalls.md）

每个坑一行，三段用 `→` 分隔，方括号标模块编号（跨模块或不属于任何模块写 `通用`）：

```markdown
- **[模块3]** 把 L2 人工评测当成了上线门槛 → 混淆了测试和运营的分工 → S0 场景走 L1 断言，L2 只做抽检
- **[通用]** 需求方口头确认的边界没落成文字 → 后期返工时无据可依 → 能力边界母表在 kickoff 时就建
```

坑的一句话是必填，原因和对策可以先空着，后面补。
