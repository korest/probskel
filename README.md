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
/probskel <problem> [--level low|mid|high] [--stages] [--async|--threaded]
```

**Examples:**
```
/probskel two-sum
/probskel LRU cache --level high
/probskel LRU cache --stages
/probskel web crawler --async
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

## Tests

Generated tests cover:
- **Edge cases** - empty, single element, boundaries
- **Happy path** - standard valid inputs
- **Performance** - large N that catches O(N²) solutions
- **Concurrency** (when applicable) - thread safety, race conditions

## Workflow

1. `/probskel binary search` → generates skeleton + tests
2. Implement your solution
3. `pytest` → run tests
4. "how did I do?" → get analysis and feedback

## Installation

```bash
git clone https://github.com/yourusername/probskel.git
ln -s /path/to/probskel ~/.claude/skills/probskel
```

## License

MIT
