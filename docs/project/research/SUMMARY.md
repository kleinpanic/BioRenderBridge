# Project Research Summary

**Project:** BioRenderBridge
**Domain:** Native macOS Swift broker for shared BioRender MCP authorization
**Researched:** 2026-09-29
**Confidence:** MEDIUM overall; LOW for upstream grant policy and historical failure causality

## Executive Summary

BioRenderBridge should be a small native Swift service that owns one upstream authorization and serves Codex, Claude Code, and OpenCode through separate stdio frontends. Use one per-user broker, Keychain-backed credentials, a single-flight refresh coordinator, and isolated protocol contexts. Ordinary startup and recovery must remain browser-free; explicit login/reconnect is the only browser-launch path. This initialization establishes a proposed design and delivery gates, not an implemented or validated bridge.

Adopt the official Swift MCP SDK only after testing exact release 0.12.1 and its resolved dependency graph. Its manifest requires Swift 6.1 and macOS 13.0, and its documented protocol support is 2025-11-25, while the research identifies 2026-07-28 as the current MCP revision. Record supported downstream and upstream revisions independently rather than claiming universal compatibility. Keep durable authorization policy outside the SDK's default authorizer: the inspected implementation has storage-clearing, refresh, registration-persistence, and multi-transport limitations that conflict with the intended broker unless explicitly addressed.

The [BioRender audit](../../biorender-audit.md) measured malformed protected-resource discovery on September 29: the authentication challenge points to authorization-server metadata missing protected-resource fields, while both expected protected-resource routes return 404. A narrowly scoped compatibility rule may be necessary; generic validation must remain strict. Historical invalid_grant and session_expired failures have no established root cause. Central ownership can prevent future local refresh races, but cannot override provider revocation or expiration. Release acceptance therefore requires independent real calls from all three installed clients and a dated soak spanning at least two actual access-token expiration boundaries and one overnight sleep; no permanent-authorization guarantee is justified.

## Key Findings

### Recommended Stack

Follow [STACK.md](STACK.md): use native platform facilities and keep the dependency graph small. SDK 0.12.1 is a researched candidate, not an accepted build dependency. The upstream manifest contains a DocC dependency on main; pinning the SDK tag alone does not make the full graph reproducible. Resolve, review revisions/licenses, commit Package.resolved, and verify locked builds before adoption. No dependency installation or build has occurred during initialization.

**Core technologies:**

- **Swift 6.1+ and SwiftPM:** compiled CLI/broker and actor-based coordination; pin the exact supported toolchain in Phase 1.
- **MCP Swift SDK 0.12.1 candidate:** protocol models and stdio/HTTP transports behind a tested integration boundary. Use its tagged source rather than examples from main.
- **Foundation/URLSession:** platform HTTPS networking with bounded cancellation and timeout behavior.
- **Security.framework:** broker-owned versioned Keychain record covering issuer/resource binding, dynamic registration credentials, token metadata, and rotation generation.
- **Per-user launchd agent:** one credential owner across harness lifetimes. A future app bundle may use SMAppService; a bare CLI is not automatically a bundled service.
- **Authenticated local IPC:** prefer signed XPC if deployment tests support it; a private Unix socket is a documented fallback with weaker same-user process isolation.

**SDK authorization hazards:** the tagged authorizer prohibits sharing across transports, clears storage on its 401 path before recovery completes, requires loaded metadata for proactive refresh and suppresses refresh errors, requires testing preservation of an omitted replacement refresh token, and leaves dynamic registration persistence to the application. Evaluate a custom HTTPClientAuthorizer or narrowly reviewed wrapper. The broker must retain valid credentials on transient failure and persist rotation before releasing waiters.

### Expected Features

The priorities in [FEATURES.md](FEATURES.md) center on reliable access and honest failure reporting.

**Must have (table stakes):**

- A tested protocol/version/capability matrix; faithful paginated discovery and tool results, including typed content and errors.
- One authorization owner, serialized refresh, durable Keychain rotation, and isolated client IDs, cursors, cancellations, and lifecycles.
- Explicit secure login/status/reconnect with distinct offline, locked-store, protocol, permission, and reconnect-required states.
- Zero browser launches during startup, tool calls, retries, expiration, invalid_grant, or sleep/wake recovery.
- Bounded safe recovery; uncertain figure mutations return outcome unknown and are not replayed automatically.
- Reversible installation, independent acceptance in all three harnesses, and actual-expiry/later-session persistence evidence.

