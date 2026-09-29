# Stack Research

**Domain:** Native macOS BioRender MCP authentication bridge
**Researched:** 2026-09-29
**Confidence:** MEDIUM — released source and official documentation inspected; no package resolution, compilation, or service installation performed during initialization.

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Swift / Swift Package Manager | Swift 6.1 minimum for the candidate SDK; exact build toolchain to pin in the first implementation phase | Compiled CLI and shared per-user broker | User-mandated native language; actors support a single credential owner; no Node runtime needed in production |
| Official MCP Swift SDK | Evaluate exact release **0.12.1**; do not track main | MCP client/server models and transports | Official released implementation with stdio and Streamable HTTP; adoption conditional on conformance/security spike |
| macOS | Proposed minimum 13.0; validate against target client/toolchain matrix | Native service platform | Matches SDK manifest and allows modern ServiceManagement APIs |
| Foundation / URLSession | OS SDK bundled with pinned Xcode/toolchain | HTTPS and network lifecycle | Platform networking rather than an extra HTTP client dependency |
| Security.framework | OS SDK | Keychain credential record | Native secret storage, explicit status handling |
| launchd per-user agent | OS facility | One broker across harness subprocesses | Shared ownership across process restarts; no root daemon needed |

The inspected [0.12.1 manifest](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Package.swift) specifies Swift tools 6.1 and macOS 13.0. Its README still says Swift 6.0 and illustrates dependency 0.11.0; use the manifest and actual tag as the compatibility authority. [Release 0.12.1](https://github.com/modelcontextprotocol/swift-sdk/releases/tag/0.12.1) includes authorization improvements; the release listing also identifies 0.12.0 as the authorization implementation milestone. These are released features, not promises inferred from main.

### Supporting Libraries and Dependency Boundaries

| Dependency | Declaration in SDK 0.12.1 | Purpose / adoption decision |
|------------|--------------------------|-----------------------------|
| apple/swift-system | from 1.0.0 | MCP library dependency |
| apple/swift-log | from 1.5.0 | MCP library logging; configure stderr/redaction, never MCP stdout |
| mattt/eventsource | from 1.1.0 | Conditional Apple-platform MCP library dependency; audit SSE parsing and transitive graph |
| apple/swift-nio | from 2.65.0 | Declared by SDK package for its conformance-server target, not a direct MCP library target dependency |
| swiftlang/swift-docc-plugin | **branch main** | Documentation dependency declared upstream; requires explicit resolution review even when not linked into runtime |

These are manifest constraints, **not resolved installed versions**. No complete transitive inventory is claimed. SDK pinning alone does not pin this entire graph. Before acceptance: resolve in the project, review every revision and license, commit Package.resolved, test locked-resolution builds, and distinguish packages fetched during resolution from targets compiled or plugins executed. If the branch dependency prevents an acceptable reproducible build, evaluate a narrowly documented vendor/fork policy rather than silently tolerating drift. [Tagged manifest](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Package.swift)

Do not add Swift Argument Parser, a database, ServiceLifecycle, a third-party Keychain wrapper, or a web framework until a concrete requirement justifies the added graph. This is a project dependency-minimization recommendation, not a claim those packages are unsafe.

### Actual Released Capability and Required Integration Work

The tagged SDK documents `Client`, `Server`, `StdioTransport`, `HTTPClientTransport`, stateful/stateless HTTP server transports, OAuth authorization code with PKCE, dynamic registration, token-provider hooks, and custom token storage. Default storage is memory-only. A browser delegate and callback handling remain application responsibilities. The HTTP server transport is a transport abstraction; do not assume it provides a packaged listener/service. [Tagged README](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/README.md), [stateful transport implementation](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Sources/MCP/Base/Transports/HTTPServer/StatefulHTTPServerTransport.swift)

**Use the SDK for protocol handling; keep durable authentication policy owned by BioRenderBridge.** Tagged `OAuthAuthorizer` has material integration hazards:

- It explicitly prohibits sharing one authorizer across multiple transports.
- Its 401 path clears storage before discovery/refresh finishes.
- Proactive refresh needs already-loaded authorization-server metadata and suppresses refresh errors.
- Saving a refresh response uses its refresh-token field directly; preservation when that field is omitted must be tested.
- Registration mutates in-memory authentication configuration; persistence of dynamic client credentials requires separate handling.

Recommendation: one broker-managed credential state machine, with a custom `HTTPClientAuthorizer` or narrowly reviewed integration layer. Retain valid credentials on transient failure, persist successful rotation before acknowledging readiness, and gate every interactive path behind explicit login. These source observations require targeted tests before choosing whether to reuse or replace built-in authorizer behavior. [Tagged authorizer source](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Sources/MCP/Base/Authorization/OAuthAuthorizer.swift)

`OAuthAccessToken` contains access/refresh token metadata and client identity, but is not a complete dynamic-registration secret store. Persist a versioned record containing issuer/resource binding, client registration credentials and expiration, token values, expiration, and rotation generation. Never assume a saved access-token object alone survives a full restart correctly. [Tagged OAuth models](https://github.com/modelcontextprotocol/swift-sdk/blob/0.12.1/Sources/MCP/Base/Authorization/OAuthModels.swift)

### Native Credential and Service Options

Use a private Keychain generic-password record managed only by the broker. Inspect every `OSStatus`; distinguish missing credentials, locked/unavailable Keychain, denial, corruption, and success. Prefer update semantics and avoid delete-then-add sequences that destroy the last valid record before a successful write. Define code-signing/access-control behavior and test installed executables, not merely a development CLI. [Apple Keychain services](https://developer.apple.com/documentation/security/keychain-services), [updating and deleting items](https://developer.apple.com/documentation/security/updating-and-deleting-keychain-items)

Start with a documented per-user LaunchAgent and deliberate install/uninstall commands. Apple documents launchd supervision and socket activation; use it for process lifecycle, not as an excuse to refresh continuously or open browsers. A future signed app bundle can use `SMAppService` to register its bundled agent and report approval status. A bare CLI is not automatically an app-bundle service. No background job is installed during initialization. [Apple launchd guide](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html), [SMAppService](https://developer.apple.com/documentation/servicemanagement/smappservice)

### Development Tools

| Tool | Purpose | Required evidence |
|------|---------|-------------------|
| Swift Testing / XCTest | Protocol fixtures, refresh state transitions, deterministic failure injection | Real expiration, invalid_grant, omitted refresh token, concurrent requests, Keychain failure |
| SwiftPM lockfile review | Dependency reproducibility | Full revisions/licenses plus reviewed updates |
| Official MCP conformance tooling | Protocol compatibility | Negotiated-version behavior and unsupported-capability rejection; tooling can remain development-only |
| Installed Codex, Claude Code, OpenCode | Acceptance | Separate initialize/discovery/tool-call results for each client |
| Native installed-service checks | Persistence acceptance | Restart, logout/login, sleep/wake, network loss, later actual expiry; zero unsolicited browser launches |

## Installation

**Deferred: initialization only.** No Package.swift, dependency downloads, build artifacts, service registration, credentials, or harness edits are required to finish project setup. At implementation, evaluate `.package(url: "https://github.com/modelcontextprotocol/swift-sdk.git", exact: "0.12.1")`, then commit the reviewed resolved graph. Recheck the current release before adoption; this is a researched candidate, not an already validated dependency lock.

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Official Swift SDK with audited auth boundary | Entire MCP implementation in Foundation | Only if measured SDK defects/dependency constraints outweigh maintaining a protocol implementation |
| CLI plus per-user broker | SwiftUI menu-bar app | Later native status/reconnect experience; not required to prove the core bridge |
| Stdio harness endpoints plus protected private broker IPC | Local Streamable HTTP service | If audited client compatibility and local authentication/Origin protection justify a listening HTTP endpoint |
| Native Keychain APIs | Keychain wrapper package | Only if reviewed maintainability benefit exceeds dependency cost |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Node/TypeScript production bridge | Conflicts with explicit language choice | Compiled Swift |
| Unpinned main dependencies / startup package downloads | Mutable supply-chain and runtime availability | Reviewed versions, lockfile, prebuilt binary |
| Three independent credential owners | Recreates refresh coordination problem | Single broker |
| Interactive SDK delegate on normal startup | Can reopen the unwanted login loop | Typed needs-login state and deliberate command |
| Long-lived secret in harness config or arguments | Spreads credential access | Broker-owned Keychain record |
| Guarantee of permanent authorization | Provider can revoke/expire grants | Measured persistence and explicit recovery |

## Version Compatibility and Open Gates

| Boundary | Finding | Remaining Gate |
|----------|---------|----------------|
| Swift SDK 0.12.1 ↔ Swift | Manifest requires 6.1 | Verify exact local/CI compiler and strict-concurrency behavior |
| SDK ↔ MCP | Tag describes 2025-11-25 support | Negotiate supported revision; don't label it newest without checking specification releases |
| SDK OAuth ↔ BioRender | Custom hooks exist | Replay sanitized discovery/error fixtures; verify provider-specific behavior without weakening generic validation |
| Broker ↔ Keychain | Platform APIs exist | Signed installed process authorization and wake/restart checks |
| SDK graph ↔ supply-chain policy | Mutable DocC declaration exists | Resolve/audit graph before adoption |

## Research Method and Confidence

GSD `research-plan` selected Jina for URL extraction; no Jina tool was exposed, so primary-source web fetch was used. Context7 resolved `/modelcontextprotocol/swift-sdk`, but returned main-branch documentation; those examples were cross-checked against tag 0.12.1. `gsd-tools query classify-confidence --provider context7 --verified` returned **MEDIUM**; the generic webfetch classification returned LOW, so raw fetches are treated as inspectable primary evidence, not runtime validation. No claim here proves the bridge or BioRender's persistence works. Release, manifest, source, and Apple references above are the reproducible evidence for the proposed stack.

---
*Stack research for BioRenderBridge; initialization only.*
