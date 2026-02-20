---
name: traceability
description: How to build and update the requirements traceability matrix (RTM) and assess coverage gaps. Use when mapping requirements to scenarios, scripts, and results.
user-invocable: false
---

## Instructions

### Building the RTM

1. Scan all scenario files in `test-cases/` for requirement references (the `Requirement:` field)
2. Scan the version manifest for Jira ticket references
3. Build a mapping: Requirement → Scenarios → Scripts → Last Run Result

### RTM Format

```markdown
| Requirement | Description | Scenarios | Reviewed | Scripts | Last Run | Status |
|---|---|---|---|---|---|---|
| REQ-AUTH-001 | User login | 8 | 8/8 | 8/8 | Feb 16 ✓ | COVERED |
| REQ-CART-007 | Coupon codes | 15 | 12/15 | 12/15 | Feb 15 ✓ | PARTIAL |
| REQ-PAY-003 | Gift cards | 0 | — | — | — | NO COVERAGE |
```

### Identifying Gaps

A gap exists when:
- A requirement has 0 scenarios → NO COVERAGE
- A feature is missing an entire coverage category (e.g., no error condition scenarios)
- Scenarios exist but are all `draft` → UNREVIEWED
- Scenarios exist but scripts haven't been generated → NO SCRIPTS
- Scripts exist but haven't been run → UNTESTED

### Coverage View

For each feature, show the coverage tree:

```
Feature: {Name} ({Requirement ID})
├─ Happy Path: {N} scenarios ({status})
├─ Edge Cases: {N} scenarios ({status})
├─ Negative: {N} scenarios ({status})
└─ Error/Recovery: {N} scenarios ({status})
```

Mark categories with 0 scenarios as `✗ gap`.