**Should have (differentiators):**

- One redacted diagnostic command separating local service, authorization, and upstream health.
- Timestamped compatibility receipts with exact client/protocol versions and safe tool-call outcomes.
- Small prebuilt Swift distribution with reviewed dependencies and no startup package downloads.

**Defer:** menu-bar UI, notifications, and release automation until core acceptance is stable (v1.x); multi-account and additional-provider support to a separately scoped v2+. Exclude cookie scraping, password storage, generic proxying, blanket retries, and permanent-authentication promises.

### Architecture Approach

Use the boundaries in [ARCHITECTURE.md](ARCHITECTURE.md). Share authorization, not MCP sessions. The broker is an upstream OAuth client and returns results to downstream callers, never tokens. A Swift actor alone is insufficient: retain a shared in-flight refresh task across suspension points and ensure only one process owns each grant.

**Major components:**

1. **Stdio frontend:** one process per harness connection; clean framing, stdout reserved for MCP, explicit shutdown.
2. **Local broker and IPC:** peer authorization, bounded input and queue sizes, lifecycle supervision, independent client contexts.
3. **Authorization coordinator and vault:** deliberate PKCE/state-protected login, generation-aware single-flight refresh, durable Keychain updates, typed recovery states.
4. **Upstream adapter and dispatcher:** revision-specific lifecycle, exact-origin compatibility rule if required, request/cursor isolation, cancellation, errors, and safe retry classification.
5. **Status and installation surfaces:** redacted diagnostics, reconnect controls, backups/rollback, and acceptance receipts.

**Protocol contract to resolve before implementation commitments:**

| Boundary | Research finding | Required gate |
|----------|------------------|---------------|
| MCP 2026-07-28 | Current revision uses per-request metadata/discovery | Verify actual SDK, upstream, and harness support; do not assume legacy initialize applies |
| MCP 2025-11-25 | Legacy initialize/initialized lifecycle; candidate SDK advertises this revision | Exercise supported handshake and capability behavior |
| Downstream versus upstream | Revisions may differ | Record both; implement tested translation or reject unsupported combinations |
| Advertised capabilities | SDK/upstream support alone does not prove forwarding | Advertise only implemented and isolated paths; no implied resources/prompts/sampling support |

### Critical Pitfalls

1. **Immediate success mistaken for persistence:** separate fixture tests, current connectivity, actual expiration, and later-session receipts. A valid JWT expiration timestamp does not prove continued service authorization.
2. **Provider defect becomes a generic security bypass:** standards-first discovery; any BioRender exception must be exact-origin, visible, fixture-tested, and removable. Do not relax issuer/resource/endpoint validation globally.
3. **Refresh races or non-durable rotation:** one process owner, generation checks, single-flight refresh, omitted-token preservation, and persistence before readiness. A lost refresh response can leave an unrecoverable rotation window; blind retries are unsafe.
4. **Session and error classes collapse into login:** distinguish MCP transport loss, tool errors, network failure, locked Keychain, and terminal grant rejection. The meaning of historical session_expired remains unknown.
5. **Local access expands or forwarding changes semantics:** authenticate IPC peers, isolate clients, never export upstream tokens, preserve failures, and prohibit replay of potentially completed mutations. Tool annotations alone do not establish retry safety.

See [PITFALLS.md](PITFALLS.md) for the detailed failure and acceptance mapping.

## Implications for Roadmap

The following five delivery phases are recommendations for subsequent work. Initialization must stop before implementation, service installation, credential access, or harness configuration changes.

### Phase 1: Protocol, SDK, and Trust Contracts

**Rationale:** version mismatch, malformed provider discovery, and deployment constraints determine whether the proposed stack can meet the requirements.

**Delivers:** exact SDK/toolchain/dependency assessment; current-versus-legacy client/upstream matrix; sanitized discovery fixtures; decision on any exact-origin compatibility rule; IPC/signing threat model and deployment spike; explicit supported capabilities and acceptance criteria.

**Addresses:** honest compatibility, complete-forwarding scope, minimal reviewed dependencies, security boundaries.

**Avoids:** unsupported current-spec claims, generic discovery bypasses, premature SDK adoption, unauthenticated local access.

