# Knowledge Preservation Audit — viv-agents Initial Extraction

Date: 2026-05-08
Auditor: extraction process (main session, viblocks-ai → viv-agents)

This document records what was removed during sanitization and classifies each removal as **intentional exclusion** (project-specific, correctly out of scope) or **knowledge loss** (generic content lost; recovery required).

## Methodology

For each agent, compare the source `.claude/agents/<agent>.md` (in viblocks-ai) against the extracted `<domain>/<agent>/AGENT.md`. Classify each delta:

- **Frontmatter restructuring** — expected; SOLID-driven (no loss)
- **Path placeholders** — expected; OCP-driven (no loss; consumer fills in)
- **Project-specific content removed** — must classify per-removal
- **Generic content removed** — investigate; potential loss

## Per-agent audit

### `nestjs-crypto-implementer`

| Removed | Classification | Rationale |
|---|---|---|
| `description` mentioning `services/core, services/bot, packages/shared` | OCP placeholder | Replaced by domain-vocabulary description (Kafka, outbox, blockchain adapters) |
| `## Required Superpowers` section listing `superpowers:*` skill names | Project coupling | Superpowers is a specific Claude Code plugin; the IRON LAW captures the discipline (TDD, verification, root-cause) without naming the plugin |
| `## Blacklist Pipeline — Domain Note (viblocks-ai)` | Project-specific | Blacklist pipeline is a viblocks domain concept; lives in a project skill, not in a generic agent |
| `## WaaS-Specific Mode (viblocks-ai Phase 1.0+)` with 15 hard refusal triggers | Project-specific (intentional removal) | The hard-refusal CONCEPT is preserved as a generic "WaaS Mode" section pointing to `waas-backend` skill; the SPECIFIC invariants live in the skill (single source of truth — viv-skills ADR-002) |
| Pattern Routing table (NestJS-specific patterns 1-26) | Skill content, not agent content | The full table belongs to `crypto-backend/SKILL.md` and `waas-backend/SKILL.md`. Agent says "read your skills' Quick Reference" — the table itself is in the skill |
| Planning Checklist with patterns 1-26 enumerated | Skill content | Same reasoning — the agent's checklist is the discipline; the patterns are the skill's content |
| References to `.claude/skills/...` paths | Path coupling | Replaced with bare skill names per viv-skills ADR-002 |
| `Co-Authored-By: Claude Sonnet 4.6 ...` literal trailer | Project-specific | Generalized to "the project's required co-author trailer per the consumer's commit policy" |
| `pnpm --filter services-core run test` literal | Project-specific | Generalized to `pnpm --filter <service-name> run test` |

**Generic content preserved:**
- TDD discipline + RED-GREEN-REFACTOR
- Verification-before-completion gate
- New Abstraction Gate (DESIGN REQUIRED escalation)
- Completion Signal contract
- Red Flags table (rationalization → recovery)
- 10 Critical Constraints (BigInt, outbox, AbortController, circuit breaker, etc.)

**Verdict:** No knowledge loss. All removals are project-specific or moved to skills.

---

### `nestjs-crypto-reviewer`

| Removed | Classification | Rationale |
|---|---|---|
| `## Blacklist Pipeline — SRP Invariants (viblocks-ai)` table | Project-specific | viblocks-domain knowledge |
| `## Bot Service — Pattern Exemptions (viblocks-ai)` | Project-specific | viblocks-architectural decision |
| Reference to `.claude/skills/blacklist-monitoring/` | Project-specific skill | Not a generic skill |
| `services/core/src/...`, `services/bot/src/...` paths in SOURCE_ROOTS | OCP placeholder | Replaced with `<backend-services>/src/`, `<shared-packages>/src/` |
| Common Mistakes about TRON/Ethereum polling, `targetAddress=null`, `TRON_POLL_BATCH_SIZE` | Project-specific | viblocks domain false positives |
| Reference to `.claude/rules/nestjs-crypto-backend.md` | Project-specific | "Rules" file is a viblocks convention; constraints live in skill SKILL.md |

**Generic content preserved:**
- 7 CRITICAL checks (C01-C07)
- 11 HIGH checks (H01-H11) including test freshness
- 9 MEDIUM checks (M01-M09)
- 3 LOW checks
- WaaS extension checks (C-WAAS-01..05, H-WAAS-01..06) preserved as conditional on `waas-backend` skill being loaded
- Phase 1-7 audit methodology
- Cross-cutting analysis steps
- Common Mistakes (generalized — Dockerfile dev/prod scope, `as any` in tests, M09 design-spec requirement)

