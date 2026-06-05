# Language-Specific Vulnerability & Anti-Pattern Library

> Compact format: Signature + Fix + Severity. Append after each cross-language session.

---

## Python (25 patterns)

### Concurrency
**PY-001: Class variable as shared state**
**Sig:** `cls.attr` or `ClassName.attr = None` at class level, mutated in methods
**Fix:** Instance variable (`self.attr`) or `threading.local()`
**Sev:** 🔴 Cross-request data leak

**PY-002: Generator as one-shot cache**
**Sig:** `yield x; next(cached_gen)` — yield without loop, called more than once
**Fix:** `hasattr` + attribute, or `functools.lru_cache`
**Sev:** 🔴 StopIteration crash on 2nd call

**PY-003: `__del__` + shared mutable state**
**Sig:** `def __del__(self): global_dict[self.key] = ...` — no lock, no existence check
**Fix:** Never mutate globals in `__del__`. Use `weakref.finalize` or explicit cleanup.
**Sev:** 🔴 GC race, interpreter-shutdown crash

**PY-004: Thread-local not cleaned up**
**Sig:** `thread_local.attr = value` in handler, `cleanup()` only deletes some attrs
**Fix:** Mirror setup/cleanup. Every `setattr` needs matching `delattr`.
**Sev:** 🟡 Stale state across requests

### Database
**PY-005: `fetchone()["col"]` null crash**
**Sig:** `cursor.fetchone()["col"]` or `.fetchone()["col"] or 0`
**Fix:** `row = cursor.fetchone(); return row["col"] if row else default`
**Sev:** 🟡 TypeError on empty result

**PY-006: Read-Modify-Write race**
**Sig:** `SELECT col → var = row[0]; var += delta; UPDATE col = var`
**Fix:** `UPDATE SET col = col + ? WHERE id = ?`
**Sev:** 🔴 Lost updates under concurrency

**PY-007: Per-request `sqlite3.connect()`**
**Sig:** `conn = sqlite3.connect("db")` inside function body, no pool
**Fix:** Module-level connection or context manager
**Sev:** 🟡 15ms wasted per call

**PY-008: Multi-step writes without transaction**
**Sig:** Multiple independent `conn.commit()` across separate connections
**Fix:** Single shared `conn` + `BEGIN → ops → COMMIT|ROLLBACK`
**Sev:** 🔴 Partial writes on failure

### Security
**PY-009: JWT `decode()` without `algorithms`**
**Sig:** `jwt.decode(token, SECRET)` — missing `algorithms=["HS256"]`
**Fix:** `jwt.decode(token, SECRET, algorithms=["HS256"])`
**Sev:** 🔴 `alg:none` auth bypass

**PY-010: Hardcoded secret with misleading comment**
**Sig:** `SECRET = "literal"` alongside `# loaded from env`
**Fix:** `SECRET = os.environ["KEY"]`
**Sev:** 🔴 Key in git history

**PY-011: `except Exception` swallowing specific errors**
**Sig:** `try: ... except Exception: return None`
**Fix:** Catch specific types. Log security events.
**Sev:** 🟡 Auth bypass via swallowed exceptions

**PY-012: `hashlib.md5()` for security tokens**
**Sig:** `md5(raw.encode()).hexdigest()` for auth/crypto
**Fix:** `hashlib.sha256()` or `secrets.token_hex()`
**Sev:** 🔴 Collision-vulnerable hash for security

**PY-013: `pickle.loads()` on untrusted input**
**Sig:** `pickle.loads(user_input)` or `pickle.load(request.body)`
**Fix:** JSON or protobuf; never unpickle untrusted data
**Sev:** 🔴 Remote code execution

**PY-014: `os.system()` / `subprocess(shell=True)` with user input**
**Sig:** `os.system(f"cmd {user_input}")` or `shell=True`
**Fix:** `subprocess.run([cmd, arg], shell=False)`
**Sev:** 🔴 Command injection

### Performance
**PY-015: `time.time()` in loop**
**Sig:** `for i in range(N): t = time.time()` — syscall per iteration
**Fix:** Call once outside loop
**Sev:** 🟢 Minor alone, significant at scale

**PY-016: `+` string concat in loop**
**Sig:** `result += item` in `for` loop; O(n²) allocations
**Fix:** `"".join(list)` or list comprehension
**Sev:** 🟡 O(n²) string allocation

**PY-017: List comprehension building then discarding**
**Sig:** `[x for x in data]` — creates list when generator would suffice
**Fix:** `(x for x in data)` — generator expression
**Sev:** 🟢 Memory for large datasets

