---
name: probskel
description: Generate coding problem skeletons with comprehensive tests. Use when user wants to practice a coding problem, create a kata, set up TDD for an algorithm challenge, or review their solution. Supports hints (low/mid/high), concurrency modes (async/threaded), and pair programming mode.
---

# Problem Skeleton Generator

## Usage

```
/probskel <problem description> [--hint low|mid|high] [--lang <language>] [--async|--threaded] [--levels] [--pair] [--socratic]
```

## Instructions

### Step 1: Parse Arguments

- **Problem description**: The coding problem to scaffold
- **Hint**: `--hint low` (default), `mid`, or `high`
- **Language**: `--lang <language>` or detect from project files
- **Concurrency**: `--async`, `--threaded`, or auto-detect
- **Levels**: `--levels` to split into progressive difficulty levels
- **Pair**: `--pair` to enable pair programming mode (collaborative by default)
- **Socratic**: `--socratic` to use question-driven pair style (implies `--pair`)

### Step 2: Detect Language

Detect from: `pyproject.toml`/`requirements.txt` → Python, `package.json`/`tsconfig.json` → TS/JS, `go.mod` → Go, `Cargo.toml` → Rust

If ambiguous, ask the user.

### Step 3: Determine Concurrency Mode

**Auto-detect based on problem type:**

| Use async | Use threading |
|-----------|---------------|
| Web crawler, API client | Image/video processing |
| Rate limiter, download manager | File batch operations |
| WebSocket, streaming | Blocking libraries (PIL, OpenCV) |
| High concurrency I/O | CPU-bound with I/O |

**Keep synchronous for:** Pure algorithms, data structures, string manipulation, math computations.

**When uncertain**, ask: async (I/O-bound) vs threading (CPU-bound/blocking) vs synchronous.

### Step 4: Progressive Levels (when `--levels` flag used)

Split the problem into 3 levels of increasing difficulty. Each level has its own tests that build on previous levels.

#### Level Structure

| Level | Focus | Tests enabled |
|-------|-------|---------------|
| **Level 1: Core** | Basic happy path, synchronous, single input | `TestLevel1_*` |
| **Level 2: Robust** | Error handling, edge cases, input validation | `TestLevel1_*`, `TestLevel2_*` |
| **Level 3: Production** | Concurrency, performance, retries, real-world scenarios | All tests |

#### How to Break Down Any Problem

Think about progression along these dimensions:

| Dimension | Level 1 (Simple) | Level 2 (Medium) | Level 3 (Hard) |
|-----------|------------------|------------------|----------------|
| **Input size** | Single item, small N | Multiple items, medium N | Large N, streaming |
| **Input validity** | Always valid | Edge cases, empty, null | Malformed, adversarial |
| **Operations** | One operation type | Multiple operations | Mixed concurrent ops |
| **State** | Stateless or simple | State with constraints | Persistent, recoverable |
| **Time** | Instant | TTL, expiration | Real-time, scheduling |
| **Concurrency** | Single-threaded | Thread-safe reads | Full concurrent R/W |
| **Failures** | Assume success | Handle errors | Retry, circuit breaker |

**Pattern**: Start with the "happy path single operation", then add constraints, then add real-world concerns.

#### Example Progressions

**Algorithm: Two Sum**
| Level | Focus | Key constraint |
|-------|-------|----------------|
| 1 | Find two numbers that sum to target | O(N²) acceptable |
| 2 | Optimize to O(N) with hash map | Handle duplicates, negatives |
| 3 | Streaming input, memory-constrained | Return all pairs, sorted |

**Data Structure: LRU Cache**
| Level | Focus | Key constraint |
|-------|-------|----------------|
| 1 | get/put with eviction | Basic O(1) operations |
| 2 | TTL expiration per item | Lazy vs eager cleanup |
| 3 | Thread-safe concurrent access | Lock granularity, no deadlocks |

**System: Rate Limiter**
| Level | Focus | Key constraint |
|-------|-------|----------------|
| 1 | Fixed window counter | Single client, in-memory |
| 2 | Sliding window log | Multiple limits (per-second, per-minute) |
| 3 | Distributed rate limiting | Redis-backed, consistency |

**I/O: File Processor**
| Level | Focus | Key constraint |
|-------|-------|----------------|
| 1 | Read and transform single file | Assume file exists, fits in memory |
| 2 | Handle missing files, encoding errors | Stream large files |
| 3 | Batch process directory | Concurrent, resume on failure |

