# Roadmap: BioRenderBridge

## Overview

Deliver a native Swift bridge that owns one BioRender authorization and serves three independently verified MCP clients. First resolve protocol, SDK and trust contracts; then deliver deliberate authorization, durable recovery, isolated forwarding, reversible installation, installed-client acceptance and real-expiry evidence. All 31 v1 requirements are unimplemented. Initialization is complete; execution is paused awaiting user direction. No phase plans or implementation are authorized by this initialization handoff.

Granularity: fine. Phase IDs: sequential (the configured project code is only a directory prefix). Use MVP vertical slices when planning runnable capabilities; the initial compatibility gate and final elapsed-evidence gate have distinct acceptance boundaries.

## Phases

**Phase Numbering:** Integer phases are planned milestone work. Decimal phases are urgent insertions and execute between their surrounding integers.

- [ ] **Phase 1: Compatibility and Trust Contract** - Establish reproducible build and explicit supported protocol/security boundaries.
- [ ] **Phase 2: Deliberate Authorization** - Authorize, retain and clear one secure credential record through compiled Swift commands.
- [ ] **Phase 3: Durable Refresh and Visible Recovery** - Coordinate refresh safely and explain failures without surprise browser launches.
- [ ] **Phase 4: Isolated MCP Forwarding** - Forward faithfully through a secure, bounded broker with independent client contexts.
- [ ] **Phase 5: Reversible Installation and Diagnostics** - Deploy and migrate explicitly with rollback and safe diagnostic export.
- [ ] **Phase 6: Three-Client Acceptance** - Prove real tool access independently in installed Codex, Claude Code and OpenCode.
- [ ] **Phase 7: Persistence and Release Evidence** - Prove actual-expiry behavior and controlled failures with honest release limits.

## Phase Details

### Phase 1: Compatibility and Trust Contract
**Goal**: Maintainers can reproduce the candidate build and determine exactly which protocol and trust contracts the bridge can support before committing to SDK or IPC choices.
**Depends on**: Nothing (first phase)
**Requirements**: SPEC-01, SPEC-02, SPEC-03, BUILD-01
**Success Criteria** (what must be TRUE):
  1. Maintainers can inspect a dated matrix of exact client versions, upstream and downstream MCP revisions, candidate SDK release and supported combinations; an unsupported combination produces an explicit rejection in the compatibility checks.
  2. Maintainers can reproduce sanitized provider discovery fixtures and inspect any exact-origin exception while issuer, resource and endpoint validation remains strict elsewhere.
  3. Maintainers can inspect a threat model covering callbacks, upstream content, local peers, storage and ambiguous retries, with evidence supporting the selected IPC/signing and deployment mechanisms.
  4. Contributors can reproduce a locked build with an exact supported Swift toolchain and reviewed direct/transitive dependencies and build plugins; mutable declarations have explicit reviewed resolutions.
**Plans**: TBD

Planning gate: SDK 0.12.1 remains a candidate. Resolve exact released behavior, protocol-era differences and the full dependency graph before adoption. Research recommendations are inputs, not acceptance evidence.

### Phase 2: Deliberate Authorization
**Mode**: mvp
**Goal**: Users can deliberately authorize, retain and disconnect one broker-owned BioRender grant using compiled Swift commands.
**Depends on**: Phase 1
**Requirements**: BUILD-02, AUTH-01, AUTH-02, AUTH-07
**Success Criteria** (what must be TRUE):
  1. Users run compiled Swift artifacts without a Node/TypeScript runtime or package downloads during ordinary startup.
  2. Users can explicitly log in or reconnect through authorization code with S256 PKCE, unpredictable state and validated issuer/resource binding; unsolicited or reused callbacks are rejected by the restricted one-use callback.
  3. After restart, users retain one broker-owned Keychain record including registration credentials, issuer/resource binding, token metadata and rotation generation; no plaintext fallback or token transfer to harnesses occurs.
  4. Users can explicitly disconnect and clear authorization; previously in-flight work cannot restore the cleared record.
**Plans**: TBD

Planning mode: MVP slice from explicit command through real authorization and durable storage. Keep provider registration and grant-policy unknowns visible.

### Phase 3: Durable Refresh and Visible Recovery
**Mode**: mvp
**Goal**: Users share durable refresh safely and receive actionable recovery states while ordinary operation remains browser-free.
**Depends on**: Phase 2
**Requirements**: AUTH-03, AUTH-04, AUTH-05, AUTH-06, UX-01, UX-02
**Success Criteria** (what must be TRUE):
  1. Concurrent callers share one in-flight refresh per grant across suspension points; a stale completion cannot overwrite a later login or logout.
  2. Successful refresh without a replacement preserves the previous refresh token, and rotated credentials survive restart before waiting callers are told the bridge is ready.
  3. During transient network or storage failures, users retain valid saved credentials; locked Keychain and failed writes are distinguishable from rejected authorization.
  4. Users can inspect actionable ready, offline, locked-store, permission, protocol, reconnect-required and uncertain states; terminal rejection or ambiguous refresh stops retries, and callers receive truthful failures.
  5. Startup, discovery, calls, expiration, background refresh, rejected grants and wake cause zero browser launches in controlled exercises; only explicit login/reconnect can launch authorization. Recheck this invariant through later integration and acceptance.
**Plans**: TBD

