# WRITING.md — 内容格式约定

本仓库的内容会被 [lucyli-lpl.github.io](https://lucyli-lpl.github.io) 自动读取渲染。**格式就是接口**：字段名写对，站点就能把方法论、修订记录、复盘、坑库连成一张网；写错不会报错，只是站上少一块。

站点构建时会跑一遍格式校验，不合规的地方会让构建失败并给出文件名 + 原因（你会收到 GitHub Actions 的失败邮件）。本地想先自查：在站点仓库里跑 `pnpm sync && pnpm validate`。

---

## 1. 方法论模块 `methodology/NN-slug/`

```
methodology/
├── 01-three-zones/
│   ├── module.md        ← 必须；frontmatter + 一段导语
│   ├── 01-core.md       ← 章节，NN- 前缀决定顺序；frontmatter 只要 title
│   ├── 02-product-view.md
│   └── diagram.svg      ← 可选，有就显示在模块页顶部
```

`module.md` frontmatter：

```yaml
order: 1                       # 必填，整数，与目录前缀一致
title: 能力三区 = 三种产品承诺   # 必填
one_liner: 一句话说清这个模块    # 建议填，站点作为"核心命题"块显示
skills: [ai-product-tasting]   # 可选，关联 skill 的目录名
since_version: "1"             # 可选，字符串
```

章节文件 frontmatter：

```yaml
---
title: 核心内核
---
```

## 2. 修订记录 `methodology/CHANGELOG.yaml`

一个数组，**新版本放最前面**。每条：

```yaml
- version: "2.2"                # 必填，字符串，带引号
  date: 2026-10                 # 可选，YYYY-MM
  summary: 一句话说明这次改了什么  # 必填
  modules: [5]                  # 必填，整数数组，对应模块 order
  trigger: project              # 可选：project | reading | discussion | internal
  trigger_ref: XX 项目复盘        # 可选，触发源的具体名称
  retro: 2026-09-negative-examples   # 可选，复盘目录名；填了站点就把矩阵格子链到复盘
  details: |                    # 可选，多行
    5.3 新增 …
```

`trigger` 不填时站点按 summary 关键词推断（复盘/真实项目 → project，精读 → reading，讨论 → discussion，其余 internal）。**能填就填**，推断不一定对。

## 3. 项目复盘 `retros/YYYY-MM-slug/`

详见 [`retros/README.md`](retros/README.md)。要点：

- 复制 `retros/_template/` 整个目录开始写；`_` 开头的目录站点不渲染
- `retro.md` frontmatter 必填 `title`；`affected_modules` 是整数数组；`status` 只能是 `ongoing` 或 `closed`
- 文件名含 `pitfall` 或 `坑` 的章节会被解析成坑卡片，**每个坑一行**：

```markdown
- **[模块3]** 坑的一句话 → 原因 → 对策
- **[通用]** 坑的一句话 → 原因 → 对策
```

方括号里是模块 order 或 `通用`；三段用 `→`（或 `->`）分隔；原因和对策可省略。这一行写错，坑库里就没有它。

## 4. 复盘反哺方法论的闭环

1. 复盘 `retro.md` 里写 `affected_modules: [5]`（可选 `produced_version: "2.2"`）
2. 方法论真的改了 → CHANGELOG 新条目写 `trigger: project` + `retro: <复盘目录名>`
3. push 后：模块页尾部出现"相关复盘"，成长记录和演化矩阵的格子直接跳到复盘页

## 5. 常见错误

| 写法 | 问题 |
|---|---|
| `version: 2.1`（无引号） | YAML 会当成数字，`1.10` 变 `1.1` |
| `modules: 5` | 要数组 `[5]` |
| `- **[模块 3]**`（有空格） | 可以；`- [模块3]`（没有 `**`）不行 |
| 坑写成两行 | 只认第一行 |
| 章节文件叫 `core.md` | 没有 `NN-` 前缀不会被读取 |
| 在站点仓库的 `.content/` 里改 | 那是临时 clone，`pnpm sync` 会删掉；在本仓库自己的 clone 里改 |
