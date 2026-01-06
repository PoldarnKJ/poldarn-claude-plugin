---
name: Deep Investigation Methodology
description: This skill should be used when encountering repeated fix failures, framework/library/platform-specific issues, or problems where training data may be outdated. Activates on patterns like "fix failed again", "still not working", "library error", "framework issue", "version mismatch", "deprecated API", "breaking change". Provides systematic research methodology to overcome LLM knowledge limitations through external source verification.
version: 1.0.0
model: opus
---

# Deep Investigation Methodology

This methodology addresses a critical limitation: LLM training data becomes outdated for rapidly evolving libraries, frameworks, and platforms. When fixes fail repeatedly, the issue is often **knowledge gaps**, not reasoning failures.

## Intervention Triggers

Activate this methodology immediately when:

1. **Repeated Fix Failures**: Same or similar error persists after 2+ fix attempts
2. **Framework/Library Issues**: Problems involving external dependencies, not business logic
3. **Version-Specific Behavior**: Errors mentioning versions, deprecations, or breaking changes
4. **API/Configuration Errors**: Issues with third-party service integration
5. **Platform Quirks**: OS-specific, browser-specific, or runtime-specific behavior

## Core Principle: Breadth-First Knowledge Acquisition

**CRITICAL**: Do NOT stop searching after finding one potential solution. LLM confidence in outdated knowledge creates dangerous false positives. Exhaustive search is mandatory.

## Five-Phase Investigation Protocol

### Phase 1: Problem Crystallization

Before any external search, crystallize the problem:

1. **Extract Core Symptoms**
   - Exact error message (preserve verbatim)
   - Stack trace key frames
   - Conditions that trigger the issue

2. **Identify Technology Stack**
   - Library/framework name and exact version
   - Runtime environment (Node version, browser, OS)
   - Related dependencies and their versions

3. **Formulate Search Keywords**
   - Primary: `[library] [version] [error message key phrase]`
   - Secondary: `[library] [symptom description]`
   - Tertiary: `[library] breaking changes [version]`

**Output**: Structured problem summary with 3-5 search query variations

### Phase 2: Exhaustive Source Gathering

Search ALL of the following sources. Do not skip any source even if an earlier source seems to provide an answer.

#### 2.1 Official Documentation (Priority 1)
- Use Context7 MCP for version-specific documentation
- Query pattern: Library ID with version (e.g., `/vercel/next.js/v14.0.0`)
- Focus on: migration guides, breaking changes, changelog

#### 2.2 GitHub Issues (Priority 2)
- Search: `site:github.com [library]/[library] [error keywords]`
- Filter: Issues from the past 12 months
- Look for: Issue labels (bug, confirmed), maintainer responses
- Check: Linked PRs, workarounds in comments

#### 2.3 Stack Overflow (Priority 3)
- Search: `site:stackoverflow.com [library] [error]`
- Filter: Sort by newest, check answers from 2024-2025
- Prioritize: Accepted answers, high-vote answers, answers from recognized contributors

#### 2.4 Release Notes & Changelogs (Priority 4)
- Check releases around the version in use
- Search: `[library] changelog`, `[library] release notes [version]`
- Look for: Breaking changes, deprecation notices, migration paths

#### 2.5 Community Resources (Priority 5)
- Discord/Slack official channels (if searchable)
- Blog posts from library maintainers
- Conference talks and announcements

**Mandatory Breadth Rule**: Collect at least 5-7 distinct sources before analysis. Finding one "solution" does NOT permit skipping remaining sources.

### Phase 3: Evidence Correlation

After gathering sources, perform correlation analysis:

1. **Consistency Check**
   - Do multiple sources agree on the cause?
   - Are there conflicting explanations?
   - What is the consensus among maintainers vs. community?

2. **Recency Validation**
   - When was each source published?
   - Is the advice still valid for current version?
   - Have there been updates since the source was written?

3. **Authority Assessment**
   - Official documentation > Maintainer comments > Community answers
   - Verified fixes (merged PRs) > Proposed workarounds
   - Multiple confirming sources > Single source

4. **Version Alignment**
   - Does the solution apply to the exact version in use?
   - Are there version-specific caveats?

### Phase 4: Hypothesis Synthesis

Based on correlated evidence, formulate hypotheses:

1. **Primary Hypothesis**
   - Most strongly supported by evidence
   - Consistent across multiple sources
   - Matches version and environment

2. **Alternative Hypotheses**
   - Other plausible explanations
   - Edge cases mentioned in sources
   - Related issues that might compound

3. **Confidence Assessment**
   - High: Multiple official sources confirm
   - Medium: Community consensus with some official support
   - Low: Single source or conflicting information

### Phase 5: Report Generation

Generate a comprehensive investigation report BEFORE any code changes:

```markdown
## Deep Investigation Report

### Problem Summary
- **Error**: [Exact error message]
- **Technology**: [Library/framework] v[version]
- **Environment**: [Runtime details]
- **Trigger**: [What causes the issue]

### Search Keywords Used
1. [Query 1]
2. [Query 2]
3. [Query 3]

### Sources Consulted

#### Official Documentation
- [Source 1]: [Key finding]
- [Source 2]: [Key finding]

#### GitHub Issues
- [Issue #N]: [Status, key points, maintainer response]
- [Issue #M]: [Status, key points]

#### Stack Overflow
- [Question link]: [Relevance, answer quality]

#### Release Notes
- [Version]: [Relevant changes]

### Evidence Correlation
| Finding | Sources | Consistency | Recency |
|---------|---------|-------------|---------|
| [Finding 1] | [n sources] | High/Med/Low | [Date range] |
| [Finding 2] | [n sources] | High/Med/Low | [Date range] |

### Hypotheses

#### Primary Hypothesis
**Cause**: [Detailed explanation]
**Evidence**: [Supporting sources]
**Confidence**: High/Medium/Low
**Proposed Solution**: [Description without code]

#### Alternative Hypotheses
1. **[Alternative 1]**: [Explanation, evidence, confidence]
2. **[Alternative 2]**: [Explanation, evidence, confidence]

### Recommendations
- [Recommended action 1]
- [Recommended action 2]

### Uncertainties & Risks
- [Unknown 1]
- [Potential risk 1]
```

## Critical Rules

### DO NOT
- Propose code changes before completing the report
- Stop searching after finding one potential solution
- Rely on training data for library/framework behavior
- Assume solutions from older versions still apply
- Skip sources because early results seem definitive

### ALWAYS
- Enable deepest thinking mode for analysis
- Gather 5+ independent sources minimum
- Cross-reference version compatibility
- Present report to user before any implementation
- Document confidence levels explicitly
- List what remains unknown or uncertain

## Quick Reference

| Phase | Key Action | Output |
|-------|------------|--------|
| 1. Crystallize | Extract symptoms, versions, keywords | Search queries |
| 2. Gather | Search ALL source categories | Raw findings |
| 3. Correlate | Cross-reference, validate, assess | Weighted evidence |
| 4. Synthesize | Form hypotheses with confidence | Ranked explanations |
| 5. Report | Document everything | Investigation report |

## Integration with Context7 MCP

For library documentation, use Context7 MCP tools:

1. **Resolve library ID**: `resolve-library-id` with library name
2. **Get versioned docs**: `get-library-docs` with specific version ID
3. **Version format**: `/org/project/version` (e.g., `/vercel/next.js/v14.0.0`)

This provides up-to-date, version-specific documentation that training data cannot match.