**Network: API Client**
| Level | Focus | Key constraint |
|-------|-------|----------------|
| 1 | Single request, parse response | Assume success |
| 2 | Handle errors, timeouts, retries | Exponential backoff |
| 3 | Concurrent requests with rate limit | Connection pooling, circuit breaker |

#### Quick Reference: Common Level Patterns

| Problem Type | Level 1 → 2 adds | Level 2 → 3 adds |
|--------------|------------------|------------------|
| **Algorithms** | Optimize complexity | Streaming/memory constraints |
| **Data structures** | Multiple operations | Concurrency |
| **Caches** | TTL/expiration | Thread safety |
| **Queues** | Priority, delays | Persistence, dead letter |
| **Pools** | Timeouts, health | Auto-scaling, metrics |
| **Crawlers** | Depth limits | Rate limiting, retries |
| **Parsers** | Error recovery | Streaming, partial results |

#### Level File Structure

**Always create separate files per level.** Each level is a standalone implementation with its own tests. This keeps levels independent and easy to work on.

```
src/
  lru_cache_level1.py      # Level 1: Basic cache
  lru_cache_level2.py      # Level 2: Cache with TTL
  lru_cache_level3.py      # Level 3: Thread-safe cache
tests/
  test_lru_cache_level1.py
  test_lru_cache_level2.py
  test_lru_cache_level3.py
```

#### Example: LRU Cache in 3 Levels

**Level 1 - Basic Cache** (`lru_cache_level1.py`)
- `get(key)` → retrieve value, update recency
- `put(key, value)` → store value
- Eviction when capacity exceeded
- Focus: O(1) operations with dict + ordering

**Level 2 - Cache with TTL** (`lru_cache_level2.py`)
- Everything from Level 1, plus:
- `put(key, value, ttl_seconds)` → item expires after TTL
- `get()` returns `None` for expired items
- Focus: Time-based expiration (lazy or eager cleanup)

**Level 3 - Thread-Safe Cache** (`lru_cache_level3.py`)
- Everything from Level 2, plus:
- Safe for concurrent readers and writers
- No deadlocks under high concurrency
- Focus: Lock granularity, atomic operations

#### Running Levels

User implements one level at a time:
```bash
# Work on Level 1
pytest tests/test_lru_cache_level1.py

# Move to Level 2 after Level 1 passes
pytest tests/test_lru_cache_level2.py

# Complete with Level 3
pytest tests/test_lru_cache_level3.py
```

Each level has independent tests. User can stop at any level with a working solution at that complexity level.

### Step 5: Generate Skeleton

#### Skeleton Hints

Hints control **guidance amount**. All hints include a clear problem description with examples at the top.

| Hint | Structure | Description |
|-------|-----------|-------|
| **low** | Essential methods only | Brief comment, minimal guidance |
| **mid** | Main + helper method stubs | Short docstrings explaining each method's purpose |
| **high** | Full decomposition with all helpers | Detailed hints per method, step-by-step approach |

**Key difference**: At `high` hint, implementing becomes "fill in the blanks" - each method has comments explaining what to do. At `low` hint, you figure out the decomposition yourself.

#### Example: LRU Cache at Different Hints

**All hints have this header:**
```python
"""
LRU Cache - Least Recently Used cache with O(1) get/put operations.

Example:
    cache = LRUCache(capacity=2)
    cache.put(1, 'a')
    cache.put(2, 'b')
    cache.get(1)       # returns 'a', marks as recently used
    cache.put(3, 'c')  # evicts key 2 (least recently used)
    cache.get(2)       # returns None (evicted)
"""
```

**Low hint** - essential structure only:
```python
class LRUCache:
    def __init__(self, capacity: int):
        # Initialize cache with given capacity
        raise NotImplementedError()

    def get(self, key: int) -> str | None:
        raise NotImplementedError()

    def put(self, key: int, value: str) -> None:
        raise NotImplementedError()
```