**PY-018: `id()` as cache/dict key**
**Sig:** `cache[id(obj)] = value` — memory address, not value
**Fix:** Use value directly or `hash(obj)`
**Sev:** 🔴 CPython interns -5~256 only; silently fails for larger

### Design
**PY-019: Mutable default argument**
**Sig:** `def fn(arg, cache={})` — shared across all calls
**Fix:** `def fn(arg, cache=None): if cache is None: cache = {}`
**Sev:** 🟡 State leaks between calls

**PY-020: `is` instead of `==` for value comparison**
**Sig:** `if x is 1000:` — identity, not equality
**Fix:** `if x == 1000:`
**Sev:** 🟢 CPython interns -5~256; >256 silently fails

**PY-021: `import *` in production**
**Sig:** `from module import *` — pollutes namespace
**Fix:** Explicit imports
**Sev:** 🟢 Name collision risk

**PY-022: `eval()` / `exec()` on user input**
**Sig:** `eval(user_input)` or `exec(f"x = {user_input}")`
**Fix:** `ast.literal_eval()` or avoid entirely
**Sev:** 🔴 Remote code execution

**PY-023: `open()` without context manager**
**Sig:** `f = open("file"); ... f.close()` — leak on exception
**Fix:** `with open("file") as f:`
**Sev:** 🟡 File handle leak

**PY-024: `__init__` doing I/O or heavy work**
**Sig:** DB query, network call, or file read in `__init__`
**Fix:** Lazy init or factory method
**Sev:** 🟢 Import-time / test-speed penalty

**PY-025: `assert` for production validation**
**Sig:** `assert user.is_admin` — removed with `-O` flag
**Fix:** `if not user.is_admin: raise PermissionError`
**Sev:** 🟡 Security check silently disabled in optimized mode

---

## JavaScript / TypeScript (25 patterns)

### Async & Concurrency
**JS-001: Sequential `await` of independent calls**
**Sig:** `const a = await f1(); const b = await f2()` — f1,f2 independent
**Fix:** `const [a, b] = await Promise.all([f1(), f2()])`
**Sev:** 🟡 N× latency multiplier

**JS-002: `forEach` + `await` — silently broken**
**Sig:** `array.forEach(async (item) => { await db.query(...) })` — loop doesn't wait
**Fix:** `for (const item of array) { await db.query(...) }` or `Promise.all(array.map(...))`
**Sev:** 🔴 Promises fire-and-forget, loop completes before queries

**JS-003: Missing `AbortController` for fetch**
**Sig:** `useEffect(() => { fetch(url).then(setData) }, [])` — no abort
**Fix:** `const ctrl = new AbortController(); fetch(url, {signal: ctrl.signal}); return () => ctrl.abort()`
**Sev:** 🟡 Stale response race on unmount

**JS-004: `Promise.all` without error handling**
**Sig:** `await Promise.all(promises)` — one rejection rejects all, others lost
**Fix:** `Promise.allSettled(promises)` or individual try-catch
**Sev:** 🟡 Silent partial failures

### React-Specific
**JS-005: `key={index}` in lists**
**Sig:** `.map((item, i) => <Item key={i} />)` — index as key
**Fix:** `key={item.id}` — stable unique identifier
**Sev:** 🟡 DOM thrash on reorder/filter

**JS-006: Inline object/function in JSX**
**Sig:** `style={{color: 'red'}}` or `onClick={() => fn(x)}` — new ref every render
**Fix:** Module-level constant or `useCallback`
**Sev:** 🟡 Breaks `React.memo`, forces re-render

**JS-007: `useEffect` missing dependencies**
**Sig:** `useEffect(() => { ... }, [])` — stale closure over props/state
**Fix:** Include all used variables in dep array; or `useRef` for intentional omission
**Sev:** 🟡 Stale data / missed updates

**JS-008: State update on unmounted component**
**Sig:** `fetch().then(data => setState(data))` — no unmount check
**Fix:** AbortController (JS-003) or `useRef(isMounted)` flag
**Sev:** 🟡 React warning + memory leak in strict mode

**JS-009: Uncontrolled → controlled input switch**
**Sig:** Input initially `undefined` value, later set to string
**Fix:** Always use `""` or controlled value; never switch modes
**Sev:** 🟢 React warning

