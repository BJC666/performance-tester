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
  version: "1.0.0"
  author: "BJC666"
  tags: [performance, profiling, bottleneck, optimization, diagnosis, latency, budget, causal-graph, entropy]
---

# Performance Tester

## Overview

A performance diagnosis methodology that combines three core phases
with five advanced analytical dimensions — producing a report that no
traditional tool or simple AI prompt can replicate.

**Three Core Phases:**

1. **Budget Planning** — translate performance goals into layered time budgets
2. **Causal Reasoning** — build a directed graph of symptoms, identify the
   *leverage bottleneck* (fix one, collapse many)
3. **Persona Narrative** — tell the request's journey in first-person,
   making performance data emotionally tangible

**Five Advanced Dimensions:**

4. **⏳ Debt Interest Rate** — predict the *date* when today's acceptable
   slowness compounds into a production incident
5. **👻 Ghost Load Detection** — find computations whose results are
   *never consumed downstream* (not dead code — dead *data flow*)
6. **💣 Blast Radius** — map how far a function's degradation propagates
   through endpoints → features → users → revenue
7. **🕳️ Time Leak Detection** — identify gradual latency creep over
   process lifetime (the performance equivalent of memory leaks)
8. **📐 Performance Entropy** — apply Shannon entropy to response time
   distribution; quantify *unpredictability* as a first-class metric

These phases and dimensions feed each other: the budget tells you *where*
to look, the causal graph tells you *what matters most*, the dimensions
reveal *hidden risks and waste*, and the narrative tells the *story*
that drives action.

## Iron Law

**ALL THREE PHASES MUST RUN. AT LEAST ONE ADVANCED DIMENSION MUST BE APPLIED. NO EXCEPTIONS.**

Every performance review MUST produce:
1. A budget table (even if inferred from benchmarks)
2. A causal graph with leverage scores
3. A persona journey narrative
4. **At least one** of the five advanced dimensions (⏳👻💣🕳️📐)

Skipping any phase = incomplete diagnosis. This is not optional.
The phases feed each other. Budget without causal = numbers without insight.
Causal without narrative = insight without action.
Report without dimensions = missed hidden risks that a standard review would overlook.

**No exceptions:**
- Not for "simple files"
- Not for "obvious problems"
- Not for "quick checks"
- Not when the user "just wants a list"
- Not when the causal graph "seems unnecessary"
- Not when "none of the dimensions apply" (at least 2-3 always apply; pick the most relevant)

**Violating the letter of the three phases + dimensions is violating the spirit of this methodology.**

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
- General code review (use `superpowers:requesting-code-review`)
- Security audit (use `bork:review-security`)
- Simple linting or style checks
- When there is no code to analyze

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

```
Health Score = max(0, 100 - penalty)

penalty = Σ (overrun_ratio - 1.0) × layer_weight × 100

overrun_ratio = actual / budget
  (capped at 5.0 — beyond 5× overrun, the score is already zero)

layer_weight:
  - Critical path functions: 0.30
  - Database queries: 0.25
  - Serialization/I/O: 0.15
  - Secondary functions: 0.10
  - Logging/instrumentation: 0.05
  - Other: 0.05

Interpretation:
  90-100: Excellent. Within budget.
  75-89:  Good. Minor overruns, acceptable.
  60-74:  Fair. Notable problems, needs attention.
  40-59:  Poor. Significant overruns, fix this sprint.
  0-39:   Critical. Systemic failure, fix immediately.
```

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
| "I already know the answer, the phases are ceremony" | Pre-judgment is the enemy of diagnosis. Build the budget and graph anyway — you may discover something surprising. |

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

Step 4: Present as a budget table
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

Step 2: Causal Edge Construction
  For each pair of nodes (A, B), check if A can cause B:
    - TIMING: A is slow → B waits for A → B's latency increases
    - RESOURCE: A consumes resource → B starves → B fails or slows
    - PROPAGATION: A times out → caller retries → request volume ×N
    - FEEDBACK: A triggers degradation → degradation logic costs CPU → A gets slower

