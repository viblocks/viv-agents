# Shared — Framework Detection & Save Report

Canonical save-report procedure for reviewer agents. After self-verification, detect the project's documentation framework (if any) and save the report accordingly.

## Step 1: Detect Framework

Check these paths in order:

| Check | Framework | Audit Directory |
|-------|-----------|----------------|
| Consumer-defined framework signal exists (a state file or marker that identifies an SDLC framework) | Project-specific framework | Read the framework state file to determine the current phase, then save under the framework's audit directory |
| None | No framework | `.claude/audits/` |

The consumer project defines its own detection rule by editing this file at vendor time (e.g. checking for a specific state file path, a marker directory, or an env var). If no framework is in use, the default `.claude/audits/` location applies.

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
