# AI Application Capability Boundary Methodology v2.1

> Status: v2.1. v2.1 adds 5.3.6 "Skill: Scene-specific on-demand prompt loading" (prompt system layering — system prompt governs global judgment rules, skill governs scene-level operating procedures; skill definition, when to use it, how it differs from MCP/plugin, internal structure and design principles). 5.4 architecture selection adds the "Zeroth Cut" (before changing architecture, ask whether a skill can solve it) + 5.4.5 "Error Cost × Detectability Matrix" (precise autonomy level-setting per action) + 5.4.6 "When capability is insufficient, narrow scope — don't add orchestration." New section 5.5 "Production Guardrails & Reliability" (three-layer guardrails, three reliability mechanisms, human-in-the-loop three-phase path). Prior v2.0 completed 5.3 Generation QA (prompt QA 8-point checklist + detail calibration), 5.4 Architecture Selection (definitions and judgment for CoT/workflow/agent, enterprise autonomy), and added Chapter 6 "AI Product Philosophy." v1.9 added "Structured Clarification vs. Inferential Clarification" distinction in 1.4. v1.8 restructured Ch5 from "three perspectives" to "Testing (pre-launch) + Data Operations (post-launch)" dual-phase (ADLC), clarifying L1/L2/L3 are testing instruments not operations tools. Data Operations became a standalone "data flow" major section (5.2, ten subsections), integrating error analysis + OpenAI/Arize feedback system review + real enterprise AI agent project practice: five-step loop, start narrow, three-source correlation (canonical event/trace_id), thumbs-down design, labeled version control, behavioral signal combination, signal value tiering, attribution routing, privacy governance.

> **Core thesis in one sentence: The three capability zones are not difficulty tiers — they are three distinct product promises: Core Zone guarantees outcomes, Edge Zone guarantees handling, Out-of-Scope Zone guarantees boundaries.**

> 📌 About the examples: This document uses two **fictional reference products** throughout to ground abstract concepts in concrete scenarios:
> - **Helpdesk Assistant**: An internal IT helpdesk AI assistant where employees query procedures ("What's the process to reset my domain account password?"), retrieve documents, find the right support engineer, and file tickets. Category: "retrieval Q&A + light operations."
> - **Expense Assistant**: An expense workflow AI assistant that helps employees check reimbursement status, fill forms, and submit. Category: "task-based / workflow-driven."
>
> These two products are illustrative vehicles, not references to any real product. The methodology itself grew out of the author's practice and iteration across multiple real enterprise AI agent projects (see the version history at the end); examples have been generalized for portability to your own product.

---

## I. Core Framework

### 1.1 Three Dimensions

- **Scenario = what it looks like**: What need did the user express? What objective conditions are currently present?
- **Action = what should happen**: The correct response the product should take given that scenario.
- **Attribution = why it didn't work**: After actual behavior falls short, which component needs to be fixed?

> Mnemonic: **Scenario looks at input, action looks at expectation, attribution looks at the fix.**
> Zone classification uses scenario; test acceptance uses expected action and outcome; operations analysis uses attribution.

Three high-frequency cross-contamination errors:

1. Inferring a capability zone backward from whether the model answered well this time.
2. Treating correct actions like probing questions or graceful fallbacks as the cause of errors.
3. Labeling a user's cross-mode expression as a model error — a user switching intent is a scenario; the system failing to correctly recognize and route it is the error.

Phrases like "insufficient knowledge" or "insufficient permissions" require distinguishing two identities:

- The product has pre-confirmed that formal knowledge isn't covered, rules conflict, or system conditions are insufficient: this is an objective supply constraint that can be used to classify a scenario as Edge Zone.
- After the model answered poorly, operations personnel speculate "it might be a knowledge base or permissions issue": this is an unvalidated attribution hypothesis and cannot be used retroactively to define the scenario.

### 1.2 Two Coordinate Systems

**Coordinate System 1: Relationship between user needs and product capability**

- Core Zone
- Edge Zone
- Out-of-Scope Zone

**Coordinate System 2: Internal processing chain of the application**

- Routing and intent recognition
- Each processing mode
- Knowledge, data, or system calls
- Response and result generation

The three capability zones do not include routing. A user switching from one goal to another does not thereby enter the Edge Zone or Out-of-Scope Zone; the routing layer should determine on each turn whether this is continuation, a correction of understanding, or a new goal.

### 1.3 Three Zones = Three Product Promises

| | Core Zone | Edge Zone | Out-of-Scope Zone |
|---|---|---|---|
| Product promise | Stably and completely fulfills the request | Does not guarantee complete fulfillment, but guarantees correct handling | Does not fulfill the request, but guarantees correct recognition and reception |
| Classification basis | A clear, designed, stable completion path exists, and the product **promises** stable, complete delivery here (promise first, validation second) | Relevant to the product's domain, but currently limited by information, knowledge, combination rules, data, or system conditions — no stable completion path yet exists | Outside the product's overall capability, or the product explicitly prohibits fulfillment |
| Primary actions | Complete directly; complete after intent unification; complete after necessary clarification | Complete the completable portion; decompose and handle; explain limitations; degrade gracefully to match the actual gap | Stop the out-of-scope portion; explain the boundary; offer permissible help or a genuine handoff path |
| Testing focus | Result is correct, complete, actionable, and meets target stability | Observe both complete-handling rate and safe-fallback stability simultaneously | Expected action matches actual action, and boundary statement, facts, and next steps are correct |
| Failure floor | Must not incorrectly degrade or fall back prematurely | Must not present uncertain content as certain conclusions; must not abandon all completable content due to partial uncertainty | Must not answer out of scope; must not fabricate capabilities, causes, or handoff results |

Zone classification is based on what the product **promises** to deliver, not on the model's performance in a particular instance or its self-expressed "confidence."

**Decoupling zone classification from acceptance (important)**: Core Zone is the product's **normative promise** — the product decides "I must deliver here stably and completely," so its identity comes from "does a clear completion path exist + has the product committed to it," **not from "did it pass acceptance testing recently."**
- Zone classification asks: is the path clear? Has the product committed? (normative)
- Acceptance asks: did the live test meet the committed metric? (empirical)
- Therefore: a Core Zone scenario that currently fails acceptance means it **hasn't fulfilled the promise and needs to be fixed** — not demoted to Edge Zone. Conversely, an Edge Zone scenario that occasionally produces a complete correct answer does not thereby become Core Zone — it still lacks a stable, committable completion path. **Performance fluctuation changes "did it pass," not "which zone it belongs to."**

Accuracy rate, recall rate, and repeated-run consistency targets for Core Zone are set by the product in its testing and acceptance plan — not fabricated as uniform numbers in the absence of baseline data.

### 1.4 The Different Purposes of Clarifying Questions Across the Three Zones

- **Core Zone clarification**: A confirmed, stably convergent clarification path exists; once decisive information is obtained, the request must be completed.
- **Edge Zone clarification**: Confirm which parts can still be completed, or determine whether current constraints can be lifted.
- **Out-of-Scope Zone clarification**: Only used to confirm whether the request truly exceeds scope, or whether it contains portions that can still be served.

It is wrong to broadly mandate "ask a clarifying question whenever information is incomplete." Clarifying questions must have a clear purpose and genuinely change the downstream outcome.

**Two modes of clarification (Structured Clarification vs. Inferential Clarification) — choose based on where the uncertainty comes from:**

| | Structured Clarification | Inferential Clarification |
|---|---|---|
| Uncertainty comes from | A **specific piece of information (field) is missing** | **Semantic / scenario ambiguity** — multiple reasonable interpretations exist |
| How to ask | Ask for the known gap ("Which department are you in?") | **Expose the AI's reasoning branches** and let the user self-identify |
| Depends on | Pre-defined fields | AI's in-context reasoning — **no pre-defined fields needed** |
| Example | Missing "department" makes ticket template matching non-unique → ask for department | "My email has a problem" → AI surfaces: "Is it that you can't receive external emails (email gateway investigation) / can't log in (account/password flow) / or it says your mailbox is full (quota expansion)?" |

- **The two modes cannot substitute for each other**: When uncertainty is "semantic ambiguity" (cannot be enumerated into fields), structured clarification cannot work — inferential clarification is required.
- **Implementation key for inferential clarification**: Have the matching step output "candidates + their respective preconditions + distinguishing factors" (the AI is already doing this reasoning — just surface it) → when multiple candidates exist, don't return all results; organize the reasoning into a counter-question → re-match with context after the user responds. No pre-defined fields needed — this is exactly where LLMs excel.
- **Common misconception**: Engineering teams tend to interpret "clarification" as "fill in a field," and conclude "clarification is impossible" when facing semantic ambiguity. Correction: clarification need not depend on fields — the AI can generate clarification branches in context. (Validated in enterprise agent projects.)
- Additional value of inferential clarification: it transforms the AI's uncertainty from "guessing in the background" to "interactive and visible," and incidentally educates the user (who may not know which step their problem is in), resulting in a better experience.

### 1.5 Acceptance Focus by Zone

**Core Zone: guarantee outcomes**

- Observe accuracy rate or task completion rate
- Observe intent recognition recall rate and sub-goal completeness
- Observe whether necessary clarification questions converge effectively
- Run the same inputs repeatedly and observe consistency of key conclusions, clarification questions, and delivered results
- Observe whether the system incorrectly degrades or falls back prematurely

**Edge Zone: guarantee handling**

- Upper-bound capability: what percentage can be handled completely?
- Floor capability: when complete handling is impossible, what percentage can complete the definite portion and degrade correctly?
- Risk indicators: out-of-scope hard answers, speculative conclusions, wholesale abandonment, incorrect fallbacks

Edge Zone runs may fluctuate between "complete fulfillment" and "correct degradation," but must not fluctuate between "correct degradation" and "confidently wrong answers."

**Out-of-Scope Zone: guarantee boundaries**

Out-of-Scope Zone does not evaluate whether the substantive answer the user sought is correct — the product should never have provided that answer in the first place. But it still checks:

- Whether expected action matches actual action
- Whether the boundary statement is accurate
- Whether causes, capabilities, or handoff results have been fabricated
- Whether the next step offered is genuinely available
- Whether permissible help within scope has been preserved

Out-of-Scope acceptance = **action match rate + boundary statement correctness** — not "passes as long as it refuses."

---

## II. How to Classify Complex Scenarios

### 2.1 Vague Expression or Insufficient Information

- A finite, clear set of decisive information exists, the product knows what to ask, and stable completion is achievable once the user provides it: **Core Zone**.
- The user doesn't know or won't provide it, or the product lacks a stably convergent clarification path: **Edge Zone**.
- Even with additional information, existing knowledge or systems cannot support a determination: **Edge Zone**.
- After clarification, the request as a whole is confirmed to exceed capability or is prohibited: **Out-of-Scope Zone**.

If a clear completion path exists but the model failed to ask for the decisive fact, asked repeatedly without converging, or fell back prematurely — this is a Core Zone badcase. The scenario should not be reclassified as Edge Zone.

### 2.2 Mixed Multi-Intent

Multi-intent is first a scenario the routing layer must recognize and decompose — it does not automatically belong to Edge Zone.

- Each sub-goal has a clear completion path: fulfill them separately; this is Core capability.
- Different sub-goals may each fall into Core, Edge, or Out-of-Scope.
- Whether the overall result meets the standard depends on whether all sub-goals were identified (nothing missed), and whether each sub-goal received the matching action.

### 2.3 Complex or Cross-Domain Knowledge

"Complex" or "rare" does not directly determine the zone. The key question is whether a confirmed, stably executable completion path exists.

- Multiple sub-questions can each be answered independently from existing knowledge: decompose and complete separately; may be Core Zone.
- Requires judging the combination, sequence, priority, or substitution relationships between different pieces of knowledge, and formal knowledge provides no such rules: Edge Zone.
- User further requests that the product make professional judgments, approvals, or take actions the product explicitly prohibits: Out-of-Scope Zone.

