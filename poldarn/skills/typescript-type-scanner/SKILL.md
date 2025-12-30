---
name: TypeScript Type Scanner
description: This skill should be used when analyzing TypeScript type health, scanning for type errors, categorizing type issues, understanding tsc error codes, or generating type health reports. Activates on requests like "scan types", "type check", "find type errors", "type health report", "analyze TypeScript errors", "categorize type issues". Provides guidance on TypeScript error classification and type health patterns.
version: 1.0.0
model: opus
---

# TypeScript Type Health Analysis Guide

Guidance for scanning TypeScript projects, understanding error codes, and categorizing type issues into actionable categories.

## Error Classification System

### Category 1: Quick Fixes

Errors that can be resolved with targeted code changes without architectural redesign.

#### Implicit Any Types (TS7xxx)

| Code | Description | Quick Fix |
|------|-------------|-----------|
| TS7005 | Variable implicitly has 'any' type | Add explicit type annotation |
| TS7006 | Parameter implicitly has 'any' type | Add parameter type |
| TS7008 | Member implicitly has 'any' type | Add property type |
| TS7010 | Function lacks return type annotation | Add return type |
| TS7015 | Element implicitly has 'any' type (index) | Add index signature |
| TS7016 | Could not find declaration file | Install @types/pkg or declare module |
| TS7017 | Element has 'any' type (object index) | Add proper type or index signature |
| TS7019 | Rest parameter implicitly has 'any' type | Type the rest parameter |
| TS7031 | Binding element implicitly has 'any' | Destructure with types |
| TS7034 | Variable implicitly has 'any[]' type | Add array type annotation |

#### Strict Null/Undefined Checks (TS2531-2533)

| Code | Description | Quick Fix |
|------|-------------|-----------|
| TS2531 | Object is possibly 'null' | Add null check or assertion |
| TS2532 | Object is possibly 'undefined' | Add undefined check or assertion |
| TS2533 | Object is possibly 'null' or 'undefined' | Add nullish check |
| TS18047 | 'x' is possibly 'null' | Null guard |
| TS18048 | 'x' is possibly 'undefined' | Undefined guard |

#### Property/Member Issues

| Code | Description | Quick Fix |
|------|-------------|-----------|
| TS2339 | Property does not exist on type | Add property to interface or use correct type |
| TS2551 | Property does not exist (did you mean?) | Fix typo in property name |
| TS2352 | Conversion may be mistake | Add type assertion with justification |
| TS2564 | Property not definitely assigned | Initialize or add definite assignment assertion |

#### Assignability Issues

| Code | Description | Quick Fix |
|------|-------------|-----------|
| TS2322 | Type 'X' is not assignable to type 'Y' | Correct the value or widen the type |
| TS2345 | Argument type not assignable to parameter | Fix argument type |
| TS2741 | Property missing in type | Add missing property |
| TS2769 | No overload matches this call | Fix arguments to match overload |

#### Function Issues

| Code | Description | Quick Fix |
|------|-------------|-----------|
| TS2554 | Expected N arguments, got M | Add/remove arguments |
| TS2555 | Expected at least N arguments | Add required arguments |
| TS2556 | Spread argument must have tuple type | Type the spread properly |
| TS2571 | Object is of type 'unknown' | Add type guard or assertion |

### Category 2: Design Issues

Errors indicating fundamental type architecture problems requiring design decisions.

#### Missing Type Definitions

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS2304 | Cannot find name 'X' | Missing type definition - needs type design |
| TS2305 | Module has no exported member | API boundary mismatch - needs interface alignment |
| TS2307 | Cannot find module | Module structure issue |

#### Generic/Constraint Problems

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS2314 | Generic type requires type arguments | Generic constraints underspecified |
| TS2315 | Type 'X' is not generic | Type system misunderstanding |
| TS2344 | Type does not satisfy constraint | Generic constraint design issue |

