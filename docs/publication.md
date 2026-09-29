# Publication boundary

Local `main` contains the complete GSD project and agent instructions. The machine pre-push guard prohibits publishing `.planning/`, `AGENTS.md`, and `CLAUDE.md` to public repositories. Keep that guard enabled.

Public GitHub branch `public` contains a reviewed documentation export: README, license, security/contribution guidance, audit, and sanitized project documents under `docs/project/`. Markdown links are rewritten to the exported paths. GSD configuration, state, agent instructions and research caches remain local.

Publish future updates by constructing a fresh reviewed export tree with the previous `public` commit as parent; never merge local `main` into `public`. Run secret scans on the proposed export and its history before pushing. No alternate working checkout or project relocation is needed.

The public repository is planning-only. Runtime validation and release claims must follow the requirements and roadmap gates.