### 2.4 Routing: Not an Independent Scenario Type, but a Complex Condition Attached to Real Needs

**Routing does not constitute its own scenario category.** This methodology defines "scenario" starting from **user needs** (what did the user do, what need did they express, what objective conditions are present). Routing is an **internal system action** with no independent user need as its starting point. AI products have no such thing as a "pure routing scenario" — routing is always attached to a real need (the user came to ask about a procedure or get a template, not to "trigger a routing event"). Treating routing as an independent scenario creates duplication with and conflicts against real-need scenarios.

Routing therefore exists in three roles, none of which enters the capability three zones:

- **① As action**: correctly identifying intent, switching, decomposing multi-intent, distinguishing "continue from context" vs. "new goal" — these are the correct system responses.
- **② As attribution**: root causes of failures may land in the intent recognition layer (went to the wrong door → inspect the router, rather than defaulting attribution to the generation layer just because the final answer was poor).
- **③ As a trace observation dimension**: when tracing conversations, routing quality can be separately observed.

**Routing-related situations that require dedicated testing** — mid-conversation goal switching, multi-intent in a single message, user correcting the system's understanding, surface similarity that is actually continuation — **are not independent scenarios, but variants of existing real-need scenarios under "multi-turn / multi-intent" complexity conditions (tested as sub-variations)**:
- Multi-turn switching = real scenarios like "asking about a procedure" under a "multi-turn condition" variant.
- Multi-intent = multiple existing real scenarios stacked; decompose and classify each separately.

Their test coverage belongs to the corresponding real scenarios (with added complexity conditions); routing quality is what to watch in attribution. This avoids both gaps and duplication.

---

## III. General Scenario Feature Library

### 3.1 Core Zone

| Feature | Description | Example (Helpdesk Assistant) |
|---|---|---|
| Clear intent, sufficient information | Stable completion path already exists | "What's the process to reset my domain account password?" |
| Colloquial but stably unifiable | Can be recognized as an existing business goal | "I want to install that design app on my new computer" → unified as: find the software installation request process |
| Missing a finite set of decisive facts | The product knows what to ask; completion is achievable once the user provides it | User wants to find the right support engineer but hasn't specified which business system; can be matched by rule once provided |
| Multiple independently completable goals | Routing can decompose; each has a completion path | Simultaneously querying an application procedure and its corresponding template form, where each can be independently matched from formal knowledge |

### 3.2 Edge Zone

| Feature | Description | Example (Helpdesk Assistant) |
|---|---|---|
| Information insufficient and uncollectable | User cannot provide necessary conditions, or product lacks a stable clarification path | User cannot specify the business system or department needed to match the right engineer |
| Persistently vague expression | Still cannot be reliably unified after minimal necessary clarification | User keeps saying only "that system won't let me log in" with no identifiable specific system throughout the conversation |
| Partial knowledge coverage | Can confirm some results, but no basis for specific edge details | Can match the application procedure, but knowledge doesn't cover the special approval exception the user asks about |
| Missing cross-domain judgment rules | Individual knowledge items exist, but combination, sequence, or substitution relationships lack basis | The same matter involves both a permissions request and an equipment requisition process; the user asks which comes first, but the knowledge base provides no sequencing rule |
| Current supply insufficient | Knowledge conflicts, validity unclear, permissions unknown, entry point failed, or personnel data invalid | Two processes have overlapping applicability and cannot be disambiguated, or ITSM cannot return a valid ticket entry point |

### 3.3 Out-of-Scope Zone

| Feature | Description | Example (Helpdesk Assistant) |
|---|---|---|
| Exceeds product capability | Requires an IT support engineer's judgment of the specific situation | Requesting a judgment on whether a high-risk permission request should be approved, or whether a failing server should be restarted |
| Explicitly prohibited items | Overstepping authority, security, privacy, or actions not yet opened | Requesting to bypass approval to grant permissions directly, view another person's account information, or auto-notify an engineer |
| Completely unrelated requests | Outside the product's service scope entirely | Requesting completion of tasks unrelated to the IT helpdesk |

---

## IV. Application Type Variants

### 4.1 Pure Q&A Type

- Output is primarily information.
- Core Zone mainly tests answer correctness, completeness, and stability.
- Errors in specialized domains may affect user decisions and cannot default to "just ask again."

### 4.2 Retrieval Q&A + Light Operations Type (e.g., Helpdesk Assistant)

The Helpdesk Assistant is not a pure Q&A type. A single AI response may simultaneously contain natural language, a procedure card, a template card, an engineer card, and an executable entry point — and may depend on knowledge retrieval, current ITSM results, and multi-turn context.

- Core Zone tests not only answers, but also resource matching, card fields, permissions, version, entry points, and operational results for consistency.
- Capability boundaries are defined at the granularity of "one user goal and its processing phase," not limited to one block of text.
- A single AI response may include multiple messages or cards that together constitute one delivery.
- Multi-intent is recognized and decomposed by the routing layer; each sub-goal is classified into its capability zone separately.
- Actions like opening, downloading, copying, and filing a ticket must be validated against real system results — showing an entry point must not be described as having already initiated the process or already contacted an engineer.

### 4.3 Task-Based or Workflow-Driven Applications (e.g., Expense Assistant)

Task-based applications require, in addition to informational correctness:

1. **Execution result**: saying it correctly ≠ doing it correctly; verify the actual action outcome.
2. **Error cost**: irreversible or high-risk actions should be more conservative and confirmed before execution.
3. **State and process nodes**: capability boundaries must be evaluated separately for each process node.
4. **Stop and recovery**: clear paths must exist for failure, timeout, duplication, reversal, and human takeover.

The action list should be tailored to the application's actual capabilities — don't force every cell to be filled to satisfy the framework. The essence of handoff is getting the user to a place that can genuinely solve their problem — not necessarily a human, but only usable when the product actually has that path.

### 4.4 Other Types

- Generative type: to be added after validation in a real project.
- Multi-Agent orchestration type: to be added after validation in a real project.

---

## V. Operationalizing Capability Boundaries: Testing + Data Operations

### 5.0 Overview: Two Phases, One Loop (ADLC)

The same capability boundary (master table) drives a continuous improvement loop that spans the release. The industry calls this ADLC (Agent Development Lifecycle): **pre-launch testing → release → post-launch data operations → feedback into the next version**.

```
        ┌─────── Master Table (single source of truth, shared axis) ──────┐
        ↓                                                                   ↓
  Pre-launch: Testing (5.1)                        Post-launch: Data Ops (5.2)
  Determine "fit to release"                        Let data drive "continuous improvement"
  ─ Data: pre-set / synthetic inputs                ─ Data: real user traffic
  ─ Purpose: gate                                   ─ Purpose: discovery & improvement
  ─ Methods: assertions(L1)/human·LLM(L2)/A-B(L3)  ─ Methods: 3-source → label → error analysis → route
        └─────────────→ Release ──────────→ Monitor ─────────────────────┘
                                                    (feedback into next version)
```

**The dividing line between testing and data operations (three overlapping lines):**

| | Testing (5.1) | Data Operations (5.2) |
|---|---|---|
| Timing | Pre-launch | Post-launch |
| Data | Pre-set / synthetic inputs | Real user traffic |
| Purpose | Gate (fit to release?) | Improvement (continuous discovery and improvement) |
| Core capability | **Quality determination** (evaluation methods + standards) | **Data flow & attribution** (data sources, correlation, metrics, attribution) |

**Key distinction: assertions (L1), human/LLM evaluation (L2), and A/B (L3) are testing instruments (tools for quality determination) — not data operations tools.** The core of data operations is not "which evaluation method to use" but "how does the data come in, how is it correlated, how does it become attributable insight." The two intersect only at the tail end: A/B (L3) is used to verify whether an optimization from the operations loop actually improved things.

> Context note: AI product quality is not determined "at launch" — it is determined by "the data operations loop after launch." Traditional software peaks at release (thereafter only bug fixes). AI products begin at release and improve through data. Data operations plans are therefore as important as testing plans, and are core assets alongside the PRD.
>
> Sources for the data operations section: Hamel/Shreya error analysis (analytical method) + OpenAI/Arize feedback system review (engineering architecture) + real project practice. Because "AI product data operations" is a field without mature public methodology, this section relies heavily on practice hypotheses and will be continuously validated through real-world project operations.

### 5.1 Testing (Pre-Launch: Determining Fit to Release)

> Testing = pre-launch, using pre-set/synthetic inputs to verify "fit to release." Methods are assertions (L1) / human·LLM (L2) / A-B (L3). The four-layer structure below clarifies "who determines, by what standard, who decides."

> **Overall skeleton of this section** (read this first; 5.1.1–5.1.6 all slot into this structure).
> Acceptance specifications are not a single framework but a **four-layer structure** — who owns each layer, what it answers, with clean boundaries:

```
Layer 4  Decision (PM)        Block release? Pass? Fix which first?
                               ↑ from "normative line × empirical value" collision
Layer 3  Normative (PM defines) Severity S0/S1/S2 → pass rate threshold per level
         Empirical (computed)   Accuracy / recall / pass rate (aggregated by scenario)
                               ↑ aggregated from
Layer 2  Determination (QA/code) Per item: L1 assertion pass/fail · L2 human good/bad (binary)
                               ↑ executed from
Layer 1  Expectation (PM defines) What behavior each scenario should exhibit
         From capability boundary methodology (Core Zone correct / Edge Zone degrade / Out-of-Scope intercept)
```

**Three key points for reading this diagram:**

1. **Separate the determination layer from the decision layer.** L1/L2 at Layer 2 only make binary determinations (pass/fail, good/bad) — **severity is not needed at determination time**. Severity, metrics, and decisions all live at Layers 3–4. If something feels "subjective" or "seemingly redundant," it's usually because it was misplaced at the determination layer — put it back at the decision layer and it fits.

2. **Each acceptance criterion is a "three-axis intersection"** — one scenario must simultaneously specify three things:
   - **Expected behavior** (determined by the capability three zones) — what should Core/Edge/Out-of-Scope each do?
   - **Severity** (determines pass rate) — S0/S1/S2
   - **Test method** (determined by determinability) — L1 assertion / L2 human
   > The same scenario may be **split into multiple rows** by severity and determinability for separate acceptance. Example: "multi-candidate clarification" → "whether clarification is triggered" is S1, use L1 assertion; "clarification question quality" is S2, use L2 human — split into two.

3. **Normative × empirical enables decision**: aggregated metrics (accuracy/pass rate) are "empirically measured (how good it currently is)"; severity is "normatively defined (how strict it should be)." The same measured 98%, paired with S0 (requires 100%) is an incident; paired with S2 (requires 85%) is over-achievement — **both are needed to make a decision**.

**Example acceptance spec table** (Helpdesk Assistant "query a procedure"):

| Scenario (from capability boundary) | Expected behavior | Severity | Pass rate | Test method |
|---|---|---|---|---|
| Typical query hits procedure | Hits correct flowId + returns C5 card | S1 | ≥95% | L1 assert flowId correct |
| Renders link without permission | No links in output when permission denied | **S0** | **100% gate** | L1 assert link empty |
| Multi-candidate, no clarification — guesses | Should trigger C3 clarification | S1 | Trigger 100% | L1 assert action==clarify |
| Clarification question quality | Asks for decisive info, not verbose | S2 | ≥85% | L2 human rating |
| Out-of-scope high-risk approval request not intercepted | Should go to C10 fallback, not hard-answer | **S0** | **100% gate** | L1(action)+L2(phrasing) |

> Naming note: **severity uses S0/S1/S2 (Severity), evaluation method uses L1/L2/L3 (Level)** — these are two orthogonal axes (one measures consequence severity, one measures determination method). Using "L" for both caused confusion early on; the rename separates them. "Excellent" (originally L3 bonus items) does not enter acceptance (acceptance is a pass/fail threshold judgment) and moves to monitoring as an observation item.