Step 3: Bottleneck Scoring
  For each node, compute LEVERAGE SCORE:
    Leverage = out_degree × avg_amplification_factor × (1 / fix_cost)

  out_degree: how many downstream nodes are affected
  amplification: how much does each edge amplify the problem (retry ×3, queue ×10)
  fix_cost: estimated effort to fix (1=trivial, 5=major refactor)

Step 4: Fix Simulation
  Select the highest-leverage node.
  Simulate its removal: which downstream nodes disappear?
  Estimate new end-to-end latency.
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
    😊 emotional state (🟢 happy 🟡 worried 🔴 suffering)

  For stages that exceed budget:
    Use emotional, first-person language.
    "This is where things went wrong."
    "I waited here for 480ms. I was only allowed 150ms."

Step 3: Tell the AFTER journey
  Same persona, but after the key bottleneck is fixed.
  Walk the same stages with projected numbers.
  Contrast: "Last time I spent 480ms here. Now: 42ms."

Step 4: End with a health score and emotional resolution
  Before: 🏥 62/100 "Penny is exhausted."
  After:  🏥 95/100 "Penny feels great."
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

## Advanced Diagnostic Dimensions

These five dimensions enrich the three-phase output with
concepts that have no equivalent in traditional performance tooling.
Each is applied automatically — the agent decides which dimensions
are relevant based on the code being analyzed.

**Dimension Selection Guide:**

| Dimension | ALWAYS Apply? | Strong Signal (must apply) | Weak/No Signal (may skip) |
|-----------|:--:|----------------------------|---------------------------|
| ⏳ Debt Interest | No | Any problem that scales with data/users; growth-dependent code paths | All problems are static (fixed-size input, no data growth) |
| 👻 Ghost Load | **Yes** | Any data fetching (SQL, API calls, file I/O) | Pure computation file with no external data sources |
| 💣 Blast Radius | **Yes** | Any function called by ≥2 consumers; shared utility code | Single-consumer leaf functions with no callers |
| 🕳️ Time Leak | No | Caches without eviction, event listeners, unbounded buffers, connection pools | Stateless pure functions; short-lived scripts/CLI tools |
| 📐 Entropy | No | Code with ≥2 distinct code paths (cache hit/miss, small/large input, success/error) | Single-path code with no branching; deterministic I/O |

**Minimum bar:** At least 2 dimensions apply to virtually any non-trivial code.
If genuinely stuck, default to 👻 Ghost Load + 💣 Blast Radius (they apply most universally).

---

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
   TTI = log(D / current_scale) / log(1 + r)
   Measured in months. Treat as ORDER-OF-MAGNITUDE estimate,
   not precise prediction. The formula assumes exponential growth;
   reality is messier. Round to nearest: <1mo / 1-3mo / 3-12mo / >1yr.

   **Caveat:** Growth rate `r` is a rough estimate from static analysis.
   State your assumptions explicitly. If the user provides actual growth
   data, prefer it over estimates. Never present TTI as a precise number
   (e.g., "8.3 months") — always use ranges or bracket labels.

   TTI < 1 month  → 🔴 IMMINENT: fix before next deploy
   TTI < 3 months → 🟡 WARNING: fix this quarter
   TTI < 12 months → 🟢 WATCH: add to roadmap
   TTI > 12 months → ⚪ FAR: monitor only
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
Step 1: Construct a response time histogram from code analysis
  (Not from monitoring data — from static reasoning about code paths)
  
  Bucket 1: < 50ms   (cache hit, simple path)
  Bucket 2: 50-100ms (normal path)
  Bucket 3: 100-200ms (complex path: more data, more joins)
  Bucket 4: 200-500ms (degraded: cache miss + N+1)
  Bucket 5: > 500ms  (worst case: cold start, full scan)

Step 2: Estimate probability of each bucket
  Based on: cache hit rates, data distribution, traffic patterns
  If unknown, assume uniform distribution as baseline

Step 3: Compute Shannon Entropy
  H = -Σ p(i) × log₂(p(i))
  
  Max entropy (uniform 5 buckets): H = log₂(5) = 2.32
  Min entropy (one bucket only): H = 0

