# Enterprise Repository Agent Template

This template directory structure should be copied for each new enterprise ticket.

## Usage
```bash
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/{actual_issue_number}/{actual_iteration}
```

## Directory Structure

```
.notes/{issue_number}/{iteration}/
├── pull/                   # ALL sources of truth (maintained by pull-agent)
│   ├── github/            # GitHub data
│   │   ├── issue.md       # Issue details
│   │   ├── pr.md          # PR description
│   │   ├── comments.md    # All comments
│   │   └── pr-diffs/      # PR diffs from GitHub
│   └── code/              # Code state tracking
│       ├── pushed/        # What's in PR/remote (changes.diff)
│       ├── staged/        # What's staged locally (staged.diff)
│       ├── unstaged/      # Modified but not staged (unstaged.diff)
│       └── final/         # Final implemented changes (final.diff)
├── plan/                  # plan-agent planning outputs
│   ├── github/            # GitHub responses
│   │   ├── pr-draft.md
│   │   └── comment-responses.md
│   └── code/              # Implementation plans
│       ├── implementation-plan.md
│       ├── phases/
│       └── viability-assessment.md
└── review/                # review-agent analysis outputs
    ├── solution-review.md
    ├── implementation-review.md
    └── diff-analysis.md
```

## Agent Workflow
1. **pull-agent**: Populates `pull/` with all sources of truth
2. **plan-agent**: Creates plans in `plan/` referencing `pull/` data
3. **review-agent**: Reviews solutions in `review/` based on `pull/` data
4. **hardcode-agent**: Implements code, saves final state back to `pull/code/final/`