### Security
**JS-010: `dangerouslySetInnerHTML` with user data**
**Sig:** `dangerouslySetInnerHTML={{__html: userInput}}`
**Fix:** Sanitize with DOMPurify; prefer text content
**Sev:** 🔴 XSS

**JS-011: `eval()` or `new Function()` with user input**
**Sig:** `eval(userInput)` / `new Function(userInput)`
**Fix:** Never evaluate user input as code
**Sev:** 🔴 Remote code execution

**JS-012: `jwt.decode()` without algorithm (jsonwebtoken)**
**Sig:** `jwt.verify(token, secret)` — no `algorithms` option
**Fix:** `jwt.verify(token, secret, {algorithms: ['HS256']})`
**Sev:** 🔴 `alg:none` bypass

**JS-013: `JSON.parse()` without try-catch**
**Sig:** `JSON.parse(rawInput)` — uncaught SyntaxError crashes
**Fix:** `try { JSON.parse(raw) } catch { /* handle */ }`
**Sev:** 🟡 Crash on malformed input

**JS-014: API key / secret in frontend code**
**Sig:** `const API_KEY = "sk-..."` — in browser bundle
**Fix:** Never in frontend. Proxy through backend.
**Sev:** 🔴 Key exposed to every user

### Performance
**JS-015: `console.log` in production hot path**
**Sig:** `console.log(JSON.stringify(largeObj))` — sync I/O + serialization
**Fix:** Guard with env check or use structured logger with sampling
**Sev:** 🟡 Blocking I/O per request

**JS-016: Synchronous `localStorage` in hot path**
**Sig:** `localStorage.getItem()` during render — sync disk I/O
**Fix:** Cache in memory; write to localStorage on idle
**Sev:** 🟡 Blocks main thread

**JS-017: `debounce` missing on search input**
**Sig:** `<input onChange={e => fetchResults(e.target.value)} />` — fetch per keystroke
**Fix:** `useDeferredValue` or 300ms debounce
**Sev:** 🟡 Request waterfall on typing

**JS-018: Large bundle: `import *` or barrel imports**
**Sig:** `import { Button } from '@mui/material'` — tree-shaking defeated
**Fix:** `import Button from '@mui/material/Button'` — path imports
**Sev:** 🟢 Bundle bloat

### Memory
**JS-019: Event listener not removed**
**Sig:** `window.addEventListener('scroll', handler)` — no cleanup
**Fix:** `useEffect(() => { window.addEventListener(...); return () => window.removeEventListener(...) })`
**Sev:** 🟡 Memory leak + ghost handlers

**JS-020: Closure capturing large scope**
**Sig:** Callback closes over entire `request` object instead of specific fields
**Fix:** Extract needed fields before closure: `const {id} = req; setTimeout(() => use(id))`
**Sev:** 🟡 GC can't collect large objects

**JS-021: `setInterval` without cleanup**
**Sig:** `setInterval(fn, 1000)` — runs forever after unmount
**Fix:** `const id = setInterval(...); return () => clearInterval(id)`
**Sev:** 🟡 Memory leak + wasted CPU

### Design
**JS-022: `==` instead of `===`**
**Sig:** `if (x == null)` — type coercion
**Fix:** `if (x === null || x === undefined)` or `if (x == null)` for null+undefined only
**Sev:** 🟢 Unexpected truthiness

**JS-023: `parseInt` without radix**
**Sig:** `parseInt("08")` — octal interpretation in older engines
**Fix:** `parseInt(str, 10)` or `Number(str)`
**Sev:** 🟢 Edge case in legacy

**JS-024: `typeof null === "object"` pitfall**
**Sig:** `if (typeof x === "object")` — matches null
**Fix:** `if (x !== null && typeof x === "object")`
**Sev:** 🟢 Null incorrectly passes object check

**JS-025: Prototype pollution via `Object.assign`**
**Sig:** `Object.assign({}, userInput)` — `__proto__` injectable
**Fix:** `Object.create(null)` or sanitize keys; use `Map`
**Sev:** 🟡 Prototype chain manipulation

---

## Java (20 patterns)

### Concurrency
**JV-001: `InheritableThreadLocal` + thread pool**
**Sig:** `InheritableThreadLocal` + `Executors.newFixedThreadPool()` — inheritance only on `new Thread()`
**Fix:** Plain `ThreadLocal` + explicit context capture in `submit()`
**Sev:** 🔴 Stale/wrong user context in pool threads

**JV-002: `HashMap` in multi-threaded context**
**Sig:** `HashMap` shared across threads without synchronization
**Fix:** `ConcurrentHashMap` or `Collections.synchronizedMap()`
**Sev:** 🔴 Infinite loop / data corruption under concurrency

