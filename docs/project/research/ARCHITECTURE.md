# Architecture Research

**Domain:** Native macOS BioRender MCP authentication broker
**Researched:** 2026-09-29
**Confidence:** MEDIUM — `gsd-tools query classify-confidence --provider websearch --verified`; primary sources checked, architecture not implemented or validated.

## Standard Architecture

### System Overview

Recommend one per-user Swift broker and one lightweight stdio frontend process per harness. Shared authorization does **not** imply sharing protocol sessions, request IDs, or figure state.

```text
Codex        Claude Code        OpenCode
  │               │                │
  └─ stdio ────────┼──── stdio ──────┘
          Swift frontend per client
                    │ authenticated local IPC
          per-user Swift broker (launchd)
          ├─ upstream connection contexts per client
          ├─ single-flight OAuth coordinator
          ├─ bounded request/retry dispatcher
          └─ redacted health state
                    │             │
               Keychain     HTTPS BioRender MCP/OAuth
```

This is a recommendation, not evidence that a particular Swift SDK release already implements every boundary. Phase 1 must test SDK and harness compatibility before committing to the transport design.

### Component Responsibilities

| Component | Responsibility | Proposed implementation |
|-----------|----------------|-------------------------|
| CLI frontend | MCP transport framing, per-client identity, clean EOF/shutdown | Swift executable `serve` subcommand; stdout exclusively protocol messages |
| Local broker | Sole credential owner; isolate requests and lifecycle by connection | Per-user launchd service; actor-managed state |
| Authorization coordinator | Explicit login, token refresh, generation tracking, recovery | Actor retaining one shared in-flight refresh task; actor isolation alone is insufficient across await points |
| Credential vault | Persist current token bundle and registration atomically | Security.framework Keychain, one versioned record; no tokens returned over IPC |
| Upstream adapter | Authenticated HTTP MCP, version-specific lifecycle and errors | Pinned Swift SDK behind a tested wrapper; provider-specific compatibility isolated |
| Dispatcher | ID/cursor mapping, deadlines, cancellation, retry classification | Request context keyed by frontend connection plus request ID |
| Status surface | Explain degraded state without surprise interaction | CLI status/reconnect first; optional SwiftUI later |

## Recommended Project Structure

```text
Sources/
├── BioRenderBridgeCLI/          # serve, status, deliberate login
├── BioRenderBridgeBroker/       # process lifecycle and IPC
├── BridgeProtocol/             # translation, request isolation
├── Authorization/              # state machine and refresh ownership
└── CredentialStore/            # Keychain boundary
Tests/
├── ProtocolConformanceTests/
├── AuthorizationStateTests/
├── IPCSecurityTests/
└── HarnessAcceptanceTests/
```

These are future module boundaries, not a request to generate source during initialization. Keep fake upstream servers and test credentials confined to tests.

## Architectural Patterns

### Pattern 1: Explicit protocol revision contract

Current MCP is **2026-07-28**, with per-request metadata and discovery. The legacy **2025-11-25** revision uses initialize/initialized negotiation. Do not call a legacy-only implementation current-spec compliant. Record downstream and upstream revisions independently; unsupported combinations must produce actionable errors. Dual-era translation is proposed only after SDK and BioRender probes establish feasibility. [Versioning](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning), [legacy lifecycle](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle).

Maintain a conformance matrix covering requested version, actual accepted revision, tool capability, unsupported features, and tested harness version. Never advertise resources, prompts, sampling, elicitation, subscriptions, or other extensions merely because the upstream advertises them; forwarding and isolation must exist first.

### Pattern 2: Single writer with durable credential generations

All clients await the same refresh operation. Capture credential generation before dispatch; reject stale completions after login/logout. Persist a successful replacement before releasing waiters. Preserve the previous refresh token when the response omits a replacement. Failed network refresh must not erase the credential record. A request with a known obsolete access-token generation may reuse the newer generation without triggering another refresh.

Rotation can make a lost response unrecoverable: the server may invalidate the old token before the new token reaches Keychain. Do not blindly retry ambiguous refresh requests; document that failure window. OAuth permits refresh expiration/revocation, so local coordination cannot promise perpetual authorization. [OAuth security BCP](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14).

### Pattern 3: Trusted IPC, untrusted payloads

