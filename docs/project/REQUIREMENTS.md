# Requirements: BioRenderBridge

**Defined:** 2026-09-29
**Core Value:** Reliable BioRender access across Codex, Claude Code and OpenCode, with browser-free ordinary operation and deliberate, visible recovery.

## v1 Requirements

All requirements below are unimplemented. This session initializes the project only.

### Protocol and provider contract

- [ ] **SPEC-01**: Maintainers can inspect a dated compatibility matrix identifying exact downstream clients, upstream revision, candidate SDK release and supported MCP revisions; unsupported combinations fail explicitly.
- [ ] **SPEC-02**: Maintainers can reproduce sanitized BioRender discovery fixtures and review any exact-origin compatibility exception without weakening issuer, resource or endpoint validation globally.
- [ ] **SPEC-03**: Maintainers can review a threat model covering OAuth callbacks, upstream content, local peers, credential storage and ambiguous retries before selecting IPC and deployment mechanisms.

### Swift and dependency integrity

- [ ] **BUILD-01**: Contributors can reproduce the build using an exact supported Swift toolchain and reviewed dependency revisions/lockfile, including build plugins and transitive packages; mutable declarations require explicit resolution and review.
- [ ] **BUILD-02**: Users run compiled Swift artifacts with no Node/TypeScript runtime or package downloads during ordinary startup.

### Deliberate authorization and secure storage

- [ ] **AUTH-01**: Users can deliberately authorize or reconnect with authorization code plus S256 PKCE, unpredictable state, validated issuer/resource binding, and a one-use restricted callback; unsolicited callbacks are rejected.
- [ ] **AUTH-02**: Users retain one broker-owned credential record in Keychain across restart, including registration credentials, issuer/resource, token metadata and rotation generation; no plaintext fallback or token delivery to harnesses exists.
- [ ] **AUTH-03**: Concurrent users of the three harnesses share one in-flight refresh per grant, including across suspension points; stale completions cannot overwrite newer login or logout state.
- [ ] **AUTH-04**: Users retain the previous refresh token when a successful refresh omits its replacement; successful rotation becomes durable before waiting callers resume.
- [ ] **AUTH-05**: Users keep valid saved credentials during transient network/storage errors; locked Keychain and failed writes are distinct from rejected authorization.
- [ ] **AUTH-06**: Users receive reconnect-required or refresh-outcome-unknown on terminal rejection or ambiguous rotation; retries stop without opening a browser or pretending access is healthy.
- [ ] **AUTH-07**: Users can explicitly disconnect and clear broker authorization; in-flight work cannot recreate the cleared credentials.

### Faithful MCP forwarding

- [ ] **MCP-01**: Each client receives correct revision-specific framing, lifecycle/discovery and negotiated capabilities; only implemented forwarding paths are advertised, and stdout contains only protocol traffic.
- [ ] **MCP-02**: Users discover all upstream tool pages with original names, input/output schemas and supported metadata; discovery failure cannot become a successful empty list.
- [ ] **MCP-03**: Users receive preserved supported content types, structured results, tool errors and protocol errors from real forwarded calls.
- [ ] **MCP-04**: Concurrent clients have isolated request IDs, cursors, cancellation, progress and session lifetimes; disconnecting one does not cancel another client’s work.
- [ ] **MCP-05**: Users recover from transport-session loss using the selected protocol without conflating it with OAuth rejection or silently replaying potentially completed operations.

### Local security and lifecycle

- [ ] **LOCAL-01**: Only authorized local peers can invoke the per-user broker; cross-user and unauthorized-peer attempts fail, with installed signing/identity checks validated for the chosen IPC.
- [ ] **LOCAL-02**: Users get one broker credential owner across simultaneous launches and restarts, with bounded requests, queues, buffers and shutdown behavior.
- [ ] **LOCAL-03**: Users recover after network restoration, sleep/wake and broker restart with bounded operation-specific backoff; ambiguous mutating calls report unknown outcome and are never automatically replayed.

### Visible recovery

- [ ] **UX-01**: Ordinary startup, discovery, calls, expiry, background refresh, rejected grants and wake produce zero browser launches; only a deliberate login/reconnect command may launch authorization.
- [ ] **UX-02**: Users can inspect distinct ready, offline, locked-store, permission, protocol, reconnect-required and uncertain states with actionable recovery; each harness receives a truthful failure rather than silent success.
- [ ] **UX-03**: Users can export redacted diagnostics separating current health from last success without tokens, callback parameters, account identifiers or private tool content.

