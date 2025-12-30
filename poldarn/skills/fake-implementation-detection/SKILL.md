---
name: Fake Implementation Detection
description: This skill should be used when reviewing code for hidden technical debt, auditing for placeholder implementations, checking for hollow code patterns, or validating production readiness. Activates on requests like "find fake implementations", "detect placeholders", "check for stubs", "audit for mock code", "find hollow implementations", "production readiness check". Provides detection patterns for behavioral mimicry - code that appears complete but performs no real work.
version: 1.0.0
model: opus
---

# Fake Implementation Detection Guide

Fake implementations are code that appears complete but performs no meaningful work. Unlike dead code (never called), fake implementations ARE called but provide no real value—the most insidious form of technical debt.

## Project Structure Discovery (CRITICAL FIRST STEP)

Before scanning for fake implementations, you MUST discover the complete project structure. Scanning only `/src/` when a project has `/packages/*/src/` directories will miss entire codebases.

### Monorepo Detection Patterns

| Pattern | Indicator Files | Common Locations |
|---------|----------------|------------------|
| npm/yarn workspaces | `"workspaces"` in package.json | packages/, apps/, libs/ |
| pnpm workspaces | pnpm-workspace.yaml | packages/, apps/, libs/ |
| Lerna | lerna.json | packages/ |
| Nx | nx.json | apps/, libs/ |
| Turborepo | turbo.json | apps/, packages/ |

### Discovery Commands

```bash
cat package.json | grep -A10 '"workspaces"'

find . -type d -name "src" ! -path "*/node_modules/*" ! -path "*/dist/*" ! -path "*/.next/*" ! -path "*/build/*"

find . -name "package.json" ! -path "*/node_modules/*" -exec dirname {} \;

ls -d packages/ apps/ libs/ modules/ services/ 2>/dev/null
```

### Ensuring Full Coverage

1. **Always check for workspaces first** - A root package.json with workspaces means multiple sub-projects exist
2. **Use find, not assumptions** - Never assume `/src/` is the only source location
3. **List before scanning** - Enumerate all directories before running detection patterns
4. **Report coverage** - Include scanned directories in the output to prove completeness

### Coverage Validation

A valid scan report MUST include:
```
Scanned Directories:
- /src/ (root)
- /packages/core/src/
- /packages/web/src/
Total: 3/3 directories (100% coverage)
```

If the report only shows `/src/` when packages exist, the scan is INCOMPLETE.

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

Scan for explicit naming indicators:

| Identifier Pattern | Risk Level |
|-------------------|------------|
| `mock*`, `fake*`, `stub*` | 🔴 Critical in prod |
| `dummy*`, `placeholder*` | 🔴 Critical in prod |
| `temp*`, `test*` (in prod) | 🟡 Warning |

Scan for admission comments:
- `// TODO`, `// FIXME`, `// HACK`, `// XXX`
- `// not implemented`, `// placeholder`
- `throw new Error("Not implemented")`

### Level 2: Structural Patterns

| Pattern | Example | Detection |
|---------|---------|-----------|
| Empty Body | `() => {}` | No statements |
| Identity Function | `(x) => x` | Returns param unchanged |
| Constant Return | `() => []` | Static value regardless of input |
| No-Op Handler | `onError={() => {}}` | Event handler does nothing |
| Console-Only | `() => { console.log(x) }` | Only side effect is logging |

### Level 3: Semantic Patterns

| Pattern | Indicator | Real vs Fake |
|---------|-----------|--------------|
| In-Memory "Database" | Module-level `Map`/`Set`/`Array` | Real: DB calls |
| Hardcoded Pool | `responses[index % responses.length]` | Real: Computed/fetched |
| Artificial Delay | `await sleep(1000)` without work | Real: Actual async operation |
| Round-Robin | Cycling through fixed options | Real: Algorithm/ML decision |

### Level 4: Behavioral Patterns

| Pattern | Symptom |
|---------|---------|
| Swallowed Errors | `catch(e) { return success }` |
| Always-Success | Never returns error states |
| Deterministic Random | Same "random" sequence |
| No Persistence | Writes don't survive restart |

## Detection Checklist

When reviewing code, check:

**Phase 0: Project Structure (MANDATORY)**
- [ ] **Workspaces**: Checked package.json for workspaces field?
- [ ] **Monorepo Config**: Checked for lerna.json, nx.json, turbo.json, pnpm-workspace.yaml?
- [ ] **Source Directories**: Found ALL src/ directories using find command?
- [ ] **Package Locations**: Found ALL package.json files (sub-projects)?
- [ ] **Coverage List**: Compiled list of ALL directories to scan?