Prefer authenticated XPC between signed bridge frontend and broker if deployment proves practical. Validate peers in both directions against the installed signing identity; do not assume an Apple-platform-code requirement fits a third-party app. Exact API availability and signing requirements need a macOS deployment spike. [Apple XPC peer requirements](https://developer.apple.com/documentation/xpc/xpcpeerrequirement).

A private Unix socket is a fallback for development, not equivalent same-user process isolation: directory 0700, socket 0600, peer UID checks, safe ownership/symlink handling, bounded frames, and no public TCP listener. Same-user malware remains in its threat model. Both designs accept only bridge operations, never arbitrary URLs or shell commands. No IPC operation exports OAuth tokens.

## Data Flow

### Request Flow

1. Frontend validates protocol envelope and assigns an isolated request context.
2. Broker checks capability/version and obtains usable authorization internally.
3. Dispatcher sends the upstream request with a unique correlated ID.
4. Results preserve schemas, structured content, errors, and metadata supported by the negotiated revision; cursors remain opaque and scoped.
5. Frontend emits the corresponding result. Diagnostics use stderr or private logs.

### State Management

Proposed states: `unauthorized`, `ready`, `refreshing`, `temporarilyUnavailable`, `reauthorizationRequired`, `credentialStoreLocked`. Separate upstream connectivity, local broker availability, and authorization health. A Keychain lock is not an OAuth revocation. An upstream MCP session loss is not necessarily a lost OAuth grant.

Only a deliberate local `login`/`reconnect` action may launch a browser. MCP requests, startup, retries, sleep/wake, and background health checks must never do so. Errors include a stable category, safe message, and reconnect command; never raw authorization URLs, tokens, or upstream error bodies.

### Retry Safety

Automatic replay is restricted to explicitly classified safe discovery/read operations and failures known to precede execution. A disconnected mutating figure request yields **outcome unknown**, not an automatic replay. Tool annotations alone are not sufficient authorization for retry. Cancellation stops local work where possible but does not claim to roll back server mutations. Apply separate bounded policies for transient discovery, transport recovery, and token refresh.

## Scaling Considerations

| Scale | Architecture adjustments |
|-------|--------------------------|
| One Mac, three harnesses | Primary target; one credential owner, independent request contexts |
| Many concurrent tool calls | Bound queued requests, response sizes, connection count, and timeouts; do not hold global locks during unrelated calls |
| Multiple macOS users | Separate launchd user domain and Keychain records; never share account tokens |
| Multiple BioRender accounts/providers | Deferred; would require explicit authorization namespaces and cache isolation |

## Anti-Patterns

- Three independent frontends refreshing the same token: recreates the concurrency failure.
- A generic transparent HTTP proxy: risks token forwarding, SSRF, and protocol/version confusion.
- One global MCP session: risks routing one client's notifications, IDs, or cancellation into another.
- Treating JWT expiration as proof of service authorization: server revocation or session policy can intervene.
- Hiding discovery failures behind empty tools: makes a broken connection look healthy.
- Treating tool text as configuration: upstream content is data, never executable setup instructions.

## Integration Points

| Boundary | Contract | Acceptance evidence |
|----------|----------|---------------------|
| Harness → frontend | stdio MCP; supported revision declared | Fresh processes in all three clients, discovery and a real read-only call |
| Frontend → broker | Authenticated bounded IPC, no credential export | Wrong peer/UID rejected; malformed frames bounded; broker restart visible |
| Broker → BioRender | HTTPS HTTP MCP plus OAuth | Version/capability audit; provider metadata exceptions documented |
| Broker → Keychain | Small versioned credential bundle | Rotation/restart/locked-store tests; logs and files contain no secrets |
| Browser → login callback | User-started code flow, PKCE, state, exact redirect validation | Wrong state/issuer/replayed callback rejected; callback listener closed |

Native OAuth should use the external browser with PKCE and a narrowly bound redirect listener; a shipped application secret must not be treated as confidential proof of client identity. [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252.html).

## Build Order and Open Decisions

1. Audit BioRender metadata and actual protocol revisions; pin SDK compatibility and threat model.
2. Implement/test Keychain, refresh state machine, and explicit authorization against fixtures.
3. Establish secure broker/frontend lifecycle and per-client protocol adapter.
4. Add real harness integrations with reversible migration and rollback.
5. Run actual expiry, later-session, concurrent-start, sleep/wake, and fault-injection acceptance before persistence claims.

Phase research must resolve XPC deployment/signing, exact retryable BioRender operations, SDK modern/legacy support, and provider refresh lifetime. None is validated by this research.

## Sources

- [MCP current versioning](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning)
- [MCP legacy lifecycle](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle)
- [MCP stdio](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio)
- [MCP Streamable HTTP](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http)
- [MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)
- [MCP security practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices)
- [OAuth security BCP](https://www.rfc-editor.org/rfc/rfc9700.html)
- [Native-app OAuth](https://www.rfc-editor.org/rfc/rfc8252.html)
- [Apple XPC peer requirements](https://developer.apple.com/documentation/xpc/xpcpeerrequirement)

---
*Architecture research for: BioRenderBridge*
*Researched: 2026-09-29*
