# BioRenderBridge

A planned native Swift MCP adapter for reliable BioRender access from Codex, Claude Code, and OpenCode.

**Status: project initialized; no adapter implementation or installable release exists yet.** This repository contains the research, requirements, roadmap, and security boundaries for development. It is an independent project, not affiliated with or endorsed by BioRender.

## The problem

The previous setup produced repeated browser prompts and later authentication failures across independently authorized clients. Immediate tool discovery succeeded, but a saved refresh token was subsequently rejected with `invalid_grant`. BioRender also advertises OAuth discovery metadata that strict MCP clients reject. A local adapter can coordinate refresh, preserve credentials, and explain failures; it cannot override server-side expiration or revocation.

See the [BioRender audit](docs/biorender-audit.md) for observations, sources, uncertainties, and the boundary between a local defect and an upstream policy.

## Recurrence and feasibility gate

On September 30, the existing Codex connection again failed when BioRender rejected its saved refresh credential. The cause remains unresolved. The roadmap now requires controlled causal investigation and an explicit go/revise/blocked decision before product implementation. A Swift rewrite is not yet a demonstrated remedy. See the [updated audit](docs/biorender-audit.md).

## Intended behavior

- One locally managed upstream authorization used through all three clients.
- Swift implementation with minimal, pinned, reviewed dependencies; no Node.js runtime or package downloads at startup.
- Keychain-backed credentials and coordinated refresh-token rotation.
- Normal startup never opens a browser. Login is an explicit action.
- Clear authentication, network, and service-health status; bounded recovery attempts.
- MCP protocol negotiation and capabilities verified against the actual upstream and client versions.
- Safe handling of cancellations, pagination, structured results, and failures; no blind retry of figure-creation operations.

These are requirements, not claims about shipped behavior. Persistent access is accepted only after real token-expiry and later-session tests. A successful immediate check does not establish permanence.

## Project documents

- [Project context](docs/project/PROJECT.md)
- [Requirements and acceptance criteria](docs/project/REQUIREMENTS.md)
- [Phased roadmap](docs/project/ROADMAP.md)
- [Research summary](docs/project/research/SUMMARY.md)
- [Security policy](SECURITY.md)
- [Contributing](CONTRIBUTING.md)

## Development status

Initialization only. No Swift package, binary, daemon, installer, harness migration, or credentials are included. Phase execution remains paused pending the owner's next instruction. The first phase must validate the protocol/SDK compatibility assumptions before implementation proceeds.

Local GSD planning metadata is committed in `.planning/`; the public branch exports reviewed documents under `docs/project/`. See [publication boundary](docs/publication.md). GSD is development tooling, not a runtime dependency of the proposed Swift product. See [development workflow](docs/development-workflow.md).

## License

MIT. BioRender and other product names belong to their respective owners. This license grants no rights to BioRender content or services.
