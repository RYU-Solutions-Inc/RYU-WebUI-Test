---
description: Run the complete testing pipeline (analyze → design → review → build → run → report)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: "<version-identifier> [--mode regression]"
---

Run the full testing pipeline for version: $ARGUMENTS

The Architect orchestrates the entire pipeline with analyst checkpoints between phases.

## Pipeline Phases

### Phase 1: Analyze
Run `/qa/analyze` for the specified version.
- Present impact analysis to analyst
- **Checkpoint:** analyst confirms before proceeding

### Phase 2: Design
Run `/qa/design` for each NEW item identified in the analysis.
- Present scenarios to analyst
- Generate review files for each component
- **Checkpoint:** tell analyst to review the `.review.md` files in their editor

### Phase 3: Review
Run `/qa/review` to process the analyst's review decisions.
- Report accepted/rejected/needs-changes counts
- If any items need revision, delegate back to Scenario Designer and generate updated review files
- **Checkpoint:** repeat until all scenarios are accepted (or analyst decides to proceed with `--force`)

### Phase 4: Build
Run `/qa/build` for accepted scenarios + MUST UPDATE items.
- Review gate enforced — only accepted scenarios are scripted
- Present generated scripts to analyst
- **Checkpoint:** analyst confirms before execution

### Phase 5: Run
Run `/qa/run --regression` (or the mode specified in arguments).
- Pre-flight → execute → interpret results
- Present results to analyst

### Phase 6: Report
Run `/qa/report` to generate all reports.
- Present release readiness summary to analyst

## Rules

- Each phase waits for analyst confirmation before proceeding to the next
- The analyst can stop the pipeline at any checkpoint to make changes
- If any phase encounters a blocker (pre-flight failure, critical gaps), the pipeline stops and reports
- The review phase (Phase 3) may loop multiple times until all scenarios are accepted

## Output

All outputs from each phase: manifest, scenarios, review files, scripts, test results, and reports.