**JV-003: Double-checked locking on non-volatile field**
**Sig:** `if (instance == null) { synchronized { if (instance == null) instance = new X() } }` — no `volatile`
**Fix:** `volatile` on field, or holder class idiom, or enum singleton
**Sev:** 🟡 Partially-constructed object visible to other threads

**JV-004: `synchronized` on method + DB call inside**
**Sig:** `synchronized void process() { db.query(...) }` — lock held across I/O
**Fix:** Narrow synchronized block to only the critical section
**Sev:** 🟡 Lock contention under load

### Memory
**JV-005: `ThreadLocal` not removed in pooled threads**
**Sig:** `ThreadLocal.set()` in request handler, no `remove()` in finally
**Fix:** `try { ... } finally { threadLocal.remove(); }`
**Sev:** 🔴 Memory leak + stale data in thread pool

**JV-006: `String` concatenation in loop**
**Sig:** `result += item` in for-loop — O(n²) String allocations
**Fix:** `StringBuilder` or `String.join()`
**Sev:** 🟡 GC pressure at scale

**JV-007: Autoboxing in hot loop**
**Sig:** `Integer sum = 0; for (int i: list) sum += i` — boxing per iteration
**Fix:** Use primitive `int sum = 0`
**Sev:** 🟢 GC churn at high throughput

### Security
**JV-008: SQL string concatenation (JDBC PreparedStatement not used)**
**Sig:** `stmt.executeQuery("SELECT * FROM users WHERE id=" + userId)`
**Fix:** `PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id=?"); ps.setInt(1, userId)`
**Sev:** 🔴 SQL injection

**JV-009: `equals()` without `hashCode()` override**
**Sig:** Custom `equals()` but no `hashCode()` — HashMap/HashSet broken
**Fix:** Always override both together
**Sev:** 🟡 HashMap silently returns null for existing keys

**JV-010: `==` for String comparison**
**Sig:** `if (str1 == str2)` — reference equality, not value
**Fix:** `if (str1.equals(str2))`
**Sev:** 🟢 Intermittent failures

### Resource
**JV-011: Resource not closed (Pre-try-with-resources)**
**Sig:** `conn = DriverManager.getConnection(...)` — no finally-close
**Fix:** `try (Connection conn = ...) { }` — try-with-resources
**Sev:** 🟡 Connection leak

**JV-012: `Stream` without terminal operation**
**Sig:** `list.stream().filter(x -> x > 0)` — no `.collect()` / `.forEach()`
**Fix:** Always terminate: `.collect(Collectors.toList())`
**Sev:** 🟢 Filter never executes (lazy)

### Performance
**JV-013: `ArrayList` with no initial capacity**
**Sig:** `new ArrayList<>()` — 10→15→22→33 reallocations
**Fix:** `new ArrayList<>(expectedSize)` — pre-size
**Sev:** 🟢 Array copy overhead

**JV-014: `String.format()` or `MessageFormat` in hot path**
**Sig:** `String.format("user %d", id)` — regex + parsing per call
**Fix:** Simple `"user " + id` or `StringBuilder`
**Sev:** 🟢 Format parsing overhead

**JV-015: `System.out.println()` in production**
**Sig:** `System.out.println(...)` in request handler — sync I/O
**Fix:** SLF4J/Logback with async appender
**Sev:** 🟡 Blocks request thread

### Design
**JV-016: Static `SimpleDateFormat` (not thread-safe)**
**Sig:** `private static final DateFormat fmt = new SimpleDateFormat(...)` — shared mutable
**Fix:** `DateTimeFormatter` (thread-safe) or `ThreadLocal<DateFormat>`
**Sev:** 🔴 Data corruption under concurrency

**JV-017: `catch (Exception e) { e.printStackTrace(); }` — swallowed**
**Sig:** Exception caught, printed, then execution continues as if nothing happened
**Fix:** Log and rethrow, or handle specifically
**Sev:** 🟡 Silent failure, inconsistent state

**JV-018: `Optional.get()` without `isPresent()`**
**Sig:** `optional.get()` — NoSuchElementException if empty
**Fix:** `optional.orElse(default)` or `optional.ifPresent(consumer)`
**Sev:** 🟡 Crash on absent value

**JV-019: Raw type usage**
**Sig:** `List list = new ArrayList()` — no generics
**Fix:** `List<String> list = new ArrayList<>()`
**Sev:** 🟢 ClassCastException at runtime

