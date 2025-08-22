# Enterprise Repository Agent System

A comprehensive 4-agent system for managing enterprise repository tickets with structured workflows and unified data management.

## Quick Start

```bash
# 0. Configure repository (one-time human setup)
cp agents/configuration/settings.local.json ./settings.local.json
# Human manually adds .gitignore entries (agents cannot modify .gitignore)
cat agents/configuration/.gitignore-requirements.md >> .gitignore

# 1. Copy template for new ticket
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/1234/i1
cd .notes/1234/i1

# 2. Run agents manually in sequence (human controls each step)
claude --agent pull-agent "Extract issue #1234 with all GitHub data and current code state"
# Check: pull/ directory populated with issue.md, comments.md, code diffs

claude --agent plan-agent "Analyze issue and create detailed implementation plan with exact phase diffs" 
# Check: plan/ directory has implementation-plan.md and phases/ with phase1.diff, phase2.diff, etc.

claude --agent review-agent "Review implementation plan for technical soundness and viability"
# Check: review/ directory has solution-review.md with go/no-go decision

# Only proceed if review approved
claude --agent hardcode-agent "Apply phase1.diff, validate tests pass, then phase2.diff, validate, then phase3.diff"
# Check: pull/code/final/final.diff created with complete implementation

claude --agent review-agent "Final review - verify implementation meets all original requirements"
# Check: review/ directory has implementation-review.md with final approval/rejection
```

## Agent Overview

| Agent | Purpose | Input | Output |
|-------|---------|--------|---------|
| **pull-agent** | Data extraction | GitHub issue/PR | `pull/` directory (source of truth) |
| **plan-agent** | Strategic planning | `pull/` data | `plan/` directory (implementation strategy) |
| **review-agent** | Quality assurance | `pull/` + `plan/` data | `review/` directory (go/no-go decisions) |
| **hardcode-agent** | Code implementation | `plan/` phases | Updates `pull/code/final/` (implemented code) |

## Directory Structure

```
.notes/{issue_number}/{iteration}/
├── pull/                   # 🏢 SINGLE SOURCE OF TRUTH
│   ├── github/            # GitHub data (issues, PRs, comments)
│   └── code/              # Code state tracking
│       ├── pushed/        # Remote/PR changes
│       ├── staged/        # Staged changes
│       ├── unstaged/      # Modified but unstaged
│       └── final/         # Final implementation
├── plan/                  # 📋 Strategic planning
│   ├── github/            # PR drafts, response plans
│   └── code/              # Implementation plans, phases
│       ├── implementation-plan.md
│       ├── viability-assessment.md  
│       └── phases/
│           ├── phases.md      # Phase overview and validation strategy
│           ├── phase1.diff    # Exact diff for phase 1
│           ├── phase2.diff    # Exact diff for phase 2
│           └── phase3.diff    # Exact diff for phase 3
└── review/                # ✅ Quality assurance
    ├── solution-review.md
    ├── implementation-review.md
    └── diff-analysis.md
```

## Key Principles

1. **pull/ as Source of Truth**: All definitive data lives in the pull/ directory
2. **Human-Controlled Workflow**: You manually run each agent in sequence
3. **Quality Gates**: review-agent provides approval checkpoints  
4. **Template-Based**: Copy structure for each enterprise ticket
5. **No Interpretation in Extraction**: pull-agent only extracts, never interprets
6. **File-Based Validation**: Agents check required input files exist before proceeding

## Documentation

- [Agent Specifications](./pull-agent.md) - Detailed specs for each agent
- [Interaction Protocols](./agent-interaction-protocols.md) - How agents coordinate
- [Usage Examples](./usage-examples.md) - Real workflow examples and troubleshooting
- [Template Structure](./template/) - Copy this for each new ticket
- [Configuration Requirements](./configuration/) - Required .gitignore and settings.local.json

## Workflow Example

