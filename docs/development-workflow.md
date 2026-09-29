# Development workflow

Initialization follows the installed GSD new-project workflow and canonical PROJECT, REQUIREMENTS, ROADMAP, STATE, configuration, and project-research templates.

## Project configuration

- Autonomous bounded work, fine-grained phases, parallel independent work where authorized.
- Research, plan checking, source grounding, security review, and post-phase verification enabled.
- Automatic advancement disabled; initialization does not start phase execution.
- In-place repository work; automatic branch/worktree creation disabled.
- Planning documents tracked in Git.
- Current session models inherited. OpenCode review preference: `minimax/MiniMax-M3`, verified available at initialization; recheck availability before invocation. Claude and Codex reviewers use their native CLIs/subscriptions. Do not enable OpenRouter or Gemini without an explicit request.
- Brave/Firecrawl capabilities may be used through the operator's environment. No API keys are stored in this public repository. Disabled/unverified provider flags are not proof of credentials or connectivity.
- Relevant installed skills are selected for MCP design, security review, and verification. These are optional developer tooling, not dependencies needed to run the eventual product.

The canonical `gsd-tools query` CLI owns configuration and state mutations where handlers exist. Initial roadmap/state bodies are generated from canonical templates because the new-project workflow delegates their creation to the roadmapper.

## Resume boundary

Use `$gsd-next` from the repository root to inspect state. Do not auto-execute the first phase. The owner explicitly asked to finish initialization and then pause. Cleanup of the parent Swift directory is a separate task and must not be inferred from this roadmap.

## Evidence rules

Research is source-backed guidance, not shipped implementation. Phase research and verification live under `.planning/phases/NN-name/`. Persistent authorization requires observations after genuine token expiration and later sessions; mocks and freshly issued tokens do not substitute for that evidence.
