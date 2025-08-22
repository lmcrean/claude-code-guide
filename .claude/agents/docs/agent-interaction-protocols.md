# Agent Interaction Protocols

## Overview
This document defines how the four enterprise repository agents interact, share data, and coordinate workflows while maintaining the pull/ directory as the single source of truth.

## Agent Relationship Map

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ pull-agent  │    │ plan-agent  │    │review-agent │    │hardcode-    │
│             │───▶│             │───▶│             │───▶│agent        │
│ Data        │    │ Planning    │    │ Quality     │    │ Implement   │
│ Extraction  │    │ Strategy    │    │ Assurance   │    │ Execute     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       ▲                                                           │
       │                                                           │
       └───────────────────────────────────────────────────────────┘
                      Final State Updates
```

## Data Flow Architecture

### Primary Data Flow (Sequential)
1. **pull-agent** → extracts and organizes all raw data → `pull/`
2. **plan-agent** → reads `pull/`, creates plans → `plan/`
3. **review-agent** → reads `pull/` and `plan/`, reviews solution → `review/`
4. **hardcode-agent** → reads `plan/`, implements code → updates `pull/code/pushed.diff`

### Secondary Data Flow (Feedback Loop)
- **hardcode-agent** → updates implementation state → `pull/code/pushed.diff`
- **pull-agent** → tracks ongoing state changes → `pull/code/staged/`, `pull/code/unstaged/`

## Directory Ownership and Access Rights

### pull/ Directory (Single Source of Truth)
**Owner**: pull-agent (primary maintainer)
**Write Access**: pull-agent, hardcode-agent (final state only)
**Read Access**: All agents

```
pull/
├── github/          # pull-agent writes, all agents read
├── code/
│   ├── pushed.diff  # pull-agent writes and maintains
│   ├── staged.diff  # pull-agent and hardcode-agent write
│   └── unstaged.diff # pull-agent and hardcode-agent write
```

### plan/ Directory
**Owner**: plan-agent
**Write Access**: plan-agent only
**Read Access**: review-agent, hardcode-agent

### review/ Directory
**Owner**: review-agent
**Write Access**: review-agent only
**Read Access**: All agents (for feedback)

## Agent Coordination Workflows

### 1. Initial Data Extraction (pull-agent)
```bash
# pull-agent responsibilities
1. Extract GitHub issue → pull/github/issue.md
2. Extract PR data → pull/github/pr.md, pull/github/comments.md
3. Capture current diffs → pull/code/{pushed,staged,unstaged}.diff
4. Signal completion → ready for plan-agent
```

### 2. Planning Phase (plan-agent)
```bash
# plan-agent workflow
1. Read pull/github/* for context
2. Read pull/code/* for current state
3. Create implementation-plan.md → plan/code/
4. Create phase breakdown → plan/code/phases/
5. Draft GitHub responses → plan/github/
6. Signal completion → ready for review-agent
```

### 3. Review Phase (review-agent)
```bash
# review-agent workflow
1. Review plan-agent outputs for viability
2. Create solution-review.md → review/
3. Provide go/no-go recommendation
4. Signal approval → ready for hardcode-agent
```

### 4. Implementation Phase (hardcode-agent)
```bash
# hardcode-agent workflow
1. Read plan/code/phases/ for implementation guide
2. Implement code phase by phase
3. Update pull/code/staged.diff during development
4. Update pull/code/pushed.diff when complete
5. Create testing artifacts → pull/code/testing/
6. Signal completion → ready for final review
```

### 5. Final Review (review-agent)
```bash
# review-agent final workflow
1. Read pull/code/staged.diff for implemented changes
2. Review implementation against requirements
3. Create implementation-review.md → review/
4. Provide final approval/rejection
```

## Inter-Agent Communication Protocols

### Input Validation
Each agent validates prerequisites by checking if required input files exist:

```bash
# pull-agent validation (minimal - mostly external data sources)
# No file prerequisites - extracts from GitHub/git

# plan-agent validation
- Requires: pull/github/issue.md (issue context)
- Requires: pull/github/comments.md (stakeholder input)
- Optional: pull/code/* (current state)

# review-agent validation  
- Requires: pull/github/issue.md (original requirements)
- Requires: plan/code/implementation-plan.md (solution to review)
- Optional: pull/code/pushed.diff (implementation to review)

# hardcode-agent validation
- Requires: plan/code/phases/phases.md (implementation strategy)
- Requires: plan/code/phases/phase*.diff (exact changes to apply)
- Optional: plan/code/implementation-plan.md (context)
```

### Data Consistency Rules

#### 1. Single Writer Principle
- Only one agent writes to its owned directories at a time
- Shared directories (pull/code/) have specific write permissions

#### 2. Read-Only Access Pattern
- Agents read from others' directories but never modify them
- Exception: hardcode-agent updates pull/code/pushed.diff as agreed handoff

#### 3. Atomic Updates
- Agents complete full sections before signaling completion
- No partial updates that could confuse other agents

#### 4. Version Control
- All directory states are tracked in git
- Each agent phase creates a commit for state tracking

## Workflow Orchestration

### Human-Orchestrated Sequential Model
The workflow is controlled entirely by humans calling one agent at a time:

```bash
# Phase 1: Data Extraction
human: claude --agent pull-agent "Extract issue #1234 data"
# pull-agent completes and reports back

# Phase 2: Planning (human checks pull/ directory has data)
human: claude --agent plan-agent "Create implementation plan"
# plan-agent validates pull/github/issue.md exists, then proceeds

# Phase 3: Plan Review (human checks plan/ directory has content)
human: claude --agent review-agent "Review implementation plan"
# review-agent validates required files exist, then provides go/no-go

# Phase 4: Implementation (human proceeds only if review approved)
human: claude --agent hardcode-agent "Apply phase diffs incrementally"
# hardcode-agent validates plan/code/phases/ exists, then implements

# Phase 5: Final Review (human checks implementation completed)
human: claude --agent review-agent "Final review of implementation"
# review-agent validates pull/code/pushed.diff exists, then reviews
```

### Key Principles
- **Human Control**: Human decides when to run each agent and in what sequence
- **No Agent-to-Agent Calls**: Agents never directly invoke other agents
- **File-Based Validation**: Agents check input file existence rather than status signals
- **Clear Reporting**: Each agent reports completion and next recommended steps

## Error Recovery and Rollback

### Rollback Scenarios
1. **Plan Rejection**: roll back to pull-agent data, restart planning
2. **Implementation Failure**: roll back to pre-implementation state
3. **Final Review Failure**: roll back to pre-implementation, revise plan

### Recovery Protocols
```bash
# Simple recovery - just delete and re-run
rm -rf .notes/{issue}/{iteration}/plan/     # Re-run plan-agent
rm -rf .notes/{issue}/{iteration}/review/   # Re-run review-agent  
rm -f pull/code/pushed.diff                # Re-run hardcode-agent

# Or use git for more surgical recovery
git checkout -- .notes/{issue}/{iteration}/plan/    # Reset planning
git checkout -- .notes/{issue}/{iteration}/review/  # Reset reviews
```

## Success Metrics

### Individual Agent Success
- **pull-agent**: All required data extracted and organized
- **plan-agent**: Comprehensive, actionable implementation plan
- **review-agent**: Clear go/no-go decisions with detailed rationale
- **hardcode-agent**: Working implementation that passes all tests

### System-Wide Success
- Seamless data flow between agents
- No data inconsistencies or conflicts
- Clear audit trail of decision-making
- Successful issue resolution with minimal human intervention