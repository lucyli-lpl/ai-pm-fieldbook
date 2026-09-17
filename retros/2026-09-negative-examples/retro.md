---
title: 慎用反例与兜底
project: enterprise-agents
period: 2026-06 ~ 2026-09
status: ongoing
started: 2026-09-17
affected_modules: [5]
tags: [prompt, knowledge-base, fallback, enterprise]
updates:
  - 2026-09-17: 初稿，来自两个企业 agent 项目的阶段性复盘
---

在提示词、知识库和兜底方案里，"不是什么 / 不要做什么"这类反例和排除性表述要慎用。反例会让 AI 趋向过度谨慎、回避决策；兜底方案若不设触发门槛，会被 AI 当作"安全出口"大量调用，减少自主思考。

判断的变化：之前以为补反例是在"收紧边界"，现在看它更多是在"压缩中间地带"——正常回答的能力被挤掉了。塑造 AI 行为应优先用正面定义和正向引导。
