---
name: ts-scan
description: Scan TypeScript project for type errors and generate categorized health report. Use this command to audit type safety, assess technical debt, or prepare for stricter TypeScript configuration.
version: 1.0.0
model: opus
---

# TypeScript Type Health Scan

Scan the current project for TypeScript type errors and generate a comprehensive health report.

## What This Command Does

1. **Detects project structure** - single project or monorepo (pnpm, lerna, nx, turborepo)
2. **Finds type check command** - from package.json scripts or uses tsc directly
3. **Runs type checking** - executes the configured type check
4. **Classifies errors** into two categories:
   - **Quick Fixes**: Errors solvable with targeted code changes (missing types, null checks, typos)
   - **Design Issues**: Errors indicating architectural problems requiring design decisions
5. **Analyzes root causes** for design issues - traces where type contracts broke
6. **Generates health report** with score, hotspots, and recommendations

## Report Contents

- **Summary**: Error counts by category with percentages
- **Health Score**: 0-100 based on error severity
- **Quick Fixes**: Grouped by error type with remediation guidance
- **Design Issues**: With root cause analysis for each group
- **Hotspots**: Files with highest error concentration
- **Recommendations**: Prioritized action items

## Usage

```
/ts-scan
```

No arguments required. The scan will:
- Automatically detect TypeScript configuration
- Handle monorepo structures
- Display results in the conversation

## Error Classification

### Quick Fix Categories
- Implicit `any` types (TS7xxx)
- Null/undefined safety (TS2531-2533)
- Missing properties (TS2339, TS2551)
- Argument mismatches (TS2554-2556)
- Simple type assignability (TS2322, TS2345)

### Design Issue Categories
- Missing type definitions (TS2304, TS2305)
- Generic constraint problems (TS2314, TS2344)
- Type incompatibility (TS2559, TS2739)
- Inheritance violations (TS2420, TS2430)
- Type complexity (TS2589, TS2590)

## Health Score Interpretation

| Score | Rating | Action |
|-------|--------|--------|
| 90-100 | Healthy | Maintain current practices |
| 70-89 | Needs Attention | Address quick fixes soon |
| 50-69 | Poor | Plan refactoring sprint |
| 0-49 | Critical | Immediate intervention needed |

---

Use the `typescript-type-scanner` agent to perform this analysis.
