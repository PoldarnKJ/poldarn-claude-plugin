# Poldarn

A Claude Code plugin that enforces code quality principles, provides deep investigation capabilities, and detects hidden technical debt.

## Features

### Skills

| Skill | Description |
|-------|-------------|
| **Code Quality Principles** | Architectural guidance for single responsibility, modular design, type-first development, functional programming style, and pragmatic library adoption |
| **Deep Investigation** | Systematic research methodology for resolving issues where LLM training data may be outdated |
| **Fake Implementation Detection** | Detection patterns for behavioral mimicry - code that appears complete but performs no real work |
| **TypeScript Type Scanner** | Guidance on TypeScript error classification and type health patterns |
| **Vue Query Patterns** | Best practices for @tanstack/vue-query in Vue 3 applications |

### Agents

| Agent | Description |
|-------|-------------|
| **code-quality-review** | Performs code quality review focusing on architectural principles and functional programming style |
| **codex-reviewer** | Deep code review with Codex assistance for functional correctness and regression detection |
| **deep-investigation** | Exhaustive external research for library/framework issues where training data may be outdated |
| **fake-implementation-detector** | Scans codebases for hollow implementations and placeholder code |
| **typescript-type-scanner** | Scans TypeScript projects and generates categorized health reports |

### Commands

| Command | Description |
|---------|-------------|
| `/poldarn:ts-scan` | Scan TypeScript project for type errors and generate categorized health report |

## Installation

### From GitHub

```bash
claude plugin marketplace add PoldarnKJ/poldarn-claude-plugin
claude plugin install poldarn
```

### Manual Installation

```bash
git clone https://github.com/PoldarnKJ/poldarn-claude-plugin.git
claude plugin marketplace add ./poldarn-claude-plugin
claude plugin install poldarn
```

## Usage

### Code Quality Review

The plugin automatically provides architectural guidance when implementing features:

```
User: "Implement a user authentication module"
Claude: [Applies code quality principles: SRP, modular architecture, type-first development]
```

### TypeScript Health Scan

```
User: "/poldarn:ts-scan"
Claude: [Runs type check, categorizes errors, generates health report]
```

### Deep Investigation

When fixes fail repeatedly, the deep investigation agent activates:

```
User: "I've tried 3 solutions but still getting this Next.js error"
Claude: [Performs exhaustive research across official docs, GitHub issues, Stack Overflow]
```

### Fake Implementation Detection

```
User: "Check this codebase for fake implementations"
Claude: [Scans for hollow code patterns, placeholder implementations]
```

## Core Principles

### 1. Single Responsibility & Separation of Concerns
- One function, one purpose
- One hook per concern
- Components render UI; business logic belongs in hooks or services

### 2. Pragmatic Library Adoption
- Search for existing solutions before implementing generic functionality
- Evaluate: community adoption, maintenance, TypeScript support, bundle size

### 3. Modular Architecture
- Feature-based organization over technical layering
- High cohesion, low coupling
- Export through barrel files to define public API

### 4. Type-First Development
- Types as specifications
- Discriminated unions over optional properties
- Branded types for domain identifiers

### 5. Functional Programming Style
- Pure functions, immutability
- Function composition
- Functions as primary unit of abstraction

## Requirements

- Claude Code CLI
- For deep-investigation agent: Context7 MCP server (optional, for versioned documentation)
- For codex-reviewer agent: Codex MCP server (optional)

## License

MIT
