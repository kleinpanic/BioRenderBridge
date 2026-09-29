# Contributing

This repository is initialized but implementation has not started. Read `.planning/PROJECT.md`, `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`, and `SECURITY.md` first.

Changes must stay within the active phase and the repository root. Do not relocate neighboring projects or automatically create worktrees. Initialization is not permission to begin execution.

Use Swift for the product. Propose dependencies with exact release provenance, license, transitive dependency review, and the capability they supply. Do not infer a released SDK's behavior from its main branch documentation.

Every protocol claim must name its MCP revision and tested harness/upstream versions. Separate documented behavior, observations, hypotheses, and verified implementation. Preserve structured results and metadata rather than reducing MCP to text-only responses.

Verification must include malformed input, expiry/rotation, concurrent clients, cancellation, sleep/wake, network loss, and explicit-login-only behavior. Use a controllable fake upstream for deterministic failure tests; live checks are an additional acceptance layer. Do not create paid figures for testing without deliberate authorization.

Never submit credentials or private diagnostic dumps. Public issue reports should contain only sanitized error codes, timestamps, versions, and reproduction steps.

Before opening a pull request, provide the changed behavior, requirement IDs, tests actually run, and remaining limits. No success badges or release claims before the associated evidence exists.
