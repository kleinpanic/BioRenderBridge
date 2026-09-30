# BioRender MCP interoperability and persistence audit

Audit date: 2026-09-29. Scope: project initialization, sanitized historical diagnostics, official documentation, and unauthenticated public HTTP discovery probes. No login, credential read, token refresh, tool execution, client configuration change, or provider contact was performed by this audit. This document is suitable for a public repository.

## Finding

The bridge addresses two separate problems: local lifecycle reliability and upstream OAuth interoperability. Public discovery is currently inconsistent with the selected MCP authorization baseline. Historical evidence also records a rejected refresh grant. The discovery issue does **not** establish the cause of the grant rejection. A single Swift credential owner can prevent local refresh races; it cannot make an expired or revoked upstream authorization valid.

## Evidence classes

| Class | Meaning | Limitation |
|---|---|---|
| Public probe | Fresh HTTP requests made during this audit without credentials | Describes discovery only, not authenticated availability |
| Historical diagnostic | Prior sanitized verification record and incident notes | Not independently replayed here; does not validate this unbuilt bridge |
| Documentation | Official sources linked below | Advertised behavior is not installed-client acceptance |
| Hypothesis | Candidate explanation awaiting discriminating evidence | Must not be presented as root cause |

The GSD provider classifier returned LOW for direct webfetch even with verified=true. Documentation-derived confidence retains that conservative label; exact public probe results are explicitly identified as measured observations rather than upgraded by inference. No compatible Jina tool was available, so official pages were fetched directly. Token lifetime and rejection causality remain unconfirmed.

## Fresh public discovery measurements

At **2026-09-29T20:20:52Z**, unauthenticated GET requests used HTTPS and Accept: application/json,text/event-stream. Only status, content type, public authentication challenge, and public metadata were retained.

| Path on https://mcp.services.biorender.com | Result |
|---|---|
| `/mcp` | HTTP 401; challenge points `resource_metadata` to `https://mcp.services.biorender.com/.well-known/oauth-authorization-server` |
| `/.well-known/oauth-authorization-server` | HTTP 200 JSON; authorization-server metadata, without `resource` or `authorization_servers` |
| `/.well-known/oauth-protected-resource` | HTTP 404 |
| `/.well-known/oauth-protected-resource/mcp` | HTTP 404 |

Advertised metadata:

| Field | Observed public value |
|---|---|
| issuer | `https://mcp.services.biorender.com` |
| authorization endpoint | `/oauth/authorize` on issuer |
| token endpoint | `/oauth/token` on issuer |
| registration endpoint | `/oauth/register` on issuer |
| scopes | profile, email, openid, offline_access |
| grant types | authorization_code, refresh_token |
| response types | code |
| token endpoint authentication | client_secret_post |
| PKCE methods | S256 |

MCP 2025-11-25 distinguishes protected-resource discovery from authorization-server discovery. RFC 9728 requires the resource identifier; MCP additionally requires authorization_servers in protected-resource metadata. The challenged document fails that shape. This is a scoped interoperability finding against those contracts, not a complete certification of BioRender's protocol implementation. Sources: [MCP authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization), [RFC 9728](https://www.rfc-editor.org/rfc/rfc9728.html).

No client registration, authorization, token, or revocation endpoint was invoked. The advertised refresh capability alone establishes neither a guaranteed lifetime nor a guarantee of refresh-token issuance for every registration.

## Historical incident evidence

Sanitized records supplied for project initialization describe the following:

| Observation | What it proves | What it does not prove |
|---|---|---|
| Earlier native Codex authorization rejected missing protected-resource `resource` field | A reported discovery incompatibility; consistent with today's public probe | Complete behavior of every client version |
| Legacy browser-opening startup helpers were replaced; stale processes were removed | A historical local remediation | That all future registration paths are clean |
| HTTP 400 invalid_grant after access-token expiration, with invalid/expired refresh-token description | The token endpoint rejected that historical refresh attempt | Whether expiration, revocation, rotation replay, client mismatch, or server defect caused it |
| session_expired despite an unexpired access-token JWT | Reported service refusal before JWT expiry | Whether the error denotes a browser/account session, OAuth grant, or MCP transport session |
| Reauthorization followed by successful refresh and authenticated discovery | Immediate access restored at the recorded time | Overnight persistence or unattended recovery from grant revocation |
| Eight-hour access-token lifetime (28,800 seconds) | Recorded access-token lifetime for those grants | Refresh-token absolute/idle lifetime |
| Fresh Claude Connected status; saved Codex/OpenCode tool calls without errors | Historical connectivity at 2026-09-29T19:52:55Z | A real tool-call verification for Claude in that record, or bridge acceptance |
| Tool discovery changed from five tools to four | Tool surface may change | That a fixed expected count is a durable success criterion |

The original sanitized records are kept outside this public project. No account identifiers, credential files, JWTs, client registrations, callback URLs, machine paths, or private search content are reproduced here.

## What invalid_grant does and does not mean

