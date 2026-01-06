---
name: Fake Implementation Detection
description: This skill should be used when reviewing code for hidden technical debt, auditing for placeholder implementations, checking for hollow code patterns, or validating production readiness. Activates on requests like "find fake implementations", "detect placeholders", "check for stubs", "audit for mock code", "find hollow implementations", "production readiness check". Provides detection patterns for behavioral mimicry - code that appears complete but performs no real work.
version: 1.0.0
model: opus
---

# Fake Implementation Detection Guide

Fake implementations are code that appears complete but performs no meaningful work. Unlike dead code (never called), fake implementations ARE called but provide no real value—the most insidious form of technical debt.

## The Behavioral Mimicry Pattern

```
FAKE IMPLEMENTATION = Code that:
├── ACCEPTS valid inputs (passes type checking)
├── RETURNS valid outputs (satisfies interface)
└── BUT performs NO meaningful work
    ├── No external I/O (DB, API, filesystem)
    ├── No state mutation that persists
    └── No computation dependent on input
```

## Detection Levels

### Level 1: Lexical Patterns
- Identifiers: `mock*`, `fake*`, `stub*`, `dummy*`, `placeholder*`
- Comments: `TODO`, `FIXME`, `HACK`, `XXX`
- Strings: "not implemented", "coming soon", "TBD"

### Level 2: Structural Patterns
| Pattern | Description |
|---------|-------------|
| Empty Body | `() => {}` - no statements |
| Identity Function | `(x) => x` - returns param unchanged |
| Constant Return | `() => []` - static value regardless of input |
| No-Op Handler | `onError={() => {}}` - ignores event |
| Console-Only | Body contains only logging |

### Level 3: Semantic Patterns
| Pattern | Real Implementation |
|---------|---------------------|
| In-Memory "Database" | Should be DB calls |
| Hardcoded Pool | Should be computed/fetched |
| Artificial Delay | Should be actual async operation |
| Round-Robin | Should be algorithm/ML decision |

### Level 4: Behavioral Patterns
| Pattern | Symptom |
|---------|---------|
| Swallowed Errors | `catch(e) { return success }` |
| Always-Success | Never returns error states |
| No Persistence | Writes don't survive restart |

## Classification Matrix

| Location | Explicit Marker | Classification |
|----------|----------------|----------------|
| Test file | Yes/No | ✅ Expected |
| Prod code | Yes (TODO/stub) | 🟡 Acknowledged debt |
| Prod code | No marker | 🔴 Hidden debt |

## Analysis Approach

1. **Discover project structure**: Check for monorepo patterns before scanning. Never assume `/src/` is the only source directory.

2. **Apply detection levels progressively**: Start with lexical patterns, then structural, semantic, and behavioral.

3. **Distinguish test from production**: Files in `*.test.*`, `*.spec.*`, `__mocks__/`, `__tests__/` are expected to contain fakes.

4. **Report coverage**: Always verify and report which directories were scanned.

## Why This Matters

Fake implementations are dangerous because they satisfy all automated checks:
- ✅ Type checking passes
- ✅ Linting passes
- ✅ Unit tests pass (if mocked)
- ✅ Code review sees "complete" code
- ❌ Only fails in production with real data/load

The best defense combines: naming conventions, integration tests, code review awareness, and CI rules blocking placeholder markers in production.