#### Type Incompatibility

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS2349 | Cannot invoke expression | Callable type design needed |
| TS2365 | Operator cannot be applied | Type algebra issue |
| TS2559 | Type has no properties in common | Completely incompatible types - design mismatch |
| TS2739 | Type missing properties from mapped type | Incomplete implementation of contract |
| TS2740 | Type missing props from literal type | Interface implementation incomplete |
| TS2790 | Index signature for type incompatible | Structural type design issue |

#### Inheritance/Contract Violations

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS2395 | Duplicate index signature | Conflicting type definitions |
| TS2416 | Property in type not assignable to same in base | Covariance/contravariance issue |
| TS2417 | Class static side incorrectly extends base | Static interface design |
| TS2420 | Class incorrectly implements interface | Contract violation |
| TS2430 | Interface incorrectly extends interface | Inheritance design issue |
| TS2610 | Property defined in base must be marked override | Inheritance contract |
| TS2684 | 'this' context wrong | Class/method design issue |

#### Type Complexity

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS2589 | Type instantiation is excessively deep | Over-complex generics |
| TS2590 | Expression produces union too complex | Type explosion |
| TS2775 | Assertions require unused identifier | Type guard design |
| TS2794 | Expected N type arguments, got M | Generic parameter mismatch |
| TS2862 | Property does not exist on nullable type | Union type narrowing needed |

#### API Boundary Issues

| Code | Description | Design Issue |
|------|-------------|--------------|
| TS4060 | Return type of exported function has/uses private name | API boundary issue |
| TS4073 | Parameter of public method from exported class | Visibility/encapsulation design |

## Root Cause Analysis Patterns

When encountering design issues, identify these common root causes:

| Pattern | Indicators | Analysis Focus |
|---------|------------|----------------|
| **Missing Domain Types** | TS2304, widespread TS7xxx | Types were never designed for this domain |
| **Any Escape Hatch** | `any` casts, TS7xxx clusters | Types exist but were bypassed |
| **Interface Drift** | TS2559, TS2739 | API changed without type update |
| **Generic Overengineering** | TS2589, TS2590 | Type system pushed beyond limits |
| **Module Boundary Leak** | TS2305, TS2307 | Internal types exposed incorrectly |
| **Inheritance Mismatch** | TS2420, TS2430 | Wrong abstraction hierarchy |
| **Covariance Violation** | TS2416, TS2417 | Substitution principle broken |

## Heuristic Rules

Beyond error codes, apply these heuristics:

1. **Error Clustering**: 5+ errors in same file → Elevate to Design Issue
2. **Type 'any' Escape**: Multiple TS7xxx in module → Types were never designed
3. **Interface Incompatibility**: TS2559 → Fundamental mismatch, not fixable with assertions
4. **Generic Complexity**: TS2589/TS2590 → Type system abuse, needs simplification
5. **Module Boundary Errors**: TS2305/TS2307 clustering → Architectural boundary issues

## Health Score Calculation

```
Score = 100 - (Quick Fix Errors × 1) - (Design Issues × 5)

Score >= 90: Healthy
Score 70-89: Needs Attention
Score 50-69: Poor
Score < 50: Critical
```

## Error Density Metric

```
Density = Total Errors / Lines of TypeScript Code × 1000

Density < 1: Excellent
Density 1-5: Good
Density 5-10: Concerning
Density > 10: Critical
```

## Severity Matrix

| Indicator | Severity | Rationale |
|-----------|----------|-----------|
| Error blocks build/runtime | 🔴 Critical | Immediate fix required |
| Design Issue in new code | 🔴 Critical | Fix before merging |
| Quick Fix in critical path | 🟡 High | Fix soon |
| 10+ errors same file | 🔴 Critical | Systemic issue |
| Quick Fix in non-critical | 🟡 Medium | Technical debt |
| Design Issue in legacy | 🟡 Medium | Plan refactoring |
| Quick Fix in tests | 🟢 Low | Lower priority |