RFC 6749 defines invalid_grant broadly: invalid, expired, revoked, or mismatched grant credentials can produce it. It is not a diagnostic code for a single cause. RFC 9700 describes rotation, revocation, and inactivity expiration as authorization-server concerns. These standards explain plausible failure classes; they do not identify which BioRender policy caused this incident. [RFC 6749 section 5.2](https://www.rfc-editor.org/rfc/rfc6749.html#section-5.2), [RFC 9700 section 4.14](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14).

Candidate explanations requiring later evidence include refresh-token expiration; invalidation after logout or another security event; stale token reuse after rotation; a client-binding mismatch; or a service defect. None is established. Independent existing harness authorizations must not be described as proven competing writers of the same token. Centralized ownership prevents a future class of races, not necessarily this past failure.

## Official documentation scope

The BioRender connector guide, updated September 13, 2026, describes connection, search, figure creation, and authentication troubleshooting. It does not specify refresh lifetime or an unattended-persistence guarantee. It notes that generation can consume credits, making search preferable for later non-mutating acceptance checks. General web-account logout guidance must not be treated as an MCP token contract. [BioRender connector guide](https://help.biorender.com/hc/en-gb/articles/30870978672157-How-to-use-the-BioRender-MCP-connector).

## Required bridge behavior derived from this audit

1. Use one upstream authorization owned by the broker; never copy independent harness tokens between clients.
2. Implement standards-based discovery first. If necessary, isolate a documented, exact-origin BioRender compatibility rule. Do not globally accept malformed metadata or arbitrary issuer/token endpoints. Record when the rule is active and retain a removal test.
3. Treat the broker as an upstream OAuth client, not an unauthenticated token vending machine. Downstream callers receive MCP results, never upstream credentials. Define a same-user IPC trust boundary before choosing transport.
4. Serialize refresh across every process touching a grant. Commit rotated credentials durably before releasing waiters. Preserve credentials on transient failures; do not blind-retry an uncertain refresh response that may already have rotated the token.
5. Distinguish network unavailability, locked Keychain, transient service failure, protocol mismatch, insufficient scope, and terminal grant rejection. Terminal rejection must become an explicit reconnect-required state, with no browser launch until requested.
6. Preserve MCP lifecycle, negotiated capabilities, cancellation and error semantics. Do not turn failed upstream tools/list into an empty successful list. Never replay a possibly completed figure mutation automatically.
7. Pass actual expiration, restart, concurrency, sleep/wake and offline recovery tests. Verify every installed harness with discovery and a safe real tool call. Run a later-session soak covering at least two actual access-token expiration boundaries and one overnight sleep; report duration and limits, never permanent success.

These are proposed requirements for subsequent implementation. None is marked implemented by this initialization audit.

## Remaining provider questions

- What are the refresh-token absolute and idle lifetimes, rotation semantics, and supported client-registration policy?
- Does session_expired identify an account session, grant session, or a different service state?
- Do account logout, password/security events, or concurrent independent grants revoke MCP grants?
- Will the challenged protected-resource metadata and well-known routes be corrected?
- Is client_secret_post dynamic registration the intended native-client integration, and are resource indicators supported correctly?

No support message has been sent. These questions are for a later explicitly authorized provider inquiry.

## Audit limits

This audit proves the public discovery responses at one timestamp. Authenticated protocol, token lifetimes, SDK behavior and native acceptance require later controlled tests. A shared adapter can improve persistence while authorization remains valid and make failure actionable; the project must not promise never to require login again.

## September 30 recurrence and implementation gate

At **2026-09-30T17:27:48Z**, the current Codex 0.159.0 startup log again recorded BioRender initialization failing because its HTTP headers helper exited with status 1. A direct helper check later in the same diagnostic turn reproduced `invalid_grant` from the saved refresh credential and exited 1. The helper reported that no browser was opened. No token value, account identifier or private log dump is retained here.

This is new evidence that the existing setup did not sustain authorization after the earlier recovery. It does not establish whether expiry, revocation, rotation, client binding or an upstream defect caused rejection, and it does not establish current Claude or OpenCode failure: those clients were not freshly tested in this incident. Reauthorization is temporary recovery, not closure of the persistence defect.

A separate startup failure at **2026-09-30T17:27:47Z** concerned Peekaboo's daemon ownership record and selected socket. It is a separate operational follow-up, not an OAuth failure and not a BioRenderBridge deliverable. No Peekaboo repair occurred here.

### Required causal investigation

Before Phase 2 product implementation, Phase 1 must produce a sanitized evidence timeline and distinguish competing explanations using controlled tests. Record actual token issue/expiry times, registration binding, credential generation/write transitions and refresh request timing without storing tokens or private content. Compare a single-owner baseline with concurrent clients using controlled independent grants; do not assume separate harness authorizations share or race on one token. Control restart, sleep and network variables. Do not deliberately revoke a working grant or contact the provider without appropriate user authorization.

| Hypothesis | Discriminating evidence needed | Status |
|---|---|---|
| Local stale/rotated token reuse | Generation and write-order trace showing reuse or excluding it under a single owner | Unknown |
| Lost rotation response or failed persistence | Request outcome and durable-write fault evidence; uncertain responses labeled explicitly | Unknown |
| Client or registration mismatch | Protected comparison of registration binding and token request construction | Unknown |
| Provider absolute/idle expiry or revocation | Controlled elapsed-time observations or authoritative provider clarification | Unknown |
| Upstream service defect or separate session policy | Reproducible minimal case after local hypotheses are tested; provider evidence if available | Unknown |

### Go / revise / blocked decision

Record which observed failure a shared Swift broker can prevent and the evidence linking the proposed mechanism to that failure. A new language, immediate refresh success, malformed discovery metadata or theoretical race prevention is insufficient causal proof.

- **Go:** evidence supports a locally correctable failure and the proposed mechanism passes a targeted reproduction, with remaining limitations stated. This permits implementation; it is not persistence acceptance.
- **Revise:** evidence favors another mechanism or integration; revise the architecture and requirements before Phase 2.
- **Blocked:** provider policy/behavior or insufficient evidence prevents a supported durable remedy. Keep the gate open, record the dependency and next discriminating test; do not silently downgrade the objective to repeated manual login.

Small diagnostic probes and feasibility experiments are allowed within Phase 1 when that phase is authorized. Product implementation, installation and harness migration remain gated. Retain Phase 7's real-expiration and overnight acceptance after any proposed remedy.