### Phase 2: Durable Authorization and Deliberate Recovery

**Rationale:** forwarding depends on a credential owner whose behavior remains correct across refresh, restart, and storage failure.

**Delivers:** broker-owned Keychain record and registration persistence; secure explicit login; single-flight generation-aware refresh; typed status/reconnect states; deterministic failure and concurrency tests. Test SDK auth hazards, omitted refresh token, failed writes, invalid_grant, ambiguous refresh timeout, and stale completion after login/logout.

**Uses:** Swift actors, Security.framework, URLSession, and the audited SDK authorizer integration selected in Phase 1.

**Addresses:** shared authorization, durable storage, serialized refresh, browser-free operation, explicit recovery.

**Avoids:** destructive credential resets, non-durable rotation, refresh stampedes, and unsolicited browser launches.

### Phase 3: Secure Broker and Faithful MCP Forwarding

**Rationale:** exercise real service/frontend boundaries and protocol semantics before changing any installed harness.

**Delivers:** protected IPC and broker lifecycle; separate stdio frontend contexts; upstream adapter; pagination/content/error preservation; tested supported-version behavior, cancellation, session recovery, bounded queues/timeouts, and operation-specific retry rules.

**Addresses:** complete tool forwarding, independent lifecycles, secure local access, safe bounded recovery, visible errors.

**Avoids:** shared-session leakage, stdout corruption, empty-success discovery failures, and duplicate figure mutations. Fake-server integration tests must pass before migration.

### Phase 4: Reversible Installation and Three-Client Acceptance

**Rationale:** library tests and direct protocol probes cannot prove installed Codex, Claude Code, and OpenCode behavior.

**Delivers:** deliberate per-user install/uninstall; configuration backups and rollback preserving unrelated entries; installed signing/Keychain/IPC checks; fresh per-harness discovery and a safe real read-only tool call; redacted diagnostic and compatibility receipts.

**Addresses:** reversible installation, three-client integration, clear status, small prebuilt distribution.

**Avoids:** accepting configuration as connectivity, treating one harness as proof for another, or using a fixed tool count as readiness. Search is preferred to generation for non-mutating acceptance; generation may consume credits.

### Phase 5: Persistence and Failure Acceptance

**Rationale:** the original failure emerged after immediate success; elapsed-time behavior is part of the deliverable.

**Delivers:** dated evidence spanning at least two actual access-token expiration boundaries and an overnight sleep, plus concurrent use, restart, sleep/wake, offline recovery, locked-store behavior, and terminal-rejection recovery. Count browser launches across ordinary operation and report the observed persistence interval and remaining limits.

**Addresses:** trustworthy persistence, durable rotation under real expiry, bounded recovery, and browser-free normal use.

**Avoids:** equating simulated expiry with elapsed expiry, attributing invalid_grant without evidence, or claiming perpetual authorization. If provider policy prevents the target behavior, record the limitation and unresolved acceptance rather than marking it passed.

### Phase Ordering Rationale

- Resolve protocol and security feasibility before binding implementation to the SDK or IPC choice.
- Establish durable authorization before composing forwarding around it; test cross-process coordination when the broker/frontend boundary is added.
- Preserve existing working configurations until fixture-based integration and rollback are ready.
- Keep final persistence acceptance separate so immediate harness success cannot silently close the later-session requirement.

### Research Flags

Phases likely needing deeper research during planning:

- **Phase 1:** `$gsd-plan-phase --research-phase 1` — SDK current/legacy support, dependency graph, provider discovery, XPC signing/API availability, and deployment feasibility.
- **Phase 2:** `$gsd-plan-phase --research-phase 2` — SDK authorizer integration, native registration model, resource binding, rotation ambiguity, and provider grant policy. Do not contact the provider without explicit authorization.
- **Phase 3:** `$gsd-plan-phase --research-phase 3` — exact BioRender retry-safe operations and error semantics; any required protocol translation.

Phases with established patterns that can usually skip broad research:

