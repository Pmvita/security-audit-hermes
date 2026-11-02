# security-audit (Hermes Agent Skill)

Hermes Agent adaptation of the [Cloudflare security-audit skill](https://github.com/cloudflare/security-audit-skill).

## About

**security-audit-hermes** is a defensive, source-first security audit workflow built for the Hermes Agent. It answers security questions and performs comprehensive source audits without drifting into vulnerability hunting for its own sake.

Its two modes:

- **Guidance mode** — answer security questions, focused reviews, methodology.
- **Full audit mode** — a six-phase comprehensive audit with coverage tracking, hunting, validation, and reporting.

Everything the agent needs to do its job lives in the bundle, including the companion references in `references/` and domain companions (client-side, cloud/deployment, data isolation, desktop/mobile IPC, memory-safety, protocols, resource exhaustion, supply-chain, web protocol/auth).

## Installation

Copy this directory into your Hermes skills directory:

```bash
# Windows
cp -r security-audit ~/.hermes/skills/

# Linux/macOS
cp -r security-audit ~/.hermes/skills/
```

Or clone directly:

```bash
git clone https://github.com/<your-user>/hermes-agent-skill-security-audit ~/.hermes/skills/security-audit
```

## What it does

Defensive, source-first security audit workflow with:
- **Guidance mode**: Answer security questions, focused reviews, methodology
- **Full audit mode**: Six-phase comprehensive audit with coverage tracking, hunting, validation, and reporting

## Hermes Agent adaptations

- Uses `delegate_task` instead of "Task tool" for spawning subagents
- Uses `read_file`, `search_files`, `terminal`, `write_file` for source inspection and artifact creation
- Uses `execute_code` for complex data processing (coverage ledger validation, fingerprint consolidation)
- Output directory defaults to `~/security-audit-skill/<repo-name>/run-<N>`

## Companion files

The `references/` directory contains:
- `RECONNAISSANCE.md` — Phase 1: Map source, plan coverage, prior-run handling
- `HUNTING.md` — Phase 2: Coverage-led hunting waves, hunter prompts, coverage critic
- `VALIDATION-AND-REPORTING.md` — Phases 3-6: Validation, structured output, verification, reporting
- `ATTACK-CLASSES.md` — Ordinary attack classes and companion selection
- `AI-AND-LLM.md` — Domain companion: AI/LLM security

Additional domain companions (CLIENT-SIDE, CLOUD-AND-DEPLOYMENT, DATA-ISOLATION-AND-LIFECYCLE, DESKTOP-MOBILE-LOCAL-IPC, MEMORY-SAFETY-AND-BINARY, PROTOCOLS-RPC-MESSAGING, RESOURCE-EXHAUSTION, SUPPLY-CHAIN-AND-RELEASE, WEB-PROTOCOL-AND-AUTH) are available in the original [Cloudflare repository](https://github.com/cloudflare/security-audit-skill/tree/main/skills/security-audit).

## License

MIT — adapted from Cloudflare's security-audit-skill (MIT licensed).
