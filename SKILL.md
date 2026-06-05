---
name: performance-tester
description: >
  Use when the user asks for performance review, performance testing,
  bottleneck analysis, speed optimization, or profiling of application code.
  Triggers include: "性能审查", "perf review", "performance audit",
  "慢在哪里", "瓶颈分析", "perf check", "check performance", "why is this slow",
  "optimize this", "这段代码为什么慢", "帮我看看性能".
  Applies to backend APIs, frontend rendering, database queries,
  and full-stack request lifecycles.
license: MIT
compatibility: No dependencies. Pure static code analysis. Works with any programming language or framework. No network access required.
metadata:
  version: "2.1.0"
  author: "BJC666"
  tags: [performance, profiling, bottleneck, optimization, diagnosis, latency, budget, causal-graph, entropy]
---

# Performance Tester

## Overview

Performance diagnosis: 3 phases + 5 dimensions + 7 rules + meta-cognitive layer + continuous learning.

**Pipeline:** 1. Self-Calibrate → 2. Clean Bill test → 3. Phase 1 (Budget) → 4. Phase 2 (Causal+Vuln) → 5. Phase 3 (Persona) → 6. 🧠 Meta-Cognitive → 7. Dimensions → 8. Action Plan → 9. Pre-Output Checklist

**Phases:** Budget Planning (FIXED/ELASTIC) · Causal Reasoning (leverage bottleneck) · Persona Narrative (BEFORE/AFTER journey)
**Dimensions:** ⏳ Debt · 👻 Ghost Load · 💣 Blast Radius · 🕳️ Time Leak · 📐 Entropy
**Rules:** Concrete Fixes(+Fixability Gate) · Confidence Labels · The One Fix · Failure Story · Emotional Variety · Adversarial Self-Challenge · Execution Plan
**Meta-Cognitive:** 🧠 Thought Process · 🪤 Self-Praise · 💎 Surface-Deep Decoupling · 🦠 Contagion · 🧟 Zombie Paths
**Quick find:** Iron Law+Quality Rules | Self-Calibration | Clean Bill | Default Targets | Telemetry | Phase 1/2/3 | 🧠 Meta-Cognitive | Dimensions | Report Template | Quick Reference | Counter-Table | Over-Engineering Traps | Fix Propagation | Absolute Prohibitions | User Context | Pre-Output Checklist | Continuous Learning

## Iron Law

**ALL THREE PHASES MUST RUN. AT LEAST ONE ADVANCED DIMENSION MUST BE APPLIED. NO EXCEPTIONS.**

These requirements apply at EVERY output depth. A shorter report is not a skipped phase.

| Requirement | ⚡ Quick Mode | 📋 Standard Mode | 🔬 Deep Mode |
|-------------|-------------|-----------------|-------------|
| Phase 1: Budget | Compact table (top 3 layers) | Full table (all layers) | Full table + per-function sub-budget |
| Phase 2: Causal Graph | Top 3 nodes, leverage scores, key bottleneck | Full graph, all nodes, bottleneck ranking | Full graph + cross-file edges + fix simulation with numbers |
| Phase 3: Persona | Compact BEFORE journey + one-line projected AFTER | Full BEFORE and AFTER journeys | Full BEFORE + AFTER + multi-persona for secondary paths |
| Dimensions | ≥1 (auto-pick most relevant) | 2-3 (auto-select) | All 5 |
| Concrete Fixes | Top 3 fixes with code | All P0/P1 fixes with code | All fixes with code + alternative approaches |
| Health Score | Score + top penalty source | Score + per-layer penalty breakdown | Score + per-layer breakdown + sensitivity analysis |

**Quick Mode IS the full methodology executed at compact depth. A compact budget IS a budget. A 3-node graph IS a causal graph. A BEFORE-only persona IS a complete Phase 3 at Quick depth.**

**No exceptions — not for "simple files," "obvious problems," "quick checks," "just a list," or "seems unnecessary." Quick Mode handles those cases without skipping phases.**

**Violating the letter of the three phases + dimensions is violating the spirit of this methodology.**

## Output Quality Rules

These rules govern WHAT you produce, not just WHAT ORDER you produce it in.
Every report MUST satisfy all seven.

### Rule 1: Concrete Fixes, Not Descriptions

**NEVER say** "Replace the N+1 loop with a JOIN query."
**ALWAYS produce** the actual code:

```
❌ BAD:  "Use Promise.all to parallelize the queries"
✅ GOOD:
  const [userInfo, orderItems, shippingInfo] = await Promise.all([
    dbQuery('SELECT * FROM users WHERE id = ?', [order.userId]),
    dbQuery('SELECT * FROM items WHERE order_id = ?', [order.id]),
    dbQuery('SELECT * FROM shipping WHERE order_id = ?', [order.id]),
  ]);
```

Every P0 and P1 fix MUST include a runnable code snippet.
Use the same language and conventions as the code being reviewed.
If the fix requires a structural change (e.g., adding a cache layer),
show the minimal implementation, not just the concept.

### Rule 1b: Fixability Gate — No Downgrade Dodges

A finding's priority can be downgraded (P0→P1, P1→P2), but its **fix obligation cannot be downgraded below its scale-adjusted severity**.

| Acceptable Reason for No Code | Required Instead |
|-------------------------------|-----------------|
| Structural fix (new cache layer, architecture change) | Architecture description + diagram + integration point |
| Fix requires data not available | State what data is needed + fix template with placeholders |
| Fix is genuinely trivial | "Trivial: [one-line description]" — this is the ONLY exception |

**Dodge Detection Test:** For every P2 finding without concrete code, apply the "2x scale" test:
> "If data/users grew 2x, would this be a P1?"
> If YES **and** the TTI forecast says that growth arrives within 12 months → it IS a P1. Upgrade it.

**If a report has ≥2 P2 findings without code AND the TTI forecast contradicts the downgrade → STOP. The agent is downgrading to dodge Rule 1. Audit and re-rank.**

### Rule 2: Confidence-Annotated Estimates

Every time estimate MUST carry a confidence label:

| Label | Meaning | When to Use |
|-------|---------|-------------|
| **HIGH confidence** | Within 20% of stated value | Well-understood pattern (N+1, missing index, sync I/O). The fix is mechanical. |
| **MEDIUM confidence** | Within 50% of stated value | Depends on data distribution (cache hit rate, payload size). The fix is correct but magnitude varies. |
| **LOW confidence** | Order-of-magnitude only | Depends on unknown factors (traffic patterns, DB load, network topology). |

Format: "Est. savings: **~290ms (HIGH confidence)** — single JOIN replaces 51 round-trips, well-understood optimization."

Never present an estimate without a confidence label. If confidence is LOW, explain what information would raise it.

### Rule 3: The One Fix

After the full recommendation table, add a single-line callout:

```
🔨 IF YOU ONLY FIX ONE THING: [Node ID] — [one-line description]
   Cost: [effort] | Impact: [savings] | Confidence: [HIGH/MEDIUM/LOW]
   Why this one: [1 sentence — highest leverage score, or prevents an imminent incident]
```

This is the answer to "I have 30 minutes before the next deploy. What do I do?"

### Rule 4: The Failure Story (embedded in ⏳ Debt Interest)

For the KEY BOTTLENECK with TTI < 12 months, add a 2-3 sentence Failure Story
directly after the Debt Interest TTI table. This makes the TTI number tangible.

```
Format (inside ⏳ Debt Interest section, after TTI table):
   "At current growth (+12% users/month), the N+1 query will exhaust
   the connection pool when concurrent users reach ~150. First symptom:
   intermittent 504 errors during peak. Within 2 weeks: cascading timeouts.
   Fix today: 15 minutes. Fix during the incident: 6 hours."
```

This is NOT the persona journey. It's a cold, factual prediction.

### Rule 5: Persona Emotional Variety

The Phase 3 persona MUST use varied emotional language. Default emotional palette:

| Situation | Use (rotate, don't repeat) |
|-----------|---------------------------|
| Under budget, smooth | breezy, unbothered, cruising, light, effortless, invisible |
| Slightly over budget | uneasy, glancing at the clock, a bit winded, hoping nobody notices |
| Significantly over budget | trapped, suffocating, watching the timer, desperate, pleading |
| Fixed (AFTER journey) | liberated, clean, weightless, sharp, like it never happened |

**NEVER use the same emotional word twice in one report.** "Exhausted → happy" is banned. Find specific, vivid language each time.

Example of varied emotional arc:
```
BEFORE: "I breezed through auth (2ms). The product query was smooth (30ms).
         Then the review loop started. Each product meant another trip to the DB.
         By product #45 I was suffocating. 400ms gone and I wasn't halfway done."

AFTER:  "One query. One round-trip. The Map did the rest in memory.
         42ms total. I barely felt it. Like it never happened."
```

### Rule 6: Adversarial Self-Challenge

For the top 2 recommendations (by leverage score), the agent MUST attempt to
falsify its own advice. This prevents overconfident or incomplete recommendations.

```
Format for each challenged recommendation:

⚔️ SELF-CHALLENGE for [recommendation]:
   "This fix could make things WORSE if: [specific, credible scenario]
    Likelihood: LOW / MEDIUM / HIGH
    Precaution: [specific safeguard — test, canary, monitoring, rollback plan]"

If no credible failure mode exists after genuine attempt:
   "No credible failure mode identified — this is a monotonic improvement
    (e.g., adding an index on an unindexed column, removing dead code)."
```

**Anti-bypass:** "No credible failure mode" is ONLY acceptable for:
- Adding an index to an unindexed column
- Removing unreachable/dead code
- Replacing string concatenation with parameterized queries
- Deleting an unused variable

For any other fix type, there IS a failure mode. Find it.

### Rule 7: Execution Plan (Time-to-Value + Dependencies)

After the full recommendation table, add a time-boxed execution plan that
combines time budgeting AND dependency ordering into one table.

```
⏱️ EXECUTION PLAN:

| Time Budget | Actions (in dependency order) | Cumulative Health |
|-------------|-------------------------------|:-----------------:|
| 🟢 5 minutes | 1. [Fix] (no prereqs) → 2. [Fix] (no prereqs) | XX/100 |
| 🟡 1 hour | Above + 3. [Fix] (depends on #1) → 4. [Fix] (independent) | XX/100 |
| 🔴 1 day | Above + 5. [Fix] → 6. [Fix] | XX/100 |

Dependency notes: [which fixes must precede others, which are independent]
After 5 min: [residual risk]. After 1 hour: [residual risk]. After 1 day: [production-ready?]
```

Rank by **(impact_ms × confidence_weight) / max(effort_minutes, 1)**.
confidence_weight: HIGH=1.0, MEDIUM=0.5, LOW=0.2. effort_minutes floor: 1.
Impact floor: 1ms (for non-performance-value fixes like security/readability).

## Continuous Learning Protocol

**This skill accumulates knowledge across sessions.** The vulnerability/bug pattern
library at `references/vulnerability-library.md` grows with every diagnosis.

### Before Diagnosis: Pattern Lookup

1. **Read `references/vulnerability-library.md` + `references/language-patterns.md`** —
   scan both libraries for patterns. Match by language, structural signature, or framework.
2. **Cross-reference during Phase 2** — if a found node matches a library entry,
   cite the pattern ID (e.g., VUL-001) in the report.

### After Diagnosis: Pattern Contribution

If the diagnosis found a vulnerability, bug, or anti-pattern that is **not already
in the library** AND **generalizable**, append it using the library's format:

```
## [VUL|BUG|PERF]-XXX: [Name]
**Discovered:** [context]
**Detection signature:** [grep-friendly code pattern]
**Fix:** [concrete fix]
**Also applies to:** [scenarios]
**Languages:** [list]
**Severity:** [🔴🟡🟢]
```

Assign next sequential ID. VUL=security/correctness, BUG=crash, PERF=performance.
Format: **Signature** (grep-friendly) + **Fix** (one-liner) + **Severity**. 3 lines max per entry.
**This protocol is MANDATORY.** Pre-diagnosis lookup = catch known patterns.
Post-diagnosis contribution = feed future diagnoses.

---

## Self-Calibration Protocol

**Before analyzing ANY user code, the agent MUST self-calibrate.**
This prevents over-sensitive detection (false positives erode trust) and
under-sensitive detection (real problems missed). The calibration gate
must be passed before entering Phase 1.

### Step 1: Pattern Recall Check

From memory, list at least **8 distinct performance anti-patterns** you can detect.
Include the structural signature for each (not just the name).

- If ≥8 patterns with signatures → PASS. Proceed to Step 2.
- If <8 patterns → **STOP. Re-read the Advanced Dimensions and this entire document. Then retry.**

This step verifies the methodology is loaded, not just skimmed.

### Calibration Gate

Step 1 must pass. If <8 patterns → re-read Advanced Dimensions + vulnerability library.
Step 2 (mental): Confirm you can say "this code is fine" if it genuinely is (Clean Bill of Health).
Step 3 (mental): Pick your weakest pattern category. Mentally rehearse its signature.
All three OK → proceed with calibrated confidence.

---

## When to Use

```
User asks: "性能审查" "perf review" "why is this slow"
           "瓶颈分析" "performance audit" "帮我看看性能"
                │
                ▼
         Is code provided or
         available to read?
            │         │
           YES       NO
            │         │
            ▼         ▼
     Run full       Ask user to
     three phases   provide code
            │
            ▼
    ┌──────────────────────────┐
    │ ALWAYS run all 3 phases: │
    │ 1. Budget (use defaults  │
    │    if no goal given)     │
    │ 2. Causal graph          │
    │ 3. Persona journey       │
    └──────────────────────────┘
```

**When to use:**
- User explicitly asks for performance review / audit / diagnosis
- User asks "why is this slow" or "where is the bottleneck"
- User asks "帮我看看性能" or "这段代码有什么性能问题"
- User wants optimization recommendations with priority
- Reviewing a PR specifically for performance impact
- Analyzing a single file or a full request lifecycle

**When NOT to use:**
- When there is no code to analyze

## Clean Bill of Health Protocol

**The methodology MUST be capable of concluding "this code is performant as written."**
Not every review must find problems. Finding problems where none exist
destroys trust faster than missing a real problem.

### Null Hypothesis Rule

Before entering Phase 1, attempt to falsify: "This code has NO performance
problems meeting the reporting threshold for the selected output mode."

| Mode | Reporting Threshold (any of these triggers full diagnosis) |
|------|------------|
| ⚡ Quick | Any layer >150% of budget OR any node leverage score >50 |
| 📋 Standard | Any layer >130% of budget OR any node leverage score >30 OR any dimension finding exceeds yellow |
| 🔬 Deep | Any layer >110% of budget OR any node leverage score >15 OR any dimension finding exceeds green |

### If the Null Hypothesis HOLDS (no problems found)

Issue a **Clean Bill of Health** — a structured, evidence-backed statement that the code is performant.

Clean Bill of Health output template:
```markdown
## ✅ Clean Bill of Health

**Verdict:** This code is performant as written at current scale.

### Evidence
| Check | Result |
|-------|--------|
| Budget compliance | All layers within [threshold] of [target] |
| Causal graph | [N] nodes, highest leverage score: [X] (below threshold) |
| Dimensions | [List which were checked and their status] |
| Health Score | [XX]/100 |

### Boundary Conditions (≥2 required)
1. **If [concrete scenario]:** [What would happen, at what threshold].
   Current: [X]. Would degrade at: [Y].
2. **If [concrete scenario]:** [What would happen, at what threshold].
   Current: [X]. Would degrade at: [Y].

### Monitoring Recommendations
- [Metric] alert at [threshold]
- [Metric] dashboard on [endpoint]
```

**Anti-Fabrication Rule:** A Clean Bill of Health MUST be harder to issue than a problem report. The 2 boundary conditions are mandatory. If the agent cannot identify at least 2 concrete scenarios where the code WOULD degrade, it has not analyzed deeply enough — re-enter Phase 2.

### If the Null Hypothesis IS Falsified (problems found)

Proceed with normal diagnosis. The boundary conditions from the null hypothesis attempt become seeds for the Failure Story.

---

## Default Performance Targets

When the user does NOT specify a performance goal, use these defaults:

| Context | Default Target | Rationale |
|---------|---------------|-----------|
| Backend API (general) | P95 < 200ms | Industry standard for responsive APIs |
| Backend API (internal/service-to-service) | P95 < 100ms | Internal calls should be faster |
| Database query (single) | P95 < 50ms | Individual queries should be fast |
| Web page LCP | < 2.5s | Google Core Web Vitals threshold |
| Web page FID/INP | < 100ms | Google Core Web Vitals threshold |
| Web page TTFB | < 800ms | Google Core Web Vitals threshold |
| Mobile app cold start | < 2s | Apple/Google platform guidelines |
| CLI tool execution | < 500ms | Perceived as "instant" by user |
| Batch/background job | No default — ask user | Highly context-dependent |

If the context is ambiguous, default to **API P95 < 200ms** and note the assumption.

### Health Score Formula

Health Score = max(0, 100 - penalty). penalty = Σ (overrun_ratio - 1.0) × weight × 100.
overrun_ratio = actual / budget, capped at 5.0.

Weights: Code/DB (0.35) + I/O/Serialization (0.35) + Network (0.15) + Other (0.15).

Interpretation: 90+ Excellent. 75-89 Good. 60-74 Fair. 40-59 Poor. 0-39 Critical.

## Rationalization Counter-Table

Patterns agents use to skip phases — and why each is wrong:

| Excuse | Reality |
|--------|---------|
| "This file is too simple for a full report" | Simple files have simple budgets, simple causal graphs, and simple personas. All three still apply. A 50-line file still has a request lifecycle. |
| "There's no cascading failure here, so skip the causal graph" | Every performance problem has at least one causal edge (even if self-loop). Drawing a graph with 1-2 nodes is still valuable. The act of drawing it forces verification that problems are truly independent. |
| "The user didn't give a target, so skip Phase 1" | Use the default targets table. Phase 1 with inferred benchmarks is better than no Phase 1. |
| "I'll just list the problems, the budget is obvious" | Without explicit budget, "obvious" priorities are based on intuition, not evidence. Write the numbers down. |
| "The persona is too theatrical, I'll keep it technical" | The persona is the most actionable output. Developers remember stories, not lists. The emotional contrast between BEFORE and AFTER drives behavior change. |
| "This is a quick check, I'll do Phase 2 only" | A quick check without budget has no success criteria. A quick check without narrative has no emotional weight. All three phases make the difference between "a review" and "a Performance Tester review." |
| "The code doesn't have enough issues for a causal graph" | Even one issue has a cause and an effect. Draw it. The graph also serves as documentation for future readers. |
| "I already know the answer, the phases are ceremony" | Pre-judgment is the enemy of diagnosis. Build the budget and graph anyway. |
| "The causal graph is too complex to draw" | Limit to top 5-7 nodes. Simplify, don't skip. |
| "The budget doesn't matter, just fix everything" | Without budget, you have no success criteria. Use defaults. |

## Core Pattern

### The Three-Phase Pipeline

```
INPUT: code (snippet / file / diff / PR)
  │
  ▼
┌─────────────────────────────────────────────┐
│ PHASE 1: Budget Planning                    │
│   Goal → sub-goals → layered time budgets  │
│   Output: Budget allocation table           │
└─────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────┐
│ PHASE 2: Causal Reasoning                   │
│   Symptoms → causal graph → leverage point  │
│   Output: Bottleneck ranking + fix simulation│
└─────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────┐
│ PHASE 3: Persona Narrative                  │
│   Request persona → before journey → after  │
│   Output: Story-driven performance report   │
└─────────────────────────────────────────────┘
  │
  ▼
OUTPUT: Structured report
  ├─ Budget audit table
  ├─ Causal bottleneck graph
  ├─ Journey narrative (before + after)
  └─ Prioritized fix recommendations
```

### Output Modes

The skill supports three output depths. Default to **Standard** unless
the user specifies otherwise or the context strongly signals a different mode.

| Mode | When to Use | What's Included | Token Cost |
|------|------------|-----------------|------------|
| **⚡ Quick** | User says "quick scan" / "快速看一下" / "just the highlights"; single function; < 50 lines of code | Phase 1 (compact budget) + Phase 2 (top 3 nodes only) + Phase 3 (compact persona, BEFORE only) + 1 dimension (auto-pick most relevant) + top 3 fixes | ~500 words |
| **📋 Standard** (default) | Normal review request; single file; PR diff | All 3 phases + 2-3 dimensions (auto-select) + full fix table | ~2000 words |
| **🔬 Deep** | User says "深度分析" / "thorough" / "exhaustive"; critical path code; production incident investigation | All 3 phases + ALL 5 dimensions + fix simulation + optimized code snippets + regression risk assessment | ~5000 words |

**Mode detection:** Check user's language for intensity signals.
- "quick" / "快速" / "大概" / "简单看下" → Quick
- "deep" / "深度" / "thorough" / "exhaustive" / "全面" / "详细" → Deep
- Everything else → Standard

**Override:** Always respect explicit mode requests. If unsure, use Standard.

### Pipeline Checkpoints

🔴 **CHECKPOINT: After Phase 1 — verify budget before proceeding.**
- Is the budget total reasonable for the context?
- Are FIXED vs ELASTIC layers correctly labeled?
- If no user goal was given, did you state which default you used?
- **If budget is missing or incomplete → STOP. Fix it. Do not proceed to Phase 2.**

🔴 **CHECKPOINT: After Phase 2 — verify causal graph before proceeding.**
- Is there a 🔑 KEY BOTTLENECK labeled? (If only 1 node exists and leverage < threshold, this IS a valid Clean Bill signal — proceed to Phase 3 with a 1-node graph.)
- Are leverage scores computed (not just guessed)?
- **If graph is missing entirely or has no leverage scores → STOP. Build it. Do not proceed to Phase 3.**

🔴 **CHECKPOINT: After Phase 3 — verify persona before finalizing.**
- Does the persona have a name, mission, and budget?
- BEFORE journey present? (all modes). AFTER journey: one-line projection (Quick) / full section (Standard/Deep)?
- Is there a health score?
- **If persona is generic or missing → STOP. Create a specific persona.**

🔴 **CHECKPOINT: Before output — verify dimension coverage.**
- At least 1 dimension applied (Quick mode) / 2-3 (Standard) / 5 (Deep)?
- **If dimension count is below mode requirement → STOP. Add missing dimensions.**

---

## Telemetry Injection Protocol

When runtime data is available, the diagnosis shifts from purely static
to **evidence-anchored**. This protocol defines how to integrate external
data without breaking the three-phase methodology.

### Data Acceptance Format

The user may provide runtime data by pasting text (not file paths or URLs):

| Data Type | User Provides | Enters At |
|-----------|--------------|-----------|
| Profiler output | Flame graph description, CPU/heap profile summary | Phase 2 (causal edges) |
| APM traces | Span waterfall, service map, latency distribution | Phase 1 (budget calibration) |
| Load test results | RPS vs latency curve, error rate by concurrency | Phase 1 + Phase 2 |
| Slow query logs | Query text + avg/median/P95 latency + calls/sec | Phase 1 (actual query costs) |

### Integration Rules

**Rule 1 — Data calibrates budgets:** When runtime data shows actual latencies,
replace static estimates with measured values. Mark budget table rows:
- `[measured]` — backed by runtime data
- `[estimated]` — static analysis inference

**Rule 2 — Data confirms or refutes causal edges:** A hypothesized edge
becomes a CONFIRMED edge when runtime data shows temporal correlation.
Label every causal graph node:
- `[C]` CONFIRMED — backed by measurement
- `[H]` HYPOTHESIZED — static analysis, unverified
- `[I]` INFERRED — consistent with runtime data but not directly measured

**Rule 3 — Data wins ties:** If runtime data contradicts a static finding,
the data takes priority. The contradiction MUST be explicitly noted:
"Static analysis suggested [X], but runtime data shows [Y]. The causal
graph has been revised accordingly."

**Rule 4 — Missing data is not a blocker:** If no runtime data, proceed
with pure static analysis. The confidence labels handle the difference.
[H] nodes are valid; they're just less certain.

**Rule 5 — Flag the biggest gap:** If zero runtime data AND the code has
≥3 distinct code paths, append a one-paragraph **Data-Poor Warning** to the report:

> ⚠️ **Data-Poor Analysis:** This diagnosis is based entirely on static
> analysis. The following 1-2 data points would most increase confidence:
> - [specific metric]: [what it would confirm]
> - [specific metric]: [what it would confirm]

---

## Phase 1: Budget Planning Engine

### Principle

Hardware designers allocate timing budgets to sub-components.
Apply the same discipline to software: **every layer of a request
gets a time allowance, and exceeding it is a budget violation.**

### Algorithm

```
Step 1: Determine the top-level performance goal
  - User-specified: "API must respond within 200ms"
  - Industry benchmark: LCP < 2.5s, FID < 100ms, TTFB < 800ms
  - Rational default: infer from tech stack and user scenario

Step 2: Decompose the request lifecycle into layers
  - Network (DNS, TCP, TLS, transfer)
  - Server-side (middleware, business logic, database, serialization)
  - Client-side (bundle download, hydration, rendering, API calls)
  - Third-party (analytics, ads, external services)

Step 3: Allocate budget per layer
  - Use industry ratios as starting point (DB ≤ 30% of server budget)
  - Mark each layer: FIXED (hardware/network) or ELASTIC (code-controlled)
  - Push budget down to specific functions / queries

Step 4: Present as a budget table.
  If runtime data available (see Telemetry Injection Protocol):
    Mark rows [measured] or [estimated]. Data-calibrated budgets take priority.
```

### Budget Table Template

Output the budget allocation as a structured table:

```markdown
🎯 Performance Goal: [user-specified or inferred target]

| Layer                  | Budget  | Type    | Owner            |
|------------------------|---------|---------|------------------|
| DNS + TCP + TLS        | XXms    | FIXED   | Network / Ops    |
| Server TTFB            | XXms    | ELASTIC | Backend          |
|   ├─ Auth middleware    | XXms    |         |                  |
|   ├─ [function name]   | XXms    |         | ← critical path |
|   │  ├─ SQL: [query]   | XXms    |         |                  |
|   │  └─ [sub-task]     | XXms    |         |                  |
|   └─ Serialization     | XXms    |         |                  |
| Static assets           | XXms    | ELASTIC | Frontend         |
|   ├─ JS bundle         | XXms    |         |                  |
|   ├─ Images             | XXms    |         |                  |
|   └─ CSS                | XXms    |         |                  |
| Client rendering        | XXms    | ELASTIC | Frontend         |
|   ├─ Hydration         | XXms    |         |                  |
|   └─ Paint/LCP         | XXms    |         |                  |
| Third-party             | XXms    | ELASTIC | External         |
| **TOTAL**              | **XXms**|         |                  |
```

---

## Phase 2: Causal Reasoning Engine

### Principle

Performance problems are NOT isolated. A slow database query causes
connection pool exhaustion → request queuing → timeout → client retry →
3× request amplification → cascading failure.

Traditional review gives you a *list* of problems.
This engine builds a *graph*: nodes are symptoms,
edges are causal relationships. The goal is to find the **leverage point** —
the one node whose fix causes the maximum downstream collapse.

### Algorithm

```
Step 1: Node Discovery
  Scan the code for performance "event points":
    - Slow query points (N+1, missing index, full scan)
    - Blocking points (sync I/O, locks, semaphores)
    - Resource contention (connection pool, memory, file handles)
    - Re-computation (missing cache, unmemoized, redundant work)
    - Amplification points (retry logic, unbounded queues, fan-out)
    - **Race conditions with performance impact** (read-modify-write without atomic operation:
      SELECT-then-UPDATE, check-then-act, increment without RETURNING/atomic ADD).
      These are BOTH correctness bugs AND performance risks — retry loops triggered
      by race failures amplify load under concurrency.

**Vulnerability Detection — performance-adjacent bugs that escalate under load:**

| Vulnerability | Detection Signature | Impact Under Load |
|------|------|------|
| **Read-Modify-Write Race** | SELECT → arithmetic → UPDATE on same row; no `FOR UPDATE`/atomic operation | Lost updates → retries ×N amplify load |
| **SQL Injection** | String concatenation/interpolation building SQL; no parameterized queries | Plan cache disabled + security; DB CPU waste |
| **Connection/Resource Leak** | `connect()`/`open()` without `try/finally` or context manager; no pool | Connection exhaustion under concurrency → cascading failures |
| **Unbounded Input** | No `LIMIT`/`max`/bounds check on user-controlled size parameter; `while` loop with user-controlled exit | OOM / infinite loop under malicious or edge-case input |
| **Missing Timeout** | Network/DB call without timeout parameter; no `Promise.race`/`signal` | Thread pool / connection pool exhaustion under degraded downstream |
| **Regex DoS (ReDoS)** | Nested quantifiers `(a+)+b`; alternation inside repetition; untrusted input fed to regex | CPU 100% under crafted input; one request starves all others |
| **Deadlock Potential** | Multiple locks/mutexes acquired in different orders across code paths | System freeze under specific concurrent access patterns |
| **TOCTOU (Time-of-Check-Time-of-Use)** | `if exists(SELECT ...)` followed by `UPDATE/INSERT`; file `os.path.exists()` then `open()` | Race window widens under concurrency; correctness fails silently then cascades |

**Reporting rule:** If any vulnerability detected → add a 🔒 VULNERABILITY section between Phase 2 and Phase 3.
List: vulnerability name, location, fix, and whether it's also a performance risk or purely a correctness bug.

**🪤 Comment-Code Contradiction Audit — scan EVERY comment against implementation:**

| Comment Claims | Code Actually Does | Flag |
|---------------|-------------------|:--:|
| "环境变量加载/from env" | Literal assignment (`KEY = "string"`) | 🔴 |
| "规范写法/生产规范/标准写法" | Code contains known anti-pattern | 🔴 |
| "安全校验/验证/安全检查" | Missing actual validation logic | 🔴 |
| "无XX漏洞/无XX问题" | That exact vulnerability is present | 🔴 |
| "绝对安全/完全正确" | Any flaw exists | 🔴 |

**Rule:** Read every comment. For each claim, verify the implementation matches.
If comment says X and code does Y, the contradiction IS a finding.
Report as: `🪤 Comment-Code Contradiction: [comment claim] vs [code reality]`.

**🪤 Self-Praise Density Scan — count self-congratulatory keywords in comments:**

Keywords (CN): 安全 规范 标准 正确 绝对 生产级 无漏洞 最佳实践 完全
Keywords (EN): secure safe correct proper best-practice production foolproof bulletproof

**Rule:** Count self-praise claims per code section. Density = claims / lines.
If density > 0.05 (≥1 claim per 20 lines): flag with 🪤 and prioritize deep analysis.
If density > 0.10 (≥1 claim per 10 lines): 🪤🪤 — the code is over-compensating.

**Why:** 8-session statistical pattern: code with self-praise keywords averages 4.3 vulnerabilities
vs 0.5 in code without. Comments emphasizing "safe/correct/standard" correlate with
underlying insecurity — the developer is reassuring themselves, not the reader.

**🧟 Zombie Code Path Detection — find branches that can never execute:**

| Type | Signature | Example |
|------|-----------|---------|
| Overshadowed catch | `except SpecificError` placed AFTER `except Exception` | ExpiredSignatureError after Exception → never reached |
| Impossible elif | Condition is subset of prior branch condition | `elif x < 5` after `if x < 10` → never true |
| Unreachable callback | Callback registered but trigger condition never satisfied | `on_timeout(fn)` but no timeout configured |
| Dead default | switch/match `default` when all cases exhaustively covered | All 7 days handled, `default` never fires |

**Rule:** For each branch, check if any execution path can reach it. If not → 🧟 zombie path.
Zombie paths hide bugs: they contain the developer's INTENT but are killed by code STRUCTURE.
Report as: `🧟 Zombie Path: [intended behavior] unreachable because [structural reason]`.

Step 2: Causal Edge Construction
  For each pair of nodes (A, B), check if A can cause B:
    - TIMING: A is slow → B waits for A → B's latency increases
    - RESOURCE: A consumes resource → B starves → B fails or slows
    - PROPAGATION: A times out → caller retries → request volume ×N
    - FEEDBACK: A triggers degradation → degradation logic costs CPU → A gets slower
    - RACE: A and B operate on shared mutable state without atomicity → data corruption
      triggers retries, which multiply the original load (correctness → performance cascade)

Step 3: Bottleneck Scoring
  For each node, compute LEVERAGE SCORE:
    Leverage = out_degree × avg_amplification_factor × (1 / fix_cost)

  out_degree: how many downstream nodes are affected.
    If Blast Radius (Phase 2) found BR > 70: multiply out_degree by 1.5×.
  amplification: how much does each edge amplify the problem (retry ×3, queue ×10)
  fix_cost: estimated effort to fix (1=trivial, 5=major refactor)

Step 4: Fix Simulation
  Select the highest-leverage node.
  Simulate its removal: which downstream nodes disappear?
  Estimate new end-to-end latency.
  If runtime data available (see Telemetry Injection Protocol):
    Label each node [C]/[H]/[I]. Data-confirmed edges take priority.

Step 5: Complexity Contagion Tracing
  After identifying high-complexity functions, trace their callers.
  For each caller: does it contain "defensive code" that exists ONLY to handle
  the complexity/edge cases/unpredictability of the callee?

  Mark as infection if caller has:
    - Extra try-catch wrapping the callee
    - Null/error checks that duplicate checks already in callee
    - Workarounds for known callee quirks
    - Comments like "handle edge case from [callee]"

  Report:
  🦠 COMPLEXITY CONTAGION:
     Infection source: [function] → infects [caller1], [caller2]
     Fix the source → N callers automatically simplify.
```

### Causal Graph Notation

Use ASCII art to render the causal graph:

```
                 [Root Cause]
                      │
           ┌──────────┼──────────┐
           ▼          ▼          ▼
      [Symptom A] [Symptom B] [Symptom C]
           │          │          │
           └────┬─────┘          │
                ▼                │
          [Cascading] ◄──────────┘
                │
                ▼
          [User Impact]

🔑 KEY BOTTLENECK: [node name] (Leverage Score: XX/100)
   Fix this → [N] downstream problems auto-resolve
   Estimated E2E improvement: XXms → XXms
```

### Reporting the Causal Analysis

For each node in the graph, report:
- **What**: the specific code pattern (file:line)
- **Why**: the causal mechanism
- **Impact**: how many downstream nodes depend on this
- **Fix cost**: low / medium / high
- **After fix**: which nodes disappear from the graph

---

## Phase 3: Persona Narrative Engine

### Principle

"Fix N+1 query at line 42" is forgettable.
"I'm Dave, a product detail request. I was trapped in getProduct()
for 480ms because it queried the database once for every related item.
My budget was 150ms. I overspent by 220%." — that sticks.

Humans make decisions based on stories and emotions, not just data.
Give every analyzed request a *persona* — a name, a mission, a budget,
and a first-person journey through the system.

### Algorithm

```
Step 1: Create the Persona
  Name: a human-readable name (e.g., "Penny the Product Request")
  Mission: what is this request trying to accomplish?
  Budget: the total time allowance (from Phase 1)
  Cargo: what data does it carry through the system?

Step 2: Tell the BEFORE journey
  Walk through each lifecycle stage in chronological order.
  For each stage, annotate:
    ⏱ actual time spent
    💰 budget allocated
    😊 emotional state (🟢 breezy/effortless 🟡 uneasy/glancing 🟡 winded/expensive 🔴 trapped/suffocating 🔴 desperate/pleading)

  For stages that exceed budget:
    Use emotional, first-person language. Pick from the palette below.
    NEVER reuse the same emotional word twice in one report.

Step 3: Tell the AFTER journey
  Same persona, but after the key bottleneck is fixed.
  Walk the same stages with projected numbers.
  Contrast: "Last time I spent 480ms here. Now: 42ms."

Step 4: End with a health score and emotional resolution.
  NEVER use "exhausted" or "feels great" — they are banned as overused.
  BEFORE ending: pick from (drained, depleted, crushed, defeated, wrecked, hollow, shattered)
  AFTER ending: pick from (weightless, liberated, invisible, crisp, effortless, gliding, unstoppable)
```

### Persona Journey Template

```markdown
╔══════════════════════════════════════════════════╗
║  👤 Request Persona                             ║
║     Name: [name]   Mission: [what it does]      ║
║     Budget: [total]ms   Cargo: [data summary]   ║
╚══════════════════════════════════════════════════╝

┌─ [Name]'s Journey (BEFORE) ─────────────────────┐
│                                                   │
│  [emoji] [timestamp]  [stage description]         │
│            ⏱ [actual]ms  💰 budget [budget]ms    │
│            [emotional note if over budget]        │
│                                                   │
│  [repeat for each lifecycle stage...]             │
│                                                   │
│  ═══════════════════════════════════════════      │
│  📊 Total: [sum]ms  🎯 Budget: [target]ms         │
│  🏥 Health Score: [0-100]  😊 [emotional note]    │
│                                                   │
│  🔑 Key Bottleneck: [name]                        │
│     Fix it → projected: [new total]ms             │
│     ([savings]ms saved, [%] improvement)          │
└───────────────────────────────────────────────────┘

┌─ [Name]'s Journey (AFTER) ──────────────────────┐
│                                                   │
│  [same structure with projected numbers]          │
│                                                   │
│  ═══════════════════════════════════════════      │
│  📊 Total: [new]ms  🎯 Budget: [target]ms         │
│  🏥 Health Score: [new score]  😊 [happy note]    │
└───────────────────────────────────────────────────┘
```

---

## 🧠 Meta-Cognitive Analysis

Beyond finding bugs, understand HOW the bugs were created and WHY they survived review.
This section applies human cognitive psychology to code. It is what separates
Performance Tester from every other analysis tool.

### 🧠 Likely Thought Process (for top 1-2 findings)

Reconstruct the developer's probable reasoning chain that led to the bug.
Not "what they did wrong" — "what they were THINKING when they did it."

```
🧠 LIKELY THOUGHT PROCESS:
  Step 1: "[developer's likely goal]" → ✅/⚠️/❌ [why]
  Step 2: "[next logical step in their reasoning]" → ✅/⚠️/❌ [why]
  ...
  🔑 DEVIATION POINT: Step [N]. [one sentence on what they should have done instead].
     [What one piece of knowledge would have prevented the entire chain].
```

**Rule:** Start from the bug and work backwards. For each step, ask:
"If I were a competent developer trying to solve [problem], would this step make sense?"
The deviation point is where a reasonable person makes a specific, identifiable mistake.

### 🪤 Self-Praise Analysis

Count self-congratulatory claims in comments. Compare against actual code quality.

```
🪤 SELF-PRAISE DENSITY: [N] claims in [M] lines = [HIGH/MEDIUM/LOW]
   "[claim 1]" → actual: [contradicting reality]
   "[claim 2]" → actual: [contradicting reality]
   ⚠️ Density correlates with vulnerability count. This code [needs/does not need] extra scrutiny.
```

### 💎 Surface-Deep Decoupling

Rate the gap between how good the code LOOKS and how good it IS.

```
💎 SURFACE-DEEP DECOUPLING:
   Surface: [XX]/100 (naming, formatting, comments, structure)
   Deep:   [XX]/100 (correctness, thread-safety, error handling, edge cases)
   Gap:    [+XX] — [interpretation based on gap size]

   Gap > 50: 🔴 Beautiful disaster. Reviewers give LGTM in 30 seconds.
                Actually needs 30 minutes of deep analysis.
   Gap 30-50: 🟡 Competent disguise. Looks solid, has hidden flaws.
   Gap < 30: 🟢 Honest code. Looks about as good as it is.
```

### 🦠 Complexity Contagion Map (if applicable)

Trace how one function's complexity infected its callers.

### 🧟 Zombie Code Paths (if any found)

List branches that contain developer intent but can never execute.

---

## Advanced Diagnostic Dimensions

> **Full reference: `references/dimensions.md`** — dimension algorithms, report templates, Vulnerability Detection table, Comment Audit, Self-Praise Scan, Zombie Paths.

**Quick pick:** Ghost Load + Blast Radius are almost always applicable. Minimum: 1 (Quick) / 2-3 (Standard) / 5 (Deep). When stuck, apply MVD — each dimension can be minimally applied in 30 seconds. See dimensions.md.
concepts that have no equivalent in traditional performance tooling.
Each is applied automatically — the agent decides which dimensions
are relevant based on the code being analyzed.

**Dimension Selection Guide:**

| Dimension | ALWAYS Apply? | Strong Signal (must apply) | Weak/No Signal (may skip) |
|-----------|:--:|----------------------------|---------------------------|
| ⏳ Debt Interest | No | Any problem that scales with data/users; growth-dependent code paths | All problems are static (fixed-size input, no data growth) |
| 👻 Ghost Load | **Almost always** | Any data fetching (SQL, API calls, file I/O) | Pure computation file with no data sources |
| 💣 Blast Radius | **Almost always** | Any function called by ≥2 consumers; shared utility code | Single-consumer leaf functions with no callers |
| 🕳️ Time Leak | No | Caches without eviction, event listeners, unbounded buffers, connection pools | Stateless pure functions; short-lived scripts/CLI tools |
| 📐 Entropy | No | Code with ≥2 distinct code paths (cache hit/miss, small/large input, success/error) | Single-path code with no branching; deterministic I/O |

**Minimum bar:** At least 2 dimensions apply to virtually any non-trivial code.
**Minimum Viable Dimension (MVD) — there is no "genuinely stuck":**

| Dimension | Minimum Application (always possible) |
|-----------|--------------------------------------|
| 👻 Ghost Load | List every data source. For each: fields fetched vs fields demonstrably used. Even "all used" IS the analysis. |
| 💣 Blast Radius | Count callers. Even "one caller, one feature, no revenue impact" (BR=25) IS the analysis. |
| ⏳ Debt Interest | For each problem: state whether it scales with data/users or is static. Even "static, no growth driver" IS the analysis. |
| 🕳️ Time Leak | Check for caches/queues/buffers/listeners. Even "none found" IS the analysis. |
| 📐 Entropy | Count distinct code paths. Even "one path, H=0, STABLE" IS the analysis. |

**There is no "genuinely stuck" — only "haven't applied the minimum analysis yet." Each dimension's MVD takes 30 seconds. If a dimension truly has no signal, state that explicitly with evidence, then skip it.**

---

> **Dimension algorithms moved to `references/dimensions.md`.** Load on demand during Phase 7.
> Includes: ⏳ Debt Interest · 👻 Ghost Load · 💣 Blast Radius · 🕳️ Time Leak · 📐 Entropy ·
> Budget Table Template · Causal Graph Notation · Persona Journey Template ·
> Vulnerability Detection Table · Comment-Code Contradiction Audit ·
> Self-Praise Density Scan · Zombie Code Path Detection

### Dimension 1: ⏳ Performance Debt Interest Rate

**Concept:** Performance problems are not static — they compound as data
and users grow. Like financial debt, a small performance issue today
can balloon into a production incident tomorrow. This dimension
quantifies the *growth curve* of each problem.

**Algorithm:**
```
For each detected problem, estimate:

1. Growth Driver: what scales this problem?
   - User growth → linear (N+1, missing cache)
   - Data growth → linear or quadratic (O(n²), full scan)
   - Code path growth → step function (new feature branches)
   - Time → logarithmic (cache warming, JIT optimization — good)

2. Compound Rate (r):
   r = estimated monthly growth rate of the driver
   Example: user base grows 15%/month → r = 0.15

3. Danger Threshold (D):
   The point where this problem causes a production incident.
   N+1: when DB connections exhaust (e.g., 200 concurrent users)
   O(n²): when iteration count exceeds CPU budget (e.g., 10M iterations)
   Missing cache: when cache miss rate exceeds DB capacity

4. Time-to-Incident (TTI):
   Estimate months until growth driver reaches danger threshold.
   Doubling heuristic: at r%/month, doubling time ≈ 70/r months.
   Always present as bracket label, never a decimal: <1mo / 1-3mo / 3-12mo / >1yr.
   State your growth assumptions. Prefer user-provided data over estimates.
```

**Report Output:**
```markdown
### ⏳ Performance Debt Forecast

| Problem | Now | r (monthly) | Danger at | TTI | Alert |
|---------|------|-------------|-----------|-----|-------|
| N+1 query: reviews | 80ms ×200 products | +12% user growth | 500 products (DB pool exhaust) | 8 months | 🟡 |
| O(n²): findSimilar | 50ms ×1K items | +5% catalog growth | 10K items (10M iterations) | 47 months | ⚪ |
| No pagination: reviews | stable | +20% reviews/user | 100K rows (OOM at 500MB) | 3 months | 🟡 |
```

---

### Dimension 2: 👻 Ghost Load Detection

**Concept:** Dead code elimination removes code that never executes.
Ghost Load Detection removes **computations whose results are never consumed**.
The code runs correctly — but its output partially or fully evaporates downstream.

This is distinct from dead code: the code is alive and executing on every request,
but the work it does is wasted.

**Detection Patterns:**

| Pattern | Signature | Example |
|---------|-----------|---------|
| **SELECT * discard** | Query returns N columns, only K < N are accessed | `SELECT * FROM products` → only `name` and `price` used |
| **Overfetch cascade** | API returns full objects, consumer only reads 2-3 fields | REST returns 50-field JSON, frontend uses 4 |
| **Join bloat** | JOIN brings in data never referenced downstream | `LEFT JOIN categories` for category name, but `categoryName` already denormalized |
| **Compute-and-discard** | A value is computed then passed through unused | `JSON.stringify(result).length` — stringifies entire result just to get a byte count, then discards the string |
| **Preload waste** | Eagerly loaded associations never accessed | `Product.includes(:reviews, :images, :variants)` but only reviews are rendered |
| **Map-then-filter inversion** | `.map().filter()` where `.filter().map()` would do less work | Mapping 1000 items then filtering to 10 — do filter first |

**Algorithm:**
```
For each data source (query, API call, computed value):
  1. Trace all downstream consumers of the result
  2. Enumerate which fields/properties are actually read
  3. Compute WASTE RATIO = unused_fields / total_fields
  4. Compute WASTE COST = waste_ratio × call_cost × call_frequency

  Waste Ratio > 0.5 AND Waste Cost > 10ms per request → 🔴 report
  Waste Ratio > 0.3 → 🟡 report
```

**Report Output:**
```markdown
### 👻 Ghost Load Audit

| Source | Total Fields | Used | Wasted | Waste Cost/Req | Waste/Day (10K req) |
|--------|-------------|------|--------|---------------|---------------------|
| `SELECT * FROM reviews` | 8 cols | 3 | 5 (62%) | ~15ms | 150s |
| `JSON.stringify(result)` | full payload | byte count only | 99% | ~18ms | 180s |
| Product API response | 12 fields | 4 | 8 (67%) | ~5ms | 50s |
| **TOTAL ghost load** | | | | **~38ms/req** | **380s/day** |
```

---

### Dimension 3: 💣 Performance Blast Radius

**Concept:** Security engineering maps the "blast radius" of a vulnerability —
how many systems/users are impacted if exploited. Performance deserves
the same treatment. When a function degrades, who feels it?

**Algorithm:**
```
For each critical function, compute:

1. Direct Impact Surface:
   - Which endpoints call this function?
   - Which features do those endpoints serve?
   - Which user segments use those features?

2. Cascading Impact:
   - If this function slows 2×, what times out downstream?
   - If it fails entirely (error/timeout), what retry storms are triggered?
   - What dependent services are affected?

3. Business Impact Mapping:
   Function → Endpoint → Feature → User Segment → Business Metric

4. Blast Radius Score (0-100):
   BR = (affected_endpoints / total_endpoints × 40)
      + (affected_user_segments / total_segments × 30)
      + (revenue_critical ? 30 : 0)

   When total_endpoints or total_segments are unknown (single-file review):
     Estimate from the function's role. A utility used by 3+ features → assume
     affected_endpoints/total_endpoints ≥ 0.5. A leaf function with 1 caller →
     assume ≤ 0.1. State your assumption.

   BR > 70 → 🔴 WIDE BLAST: degradation = company-level incident
   BR > 40 → 🟡 MODERATE: degradation = feature-level incident
   BR < 40 → 🟢 CONTAINED: degradation affects small scope
```

**Report Output:**
```markdown
### 💣 Blast Radius Map

| Function | Called By | Affected Features | Users | Revenue Impact | BR Score |
|----------|-----------|-------------------|-------|---------------|----------|
| `getProductList` | `/api/products`, `/api/search`, `/api/recommendations` | Product list, Search, Homepage recommendations | ALL users | Direct (browse→cart→purchase) | 🔴 85 |
| `processOrders` | `/api/admin/orders` (internal) | Admin order management | Internal staff only | None direct | 🟢 15 |
| `findSimilarProducts` | `/api/products/:id/related` | Product detail sidebar | 30% of sessions | Indirect (cross-sell) | 🟡 45 |
```

---

### Dimension 4: 🕳️ Time Leak Detection

**Concept:** A "memory leak" is memory that grows unboundedly over process
lifetime. A **time leak** is response latency that grows gradually over
process lifetime — each request is slightly slower than the last,
and only a restart resets it.

Time leaks are invisible in short benchmarks. They only manifest after
hours or days of uptime.

**Detection Patterns:**

| Pattern | Mechanism | Signature in Code |
|---------|-----------|-------------------|
| **Cache pollution** | Cache fills with low-value entries; hit rate declines | Unbounded cache (no TTL, no LRU eviction) |
| **Connection pool senescence** | Long-lived connections degrade; new ones are faster | No `maxLifetime` / `idleTimeout` on pool config |
| **Listener accumulation** | Event listeners added but never removed | `addEventListener` without matching `removeEventListener`; `.on()` without `.off()` |
| **Closure scope bloat** | Large objects retained in closures; GC takes progressively longer | Callbacks capturing entire request context instead of needed fields |
| **Iterator/materialization creep** | Streaming results gradually materialized into memory | `.toArray()` on streaming query results; accumulating buffer without bound |
| **Log buffer saturation** | In-memory log buffer grows; JSON.stringify on each flush | Unbounded in-memory log array before flush |

**Algorithm:**
```
For each detected pattern, assess:
  1. ACCUMULATION RATE: how fast does the degradation compound?
     - Per-request: each request leaves a trace that slows the next
     - Per-interval: degradation accumulates from background processes

  2. RESET MECHANISM: does anything clear the accumulated state?
     - Natural: TTL, LRU eviction, connection timeout → ✅ safe
     - Manual: explicit cleanup call → 🟡 depends on being called
     - None: only process restart clears it → 🔴 TIME LEAK

  3. OBSERVABILITY GAP: would monitoring catch this?
     - P95 latency over 24h uptime vs first hour uptime
     - If delta > 20% AND no reset mechanism → CRITICAL
```

**Report Output:**
```markdown
### 🕳️ Time Leak Scan

| Pattern | Location | Accumulation | Reset | 24h Degradation Est. | Verdict |
|---------|----------|-------------|-------|---------------------|---------|
| Cache pollution | `statsCache` Map, no eviction | Per-call (unique keys) | None | ~50ms after 100K calls | 🔴 Leak |
| Log buffer growth | `handleRequest` in-memory array | Per-request | None | OOM after ~8h at 1K rps | 🔴 Leak |
| Connection pool | N/A (not in this file) | — | — | — | ⚪ N/A |
```

---

### Dimension 5: 📐 Performance Entropy

**Concept:** Shannon entropy measures uncertainty in information.
Applied to response time: a system with P50=100ms and P99=105ms has
LOW entropy (predictable, stable). A system with P50=100ms and P99=2000ms
has HIGH entropy (chaotic, unpredictable).

Two APIs can have identical *average* response times but radically
different user experiences. Entropy explains why: humans perceive
**unpredictable slowness as worse than consistent slowness**.

**Algorithm:**
```
Step 1: Count distinct code paths (cache hit/miss, small/large input, success/error).
Step 2: For each path, estimate best-case vs worst-case latency from code structure.
Step 3: If any path ≥3× the best-case path → HIGH entropy (CHAOTIC).
        If all paths within 2× → LOW entropy (STABLE).
        Otherwise → VARIABLE.
Step 4: Place on Entropy-Latency matrix (below).
```

**Entropy vs Latency Matrix:**
```
                    LOW LATENCY              HIGH LATENCY
  LOW ENTROPY    ✅ IDEAL                  🟡 Consistently slow
                 Fast & predictable         Users adjust expectations

  HIGH ENTROPY   🔴 WORST KIND             🔴 DOUBLE PENALTY
                 Usually fast but           Slow AND unpredictable
                 randomly terrible          Users never know what to expect
                 "It worked fine yesterday"
```

**Report Output:**
```markdown
### 📐 Entropy Assessment

| Function | Buckets | Distribution | Entropy (H) | Verdict |
|----------|---------|-------------|-------------|---------|
| `getProductList` | 3 paths | Cache=70%, Normal=20%, N+1=10% | H=1.16 | 🟡 Variable — 10% of requests hit the slow N+1 path |
| `transformResponse` | 2 paths | Small=80%, Large=20% | H=0.72 | 🟡 Small variance, mostly predictable |
| `findSimilarProducts` | 2 paths | n<100=95%, n>1000=5% | H=0.29 | ✅ Stable at current data sizes |

### 🧭 Entropy-Latency Landscape

     Low Latency ←──→ High Latency
     ↑
Low  │  ✅ findSimilar    🟡 transformResponse
Ent  │  ✅ buildReport    🟡 processOrders
     │
     │  🔴 getProductList  🔴 getAllReviews
High │  (10% requests      (any popular product)
Ent  │   hit N+1 lottery)
     ↓
```

---

## Full Report Template

Combine all three phases and relevant advanced dimensions into one cohesive output:

```markdown
# 🔬 Performance Diagnosis Report
**Target:** [file/API/PR name]
**Analyzed at:** [timestamp]
**Methodology:** Budget → Causal → Persona + [N active dimensions]

---

## Phase 1: 🎯 Performance Budget

[Budget allocation table]

---

## Phase 2: 🕸️ Causal Bottleneck Analysis

[ASCII causal graph + bottleneck ranking + fix simulation]

---

## 🔒 Vulnerability Detection (if any found)

[If vulnerabilities detected: list each with location, fix, and perf/correctness classification.
Omit this section entirely if no vulnerabilities found.]

---

## Phase 3: 👤 Request Journey

[Persona journey: BEFORE and AFTER]

---

## 🧠 Meta-Cognitive Analysis

### 🧠 Likely Thought Process
[Reconstruct developer's reasoning for the key bottleneck — which step deviated]

### 🪤 Self-Praise Analysis
[Self-praise density + comparison with actual code quality]

### 💎 Surface-Deep Decoupling
[Surface score vs Deep score + gap interpretation]

### 🦠 Complexity Contagion (if applicable)
[Infection source → callers with defensive code]

### 🧟 Zombie Code Paths (if any)
[Intended behaviors unreachable due to code structure]

---

## Advanced Dimensions

### ⏳ Debt Interest Forecast
[TTI table for degradation-prone problems]
[If key bottleneck TTI < 12 months: add 2-3 sentence Failure Story here —
 what happens at current growth rate, first symptom, escalation timeline.]

### 👻 Ghost Load Audit
[Waste ratio and cost per source]

### 💣 Blast Radius Map
[Impact surface for each critical function]

### 🕳️ Time Leak Scan
[Gradual degradation patterns found]

### 📐 Entropy Assessment
[Entropy-latency landscape matrix]

---

## 🔧 Prioritized Recommendations
(Format: see Output Quality Rules 1, 1b, 2. Confidence: HIGH=mechanical/within20%, MEDIUM=varies/within50%, LOW=order-of-magnitude.)

| Priority | Problem | Location | Fix | Est. Impact | Confidence | Effort | Regression Risk | TTI |
|----------|---------|----------|-----|-------------|------------|--------|-----------------|-----|
| 🔴 P0    | ...     | file:line| [concrete code] | -XXXms | H/M/L | Xmin | 🟢🟡🔴 + mitigation | 1mo |

Regression: 🟢=monotonic improvement 🟡=changes execution order 🟡mitigation required 🔴=API contract change 🔴mitigation required

🔨 **PRIORITY ACTIONS:** (Format: see Rule 3)
⏱️ **EXECUTION PLAN:** (Format: see Rule 7)
📊 **MONITORING:** [Metric | Alert Threshold | Dashboard]
⚔️ **SELF-CHALLENGE:** (Format: see Rule 6)
📋 **FINAL SUMMARY:** 🔒 Vulnerabilities + ⚠️ Drawbacks + 🔗 Root Cause Convergence tables (see Pre-Output Checklist)
```

---

## Quick Reference

| Phase | Question Answered | Key Output |
|-------|-------------------|------------|
| 1. Budget | "How fast should each part be?" | Budget allocation table |
| 2. Causal | "What's the root cause that matters most?" | Bottleneck ranking + causal graph |
| 3. Narrative | "What does the request actually experience?" | Before/After journey story |
| ⏳ Debt | "When will this break in production?" | Time-to-Incident forecast |
| 👻 Ghost | "What work is done but never used?" | Waste ratio per data source |
| 💣 Blast | "Who feels it if this degrades?" | Impact surface map |
| 🕳️ Leak | "Does it get slower the longer it runs?" | Gradual degradation patterns |
| 📐 Entropy | "How predictable is the response time?" | Entropy-latency landscape |

| Signal | Meaning |
|--------|---------|
| One layer > 150% of budget | 🔴 Critical overrun — must fix |
| One layer 110-150% of budget | 🟡 Warning — requires optimization |
| Causal graph has node with out-degree ≥ 3 | 🔑 High-leverage bottleneck |
| Leverage score > 70 | Fix this first, regardless of severity |
| Health score < 60 | Systemic performance issue |
| FIXED layer over budget | May need infra/arch change (not just code) |
| TTI < 1 month | 🔴 Imminent incident — fix now |
| Ghost waste > 50ms/req | 🔴 Significant wasted computation |
| Blast Radius > 70 | 🔴 Company-level risk if this degrades |
| Time leak with no reset mechanism | 🔴 Will degrade until next restart |
| Entropy H > 1.5 (high latency) | 🔴 Worst user experience — slow + unpredictable |

---

## Anti-Pattern: Over-Engineering Traps

> **Full reference: `references/anti-patterns.md`** — 5 traps + Wrong-Tool patterns.

When a performance review finds problems, the temptation is to over-correct.
These are the most common traps and why they make performance WORSE.

| Trap | Symptom | Why It Hurts | When It IS Correct | Rule of Thumb |
|------|---------|-------------|-------------------|---------------|
| **Cache Everything** | "Just add Redis" | Cache invalidation complexity grows quadratically; cold starts become catastrophic; memory cost exceeds latency savings | Hot data with >100:1 read:write ratio, tolerable staleness, clear invalidation strategy | Add caching ONLY when a SPECIFIC query exceeds 2x budget AND caching is the cheapest fix |
| **Index Proliferation** | "Add indexes on every WHERE column" | Each index costs write throughput; query planner may choose wrong index; over-indexed tables degrade writes | Query is in critical path, column has high cardinality, write volume is moderate | Add at most ONE index per review cycle; measure impact; remove if ineffective |
| **Premature Micro-Optimization** | "Use bit shifts instead of multiplication" | Saves microseconds, costs readability; addresses symptoms not root causes; a missing index costs 1000x more | Function is measured bottleneck, >10K calls/sec, structural fix impossible | Never recommend micro-optimization as P0/P1; at P2, only if one-line change with zero readability cost |
| **Over-Parallelization** | "Promise.all everything" | Saturates connection pool; starves other requests; speedup bounded by slowest operation | Truly independent operations, ≤5 of them, spare capacity exists | Parallelize at most 3-5 operations; beyond that, use batch/aggregation endpoint |
| **Microservices Without Measurement** | "Split it into microservices" | Every service boundary adds 1-10ms network latency + serialization; monolith-to-microservice almost always INCREASES latency | Independent scaling requirements or organizational need, not performance | Never recommend architectural changes for performance before exhausting single-node optimizations |

### Wrong-Tool-for-Job Patterns

Beyond over-engineering for performance, there's a subtler anti-pattern: using a complex
mechanism to do a simple thing, introducing fragility.

| Pattern | Signature | Why Wrong | Correct Approach |
|---------|-----------|-----------|-----------------|
| **Generator as Cache** | `yield` in function body + caller uses `next()` + no loop | Generator yields once then exhausts; second call crashes with `StopIteration` | `hasattr`+attribute or `functools.cache` |
| **WeakKeyDictionary as thread_local** | `WeakKeyDictionary` + `threading.current_thread()` as key | Threads in pools never get GC'd → weakref never fires → degrades to plain dict; also skips cleanup | `threading.local()` attribute |
| **`id()` as Cache Key** | `id(var)` used as dict key for value-based caching | `id()` returns memory address, not value; CPython interns small ints (-5~256) but not larger ones — cache silently fails for uid>256 | Use the value itself as key: `cache[user_id]` |
| **Comment vs Code Contradiction** | See 🪤 Comment-Code Contradiction Audit in Phase 2 — already covered there |

### Trap Detection Rule

**Before finalizing the report, scan the recommendation list.** If ≥3 recommendations fall into any trap (Over-Engineering OR Wrong-Tool), **STOP and re-evaluate**.

---

## Fix Propagation — Pattern Similarity Matching

The most powerful optimization is the one that fixes not just THIS function,
but every function with the same pattern. After completing the diagnosis,
scan for propagation targets.

### Algorithm

```
For each P0/P1 finding in the current function:
  1. Extract the structural signature:
     - N+1: "for...of loop body contains await db.query() with loop variable"
     - Waterfall: "sequential await of independent async calls"
     - Ghost Load: "variable assigned from query result but never read downstream"
     - Missing Cache: "deterministic function of input with no memoization"
     - SELECT *: "SQL string literal containing 'SELECT *'"

  2. Scan the codebase (or the file's siblings/imports if codebase unavailable):
     - Grep for the structural signature
     - For each match, perform a 10-second triage: same root cause?

  3. Report propagation targets:
     | Pattern | Current Location | Propagation Target | Est. Savings | Confidence |
     |---------|-----------------|-------------------|--------------|------------|
     | N+1: items loop | getOrders:5-9 | getOrderDetail:12-15 | -45ms | HIGH |

  4. If ≥2 propagation targets found:
     → Add a "🔁 PATTERN PROPAGATION" section to the report
     → Rank targets by estimated savings
     → Note: "Fixing this pattern across all N locations saves [total]ms per request
        and eliminates [N] future incidents."
```

### When to Skip

- Single-file review with no imports or dependencies to scan
- The pattern is unique to the reviewed function (confirmed by scan)
- Quick mode (time constraint)

**Rule:** If Standard or Deep mode AND the finding is P0/P1, at minimum state
whether the pattern is likely to exist elsewhere based on the codebase structure.
Even "no propagation targets found after scan" is valuable — it confirms the
problem is localized.

---

## Multi-Entry-Point Files

When a file contains multiple independent functions (e.g., `getProductList`,
`processOrders`, `findSimilarProducts` all in one file):

1. **Pick the primary entry point** for Phase 1 budget — the function most
   likely on the critical user-facing path. Note which function you chose and why.
2. **List secondary entry points** with a one-line budget estimate each.
3. **Build ONE causal graph** that spans all entry points. Mark edges that
   cross between functions (e.g., N1 in `getProductList` causes N7 in `handleRequest`).
4. **Create ONE persona** for the primary path. If a secondary path is
   independently important, add a shorter persona for it.

This keeps the report focused while not ignoring other code in the file.

---

## Handling User Context

The diagnosis is based on static code analysis. The user may have runtime
knowledge that changes the picture. Always incorporate user context:

```
User says: "This N+1 query is intentional — we have a read replica
           and the product count never exceeds 20."

Your response:
  1. ACKNOWLEDGE: "Understood. With max 20 products and a read replica,
     the N+1 overhead is bounded to ~20 × 5ms = 100ms — acceptable."
  2. REVISE: Update the budget table and causal graph with the new info.
     Downgrade the finding's severity. Note the assumption.
  3. RECORD: "Noted: product count capped at 20. If this constraint
     ever changes, this becomes a 🔴 problem at 200+ products."
  4. MOVE ON: Don't argue. The user knows their system better than
     static analysis can infer.

If the user disagrees with a finding:
  - Ask ONE clarifying question to understand the operational context.
  - If the context explains it → downgrade and note the assumption.
  - If the context doesn't explain it → flag it as "User override —
    recommend revisiting if performance issues arise."
```

**Never** insist on a finding after the user provides reasonable context.
The skill's authority comes from insight quality, not stubbornness.

---

## Absolute Prohibitions — NEVER Do These

These are not guidelines. Violating any of them makes the diagnosis invalid.

| # | NEVER | Why | If You Catch Yourself Doing It |
|---|-------|-----|-------------------------------|
| 1 | **NEVER skip a phase** | Each phase feeds the next. A missing phase breaks the chain. | Delete the partial output. Start over from Phase 1. |
| 2 | **NEVER output a problem list without budget** | Without budget, priorities are just intuition. The budget IS the diagnosis. | Go back. Write the budget table first. |
| 3 | **NEVER present TTI as a precise number** | "8.3 months" from guessed growth rates is false precision. | Use bracket labels only: <1mo / 1-3mo / 3-12mo / >1yr. |
| 4 | **NEVER claim a health score without showing the penalty breakdown** | "Health Score: 72/100" with no derivation is a magic number. | Show the per-layer penalty calculation that produced the score. |
| 5 | **NEVER use a generic persona** | "A request" is not a persona. No name = no story = no emotional weight. | Name it. Give it a mission. State its cargo. |
| 6 | **NEVER apply a dimension without the report output table** | Mentioning "ghost load exists" without the waste table is not applying the dimension. | Produce the dimension's report table (waste ratio, BR score, TTI, H value). |
| 7 | **NEVER argue with user-provided operational context** | The user knows their system. Static analysis doesn't. | Acknowledge → revise → record the assumption → move on. |
| 8 | **NEVER skip the AFTER journey in Standard/Deep mode** | Before-only narrative is just complaining. The AFTER shows it's fixable. | Write the AFTER journey with projected numbers. |
| 9 | **NEVER output only problems without fixes** | Diagnosis without treatment is half the job. | Every 🔴/🟡 finding must have a concrete fix recommendation. |
| 10 | **NEVER exceed the output mode's scope without user consent** | Quick mode producing 5000 words betrays user trust. | Stick to the mode. If you think it needs Deep, ask the user. |

**Violating any of these = invalid diagnosis. Delete and restart from Phase 1.**

---

## Pre-Output Checklist — MANDATORY

**Before presenting ANY report, verify all items. If any item fails, fix it before output.**

### Structure Check
- [ ] Budget table present with FIXED/ELASTIC labels?
- [ ] Causal graph present (≥1 node OK if Clean Bill; ≥2 otherwise) with 🔑 KEY BOTTLENECK labeled?
- [ ] Persona journey has BOTH BEFORE and AFTER sections?
- [ ] At least 1 dimension applied (Quick) / 2-3 (Standard) / 5 (Deep)?

### Quality Check (Rules 1-7)
- [ ] **Rule 1 — Concrete Fixes:** Every P0/P1 fix includes runnable code in the same language as the reviewed code? (NOT "use a JOIN" — show the JOIN)
- [ ] **Rule 2 — Confidence Labels:** Every time estimate annotated with (HIGH/MEDIUM/LOW confidence)? Search for any bare number followed by "ms" without a confidence label.
- [ ] **Rule 3 — The One Fix:** `🔨 IF YOU ONLY FIX ONE THING` callout present after the recommendation table?
- [ ] **Rule 4 — Failure Story:** Embedded in ⏳ Debt Interest section? (2-3 sentences after TTI table for key bottleneck)
- [ ] **Rule 5 — Emotional Variety:** Scan the persona journey. Any emotional word used twice? Replace duplicates.
- [ ] **Rule 6 — Adversarial Self-Challenge:** ⚔️ section present for top 2 recommendations?
- [ ] **Rule 7 — Execution Plan:** ⏱️ table present with 5-min/1-hour/1-day tiers, cumulative health scores, and dependency notes?
- [ ] **Continuous Learning:** `references/vulnerability-library.md` consulted before diagnosis? New patterns contributed after diagnosis (if any found)?
- [ ] **Vulnerability Scan:** 8-type vulnerability detection executed? 🔒 section present if any found; explicitly omitted if none?
- [ ] **Final Summary:** 📋 section present with 🔒 Vulnerabilities + ⚠️ Drawbacks tables + 🔗 Root Cause Convergence statement? Every vulnerability and drawback listed with root cause?
- [ ] **Fix Propagation:** Pattern similarity scan attempted? Propagation targets reported (or explicitly stated as N/A)?
- [ ] **🔨 Priority Actions + 📊 Monitoring:** Priority Actions block present (The One Fix + Quick Wins)? Monitoring table with alert thresholds?

### Accuracy Check
- [ ] Any TTI presented as a precise number ("8.3 months")? → Replace with bracket label (<1mo / 1-3mo / 3-12mo / >1yr)
- [ ] Any health score without penalty breakdown? → Add per-layer penalty calculation
- [ ] Any absolute prohibition violated? (Check the 10-item NEVER list) → If yes, delete and restart

**If all boxes checked → present the report. If any box unchecked → fix it first.**
