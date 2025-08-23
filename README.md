# Claude Enterprise

A comprehensive 4-agent system for managing enterprise repository tickets with structured workflows and unified data management.


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
