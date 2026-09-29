# Pitfalls Research

**Domain:** Native Swift broker for BioRender MCP across three AI harnesses
**Researched:** 2026-09-29
**Confidence:** LOW for causal/lifetime claims and provider-classified web research; measured public discovery observations are separately identified in the [audit](../../biorender-audit.md).

## Critical Pitfalls

### Pitfall 1: Claiming persistent authorization after an immediate success

**What goes wrong:** A successful connection is marketed as a permanent fix, then unattended startup fails after expiry.

**Why it happens:** Local startup, access-token expiry, refresh lifetime, and upstream grant validity are collapsed into one status.

**How to avoid:** Track separate acceptance evidence for real expiration, two expiration boundaries, overnight sleep, restarts and all three installed clients. Upstream rejection remains terminal until explicit reconnect.

**Warning signs:** Only mocked refresh tests; timestamp-free Connected screenshots; treating JWT expiration as the complete server policy.

**Phase to address:** Protocol baseline and final persistence acceptance.

### Pitfall 2: Turning a provider discovery defect into an insecure generic bypass

**What goes wrong:** A fallback accepts arbitrary resource metadata, follows attacker-controlled endpoints, or disables resource binding to accommodate one provider.

**Why it happens:** Public BioRender discovery currently points protected-resource discovery at an authorization-server document. Both expected protected-resource routes returned 404 during this audit.

**How to avoid:** Keep standard discovery as the default; require an explicit exact-origin compatibility policy and malformed/hostile fixture tests. Mark the deviation and retest it for removal when upstream changes.

**Warning signs:** Globally missing issuer/resource checks; silent fallback; describing endpoint hardcoding as full standards compliance.

**Phase to address:** Protocol contract and upstream compatibility audit.

### Pitfall 3: Multiple owners or non-durable rotation

**What goes wrong:** Concurrent refreshes or a crash after upstream rotation leave an unusable saved token; a daemon per harness defeats central ownership.

**Why it happens:** An in-process actor alone does not serialize separate processes. Persisting after returning success leaves a crash window.

**How to avoid:** One per-user credential owner, bounded waiters, atomic secret updates, controlled broker lifecycle and fault injection around each transition. Treat an ambiguous refresh timeout as uncertain state rather than blindly reusing a possibly invalidated token.

**Warning signs:** Three refresh schedulers, refresh tokens in harness configuration, stale-token overwrite after a second request.

**Phase to address:** Credential storage and refresh state machine.

### Pitfall 4: Confusing MCP sessions with OAuth/account sessions

**What goes wrong:** The bridge responds to every session error with login or retries expired transport identifiers using the same broken path.

**Why it happens:** Historical session_expired text has not been mapped to a documented BioRender error contract.

**How to avoid:** Separate transport-session renewal, HTTP authorization errors, tool-result errors and refresh-grant errors. Preserve sanitized error categories and explicitly label unknown service errors.

**Warning signs:** Every HTTP error classified as invalid_grant; browser authorization used as routine reconnection; tool errors swallowed.

**Phase to address:** MCP forwarding and recovery.

### Pitfall 5: A local broker broadens access or changes tool semantics

**What goes wrong:** Another local user or web origin can invoke private-account tools, or replay creates duplicate figures.

**Why it happens:** Loopback is treated as authentication, and all RPCs are treated as idempotent reads.

**How to avoid:** Define same-user IPC authorization, isolate client sessions/capabilities and request IDs, never expose upstream bearer tokens, validate any HTTP Origin, bound inputs, and do not automatically replay mutations after ambiguous completion. Prefer stdio shims to a same-user protected broker IPC unless an authenticated HTTP design is justified.

**Warning signs:** Unauthenticated TCP listener, broad bind address, arbitrary forwarded credentials, stdout diagnostics in a stdio MCP stream.

**Phase to address:** Local boundary and protocol forwarding.

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|---|---|---|---|
| Fixed BioRender endpoint compatibility | Works around broken discovery | Hides upstream changes | Only narrowly scoped, documented and tested |
| Secret JSON files | Easy prototype | Exposure in backups and publication | Never for production credentials |
| Swift SDK main branch | Fast feature access | Non-reproducible dependency behavior | Research inspection only; pin release before implementation |
| Hardcoded tool count | Simple smoke assertion | False alarms as service evolves | Never as sole readiness criterion |
| Keepalive refresh loop | May appear to prevent idle expiry | Unproven policy dependence and needless traffic | Only with explicit documented lifecycle rationale; not a permanence fix |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|---|---|---|
| BioRender OAuth | offline_access means indefinite authorization | Record policy uncertainty and surface terminal failure |
| Codex | Helper-based token discovery means native authStatus must say authenticated | Verify actual protocol/tool behavior and explain auth ownership |
| Claude/OpenCode | Shared broker success proves harness success | Separate installed-client acceptance tests |
| Keychain | Locked storage means grant is invalid | Report local locked state without revoking/replacing authorization |
| MCP | Advertise every SDK capability | Advertise only faithfully implemented forwarding paths |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|---|---|---|---|
| Refresh stampede | Parallel waits and repeated token requests | Single-flight refresh and bounded queues | Two simultaneous clients |
| Backoff exceeds client startup timeout | Harness marks healthy-but-retrying server dead | Explicit timeout budget and readable unavailable state | Network outage during startup |
| Unlimited SSE buffering | Rising memory or blocked clients | Bounded buffers, cancellation, per-client isolation | One long-lived or stalled response |