**Verdict:** No knowledge loss. WaaS-specific checks preserved as conditional extensions.

---

### `reactjs-crypto-implementer`

| Removed | Classification | Rationale |
|---|---|---|
| `services/ui` paths in `description` | OCP placeholder | Generalized |
| Stack Tecnológico section (Vite + React 19 + TS strict, Tailwind, etc.) | Skill content | Belongs in `react-frontend/SKILL.md` — the agent says "read the skill's stack" |
| Quick Reference table (patterns 1-11) | Skill content | Belongs in `react-crypto-frontend/SKILL.md` |
| Planning Checklist with pattern numbers | Skill content | Same reasoning |
| `pnpm --filter services-ui run test` literal | Project-specific | Generalized to `pnpm --filter <frontend-app> run test` |
| `superpowers:*` skill name references | Project coupling | Generalized to discipline statements |

**Generic content preserved:**
- TDD + verification + new-abstraction gate + completion signal
- Red Flags table (E2E-as-test-bypass, hex hardcode, number-for-money, SSE-cache-replace, etc.)
- 10 Critical Constraints

**Verdict:** No knowledge loss.

---

### `reactjs-crypto-reviewer`

| Removed | Classification | Rationale |
|---|---|---|
| `services/ui/src/...` paths in SOURCE_ROOTS | OCP placeholder | Replaced with `<frontend-app>/src/` |
| Reference to `.claude/rules/react-crypto-frontend.md` | Project-specific | Same reasoning as backend reviewer |
| `services/ui/src/index.css` mention in TOKEN-01 source | Project-specific | Generalized to "the project's token system" |
| `#6B7280` exception literal | Project-specific | Generalized to "the project's documented exception list" |

**Generic content preserved:**
- 5 CRITICAL checks
- 9 HIGH checks (including TOKEN-01 and H-TEST-01..04)
- 7 MEDIUM checks
- 3 LOW checks
- Phase 1-7 methodology
- 11 Common Mistakes (generalized)

**Verdict:** No knowledge loss.

---

### `infra-devops-implementer`

| Removed | Classification | Rationale |
|---|---|---|
| `services/*/src/` mentioned in Out-of-scope table | OCP placeholder | Generalized |
| Reference to `.claude/skills/{docker-patterns,ci-cd-patterns,infra-cloud}/SKILL.md` literal paths | Project-specific path | Replaced with bare skill names |
| Reference to `.claude/context/routing-table.json` and `typed-agent-strategy.md` | Project-specific | These are viblocks routing/architecture artifacts — out of scope per ADR-005 |
| "SS2 compliance" terminology | Project-specific terminology | Generalized to "single-source-of-truth Makefile" |

**Generic content preserved:**
- Pattern Routing table (task → skills)
- Scope (in/out)
- IRON LAW with all 6 verification command groups (Dockerfile, Compose, Actions, Makefile, scripts, IaC)
- Red Flags table (9 entries)
- 10 Critical Constraints
- Planning Checklist (Structural + Conditional)

**Verdict:** No knowledge loss.

---

### `infra-devops-reviewer`

| Removed | Classification | Rationale |
|---|---|---|
| `services/*/src/...` mentions in exclusions | OCP placeholder | Generalized |
| Reference to `.claude/context/routing-table.json` and `typed-agent-strategy.md` | Project-specific | Same as implementer |
| "SS2 compliance" → "single-source-of-truth Makefile" | Terminology generalization | No loss |

**Generic content preserved:**
- 9 CRITICAL checks
- 10 HIGH checks
- 7 MEDIUM checks
- Phase 1-7 methodology
- 10 Common Mistakes
- Scope-boundary notes with `security-reviewer`

**Verdict:** No knowledge loss.

---

### `security-reviewer`

| Removed | Classification | Rationale |
|---|---|---|
| `### Blacklist Pipeline — SRP Invariants (viblocks-ai)` table | Project-specific | Same as backend reviewer |
| Reference to `.claude/skills/blacklist-monitoring/decisions.md` | Project-specific skill | Not generic |
| Reference to `.claude/skills/owasp-security/SKILL.md` (the original skill name) | Renamed | The skill is now `web-security` in viv-skills |
| `S33` literal check ID embedded in prose | Skill content (lives in `web-security/SKILL.md`) | Mentioned check IDs preserved as `S##` placeholder |
| `VITE_PUBLIC_*` literal | Generalized | Now reads "Public-prefixed env vars (e.g. `VITE_PUBLIC_*`, `NEXT_PUBLIC_*`)" |

