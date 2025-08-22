# Enterprise Repository Agent System

A comprehensive 4-agent system for managing enterprise repository tickets with structured workflows and unified data management.

## Quick Start

```bash
# 1. Copy template for new ticket
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/1234/i1

# 2. Run full workflow
claude --agent pull-agent "Extract issue #1234 data"
claude --agent plan-agent "Create implementation plan" 
claude --agent review-agent "Review plan viability"
claude --agent hardcode-agent "Implement solution"
claude --agent review-agent "Final review"
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
└── review/                # ✅ Quality assurance
    ├── solution-review.md
    ├── implementation-review.md
    └── diff-analysis.md
```

## Key Principles

1. **pull/ as Source of Truth**: All definitive data lives in the pull/ directory
2. **Sequential Workflow**: Agents work in sequence with clear handoffs
3. **Quality Gates**: review-agent provides approval checkpoints
4. **Template-Based**: Copy structure for each enterprise ticket
5. **No Interpretation in Extraction**: pull-agent only extracts, never interprets

## Documentation

- [Agent Specifications](./pull-agent.md) - Detailed specs for each agent
- [Interaction Protocols](./agent-interaction-protocols.md) - How agents coordinate
- [Usage Examples](./usage-examples.md) - Real workflow examples and troubleshooting
- [Template Structure](./template/) - Copy this for each new ticket

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
pull-agent → plan-agent → review-agent → hardcode-agent
     ↑                                           │
     └───────── Final State Updates ─────────────┘
```

- **pull-agent**: Maintains all sources of truth
- **plan-agent**: Reads pull/, creates plans in plan/
- **review-agent**: Reviews pull/ and plan/, provides quality gates
- **hardcode-agent**: Reads plan/, implements code, updates pull/code/final/

## Success Metrics

✅ **Individual Agent Success**
- pull-agent: All required data extracted and organized
- plan-agent: Comprehensive, actionable implementation plan  
- review-agent: Clear go/no-go decisions with rationale
- hardcode-agent: Working implementation passing all tests

✅ **System Success**
- Seamless data flow between agents
- Clear audit trail of decisions
- Successful issue resolution
- Reusable process for enterprise tickets