Below is the four-layer structure expanded: Layer 2 in 5.1.1–5.1.3 (evaluation methods and determination), Layers 3–4 in 5.1.4 (pass rates) and 5.1.6 (severity and decision principles), organizational ownership in 5.1.5. Key observation points by zone:

- Core Zone: result accurate, complete, stable
- Edge Zone: complete-handling rate + correct-degradation rate + out-of-scope hard-answer rate
- Out-of-Scope Zone: action match rate + boundary statement correctness
- Multi-turn / multi-intent: not listed as a separate dimension, but covered as **variants of existing real scenarios under complex conditions** (multi-turn switching, multi-intent decomposition); routing/intent recognition layer is what to watch during attribution
- Task or operation: additionally test real execution results and success confirmation

#### 5.1.1 Evaluation in Three Layers (Different determination methods → Different types of problems each can test)

Evaluation is divided into three layers by "who determines, and how costly" — the three are not incremental in capability and cannot substitute for each other; each tests its own domain and the combination is complete:

| | Determination subject | What it can test | Cost / scale | Usage cadence |
|---|---|---|---|---|
| L1 Assertion testing | Code assertion | Only **objectively determinable attributes** (format, count, unauthorized access, fields, whether an enum is hit, required actions) | Cheap, batchable | Every code/prompt change |
| L2 Human + model evaluation | Human / LLM | **Semantic quality requiring judgment** (professionalism, honest degradation, action appropriateness, tone) | Expensive, volume-limited, requires careful design | Periodically |
| L3 A/B testing | Real user behavior | **True causal impact** (did users actually benefit) | Most expensive, requires production data | After major product changes |

The layering principle: use cheap, fast checks to catch most problems, leaving only what genuinely requires judgment for higher layers. What can be asserted goes to L1; what requires judgment goes to L2; what requires confirming real benefit goes to L3.

**On L3's role** (avoiding misunderstanding): L1/L2 test "did the AI answer correctly/well"; L3 tests "did the AI actually drive the desired user behavior and business outcome" — a leap from "output quality" to "real impact." Three judgments:
- L3 is a **general product/data skill** (traffic splitting, control groups, statistical significance) — not an AI-specific challenge; no need to reinvent the wheel within AI evaluation.
- L3 is naturally **lagged**: the precondition is that L1/L2 have brought quality to a sufficient level (otherwise A/B results are polluted by bugs), and it requires real traffic and sufficient sample size — unusable during cold start.
- **But L3's precondition — instrumentation that can measure real outcomes — must be in place at launch**, otherwise there is no data to compare against later. See 5.2 on behavioral/outcome signal instrumentation.

#### 5.1.2 Assertions (The atomic unit of L1 assertion testing)

- An assertion = a check on **a specific attribute of the output that code can automatically determine true or false**; passes if true, reports an error if not.
- Assertions test "objectively necessary conditions (boundaries of acceptable behavior)" — **not** "text equals some reference answer." Natural language paraphrases will trigger false failures; asserting text equality is almost never right.
- Assertions primarily operate on **structured fields** (e.g., flowId, templateId, agentId, downloadState), not on the model's raw natural language output.

**PM's high-value action: extract the "bone" that can be objectively determined from each product requirement and hand it to L1; leave the "meat" that requires semantic interpretation to L2.**
Example: "When permission is denied, politely inform the user but don't provide a link" — "don't provide a link" is assertable (link field empty / output contains no URL), hard-gated by L1; "politely" requires judgment, handled by L2.

#### 5.1.3 L1 Assertion Testing: Three Implementation Steps

**① Decompose by feature → scenario; assign an assertion to each scenario**
- Break capability into features; break each feature into scenarios by "result shape"; configure assertions per scenario.
- Assertions fall into three types, and must be designed in pairs/groups — positive samples alone are insufficient:
  - **Positive samples (recall)**: typical query → expects the correct resource to be hit (e.g., flowId == correct procedure). Tests "are things that should be hit, actually being hit."
  - **Negative samples (precision)**: assert "things that must not happen, do not happen." Two forms — ① same question, assert that other procedures are not mistakenly triggered (same question, different angle as positive sample); ② purely out-of-scope questions, assert flowId/templateId/agentId are all empty (independent questions). Tests "are things that should not be hit, being mistakenly hit."
  - **Boundary samples (split in two)**: vague/multi-candidate query → L1 only asserts "whether the correct action was triggered" (action == clarify / fallback — determinable); the **quality** of that action (was the question good? did it ask for decisive information?) goes to L2. L1 holds the floor ("didn't guess randomly"), L2 refines quality ("asked well").
- Also set **universal assertions** (not tied to a feature; must hold globally) — corresponding to the methodology's **S0 red lines**, such as "output must not contain internal UUIDs" and "unauthorized response must not contain links." S0 assertions become **release gates** (absolute veto), do not participate in average pass rates, and one failure blocks launch.

**② Use LLMs to batch-synthesize test cases**
- No need to wait for production data. Synthesize from reasonable guesses about usage patterns; refine with real usage patterns once a small batch of users is live.
- **Deliberately make things hard**: if all tests pass, the tests may be too easy. The signal of a good test suite is that the model fails some — those failures are exactly the problems to optimize.
- **Paired / closed-loop design**: e.g., "create a contact + query back to verify exactly 1 record exists" — use two steps to mutually validate, converting subjective judgment into an objective count check.
- The Helpdesk Assistant's "clarification" examples (password reset vs. account unlock, VPN application vs. VPN fault report, software installation permission vs. SaaS account provisioning, etc.) are natural "easily confused negative sample" seeds — use them to have an LLM batch-generate confusing queries for each pair.

**③ Run regularly + track trends**
- Run with the lowest-friction method in your tech stack (e.g., CI trigger on every change).
- **Don't just look at single-run pass rate — look at the trend curve**: "Is this week better than last week?" Archive test results together with the **prompt version** (outside CI for long-term analysis), so when quality drops you can trace it to which change caused it. This is what "treat prompt like code — versioned and regression-tested" means in practice.

#### 5.1.4 How to Set Pass Rates (PM's product decision)