Step 4: Interpret
  H < 0.5:  STABLE — response time is tightly clustered. Users get consistent UX.
  0.5 ≤ H < 1.5: VARIABLE — noticeable inconsistency. Some users are unhappy.
  H ≥ 1.5: CHAOTIC — wild swings. User experience is a lottery. Trust erodes.
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

## Phase 3: 👤 Request Journey

[Persona journey: BEFORE and AFTER]

---

## Advanced Dimensions

### ⏳ Debt Interest Forecast
[TTI table for degradation-prone problems]

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

| Priority | Problem | Location | Fix | Est. Impact | TTI |
|----------|---------|----------|-----|-------------|-----|
| 🔴 P0    | ...     | file:line| ... | -XXXms      | 1mo |
| 🟡 P1    | ...     | file:line| ... | -XXms       | 8mo |
| 🟢 P2    | ...     | file:line| ... | -Xms        | >1yr |
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
| One layer 110-150% of budget | 🟡 Warning — should optimize |
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

## Common Mistakes

| Mistake | Correction |
|---------|------------|
| Skipping Phase 1 and just listing problems | Without a budget, you can't tell which problems actually matter |
| Treating symptoms as independent | Always check: does A cause B? Build the graph before ranking |
| Using generic, impersonal language in Phase 3 | The persona must be specific: name, mission, cargo. Generic = forgettable |
| Setting unrealistic budgets | Use industry benchmarks. 50ms for a complex DB query is not realistic |
| Fixing by severity alone, ignoring leverage | A 🟡 warning with leverage score 90 matters more than a 🔴 with leverage 10 |
| Forgetting FIXED vs ELASTIC | Don't recommend "optimize DNS" — that's a fixed cost. Focus on elastic layers |
| Over-allocating budget (sum > target) | Every layer's budget must sum to exactly the top-level target |

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

## Dimension–Causal Graph Integration

The five dimensions are not standalone — they enrich the Phase 2 causal graph:

- **⏳ Debt Interest:** adds a TIME axis to causal edges. An edge labeled "TIMING" today may become "RESOURCE" tomorrow as the debt compounds.
- **👻 Ghost Load:** may reveal HIDDEN NODES — computations not visible as "problems" because they're not slow, but they're wasteful. Add ghost nodes to the graph with dotted edges.
- **💣 Blast Radius:** directly informs `out_degree` in the leverage score. A node with BR=85 should have its out_degree multiplied by 1.5× in the leverage formula.
- **🕳️ Time Leak:** adds FEEDBACK loops to the graph. A time leak is a self-reinforcing cycle: slow → accumulate state → slower.
- **📐 Entropy:** labels each node with its H value. High-entropy nodes in the causal graph are "unstable variables" — their behavior is unpredictable, making the entire graph's predictions less reliable.

**Rule:** If a dimension finding changes a node's leverage score by ≥15 points, redraw the affected portion of the causal graph.

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

## Red Flags — STOP and Re-evaluate

- "I'll just list all the performance issues I see" → You're skipping budget and causal analysis. Go back.
- "The causal graph is too complex to draw" → Limit to top 5-7 nodes. Simplify, don't skip.
- "I'll skip the persona, it's not technical" → The persona is the most actionable part. Don't skip it.
- "The budget doesn't matter, just fix everything" → Without budget, you have no success criteria. Use defaults.
- "I can tell the problem without building the graph" → Single-point analysis misses cascading failures.
- "This code is too simple for a full report" → Simple code = simple three phases. Still run them all.
- "The user didn't give a target number" → Use the Default Performance Targets table. Never skip Phase 1.
- "There's only one problem, no need for a graph" → One node + one edge = still a graph. Draw it.
- "This is a quick pass, let me be efficient" → Efficiency is following the methodology, not skipping it. The three phases ARE the efficient path.
- "The problems are all independent, causal graph is pointless" → If truly independent, the graph proves it. Drawing it IS the verification. Don't assume independence.

**All of these mean: Go back. Run the full three-phase pipeline. No phase is optional.**
