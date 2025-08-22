# Enterprise Repository Agent Usage Examples

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