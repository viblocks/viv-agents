## Context Navigation (Graphify)

This repo is **markdown-only**. Graphify's AST mode produces no graph for non-code, so the 3-layer rule collapses to 2 layers.

### 2-Layer Query Rule (markdown-only repo)
1. **First:** query the Obsidian vault (`logs/`, `architecture/decisions.md`) for project-level decisions and recent context.
2. **Second:** read raw `.md` files when answering domain questions or editing.

### When to use Graphify
- Not applicable today — `graphify update .` will report "No code files found".
- If code is added later, run `graphify update . --obsidian --obsidian-dir ~/AI/vault/graphify/<this-repo>` and switch to the standard 3-layer rule.

### Do NOT
- Don't manually modify files inside `graphify-out/` (none today, but future-proof).
- Don't re-read every file in a session — use index files (READMEs, SKILL.md, etc.) first.
