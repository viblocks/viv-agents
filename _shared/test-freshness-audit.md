# Shared — Test Freshness Audit

Canonical H-TEST-0x and M-TEST-0x rules for reviewer agents that audit test hygiene against a diff. Path globs are supplied by the consuming reviewer (see "Parametrization" at the bottom).

Reviewers that consume this file: any reviewer that audits a codebase with co-located tests (typically backend and frontend reviewers). Test glob paths differ per reviewer.

## HIGH — Test Freshness

| ID | Rule | What to Look For |
|----|------|-----------------|
| H-TEST-01 | Skipped test introduced | `.skip(`, `xit(`, `xdescribe(`, `it.todo(`, `describe.skip(`, or `test.skip(` **added** in this diff without an inline justification comment (e.g. `// TODO <issue-id>: ...`) |
| H-TEST-02 | Source changed, test not updated | Non-test file modified under the reviewer's source root without a sibling spec file also modified in the same diff. Exception: pure type-only files, barrel `index.ts` re-exports, pure style/config files. Files renamed or deleted must also move or delete their tests |
| H-TEST-03 | Stale mock | A dependency's shape changed in the diff (method signature, DTO field, event payload, schema) and a test mock for that dependency was NOT updated in the same diff. Look for mock factory calls (`jest.mock`, `vi.mock`, `mockResolvedValue({...})`), manual stubs, or HTTP route mocks returning the old shape |
| H-TEST-04 | Branch added without test | New conditional (`if`, `switch case`, early return, ternary with side effect) introduces a code path not exercised by any existing or added test. If no added test hits the new path → HIGH |

## MEDIUM — Test Freshness

| ID | Rule | What to Look For |
|----|------|-----------------|
| M-TEST-01 | Test description drifted | Test `describe`/`it` description no longer matches what the production code actually does after this diff (e.g., test says "rejects invalid addresses" but code now accepts them). Grep test strings against new code behavior |
| M-TEST-02 | Test exists but asserts nothing new | Test modified in diff but assertions still cover the OLD behavior — production logic changed, test was "updated" only in fixture values. Look for tests whose `expect(...)` patterns do not exercise the changed branches |
| M-TEST-03 | Orphaned test after code deletion | Spec references a symbol (class, function, token, component, hook, store) that no longer exists after the diff — test still passes because dead code is mocked, but validates nothing real |

## Detection Steps

Run these in order against the diff:

### a. Skipped tests
```bash
git diff origin/main...HEAD -- <TEST_GLOBS> | grep -E '^\+.*\b(\.skip\(|xit\(|xdescribe\(|it\.todo\(|describe\.skip\(|test\.skip\()'
```
Each added line without an inline issue-tracker justification → H-TEST-01.

### b. Source modified without test touched
`git diff --name-only origin/main...HEAD` → split into source files (non-test) and test files. For each modified source file under the reviewer's source root, check that at least one spec in the same module directory is also in the diff. Exempt files vary per reviewer — see Parametrization. Missing sibling spec → H-TEST-02.

### c. Stale mock detection
For each modified interface / abstract class / DTO / event / schema in the diff, grep spec files for mocks of that symbol. If the mock's shape does not match the new signature and the spec was NOT modified in this diff → H-TEST-03.

### d. New branch without coverage
For each new conditional in the diff, identify the function or component containing it, then grep its spec for a test case that exercises the new branch (by input value, mock setup, `describe` name, or user interaction). No matching test → H-TEST-04.

### e. Description drift
For specs modified in the diff, read the new `describe`/`it` strings and the corresponding production code. If the description describes OLD behavior → M-TEST-01.

### f. Assertions did not change
Diff the spec file and check that `expect(...)` / `toBeInTheDocument` / `toHaveText` / `toHaveValue` lines changed, not only setup/fixture values. Spec touched but assertions unchanged → M-TEST-02.

### g. Orphaned tests
Grep specs for imports / references to symbols that were deleted in the diff → M-TEST-03.

## Parametrization

Each consuming reviewer must specify (in its own AGENT.md `Shared References` section):

1. **TEST_GLOBS**: quoted path globs for its own test files, fed to step (a).
   - Backend example: `'*.spec.ts' '*.test.ts'`
   - Frontend example: `'*.spec.ts' '*.spec.tsx' '*.test.ts' '*.test.tsx' 'e2e/**/*.spec.ts'`

2. **SOURCE_ROOTS**: glob roots for step (b) source-file partition.
   - Backend example: `<backend-services>/src/`, `<shared-packages>/src/`
   - Frontend example: `<frontend-app>/src/`, `<frontend-app>/e2e/`

3. **EXEMPT_FILES**: source files exempt from H-TEST-02.
   - Backend example: `*.dto.ts` with only new optional fields, `*.module.ts` with only DI wiring, `index.ts` barrels
   - Frontend example: pure type definition files, barrels, CSS/style-only changes

4. **MOCK_PATTERNS**: strings to grep for step (c).
   - Backend example: `jest.mock`, `vi.mock`, `mockResolvedValue`, `mockReturnValue`, manual stubs
   - Frontend example: `vi.mock`, MSW handlers, HTTP route mocks
