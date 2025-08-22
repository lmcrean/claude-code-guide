# Enterprise Repository Agent Usage Examples

## Quick Start Commands

### Setup New Ticket
```bash
# Copy template structure for new ticket
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/1234/i1

# Navigate to ticket directory
cd .notes/1234/i1
```

### Full Workflow Example
```bash
# 1. Extract issue data
claude --agent pull-agent "Extract all data for issue #1234"

# 2. Create implementation plan  
claude --agent plan-agent "Create implementation plan based on pull/ data"

# 3. Review the plan
claude --agent review-agent "Review the implementation plan for viability"

# 4. Implement the solution
claude --agent hardcode-agent "Implement the solution following the plan phases"

# 5. Final review
claude --agent review-agent "Perform final review of implementation"
```

## Detailed Workflow Examples

### Example 1: Bug Fix Workflow

#### Issue Context
- **Issue #1234**: "Search autocomplete fails when typing partial words"
- **Repository**: Large React application
- **Complexity**: Medium

#### Step 1: Data Extraction (pull-agent)
```bash
# Command
claude --agent pull-agent "Extract issue #1234 data and current codebase state"

# Expected outputs in pull/
pull/
├── github/
│   ├── issue.md              # Issue: Search autocomplete fails...
│   ├── comments.md           # 5 user comments with reproduction steps
│   └── pr-diffs/            # No existing PR
├── code/
│   ├── pushed/              # Empty (no existing changes)
│   ├── staged/              # Empty
│   ├── unstaged/            # Empty
│   └── final/               # Empty (no implementation yet)
```

#### Step 2: Planning (plan-agent)
```bash
# Command
claude --agent plan-agent "Create implementation plan for search autocomplete bug"

# Expected outputs in plan/
plan/
├── github/
│   └── pr-draft.md          # "Fix search autocomplete for partial word matching"
└── code/
    ├── implementation-plan.md    # Root cause: string matching logic
    ├── viability-assessment.md  # Low complexity, high impact
    └── phases/
        ├── phases.md            # Phase overview and validation strategy
        ├── phase1.diff          # Diff to reproduce issue + add tests
        ├── phase2.diff          # Diff to fix matching algorithm
        └── phase3.diff          # Diff for edge cases and cleanup
```

#### Step 3: Plan Review (review-agent)
```bash
# Command
claude --agent review-agent "Review implementation plan for search autocomplete fix"

# Expected outputs in review/
review/
├── solution-review.md       # ✅ Plan approved with minor suggestions
└── viability-report.md     # Recommend: Proceed with implementation
```

#### Step 4: Implementation (hardcode-agent)
```bash
# Command  
claude --agent hardcode-agent "Implement search autocomplete fix following phases"

# Expected outputs back in pull/
pull/code/
├── final/final.diff         # Complete implementation diff
├── staged/staged.diff       # What's currently staged
└── testing/
    ├── reproduce-issue.js   # Script that reproduces original bug
    ├── test-autocomplete.js # New tests for fix
    └── validation-results.md # Test execution results
```

#### Step 5: Final Review (review-agent)
```bash
# Command
claude --agent review-agent "Final review of search autocomplete implementation"

# Expected outputs in review/
review/
├── implementation-review.md # ✅ Implementation meets requirements
└── diff-analysis.md        # Code quality assessment
```

### Example 2: Feature Implementation Workflow

#### Issue Context
- **Issue #5678**: "Add dark mode toggle to user preferences"
- **Repository**: Vue.js application with Vuex
- **Complexity**: High (affects multiple components)

#### Complete Workflow
```bash
# Setup
cp -r agents/template/.notes/{issue_number}/{iteration} .notes/5678/i1
cd .notes/5678/i1

# 1. Data extraction
claude --agent pull-agent "Extract issue #5678 - dark mode toggle feature request"

# Result: pull/github/issue.md shows detailed requirements
# Result: pull/github/comments.md shows user preferences and mockups

# 2. Planning
claude --agent plan-agent "Plan dark mode toggle implementation"

# Result: plan/code/implementation-plan.md shows:
# - Vuex state management approach
# - Component modification strategy  
# - CSS variable system design
# Result: plan/code/phases/phases.md shows phase overview and validation strategy
# Result: plan/code/phases/phase1.diff, phase2.diff, etc. show exact diffs to implement

# 3. Initial review
claude --agent review-agent "Review dark mode implementation plan"

# Result: review/solution-review.md recommends proceeding with modifications

# 4. Implementation
claude --agent hardcode-agent "Implement dark mode toggle feature"

# Result: pull/code/final/final.diff shows complete implementation
# Result: pull/code/testing/ shows theme switching tests

# 5. Final review
claude --agent review-agent "Final review of dark mode implementation"

# Result: review/implementation-review.md confirms requirements satisfied
```

