# Advanced Diagnostic Dimensions — Full Algorithms & Templates

> Loaded on demand during Phase 6 (Dimensions). Contains heavy algorithm details
> and report output templates for all 5 dimensions + supporting detection tables.

## Budget Table Template

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
| Client rendering        | XXms    | ELASTIC | Frontend         |
| Third-party             | XXms    | ELASTIC | External         |
| **TOTAL**              | **XXms**|         |                  |
```

## Causal Graph Notation

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

## Persona Journey Template

```markdown
╔══════════════════════════════════════════════════╗
║  👤 Request Persona                             ║
║     Name: [name]   Mission: [what it does]      ║
║     Budget: [total]ms   Cargo: [data summary]   ║
╚══════════════════════════════════════════════════╝

┌─ [Name]'s Journey (BEFORE) ─────────────────────┐
│  [emoji] [timestamp]  [stage description]         │
│            ⏱ [actual]ms  💰 budget [budget]ms    │
│            [emotional note if over budget]        │
│  ═══════════════════════════════════════════      │
│  📊 Total: [sum]ms  🎯 Budget: [target]ms         │
│  🏥 Health Score: [0-100]  😊 [emotional note]    │
│  🔑 Key Bottleneck: [name]                        │
└───────────────────────────────────────────────────┘

┌─ [Name]'s Journey (AFTER) ──────────────────────┐
│  [same structure with projected numbers]          │
└───────────────────────────────────────────────────┘
```

---

## Dimension Selection Guide

| Dimension | Almost always? | Strong Signal | Weak/No Signal |
|-----------|:--:|---------------|----------------|
| ⏳ Debt Interest | No | Any problem that scales with data/users | All problems are static |
| 👻 Ghost Load | Almost always | Any data fetching (SQL, API calls, file I/O) | Pure computation only |
| 💣 Blast Radius | Almost always | Any function called by ≥2 consumers | Single-consumer leaf functions |
| 🕳️ Time Leak | No | Caches without eviction, listeners, unbounded buffers | Stateless pure functions |
| 📐 Entropy | No | ≥2 distinct code paths | Single-path deterministic code |

**MVD (Minimum Viable Dimension):** There is no "genuinely stuck."

| Dimension | Minimum Application |
|-----------|-------------------|
| 👻 Ghost Load | List every data source. Even "all used" IS the analysis. |
| 💣 Blast Radius | Count callers. Even "one caller" IS the analysis. |
| ⏳ Debt Interest | State whether it scales. Even "static" IS the analysis. |
| 🕳️ Time Leak | Check for caches/queues/buffers. Even "none found" IS the analysis. |
| 📐 Entropy | Count code paths. Even "one path, H=0" IS the analysis. |

---

## Dimension 1: ⏳ Performance Debt Interest Rate

**Concept:** Performance problems compound as data/users grow.

**Algorithm:**
1. Growth Driver: user growth (linear), data growth (linear/quadratic), code path growth (step), time (logarithmic — good)
2. Compound Rate (r): estimated monthly growth rate. Example: 15%/month → r=0.15
3. Danger Threshold (D): point where problem causes incident (pool exhaust, OOM, timeout)
4. TTI: Estimate months to danger threshold. Doubling heuristic: at r%/month, doubling time ≈ 70/r. Always bracket label: <1mo / 1-3mo / 3-12mo / >1yr. State growth assumptions.

**Report Output:**
```markdown
### ⏳ Performance Debt Forecast
| Problem | Now | r (monthly) | Danger at | TTI | Alert |
|---------|------|-------------|-----------|-----|-------|
```

---

## Dimension 2: 👻 Ghost Load Detection

**Concept:** Computations whose results are never consumed. Not dead code — dead data flow.

**Detection Patterns:**

| Pattern | Signature | Example |
|---------|-----------|---------|
| SELECT * discard | N columns queried, K < N accessed | Only `name` and `price` used from full row |
| Overfetch cascade | API returns 50-field JSON, frontend uses 4 | REST full objects → sparse consumption |
| Join bloat | JOIN brings data never referenced downstream | LEFT JOIN for denormalized field |
| Compute-and-discard | Value computed then passed through unused | `JSON.stringify(x).length` |
| Preload waste | Eager-loaded associations never accessed | `.includes(:reviews, :images)` but only reviews rendered |
| Map-then-filter | `.map().filter()` where `.filter().map()` is cheaper | Map 1000 then filter to 10 |

**Algorithm:** Trace downstream consumers. WASTE RATIO = unused / total. WASTE COST = ratio × call_cost × frequency. Waste Ratio > 0.5 AND Waste Cost > 10ms → 🔴. Waste Ratio > 0.3 → 🟡.

**Report Output:**
```markdown
### 👻 Ghost Load Audit
| Source | Total Fields | Used | Wasted | Waste Cost/Req | Waste/Day (10K req) |
|--------|-------------|------|--------|---------------|---------------------|
```

---

## Dimension 3: 💣 Performance Blast Radius

**Concept:** When a function degrades, who feels it? Security blast radius applied to performance.

**Algorithm:**
1. Direct Impact: which endpoints/features/users?
2. Cascading Impact: what times out? what retry storms?
3. Business Mapping: Function → Endpoint → Feature → User → Revenue
4. BR Score = (affected_endpoints/total × 40) + (affected_segments/total × 30) + (revenue_critical ? 30 : 0)
   When unknowns: estimate from function role. Utility used by 3+ features → assume ≥0.5. Leaf → ≤0.1.
   BR > 70: 🔴 WIDE. BR > 40: 🟡 MODERATE. BR < 40: 🟢 CONTAINED.

**Report Output:**
```markdown
### 💣 Blast Radius Map
| Function | Called By | Affected Features | Users | Revenue Impact | BR Score |
|----------|-----------|-------------------|-------|---------------|----------|
```

---

## Dimension 4: 🕳️ Time Leak Detection

**Concept:** Latency grows gradually over process lifetime. Like memory leaks, but for time.

**Detection Patterns:**

| Pattern | Mechanism | Signature |
|---------|-----------|-----------|
| Cache pollution | Cache fills with low-value entries | No TTL, no LRU eviction |
| Connection senescence | Long-lived connections degrade | No maxLifetime/idleTimeout |
| Listener accumulation | Listeners added, never removed | addEventListener without remove |
| Closure bloat | Large objects in closures | Callback captures entire request context |
| Iterator creep | Streaming results materialized | .toArray() on streaming query |
| Log buffer saturation | In-memory log grows unbounded | Unbounded array before flush |

**Algorithm:** Assess ACCUMULATION RATE (per-request? per-interval?). Check RESET MECHANISM (natural TTL? manual cleanup? none?). OBSERVABILITY GAP: P95 over 24h vs first hour. Delta >20% + no reset → CRITICAL.

**Report Output:**
```markdown
### 🕳️ Time Leak Scan
| Pattern | Location | Accumulation | Reset | 24h Degradation Est. | Verdict |
|---------|----------|-------------|-------|---------------------|---------|
```

---

## Dimension 5: 📐 Performance Entropy

**Concept:** Unpredictable slowness is worse than consistent slowness. Shannon entropy applied to response time distribution.

**Algorithm:**
1. Count distinct code paths (cache hit/miss, small/large input, success/error)
2. For each path, estimate best-case vs worst-case latency from code structure
3. If any path ≥3× best-case → HIGH entropy (CHAOTIC). If all within 2× → LOW (STABLE). Otherwise → VARIABLE.
4. Place on matrix:

```
                    LOW LATENCY              HIGH LATENCY
  LOW ENTROPY    ✅ IDEAL                  🟡 Consistently slow
  HIGH ENTROPY   🔴 Usually fast but       🔴 Slow AND unpredictable
                 randomly terrible
