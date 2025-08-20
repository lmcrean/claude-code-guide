# Claude Code Issue Workflow Guide

## Quick Reference
```
1. Fetch & document issues          → .notes/{issue}/issue-summary.md
   → HUMAN CHECKPOINT: select best issue
2. Plan implementation             → implementation-plan.md  
3. Reproduce the issue             → testing/reproduce-issue.{ext}
4. Assess viability               → viability-assessment.md
   → HUMAN CHECKPOINT: proceed?
5. Break into phases              → phases/phase-N-diff.md
6. Review implementation plan     → validate approach
7. Implement phase by phase       → test & validate each phase
   → HUMAN CHECKPOINT: complete?
8. Draft pull request            → pr-draft.md
```

## File Organization Structure

```
.notes/{issue_number}/
├── issue-summary.md           # Captured issue details
├── implementation-plan.md     # High-level technical approach
├── viability-assessment.md    # Complexity, impact, time estimates
├── phases/                    # Implementation phases
│   ├── phase-1-diff.md       
│   ├── phase-2-diff.md       
│   └── ...
├── testing/                   # Test files and validation
│   ├── reproduce-issue.{ts,mjs,py}
│   ├── test-{feature}.{ts,mjs,py}
│   └── validation-results.md
└── pr-draft.md               # Final pull request draft
```

## Detailed Workflow Steps

### 1.1 Issue List retrieval
- Navigate to `.notes/` directory
- Use GitHub API/tools to fetch recent issues
- Fetch a list of recent issues on a list with Issue title; labels; comment count; date posted
- Fetch 100 most recent issues in `.notes/issues_recent_100/{issue number}.md` with metadata at the top (linked PR's, labels) issue description and comments at the bottom/

Human selects issue at this point. From here on we work in `.notes/{issue_number}/`

### 1.2 Issue Selection & Documentation
- Navigate to `.notes/` directory
- Use GitHub API/tools to Create `.notes/{issue_number}/issue-summary.md`
- Capture: title, description, comments, linked PRs, labels, metadata

### 2. Implementation Planning  
- Create `implementation-plan.md`
- Analyze affected codebase components
- Design solution approach with alternatives
- Identify risks and dependencies

### 3. Issue Reproduction
- Create reproduction script in `testing/reproduce-issue.{ext}`
- Reproduce via unit tests OR manual terminal commands
- Document steps and results
- Validate issue exists and understand scope

### 4. Viability Assessment
- Create `viability-assessment.md`
- Evaluate: Impact (High/Med/Low), Complexity, Risk, Time estimate
- **→ CHECKPOINT: Human dev decides whether to proceed**

### 5. Detailed Phase Planning
- Create numbered phase files in `phases/` directory
- Each phase should be independently testable
- Include acceptance criteria and test commands
- Design for minimal breaking changes

### 6. Implementation Review
- Review phases for logic and completeness
- Validate test strategy
- Check risk mitigation
- **→ CHECKPOINT: Review diffs and validate approach**

### 7. Phase-by-Phase Implementation
- Implement one phase at a time
- Run tests after each phase
- Document deviations and adjustments
- Update validation results
- **→ CHECKPOINT: Human dev review once complete**

### 8. Pull Request Drafting
- Record the final diff in `final-changes.diff` so you understand the changes to the codebase
- Create `pr-draft.md` with complete PR content
- Include: Problem statement, solution overview, implementation details any important test results, guide for admin to validate
- Add testing info, breaking changes, checklist
- Make it comprehensive and reviewable


Try and avoid verbosity in the description for example like this minimal 4-part structure:

```
Fixes #1474 - completion now works when typing partial subphrases like "open" for cached entries like "open webpage".

## Problem
When cache contains multi-part subphrases (e.g., "open webpage" → "browser.openWebPage"), typing just "open" returns no completions.

## Solution
Added prefix-based transform indexing to enable partial subphrase completion:

- **transforms.ts**: Added prefix indexing for compound keys and `getByPrefix()` method
- **store.ts**: Added `getTransformsByPrefix()` API for cross-namespace prefix search  
- **requestCompletion.ts**: Integrated prefix completions into completion flow

## Testing
1. Execute: `open bbc webpage` (populates cache)
2. Type: `open` + TAB
3. Expected: Shows "open webpage" completion

Works for left-to-right partial prefixes of cached entries (e.g., if "open bbc webpage" is cached, typing "open" completes to "open bbc webpage", if "play music from spotify" is cached, typing "play music" completes to "play music from spotify").

Gracefully falls back to existing completion behavior when no cached entries match the prefix.

## Breaking Changes
None. Purely additive enhancement with minimal memory overhead from prefix indexing.
```