**High hint** - guided implementation:
```python
class LRUCache:
    def __init__(self, capacity: int):
        # TODO: Store capacity
        # TODO: Initialize dict for O(1) key lookup
        # TODO: Initialize structure to track access order
        #       (consider: OrderedDict, or dict + doubly linked list)
        raise NotImplementedError()

    def get(self, key: int) -> str | None:
        # TODO: Return None if key not found
        # TODO: Move key to "most recently used" position
        # TODO: Return the value
        raise NotImplementedError()

    def put(self, key: int, value: str) -> None:
        # TODO: If key exists, update value and mark as recently used
        # TODO: If at capacity, remove least recently used item
        # TODO: Add new key-value pair as most recently used
        raise NotImplementedError()

    def _move_to_end(self, key: int) -> None:
        # Helper: Move existing key to most recently used position
        raise NotImplementedError()

    def _evict_lru(self) -> None:
        # Helper: Remove the least recently used item
        raise NotImplementedError()
```

#### Concurrency Patterns

**Async skeleton includes:**
- `import asyncio, aiohttp`
- `async def` functions
- `asyncio.Semaphore` for concurrency limits
- Proper session management with `async with`
- Retry helper with exponential backoff (for network operations)

**Threading skeleton includes:**
- `from concurrent.futures import ThreadPoolExecutor`
- `import threading`
- `threading.Lock()` for shared state
- `max_workers` parameter

#### Retry Pattern (for network/I/O operations)

When problem involves external services, include retry logic:
- Exponential backoff: `delay = base * (2 ** attempt)`
- Jitter: add randomness to prevent thundering herd
- Max retries: configurable limit (default 3)
- Retryable errors: network timeouts, 5xx responses, connection refused
- Non-retryable: 4xx client errors, validation failures

### Step 6: Generate Tests

**Focus on real-world scenarios**, not just academic edge cases. Tests should reflect what happens in production.

#### Standard Test Categories

| Category | What to test |
|----------|--------------|
| **Invalid input** | Malformed data, wrong types, missing fields, corrupt files |
| **Boundary conditions** | Empty, single item, exactly at limits, off-by-one scenarios |
| **Real-world data** | Unicode, special characters, whitespace, very long strings |
| **Error handling** | What happens when dependencies fail? Graceful degradation? |
| **Performance** | Large N that timeouts with O(N²) but passes with O(N) |

#### Real-World Scenarios (adapt to problem type)

| Scenario | Examples |
|----------|----------|
| **Partial failures** | 3 of 10 API calls fail - does it handle gracefully? |
| **Timeouts** | External service is slow - does it timeout and retry? |
| **Resource cleanup** | Files/connections closed on error? No leaks? |
| **Idempotency** | Running twice produces same result? Safe to retry? |
| **Data consistency** | Concurrent updates don't corrupt state? |

#### Retry Tests (when applicable)

| Scenario | What to verify |
|----------|----------------|
| **Succeeds after transient failure** | Fails twice, succeeds third - returns success |
| **Respects max retries** | After N failures, gives up with proper error |
| **Exponential backoff** | Delays increase between attempts |
| **Non-retryable errors** | 400 Bad Request fails immediately, no retry |
| **Timeout per attempt** | Each retry has its own timeout, not cumulative |

#### Async Test Categories (additional)

| Category | What to test |
|----------|--------------|
| **Concurrency limits** | Respects max_concurrent, doesn't overwhelm external services |
| **Partial failures** | Some tasks fail, others complete, proper error aggregation |
| **Cancellation** | Clean shutdown when cancelled, no orphaned tasks |
| **Backpressure** | Handles slow consumers, doesn't OOM on fast producers |

#### Threading Test Categories (additional)

| Category | What to test |
|----------|--------------|
| **Thread safety** | Shared state not corrupted under concurrent access |
| **Deadlock prevention** | No circular lock dependencies |
| **Resource contention** | Performs well under high concurrency |
| **Graceful shutdown** | Threads terminate cleanly, no zombie processes |

#### Test Patterns

**Python (pytest):**
- Group by scenario: `TestInvalidInput`, `TestPartialFailures`, `TestPerformance`
- Use `@pytest.mark.timeout(5)` for performance tests
- Use `@pytest.mark.asyncio` for async tests
- Mock external dependencies to simulate failures
- Use `pytest.raises` with specific exception types

### Step 7: Output Summary

Print summary showing:
- Files created with level and mode
- Test counts by category
- Command to run tests (`pytest`, `npm test`, `go test`)
- Reminder: "ask 'review my solution' or 'how did I do?' for analysis"

### Step 8: Pair Programming Mode (when `--pair` or `--socratic` used)

