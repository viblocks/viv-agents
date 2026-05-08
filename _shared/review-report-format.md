# Shared — Review Report Format

Canonical report template + verdict rules + self-verification checklist for autonomous reviewer agents in this repo. When a reviewer references this file, follow it verbatim except where the reviewer overrides a specific section.

Reviewers that consume this file: `nestjs-crypto-reviewer`, `reactjs-crypto-reviewer`, `infra-devops-reviewer`. Other reviewers (e.g. `security-reviewer`) may define their own structured output.

## Report Generation

```markdown
## Audit Report — [date]

### Summary
- Critical: N findings
- High: N findings
- Medium: N findings
- Low: N findings

### Critical Findings
[Cxx] [rule name] — file.ts:42
→ Found: `actual code snippet`
→ Expected: `description of correct implementation`

### High Findings
(same format)

### Medium Findings
(same format)

### Low Findings
(same format)

### Passed Checks
- [Cxx] PASS: <one-line summary>

### Not Applicable
- [Cxx] N/A: <reason>

### Verdict
[BLOCK / PASS WITH RESERVATIONS / PASS]
```

## Verdict Rules

- **BLOCK**: Any CRITICAL finding present
- **PASS WITH RESERVATIONS**: HIGH findings present, no CRITICAL
- **PASS**: Only MEDIUM/LOW findings or none

## Self-Verification (MANDATORY before presenting the report)

1. Re-read every file cited in a CRITICAL or HIGH finding
2. Confirm line numbers are accurate
3. Confirm code snippets match what is actually in the file
4. If any finding is wrong → remove it and add a note in Verification Notes

A finding removed on re-read is better than a finding kept that cannot be defended. Follow the owning reviewer's Common Mistakes section to avoid repeating known false positives.

## Important Constraints

- Never assess code you have not read — if a file does not exist, say so
- Never assume a pattern is correct because the file is named correctly
- Never report a finding without a specific file path and line number
- If you find something not in the checklist, report it as "ADDITIONAL" finding with severity
- **NEVER modify source code** — you are a reviewer, not a fixer. Only write the report and audit trail entries
