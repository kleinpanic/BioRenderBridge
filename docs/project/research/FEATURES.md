# Feature Research

**Domain:** Shared native BioRender MCP access for three local AI harnesses
**Researched:** 2026-09-29
**Confidence:** MEDIUM — classified by GSD `websearch --verified`; normative sources checked, proposed features unbuilt.

## Feature Landscape

### Table Stakes (Users Expect These)

| Feature | Why expected | Complexity | Testable acceptance |
|---------|--------------|------------|---------------------|
| Honest protocol compatibility | Proper MCP rather than an ad hoc tool wrapper | HIGH | Record pinned SDK/harness/upstream revisions; test supported legacy handshake and/or modern discovery paths; reject unsupported revisions |
| Complete tool forwarding | Preserve useful BioRender access | HIGH | Consume all discovery pages; preserve names, schemas, supported content, and errors; no silently omitted tool |
| One upstream authorization | Eliminate three independent login chores | HIGH | Three fresh harness sessions operate using broker-owned authorization; credentials absent from frontend IPC/config |
| Serialized refresh | Avoid competing token rotations | HIGH | Concurrent expiry requests cause one refresh; all awaiters receive current generation; stale writers cannot overwrite login/logout |
| Durable Keychain storage | Survive normal process restarts safely | MEDIUM | Restart after rotation; test locked store and failed writes; no plaintext token fallback |
| Browser-free ordinary operation | Central user pain | MEDIUM | Instrument browser-launch boundary: zero calls from startup, tool requests, expired access, invalid_grant, retries, or wake |
| Deliberate secure login | Recover only when consent is required | HIGH | Explicit command opens correct authorization flow; PKCE/state/issuer and callback replay tests pass |
| Visible actionable failure | Avoid silently broken tools | MEDIUM | Auth, offline, permission, locked Keychain, and protocol failures have distinct redacted statuses in each harness |
| Safe bounded recovery | Avoid duplicates and retry storms | HIGH | Timeouts on potentially executed figure mutations cause no replay; safe operation retries capped; invalid_grant ends refresh loop |
| Independent client lifecycles | Shared auth must not mix requests | HIGH | Identical IDs/cursors from concurrent clients remain isolated; disconnecting one leaves others usable |
| Trustworthy persistence evidence | Prior immediate success failed later | HIGH | Actual expired-token and later-session checks, sleep/wake, service restart, network loss, and all three fresh client acceptance recorded |
| Reversible installation | Existing working setup must be recoverable | MEDIUM | Config backup and rollback validated across relevant global/project scopes without removing unrelated MCP entries |

Protocol details must follow the selected revision: current MCP tools support pagination, typed content and errors; tool annotations are untrusted unless the server is trusted. Forwarding must not convert application failures to fake successes or use annotation claims as the sole basis for safe replay. [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools).

### Differentiators (Competitive Advantage)

| Feature | Value proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| One diagnostic command | Separates local service, authorization, and upstream health | MEDIUM | Redacted output suitable for bug reports; current status distinct from last known success |
| Three-client compatibility receipt | Makes proof inspectable instead of relying on configuration | MEDIUM | Exact versions, timestamp, protocol revision and read-only result; no sensitive result payload |
| Small Swift distribution | Meets dependency preference | MEDIUM | Reviewed pinned dependency graph; no package downloads during startup |
| Optional native status UI | Convenient deliberate recovery | MEDIUM | Defer menu bar/notifications until CLI behavior and recovery are validated |

### Anti-Features (Commonly Requested, Often Problematic)

