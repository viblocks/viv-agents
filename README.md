# viv-agents

Reusable Claude Code typed-agent definitions, organized by domain and **layered to mirror the viv-skills dependency stack**. Pure descriptors — no enforcement, no routing, no workflow orchestration. Companion repo to [viv-skills](https://github.com/viblocks/viv-skills).

## What's an agent (in this repo)?

An agent is a single `.md` file at `<domain>/<name>.md`, with:
- Frontmatter declaring identity, knowledge (skills consumed), tools, and behavior assertions
- Body containing the agent's system prompt (principles, invocation, checklists, constraints)

Agents are read on-demand by Claude Code at runtime. The file format matches Claude Code's standard `.claude/agents/<name>.md`.

## Repository structure

```
viv-agents/
├── _shared/                              ← shared review formats consumed by reviewers
│   ├── review-report-format.md
│   ├── framework-detection.md
│   └── test-freshness-audit.md
├── architecture/
│   ├── decisions.md                      ← ADRs documenting design choices
│   └── preservation-audit.md
├── backend/                              (6 agents — 3-tier stack: base / crypto / waas)
│   ├── backend-implementer.md
│   ├── backend-reviewer.md
│   ├── backend-crypto-implementer.md
│   ├── backend-crypto-reviewer.md
│   ├── backend-waas-implementer.md
│   └── backend-waas-reviewer.md
├── frontend/                             (6 agents — 3-tier stack: base / crypto / waas)
│   ├── frontend-implementer.md
│   ├── frontend-reviewer.md
│   ├── frontend-crypto-implementer.md
│   ├── frontend-crypto-reviewer.md
│   ├── frontend-waas-implementer.md
│   └── frontend-waas-reviewer.md
├── devops/                               (2 agents — no layering; cross-cutting skills)
│   ├── infra-devops-implementer.md
│   └── infra-devops-reviewer.md
├── security/
│   └── security-reviewer.md
└── testing/
    └── dev-testing-strategy-reviewer.md
```

**16 agents total.** Domain subdirectories are organizational only — invisible after vendoring.

## Layered stacks (alignment with viv-skills)

The backend and frontend domains each have a **3-tier dependency stack**, mirroring `viv-skills`:

```
Backend:  backend-waas-*           ← consumes waas-backend
              ↑
          backend-crypto-*         ← consumes crypto-backend
              ↑
          backend-*                ← consumes nestjs-backend (the base framework skill)

Frontend: frontend-waas-*          ← consumes waas-frontend
              ↑
          frontend-crypto-*        ← consumes react-crypto-frontend
              ↑
          frontend-*               ← consumes react-frontend (the base framework skill)
```

Each tier inherits skills from below and adds its own. Each agent has a focused SRP — no conditional logic mixing tiers in one body.

`devops`, `security`, `testing` have no layering because the corresponding skills are independent (no `requires:` chain).

## Available agents by domain

### `backend/` — 3-tier stack

| Agent | Tier | Required skills | Optional skills |
|---|---|---|---|
| `backend-implementer` | base | `nestjs-backend`, `root-cause-discipline` | — |
| `backend-reviewer` | base | `nestjs-backend`, `root-cause-discipline` | `prisma-patterns`, `web-security` |
| `backend-crypto-implementer` | +crypto | `nestjs-backend`, `crypto-backend`, `root-cause-discipline` | `prisma-patterns` |
| `backend-crypto-reviewer` | +crypto | `nestjs-backend`, `crypto-backend`, `root-cause-discipline` | `prisma-patterns`, `web-security` |
| `backend-waas-implementer` | +waas | `nestjs-backend`, `crypto-backend`, `waas-backend`, `root-cause-discipline` | `prisma-patterns`, `web-security` |
| `backend-waas-reviewer` | +waas | same as implementer | `prisma-patterns`, `web-security` |

### `frontend/` — 3-tier stack

| Agent | Tier | Required skills | Optional skills |
|---|---|---|---|
| `frontend-implementer` | base | `react-frontend`, `root-cause-discipline` | — |
| `frontend-reviewer` | base | `react-frontend`, `root-cause-discipline` | `web-security` |
| `frontend-crypto-implementer` | +crypto | `react-frontend`, `react-crypto-frontend`, `root-cause-discipline` | — |
| `frontend-crypto-reviewer` | +crypto | `react-frontend`, `react-crypto-frontend`, `root-cause-discipline` | `web-security` |
| `frontend-waas-implementer` | +waas | `react-frontend`, `react-crypto-frontend`, `waas-frontend`, `root-cause-discipline` | — |
| `frontend-waas-reviewer` | +waas | same as implementer | `web-security` |

### `devops/` — no layering

| Agent | Required skills |
|---|---|
| `infra-devops-implementer` | `docker-patterns`, `ci-cd-patterns`, `infra-cloud`, `root-cause-discipline` |
| `infra-devops-reviewer` | same |

### `security/` and `testing/` — single agent each

| Agent | Required skills |
|---|---|
| `security-reviewer` | `web-security`, `root-cause-discipline` |
| `dev-testing-strategy-reviewer` | `dev-testing-strategy`, `root-cause-discipline` |

## Naming convention

Agent names use **stack/domain prefix**, not framework prefix:

- `backend-implementer` (not `nestjs-implementer`)
- `frontend-implementer` (not `react-implementer`)
- `backend-crypto-implementer` (not `nestjs-crypto-implementer`)

The framework concrete (e.g. NestJS for backend, React for frontend) lives in the **declared skills**, not in the agent name. This decouples agent identity from current framework choice — if a future skill `fastify-backend` exists, the same `backend-implementer` agent can consume it.

See ADR-012 for the full rationale.

## Frontmatter contract (SOLID-compliant)

```yaml
---
name: <agent-name>
type: implementer | reviewer
domain: backend | frontend | devops | security | testing
description: <competence description, no project paths>
skills:
  required: [<skill-name>, ...]
  optional: [<skill-name>, ...]
tools: [<Claude Code tool>, ...]
behavior:
  modifies_meta_config: <bool>
  modifies_source_code: <bool>
  uses_network: <bool>
  performs_destructive_git: <bool>
---
```

Every agent declares all four behavior fields explicitly.

**Out of scope** (lives in other components):
- ❌ `scope.paths` — that's routing
- ❌ `post_implementation_chain` — that's workflow
- ❌ Hook script names / marker files — that's enforcement

See `architecture/decisions.md` for the full SOLID rationale.

## Composability

`viv-agents` declares contracts. Possible additional components:

| Component | What it does | Status |
|---|---|---|
| **viv-skills** | Knowledge each agent consumes | [Available](https://github.com/viblocks/viv-skills) |
| **viv-hooks** | Enforcement layer (reads `behavior:`) | Not yet extracted |
| **viv-routing** | Path → agent dispatch | Not yet extracted |
| **viv-workflows** | Implementer → reviewer chains | Not yet extracted |

These are *possible extensions*, not commitments. `viv-agents` works **standalone in behavioral mode**.

## What to install per project type

| Project type | Agents to vendor |
|---|---|
| Plain NestJS backend | `backend/backend-implementer.md` + `backend/backend-reviewer.md` |
| NestJS + crypto | `backend/backend-crypto-*.md` (the base agents are not needed if you only audit crypto code; but vendor them too if your codebase has non-crypto modules) |
| WaaS backend | full backend stack: 6 agents |
| Plain React UI | `frontend/frontend-*.md` (base) |
| React + crypto | `frontend/frontend-crypto-*.md` |
| WaaS UI | full frontend stack: 6 agents |
| Anything with CI/CD | + `devops/infra-devops-*.md` |
| User-facing API with auth | + `security/security-reviewer.md` |
| Anything with non-trivial test suite | + `testing/dev-testing-strategy-reviewer.md` |

## Layered review chains

When auditing a diff that touches multiple tiers, run the relevant reviewers in order and aggregate:

- Pure base backend changes → `backend-reviewer`
- Crypto-aware backend → `backend-reviewer` + `backend-crypto-reviewer`
- WaaS backend → `backend-reviewer` + `backend-crypto-reviewer` + `backend-waas-reviewer`

Each reviewer has a focused checklist; they de-duplicate findings by check ID. The base reviewer never flags crypto/WaaS issues; crypto reviewer never flags WaaS issues. Layering keeps each agent's SRP clean.

A future `viv-workflows` would automate this composition; today the consumer (or main session) sequences the reviewers.

## How to vendor

```bash
# Clone the repos
git clone git@github.com:viblocks/viv-agents.git ~/AI/vault/viv-agents
git clone git@github.com:viblocks/viv-skills.git ~/AI/vault/viv-skills

# Vendor agents — copy the .md files directly
mkdir -p /path/to/project/.claude/agents

# Example: NestJS+crypto backend (3 layers of agents)
cp ~/AI/vault/viv-agents/backend/backend-implementer.md         /path/to/project/.claude/agents/
cp ~/AI/vault/viv-agents/backend/backend-reviewer.md            /path/to/project/.claude/agents/
cp ~/AI/vault/viv-agents/backend/backend-crypto-implementer.md  /path/to/project/.claude/agents/
cp ~/AI/vault/viv-agents/backend/backend-crypto-reviewer.md     /path/to/project/.claude/agents/

# Vendor _shared/ if you're vendoring any reviewer
cp -r ~/AI/vault/viv-agents/_shared /path/to/project/.claude/agents/

# Vendor the skills the agents declare they consume
mkdir -p /path/to/project/.claude/skills
cp -r ~/AI/vault/viv-skills/backend/nestjs-backend  /path/to/project/.claude/skills/
cp -r ~/AI/vault/viv-skills/backend/crypto-backend  /path/to/project/.claude/skills/
cp -r ~/AI/vault/viv-skills/discipline/root-cause-discipline /path/to/project/.claude/skills/

# Commit
cd /path/to/project
git add .claude/agents .claude/skills
git commit -m "Add viv-agents (backend+crypto stack) and viv-skills"
```

Agent files reference `.claude/agents/_shared/<file>.md` (project-rooted paths) — they resolve correctly at runtime when vendored as shown.

## Filling in placeholders

| Placeholder | Replace with |
|---|---|
| `<backend-services>/**` | Your backend service paths (e.g. `services/core/**`) |
| `<shared-packages>/**` | Your shared package paths (e.g. `packages/shared/**`) |
| `<frontend-app>/**` | Your frontend paths (e.g. `services/ui/**`) |
| `<service-name>` | Your test command target (e.g. for `pnpm --filter <service-name> run test`) |

Search-and-replace at vendor time, or leave as documentation.

## Why vendoring instead of a package manager?

Same reasoning as `viv-skills`: agents are static text files. Direct copy is the simplest mechanism that works correctly. Updates require manual re-copy — accepted trade-off.

## Adding a new agent

1. Pick the domain (or propose new — see ADR-007 on open vocabulary)
2. Pick the tier (or add a new one — see ADR-013)
3. Create `<domain>/<name>.md` with the SOLID-compliant frontmatter
4. Sanitize body of project-specific knowledge (use placeholders)
5. Update tables in this README
6. Commit

## Known limitations

1. **`description` is dual-purpose** — Claude Code uses it for auto-suggested dispatch. Sanitized descriptions trade precision for portability.
2. **Implementer ↔ reviewer pairing not declarative** — agents don't declare paired reviewers (workflow concern, see ADR-005). Naming convention does the work.
3. **Routing is the consumer's responsibility** — no built-in path → agent mapping.
4. **Enforcement is optional and external** — `behavior:` is declarative; no enforcement in this repo.
5. **Skills referenced are advisory at vendor time** — vendoring an agent without its skills means the agent fails at runtime.
6. **`type` and `domain` are recommended vocabulary, not enums** — see ADR-007.
7. **The `infra-devops-implementer` is the only agent with `modifies_meta_config: true`** — it edits CI/CD workflows. Consumers wiring enforcement should grant accordingly.
8. **Layered reviewers may have overlapping check IDs across tiers** — aggregator de-duplicates by ID. Today done by hand; a future workflow component would automate.

## Skill design conventions inherited from viv-skills

- **Cross-skill references** in agent bodies use **bare skill names** (e.g. `crypto-backend/24-threat-model.md`). No domain prefix.
- **Pattern numbering**: gaps are intentional. Don't renumber.
- **Vendoring**: upstream lives at `<domain>/<name>.md`. Consumer's `.claude/agents/<name>.md`.

## License

TBD
