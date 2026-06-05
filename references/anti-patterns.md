# Anti-Pattern Reference

> Over-engineering traps + wrong-tool patterns. Load on demand.

## Over-Engineering Traps

| Trap | Symptom | Why Hurts | When Correct | Rule |
|------|---------|-----------|-------------|------|
| Cache Everything | "Just add Redis" | Invalidation complexity; cold start catastrophe | >100:1 read:write; clear invalidation | Add cache ONLY when query >2x budget |
| Index Proliferation | "Index every WHERE column" | Write throughput cost; planner confusion | Critical path; high cardinality | One index per cycle; measure |
| Premature Micro-Opt | "Use bit shifts" | Saves µs, costs readability | Measured bottleneck; >10K calls/sec | Never P0/P1; P2 only if one-line |
| Over-Parallelization | "Promise.all everything" | Pool saturation; starves others | ≤5 independent ops; spare capacity | Max 3-5 parallel ops |
| Microservices Panic | "Split to microservices" | +1-10ms per boundary; almost always increases latency | Independent scaling NEED, not perf want | Exhaust single-node first |

**Trap Detection Rule:** If ≥3 recommendations hit any trap → STOP. Replace with simplest fix.

## Wrong-Tool-for-Job Patterns

| Pattern | Signature | Why Wrong | Correct |
|---------|-----------|-----------|---------|
| Generator as Cache | yield + next() + no loop | Yields once then exhausts; StopIteration on 2nd call | hasattr+attribute or functools.cache |
| WeakKeyDictionary as thread_local | WeakKeyDictionary + current_thread() as key | Threads in pools never GC'd; weakref never fires | threading.local() attribute |
| id() as Cache Key | id(var) as dict key for value caching | CPython interns -5~256 only; silently fails for larger | Use value itself as key |
| Comment vs Code | See 🪤 Audit in Phase 2 — already covered | | |