| Feature | Why requested | Why problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Permanent authorization guarantee | End repeated login frustration | Provider controls grant expiry and revocation | Maximize recoverability; report unavoidable reauthorization honestly |
| Auto-opening Safari on auth errors | Convenient immediate repair | Recreates core complaint | Explicit reconnect command only |
| Cookie scraping or password storage | Avoid browser login | Expands trust and bypasses intended OAuth flow | Supported authorization code flow |
| Replay every failed tool call | Appears resilient | Can duplicate figure changes after ambiguous failures | Classified safe retries; outcome-unknown errors |
| All-purpose MCP gateway | Broad reuse | Multiplies trust boundaries and protocol scope | BioRender-specific v1 |
| Public debug dumps | Easy support | Can expose research content and credentials | Sanitized metadata-only receipts |
| Shared MCP session for all clients | Fewer connections | Cross-client response/cancellation leakage | Shared credential owner, isolated client contexts |

## Feature Dependencies

```text
Provider + protocol audit → supported version contract → forwarding acceptance
Threat model → credential/IPC boundaries → shared broker → three-client migration
Refresh state machine → fault-injection evidence → actual expiry/later-session proof
Status/error contract → CLI recovery → optional native UI
```

### Dependency Notes

- Installation depends on passing fake-server tests and having a rollback path; configuration alone is not acceptance.
- Persistence validation depends on real elapsed expiration. Manipulating a local clock/expiry field is a separate test, not a substitute.
- No UI should conceal the distinction between temporary network loss and invalidated authorization.

## MVP Definition

### Launch With (v1)

- [ ] Documented BioRender conformance audit and supported protocol matrix.
- [ ] Swift broker, Keychain credential ownership, serialized refresh, secure local IPC.
- [ ] Functional stdio frontend usable by Codex, Claude Code, and OpenCode.
- [ ] Explicit login/status/reconnect, no background browser launches, bounded safe retries.
- [ ] Reversible installation and independently recorded acceptance in each harness.
- [ ] Evidence across expiration and later sessions, with upstream limitations stated.

### Add After Validation (v1.x)

- [ ] Menu-bar status/reconnect when CLI recovery is stable.
- [ ] Opt-in deduplicated desktop notifications once notification behavior is specified.
- [ ] Signed release automation after deployment and dependency review gates are met.

### Future Consideration (v2+)

- [ ] Multi-account support only with distinct credential/cursor/cache namespaces.
- [ ] Additional MCP providers only if a separate product decision expands scope.

## Feature Prioritization Matrix

| Feature | User value | Implementation cost | Priority |
|---------|------------|---------------------|----------|
| Protocol/provider audit and security boundaries | HIGH | MEDIUM | P1 |
| Centralized durable refresh | HIGH | HIGH | P1 |
| Three-client forwarding and safe errors | HIGH | HIGH | P1 |
| Browser-free recovery and status | HIGH | MEDIUM | P1 |
| Expiry and later-session acceptance | HIGH | HIGH | P1 |
| SwiftUI menu bar | MEDIUM | MEDIUM | P2 |
| Multi-provider gateway | LOW | HIGH | P3/out of v1 |

## Existing Approach Comparison

| Feature | Existing direct per-harness setup | Proposed BioRenderBridge |
|---------|----------------------------------|--------------------------|
| Credential owner | Each harness/helper separately, per project history | One broker; account scope explicit |
| Recovery presentation | Different per-client warnings/status | Common categorized status surfaced through each frontend |
| Browser control | Prior legacy helper opened browsers; later repair aimed to prevent this | Enforced single explicit login entry point with regression evidence |
| Persistence proof | Immediate successes followed by later failure, per PROJECT.md | Acceptance spans real expiry and later sessions |

This comparison uses recorded project context, not a fresh credential or installed-client audit. No account tokens were accessed during this research.

## Sources

- [Project context](../PROJECT.md) — user needs and historical failures, not implementation proof.
- [Current MCP release](https://blog.modelcontextprotocol.io/posts/2026-07-28/) — modern protocol architecture.
- [MCP version compatibility](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning).
- [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools).
- [MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization) — resource-bound authorization, confidentiality, refresh discretion.
- [RFC 9700 refresh protection](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14) — rotation/revocation constraints.
- [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252.html) — native authorization.

---
*Feature research for: BioRenderBridge*
*Researched: 2026-09-29*
