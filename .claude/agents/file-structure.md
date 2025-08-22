# Enterprise Agent File Structure

## Overview
This document defines the file structure that all enterprise agents must follow when working with GitHub issues and repository tickets.

## Core Structure Pattern
All agent work is organized under the `.notes/{issue}{iteration}/` directory structure:

```
.notes/
└── {issue}{iteration}/           # e.g., "123a", "456b", "789c"
    ├── pull/                     # Data extraction (pull-agent)
    │   ├── github/
    │   │   ├── issue.md
    │   │   ├── pr.md
    │   │   ├── comments.md
    │   │   └── pr-diffs/
    │   └── code/
    │       ├── pushed.diff
    │       ├── staged.diff
    │       ├── unstaged.diff
    │       └── testing/
    ├── plan/                     # Strategic planning (plan-agent)
    │   ├── github/
    │   │   ├── pr-draft.md
    │   │   └── comment-responses.md
    │   └── code/
    │       ├── implementation-plan.md
    │       ├── viability-assessment.md
    │       └── phases/
    │           ├── phases.md
    │           ├── phase1.diff
    │           ├── phase2.diff
    │           └── phase3.diff
    └── review/                   # Quality assurance (review-agent)
        ├── solution-review.md
        ├── implementation-review.md
        ├── diff-analysis.md
        └── viability-report.md
```

## Issue/Iteration Naming Convention
- **{issue}**: GitHub issue number (e.g., "123", "456")
- **{iteration}**: Alphabetic iteration marker (e.g., "a", "b", "c")
- **Examples**: `123a`, `456b`, `789c`

## Agent Path Resolution
All agents must determine the current working context and use the full `.notes/{issue}{iteration}/` path prefix.

### Determining Current Context
Agents should look for context in this priority order:
1. **Explicit parameter**: Human provides issue/iteration explicitly
2. **Current directory**: Check if already in a `.notes/{issue}{iteration}/` directory
3. **Recent directories**: Use most recent `.notes/{issue}{iteration}/` directory
4. **Prompt user**: Request issue/iteration if cannot be determined

### Path Construction Examples
```bash
# Correct paths
.notes/123a/pull/github/issue.md
.notes/123a/plan/code/implementation-plan.md
.notes/123a/review/solution-review.md

# WRONG - Never use bare directory names
pull/github/issue.md          # Missing .notes/{issue}{iteration}/
plan/code/phases/phases.md    # Missing .notes/{issue}{iteration}/
```

## Agent-Specific Responsibilities

### pull-agent
- Creates and maintains `.notes/{issue}{iteration}/pull/` directory
- Extracts all raw data from GitHub and git
- No interpretation, only data organization

### plan-agent
- Creates and maintains `.notes/{issue}{iteration}/plan/` directory
- Reads from `.notes/{issue}{iteration}/pull/` for context
- Creates .md and .diff files within plan/ structure

### review-agent
- Creates and maintains `.notes/{issue}{iteration}/review/` directory
- Reads from both `.notes/{issue}{iteration}/pull/` and `.notes/{issue}{iteration}/plan/`
- Provides quality assurance and go/no-go decisions

### hardcode-agent
- Reads from `.notes/{issue}{iteration}/plan/` for implementation guidance
- Updates `.notes/{issue}{iteration}/pull/code/pushed.diff` with implemented changes
- Does not create its own directory structure

## Directory Creation Rules
1. **Always use absolute paths** starting with `.notes/{issue}{iteration}/`
2. **Create parent directories** as needed with `mkdir -p`
3. **Validate context** before creating any directories
4. **Never create directories** at project root level (except .notes/ itself)

## Error Prevention
Common mistakes to avoid:
- Creating `pull/`, `plan/`, `review/` at project root
- Using relative paths without `.notes/{issue}{iteration}/` prefix
- Working in wrong iteration directory
- Creating files outside the established structure

## Integration with Version Control
The `.notes/` directory is tracked in git to preserve:
- Audit trail of decisions and implementations
- Team collaboration artifacts
- Historical context for future iterations
- Knowledge sharing and process improvement

See `.claude/agents/configuration/.gitignore-requirements.md` for version control setup.