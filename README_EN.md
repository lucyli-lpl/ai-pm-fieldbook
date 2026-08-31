# AI PM Fieldbook · Capability Boundary Methodology for Enterprise AI Products

> Distilled from frontline practice, written one insight at a time. Not a theory textbook — these are the working methods an enterprise AI PM has stumbled through, crystallized, and iteratively refined in real projects.

**Core thesis in one sentence: The three capability zones are not difficulty tiers — they are three distinct product promises: Core Zone guarantees outcomes, Edge Zone guarantees handling, Out-of-Scope Zone guarantees boundaries.**

---

## Who This Is For

This methodology is grounded in enterprise AI project experience and is designed for **0-to-1 phases** of enterprise AI applications with well-defined rules and process constraints.

**If you're doing personal vibecoding or just validating a quick demo — this is not for you. Don't use it.** It will slow you down.

**But if you're building an enterprise AI application where mistakes have real consequences and someone is accountable for going live, it will help you avoid costly detours.** Especially if you find yourself in this situation: you can't single-handedly build the full stack, and you don't have a tight-knit AI team that can iterate quickly. Instead, you're embedded in a clearly divided team — UI designers, frontend engineers, testers, operations — each in their own lane, with coordination friction at every handoff.

In that environment, the explicit deliverables this methodology demands at each stage (capability boundary master table, acceptance criteria spec, badcase taxonomy…) become the shared language that cuts through friction and aligns the whole team to a single standard. In a frictionless environment, these artifacts are redundant. In a friction-heavy environment, they are the lubricant.

This methodology feels "heavy" because it assumes you can't align with your colleagues over coffee — you need documentation. That's exactly the reality most enterprise AI PMs are living in.

---

## What Problem It Solves

Traditional software has deterministic capabilities (click a button, get a page transition). AI capabilities are fuzzy, bounded, and prone to failure at the edges. So the most important thing to specify precisely in an AI product is not "what features exist," but rather:

- Where do the capability boundaries lie (Core / Edge / Out-of-Scope)?
- What should the product behavior look like in each zone?
- What counts as passing? What counts as a violation?
- When something fails, how do you attribute the cause and drive improvement?

**The core work of this methodology is translating "the model's fuzzy capabilities" into "clear product promises."** It gives you a complete judgment framework — from project kickoff, through testing, launch, and ongoing operations — so that the hardest and most critical part of AI product development has a repeatable process.

---

## Where to Start Reading

No need to read cover to cover. Jump directly to the section that matches where you are right now:

| What you're working on | Read these |
|---|---|
| **Just kicked off, defining product boundaries** | Chapter 1 (Core Framework) + Chapter 6 (AI Product Philosophy) — establish the "three zones = three promises" judgment framework |
| **Stuck classifying a complex edge case** | Chapter 2 (How to classify complex scenarios) + Chapter 3 (General scenario feature library) |
| **Preparing tests / writing acceptance criteria** | 5.1 Testing (four-layer structure, S0/S1/S2 × L1/L2/L3, acceptance spec) |
| **Post-launch, doing data ops / badcase analysis** | 5.2 Data Operations (five-step loop, error analysis, attribution & routing) |
| **Reviewing prompts / designing skills with engineering** | 5.3 Generation QA (prompt ladder, system prompt vs skill layering) |
| **Choosing architecture / setting guardrails** | 5.4 Architecture selection (CoT/workflow/agent) + 5.5 Production guardrails & reliability |

**First-time readers: at minimum, read Chapter 1 and Chapter 6** — they are the foundation. Everything else builds on the "three capability zones" judgment framework.

---

## About the Examples

This methodology uses two **fictional reference products** throughout to ground abstract concepts in concrete scenarios:

- **Helpdesk Assistant**: An internal IT helpdesk AI assistant where employees ask about procedures ("What's the process to reset my password?"), retrieve documents, find the right support engineer, and file tickets. Category: "retrieval Q&A + light operations."
- **Expense Assistant**: An expense workflow AI assistant that helps employees check reimbursement status, fill forms, and submit. Category: "task-based / workflow-driven."

These two products are illustrative vehicles, not references to any real product. The methodology itself grew out of the author's practice and iteration across multiple real enterprise AI agent projects (see the version history at the end of the main document, where every correction from v1 → v2.1 is documented). The examples have been generalized so you can apply them to your own product.

---

## Table of Contents

- [`methodology/00-capability-boundary-methodology_EN.md`](methodology/00-capability-boundary-methodology_EN.md) — Methodology main text (current v2.1)

> Planned: Split the main text into per-chapter files, with a reusable template library (capability boundary master table, acceptance spec, badcase taxonomy) and Claude Skill compositions — enabling "load-on-demand, lightweight usage." Watch for updates.

---

## Status & Evolution

This is a **living document**. It evolved from v1 to v2.1, with each version correcting something discovered in a real project. Many sections are explicitly marked "practice hypothesis, pending validation" — that's intentional honesty. The field of AI product methodology is still young. Rather than claiming completeness, this document marks its own boundaries and evolves with practice.

If you've used this in your own project and found something that doesn't hold up or isn't covered, please open an issue — that's exactly how the next version gets written.

---

## License

[MIT](LICENSE) © D1One-hue
