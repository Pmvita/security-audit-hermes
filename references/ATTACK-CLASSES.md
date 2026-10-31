# Attack Classes

## Attack classes — choose and split based on Phase 1

Select attack classes relevant to the application type. Not every class applies to every codebase. The list below is a starting point; add application-specific classes from Phase 1 and split large codebases per subsystem. Frame work as finding, validating, fixing, and prioritizing vulnerabilities.

Keep validation to source review and bounded local fixtures; do not develop payload chains, test availability on live services, or take action in shared environments.

Use `confirmed` only when source evidence and bounded validation establish the full boundary and meaningful result. Use `needs_validation` when a specific deployment, provider, platform, identity, or runtime fact is unavailable; state the missing fact and the safe owner-observed or local check that resolves it.

> **Native / binary / kernel targets** (C/C++/Rust-unsafe, kernel modules, parsers and decoders, FFI, concurrent runtimes, binary loaders, JITs, firmware): use the memory-safety, integer/ABI, concurrency, binary-loader, and privileged-interface classes in [MEMORY-SAFETY-AND-BINARY.md](MEMORY-SAFETY-AND-BINARY.md).
>
> **AI / LLM / agent targets** (chatbots, RAG, persistent memory, tool-calling agents, MCP servers/clients, prompt assembly, or model-controlled actions): use [AI-AND-LLM.md](AI-AND-LLM.md).

## Core attack classes

### Authentication & Authorization
- Missing or bypassed authentication on endpoints
- IDOR (Insecure Direct Object Reference) — access to other users' resources
- Privilege escalation — horizontal or vertical
- JWT/token validation flaws (none algorithm, signature bypass, expiry)
- Session fixation, session hijacking
- OAuth/SAML misconfigurations

### Input Validation & Injection
- SQL injection (including second-order)
- Command injection
- SSRF (Server-Side Request Forgery)
- Path traversal / directory traversal
- XXE (XML External Entity)
- Deserialization vulnerabilities
- Template injection (SSTI)

### Data Exposure
- Sensitive data in logs, errors, or responses
- Mass assignment / over-posting
- Information disclosure via error messages
- PII leakage
- Insecure storage of secrets

### Business Logic
- Race conditions / TOCTOU
- Price manipulation / discount abuse
- Workflow bypass (skipping steps)
- Rate limiting bypass
- Replay attacks

### Cryptography
- Weak algorithms (MD5, SHA1 for security)
- Hardcoded keys/secrets
- Insufficient entropy
- Certificate validation bypass
- Padding oracle attacks

### Infrastructure & Deployment
- Missing security headers (CSP, HSTS, X-Frame-Options)
- CORS misconfiguration
- Debug endpoints exposed in production
- Default credentials
- Unnecessary services/ports exposed

### Export/Backup as exfiltration
Can a low-privilege user trigger an export, snapshot, or backup that includes data above their access level? Can they export other users' data? Does the export include deleted/draft/private content? Revision history that was supposed to be pruned?

### Import/Restore as injection
Can import overwrite existing data? Can it create records that bypass normal validation? Can it inject content into collections the user has no write access to? Does it respect the same permission model as the UI?

### Search/Filter/Sort as oracle
Can search queries reveal whether content exists that the user cannot directly access? Do filter parameters let users probe statuses, roles, or fields they should not know about? Does sorting by a hidden field reveal its values through result ordering?

### Notification/Webhook as SSRF
Can a user set a notification URL, webhook URL, or callback URL that the server fetches? Is it validated against internal networks? What about after a redirect?

### Cache poisoning
Can draft content be discovered through search, RSS feeds, sitemaps, or API listing endpoints? Can cache headers cause a CDN to serve private content publicly?

## Companion selection

Select domain companions based on trust-sensitive boundaries found in Phase 1:

| Companion | When to use |
|-----------|-------------|
| [AI-AND-LLM.md](AI-AND-LLM.md) | Language model participates in trust-sensitive decision |
| [CLIENT-SIDE.md](CLIENT-SIDE.md) | Browser/client-side code with security implications |
| [CLOUD-AND-DEPLOYMENT.md](CLOUD-AND-DEPLOYMENT.md) | Cloud infrastructure, IaC, deployment configs |
| [DATA-ISOLATION-AND-LIFECYCLE.md](DATA-ISOLATION-AND-LIFECYCLE.md) | Data storage, tenant isolation, lifecycle management |
| [DESKTOP-MOBILE-LOCAL-IPC.md](DESKTOP-MOBILE-LOCAL-IPC.md) | Desktop/mobile apps, local IPC, native code |
| [MEMORY-SAFETY-AND-BINARY.md](MEMORY-SAFETY-AND-BINARY.md) | C/C++/Rust unsafe, parsers, binary loaders |
| [PROTOCOLS-RPC-MESSAGING.md](PROTOCOLS-RPC-MESSAGING.md) | RPC, messaging, protocol implementations |
| [RESOURCE-EXHAUSTION.md](RESOURCE-EXHAUSTION.md) | DoS, resource limits, rate limiting |
| [SUPPLY-CHAIN-AND-RELEASE.md](SUPPLY-CHAIN-AND-RELEASE.md) | Dependencies, CI/CD, release processes |
| [WEB-PROTOCOL-AND-AUTH.md](WEB-PROTOCOL-AND-AUTH.md) | Web protocols, HTTP, authentication flows |

## Important

For any finding this agent reports, it must verify the full code path, not just surface appearance. If a cookie is missing `HttpOnly`, check whether the cookie contains security-sensitive data and whether JS needs to read it by design. If an error message contains a field name, check whether the field is ever actually populated with sensitive data. A flag is not a finding — trace the impact before reporting.
