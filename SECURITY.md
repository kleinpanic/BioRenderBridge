# Security policy

BioRenderBridge is currently a planning-only repository. There is no supported runtime release yet.

## Reporting

Report suspected vulnerabilities privately through GitHub's Security tab if private vulnerability reporting is available. If it is unavailable, open an issue requesting a private reporting channel without including vulnerability details or secrets. Never post tokens, authorization codes, cookies, Keychain exports, private figures, or unredacted HTTP traces.

## Required security boundaries

- Credentials stay in the user's Keychain; repository files, process arguments, diagnostics, and client configurations must not contain upstream bearer or refresh tokens.
- A single broker owns refresh rotation; concurrent harnesses must not independently consume the same refresh token.
- Local IPC must verify the intended local user/client boundary. Loopback alone is not authentication. The final IPC mechanism is subject to design and security review.
- Browser authentication requires a deliberate action; OAuth state and PKCE are mandatory. Discovery/redirect destinations must be validated and restricted.
- Upstream content, schemas, and tool descriptions are untrusted data. Preserve protocol behavior without granting extra local privileges.
- Do not blindly retry non-idempotent figure creation after ambiguous transport failures.
- Audit direct and transitive dependencies; pin reviewed versions, preserve the resolution file, and prohibit runtime package fetching.
- Release signing, installation permissions, rollback, and uninstall must be verified before distribution.

A language choice is not a security guarantee. No claim of permanent authorization is valid without corresponding upstream support and measured lifecycle tests.
