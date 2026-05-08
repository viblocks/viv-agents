# viv-agents

Reusable Claude Code typed-agent definitions, organized by domain. **Pure descriptors** — no enforcement, no routing, no workflow orchestration. Companion repo to [viv-skills](https://github.com/viblocks/viv-skills).

## What's an agent (in this repo)?

An agent is a single `.md` file at `<domain>/<name>.md`, with:
- Frontmatter declaring identity, knowledge (skills consumed), tools, and behavior assertions
- Body containing the agent's system prompt (principles, invocation, checklists, constraints)

Agents are read on-demand by Claude Code at runtime. The file format matches Claude Code's standard `.claude/agents/<name>.md` — no transformation needed at vendor time except copying.

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
├── backend/
│   ├── nestjs-crypto-implementer.md
│   └── nestjs-crypto-reviewer.md
├── frontend/
│   ├── reactjs-crypto-implementer.md
│   └── reactjs-crypto-reviewer.md
├── devops/
│   ├── infra-devops-implementer.md
│   └── infra-devops-reviewer.md
├── security/
│   └── security-reviewer.md
└── testing/
    └── dev-testing-strategy-reviewer.md
```

8 agents total. Domain subdirectories are organizational only — they're invisible after vendoring.

## Available agents by domain

### `backend/` — NestJS + Prisma + Kafka

| Agent | Type | Required skills | Optional skills |
|---|---|---|---|
| `nestjs-crypto-implementer` | implementer | `nestjs-backend`, `crypto-backend`, `root-cause-discipline` | `prisma-patterns`, `waas-backend` |
| `nestjs-crypto-reviewer` | reviewer | `nestjs-backend`, `crypto-backend`, `root-cause-discipline` | `prisma-patterns`, `waas-backend`, `web-security` |

### `frontend/` — React + TanStack Query + Zustand

| Agent | Type | Required skills | Optional skills |
|---|---|---|---|
| `reactjs-crypto-implementer` | implementer | `react-frontend`, `react-crypto-frontend`, `root-cause-discipline` | `waas-frontend` |
| `reactjs-crypto-reviewer` | reviewer | `react-frontend`, `react-crypto-frontend`, `root-cause-discipline` | `waas-frontend`, `web-security` |

### `devops/` — Containerization, CI/CD, IaC

| Agent | Type | Required skills | Optional skills |
|---|---|---|---|
| `infra-devops-implementer` | implementer | `docker-patterns`, `ci-cd-patterns`, `infra-cloud`, `root-cause-discipline` | — |
| `infra-devops-reviewer` | reviewer | same as implementer | — |

### `security/` — OWASP-aligned

| Agent | Type | Required skills | Optional skills |
|---|---|---|---|
| `security-reviewer` | reviewer | `web-security`, `root-cause-discipline` | — |

### `testing/` — Test strategy and CI gates

| Agent | Type | Required skills | Optional skills |
|---|---|---|---|
| `dev-testing-strategy-reviewer` | reviewer | `dev-testing-strategy`, `root-cause-discipline` | — |

## Frontmatter contract (SOLID-compliant)

Each agent file declares **only intrinsic properties** of the agent:

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
  modifies_meta_config: <bool>      # default false; only true for infra-devops-implementer
  modifies_source_code: <bool>      # default true for implementers, false for reviewers
  uses_network: <bool>              # default false
  performs_destructive_git: <bool>  # default false
---
```

Every agent declares **all four behavior fields explicitly**. No defaults are inferred from the agent name or type — the contract is fully machine-readable.

**Out of scope** (explicitly excluded; lives in other components):
- ❌ `scope.paths` — that's routing, not agent identity
- ❌ `post_implementation_chain` — that's workflow, not agent identity
- ❌ Hook script names — that's enforcement implementation, not agent contract
- ❌ Marker file paths — that's enforcement mechanism, not agent contract

See `architecture/decisions.md` for the full SOLID rationale.

## Composability — how this fits with other components

`viv-agents` declares contracts. To compose the full typed-agents strategy, possible additional components include:

| Component | What it does | Status |
|---|---|---|
| **viv-skills** | Knowledge each agent consumes (`skills.required` + `skills.optional`) | [Available](https://github.com/viblocks/viv-skills) |
| **viv-hooks** | Enforcement layer that reads `behavior:` and blocks violations | Not yet extracted |
| **viv-routing** | Path → agent dispatch automation | Not yet extracted |
| **viv-workflows** | Implementer → reviewer chains, post-implementation gates | Not yet extracted |

These are *possible extensions*, not commitments. `viv-agents` works **standalone in behavioral mode**: the user/LLM dispatches manually based on the agent's description. No automation required.

## What to install per project type

| Project type | Agents to vendor (paths from this repo) |
|---|---|
| NestJS backend with crypto | `backend/nestjs-crypto-implementer.md` + `backend/nestjs-crypto-reviewer.md` |
| React UI with crypto | `frontend/reactjs-crypto-implementer.md` + `frontend/reactjs-crypto-reviewer.md` |
| Full crypto platform (backend + frontend) | both backend + both frontend |
| Anything with CI/CD | + `devops/infra-devops-implementer.md` + `devops/infra-devops-reviewer.md` |
| User-facing API with auth | + `security/security-reviewer.md` |
| Anything with non-trivial test suite | + `testing/dev-testing-strategy-reviewer.md` |

## How to vendor

```bash
# Clone the repos
git clone git@github.com:viblocks/viv-agents.git ~/AI/vault/viv-agents
git clone git@github.com:viblocks/viv-skills.git ~/AI/vault/viv-skills

# Vendor agents — copy the .md file directly into .claude/agents/
mkdir -p /path/to/project/.claude/agents
cp ~/AI/vault/viv-agents/backend/nestjs-crypto-implementer.md /path/to/project/.claude/agents/
cp ~/AI/vault/viv-agents/backend/nestjs-crypto-reviewer.md    /path/to/project/.claude/agents/

# If vendoring any reviewer, also vendor _shared/
cp -r ~/AI/vault/viv-agents/_shared /path/to/project/.claude/agents/

# Vendor the skills the agent declares it consumes (see frontmatter `skills:` field)
mkdir -p /path/to/project/.claude/skills
cp -r ~/AI/vault/viv-skills/backend/nestjs-backend  /path/to/project/.claude/skills/
cp -r ~/AI/vault/viv-skills/backend/crypto-backend  /path/to/project/.claude/skills/
cp -r ~/AI/vault/viv-skills/discipline/root-cause-discipline /path/to/project/.claude/skills/

# Commit
cd /path/to/project
git add .claude/agents .claude/skills
git commit -m "Add viv-agents and viv-skills"
```

The agent files reference `.claude/agents/_shared/<file>.md` (project-rooted paths). After vendoring as shown, those paths resolve correctly at runtime.

## Filling in placeholders

Agent bodies use placeholders for project-specific paths:

| Placeholder | Replace with |
|---|---|
| `<backend-services>/**` | Your project's backend service paths (e.g. `services/core/**`, `services/api/**`) |
| `<shared-packages>/**` | Your project's shared package paths (e.g. `packages/shared/**`, `libs/contracts/**`) |
| `<frontend-app>/**` | Your project's frontend paths (e.g. `services/ui/**`, `apps/web/**`) |
| `<service-name>` | Your project's test command target (e.g. for `pnpm --filter <service-name> run test`) |

Search-and-replace at vendor time, or leave them as documentation of the intended scope.

## Why vendoring instead of a package manager?

Same reasoning as `viv-skills`: agents are static text files. There's no runtime dependency to resolve, no version compatibility matrix, no build step. The simplest mechanism that works correctly is direct copy.

Trade-off: updates require manual re-copy. Trade-off accepted because the alternative (submodules, package manager) adds complexity that doesn't pay back until many consumers exist.

## Adding a new agent to this repo

1. Pick the domain it belongs to (or propose a new one — see ADR-007 on open vocabulary)
2. Create the file: `<domain>/<agent-name>.md`
3. Add frontmatter with the SOLID-compliant contract (identity, skills, tools, behavior)
4. Sanitize the body of any project-specific knowledge (paths, issue numbers, brand names — replace with placeholders)
5. Update the table in this README under the appropriate domain section
6. Commit

## Known limitations

These are real trade-offs in the current design.

1. **`description` is dual-purpose**. Claude Code uses `description` for auto-suggested dispatch. Sanitized descriptions (no project paths) trade dispatch precision for portability. Rich domain vocabulary (Kafka, outbox, Prisma) compensates partially.

2. **Implementer ↔ reviewer pairing is not declarative**. The agent does not declare its paired reviewer (workflow orchestration was excluded for SRP — see ADR-005). Consumers infer pairing by naming convention (`X-implementer` ↔ `X-reviewer`). A future workflow component would own this concern explicitly.

3. **Routing is the consumer's responsibility**. There is no built-in path → agent mapping. Each consumer wires this up manually.

4. **Enforcement is optional and external**. The `behavior:` block is a declaration; nothing in `viv-agents` enforces it. Consumers can use their own hooks, a future enforcement repo, or no enforcement at all (behavioral mode).

5. **Skills referenced are advisory at vendor time**. `viv-agents` does not validate that consumed skills are actually present. If you vendor `nestjs-crypto-implementer` without vendoring `nestjs-backend`, the agent will fail at runtime when it tries to read the skill.

6. **`type` and `domain` are recommended vocabulary, not enforced enums**. New domains (`mobile`, `data-engineering`) and new types are accepted. Tooling that depends on the recommended set should warn, not block. See ADR-007.

7. **The `infra-devops-implementer` is the only agent with `modifies_meta_config: true`**. It edits CI/CD workflows and build scripts — files that may overlap with the consumer's own automation. Consumers wiring up enforcement should grant write access on `.github/workflows/**`, `Makefile`, `Dockerfile*`, `scripts/**` to this agent only.

## Skill design conventions inherited from viv-skills

- **Cross-skill references** in agent bodies use **bare skill names** (e.g. `crypto-backend/24-threat-model.md`). No domain prefix.
- **Pattern numbering**: gaps are intentional and stable. Cross-references depend on stable IDs — don't renumber.
- **Vendoring path mismatch**: upstream lives at `<domain>/<name>.md` for organization. Consumer's `.claude/agents/<name>.md` (no domain prefix).

## License

TBD
