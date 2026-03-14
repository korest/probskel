# probskel

A Claude Code skill that generates coding problem skeletons with comprehensive tests.

**Perfect for:**
- **Learning a new language** - practice with real problems, not just tutorials
- **Improving problem-solving skills** - work through progressively harder challenges
- **Interview prep** - practice algorithms and data structures with instant feedback
- **Teaching** - create exercises with built-in test suites

The skill creates a skeleton (stub code) and a full test suite for any problem you describe. You implement the solution, run tests, and ask "how did I do?" for analysis.

## Usage

```
/probskel <problem> [--level low|mid|high] [--stages] [--async|--threaded] [--pair] [--socratic]
```

**Examples:**
```
/probskel two-sum
/probskel LRU cache --level high
/probskel LRU cache --stages
/probskel web crawler --async
/probskel LRU cache --pair
/probskel two-sum --pair --hint low
/probskel rate limiter --socratic
```

## Levels (Guidance Amount)

Levels control how much help the skeleton provides, not problem difficulty.

| Level | Structure | Best for |
|-------|-----------|----------|
| `low` | Essential methods only, minimal hints | Experienced, want a challenge |
| `mid` | Main + helper stubs with brief docs | Balanced guidance |
| `high` | Full decomposition, step-by-step hints | Learning, or complex problems |

All levels include a clear problem description with examples.

## Stages (Progressive Difficulty)

Use `--stages` to split a problem into 3 progressively harder challenges, each with separate files and tests.

**Example: `/probskel LRU cache --stages`**

| Stage | File | Focus |
|-------|------|-------|
| 1 | `lru_cache_stage1.py` | Basic get/put with eviction |
| 2 | `lru_cache_stage2.py` | Add TTL expiration |
| 3 | `lru_cache_stage3.py` | Thread-safe concurrent access |

Work through stages one at a time:
```bash
pytest tests/test_lru_cache_stage1.py  # Start here
pytest tests/test_lru_cache_stage2.py  # After stage 1 passes
pytest tests/test_lru_cache_stage3.py  # Full solution
```

## Pair Programming Mode

Use `--pair` to get a senior pair programming partner after skeleton generation. Instead of solving alone, you work through the implementation step by step with guided feedback.

- **`--pair`** — Collaborative style (default). Shares reasoning openly, explains trade-offs, points out patterns — like working with a senior colleague.
- **`--socratic`** — Question-driven style. Leads with targeted questions to help you discover the answer yourself.

Both modes are independent of `--level` — combine them freely.

### Workflow with `--pair`

1. `/probskel LRU cache --pair` → generates skeleton + tests
2. "Let's start" → AI breaks implementation into steps and guides you through each one
3. Write code for each step → get feedback on what works, what to improve, and why
4. After all steps → automatic full analysis

## Tests

Generated tests cover:
- **Edge cases** - empty, single element, boundaries
- **Happy path** - standard valid inputs
- **Performance** - large N that catches O(N²) solutions
- **Concurrency** (when applicable) - thread safety, race conditions

## Workflow

1. `/probskel binary search` → generates skeleton + tests
2. Implement your solution (or use `--pair` for guided implementation)
3. `pytest` → run tests
4. "how did I do?" → get analysis and feedback

## Installation

```bash
git clone https://github.com/yourusername/probskel.git
ln -s /path/to/probskel ~/.claude/skills/probskel
```

## License

MIT