### Installation and client acceptance

- [ ] **INSTALL-01**: Users can explicitly install, upgrade, roll back and uninstall a per-user deployment, with reviewed permissions/signing and documented credential-retention behavior.
- [ ] **INSTALL-02**: Users can migrate each harness with configuration backups and rollback that preserve unrelated MCP entries and account settings.
- [ ] **CLIENT-01**: An installed Codex session independently completes supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.
- [ ] **CLIENT-02**: An installed Claude Code session independently completes supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.
- [ ] **CLIENT-03**: An installed OpenCode session independently completes supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.

### Persistence acceptance and release

- [ ] **PROOF-01**: Users have dated real-service evidence spanning at least two actual issued access-token expiry boundaries and one overnight sleep, including fresh sessions in all three harnesses; simulated expiry is labeled separately.
- [ ] **PROOF-02**: Users have separately labeled controlled fault evidence for concurrent refresh, crash/write failures, omitted replacement tokens, locked Keychain, offline recovery and terminal rejection, including a zero-background-browser assertion.
- [ ] **PROOF-03**: Users can inspect a release readiness record with tested versions, elapsed persistence interval, security/dependency findings and unresolved provider limits; failed gates remain open and no permanent authorization promise is made.

## v2 Requirements

- **UI-01**: Optional native menu-bar status and deliberate reconnect UI after CLI acceptance.
- **UI-02**: Opt-in deduplicated notifications after notification behavior is specified.
- **DIST-01**: Automated signed release publishing after the deployment contract is proven.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Guaranteed permanent authorization | BioRender controls expiration and revocation |
| Automatic browser login on startup/error | Recreates the originating problem |
| Cookie scraping or password storage | OAuth consent remains authoritative |
| Generic multi-provider or multi-account gateway | Separate future scope decision |
| Reimplementing BioRender figure creation | Forward supported upstream tools |
| Node/TypeScript production service | Explicit Swift requirement |
| Swift directory cleanup or active client changes in initialization | Separate user-directed work |

## Traceability

Each v1 requirement maps to exactly one owning phase. Pending means unimplemented; initialization does not satisfy implementation or acceptance.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SPEC-01 | Phase 1 | Pending |
| SPEC-02 | Phase 1 | Pending |
| SPEC-03 | Phase 1 | Pending |
| BUILD-01 | Phase 1 | Pending |
| BUILD-02 | Phase 2 | Pending |
| AUTH-01 | Phase 2 | Pending |
| AUTH-02 | Phase 2 | Pending |
| AUTH-03 | Phase 3 | Pending |
| AUTH-04 | Phase 3 | Pending |
| AUTH-05 | Phase 3 | Pending |
| AUTH-06 | Phase 3 | Pending |
| AUTH-07 | Phase 2 | Pending |
| MCP-01 | Phase 4 | Pending |
| MCP-02 | Phase 4 | Pending |
| MCP-03 | Phase 4 | Pending |
| MCP-04 | Phase 4 | Pending |
| MCP-05 | Phase 4 | Pending |
| LOCAL-01 | Phase 4 | Pending |
| LOCAL-02 | Phase 4 | Pending |
| LOCAL-03 | Phase 4 | Pending |
| UX-01 | Phase 3 | Pending |
| UX-02 | Phase 3 | Pending |
| UX-03 | Phase 5 | Pending |
| INSTALL-01 | Phase 5 | Pending |
| INSTALL-02 | Phase 5 | Pending |
| CLIENT-01 | Phase 6 | Pending |
| CLIENT-02 | Phase 6 | Pending |
| CLIENT-03 | Phase 6 | Pending |
| PROOF-01 | Phase 7 | Pending |
| PROOF-02 | Phase 7 | Pending |
| PROOF-03 | Phase 7 | Pending |

**Coverage:**
- v1 requirements: 31 total
- Mapped to phases: 31
- Unmapped: 0
- Duplicate ownership: 0
- Implemented: 0

## Acceptance Rules

Fixture tests, current real connectivity, installed-client acceptance and elapsed persistence are separate evidence classes. Failures and upstream policy limits stay visible. Review the [audit](../biorender-audit.md) and [research summary](research/SUMMARY.md) before planning. Protocol behavior follows the exact supported revision, not an assumed universal initialize handshake.

---
*Last updated: 2026-09-29 during project initialization.*