## Agent-Specific Usage Patterns

### pull-agent Patterns
```bash
# Extract specific issue
"Extract issue #1234 with all comments and related PRs"

# Update current state
"Update code state tracking - capture current staged and unstaged changes"

# Extract PR data
"Pull PR #567 data including diffs and review comments"

# Complete data refresh
"Refresh all pull/ data for issue #1234 - GitHub and code state"
```

### plan-agent Patterns
```bash
# Create initial plan
"Create comprehensive implementation plan for the extracted issue data"

# Plan specific aspect
"Focus on GitHub response strategy - draft PR description and comment replies"

# Revise existing plan
"Revise implementation plan based on new constraints in comments"

# Phase-focused planning
"Create detailed phase breakdown for the database migration approach"
```

### review-agent Patterns
```bash
# Plan review
"Review implementation plan for technical soundness and completeness"

# Implementation review
"Review final implementation against original requirements"

# Specific aspect review
"Focus review on security implications of the authentication changes"

# Diff analysis
"Analyze the implementation diff for breaking changes and performance impact"
```

### hardcode-agent Patterns
```bash
# Full implementation
"Implement complete solution following all phases in the plan"

# Single phase
"Implement only phase 2 - database schema changes"

# Test-focused implementation
"Focus on creating comprehensive tests for the new feature"

# Reproduction script
"Create reproduction script for the original issue before fixing"
```

## Troubleshooting Common Scenarios

### Scenario 1: Plan Rejection
```bash
# If review-agent rejects the plan
claude --agent plan-agent "Revise implementation plan addressing the review concerns"
claude --agent review-agent "Re-review the revised implementation plan"
```

### Scenario 2: Implementation Blocked
```bash
# If hardcode-agent gets blocked
claude --agent review-agent "Analyze implementation blocker and suggest alternatives"
claude --agent plan-agent "Create alternative approach for the blocked implementation"
```

### Scenario 3: Missing Context
```bash
# If more data is needed
claude --agent pull-agent "Extract additional context - related issues and recent changes"
claude --agent plan-agent "Update plan with newly available context data"
```

### Scenario 4: Final Review Fails
```bash
# If implementation doesn't meet requirements
claude --agent review-agent "Identify specific gaps in implementation"
claude --agent hardcode-agent "Address the identified implementation gaps"
claude --agent review-agent "Re-review the updated implementation"
```

## Integration with Existing Workflows

### Git Integration
```bash
# After successful implementation
git add .notes/1234/i1/
git commit -m "Complete enterprise ticket #1234 - search autocomplete fix

- Extracted issue data and requirements
- Created comprehensive implementation plan
- Implemented fix with full test coverage
- Validated solution meets all requirements"
```

### CI/CD Integration
```bash
# Automated validation
./scripts/validate-ticket.sh .notes/1234/i1/
# Checks:
# - All required files present
# - Tests pass
# - Code quality metrics met
# - Documentation complete
```

### Team Handoff
```bash
# Prepare for team review
claude --agent plan-agent "Generate team review summary from all agent outputs"
# Creates consolidated summary for team review meeting
```

## Best Practices

### Directory Management
- Always start with fresh template copy for each iteration
- Keep separate iterations (i1, i2, i3) for different approaches
- Archive completed tickets for reference

### Agent Coordination
- Run agents in sequence unless specifically coordinating parallel work
- Always check status files before proceeding to next agent
- Use review-agent as quality gate between major phases

### Data Consistency
- Let pull-agent maintain all source of truth data
- Don't manually edit pull/ directory - use agents to update
- Regularly validate that agent outputs are consistent with each other