After skeleton generation and output summary, suggest pair programming:

> "When you're ready, I can walk you through the implementation step by step as your pair programming partner."

**Do NOT auto-start.** Wait for the user to explicitly ask to begin.

#### Starting a Pair Session

When the user asks to start:

1. Read the generated skeleton and tests.
2. Break the implementation into logical steps (e.g., for LRU Cache: "constructor → `get` → `put` → eviction helper").
3. Present the first step with reasoning: "Let's start with the constructor — it sets up the data structures everything else depends on."
4. Wait for the user to write their code.

#### After Each Step

1. Read the user's code.
2. Run the relevant tests if possible.
3. Review:
   - What they got right and why it works.
   - What could be improved and **why** — explain the reasoning, don't just say "change X to Y."
   - If a language-specific idiom applies, show the pattern and explain when to reach for it.
4. When the step is solid, present the next one.

#### Collaborative Style (default with `--pair`)

Act like a senior colleague at a whiteboard:

- **Share reasoning openly**: "I'd reach for an OrderedDict here because it gives us O(1) move-to-end, which we need for the LRU update."
- **Explain trade-offs**: "You could use a plain dict + doubly linked list instead — more code but avoids the OrderedDict overhead. For this problem either works."
- **Point out patterns in context**: "This is a good spot for a context manager — if the file open fails, your lock never gets released."
- **Give honest feedback**: If something works but isn't great, say so directly. Don't pad with unnecessary praise.
- **Explain new concepts when they arise**: If you suggest using a technique or pattern the user might not know, explain it the way you'd explain it to a colleague — concisely, with the "why" front and center.

#### Socratic Style (`--socratic`)

Lead with questions, let the user discover answers:

- **Ask targeted questions**: "What data structure gives you O(1) lookup by key?" / "What happens to your cache when it's full and you call `put`?"
- **Only explain directly when the user is stuck** or explicitly asks.
- **Ask the user to predict behavior**: "Before we run the tests — what do you think will happen with this edge case?"
- **Gradually reduce guidance** as the user demonstrates understanding.

#### Finishing the Pair Session

After all steps are implemented:
- Run the full test suite.
- Transition naturally into Analysis Mode (Step 9) for a holistic review.
- Include the language-specific best practices checklist in the review.

### Step 9: Analysis Mode

