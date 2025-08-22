# Enterprise Agent File Structure

## Overview
This document defines the file structure that all enterprise agents must follow when working with GitHub issues and repository tickets.

## Core Structure Pattern
All agent work is organized under the `.notes/{issue}{iteration}/` directory structure:

```
.notes/
└── {issue}/
    └── {iteration}/           # e.g., "i1", "i2", "i2"
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
- **{iteration}**: Alphabetic iteration marker (e.g., "i1,i2,i3,i4")