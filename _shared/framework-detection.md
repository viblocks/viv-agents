# Shared — Framework Detection & Save Report

Canonical save-report procedure for reviewer agents. After self-verification, detect the project's documentation framework (if any) and save the report accordingly.

## Step 1: Detect Framework

Check these paths in order:

| Check | Framework | Audit Directory |
|-------|-----------|----------------|
| `<docs-framework-state-file>` exists (e.g. an SDLC framework state file) | Project-specific framework | Read the framework state file to determine the current phase, then save under the framework's audit directory |
| None of the above | No framework | `.claude/audits/` |

Replace `<docs-framework-state-file>` with the consumer project's framework signal (e.g. `aidlc-docs/aidlc-state.md`, `process/state.json`). Each consumer can define their own detection rule in this file at vendor time.

## Step 2: Save the Report

1. Create the audit directory if it does not exist.
2. Write the report to `{audit-dir}/YYYY-MM-DD-{scope-prefix}-{scope}.md`. The `scope-prefix` is supplied by the calling reviewer (each reviewer documents its own prefix).
3. If a file with the same name exists, append a counter (e.g., `-2`, `-3`).

## Step 3: Update Framework Trail (if applicable)

If a project framework expects an audit trail file, append a one-line entry referencing the report. Use `Edit` (append), never `Write` (overwrite).

## Step 4: Inform the User

Tell the user:
- The file path where the report was saved
- The verdict (BLOCK / PASS WITH RESERVATIONS / PASS)
- Whether the framework audit trail was updated (if applicable)