## Security Mistakes

| Mistake | Risk | Prevention |
|---|---|---|
| Sharing upstream tokens with harnesses | Multiple secret stores and authority confusion | Return results only; broker owns authorization |
| Trusting public-client embedded secret | Extractable binary-wide secret | Confirm provider native-client model; per-install protected registration if required |
| Logging OAuth callback/query strings | Grant or state leakage | Allowlisted diagnostics; redact URLs and request bodies |
| Automatic dependency execution at startup | Supply-chain exposure | Compile pinned reviewed dependencies into release artifacts |
| Blind trust in upstream tool descriptions | Prompt injection carried into clients | Treat all remote descriptions/results as untrusted; preserve consent boundaries |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---|---|---|
| Browser opens during connection | Repeated interruptions | Only explicit login/reconnect can launch it |
| Failed tools become empty successful list | Silent loss of capability | Explicit actionable unavailable/auth state |
| Alert from every harness | Notification storm | Broker deduplicates transitions; callers still receive accurate errors |
| Permanent-auth wording | False expectation | Explain tested persistence interval and provider-controlled limits |

## "Looks Done But Isn't" Checklist

- [ ] **Authentication:** Real expired access-token refresh, durable rotation and terminal invalid_grant recovery verified.
- [ ] **Persistence:** Later-session evidence spans two real expiry boundaries and an overnight sleep, with recorded limits.
- [ ] **Clients:** Independent initialize, discovery and safe real tool call through Codex, Claude and OpenCode.
- [ ] **Browser behavior:** Ordinary startup, refresh failure and network loss produce zero automatic browser launches.
- [ ] **Protocol:** Negotiation, capabilities, notifications, cancellation, structured errors, session loss and pagination tested where advertised.
- [ ] **Privacy:** Public fixtures contain no personal files, tokens, account IDs or private machine paths.
- [ ] **Recovery:** Locked Keychain, daemon restart, interrupted rotation and revoked grant each have distinct tested outcomes.

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---|---|---|
| Terminal grant rejection | MEDIUM | Stop refresh loop; preserve redacted evidence; request deliberate reconnect once |
| Transport-session loss | LOW | Renegotiate session per transport contract; do not replay uncertain mutation |
| Uncertain rotation result | HIGH | Prevent stale writers; report uncertainty; reconnect if durable usable token cannot be established |
| Discovery contract changes | MEDIUM | Fail visibly; update scoped compatibility fixtures; retest standards route |
| Keychain temporarily locked | LOW | Report unlock-required; retry storage access after explicit unlock without resetting grant |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---|---|---|
| Malformed discovery and unsupported promises | Protocol baseline | Public response fixtures plus documented deviation |
| Races, ambiguous token updates, local leakage | Credential lifecycle | Concurrent clients and crash/timeout injection |
| Broken forwarding/session semantics | MCP integration | Protocol contract fixtures and each installed harness |
| Browser storms and invisible failures | Recovery UX | Count browser launches and deduplicated state transitions |
| Short-lived proof mistaken for persistence | Acceptance/release | Dated soak results with real expiry and sleep/wake |

## Sources

- [BioRender audit and fresh public measurements](../../biorender-audit.md) — historical and current evidence distinguished.
- [BioRender connector guide](https://help.biorender.com/hc/en-gb/articles/30870978672157-How-to-use-the-BioRender-MCP-connector) — checked 2026-09-29; no refresh-lifetime promise located.
- [MCP authorization baseline](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) — discovery, resource binding and OAuth roles.
- [MCP transports](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports) — lifecycle and transport security baseline.
- [MCP security practices](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices) — proxy and token-boundary concerns.
- [RFC 6749 section 5.2](https://www.rfc-editor.org/rfc/rfc6749.html#section-5.2) — invalid_grant has multiple possible causes.
- [RFC 9700 section 4.14](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14) — rotation, replay and provider-controlled expiration.
- [RFC 9728](https://www.rfc-editor.org/rfc/rfc9728.html) — protected-resource metadata contract.

---
*Pitfalls research for: BioRenderBridge*
*Researched: 2026-09-29*