**Phase 1-4: Detection Patterns**
- [ ] **Naming**: Any mock/fake/stub/dummy/placeholder in identifiers?
- [ ] **Comments**: Any TODO/FIXME/HACK/XXX in production code?
- [ ] **Empty Bodies**: Functions that do nothing?
- [ ] **Identity Functions**: Functions returning input unchanged?
- [ ] **Constant Returns**: Functions always returning same value?
- [ ] **Module State**: Collections at module level pretending to be databases?
- [ ] **Artificial Delays**: Sleep/timeout without real async work?
- [ ] **Error Swallowing**: Catch blocks that silently succeed?
- [ ] **Missing Integration**: Types imported but not implementations?

**Final: Coverage Verification**
- [ ] **All Scanned**: Every discovered directory was scanned?
- [ ] **Report Includes**: Scanned directories listed in output?

## Classification Matrix

| Location | Explicit Marker | Classification |
|----------|----------------|----------------|
| Test file | Yes/No | ✅ Expected |
| Prod code | Yes (TODO/stub) | 🟡 Acknowledged debt |
| Prod code | No marker | 🔴 Hidden debt |

## Common Patterns by Domain

### API/Service Layer
```typescript
async function fetchUserData(id: string) {
  return { id, name: "Test User", email: "test@example.com" }
}
```
**Red flag**: Hardcoded response, no actual fetch

### Event Handlers
```typescript
<Button onClick={() => {}} />
<Form onSubmit={() => console.log("submitted")} />
```
**Red flag**: Empty handler, console-only handler

### Error Handling
```typescript
try {
  await riskyOperation()
} catch (e) {
  return { success: true, data: null }
}
```
**Red flag**: Error swallowed, returns success anyway

### Data Persistence
```typescript
const users = new Map<string, User>()

export function saveUser(user: User) {
  users.set(user.id, user)
  return { success: true }
}
```
**Red flag**: In-memory storage masquerading as persistence

### Randomization/Selection
```typescript
const responses = ["Great!", "Thanks!", "Got it!"]
let index = 0

export function getResponse() {
  return responses[index++ % responses.length]
}
```
**Red flag**: Deterministic cycling, not real intelligence

## Remediation Strategies

1. **Explicit Markers**: Require `// STUB:` or `// MOCK:` prefixes for intentional placeholders
2. **CI Enforcement**: Fail builds if marker prefixes exist in production code
3. **Interface Contracts**: Use type-level markers like `NotImplemented<T>` wrapper types
4. **Integration Tests**: Verify actual I/O, persistence, and side effects
5. **Code Review Protocol**: Add "fake implementation check" to review checklist

## Quick Reference Commands

### Project Structure Discovery (Run First)

```bash
cat package.json | grep -A10 '"workspaces"'

find . -type d -name "src" ! -path "*/node_modules/*" ! -path "*/dist/*"

find . -name "package.json" ! -path "*/node_modules/*" -exec dirname {} \;

ls -d packages/ apps/ libs/ modules/ services/ 2>/dev/null
```

### Detection Patterns (Run Against All Discovered Directories)

```bash
# Find mock/fake/stub identifiers (replace . with specific paths)
grep -rE '\b(mock|fake|stub|dummy|placeholder)[A-Z_]' .

# Find TODO/FIXME comments
grep -rE '//\s*(TODO|FIXME|HACK|XXX)' .

# Find empty arrow functions
grep -rE '=>\s*\{\s*\}' .

# Find module-level collections
grep -rE '^(const|let)\s+\w+\s*=\s*new\s*(Map|Set)\(' .

# Find swallowed errors
grep -rE 'catch\s*\([^)]*\)\s*\{[^}]*return\s*(true|null|\{\s*success)' .
```

Note: Replace `.` with specific discovered source directories, or use `--include='*.{ts,tsx,js,jsx}'` to filter file types.

## Key Insight

Fake implementations are dangerous because they satisfy all automated checks:
- ✅ Type checking passes
- ✅ Linting passes
- ✅ Unit tests pass (if mocked)
- ✅ Code review sees "complete" code
- ❌ Only fails in production with real data/load

The best defense is a combination of:
1. Naming conventions that expose intent
2. Integration tests that verify real behavior
3. Code review awareness of these patterns
4. CI rules blocking placeholder markers in prod
