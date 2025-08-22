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
4. **hardcode-agent** → reads `plan/`, implements code → updates `pull/code/final/`

### Secondary Data Flow (Feedback Loop)
- **hardcode-agent** → updates implementation state → `pull/code/final/`
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
│   ├── pushed/      # pull-agent writes and maintains
│   ├── staged/      # pull-agent and hardcode-agent write
│   ├── unstaged/    # pull-agent and hardcode-agent write  
│   └── final/       # hardcode-agent writes, all agents read
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
3. Capture current diffs → pull/code/{pushed,staged,unstaged}/
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
3. Update pull/code/staged/ during development
4. Update pull/code/final/ when complete
5. Create testing artifacts → pull/code/testing/
6. Signal completion → ready for final review
```

### 5. Final Review (review-agent)
```bash
# review-agent final workflow
1. Read pull/code/final/ for implemented changes
2. Review implementation against requirements
3. Create implementation-review.md → review/
4. Provide final approval/rejection
```

## Inter-Agent Communication Protocols

### Status Signaling
Each agent signals completion and readiness through standardized status files:

```bash
# Agent completion signals
.notes/{issue}/{iteration}/
├── .status/
│   ├── pull-complete        # pull-agent finished
│   ├── plan-complete        # plan-agent finished
│   ├── review-approved      # review-agent approved plan
│   ├── implementation-complete  # hardcode-agent finished
│   └── final-review-complete    # review-agent final approval
```

### Error and Blocking Protocols
```bash
# Error handling
.status/
├── pull-blocked             # pull-agent cannot proceed
├── plan-needs-revision      # plan-agent needs more data
├── review-rejected          # review-agent rejected plan/implementation
└── implementation-blocked   # hardcode-agent blocked on implementation
```

### Data Consistency Rules

#### 1. Single Writer Principle
- Only one agent writes to its owned directories at a time
- Shared directories (pull/code/) have specific write permissions

#### 2. Read-Only Access Pattern
- Agents read from others' directories but never modify them
- Exception: hardcode-agent updates pull/code/final/ as agreed handoff

#### 3. Atomic Updates
- Agents complete full sections before signaling completion
- No partial updates that could confuse other agents

#### 4. Version Control
- All directory states are tracked in git
- Each agent phase creates a commit for state tracking

## Workflow Orchestration

### Sequential Execution Model
```python
def enterprise_ticket_workflow(issue_number, iteration):
    # Phase 1: Data Extraction
    pull_agent.extract_data(issue_number)
    wait_for_status('pull-complete')
    
    # Phase 2: Planning
    plan_agent.create_plan(pull_data)
    wait_for_status('plan-complete')
    
    # Phase 3: Plan Review
    review_agent.review_plan(pull_data, plan_data)
    wait_for_status('review-approved')
    
    # Phase 4: Implementation
    hardcode_agent.implement(plan_data)
    wait_for_status('implementation-complete')
    
    # Phase 5: Final Review
    review_agent.final_review(pull_data, implementation_data)
    wait_for_status('final-review-complete')
```

### Parallel Execution Opportunities
- pull-agent can update ongoing state while hardcode-agent implements
- review-agent can provide interim feedback during implementation phases
- plan-agent can refine GitHub responses based on implementation progress

## Error Recovery and Rollback

### Rollback Scenarios
1. **Plan Rejection**: roll back to pull-agent data, restart planning
2. **Implementation Failure**: roll back to pre-implementation state
3. **Final Review Failure**: roll back to pre-implementation, revise plan

### Recovery Protocols
```bash
# Rollback commands
git checkout -- .notes/{issue}/{iteration}/plan/    # Reset planning
git checkout -- .notes/{issue}/{iteration}/review/  # Reset reviews
git reset pull/code/final/                          # Reset implementation
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