**JV-020: `static` field injection (`@Autowired` on static)**
**Sig:** `@Autowired private static Service svc` — Spring can't inject statics
**Fix:** Instance field or `@PostConstruct` setter
**Sev:** 🟡 NullPointerException at runtime

---

## Go (15 patterns)

### Concurrency
**GO-001: Goroutine leak — no cancellation**
**Sig:** `go func() { ch <- result }()` — no select with context/done channel
**Fix:** `select { case ch <- result: case <-ctx.Done(): return }`
**Sev:** 🔴 Goroutine leak under timeout/cancellation

**GO-002: `sync.Mutex` copy**
**Sig:** `func (s Service) Process() { s.mu.Lock() }` — value receiver copies mutex
**Fix:** Pointer receiver: `func (s *Service) Process()`
**Sev:** 🔴 Each call locks a different mutex copy — no actual synchronization

**GO-003: Closing channel from receiver side**
**Sig:** Receiver calls `close(ch)` — sender may panic on send to closed channel
**Fix:** Only sender closes. Use `sync.Once` or context for broadcast shutdown.
**Sev:** 🟡 Panic: send on closed channel

**GO-004: `for range` loop variable capture**
**Sig:** `for _, item := range items { go func() { use(item) }() }` — all goroutines see last item
**Fix:** `item := item` inside loop (Go 1.22+ fixed this for loop vars)
**Sev:** 🔴 All goroutines operate on same value

### Memory
**GO-005: Slice from array retains whole array**
**Sig:** `small := large[:5]` — small slice keeps entire large array alive
**Fix:** `small := make([]T, 5); copy(small, large[:5])`
**Sev:** 🟡 Memory bloat — large array can't be GC'd

**GO-006: `defer` in loop**
**Sig:** `for { f, _ := os.Open(...); defer f.Close() }` — all defers stack until function return
**Fix:** Wrap in closure: `func() { f, _ := os.Open(...); defer f.Close(); ... }()`
**Sev:** 🟡 File descriptor exhaustion

### Error Handling
**GO-007: `err` shadowed in scope**
**Sig:** `result, err := fn1(); if err != nil { ... }; otherResult, err := fn2()` — new err shadows
**Fix:** Use `var err error` or `=` instead of `:=` for second assignment
**Sev:** 🟡 Silent variable shadowing

**GO-008: `panic` in HTTP handler without recover**
**Sig:** Handler panics → entire connection crashes, no recovery
**Fix:** `net/http` has built-in recover per goroutine, but custom goroutines need `defer recover()`
**Sev:** 🔴 Server crash on unhandled panic

### Performance
**GO-009: `string` concat in loop**
**Sig:** `s += item` in for-loop — O(n²) string allocation
**Fix:** `strings.Builder` or `[]string` + `strings.Join()`
**Sev:** 🟡 GC pressure

**GO-010: `fmt.Sprintf` in hot path**
**Sig:** `fmt.Sprintf("user_%d", id)` — reflection overhead
**Fix:** `strconv.Itoa(id)` or `"user_" + strconv.Itoa(id)`
**Sev:** 🟢 Format parsing overhead

**GO-011: `json.Marshal` + `json.Unmarshal` for clone**
**Sig:** `json.Unmarshal(json.Marshal(obj))` — serialize+deserialize to copy
**Fix:** Manual copy or `github.com/jinzhu/copier` — avoid JSON round-trip for deep copy
**Sev:** 🟡 CPU + GC for large objects

### Design
**GO-012: Global `map` without mutex**
**Sig:** `var cache = make(map[string]interface{})` — concurrent read/write panics
**Fix:** `sync.RWMutex` or `sync.Map`
**Sev:** 🔴 Fatal: concurrent map writes

**GO-013: Nil interface with non-nil concrete value**
**Sig:** `var err error = (*MyError)(nil)` — err != nil is true
**Fix:** Return `nil` explicitly, not typed nil
**Sev:** 🟡 Interface nil check unexpected behavior

**GO-014: `time.After` in select loop — memory leak**
**Sig:** `for { select { case <-time.After(d): ... } }` — new timer each iteration
**Fix:** `t := time.NewTimer(d); defer t.Stop()` — reuse timer
**Sev:** 🟡 Timer leak + memory growth

