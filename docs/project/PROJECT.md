# BioRenderBridge

## What This Is

A native Swift adapter for macOS users that provides one locally managed BioRender MCP connection to Codex, Claude Code, and OpenCode. It centralizes authorization, credential storage, refresh coordination, connection health, and explicit recovery so ordinary AI-client startup does not open Safari or leave failures unexplained.

## Core Value

Reliable BioRender access across all three clients with automatic recovery where supported and clear, deliberate recovery when user authorization is genuinely required.

## Requirements

### Validated

(None yet — this Swift project has not been implemented.)

### Active

- [ ] Connect Codex, Claude Code, and OpenCode to a shared local Swift adapter that forwards real BioRender MCP tool calls.
- [ ] Manage one upstream authorization centrally, with securely stored credentials and serialized refresh-token rotation.
- [ ] Use macOS Keychain for credential storage and keep secrets out of logs, command arguments, and tracked files.
- [ ] Refresh automatically when permitted and retry transient failures with bounded backoff.
- [ ] Keep normal startup browser-free; browser authorization must be a deliberate user action.
- [ ] Surface authentication failure explicitly to the calling harness and provide consistent status and reconnect controls.
- [ ] Recover across client restarts, concurrent clients, machine sleep/wake, and network interruptions without losing valid credentials.
- [ ] Verify version-appropriate handshake/discovery and real tool execution in all three installed clients.
- [ ] Verify actual expiration and later-session behavior before claiming persistent authentication.
- [ ] Keep dependencies minimal, pinned, and reviewed; do not download packages during normal startup.

### Out of Scope

- Implementation during this initialization session — user explicitly requested initialization only.
- Cleanup, relocation, or deletion of neighboring Swift projects/worktrees — a separate follow-up task awaiting user direction.
- A TypeScript/Node.js implementation — user explicitly prefers Swift due to supply-chain concerns.
- A universal multi-provider MCP gateway — initial scope is BioRender.
- Bypassing BioRender authorization or guaranteeing acceptance of revoked credentials — the service controls its authorization policy.
- Figure-generation product features — forward the upstream MCP tools rather than recreate BioRender.

## Context

On September 28–29, 2026, the originating user experienced repeated BioRender browser authorization prompts and later a Codex MCP startup failure. Legacy mcp-remote startup registrations were replaced with native HTTP connections. Codex currently uses a Python HTTP-headers helper because BioRender's WWW-Authenticate resource_metadata URL points to authorization-server metadata without the required protected-resource field. Claude Code and OpenCode use independent native OAuth connections.

Immediate authenticated startup checks passed, but later BioRender rejected a saved refresh token with HTTP 400 invalid_grant. A session_expired response had also been observed before an access token's JWT expiration. Reauthorization restored current connections. Successful immediate checks therefore do not prove overnight persistence.

The current service advertises offline_access and refresh_token. Observed access-token lifetime was 28,800 seconds; refresh tokens were opaque and no documented refresh-token lifetime was found. Actual tool calls with saved Codex and OpenCode credentials succeeded without authentication errors but returned zero search matches. Claude reported Connected. These are historical findings about the existing setup, not validation of this unbuilt Swift project.

Private diagnostic records and a Python credential helper exist outside this repository. Only sanitized findings belong here; never copy credentials or private account data into this repository. No vendor support message has been sent.

Initial source review found that the official Swift MCP SDK documents client/server support, stdio, Streamable HTTP, OAuth authorization-code flows, and custom token storage. Its exact released capabilities and transitive dependencies must be verified before adoption; repository main documentation alone is insufficient compatibility evidence.

Primary sources for subsequent research:
- https://github.com/modelcontextprotocol/swift-sdk
- https://github.com/modelcontextprotocol/swift-sdk/blob/main/Package.swift
- https://help.biorender.com/hc/en-gb/articles/30870978672157-How-to-use-the-BioRender-MCP-connector
- https://mcp.services.biorender.com/.well-known/oauth-authorization-server
- https://code.claude.com/docs/en/mcp
- https://opencode.ai/docs/mcp-servers/

## September 30 correction

Codex startup and a direct helper check again failed with `invalid_grant`; the prior authentication recovery failed persistence. The cause remains unproven. A Swift broker may prevent local races and improve recovery, but cannot be presumed to fix this rejection. The [audit](../biorender-audit.md) now requires causal investigation and a go/revise/blocked feasibility decision before Phase 2. Provider-limited behavior must remain an explicit blocker or scope decision, not be relabeled as a permanent fix.

Peekaboo also failed startup due to a daemon/socket ownership mismatch. Track it as separate operational work, outside this project's implementation scope. Neither failure is fixed by this documentation update.

## Constraints

- **Location:** the repository root is the canonical project home; do not relocate it or create alternative checkouts automatically.
- **Language:** Swift for the adapter; Apple frameworks preferred where appropriate.
- **Dependencies:** Small, explicit, reviewed dependency graph with reproducible version pinning. Swift is not presumed immune to supply-chain attacks.
- **Compatibility:** All three clients must be verified independently; one protocol test is not proof of installed client acceptance.
- **Security:** Preserve consent boundaries, protect local IPC, redact diagnostics, and use platform credential storage.
- **Behavior:** No random Safari launches, silent auth failures, unlimited retries, or unsupported permanent-authentication claims.
- **Session scope:** Initialize project artifacts only, then stop for further direction. Do not modify active harness configuration or install the adapter during setup.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build in Swift | Explicit user preference for native implementation and controlled dependencies | Accepted; implementation pending |
| One shared adapter for three harnesses | Centralize refresh coordination and reduce separate authorization chores | Conditional hypothesis; requires SPEC-04/05 evidence before implementation |
| Keychain-backed credential management | Avoid plaintext credentials in project/configuration artifacts | Required; implementation pending |
| Deliberate login, automatic refresh | Balance persistent access with the prohibition on surprise browser launches | Required; implementation pending |
| Separate present connectivity from persistence | Prior immediate-success claims failed overnight | Required verification boundary |
| Stop after initialization | User will provide more direction and wants Swift-directory cleanup handled separately | Explicit session boundary |

## Evolution

This document evolves at phase transitions and milestone boundaries.

After each phase transition, review active requirements, move verified requirements to Validated, record invalidated requirements with reasons, and update decisions and context. After each milestone, review all sections and confirm the core value and scope still reflect the user's intent.

---
*Last updated: 2026-09-29 during project initialization; initialization baseline; see STATE.md for current handoff.*