```bash
# Enterprise ticket #1234: "Search autocomplete fails for partial words"

# 1. Setup ticket workspace
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/1234/i1
cd .notes/1234/i1

# 2. Extract all data (issues, PRs, current code state)
claude --agent pull-agent "Extract issue #1234 data and current state"
# Creates: pull/github/issue.md, pull/code/{pushed,staged,unstaged}/

# 3. Create strategic implementation plan
claude --agent plan-agent "Plan implementation for search autocomplete fix"
# Creates: plan/code/implementation-plan.md, plan/code/phases/

# 4. Review plan for viability
claude --agent review-agent "Review implementation plan"
# Creates: review/solution-review.md (✅ approved / ❌ needs revision)

# 5. Implement the solution phase by phase
claude --agent hardcode-agent "Implement autocomplete fix following phases"
# Updates: pull/code/final/final.diff, pull/code/testing/

# 6. Final quality review
claude --agent review-agent "Final review of implementation"
# Creates: review/implementation-review.md (✅ ready / ❌ needs fixes)
```

## Agent Communication Flow

```
Human → pull-agent → Human → plan-agent → Human → review-agent → Human → hardcode-agent → Human
         ↓                     ↓                      ↓                      ↓
      pull/ data           plan/ data            review/ data        pull/code/final/
```

- **Human Control**: You decide when to run each agent and review outputs
- **pull-agent**: Extracts and maintains all sources of truth
- **plan-agent**: Reads pull/, creates implementation plans in plan/
- **review-agent**: Reviews pull/ and plan/, provides go/no-go decisions
- **hardcode-agent**: Reads plan/, implements code, updates pull/code/final/

## Example Agent Prompts

### pull-agent Examples
```bash
# Basic issue extraction
claude --agent pull-agent "Extract issue #1234 with all GitHub data"

# Include current code state  
claude --agent pull-agent "Extract issue #5678 data and capture current git state - staged, unstaged, and remote changes"

# Focus on specific PR
claude --agent pull-agent "Pull data for issue #999 including linked PR #123 with all diffs and review comments"

# Update existing data
claude --agent pull-agent "Refresh all pull/ data for issue #1234 - check for new comments and code changes"
```

### plan-agent Examples  
```bash
# Standard planning
claude --agent plan-agent "Analyze the extracted issue data and create comprehensive implementation plan with phase diffs"

# Focus on specific aspect
claude --agent plan-agent "Create implementation plan focusing on database changes for the user authentication issue"

# Alternative approaches
claude --agent plan-agent "Plan implementation with 2-3 alternative approaches and recommend the best one"

# GitHub response planning
claude --agent plan-agent "Create implementation plan and draft GitHub PR description following repository conventions"
```

### review-agent Examples
```bash
# Plan review
claude --agent review-agent "Review implementation plan for technical soundness, risks, and completeness"

# Implementation review
claude --agent review-agent "Final review of implementation - verify all requirements met and code quality acceptable"

# Security focused review
claude --agent review-agent "Review implementation focusing on security implications and potential vulnerabilities"

# Performance review
claude --agent review-agent "Review implementation plan with emphasis on performance impact and scalability"
```

### hardcode-agent Examples
```bash
# Standard implementation
claude --agent hardcode-agent "Implement solution following phase diffs - apply phase1.diff, validate, then phase2.diff, validate, then phase3.diff"

# Focus on specific phase
claude --agent hardcode-agent "Implement only phase 1 of the plan - setup and foundation changes with full testing"

# Implementation with testing focus
claude --agent hardcode-agent "Implement complete solution with comprehensive test coverage and validation at each phase"

# Cautious implementation
claude --agent hardcode-agent "Implement solution incrementally, running full test suite after each phase before proceeding"
```

## Success Metrics

✅ **Individual Agent Success**
- pull-agent: All required data extracted and organized
- plan-agent: Comprehensive, actionable implementation plan  
- review-agent: Clear go/no-go decisions with rationale
- hardcode-agent: Working implementation passing all tests

✅ **System Success**
- Human maintains full control over workflow progression
- Clear audit trail of decisions in directory structure
- Successful issue resolution with quality gates
- Reusable process for enterprise tickets