**GO-015: `http.ListenAndServe` without graceful shutdown**
**Sig:** `http.ListenAndServe(":8080", nil)` — no shutdown hook
**Fix:** `srv := &http.Server{...}; go srv.ListenAndServe(); ... srv.Shutdown(ctx)`
**Sev:** 🟡 In-flight requests killed on SIGTERM

---

## C++ (15 patterns)

### Memory
**CPP-001: `new` without `delete` — raw owning pointer**
**Sig:** `T* ptr = new T(...)` — no `delete`, no smart pointer
**Fix:** `auto ptr = std::make_unique<T>(...)` or `std::make_shared<T>(...)`
**Sev:** 🔴 Memory leak

**CPP-002: `shared_ptr` alias constructor drops `const`**
**Sig:** `std::shared_ptr<T>(owner, &owner->get_const_ref())` — non-const T despite const source
**Fix:** `std::shared_ptr<const T>(owner, &owner->get_const_ref())`
**Sev:** 🔴 Const stripped silently

**CPP-003: Dangling reference to temporary**
**Sig:** `const auto& ref = get_temp()` — temporary destroyed at end of statement if not lifetime-extended
**Fix:** `auto val = get_temp()` — take by value
**Sev:** 🟡 Use-after-free

**CPP-004: `vector` iterator invalidation**
**Sig:** `for (auto it = v.begin(); it != v.end(); ++it) { v.push_back(x); }` — reallocation invalidates iterators
**Fix:** Reserve capacity first, or use index-based loop
**Sev:** 🟡 Undefined behavior

### Concurrency
**CPP-005: `shared_ptr` reference count race**
**Sig:** Multiple threads copy/destroy same `shared_ptr` — non-atomic ref count for non-atomic specializations
**Fix:** Standard `shared_ptr` is thread-safe for ref count; avoid raw pointer sharing
**Sev:** 🟡 Double-free if using non-standard shared_ptr

**CPP-006: `std::mutex` not unlocked on exception**
**Sig:** `mtx.lock(); /* code that may throw */ mtx.unlock();`
**Fix:** `std::lock_guard<std::mutex> lock(mtx);` — RAII
**Sev:** 🔴 Deadlock: mutex never released

### Security
**CPP-007: `sprintf` / `strcpy` — buffer overflow**
**Sig:** `char buf[64]; sprintf(buf, "%s", user_input)` — no bounds check
**Fix:** `snprintf(buf, sizeof(buf), ...)` or `std::string` / `std::format`
**Sev:** 🔴 Buffer overflow

**CPP-008: `system()` with user input**
**Sig:** `std::system(("ls " + user_path).c_str())` — shell injection
**Fix:** Use filesystem API; never pass user input to shell
**Sev:** 🔴 Command injection

### Performance
**CPP-009: Pass-by-value for large objects**
**Sig:** `void fn(std::vector<int> v)` — copies entire vector
**Fix:** `void fn(const std::vector<int>& v)` or `std::span`
**Sev:** 🟡 Unnecessary copy per call

**CPP-010: `std::endl` instead of `'\n'`**
**Sig:** `std::cout << "msg" << std::endl` — flushes every time
**Fix:** `std::cout << "msg\n"` — only flush when needed
**Sev:** 🟢 I/O performance degradation

**CPP-011: `map` instead of `unordered_map` for key-only lookup**
**Sig:** `std::map` used when ordering not needed — O(log n) vs O(1)
**Fix:** `std::unordered_map` when no ordering requirement
**Sev:** 🟢 Tree overhead for simple lookups

### Design
**CPP-012: Rule of 3/5 violation**
**Sig:** Custom destructor without copy constructor / copy assignment
**Fix:** Implement all 5, or `= delete` them, or use RAII members
**Sev:** 🟡 Double-free / shallow copy

**CPP-013: Virtual destructor missing in base class**
**Sig:** `class Base { ~Base() {} }` — non-virtual, derived destructor never called
**Fix:** `virtual ~Base() = default;`
**Sev:** 🔴 Resource leak on `delete base_ptr`

**CPP-014: `static` initialization order fiasco**
**Sig:** `static Foo foo; static Bar bar(foo.get())` — across translation units, order undefined
**Fix:** Meyer's singleton: `static Foo& get() { static Foo f; return f; }`
**Sev:** 🟡 Use-before-init crash

**CPP-015: `reinterpret_cast` breaking strict aliasing**
**Sig:** `reinterpret_cast<float*>(&int_val)` — undefined behavior
**Fix:** `std::bit_cast<float>(int_val)` (C++20) or `memcpy`
**Sev:** 🟡 Undefined behavior under optimization