**Trigger phrases** (use this skill's analysis mode when user says any of these):
- "how did I do?"
- "review my solution"
- "analyze my solution"
- "tell me how I did"
- "verify my solution"
- "check my implementation"
- "grade my code"

When triggered:

1. Read their implementation
2. Run tests, capture results
3. Provide analysis covering:

| Area | What to review |
|------|----------------|
| **Test results** | Pass/fail counts, which scenarios failed |
| **What they did well** | Correct approach, clean code, good naming |
| **Data structures** | Optimal choice? (dict for O(1) lookup, set for membership, heapq for top-k) |
| **Complexity** | Time/space optimal? What's the bottleneck? Unnecessary copies? |
| **Language best practices** | Idiomatic patterns for the language (see checklist below) |
| **Concurrency** | Proper semaphores/locks? Race conditions? Backpressure handling? |
| **Retry logic** | Exponential backoff? Max retries? Distinguishes retryable vs non-retryable errors? |
| **Error handling** | Fails gracefully? Specific exceptions? Proper cleanup? |
| **Production readiness** | See checklist below |

#### Language-Specific Best Practices Checklist

**Python:**
| Practice | Instead of | Use |
|----------|------------|-----|
| **List comprehension** | `for` loop appending to list | `[x*2 for x in items]` |
| **Dict comprehension** | `for` loop building dict | `{k: v for k, v in pairs}` |
| **Set comprehension** | `for` loop adding to set | `{x for x in items if x > 0}` |
| **Generator expression** | List comprehension when iterating once | `sum(x*2 for x in items)` |
| **str.join()** | Concatenation in loop | `', '.join(strings)` |
| **f-strings** | `%` formatting or `.format()` | `f"Hello {name}"` |
| **Context managers** | Manual open/close | `with open(f) as file:` |
| **enumerate()** | Manual index tracking | `for i, item in enumerate(items):` |
| **zip()** | Parallel index iteration | `for a, b in zip(list1, list2):` |
| **any()/all()** | Loop with flag variable | `any(x > 0 for x in items)` |
| **defaultdict/Counter** | Manual dict initialization | `from collections import defaultdict` |
| **Unpacking** | Index access | `first, *rest = items` |
| **Walrus operator** | Separate assignment and condition | `if (n := len(items)) > 10:` |
| **pathlib** | String path manipulation | `from pathlib import Path` |

**TypeScript/JavaScript:**
| Practice | Instead of | Use |
|----------|------------|-----|
| **Array methods** | `for` loops | `.map()`, `.filter()`, `.reduce()` |
| **Template literals** | String concatenation | `` `Hello ${name}` `` |
| **Destructuring** | Repeated property access | `const { name, age } = user` |
| **Spread operator** | Manual copying | `[...arr1, ...arr2]` |
| **Optional chaining** | Nested null checks | `user?.address?.city` |
| **Nullish coalescing** | `\|\|` for defaults | `value ?? defaultValue` |
| **Object shorthand** | `{name: name}` | `{name}` |
| **Array.from()** | Manual iteration | `Array.from(nodeList)` |
| **Set for uniqueness** | Manual deduplication | `[...new Set(items)]` |
| **Promise.all()** | Sequential awaits | `await Promise.all(promises)` |
| **async/await** | `.then()` chains | `const result = await fetch()` |
| **const/let** | `var` | Prefer `const`, use `let` when reassigning |

**Go:**
| Practice | Instead of | Use |
|----------|------------|-----|
| **Error wrapping** | Plain error return | `fmt.Errorf("context: %w", err)` |
| **defer for cleanup** | Manual cleanup calls | `defer file.Close()` |
| **errors.Is/As** | String comparison | `errors.Is(err, ErrNotFound)` |
| **strings.Builder** | String concatenation | `var b strings.Builder` |
| **make() with capacity** | Growing slices | `make([]int, 0, expectedLen)` |
| **Range over slice** | Index-based iteration | `for i, v := range items` |
| **Named return values** | Explicit returns in defer | `func f() (err error)` |
| **Embed for composition** | Inheritance patterns | Struct embedding |
| **Table-driven tests** | Repeated test functions | `tests := []struct{...}` |
| **Context for cancellation** | Manual timeouts | `ctx, cancel := context.WithTimeout()` |

**Rust:**
| Practice | Instead of | Use |
|----------|------------|-----|
| **Iterator methods** | Manual loops | `.iter().map().filter().collect()` |
| **? operator** | Match on Result/Option | `let value = result?;` |
| **if let / while let** | Full match for single variant | `if let Some(x) = opt {}` |
| **Destructuring** | Field-by-field access | `let Point { x, y } = point;` |
| **String slices** | Owned strings when borrowing | `fn f(s: &str)` vs `fn f(s: String)` |
| **collect() with turbofish** | Manual collection building | `items.collect::<Vec<_>>()` |
| **Entry API** | Contains + insert | `map.entry(k).or_insert(v)` |
| **Clone on write** | Unnecessary cloning | `Cow<str>` for optional ownership |
| **derive macros** | Manual trait impls | `#[derive(Debug, Clone)]` |
| **impl Into/AsRef** | Concrete types in params | Generic over input types |

#### Production Readiness Checklist

| Concern | Questions to ask |
|---------|------------------|
| **Input validation** | Validates early? Clear error messages? |
| **Failure modes** | What happens when X fails? Retries? Fallbacks? |
| **Resource management** | Connections/files closed? Context managers used? |
| **Observability** | Could add logging? Metrics? How to debug in prod? |
| **Configuration** | Hardcoded values that should be configurable? |
| **Idempotency** | Safe to retry? Side effects handled? |
| **Security** | Input sanitized? Injection risks? Secrets exposed? |

## Directory Structure

Separate source code from tests. Detect existing structure or create standard layout.

**Python:**
```
src/
  problem_name.py
tests/
  test_problem_name.py
```

**TypeScript/JavaScript:**
```
src/
  problem-name.ts
tests/
  problem-name.test.ts
```

**Go:** (tests colocated by convention)
```
problem_name.go
problem_name_test.go
```

**Rust:**
```
src/
  lib.rs           # for library crates
  problem_name.rs  # for binary crates
tests/
  problem_name_test.rs
```

If project already has `src/`, `lib/`, or `tests/` directories, follow existing conventions.