```

**Report Output:**
```markdown
### 📐 Entropy Assessment
| Function | Paths | Distribution | Entropy | Verdict |
|----------|-------|-------------|---------|---------|

### 🧭 Entropy-Latency Landscape
[2×2 matrix with functions placed]
```

---

## Vulnerability Detection Table

| Vulnerability | Detection Signature | Impact Under Load |
|------|------|------|
| Read-Modify-Write Race | SELECT → arithmetic → UPDATE; no FOR UPDATE/atomic | Lost updates → retries multiply |
| SQL Injection | String concat building SQL; no parameterization | Plan cache disabled; DB CPU waste |
| Connection/Resource Leak | connect()/open() without try/finally or context manager | Pool exhaustion → cascade |
| Unbounded Input | No LIMIT/max/bounds on user-controlled size | OOM / infinite loop |
| Missing Timeout | Network/DB call without timeout parameter | Thread pool exhaustion |
| Regex DoS (ReDoS) | Nested quantifiers; untrusted input fed to regex | 100% CPU on crafted input |
| Deadlock Potential | Multiple locks acquired in different orders | System freeze |
| TOCTOU | if exists(SELECT) then UPDATE; exists() then open() | Race window widens under concurrency |

## Comment-Code Contradiction Audit

| Comment Claims | Code Does | Flag |
|---------------|----------|:--:|
| "环境变量加载/from env" | Literal assignment | 🔴 |
| "规范写法/生产规范" | Contains known anti-pattern | 🔴 |
| "安全校验/安全检查" | Missing validation logic | 🔴 |
| "无XX漏洞" | That vulnerability present | 🔴 |
| "绝对安全/完全正确" | Any flaw exists | 🔴 |

## 🪤 Self-Praise Density Scan

Keywords (CN): 安全 规范 标准 正确 绝对 生产级 无漏洞 最佳实践 完全
Keywords (EN): secure safe correct proper best-practice production foolproof bulletproof

Density = claims / lines. >0.05: flag 🪤. >0.10: 🪤🪤 over-compensating.

## 🧟 Zombie Code Path Detection

| Type | Signature |
|------|-----------|
| Overshadowed catch | except SpecificError AFTER except Exception |
| Impossible elif | Condition subset of prior branch |
| Unreachable callback | Callback registered, trigger never fires |
| Dead default | All cases covered, default never reaches |
