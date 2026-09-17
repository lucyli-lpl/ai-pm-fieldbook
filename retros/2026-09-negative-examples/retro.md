---
title: 慎用反例与兜底
project: enterprise-agents
period: 2026-06 ~ 2026-09
status: closed
started: 2026-09-17
affected_modules: [5]
tags: [prompt, knowledge-base, fallback]
updates:
  - 2026-09-17: 初稿
---

在提示词、知识库和兜底方案里，"不是什么 / 不要做什么"这类反例和排除性表述要慎用。反例会让 AI 趋向过度谨慎、回避决策；兜底方案若不设触发门槛，会被 AI 当作"安全出口"大量调用，减少自主思考。应优先用正面定义和正向引导来塑造 AI 行为。
