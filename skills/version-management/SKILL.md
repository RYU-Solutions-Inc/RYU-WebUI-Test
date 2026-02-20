---
name: version-management
description: How to create version folders, write manifests, resolve inheritance chains, and assemble regression sets. Use when managing version-specific test sets.
user-invocable: false
---

## Instructions

### Creating a New Version

1. Create directory: `versions/v{X}/`
2. Create manifest: `versions/v{X}/manifest.json` with schema from `features/impact-analysis.md`
3. Set `base` to the previous version identifier
4. Populate `new`, `must_update`, `check`, `inherited` from impact analysis results
5. Calculate `regression_scope` — for each component, identify which version folder contains its scripts

### Resolving Inheritance

To build the complete test set for any version:

1. Start at the target version's manifest
2. For each component listed in `regression_scope`, note which version to pull from
3. Walk backward through `base` references if any component isn't explicitly listed
4. The resolved set = union of all components with their source version paths

This is a deterministic lookup — no AI reasoning needed. The regression runner script automates this.

### Manifest Updates

Manifests should be updated when:
- Impact analysis completes (initial categorization)
- Scenarios are designed (update scenario counts)
- Scenarios are reviewed (update review status)
- Scripts are generated (update script counts)
- Tests are run (update CHECK items with pass/fail results)

### Resolving Individual Tests

To resolve a specific TC-ID to its script file path:

1. Search `test-cases/` markdown files for the TC-ID (e.g., `TC-CART-008`)
2. Identify the component from the file name (e.g., `test-cases/cart.md` → component is `cart`)
3. Use the same inheritance resolution as full regression, but only for this component
4. The script file is at `versions/{resolved-version}/{component}/{TC-ID}.spec.ts` or within a shared component spec file — use `--grep TC-CART-008` to target the specific test

This reuses the existing inheritance chain. No special logic — it's just regression resolution narrowed to one component, then filtered to one test.

### Manifest Validation

Before reading any `manifest.json`, validate its integrity:

1. **Parse check:** Attempt to parse the file as JSON. If parsing fails:
   - Check for common corruption patterns:
     - Git merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) — tell analyst to resolve the merge conflict
     - Truncated JSON (missing closing `}` or `]`) — likely an interrupted write
     - Empty file (0 bytes) — likely a failed create
   - Check for backup file `.manifest.json.bak` in the same directory
   - If backup exists, offer to restore: "manifest.json is corrupted ({reason}). A backup exists from {date}. Restore from backup?"
   - If no backup, report the error and suggest manual repair
2. **Schema check:** Verify required fields exist: `version`, `base` (except v1.0), `regression_scope`
3. **Reference check:** Verify `base` points to an existing version folder with a valid manifest

If validation fails, do NOT proceed with the operation. Report the specific issue to the analyst.

### Manifest Backup

Before writing to any `manifest.json`:

1. If the file already exists, copy it to `.manifest.json.bak` in the same directory
2. Write the updated manifest
3. Verify the written file parses as valid JSON (read it back and parse)
4. If the write verification fails, restore from `.manifest.json.bak` and report the error

This ensures a known-good backup always exists for recovery.

### Version Naming

Match the app's versioning scheme. If the app uses semver (1.0.0), use that. If it uses date-based (2026-02-16), use that. Document the convention in CLAUDE.md.
