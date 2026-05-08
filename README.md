# viv-agents

Reusable Claude Code typed-agent definitions, organized by domain. **Pure descriptors** — no enforcement, no routing, no workflow orchestration. Companion repo to [viv-skills](https://github.com/viblocks/viv-skills).

## What's an agent (in this repo)?

An agent is a directory with:
- `AGENT.md` — frontmatter (identity, knowledge, tools, behavior) + system prompt body
- (optional) `references/`, `examples/` — supporting material

Agents are read on-demand by Claude Code at runtime. They're plain text — no runtime, no dependencies, no installation step beyond copying the directory into your project's `.claude/agents/`.

## Repository structure

```
viv-agents/
├── _shared/                          ← shared review formats (consumed by reviewers)
├── architecture/decisions.md         ← ADRs documenting design choices
├── backend/
│   ├── nestjs-crypto-implementer/
│   └── nestjs-crypto-reviewer/
├── frontend/
│   ├── reactjs-crypto-implementer/
│   └── reactjs-crypto-reviewer/
├── devops/
│   ├── infra-devops-implementer/
│   └── infra-devops-reviewer/
├── security/
│   └── security-reviewer/
└── testing/
    └── dev-testing-strategy-reviewer/
```

8 agents total. Domain subdirectories are organizational — when vendored, the inner agent directory itself goes into your project's `.claude/agents/`, so the upstream domain directory is invisible at runtime.

## Available agents by domain

### `backend/` — NestJS + Prisma + Kafka

| Agent | Type | Description | Skills consumed |
|---|---|---|---|
| `nestjs-crypto-implementer` | implementer | Writes NestJS backend code with crypto/blockchain concerns | `nestjs-backend`, `crypto-backend`, `prisma-patterns`, `waas-backend` (opt), `root-cause-discipline` |
| `nestjs-crypto-reviewer` | reviewer | Audits NestJS backend code (SOLID, outbox, idempotency, BigInt, threat model) | same as implementer |

### `frontend/` — React + TanStack Query + Zustand

| Agent | Type | Description | Skills consumed |
|---|---|---|---|
| `reactjs-crypto-implementer` | implementer | Writes React frontend with crypto-grade precision (BigInt, lifecycle SM, Error Boundaries) | `react-frontend`, `react-crypto-frontend`, `waas-frontend` (opt), `root-cause-discipline` |
| `reactjs-crypto-reviewer` | reviewer | Audits React frontend (precision, server state, security, semantic tokens) | same as implementer |

### `devops/` — Containerization, CI/CD, IaC

| Agent | Type | Description | Skills consumed |
|---|---|---|---|
| `infra-devops-implementer` | implementer | Writes Dockerfiles, compose, GitHub Actions, Makefile, IaC | `docker-patterns`, `ci-cd-patterns`, `infra-cloud`, `root-cause-discipline` |
| `infra-devops-reviewer` | reviewer | Audits infra hygiene (multi-stage, SHA pins, healthchecks, shellcheck) | same as implementer |

### `security/` — OWASP-aligned

| Agent | Type | Description | Skills consumed |
|---|---|---|---|
| `security-reviewer` | reviewer | Audits code against OWASP Top 10 — injection, access control, misconfig, XSS, supply chain | `web-security`, `root-cause-discipline` |

### `testing/` — Test strategy and CI gates

| Agent | Type | Description | Skills consumed |
|---|---|---|---|
| `dev-testing-strategy-reviewer` | reviewer | Audits testing strategy (platform decoupling, suite uniformity, performance gates, isolation) | `dev-testing-strategy`, `root-cause-discipline` |

## SOLID-compliant frontmatter contract

Each `AGENT.md` declares **only intrinsic properties** of the agent:

```yaml
---
name: <agent-name>
type: implementer | reviewer
domain: backend | frontend | devops | security | testing
description: <competence description, no project paths>
skills: [<skill-name>, ...]            # consumed by skill loader
tools: [<Claude Code tool>, ...]       # consumed by Claude Code runtime
behavior:                              # consumed by enforcement layers (if present)
  modifies_meta_config: <bool>
  uses_network: <bool>
  performs_destructive_git: <bool>
  modifies_source_code: <bool>         # reviewer-specific
---
```

**Out of scope** (lives in other components, never in agent frontmatter):
- ❌ `scope.paths` — that's routing, not agent identity
- ❌ `post_implementation_chain` — that's workflow, not agent identity
- ❌ Hook script names — that's enforcement implementation, not agent contract
- ❌ Marker file paths — that's enforcement mechanism, not agent contract

See `architecture/decisions.md` for the full SOLID rationale.

## Composability — how this fits with other components

`viv-agents` declares contracts. To compose the full typed-agents strategy you may add:

| Component | What it does | Repo |
|---|---|---|
| **viv-skills** | Knowledge each agent consumes (`skills:` field) | [viblocks/viv-skills](https://github.com/viblocks/viv-skills) |
| **viv-hooks** *(future)* | Enforcement layer that reads `behavior:` and blocks violations | — |
| **viv-routing** *(future)* | Path → agent dispatch automation | — |
| **viv-workflows** *(future)* | Implementer → reviewer chains, post-implementation gates | — |

**Without those, agents work in behavioral mode**: the user/LLM dispatches manually, the agent does its job based on its system prompt, no automatic enforcement. `viv-agents` is **standalone usable**.

## What to install per project type

| Project type | Agents to vendor (paths from this repo) |
|---|---|
| NestJS backend with crypto | `backend/nestjs-crypto-implementer` + `backend/nestjs-crypto-reviewer` |
| React UI with crypto | `frontend/reactjs-crypto-implementer` + `frontend/reactjs-crypto-reviewer` |
| Full crypto platform (backend + frontend) | both backend + both frontend |
| Anything with CI/CD | + `devops/infra-devops-implementer` + `devops/infra-devops-reviewer` |
| User-facing API with auth | + `security/security-reviewer` |
| Anything with non-trivial test suite | + `testing/dev-testing-strategy-reviewer` |

## How to vendor

```bash
# Clone this repo somewhere
git clone git@github.com:viblocks/viv-agents.git ~/AI/vault/viv-agents

# Vendor only the inner agent directory — the domain wrapper is just for organization
cp -r ~/AI/vault/viv-agents/backend/nestjs-crypto-implementer /path/to/project/.claude/agents/
cp -r ~/AI/vault/viv-agents/backend/nestjs-crypto-reviewer    /path/to/project/.claude/agents/

# AGENT.md is the standard Claude Code agent file. Rename it during vendoring if your
# Claude Code version expects a different filename, or symlink for compatibility.
# Rename to <agent-name>.md flat-file form (legacy Claude Code format):
mv /path/to/project/.claude/agents/nestjs-crypto-implementer/AGENT.md \
   /path/to/project/.claude/agents/nestjs-crypto-implementer.md
rmdir /path/to/project/.claude/agents/nestjs-crypto-implementer

# Also vendor _shared/ if you're vendoring any reviewer
cp -r ~/AI/vault/viv-agents/_shared /path/to/project/.claude/agents/

# Vendor the skills the agent declares it consumes (see frontmatter `skills:` field)
# from viv-skills. Example for nestjs-crypto-implementer:
cp -r ~/AI/vault/viv-skills/backend/nestjs-backend  /path/to/project/.claude/skills/
cp -r ~/AI/vault/viv-skills/backend/crypto-backend  /path/to/project/.claude/skills/
# ... etc

# Adjust _shared/ relative paths in each AGENT.md if you flattened the structure
# (the default `../../_shared/` assumes the directory layout was preserved)

# Commit
cd /path/to/project
git add .claude/agents .claude/skills
git commit -m "Add viv-agents and viv-skills"
```

## Filling in placeholders

Agent bodies use placeholders for project-specific paths:

| Placeholder | Replace with |
|---|---|
| `<backend-services>/**` | Your project's backend service paths (e.g. `services/core/**`, `services/api/**`) |
| `<shared-packages>/**` | Your project's shared package paths (e.g. `packages/shared/**`, `libs/contracts/**`) |
| `<frontend-app>/**` | Your project's frontend paths (e.g. `services/ui/**`, `apps/web/**`) |
| `<service-name>` | Your project's test command target (e.g. for `pnpm --filter <service-name> run test`) |

Placeholders appear in:
- AGENT.md prose (paths mentioned in checklist phases, scope sections)
- The `_shared/test-freshness-audit.md` Parametrization section (filled per-reviewer in their AGENT.md)

Search-and-replace at vendor time, or leave them as documentation of the intended scope.

## Why vendoring instead of a package manager?

Same reasoning as `viv-skills`: agents are static text files. There's no runtime dependency to resolve, no version compatibility matrix, no build step. The simplest mechanism that works correctly is direct copy.

Trade-off: updates require manual re-copy. Trade-off accepted because the alternative (submodules, package manager) adds complexity that doesn't pay back until many consumers exist.

## Adding a new agent to this repo

1. Pick the domain it belongs to (or propose a new one — see ADR-007 on open vocabulary)
2. Create the directory under that domain: `<domain>/<agent-name>/`
3. Add `AGENT.md` with the SOLID-compliant frontmatter (identity, skills, tools, behavior — nothing else)
4. Sanitize the body of any project-specific knowledge (paths, issue numbers, brand names — replace with placeholders)
5. Update the table in this README under the appropriate domain section
6. Commit

## Known limitations

These are real trade-offs in the current design. They're documented honestly so consumers know what to expect.

1. **`description` is dual-purpose**. Claude Code uses `description` to suggest agent dispatch. Sanitized descriptions (no project paths) trade dispatch precision for portability. Rich domain vocabulary (Kafka, outbox, Prisma) compensates partially.

2. **Implementer ↔ reviewer pairing is not declarative**. The agent does not declare its paired reviewer (post_implementation_chain was excluded for SRP — see ADR-005). For now, consumers infer pairing by naming convention (`X-implementer` ↔ `X-reviewer`). A future `viv-workflows` will own this concern.

3. **Routing is the consumer's responsibility**. There is no built-in path → agent mapping. Each consumer wires this up manually until `viv-routing` exists.

4. **Enforcement is optional and external**. The `behavior:` block is a declaration; nothing in `viv-agents` enforces it. Consumers can use their own hooks, future `viv-hooks`, or no enforcement at all (behavioral mode).

5. **Skills referenced as optional are advisory**. `viv-agents` does not validate that consumed skills are actually present. If you vendor `nestjs-crypto-implementer` without vendoring `nestjs-backend`, the agent will fail at runtime when it tries to read the skill — discovery is the consumer's responsibility.

6. **`type` and `domain` are recommended vocabulary, not enforced enums**. New domains (`mobile`, `data-engineering`) and new types are accepted. Tooling that depends on the recommended set should warn, not block. See ADR-007.

## Skill design conventions inherited from viv-skills

- **Cross-skill references**: pattern bodies use bare skill names (e.g. `crypto-backend/24-threat-model.md`). Agents inherit the same convention — no domain prefix.
- **Pattern numbering**: gaps are intentional and stable. Cross-references depend on stable IDs — don't renumber.
- **Vendoring path mismatch**: upstream lives at `<domain>/<name>/` for organization. Consumer's `.claude/agents/<name>/` (no domain prefix).

## License

TBD