Planning mode: MVP slice from a credential-dependent operation through refresh or visible recovery. Controlled fixtures establish behavior here; elapsed real-service persistence remains Phase 7.

### Phase 4: Isolated MCP Forwarding
**Mode**: mvp
**Goal**: Authorized local clients can discover and call upstream tools faithfully through one secure broker while remaining independent during failures and recovery.
**Depends on**: Phase 3
**Requirements**: MCP-01, MCP-02, MCP-03, MCP-04, MCP-05, LOCAL-01, LOCAL-02, LOCAL-03
**Success Criteria** (what must be TRUE):
  1. Each client receives revision-specific framing, lifecycle/discovery and only implemented negotiated capabilities; stdout contains only protocol traffic. All tool pages preserve original names, schemas and supported metadata, and discovery failures remain failures.
  2. Real forwarded calls preserve supported content types, structured results, tool errors and protocol errors. Concurrent client IDs, cursors, cancellation, progress and session lifetimes stay isolated; one disconnect does not cancel another client's work.
  3. Cross-user and unauthorized-peer calls fail using the selected IPC, with signing/identity checks verified against installed executables in a controlled deployment before harness migration.
  4. Simultaneous launches and restarts leave exactly one broker credential owner, and users observe bounded request, queue, buffer and shutdown behavior under overload or disconnect.
  5. Users recover from session loss, network restoration, sleep/wake and broker restart with bounded operation-specific backoff; transport loss is distinct from OAuth rejection, and potentially completed mutations report unknown outcome without automatic replay.
**Plans**: TBD

Planning mode: MVP slice from an independent stdio client through authenticated local IPC to an upstream tool result. Fake-server integration must pass before changing installed harness registrations. Revalidate earlier authorization invariants at the process boundary.

### Phase 5: Reversible Installation and Diagnostics
**Mode**: mvp
**Goal**: Users can explicitly deploy, migrate and remove the bridge with recoverable configuration changes and safe diagnostic evidence.
**Depends on**: Phase 4
**Requirements**: INSTALL-01, INSTALL-02, UX-03
**Success Criteria** (what must be TRUE):
  1. Users can explicitly install, upgrade, roll back and uninstall a per-user deployment with reviewed permissions/signing and clear credential-retention behavior.
  2. Users can migrate each harness from backed-up configuration and restore it while preserving unrelated MCP entries and account settings.
  3. Users can export redacted diagnostics that distinguish current health from last success and contain no tokens, callback parameters, account identifiers or private tool content.
**Plans**: TBD

Planning mode: MVP slice from explicit deployment/migration command through observable status and rollback. Recheck Phase 4 peer authorization and Phase 3 browser behavior on the final installed artifacts.

### Phase 6: Three-Client Acceptance
**Goal**: Users can access BioRender through each installed harness with independent evidence of actual bridge use.
**Depends on**: Phase 5
**Requirements**: CLIENT-01, CLIENT-02, CLIENT-03
**Success Criteria** (what must be TRUE):
  1. A fresh installed Codex session completes its supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.
  2. A fresh installed Claude Code session independently completes its supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.
  3. A fresh installed OpenCode session independently completes its supported handshake/discovery and a safe real read-only BioRender tool call through the bridge, with exact versions and dated sanitized evidence.
**Plans**: TBD

Acceptance boundary: configuration, library tests and one client's success cannot substitute for another installed client's acceptance. Use safe read-only operations; current connectivity does not establish persistence.

### Phase 7: Persistence and Release Evidence
**Goal**: Users can judge release readiness from elapsed real-service persistence, separately labeled fault evidence and unresolved provider limits.
**Depends on**: Phase 6
**Requirements**: PROOF-01, PROOF-02, PROOF-03
**Success Criteria** (what must be TRUE):
  1. Users can inspect dated real-service evidence spanning at least two actual issued access-token expiry boundaries and one overnight sleep, including fresh sessions in all three harnesses; simulated expiry is labeled separately.
  2. Users can inspect separately labeled controlled evidence for concurrent refresh, crash/write failures, omitted replacement tokens, locked Keychain, offline recovery and terminal rejection, including zero background browser launches.
  3. Users can inspect a release readiness record with tested versions, elapsed persistence interval, security/dependency findings and unresolved provider limits; failed gates stay open and no permanent-authorization promise is made.
**Plans**: TBD

Acceptance gate: measure actual newly issued expiries; historical token lifetimes are not a substitute. Provider rejection or insufficient elapsed evidence leaves persistence acceptance open.

## Progress

**Execution Order:** 1 → 2 → 3 → 4 → 5 → 6 → 7, only after further user direction.

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Compatibility and Trust Contract | 0/TBD | Not started | - |
| 2. Deliberate Authorization | 0/TBD | Not started | - |
| 3. Durable Refresh and Visible Recovery | 0/TBD | Not started | - |
| 4. Isolated MCP Forwarding | 0/TBD | Not started | - |
| 5. Reversible Installation and Diagnostics | 0/TBD | Not started | - |
| 6. Three-Client Acceptance | 0/TBD | Not started | - |
| 7. Persistence and Release Evidence | 0/TBD | Not started | - |

Coverage: 31/31 v1 requirements assigned exactly once; 0 implemented. No v1 graphical UI phase is planned; optional native status UI remains v2.