**Generic content preserved:**
- 4 Principles
- Phase 1-7 methodology
- File Pattern → Categories table
- False-Positive Guard (CORS dev-only, parameterized templates, env-branch business logic vs dangerous-surface)
- Verdict rules
- Finding format
- Scope boundary with `infra-devops-reviewer` (security vs infra hygiene)

**Verdict:** No knowledge loss. Skill rename (owasp-security → web-security) reflects what's already in viv-skills.

---

### `dev-testing-strategy-reviewer`

| Removed | Classification | Rationale |
|---|---|---|
| `services/ui/e2e/...`, `services/core/...` literal paths | OCP placeholder | Generalized to `<frontend-app>/e2e/`, etc. |
| Reference to `.claude/rules/dev-testing-strategy.md` | Project-specific | Generalized to "the skill's pattern files are the authoritative source" |
| `Kafka` literal in H07/H08 (where genericized) | Generalized | "Event-stream consumer (Kafka, NATS, etc.)" — preserves the rule, generalizes the technology |
| `blacklist.address` example | Project-specific | Generalized to `<topic>` |
| `Test.createTestingModule()` in M02 | Now framing as "NestJS-style ... or equivalent" | Preserves the rule, allows for non-NestJS test frameworks |

**Generic content preserved:**
- 4 Principles
- 10 CRITICAL checks
- 8 HIGH checks
- 7 MEDIUM checks
- 3 LOW checks
- Phase 1-9 methodology
- Platform decoupling deep-scan commands
- Test data isolation scan procedure
- Verdict rules

**Verdict:** No knowledge loss.

---

## Aggregate verdict

**8 agents audited. Zero net knowledge loss.**

All removed content classifies as one of:
1. **Project-specific (intentional exclusion)** — viblocks domain knowledge (blacklist pipeline, services/* paths, VI-* issue numbers, project-specific rule files, project-specific skill paths)
2. **Skill content (correctly relocated)** — pattern routing tables, planning checklists with pattern numbers, stack technology lists; these belong in skills' SKILL.md files per viv-skills ADR-002
3. **Path placeholders** — `<backend-services>/**`, `<frontend-app>/**`, `<service-name>` per ADR-004 (OCP)
4. **Project coupling generalizations** — `superpowers:*` skill name references replaced with discipline statements; `Co-Authored-By: Claude Sonnet 4.6` replaced with "the project's required trailer"

**Generic methodology, checks, and discipline preserved 100%** across all 8 agents.

## Cross-skill consistency

Verified that:
- All references to skills use **bare names** (no domain prefix), per viv-skills ADR-002 cross-skill convention
- All references to shared review formats use **relative path** `../../_shared/<file>.md`, an internal contract within viv-agents
- No reference to viblocks-specific skills (`blacklist-monitoring`)
- Skill names align with viv-skills repo: `nestjs-backend`, `crypto-backend`, `waas-backend`, `prisma-patterns`, `react-frontend`, `react-crypto-frontend`, `waas-frontend`, `docker-patterns`, `ci-cd-patterns`, `infra-cloud`, `web-security`, `dev-testing-strategy`, `root-cause-discipline`

## Sign-off

Audit complete. Extraction proceeds to commit.

---

## Post-extraction review (2026-05-08, same day)

A critical review was performed after the initial commit. It identified design issues in the extraction process itself (not knowledge losses). Fixes applied in a follow-up commit:

- **Flat-file format** — agents renamed from `<domain>/<name>/AGENT.md` to `<domain>/<name>.md` to match Claude Code's runtime expectation. See ADR-008.
- **Project-rooted shared paths** — `_shared/` references changed from relative `../../_shared/` to project-rooted `.claude/agents/_shared/`. See ADR-009.
- **Skills schema** — `skills:` restructured from flat array (with YAML comments) to `{ required: [...], optional: [...] }` for machine-readability. See ADR-010.
- **All `behavior` fields explicit** — every agent now declares all four behavior fields explicitly, removing implicit defaults. See ADR-011.
- **Implementer's reference to reviewer's `_shared/`** — removed (the implementer doesn't consume those files).
- **Prose comment in `infra-devops-implementer` frontmatter** — moved to body as a documented section.
- **`framework-detection.md` AI-DLC reference** — generalized to "consumer-defined framework signal".
- **README vendor instructions** — simplified (no rename gymnastics needed).
- **Conditional logic in `nestjs-crypto-implementer.description`** — removed the "(when paired with WaaS skills)" qualifier; description now describes competence unconditionally.

Knowledge content unchanged by these corrections — they are structural / format / DIP improvements, not content changes.