- **Perspective shift**: a pass rate is not "how good I want it to be" — it's "how bad I can tolerate" — you're setting the acceptable failure rate.
- **Set by failure cost, not question difficulty**: first tier your tests by **severity S0–S2**, then assign pass rates per tier:
  - S0 hard defects (unauthorized access, fabrication, out-of-scope judgment) → **100% zero-tolerance, made into a release gate** — not included in averages, one failure immediately blocks launch.
  - S1 functional failures (wrong routing, wrong resource recommended) → High (e.g., 95%+), especially for Core Zone.
  - S2 experience defects (should have asked a clarifying question but didn't, verbose output) → Medium (e.g., 80–90%), iterable.
  - "Excellent" is a bonus observation item, not a hard acceptance threshold (moved to monitoring).
- **Cold start with no baseline**: first define "direction + relative relationships" (who must be higher than whom), run a round to get real baselines, then set targets as "improvement over baseline" — don't invent ideal values you can't reach for a long time.
- **Consequences check**: every threshold must be able to answer "what will I do if the metric falls below this line?" (block release / trigger iteration / observe only). A threshold that can't answer this is a fake metric.
- Pass rate is a statistic for "a batch of questions," so **the representativeness of the scenario set** directly determines whether it's meaningful (connects to Day 3 real-distribution sampling).

#### 5.1.5 Organizational Implementation: PM's Responsibility Boundary in Evaluation

Hamel's method presupposes "viewing data / building tools / setting requirements" is all one person's job. In an organization with PM/QA/Dev role divisions, it must be translated into collaboration. Core principle: **PM defines the "target state"; QA defines "how to verify the target state is reached."**

**Responsibility boundary:**

| Dimension | Who defines it |
|---|---|
| What scenarios to test (which to cover, especially S0, easily confused negative samples, key boundaries) | **PM** |
| Expected behavior per scenario (what action, what result) | **PM** |
| How good is "passing" (pass standard, pass rate, S0 zero-tolerance) | **PM** |
| Which metrics to watch and their direction/priority | **PM** |
| Which specific questions to trigger a scenario (exact question design, quantity, generation method) | QA |
| How to determine whether a question passed (assertion code, rubric implementation) | QA / Dev |
| How to run, how often, with what tools | QA / Dev |

**What PM should deliver, at what granularity**: "scenario category + expected behavior per category + pass standard (including L1/L2 assignment and pass rate/zero-tolerance) + metrics to watch and direction" — stop there. **Define scenario categories, not every individual question**; specific question design, assertion implementation, rubric, and run orchestration go to QA.
> The skeleton of this test objectives document ≈ PRD state transition table (one row per product function) + capability boundary tiers + pass rates. Take each row of the state transition table and add two things: ① L1 or L2? ② Pass standard/pass rate.

**How precisely to specify metrics (including two pitfalls):**
- **Classification tasks** (intent routing, resource matching) → use **precision + recall** (broken down by intent). When both false negatives and false positives are costly, use F1. PM defines "which to watch + which has priority" (e.g., "mis-routing to the end-to-end flow would trigger an erroneous submission, so precision takes priority"). **Numeric values must be calibrated by running against a baseline — don't invent them upfront.**
- **Pitfall 1: metric misuse** — generation quality, clarification quality, degradation honesty **have no single right or wrong answer and cannot use accuracy/F1**; they should use L2 human evaluation pass rate / human-AI agreement rate. Forcing classification metrics onto subjective quality is an error.
- **Pitfall 2: class imbalance** — in real traffic, some intent may dominate while badcases are rare. **Don't look at a single total accuracy** (guessing the majority class can inflate it); **break down stats by intent / scenario category**.

**Data viewing ownership**: PM and QA look at the same data, but for different purposes — QA for **execution** (did it pass, is coverage sufficient); PM for **definition** (what counts as good, which failures matter most, where the boundary is). What PM should advocate for is **access to raw traces** (PMs should be looking at user data; this isn't overstepping QA's execution authority) — then produce the scoring rubric, S0 list, badcase taxonomy and attribution guidelines for QA to use during execution.

**On tooling**: the data viewing / labeling tool is not a side project the PM builds alone — it's an **internal tool requirements doc written by the PM** (whose users are the evaluation team): aggregate conversation + knowledge hit + permission result + feedback onto one screen, filterable by intent, supporting binary labeling. During cold start, use a **spreadsheet as a stopgap** to get the process running first (Hamel ran LLM judge alignment with Excel early on). Then use the proved value to earn a proper engineering slot — prove value first, then earn resources.

#### 5.1.6 Decision Layer Principles: Defining Severity and Pass Rates (PM Cannot Delegate)

The two-piece decision layer: **severity is the cause** (tier scenarios by consequence severity), **pass rate is the effect** (assign a threshold per tier). Both ultimately reflect the same thing — product values: what must never be broken (safety), what should be achieved (accuracy), what is best to do well (experience).

**Principles for setting severity:**

- **Anchor**: severity measures "how deeply the consequence harms users/business and whether it is reversible" — not the size of the error, and certainly not how inconvenient it is for the team to fix. An apparently small error (silently gave an unauthorized link) may be S0; an apparently large error (verbose answer) may be S2.
- **Three tests — any one pointing to severity escalates it (take the worst, not the average)**:
  1. **Reversibility**: can it be undone if wrong? Irreversible (already triggered a wrong action, already leaked, already gave an error in professional judgment that was taken as fact) → escalate toward S0. Errors that trigger real system actions (filing a ticket, rendering a clickable link) are less reversible — escalate.
  2. **Externality**: does the harm stay within the conversation, or spill into the real world (compliance risk, causes the user to take a wrong business/legal action, damages trust)? Spillover → S0.
  3. **Which promise is violated**: Safety > Accuracy > Experience. Violating boundary/safety (unauthorized access, out-of-scope, fabrication) → S0; violating result/accuracy (Core Zone scenario should have been correct but wasn't) → S1; violating handling/experience (inelegant degradation, verbose clarification) → S2.
- **S0 should be few, hard, and exhaustively enumerable**: S0 is the release gate; its authority comes from scarcity. A closed list that can be enumerated item by item (Helpdesk Assistant: approximately — unauthorized link rendering, fabricated engineer/procedure, out-of-scope substituting for user in high-risk approval judgment, leaking another person's account information…). Gut-check: "If this fails and I still insist on launching, can I bear the consequences?" Can't → S0. Too many red lines and the team will route around or silently downgrade them, making the red lines ineffective.
- **Write the rationale, not just the label**: don't write "unauthorized access = S0"; write "unauthorized access = S0 because irreversible + external spillover + violates safety promise." Rationale makes classification stable, alignable (QA/engineering/leadership see the logic not just the call), and reviewable when scope changes.
- **When uncertain, default to the middle tier**: not sure if it's S0 → put at S1 and observe; don't dilute the red lines. Not sure if it can be ignored → put at S1/S2 and record; don't immediately call it "not our problem." Calibrate with real consequences post-launch (if an S2 triggers many complaints, escalate).

**Principles for setting pass rates** (effect, continuing from 5.1.4):

- Perspective: not "how good I want it" but "how bad I can tolerate" — you're defining the acceptable failure rate.
- Assign thresholds by severity: S0 → 100% zero-tolerance gate (not in averages, one failure blocks); S1 → high (e.g., 95%+); S2 → medium (80–90%).
- Cold start: first define direction and relative relationships (who must be higher than whom); calibrate absolute values after getting a baseline. Don't invent long-term ideal values you can't reach.
- Consequences check: every threshold must be able to answer "what will I do if it falls below this line?" (block / iterate / observe); thresholds that can't answer this are fake metrics.

### 5.2 Data Operations (Post-Launch: Data-Driven Continuous Improvement)

Data operations is a discipline about **"data flow and attribution."** Its core is not "which evaluation method to use" but: how does data come in → how is it correlated → how does it become attributable insight → how does it drive improvement. The methodological core: **personally review large volumes of real data, remove all friction** (scores lie; raw conversations don't).

#### 5.2.1 Operations Framework: Five-Step Loop (Starting Point First)

```
[Cold start first round] error analysis discovers real high-frequency failures ──┐(Starting point: look at data first, don't define metrics first)
    ↓                                                                              │
① What metrics to monitor (each metric corresponds to one type of real failure)   │
    ↓                                                                              │
② How data comes in (thumbs-down + instrumentation + backend labeling, 3 sources) │
    ↓                                                                              │
③ How to analyze (label by master table → count frequencies → error analysis to find patterns)
    ↓                                                                              │
④ How to optimize (attribution determines who fixes; high-freq edge → promote to core)
    ↓                                                                              │
⑤ How to confirm improvement (did the corresponding metric drop + track trends) ───┘(loop)
```

**Critical sequence correction: the starting point is not "define metrics" — first run a round of error analysis.** Starting from "define metrics" tends to slide toward the useless generic metrics Hamel warns against ("coherence 3.72 → 4.2 — you have no idea if the system got better"). **Metrics must grow from real failures**: don't arbitrarily decide "we'll monitor clarification rate" — error analysis finds that "should have asked a clarifying question but didn't" is a high-frequency failure; then you monitor it. Metrics measure failures; failures must be discovered first.

#### 5.2.2 Start Narrow: Where to Begin Defining Badcases (Don't Boil the Ocean)

During cold start, don't try to cover all failures with the master table at once. **Pick the single highest-frequency, most measurable failure and run the complete loop for it** (source: OpenAI/Sy Truong "avoid boiling the ocean").

- **The first round's purpose is to validate "can the operations process actually run" — not to "capture as much as possible."** Pick one failure (e.g., a high-frequency routing error, or "should have clarified but didn't"), and run through the full cycle: discover → label → attribute → fix → verify. Then replicate for other types.
- **Have a clear completion metric**: be able to say "this failure is fixed when it drops to X."

**Priority for capturing badcases in the first round (by signal purity):**
1. **User-confirmed failures (purest signal, highest priority)**: corrections in conversation ("no, I meant X"), thumbs-down (especially with reason), "regenerate/edit" clicks — the user has already determined this is a problem; received = clean badcase.
2. **System-objectively-marked failures (unambiguous)**: triggered fallbacks (system self-assesses it can't handle), user abandonment after clarification, explicit errors (broken link).
3. **Behavior-inferred suspected failures (noisy, use caution)**: user abandonment, multi-turn unresolved — add to "suspected pool," validate by sampling, don't directly count as badcases (see 5.2.6).

#### 5.2.3 Three Data Sources: Roles and Correlation

**Three-source roles (complementary, covering different blind spots):**

| Data source | What it captures | Who it covers |
|---|---|---|
| Thumbs-down (user active feedback) | User's explicit dissatisfaction + reason options | Users willing to speak up |
| Instrumentation (user behavior) | Objective behavior (clicks, abandonment, regeneration…) | All users (including silent ones) |
| Backend labeling (attribution classification) | Tagging data with master table dimension labels | — (processing, not collection) |

> System-proactive signals (computed backend, not triggered by user action) are the operations workhorse because they cover silent users: fallback trigger rate, clarification rate, multi-turn unresolved rate, same-question recurrence rate, behavioral/outcome signals.

**Three-source correlation: use a canonical feedback event to anchor all three sources together** (source: OpenAI/Arize).
- **Not three tables hardjoined — design a unified "feedback event" structure** that funnels thumbs-down, instrumentation, and conversation corrections into one place.
- **Correlation key = trace_id / conversation_id**: every feedback event carries the conversation identifier, so thumbs-down, instrumentation, and labels naturally point to the same conversation.
- **Preserve raw text + provenance; schema supports "reprocessing"**: because the master table and labelers will evolve, raw data is preserved in full so old data can be relabeled with a new taxonomy when the master table is upgraded. **Storing only "pre-labeled tags" is insufficient — the raw conversation must be stored; otherwise, master table changes render historical data worthless.**

#### 5.2.4 Thumbs-Down Option Design Principles

The essence of thumbs-down is **making the user's thumbs-down action simultaneously accomplish the initial classification of the badcase**.

- **Options map to internal attribution**: each colloquial option has an internal tag behind it, mapped to the master table's failure type/attribution dimension (e.g., "didn't understand me" → routing/intent error; "answer was inaccurate" → knowledge/generation). This turns "user thumbs-down" directly into "labeling input."
- **Let users select by "phenomenon," not by "technical attribution"**: users can accurately perceive "wrong answer / page froze," but struggle to accurately distinguish "forgot context vs. never understood in the first place" (that's technical attribution). Options with technical attribution framing (e.g., "forgot context") yield low-confidence labels that should be treated as "suspected" for backend verification.
- **Thumbs-down and reason selection are two independent events; not choosing a reason still counts as thumbs-down**: prioritize recall (was there dissatisfaction), then add precision (why). Users too lazy to choose a reason — a plain thumbs-down is already a valid negative signal.
- **Include "Other + free text"**: catches new failure types not anticipated in the master table (= users doing open-ended coding for you), an entry point for discovering master table blind spots.
- **Remove catch-all options (like "not helpful")**: they cannot be reverse-mapped to any specific failure and generate dirty data. "No selection also counts as thumbs-down" already catches lazy users; "Other" serves as an honest fallback.
- **Don't add follow-up questions after thumbs-down**: user patience is already low at thumbs-down time; follow-up questions cause frustration. All major companies avoid this.

> Options should fit the application type: the Helpdesk Assistant is "retrieval Q&A + light operations," but if a certain failure type has been architecturally eliminated (e.g., resource and link are now in 1:1 correspondence → "resource mismatch" can't exist — if the wrong thing is shown, it must be a wrong upstream content judgment), there's no need to add an option for it — good architecture can eliminate an entire failure class.

#### 5.2.5 Backend Labeling Plan

Labeling = tagging each badcase with master table dimension labels. **The master table is the labeling dictionary** (testing uses it as the acceptance standard; operations uses it as the labeling dictionary — single source of truth).

**Three-dimension labeling:**

| Dimension | Values | Use |
|---|---|---|
| Failure level (severity) | S0 / S1 / S2 | Priority, whether to block |
| Attribution | Routing/intent / Knowledge base / Generation / System-frontend / Light-op execution | **Determines who fixes it (most critical)** |
| Scenario | Master table ID (T01-1…) | Locates the row in the master table; feeds back into it |

**Human-AI division for labeling (same as error analysis):** Humans (PM/domain expert) first look at the data, discover new labels not in the master table, define granularity (A); LLM batch-labels and counts using the updated master table (B).

**The labeler (classifier) must be versioned** (source: OpenAI):
- When using an LLM to auto-label, that LLM is the "classifier."
- **Changing the labeler's prompt/logic causes spurious fluctuations in certain badcase statistics** — what looks like "the product got worse" (product trend) may actually be "the measuring stick got more sensitive" (classifier update masquerading as trend).
- Therefore: version every master table and labeler change; when the labeler changes, recalibrate it (like recalibrating an LLM judge), or past and current statistics are not comparable.
- **Look at precision by category, not total precision**: different failure categories have different mis-classification costs (matching severity); S0-category labeling precision requirements far exceed S2.

#### 5.2.6 Behavioral Signals: Inferred, Not Certain — Watch Trends, Not Individual Cases

Behavioral signals (abandonment, multi-turn unresolved, etc.) are **inferences** — individual signals are noisy; **evidence sequence combinations** improve confidence:
- **"User explicitly corrects"**: result card → user denies/restates same intent → often accompanied by "regenerate." Sequence combination yields high confidence.
- **"User abandonment" vs. "casual conversation ending"**: the key distinction is not "was an ID triggered" but "was there an unsatisfied explicit business intent" — abandonment = had business intent but didn't click any result entry and stopped; casual = never had a clear business intent.

**But even combinations are imperfect**, so: **behavioral signals are reliable for group trends, not individual determinations.** Correct usage: define a loose "suspected" rule (has business intent + no result entry click + stops) to populate a candidate pool, then **sample to estimate proportions** — don't try to make precise per-item determinations.

> Sampling bias warning (source: QASkills): looking only at thumbs-down samples **overestimates failure rate** (thumbs-down is a negatively biased sample, not random). Don't use the thumbs-down pool directly to calculate "overall failure rate"; reports must specify what the sample represents.

#### 5.2.7 Signal Value Tiering: Allocate Processing Cost by Signal Value

Different signals have different signal-to-noise ratios; allocate processing cost accordingly (source: IrisAgent + real project practice):
- **High-value signals (explicit user corrections, thumbs-down with reasons) → human review**: high information content, user cares — worth the high processing cost.
- **High-volume, low-information signals (behavior instrumentation, plain thumbs-down) → LLM batch processing / aggregate statistics**.
- Reference signal weight ranking: explicit correction > thumbs-down with reason > plain thumbs-down > session abandonment.

#### 5.2.8 Error Analysis: The Analytics Engine of Operations

(The method for badcase labeling and master table calibration; the core of operations loop step ③ "how to analyze." Source: Hamel/Shreya: open coding → axial coding → theoretical saturation.)

**Four-step process:**
1. **Build the dataset**: collect representative interaction traces (trace = input + system output).
2. **Open coding (human; PM/domain expert leads)**: before looking at the master table, faithfully record the **first failure** in each trace like "journaling" (upstream errors cause downstream cascades; the first failure is usually the root cause).
3. **Axial coding**: synthesize scattered notes into a failure taxonomy and **count frequencies per category** (turning "a pile of qualitative problems" into "an action map ranked by frequency").
4. **Iterate to theoretical saturation**: stop when new traces no longer produce new categories. Rule of thumb: review at least 100 first to establish a base; then stop when ~20 consecutive traces yield no new categories. **Capture real high-frequency failures — don't strive for exhaustiveness.**

**Three key insights:**
- **① Breaking the bootstrapping cycle**: error analysis requires interactions, but it's the first step — no data during cold start? Solution: interaction = input + output; what's missing is only inputs, which can be **synthesized**. Synthesis starts from **deductive assets** (PRD business definitions + PRD real corpus + master table hypotheses). But synthesis only covers "things you can think of" — **real traces from actual launch/gray release are irreplaceable** (they surface failures beyond imagination).
- **② Human-AI division**: axial coding has two sub-tasks — **A: discovering categories** (creative, must break out of existing classification) **must be human** (bottom-up induction, look before seeing the master table, then compare with the master table to find gaps, human defines granularity); **B: applying categories + counting** (mechanical, needs scale and consistency) **hand to LLM** (using the updated master table as basis is perfectly appropriate). "Using LLM to help" means B, not A; once separated, "discovering blind spots" and "using the master table as basis" no longer conflict.
- **③ Deduction × induction complementary**: the master table is a **deductive hypothesis** (reasoned from the methodology + PRD); error analysis uses real data to **inductively calibrate**. Optimal path = deduction as foundation (master table gives direction) + inductive calibration (data fills in surprises), iterative loop.

**Complete flow embedded in two rounds of testing:**
```
[Cold start · Round 1: Exploration — produces calibrated master table]
1 Build dataset: PRD corpus + master table guidance → synthesize challenging inputs → feed to system → synthetic traces
2 Open coding (human): before looking at master table, record first failure per trace
3 Axial coding: A discover categories (human: induce → compare master table → find gaps) B apply + count (LLM)
4 Iterate to saturation: review 100 as base; stop at ~20 consecutive traces with no new category; capture high-frequency
   → Output: master table v+1 (data-calibrated) + frequency/priority per category + stable classification standard
[Round 2: Formal testing — using calibrated master table to test for pass/fail]
[Post-launch: thumbs-down trace priority sampling → continuous error analysis → master table continuously evolves]
```
> Round 1 (exploration) ≠ Round 2 (acceptance): Round 1's output is "a better master table," not "a pass rate." Allow it to be messy, allow incomplete coverage. Don't apply acceptance standards to Round 1.

#### 5.2.9 Attribution and Routing

- **Attribution relies on infrastructure, not intuition**: attribution (routing/knowledge/generation/system/execution) is found via **searchable traces + assertions that can auto-flag errors**: went to the wrong door → inspect the router; incomplete answer → inspect knowledge base/generation; persona broke down → inspect system prompt.
- **Attribution determines routing**: the attribution dimension is the interface between operations and engineering — correct attribution routes badcases automatically to the right person.
- **High-frequency edge → promote to core**: the highest product-value output from operations — discovering that a high-frequency edge scenario is worth investing in, and upgrading it from edge capability to core capability.
- Attribution only happens after actual performance falls short — **unvalidated speculation must not be written back into the capability boundary**.

#### 5.2.10 Privacy and Data Governance (Especially Helpdesk Assistant)

Define privacy rules for the feedback system from day one (source: OpenAI): who can view raw conversations, how long data is retained, what must be desensitized, which downstream agents can query raw conversations. The Helpdesk Assistant involves IT service data (including employee accounts and device information) — this section is non-optional. **Aggregated signals are only useful when the collection policy is clearly readable to those responsible.**

### 5.3 Generation QA (Used Both Pre and Post Launch: Writing and Enforcing Capability Boundaries in Prompts)

Generation QA = PM's quality control over "whether the prompts engineering wrote are sufficient and whether they hold the capability boundary." Method sources: Anthropic Prompt Engineering official documentation + real project practice.

#### 5.3.1 Prerequisite: Framework Standards Come Before Prompt Engineering

Before touching prompts, you need three things: **success definition + testing method + first draft** — otherwise it's blind optimization.
- **"Success criteria" has two levels**: framework standards (capability boundaries, expected behavior per zone, edge handling) > test questions (cases). **Framework is the mother; test questions are the children. Framework first.**
- Framework standard (= capability boundary master table) serves dual purpose: ① guides what the prompt should say ② serves as the yardstick for prompt QA (does the prompt cover every boundary and edge case handling in the framework). **Without the master table, prompt QA has no measuring stick; with it, QA has a standard.**
- Core mental model: **treat writing prompts like giving instructions to "an extremely intelligent new employee who has zero background on your project"** — they don't lack intelligence, they lack the business context and implicit expectations in your head. "What you think doesn't need saying" is exactly what most needs to be said.

#### 5.3.2 Prompts Are Not a Cure-All: First Attribute, and Attribute to "Which Segment"

- Not every failure should be fixed with a prompt. Attribute before QA: is this a prompt problem, or a knowledge supply / system problem? Using prompts to patch a knowledge base gap is treating the wrong thing.
- **Attribution dimension is not "prompt vs. non-prompt" — it's "which component / which segment of the prompt"**: an agent has multiple prompt segments — routing/intent recognition (one segment), matching/retrieval, response generation (another segment) — plus non-prompt knowledge supply and system engineering.
- Key clarification: **"fixing the router" and "fixing the generation prompt" are both changing prompts, but different segments**. Attribution must be precise: "which segment to fix": intent misclassified → fix the routing prompt segment; wrong phrasing → fix the generation prompt segment; content missing → fix the knowledge base (prompts can't save it); interface wrong → fix the system. **Precise to "which segment" — engineering won't fix the wrong thing** (look at the trace: if the intent step was already wrong → fix routing; if intent was correct but later steps were wrong → look downstream).

#### 5.3.3 Prompt Engineering Ladder (From Most Effective to Most Fine-Grained)

Anthropic's techniques form an ordered ladder, not a scattered set of tips:

1. **Clear, direct instructions** (most effective): say positively "what you want" (not just "what you don't want"), give ordered steps, be specific enough to execute. Test: can a zero-background new employee follow this without guessing?
2. **Multiple examples**: give "input → ideal output" pairs; especially include examples of boundary/easily-confused scenarios (the master table's easily confused scenarios are both test questions and the best examples to include). Test: does the prompt include examples? Do they cover easily confused scenarios?
3. **Chain of thought (CoT)**: have the model write out reasoning before giving a conclusion, forcing it to actually work through intermediate steps — improves complex task accuracy. But adds latency cost; use only for tasks worth thinking harder about. **New reasoning models have internalized CoT — don't add it manually.** (Shares the same root as the Helpdesk Assistant's "inferential clarification" — both are "making the model's internal reasoning visible": one improves accuracy for the model, one resolves ambiguity for the user.)
4. **XML tags**: give the prompt structure — let the model distinguish instructions / context / examples / user input; key rules must be explicitly framed, not buried in paragraphs; long documents go first, query goes last (can improve ~30%); tags should be semantic and consistent.
5. **Role definition**: a one-sentence lever that activates an entire set of implicit behaviors ("you are a senior IT support engineer" implies professional, rigorous, cautious). But **it only sets the general direction, not details** — a persona can't just say "who you are."

#### 5.3.4 What to Put in the System Prompt + 8-Point QA Checklist

System prompt holds "globally stable identity and behavioral principles." A complete persona has five layers: ① Role/identity ② Capability declaration (especially "what it explicitly doesn't do") ③ Behavioral rules (clarification/fallback/tone) ④ Red lines (S0, explicit) ⑤ XML structure.

**8-point QA checklist (when engineering gives you a prompt version, check each):**
1. Role: is it defined? Is the tone right?
2. Capability declaration: does it say "what it can do"? Especially "what it explicitly won't do"?
3. Clarification logic: is it specified clearly — when to ask, what to ask? (Not vaguely "ask when information is incomplete")
4. Fallback logic: is it specified clearly by category — when to fall back, what to communicate, what action follows?
5. Red lines (S0): are absolutely prohibited actions explicitly listed in the system layer?
6. Concretely executable: for each rule, can a zero-background new employee follow it without guessing?
7. Structured: is it organized in blocks (XML), or mushed into one blob?
8. Drift-resistant: are the most critical rules in a prominent position, sufficiently highlighted? (Long conversations dilute system rules — critical rules must be anchored prominently and structured; don't assume the model will remember them throughout)

#### 5.3.5 System Prompt Detail Calibration (Write It Right, Not Write It All)

The hardest feel in QA — not "more detail is better" but "detailed where it matters, leave whitespace where it doesn't." Three tests determine "what goes in the system prompt":

- **Does it affect behavior?** Only background that affects behavior goes in (who is served, identity, usage context); purely introductory info doesn't (company overview, irrelevant brand history). Example: "serves internal employees, not external customers" goes in (affects tone boundary); "the company was founded in year X" doesn't.
- **Can it change?** Can't change (red lines, judgment rules, core information) → write it in, write it specifically. Can change (phrasing, tone details) → give "key points + examples" — don't lock down word-for-word.
- **Can it be directly executed?** Yes → sufficient. Requires guessing → make it specific enough to execute.

**How to write fallbacks (demonstrating this calibration):** judgment rules must be locked in (when to take which fallback + what determines it), **but phrasing doesn't need to be locked word-for-word** — use "what to convey (key points) + example tone." Judgment and core information don't drift; phrasing retains naturalness (otherwise responses sound robotic).

> Warning against the opposite: a system prompt is not better the longer it gets. Too long → key rules are buried; too many rules → mutual conflicts; phrasing locked too tight → robotic. The goal is "write it right," not "write it all."

#### 5.3.6 Skill: Scene-Specific On-Demand Prompt Loading (Prompt System Layering)

5.3.1–5.3.5 covered what the system prompt should say, how to QA it, and how to calibrate its detail level. But one structural problem remains unsolved: **the more the system prompt carries, the harder it is to avoid the three failure modes from 5.3.5** (too long → key rules buried; too many rules → mutual conflicts; phrasing too locked → robotic). The root cause: the system prompt is **always-on globally** — loaded in full on every conversation turn, regardless of what scenario the user is currently in. When a product has multiple scenes each with their own operating procedures, stuffing everything into the system prompt causes mutual dilution.

Skill is the solution: **upgrading the prompt system from "one monolithic system prompt" to a two-layer structure of "system prompt (global layer) + skill (scene layer)."**

##### 5.3.6.1 What Is a Skill

Skill = **scene-specific, on-demand-loaded prompt**. Essentially it's just a prompt, but with two key differences:

- **On-demand loading**: at rest it occupies only one trigger description (a few dozen tokens); when a scene is matched, the full instructions are loaded into context. When not matched, it consumes no tokens and doesn't dilute global rules.
- **Scene-specific**: each skill serves one concrete scenario (e.g., "fallback handling," "query procedure," "find template"), and internally can specify the complete operating procedure for that scenario — steps, sequence, which tools to call, output format, tone requirements, boundary handling.

In one sentence: **system prompt is "who you are and how to behave globally"; skill is "for this scenario, specifically how to execute."**

##### 5.3.6.2 Skill vs. MCP vs. Plugin

The three solve completely different problems and must not be confused:

| | What problem it solves | Nature | Example (Helpdesk Assistant) |
|---|---|---|---|
| **Skill** | Model has the ability, but doesn't know **how to do it by your standard** | Scene-specific operating procedure (prompt) | "Fallback response operating procedure" — what to say first, what next, what information must be included, tone requirements |
| **MCP** | Model **can't reach external systems and data** | External connection channel (tool call protocol) | Query ITSM to get procedure entry point, call engineer data API |
| **Plugin** | Need to **package skill + MCP + commands for distribution** as an installable unit | Distribution and composition shell | Package the full Helpdesk Assistant capability (knowledge retrieval skill + ITSM connection MCP + fallback skill) for other teams to reuse |

**Relationship**: a skill can specify "which MCP tool to use at step N"; a plugin can contain multiple skills and MCPs. The three are nested, not mutually exclusive.

**Decision mnemonic:**
- Failure because "can't connect to external system" → MCP
- Failure because "model knows what to do, but not how to do it by your standard" → Skill
- Need to package a complete set of capabilities for others to use → Plugin

##### 5.3.6.3 When to Use a Skill (Conditions for Use)

**Attribution checklist (eliminate each item; what remains is skill's domain):**

| Elimination item | If yes, what to change |
|---|---|
| Model can't reach external systems/data | → MCP (add interface) |
| Flow direction requires orchestration (multi-step tool calls, look at result then decide next step) | → Workflow (code orchestration) |
| Model needs to autonomously plan an unknown flow | → Agent |
| Model's reasoning ability insufficient (complex logic, multi-step inference) | → CoT / reasoning model |
| Global rules in system prompt themselves are wrong or missing | → Fix system prompt |
| Knowledge base / data is missing content | → Supplement knowledge base |

**None of the above, but model execution quality is still unstable — it knows what to do, but not how to do it by your standard**: → That's skill's job.

**Typical signals** (the following situations likely require a skill):

1. **Same scenario, execution varies each time** — format, step sequence, information organization, level of detail all differ across runs, even though system prompt already has rules. Means the rules are too abstract; this scenario needs a concrete operating procedure.
2. **A scenario's operating procedure is complex enough that putting it in system prompt would bury other rules** — e.g., fallback handling has five sub-situations each with phrasing key points and follow-up actions; putting it all in system prompt would push routing rules to the margins.
3. **Rules for different scenarios interfere with each other** — "query procedure" requires rigorous restraint; "casual conversation fallback" requires warm guidance. Two sets of tone rules in the same system prompt cause the model to mix them.
4. **Changing one thing in system prompt causes another scenario's behavior to drift** — because global rules are so numerous, a change in one ripples into other scenarios.

##### 5.3.6.4 System Prompt vs. Skill Division Principles

**What stays in system prompt (global layer):**
- Role identity ("you are the Helpdesk Assistant, an internal IT helpdesk AI assistant for [company]")
- Capability declaration (what it can do; what it explicitly won't do)
- Global judgment rules (when to ask clarifying questions, when to fall back, when to refuse — **judgment conditions**, not execution details)
- S0 red lines (absolutely prohibited, explicitly listed)
- Global tone baseline
- **Skill index** (tells the model "what skills exist and when to trigger each" — doesn't need full skill content, just trigger descriptions)

**What gets extracted into skills (scene layer):**
- Complete operating procedure for a scenario (steps, sequence, what to do at each step)
- What tools to call and in what order for that scenario
- Output format and information organization requirements for that scenario
- Tone and phrasing key points for that scenario (not locked word-for-word — "what must be conveyed + example")
- Edge handling and degradation for that scenario

**Division mnemonic**: system prompt governs "when to take this path" (judgment rules); skill governs "how to navigate once on this path" (execution procedure).

**Helpdesk Assistant fallback example — two-layer separation:**

```
[System prompt only contains judgment rules]
Enter fallback handling when any of the following conditions hold:
- User need exceeds product capability scope (Out-of-Scope Zone)
- Knowledge base cannot support a complete answer and clarification cannot lift the constraint (Edge Zone degradation)
- System returns an anomaly that prevents normal completion
→ For specific fallback operation, refer to skill:fallback-handling

[skill:fallback-handling contains the execution procedure]
Fallback handling operating procedure:
1. First confirm: what evidence determines we enter fallback (no knowledge base hit / system anomaly / out-of-scope determination)
2. Response structure:
   - First paragraph: state what CAN be done (if anything) — do not start with an apology
   - Second paragraph: honestly state what currently cannot be done and why
   - Third paragraph: provide a genuinely available next step (human channel/alternative path) — don't fabricate
3. Handle by sub-situation:
   - Out-of-scope request → clearly state it is outside service scope + provide genuine handoff path
   - Knowledge insufficient → state confirmed portion + flag uncertain portion + suggest contacting IT support team
   - System anomaly → state that current service is temporarily limited + provide alternative contact method
4. Prohibited: fabricating capabilities/causes/handoff results; presenting uncertain content as certain
5. Tone key points: restrained, not excessively apologetic, not deflecting (Example: "What I can confirm right now is… but for…, I'd suggest contacting the IT support team directly — they can give you a more accurate assessment.")
```

##### 5.3.6.5 What Goes Inside a Skill (Structure)

A complete skill structure:

1. **Trigger description** (one sentence, permanently in system prompt's skill index): which scenario triggers this skill.
2. **Preconditions** (optional): what must be confirmed before entering this skill (e.g., "user need has been determined to be Out-of-Scope Zone").
3. **Operating steps**: in sequence, specify the complete execution flow for this scenario — what to do at each step, what tools to call (if any), what to output.
4. **Output specification**: response structure, information organization, and format requirements for this scenario.
5. **Branch handling**: different sub-situations within this scenario and how to handle each.
6. **Boundaries and prohibitions**: errors specifically to be avoided in this scenario (may reference the master table's S0/S1 key points for this scenario).
7. **Tone and phrasing key points**: not locked word-for-word — give "what must be conveyed + example tone" (same calibration principle as 5.3.5).

**Design principles:**

- **One skill = one scenario — don't stuff multiple in** — "query procedure" and "find template" are two skills, not one "find things" skill. Granularity aligned with the scenario classification in the capability boundary master table.
- **Skill-level detail/whitespace calibration follows the same three tests as 5.3.5** — judgment rules locked in, phrasing gets key points, "can it be executed" is the bar. A skill is not a second system prompt — it can't become another blob.
- **A skill can specify tool call sequence** — "step 1: query knowledge base; step 2: query ITSM for entry point; step 3: assemble response" — making the skill the operational manual for a workflow node's internals. (A skill doesn't determine flow direction — that's the workflow's job — but it determines how one step is executed internally.)
- **Skill's relationship to the master table**: master table defines "what result this scenario should achieve" (acceptance standard); skill specifies "how to execute to achieve that result" (operating manual). Master table is "what"; skill is "how."
- **Skill testing**: skills don't change the testing method — still use the master table's expected behavior and L1/L2 determination. Skills change "what instructions the model receives to achieve the expected behavior." If adding a skill improves pass rate for that scenario, the previous failures were indeed "doesn't know how to do it" rather than "can't do it."

##### 5.3.6.6 Relationship to Other Parts of the Methodology

- **Connects to 5.3.2 (prompts are not a cure-all)**: when attributing "to which segment of the prompt," a skill is an independent prompt segment. Attribution finds "query-procedure response format is unstable" → fix skill:query-process, not the system prompt, not the workflow.
- **Connects to 5.3.5 (detail calibration)**: skill is the structural solution to system prompt bloat — not cutting the detail, but putting the detail in the right place. Global layer is lean (written right); scene layer is rich (loaded on demand).
- **Connects to 5.4 (architecture selection)**: skills don't change architecture. Workflow decides flow direction; skill decides execution quality within a step. A workflow node can load a skill internally to improve that step's stability (see 5.4 Zeroth Cut).
- **Connects to the capability three zones**: Core Zone skills should cover complete operating procedures (guarantee outcomes); Edge Zone skills should include degradation handling (guarantee handling); Out-of-Scope Zone skills should specify boundary statements and handoff paths (guarantee boundaries).

### 5.4 Architecture Selection: CoT / Workflow / Agent / Skill and Autonomy

For a multi-step AI task, first ask "who decides the steps, who holds the decision rights" — then select the architecture. This determines how capability boundaries land at the system level. **But before changing architecture, first rule out a lighter-weight possibility.**

#### 5.4.1 Zeroth Cut: Is Changing the Architecture Even Necessary?

Before changing architecture, ask: **is the failure in "the flow direction," or in "a step's execution quality"?**

- **Flow direction wrong** (should take path A, took path B; should call a tool but didn't; should have clarified but hard-answered) → this is an architecture problem; read on to the First Cut and Second Cut.
- **Flow direction correct, but a step is executing poorly** (went to the correct fallback branch, but the fallback response's format/information organization/tone varies each time; went to the correct procedure-query node, but the response structure is unstable) → **this is not an architecture problem — it's a skill problem.**

Skill is the lightest intervention: don't change workflow orchestration, don't change system prompt global rules, don't introduce agent autonomy — just give the model a concrete operating procedure for that one step, specifying "do it this way in this scenario." Loaded on demand, active only in that step, doesn't affect other scenarios.

**Decision mnemonic**: if the problem can be described as "the model knows what to do but doesn't know how to do it by your standard" — that's a skill. If it can be described as "the model took the wrong path" or "the model didn't take the path it should have" — that requires changing architecture.

> Connects to attribution: 5.3.2 says "attribute to which segment of the prompt" — skill is precisely what lets attribution reach "a specific scenario's operating procedure." Changing skill:fallback-handling only affects the fallback scenario; changing system prompt routing rules may affect all scenarios. **Use skills when skills can solve it; use system prompt when system prompt can solve it; use architecture change only when the above fail.** Intervention priority: skill (scene level) → system prompt (global level) → workflow/agent (architecture level), lightest to heaviest.

#### 5.4.2 First Cut: Who Decides the Steps

| Step source | Use what |
|---|---|
| Model decides on the fly (I don't know the steps) | Pure thinking → CoT / reasoning model; needs to call tools autonomously → agent (with planning) |
| Human already knows the steps (I know the correct flow) | Pure thinking, no tool calls → write into prompt (pre-defined CoT template); needs tool calls with branches → **workflow orchestration** |

- **CoT is for "model decides on the fly" — not for "human already knows the steps and wants the model to follow them."** When the human already has clear steps (especially involving tool calls, look-at-result-then-act), the correct implementation is **workflow (code orchestration)** — not CoT, and not letting an agent autonomous-plan it. Known flows should be fixed; don't let the model re-invent them each time (introduces instability). Echoes *Building Effective Agents*: "use workflow when you can, agent as last resort."

#### 5.4.3 Second Cut: The Real Dividing Line Between Workflow and Agent = Who Decides "Which Path Next"

- **The dividing line is not "whether AI thinks" — it's "who decides which path to take next":**
  - **Workflow**: flow direction is decided by code/pre-defined rules. AI may do arbitrarily complex thinking **within a step** (including understanding natural language, semantic judgment), but "which path to take next" is decided by code based on the AI's output from that step.
  - **Agent**: flow direction is decided by the AI itself (decides which tools to call, whether to continue, how to plan).
- **Decision mnemonic**: replace the AI at that step with "an extraordinarily smart function with fuzzy input and deterministic output" — if the whole flow can still run according to your flow diagram → **workflow**; if the flow diagram itself needs the AI to decide in real time → **agent**.
- **Inference**: tool returns natural language that requires AI to interpret semantically → still workflow (entirely legitimate to embed an "AI-interprets-to-structured-result" step within the workflow; AI is the "fuzzy-input, deterministic-output" smart node — not the flow decision-maker).
- **The AI interpretation step should output a confidence score**: low confidence routes to "clarification/fallback" branch (= technical implementation of Edge Zone), still workflow.

#### 5.4.4 Enterprise Context: Minimize Autonomy

In enterprise AI products, autonomy is a spectrum (pure workflow ←→ pure agent), and **enterprise contexts lean heavily left** (mostly deterministic flows, with AI judgment embedded at a few nodes). Reasons:

1. **High error cost, needs controllability and auditability**: enterprise errors are often irreversible (wrong reimbursement, wrong legal guidance), and agent autonomy = unpredictability. Enterprises need to explain "why each step went this way" — workflow is auditable; agent is not.
2. **The correct flow is usually already known**: enterprises have definite business processes/policies. If already known, use workflow to fix it. Having an agent explore an already-known answer is deliberately inviting instability.
3. **Stability and consistency required**: an agent "thinks" differently each time — the same input may take different paths across runs. Enterprises need consistency.

**Principle**: minimize autonomy — give only what is "genuinely needed for intelligent judgment and impossible to pre-determine," and fix everything else with workflow. A more mature enterprise AI design is "confine AI in the workflow's cage; give a little autonomy only where it must be given."

> Connects to capability boundaries: **Core Zone (deterministic capability) suits workflow fixation; Edge Zone (judgment needed) is where model reasoning is required.**
> Helpdesk/Expense Assistants are most likely "workflow as skeleton + AI embedded at a few nodes," not pure agents. The Helpdesk Assistant's "inferential clarification" specifically converts AI autonomous judgment into user choice — recapturing one layer of controllability. This is precisely the safer design in enterprise contexts.

#### 5.4.5 Autonomy Level-Setting Tool: Error Cost × Detectability Matrix

5.4.4 said "minimize autonomy," but concretely which actions can be released and which must be retained? Don't set one safety level for "the whole product" broadly — instead, for **each action**, judge separately by "error cost × detectability."

```
                     Error easy to detect       Error hard to detect (silent corruption)
High error cost      Deploy with human approval  Don't deploy unless strict guardrails exist
Low error cost       Deploy freely, autonomous   Deploy with monitoring
```

**Usage: put each action/function of the product into a cell separately.**

Expense Assistant example:
- "Query reimbursement status": low cost + easy to detect → bottom left, can be autonomous
- "Submit reimbursement": high cost (wrong amount irreversible) + may be hard to detect (quietly submitted the wrong thing) → top right, must have strict guardrails (pre-execution confirmation + human approval)

Helpdesk Assistant example:
- "Query procedure entry": moderate cost (if wrong, user will still make their own judgment) + relatively easy to detect → deploy with monitoring
- "Display unauthorized content": high cost + hard to detect (user doesn't know they shouldn't have seen this) → must have strict guardrails

**Key insight: different functions within the same product land in different cells, requiring different levels of autonomy and guardrails.** This matrix turns the intuition from 5.4.4 ("enterprise should retain autonomy") into an operable instrument — not broadly conservative, but precisely level-set per action.

> Mapping to the methodology: "high cost" cells ≈ S0 red lines (zero-tolerance, irreversible); "detectability" corresponds to behavioral signal design in 5.2 data operations (can this type of error be automatically detected in production?).

#### 5.4.6 When Capability Is Insufficient: Narrow Scope, Don't Add Orchestration

When a component's AI is unreliable, the intuitive response is "wrap another agent layer to make it smarter" or "add an orchestration layer." **This intuition is wrong.** The correct approach (in priority order):

1. **Narrow the scope**: let this component only handle the subset it can reliably handle; everything else goes to fallback/human.
2. **Simplify the problem**: remove the step that depends on the weak capability; find a simpler way to achieve the same goal.
3. **Add scaffolding**: use deterministic tools to compensate — e.g., add a validator to catch errors upstream, rather than adding a "smarter AI" to review the first AI.

**The essence**: an unreliable component needs "less to do" or "a deterministic tool as a backstop" — not "more to do." This is consistent with 5.4.4's "use workflow when you can, agent as last resort" — when problems arise, move toward determinism, not toward autonomy.

> Connects to capability zones: when a function's live performance is unstable, the first choice is not to upgrade architecture to make it stable — it's to consider demoting it from Core Zone to Edge Zone (acknowledge the capability boundary; design a degradation path). Architecture upgrade is a means, not a goal — problems solvable by boundary design should not be solved by architecture.

### 5.5 Production Guardrails and Reliability (System-Level Assurance Post-Launch)

5.1–5.4 covered testing (pre-launch gate), data operations (post-launch improvement), generation QA (prompt layer), and architecture selection (orchestration layer). One piece is still missing: **system-level protection mechanisms running in real time in the production environment post-launch** — guardrails, reliability mechanisms, human-in-the-loop.

This is not testing (testing is pre-launch). It's not data operations (data operations is post-hoc analysis and improvement). This is **the protection layer active at runtime** — the checks and interceptions that every model output passes through.

#### 5.5.1 Three-Layer Guardrails: Classified by "Where in the Pipeline"

Guardrails are not one monolithic concept. Divided by "where in the pipeline to intercept" there are three types — each solves different problems and cannot substitute for each other:

```
User input → ① Input guardrail → Core processing (routing/matching/generation) → ② Output guardrail → ③ Process guardrail → Return to user/execute action
```

**① Input guardrail — intercepts "before the model has processed"**

Screen user input before it reaches core logic: is there a prompt injection attack? Is there an unauthorized request? Is there content that should not be processed?

**Key design principle: the input guardrail should be an independent processing step — not a sentence mixed into the answer prompt.** Having the same model "answer well" and "screen strictly" — two goals that interfere with each other. Use an independent model instance (or rule engine) dedicated to screening, separate from the core answer model. Each does its own thing without compromise.

This is consistent with the workflow mindset — **the guardrail is an independent node in the workflow, not a sentence mixed into the answer prompt**.

> Methodology mapping: the input guardrail's responsibility ≈ Out-of-Scope Zone identification (out-of-scope/prohibited-item interception), but it is not a rule in the prompt — it's an independent system step.

**② Output guardrail — intercepts "after model output, before returning to user"**

After the model has given a response, run one more check before returning it to the user: has it leaked anything it shouldn't say (internal UUIDs, sensitive data)? Is the format correct? Has it made out-of-scope judgments?

**Key insight: the S0 assertions from 5.1 ("output must not contain UUIDs," "output must not contain links when unauthorized") are not just tests that run once — they can run in real time in production.** Every model output passes through S0 assertions, catching issues in real time rather than waiting for testing to discover them. This is the upgrade of assertions from "testing tool" to "production guardrail" — the same rules, two uses.

> Methodology mapping: output guardrail = S0 red line assertions made real-time in production. The S0 universal assertions (not feature-bound; must hold globally) defined in 5.1.3 directly become real-time runtime checkers in production.

**③ Process guardrail — intercepts "before critical actions execute"**

Before actually triggering an irreversible operation (submit reimbursement, initiate approval, send notification), force a stop and confirmation. This is the last line of defense.

> Methodology mapping: process guardrail = the system-level implementation of what 4.3 (task-based application variants) wrote: "irreversible or high-risk actions should be more conservative and confirmed before execution."

**Each of the three prevents a different problem:**
- Input guardrail prevents "things that shouldn't come in, coming in"
- Output guardrail prevents "things that shouldn't go out, going out"
- Process guardrail prevents "things that shouldn't be done, being done"

#### 5.5.2 Three Reliability Mechanisms (The Agent's Self-Recovery Toolkit)

Guardrails are "external interception." The following three mechanisms are "the agent's own reliability capabilities" — letting the agent know whether it did things correctly and whether it can self-correct.

**① Get ground truth from the environment after each step**

After the agent executes each step, it must get a "real result" from the environment to judge progress — what did the tool call return? Did the system state change? Did the operation succeed? — not have the AI continue based on imagination.

- After the Expense Assistant submits a reimbursement, it must receive a real system receipt (success/failure/error code), not assume "it probably succeeded"
- After the Helpdesk Assistant queries ITSM, it must receive the real returned procedure ID, not have the AI guess one from memory
- **Precondition for determining whether a component can be autonomous: can it receive real feedback?** Yes → can be more autonomous (because it can self-verify); No → must add external checks or human fallback

> This is why "coding" is an ideal agent scenario — code can run tests, see errors, self-correct. While "legal advice" is not — after giving advice, there's no immediate feedback telling the AI whether the advice was correct.

**② Verification loop**

Embed a "check your own work" step in the agent's flow. Core judgment: **when evaluating whether any component can be autonomous, first ask "can it verify that it did things correctly?" Can verify → can be more autonomous; cannot verify → needs human or tool backup.**

- After the Helpdesk Assistant matches a procedure, can it check "does this procedure actually correspond to what the user asked about"? If yes (e.g., validate procedure tags against user keywords), this step can be more autonomous.
- After the Expense Assistant fills a reimbursement form, can it check "do the amount and invoice match"? If yes (call a validation tool), this step can be more autonomous.

**Priority for designing verification loops:**
1. Best: validate with deterministic tools (code assertion, rule engine, data comparison) — result is certain
2. Second: validate with an independent model instance (one model generates, another model checks) — more reliable than self-checking
3. Worst: have the same model check itself — prone to self-justification ("I'm right because that's what I thought")

> Corresponds to 5.4.5 matrix: "detectability" dimension is essentially "whether a verification loop exists." Yes → errors easy to detect → can grant more autonomy; No → errors hard to detect → needs stronger guardrails.

**③ Error recovery**

A reliable agent needs clear stop conditions and error recovery strategies. What to do when things fail must be designed upfront — not improvised when failure occurs.

This is exactly what methodology 4.3 already covered: "Stop and recovery: clear paths must exist for failure, timeout, duplication, reversal, and human takeover." Adding specific paths:

- **Retry**: for transient failures (network timeout, API rate limiting), auto-retry is acceptable, but with a retry cap.
- **Alternative path**: for within-capability but current-path failures (retrieval misses), try alternatives (different retrieval strategy, broader matching range).
- **Degrade**: for Edge Zone issues, take safe degradation (don't hard-answer; give "what can be confirmed + handoff recommendation").
- **Hand to human**: for situations beyond recovery, hand off clearly (with already-gathered context, so the human doesn't start from scratch).
- **Stop**: for detected potential irreversible damage, stop immediately (don't retry, don't try alternatives — go directly to fallback).

#### 5.5.3 Human-in-the-Loop: Not "Whether to" but "At Which Points, In What Form"

**Autonomy is earned, not granted all at once.** The practical path has three phases:

**Phase 1: Gray release / early stage — full human review + strong guardrails**
- All three guardrail layers active (input/output/process)
- High-risk actions reviewed 100% (every item)
- Purpose: discover problems, build trust, accumulate data on "which actions are stable, which aren't"
- All cells of the 5.4.5 matrix default to the most conservative starting point

**Phase 2: Scale-up — selective human review + focused guardrails**
- Stable actions released to autonomous (low-cost + easy-to-detect cells in the matrix)
- Unstable ones maintain human review (high-cost + hard-to-detect cells)
- Guardrails focused: input guardrail may relax; output and process guardrails maintained
- Error analysis continues (5.2); when new problem types are discovered → add new guardrails

**Phase 3: Mature — agent proactively reports + monitoring as main mode**
- **Teach the agent to recognize "I'm uncertain" and proactively report for human review** — rather than having humans check every item. This is better than "humans proactively checking": humans are prone to miss things and it's costly; agent proactive reporting means humans only handle what genuinely needs judgment. This is another form of the methodology's "Edge Zone clarification/degradation" — the agent itself says "I'm uncertain, let a human handle it."
- Human review handles only reported items + sampling re-checks
- Guardrails internalized into the system; operations rely on monitoring metrics
- **But top-right cells (high cost + hard to detect) may permanently maintain guardrails, never relaxed** — the system-level embodiment of S0 red lines

**Human-in-the-loop hidden risk: automation bias**

When the model is right most of the time, human reviewers become desensitized and start blindly trusting the model while ignoring details. Attackers or systematic errors can exploit this — hiding problematic actions within a large volume of correct ones, getting them past fatigued human review.

Countermeasures:
- **Make human review targeted** — not "review every item" but "only send high-risk/low-confidence items for review" (echoes 5.2.7 signal value tiering: spend the most expensive human judgment cost on the highest signal-value samples)
- **Review interface should highlight "the point requiring human judgment"** — don't make people find the one problematic thing buried among a pile of correct information
- **Regularly refresh review strategy** to prevent human review from becoming "mechanically click approve"

> The essence of the three-phase path: **autonomy is incrementally expanded as reliability is proven** — full review → release stable ones → maintain review for unstable ones. Some cells may never be released (high cost + hard to detect). This is not "being lazy with fewer reviews" — it's "concentrating limited human capacity on what genuinely requires human judgment."

---

## VI. AI Product Philosophy (The Underlying Cognition Throughout)

All the details of the methodology ultimately converge to a few product beliefs — these are the judgment anchors to return to again and again when building AI products.

**① The essence of AI products is a series of tradeoffs; the PM is the one making the calls.**
Almost no decision is simply "right vs. wrong" — it's all tradeoffs: pass rate level (recall vs. precision), workflow vs. agent (controllable vs. flexible), system prompt detail (explicit vs. bloated, drift-resistant vs. robotic), thumbs-down options (user cost vs. data quality), severity (conservative interception vs. release experience), synthetic vs. real data (fast cold start vs. covering real scenarios). Traditional software has many deterministic decisions; AI products are everywhere probability, boundary, and degree-of-freedom tradeoffs. **Engineering can implement, but "which direction to trade off" is a product judgment — PM's core value is not "writing clear requirements" but "making a series of right tradeoffs."**

**② The core work of AI products is translating "the model's fuzzy capability" into "clear product promises."**
Traditional software's capability is deterministic (click button, get page). AI capability is fuzzy, bounded, and prone to failure at the edges. So the thing most in need of precise specification in an AI product is not "what features exist" but: where do capability boundaries lie (Core/Edge/Out-of-Scope), what should behavior look like in each zone, what counts as passing/violating, and how are failures attributed and improved — **this is the capability boundary master table, and it is the true core of an AI product PRD**. UI, flows, and fields are the periphery.

**③ Capability boundary specification must happen early — it is the foundation of everything.**
Testing, operations, and prompt QA all build on "capability boundaries specified clearly first" — the master table is the foundation; everything else is the building on top. **A PRD that ends after defining the model and the interaction has missed the hardest and most critical part of AI product development.** The capability boundary master table should be built concurrently with or earlier than the PRD, not patched in late-project (if left too late, testing/operations/prompts all become reactive).

> One-line convergence: **The core work of AI products is translating "the model's fuzzy capability" into "clear product promises" — this specification (capability boundary) must be done early, is the foundation of all testing/operations/prompts, and every point in it is a tradeoff whose direction the PM must decide.**

---

## VII. Version History

- **v1**: Formed the skeleton of three zones, actions, and three perspectives.
- **v1.1**: Split scenario, action, and attribution; introduced the two coordinate systems (capability three zones and processing chain); moved routing and intent switching out of Out-of-Scope Zone.
- **v1.2**: Corrected through real project discussion — three zones upgraded to three product promises; Core Zone adds accuracy/recall/repeated-consistency requirements; Edge Zone simultaneously observes complete handling and safe degradation; clarified that vague/multi-intent/complex questions don't automatically belong to any one zone; added cross-domain knowledge judgment rules; classified this product type as "retrieval Q&A + light operations"; Out-of-Scope Zone acceptance changed to "action match rate + boundary statement correctness."
- **v1.3**: Deep-read Hamel's *Your AI Product Needs Evals* and integrated with real project implementation. Added "evaluation operationalization" in 5.1 acceptance perspective — 1) three evaluation layers distinguished by "determination method → types of problems testable," not capability-incremental or substitutable; 2) the essence of assertions (test objective attributes, not text equality) + PM's "extract bone, leave meat" action; 3) L1 three-step implementation: feature→scenario decomposition + positive/negative/boundary sample group design (including L0 gate), LLM case synthesis (deliberately hard, paired closed-loop), regular runs + track trends + bind prompt versions; 4) pass rate definition: tolerance-for-failure perspective, tier by failure cost, L0 zero-tolerance gate, cold start direction-first then baseline, consequences check.
- **v1.4**: Completed full Hamel deep-read (L2/L3) and integrated with real project — 1) completed three-layer evaluation with L3 positioning (tests real impact not quality, general data skill, naturally lagged, but instrumentation must be done at launch); 2) added 5.1.5 "PM responsibility boundary": division-of-responsibility scale, granularity stopping at "scenario category + expectation + standard + metric direction," two metric pitfalls (don't apply F1 to subjective quality, don't look at total accuracy with class imbalance), data viewing division, tool as requirements push; 3) 5.2 monitoring refilled with L2 methods (trace, zero-friction data viewing, binary labeling, LLM judge alignment discipline), L3 behavioral/outcome signal instrumentation, cold start data strategy (PRD corpus as seed / coverage before distribution / gray release extraction), attribution via infrastructure.
- **v1.5**: Clarified overall structure of acceptance perspective — 1) added four-layer structure diagram at start of 5.1 (expectation → determination → normative/empirical → decision) + three-axis intersection (expected behavior/severity/test method) + acceptance spec table example; 2) key clarification "determination layer vs. decision layer" separation: L1/L2 binary determination needs no severity; severity/metrics/decisions at upper layers; 3) distinction between severity (normative, defined) and aggregated metrics (empirical, computed); decision = normative line × empirical value; 4) naming correction: severity S0/S1/S2 and evaluation method L1/L2/L3 are orthogonal axes, no longer sharing "L"; 5) added 5.1.6 decision layer principles: severity definition (consequence anchor, reversibility/externality/promise three-test escalation, S0 few and hard, write rationale, uncertain defaults to middle tier) + pass rate (continuing 5.1.4).
- **v1.6**: Three foundational corrections (pre-actual-project retrospective) — 1) **decouple zone classification from acceptance**: Core Zone is normative commitment; classification asks "path clear + committed?", acceptance asks "did testing pass?"; performance fluctuation changes "pass/fail," not "which zone." Removed "empirically validated accuracy" from classification criteria. 2) **routing demotion**: routing does not constitute independent scenarios (scenarios start from user needs; routing is internal action; forcing it in creates duplication/conflict), changed to three roles — action/attribution/trace dimension; multi-turn switching and multi-intent as complex-condition variants of real scenarios, tested as sub-variations, attribution focusing on routing layer. 2.4 and 5.1 table rewritten accordingly. 3) **naming cleanup**: severity unified to S0/S1/S2 (remnant L0–L3 in 5.1.4 changed to S); evaluation methods retain L1/L2/L3; "unit test" renamed to "assertion test."
- **v1.7**: Added 5.2.5 "error analysis operating procedure" (Day 3 deep-read implementation) — open coding → axial coding → theoretical saturation four steps; three extended insights: ① breaking the bootstrapping cycle (synthetic input + deductive priors, breaking the "no data at step 1" chicken-and-egg problem); ② human-AI division (A: discovering categories must be human and not bound by the master table; B: applying categories + counting handed to LLM); ③ deduction × induction complementary (master table is deductive hypothesis; error analysis is inductive calibration; iterative loop); complete flow embedded in "two rounds of testing" with "Round 1 exploration ≠ Round 2 acceptance" reminder.
- **v1.8**: Restructured Ch5 from "three implementation perspectives" to "testing + data operations" dual-phase (ADLC) — 1) clarified **L1/L2/L3 are testing methods, not data operations tools**; testing and operations distinguished by four lines (timing/data/purpose/core capability — testing = quality determination, operations = data flow + attribution), intersecting only at tail end A-B validation; 2) 5.1 testing: original acceptance content moved in as a whole, positioned as "pre-launch determination of fit to release"; 3) 5.2 data operations became standalone "data flow" major section (ten subsections): five-step loop + starting-point priority, start narrow (don't boil the ocean, capture badcases by signal purity), three-source roles + canonical event/trace_id correlation, thumbs-down design principles, backend labeling + classifier versioning, behavioral signals (inferred not certain / watch trends not individuals / sampling bias), signal value tiering, error analysis engine, attribution routing, privacy governance; 4) data operations method sources: Hamel error analysis + OpenAI/Arize feedback system review + real project practice (field lacks mature public methodology; heavily practice hypotheses, to be continuously validated).
- **v1.9**: Added "two modes of clarification" in 1.4 — **Structured Clarification (fill known fields) vs. Inferential Clarification (expose AI's reasoning branches)**. Key: when uncertainty comes from "semantic ambiguity" rather than "missing specific field," field-driven clarification doesn't work — must use inferential clarification (AI generates branches in context; no pre-defined fields needed). Corrects common engineering misconception "semantic ambiguity = cannot clarify." Validated through real project multi-ambiguity clarification scenarios.
- **v2.0**: Day 4 deep-read (Anthropic Prompt Engineering) implementation + milestone wrap-up — 1) 5.3 generation QA: framework standards before test questions, prompts not a cure-all (attribute to "which segment"), prompt engineering ladder (clear instructions/examples/CoT/XML/role), system prompt five layers + 8-point QA checklist, detail calibration (only what affects behavior / judgment rules locked / write right not all); 2) 5.4 architecture selection: two-cut judgment for CoT vs. workflow vs. agent (who decides the steps, who decides which path next), "extremely smart function" decision mnemonic, AI interpretation step + confidence branch still workflow, enterprise context minimizes autonomy; 3) added Chapter 6 "AI Product Philosophy": AI products are a series of tradeoffs (PM is the decision-maker), core work is translating fuzzy capability into clear promises, capability boundary master table must be early (is the foundation of everything).
- **v2.1**: Completed prompt system layering, architecture selection pre-check, and production guardrails & reliability — 1) 5.3.6 added "Skill: scene-specific on-demand prompt loading": defined skill as "scene-specific on-demand loaded prompt" (normally only a trigger description; loaded fully when scene matches); distinguished skill/MCP/plugin (skill=operating procedure, MCP=external connection, plugin=packaging for distribution); provided conditions for use (attribution elimination checklist — after eliminating MCP/workflow/agent/CoT/system prompt/knowledge base, remaining "knows what to do but not how to do it by your standard" is skill's domain); specified system prompt vs. skill division principles (system prompt governs "when to take this path"; skill governs "how to navigate once on this path"), including Helpdesk Assistant fallback two-layer separation example; specified skill internal structure (trigger description/preconditions/operating steps/output spec/branch handling/boundaries/tone key points) and design principles (one skill = one scenario, detail follows 5.3.5 calibration, can specify tool call sequence, relationship to master table is "how vs. what"); explained relationship to other parts of the methodology (attribution precision, detail calibration structural solution, no architecture change, capability three zone correspondence). 2) 5.4 architecture selection added "Zeroth Cut" (5.4.1): rule out skill before changing architecture — when failure is in "one step's execution quality" not "flow direction," use skill not architecture; intervention priority: skill (scene) → system prompt (global) → workflow/agent (architecture), lightest to heaviest. Original First Cut/Second Cut/enterprise context shifted to 5.4.2/5.4.3/5.4.4. 3) 5.4.5 added "Error Cost × Detectability Matrix": turns "enterprise should retain autonomy" intuition into an operable instrument — level-set each action by "error cost × detectability" (different functions in same product land in different cells), using Expense/Helpdesk Assistant as examples. 4) 5.4.6 added "When capability insufficient, narrow scope not orchestration": when an AI component is unreliable, the correct response is narrow scope/simplify/add scaffolding (deterministic tools as backstop) — not adding orchestration or more agents. 5) 5.5 added "Production Guardrails & Reliability" (three subsections): ① Three-layer guardrails by "where to intercept" — input guardrail (independent model/step for screening; don't mix into answer prompt), output guardrail (S0 assertions upgraded from testing tool to real-time production checker), process guardrail (confirmation before high-risk action execution); ② Three reliability mechanisms — get ground truth from environment after each step, verification loop (can it verify correctness? → determines how much autonomy it can have), error recovery (retry/alternative path/degrade/hand to human/stop — five-level path); ③ Human-in-the-loop three-phase path — full human review → selective review → agent proactively reports + monitoring as main mode, including automation bias risk countermeasures. Source: Anthropic reliability and guardrails practices.

> Usage note: the skeleton should be continuously validated in real projects. Project facts determine specific scenarios and actions; general methods cannot replace PRDs, formal knowledge, system capabilities, and confirmed product rules.
> Pending validation list:
> ~~① Whether Core Zone classification should be decoupled from empirical accuracy~~ → **Resolved in v1.6**: zone classification looks at path + commitment; acceptance looks at empirical testing; the two are decoupled.
> ~~② Routing/Out-of-Scope clarification ownership~~ → **Resolved in v1.6**: routing demoted to complex-condition variant + attribution + observation dimension; not an independent scenario.
> ③ (Still pending validation, confirm when Day 2 begins hands-on work) For Edge Zone, whether "safe degradation stability > complete handling rate" priority should be explicitly written as an ordering in the acceptance spec, giving QA a basis for choosing between two options.