- **Phase 4:** reversible configuration migration and per-client acceptance; verify installed client versions and current configuration syntax during planning.
- **Phase 5:** execute the defined fault/soak matrix and classify evidence. Reopen targeted research if new service behavior exposes an unresolved policy or protocol assumption.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | MEDIUM | Tagged manifest/source and Apple documentation inspected; no resolved graph, compile, or installed-service validation |
| Features | MEDIUM | Requirements and normative sources align; proposed capabilities remain unbuilt |
| Architecture | MEDIUM | Established isolation/credential patterns; deployment, protocol translation, and provider behavior remain gates |
| Pitfalls | LOW for provider causality/policy | Broad failure modes are documented; historical causes and lifetimes unconfirmed; exact public discovery observations are separately measured |

**Overall confidence:** MEDIUM for roadmap direction, LOW for any claim about unattended grant lifetime. Primary-source authority does not upgrade untested integration into runtime proof; retain the source files' conservative research classifications.

### Gaps to Address

- **Grant policy:** refresh absolute/idle lifetime, rotation/revocation behavior, concurrent independent grants, and the meaning of session_expired remain unknown. Use later controlled evidence or an explicitly authorized provider inquiry; do not infer policy from offline_access.
- **Native client registration:** validate advertised client_secret_post, per-install registration persistence, resource indicators, and supported native-client integration. Embedded application secrets are not confidential identity proof.
- **Discovery evolution:** the public measurement is time-specific; retest standards discovery before enabling or retaining the compatibility rule.
- **Protocol and SDK compatibility:** verify exact harness/upstream revisions and candidate SDK behavior. Dual-era translation is a decision to validate, not an assumed feature.
- **Installed security:** prove signing identity, peer checks, Keychain access, macOS minimum, restart, and wake behavior using installed executables.
- **Failure causality:** historical invalid_grant does not establish races between separate harness authorizations. Centralization prevents a class of future failures without proving it fixes that past cause.
- **Acceptance duration:** historical access tokens lasted 28,800 seconds, but future grants must be measured. Plan elapsed-time acceptance around actual issued expiry and preserve unresolved outcomes.

## Sources

### Local synthesis inputs and measured evidence

- [STACK.md](STACK.md), [FEATURES.md](FEATURES.md), [ARCHITECTURE.md](ARCHITECTURE.md), [PITFALLS.md](PITFALLS.md) — research findings and their original confidence limits.
- [BioRender interoperability audit](../../biorender-audit.md) — unauthenticated discovery measurements at 2026-09-29T20:20:52Z, sanitized historical evidence, and unresolved provider questions. No authenticated bridge validation.
- [PROJECT.md](../PROJECT.md) — scope and user requirements; historical context is not current acceptance.

### Primary documentation and tagged source

- [Swift SDK 0.12.1 manifest](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Package.swift), [release](https://github.com/modelcontextprotocol/swift-sdk/releases/tag/0.12.1), [README](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/README.md), [OAuthAuthorizer](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Sources/MCP/Base/Authorization/OAuthAuthorizer.swift), [OAuthModels](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Sources/MCP/Base/Authorization/OAuthModels.swift) — candidate stack and auth hazards.
- [MCP 2026-07-28 versioning](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning), [tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools), [authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization), [legacy lifecycle](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle), [legacy authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) — version-specific contracts.
- [RFC 9728](https://www.rfc-editor.org/rfc/rfc9728.html), [RFC 6749 section 5.2](https://www.rfc-editor.org/rfc/rfc6749.html#section-5.2), [RFC 9700 section 4.14](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14), [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252.html) — discovery, grant errors, rotation constraints, and native OAuth.
- [Apple Keychain](https://developer.apple.com/documentation/security/keychain-services), [launchd](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html), [SMAppService](https://developer.apple.com/documentation/servicemanagement/smappservice), [XPC peer requirements](https://developer.apple.com/documentation/xpc/xpcpeerrequirement) — platform integration contracts, pending deployment validation.
- [BioRender connector guide](https://help.biorender.com/hc/en-gb/articles/30870978672157-How-to-use-the-BioRender-MCP-connector) — advertised connector behavior; no documented refresh-lifetime guarantee located by the audit.

### Historical and inferential evidence (limited confidence)

- Sanitized historical observations reproduced in the local audit establish earlier symptoms and immediate recovery only. No private diagnostics or credentials were copied into this summary.
- Proposed phase structure, component boundaries, and SDK integration choice are synthesis recommendations awaiting the listed implementation gates.

---
*Research completed: 2026-09-29*
*Ready for roadmap: yes; initialization only, implementation and acceptance pending.*
