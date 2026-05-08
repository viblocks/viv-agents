# Architecture Decisions — viv-agents

Sequential ADRs documenting the design of this repo. New ADRs append; superseded ADRs update their Status line to point to the replacement.

---

## ADR-001 — Agents in a separate repo from skills

**Status**: Accepted
**Decision**: viv-agents is a standalone repo, not a subdirectory of viv-skills.
**Why**: SRP at the repo level. Skills change when domain knowledge evolves (new patterns, sanitization). Agents change when the role definition evolves (new skill consumed, new tool, new behavior assertion). These are independent change cycles. Co-locating them would create a god-repo whose history mixes two unrelated reasons-to-change.
**Alternatives considered**:
- Subdirectory `agents/` inside viv-skills — rejected (violates repo-level SRP).
- Submodule from viv-agents to viv-skills — rejected (submodules add fragility; the user explicitly rejected this for skills under ADR-002 of viv-skills).
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-002 — Frontmatter as declarative contract (DIP)

**Status**: Accepted
**Decision**: Each AGENT.md frontmatter declares contracts (identity, knowledge, tools, behavior) that external systems consume read-only. The agent never references those external systems.
**Why**: Dependency Inversion Principle. The agent (high-level policy: "what this expert does") must not depend on low-level mechanisms (routing, enforcement, workflow orchestration). Inverted: external systems depend on the contract surface the agent declares.
**Alternatives considered**:
- Prose-only AGENT.md (consumers parse markdown) — rejected (no machine-readable contract; brittle for tooling).
- Separate contract files (e.g. `agent.json` + `agent.md`) — rejected (frontmatter is the standard Claude Code agent format; introducing parallel files breaks runtime compatibility).
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-003 — Zero coupling to enforcement layer (LSP)

**Status**: Accepted
**Decision**: AGENT.md never references hooks, marker files, enforcement scripts, or any specific enforcement mechanism. Behavioral assertions are intrinsic ("this agent modifies meta-config: false"), not externally granted permissions ("this agent has self_mod permission").
**Why**: Liskov Substitution. Any enforcement layer (none, custom, future viv-hooks) must be substitutable without modifying agents. The agent declares facts about itself; enforcement layers read those facts.
**Alternatives considered**:
- Embed hook-script names in AGENT.md (`hook: enforce-routing.sh`) — rejected (couples agent to a specific implementation; LSP violation).
- Use OS-level capabilities (network, filesystem) instead of project-level (meta-config) — rejected (too low-level; doesn't capture domain-meaningful constraints like "this agent edits CI/CD config").
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-004 — Sanitization via fill-in templates (OCP)

**Status**: Accepted
**Decision**: Project-specific paths in agent prose and examples are replaced by placeholders (e.g. `<backend-services>/**`). The consumer fills them in at vendor time. No project-specific identifier (path, issue number, brand name) survives extraction.
**Why**: Open/Closed Principle. Other projects must extend agents (apply them to their own paths) without modifying the source. Hardcoded paths force modification.
**Alternatives considered**:
- Conservative (leave viblocks paths as examples) — rejected (consumer has to know to delete them; error-prone).
- Strict templating engine (Handlebars/Jinja) — rejected (adds tooling burden; YAML comments are sufficient).
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-005 — Routing, workflow, enforcement live outside this repo

**Status**: Accepted
**Decision**: The following concerns are explicitly out of scope for viv-agents:
- **Routing**: which path → which agent. Lives in consumer config (or future `viv-routing`).
- **Workflow**: order of agent invocation, post-implementation chains, gates. Lives in consumer config (or future `viv-workflows`).
- **Enforcement**: hard-deny rules, marker files, role detection. Lives in consumer config (or future `viv-hooks`).
**Why**: Single Responsibility at the repo level. Each of these concerns has independent change cycles and independent consumers. Bundling them into viv-agents would force consumers to vendor concerns they may not need (ISP violation).
**Alternatives considered**:
- Bundle routing config in viv-agents (`routing-table.template.json`) — rejected (couples agent definitions to project structure; some consumers may dispatch manually without a routing table).
- Bundle workflow config (`chains.yml`) — rejected (couples agents to a specific orchestration model; some consumers may use ad-hoc workflows).
**Consequence**: viv-agents works **standalone in behavioral mode** (user/LLM dispatches manually). External components are optional and pull-only.
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-006 — Agent body sanitization (separate from frontmatter)

**Status**: Accepted
**Decision**: The body of AGENT.md (system prompt) is sanitized of project-specific knowledge using the same rules as skill extraction (viv-skills ADR-002): fill-in templates for paths, removal of project-specific issue numbers and brand names, generalization of domain-specific examples.
**Why**: Frontmatter purity is necessary but not sufficient. A frontmatter declaring `domain: backend` paired with a body that says "review the blacklist pipeline (pollers → enrichment → guardian)" still couples the agent to viblocks-ai. Sanitization must extend to prose.
**Alternatives considered**:
- Leave bodies untouched as "examples" — rejected (consumer has to mentally translate; error-prone; defeats the purpose of extraction).
**Source**: [[2026-05-08-typed-agents-extraction-design]]

---

## ADR-007 — Vocabulary over enums for `type` and `domain`

**Status**: Accepted
**Decision**: `type` (implementer | reviewer) and `domain` (backend | frontend | devops | security | testing) are documented as **recommended vocabulary**, not enforced enums. Tooling validates against the recommended set but does not block unknown values; new types/domains can be added by consumers without forking.
**Why**: OCP. A closed enum forces a fork or PR upstream every time a consumer needs a new type (e.g. `data-engineer-implementer`) or domain (e.g. `mobile`). Open vocabulary keeps extension friction low.
**Alternatives considered**:
- Closed enum with strict validation — rejected (forces upstream changes for downstream extensions).
- Free-form strings with no recommendation — rejected (loses consistency benefit; tooling can't generate domain-grouped READMEs).
**Source**: [[2026-05-08-typed-agents-extraction-design]]
