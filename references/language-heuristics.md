# Language-Specific Heuristics

> **Full library: `language-patterns.md`** (100 patterns across Python/JS/Java/Go/C++).
> This file tracks per-language diagnostic experience. Append after each cross-language session.

## Coverage

| Language | Patterns | Sessions | Last |
|----------|:--:|:--:|------|
| Python | 25 | 9 | 2026-06-05 |
| JavaScript/TypeScript | 25 | 0 | — |
| Java | 20 | 1 | 2026-06-05 |
| Go | 15 | 1 | 2026-06-05 |
| C++ | 15 | 1 | 2026-06-05 |

## Detection Priority by Language

When analyzing unfamiliar code, prioritize:

1. **Concurrency patterns** (thread safety, goroutine/channel usage, async/await)
2. **Memory/ownership patterns** (GC pressure, leaks, smart pointer misuse)
3. **Error handling gaps** (swallowed errors, over-broad catch, missing finally)
4. **Library-specific footguns** (JWT decode, ORM lazy-load, regex backtracking)
5. **Performance anti-patterns** (N+1, per-request connection, sync I/O in hot path)
